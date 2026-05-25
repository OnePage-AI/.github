> 요약
프로젝트 진행 중 "로그인 직후 바로 페이지를 새로고침하면 로그아웃되는 현상"을 발견했습니다.
원인은 백엔드 로직의 Race Condition(경쟁 상태) 때문이었으며, DB 의존성을 제거한 Stateless 토큰 검증 방식으로 변경하여 해결했습니다.
>

---

## 1. 문제 현상 정의

버그의 재현 조건은 매우 구체적이었습니다. API 호출 여부가 성공과 실패를 가르는 핵심 변수였습니다.

| 시나리오 | 행동 순서 | 결과 |
| --- | --- | --- |
| A (실패) | 로그인 → 즉시 새로고침(F5) | 로그아웃 |
| B (성공) | 로그인 → API 호출(AI 요약 등) 1회 수행 → 새로고침 | 로그인 유지 |

의문점: 왜 API를 한 번이라도 호출하고 나면 로그인이 유지될까?

---

## 2. 원인 분석 과정

### 가설 1: 프론트엔드 토큰 갱신 로직 오류?

처음에는 `apiClient.ts`의 인터셉터 로직을 의심했습니다.
하지만 코드를 확인한 결과, `401 Unauthorized` 발생 시 토큰을 갱신하는 로직은 정상적으로 구현되어 있었습니다.

```tsx
// apiClient.ts
export const apiFetch = async (url: string, options: RequestInit = {}): Promise<Response> => {
    // ... 초기 요청 ...
    let response = await fetch(url, { ...options, headers });

    // 401 에러 감지 시 토큰 갱신 시도
    if (response.status === 401) {
        console.log("[apiFetch] Token expired. Attempting to refresh...");
        const newAccessToken = await refreshToken(); // 갱신 요청

        if (newAccessToken) {
            headers.set("Authorization", `Bearer ${newAccessToken}`);
            response = await fetch(url, { ...options, headers }); // 재요청
        }
    }
    return response;
};

```

결론: 프론트엔드 로직은 정상이며, 백엔드에서 401을 반환하는 원인을 찾아야 했습니다.

### 가설 2: Race Condition(경쟁 상태) – Root Cause

"로그인 직후"라는 타이밍에 주목하여 백엔드와 DB 사이의 흐름을 분석했습니다.

그 결과, DB 저장 지연(Latency)으로 인한 경쟁 상태가 원인임을 확인했습니다.

### 문제 발생 흐름

1. Frontend: 로그인 요청
2. Backend: 토큰 생성 후 DB 저장 명령(비동기) 수행, 동시에 프론트엔드로 응답 반환
3. Frontend: 응답 수신 직후 새로고침 → 토큰 재발급 요청(reissue)
4. Backend: 재발급 요청 수신 → DB에서 Refresh Token 조회
5. DB: 아직 저장이 완료되지 않음
6. Backend: 토큰이 존재하지 않는 것으로 판단 → 401 에러 반환
7. Frontend: 로그아웃 처리

핵심 원인:

백엔드가 토큰을 발급한 직후 DB 저장이 완료되기 전에, 프론트엔드가 해당 토큰의 유효성을 검증 요청하면서 문제가 발생했습니다.

---

## 3. 해결 방안 및 코드 변경

Race Condition을 근본적으로 해결하기 위해, 토큰 재발급 시 DB 조회 의존성을 제거했습니다.

JWT(Refresh Token)는 자체 서명(Signature)을 포함하므로, 서명과 만료 검증만 통과하면 토큰의 유효성을 보장할 수 있습니다.

### AS-IS (기존 코드)

토큰 검증 이후 DB를 한 번 더 조회하는 로직이 문제였습니다.

```java
// MemberCommandService.java
@Override
public MemberResponseDTO.AccessTokenResponse reissueToken(String refreshToken) {
    // 1. 토큰 서명 검증
    jwtProvider.validateToken(refreshToken);

    // 2. DB 조회 (Race Condition 발생 지점)
    MemberToken memberToken = memberTokenRepository.findByRefreshToken(refreshToken)
            .switchIfEmpty(Mono.error(new JwtHandler(ErrorStatus.INVALID_REFRESH_TOKEN)))
            .block();

    // ... 토큰 발급 로직 ...
}

```

### TO-BE (개선된 코드)

DB 조회를 제거하고, Refresh Token의 payload에서 사용자 정보를 추출해 Access Token을 재발급합니다.

