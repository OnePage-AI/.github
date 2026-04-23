<div align="center">
  <img width="475" height="512" alt="KakaoTalk_20260421_230609684" src="https://github.com/user-attachments/assets/70c73298-4a48-406f-a299-b7ae23258f76" />

  <h2>원페이지(OnePage)</h2>
  <p>
    사용자가 업로드한 강의 자료(PDF, PPTX, DOCX)를 AI가 분석하여 <br>
    학습에 최적화된 정리 결과물, 요약, 퀴즈를 생성해주는 학습 솔루션입니다.
  </p>
  <p>
    <strong>"강의 내용을 덜 정리하고, 더 이해하며, 시험 대비까지 연결"</strong>하는 것을 목표로, <br>
    효율적인 학습 경험을 제공해드립니다.
  </p>
</div>

<br>

<h2>목차</h2>
<ul>
  <li><a href="#프로젝트-소개">프로젝트 소개</a></li>
  <li><a href="#문제-정의">문제 정의</a></li>
  <li><a href="#핵심-기능">핵심 기능</a></li>
  <li><a href="#서비스-사용-흐름">서비스 사용 흐름</a></li>
  <li><a href="#이런-분께-추천해요">이런 분께 추천해요</a></li>
  <li><a href="#현재-상태-및-향후-계획">현재 상태 및 향후 계획</a></li>
</ul>

<h2 id="프로젝트-소개">🔍 프로젝트 소개</h2>
<p>
  원페이지는 방대한 강의 자료를 <strong>Google Gemini 기반의 3단계 하이브리드 AI 파이프라인</strong>으로 재구성합니다. 
  단순한 텍스트 요약을 넘어, 실제 강의 맥락을 유지하면서 시험에 꼭 필요한 정보만을 선별하여 제공하는 학습 보조 서비스입니다.
</p>

<div align="center">
  <br>
</div>

<h2 id="문제-정의">❓ 문제 정의</h2>
<p>일반적인 LLM(GPT, Gemini 등)을 학습에 활용할 때 많은 학우들이 다음과 같은 한계를 느낍니다.</p>
<ul>
  <li><strong>정보의 누락:</strong> 자료가 길어지면 AI가 임의로 내용을 생략하거나 문맥을 놓침</li>
  <li><strong>부정확한 환각:</strong> 강의안에 없는 내용을 지어내어 학습에 혼선을 초래</li>
  <li><strong>재정리의 늪:</strong> 가독성이 낮아 결국 사용자가 다시 '요약의 요약'을 해야 하는 비효율 발생</li>
</ul>

<h2 id="핵심-기능">💡 핵심 기능</h2>
<p>원페이지는 기술적 차별화를 통해 학습의 질을 높입니다.</p>
<ul>
  <li><strong>환각 없는 기술:</strong> 외부 데이터 없이 오직 업로드된 강의 자료 내에서만 근거를 찾아 답변하여 정확도를 극대화했습니다.</li>
  <li><strong>대용량 자료 청킹 분석:</strong> 각 페이지를 단위별로 분리 분석한 뒤 다시 구조화하여, 수백 페이지의 전공 서적도 디테일 누락 없이 요약합니다.</li>
  <li><strong>학습 맞춤형 3가지 모드:</strong>
    <ul>
      <li><strong>강의 복습 모드:</strong> 전체 흐름을 깊이 있게 이해할 수 있는 체계적인 노트 정리</li>
      <li><strong>벼락치기 모드:</strong> 시험 직전 빠르게 암기할 수 있는 고밀도 핵심 요약</li>
      <li><strong>퀴즈 생성 모드:</strong> 자가 진단을 위한 객관식/주관식 문제 및 상세 해설 제공</li>
    </ul>
  </li>
</ul>

<div align="center">
  <img width="303" height="429" alt="image" src="https://github.com/user-attachments/assets/6db2a07b-7928-41b7-95b6-8b325f19d522" />
  <br>
</div>

<h2 id="서비스-사용-흐름">🛣️ 서비스 사용 흐름</h2>
<ol>
  <li><strong>로그인:</strong> Google 계정을 통해 간편하게 시작합니다.</li>
  <li><strong>자료 업로드:</strong> 분석을 원하는 PDF, PPTX, DOCX 파일을 업로드합니다.</li>
  <li><strong>학습 모드 선택:</strong> '복습', '벼락치기', '퀴즈' 중 필요한 모드를 선택합니다.</li>
  <li><strong>AI 분석 및 확인:</strong> 다단계 프롬프트를 거친 최적의 정리본을 즉시 확인합니다.</li>
  <li><strong>보관 및 관리:</strong> 결과물은 개인 보관함에 자동 저장되어 언제든 다시 열람할 수 있습니다.</li>
</ol>

<div align="center">
  <img width="614" height="308" alt="image" src="https://github.com/user-attachments/assets/9082951e-b638-49fa-b944-97a494ae966c" />
  <br>
</div>

<h2 id="이런-분께-추천해요">👍 이런 분께 추천해요</h2>
<ul>
  <li>정리 작업보다 실제 '공부와 암기'에 더 많은 시간을 쓰고 싶은 분</li>
  <li>방대한 전공 강의 자료의 핵심을 빠르게 파악해야 하는 시험 기간의 학우</li>
  <li>단순 요약이 아니라 믿을 수 있는 정확한 학습 자료가 필요한 분</li>
</ul>

<h2 id="향후-계획">🚀 향후 계획</h2>
<h3>앞으로의 원페이지</h3>
<ul>
  <li><strong>STT(Speech-to-Text) 통합:</strong> 음성 강의 파일을 텍스트로 변환하여 즉시 요약하는 기능 추가</li>
  <li><strong>학습 패턴 분석:</strong> 사용자별 취약점을 파악하여 개인화된 학습 가이드 제공</li>
</ul>