```java
// MemberCommandService.java
@Override
public MemberResponseDTO.AccessTokenResponse reissueToken(String refreshToken) {
    // 1. 토큰 서명 및 만료 검증
    jwtProvider.validateToken(refreshToken);

    // 2. 토큰 payload에서 사용자 정보(uid) 추출
    String uid = jwtProvider.getMemberUid(refreshToken);

    // 3. 새 Access Token 발급
    String newAccessToken = jwtProvider.generateAccessToken(uid);
    LocalDateTime newAccessTokenExpireAt = jwtProvider.getExpiredAt(newAccessToken);

    log.info("Access token reissued for UID: {}", uid);

    return MemberResponseDTO.AccessTokenResponse.builder()
            .accessToken(newAccessToken)
            .accessTokenExpireAt(newAccessTokenExpireAt)
            .build();
}

```

---

## 4. 최종 결과

1. Race Condition 해결
    1. DB I/O 속도와 관계없이 토큰 검증이 가능해져, 로그인 직후 새로고침 시에도 로그인이 안정적으로 유지됩니다.
2. 성능 향상
    1. 토큰 재발급 시 발생하던 DB 조회 쿼리가 제거되어 응답 속도가 개선되었습니다.
3. Stateless 아키텍처 강화
    1. JWT의 상태 비저장 특성을 더 잘 살린 구조로 개선되었습니다.

### 인증 방식 Stateful vs Stateless 결정

`reissueToken` 메서드에서 DB 조회를 제거하자, 한 가지 중요한 질문이 생겼습니다.

> "이제 리프레시 토큰을 DB에 저장하는 MemberToken 엔티티 자체가 불필요해진 것 아닌가?"
>

재발급 로직이 더 이상 DB를 읽지 않는다면, 로그인 시마다 DB에 토큰을 저장하는 로직은 불필요한 작업이 될 수 있습니다.

### 1. 서버 측 토큰 무효화 기능

분석 결과, 기존 `MemberToken` DB는 단순 저장 외에 또 다른 역할을 하고 있었습니다.

바로 로그아웃 시 토큰을 무효화(Revoke)하는 기능입니다.

- 기존 방식 (Stateful)
    1. 로그인 시 리프레시 토큰을 DB에 저장한다.
    2. 로그아웃 시 `logout` 메서드가 DB에서 해당 리프레시 토큰을 삭제한다.
    3. 토큰 탈취 후 재발급을 시도하더라도, DB에 토큰이 존재하지 않으므로 재발급을 거부할 수 있다. (수정 전 로직 기준)
- 현재 상태 (Stateless 재발급)
    - `reissueToken` 로직에서 DB 조회를 제거하면서, 로그아웃 시 DB에서 토큰을 삭제하더라도 재발급을 막을 수 없는 구조가 되었다.
    - 이로 인해 로그아웃 이후에도 리프레시 토큰의 유효기간이 만료될 때까지 재사용이 가능한 상태가 되었다. 기존 기준으로는 최대 14일간 유효하다.

### 2. Stateless vs Stateful

이 문제를 해결하기 위해 인증 아키텍처의 방향성을 다시 검토하게 되었습니다.

| 구분 | Stateless 방식 (상태 비저장) | Stateful 방식 (상태 저장) |
| --- | --- | --- |
| 동작 | 토큰 자체의 유효성을 암호학적으로 검증 | 토큰 유효성 + DB에 저장된 상태를 함께 검증 |
| 장점 | 구조 단순, 확장 용이DB 부하 없음으로 빠른 처리 가능Race Condition 원천 차단 | 서버에서 토큰 강제 무효화 가능모든 기기 로그아웃 등 보안 기능 구현 가능활성 세션 관리 용이 |
| 단점 | 서버에서 토큰을 강제로 무효화할 수 없음 | 구조가 복잡해지고 DB 부하 발생Race Condition 등 구현 난이도 증가 |
| 적용 사례 | 대부분의 현대적인 웹·모바일 서비스 | 금융, 결제 등 높은 보안 수준이 요구되는 서비스 |

## 3. 최종 결정: 절충안 채택

검토 끝에 다음과 같은 절충안을 선택했습니다.

Stateless 방식을 채택하여 Race Condition 문제를 해결하되, 리프레시 토큰의 유효 기간을 기존 14일에서 1일로 대폭 단축하는 방향입니다.

- 구조 단순화 및 안정성 확보

  Stateful 방식에서 발생할 수 있는 복잡한 동기화 문제와 Race Condition을 피하고, 인증 구조를 단순하게 유지합니다.

- 보안 리스크 완화

  Stateless 방식의 한계인 토큰 탈취 위험을 유효 기간 단축으로 보완합니다. 리프레시 토큰이 탈취되더라도 공격자가 사용할 수 있는 시간은 최대 1일로 제한되어, 실질적인 보안 위험을 수용 가능한 수준으로 낮출 수 있습니다.


이 결정에 따라 `MemberToken` 엔티티와 관련된 DB 로직을 모두 제거하고, 리프레시 토큰의 유효 기간을 1일로 변경하는 방향으로 최종 정리했습니다.