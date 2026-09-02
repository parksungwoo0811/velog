<blockquote>
<p>이 글은 <strong>「머신러닝 및 딥러닝 이해 - Part A: Machine Learning」</strong> 교재를 처음 접하는 학습자를 위한 개념 중심 학습 노트입니다. 원문의 문장을 그대로 옮기기보다 핵심 개념을 쉬운 말과 예시로 재구성했습니다.</p>
<p>교재에 포함된 개인 식별 정보, 내부 표기, 배포 관련 문구, 로컬 파일 경로 등 학습에 필요하지 않은 정보는 공개하지 않았습니다. 이 글은 Part A의 세 대단원, 참고 사례, 요약 내용, 퀴즈를 모두 다룹니다.</p>
</blockquote>
<hr />
<h2 id="전체-지도">전체 지도</h2>
<p>이 교재의 흐름은 다음과 같습니다.</p>
<ol>
<li><strong>Artificial Intelligence</strong><ul>
<li>AI의 역사</li>
<li>AI, ML, DL의 관계</li>
<li>통계와 머신러닝의 접근 차이</li>
<li>분류와 회귀</li>
<li>모델 성능에 영향을 주는 요인</li>
</ul>
</li>
<li><strong>Machine Learning</strong><ul>
<li>머신러닝의 정의</li>
<li>Task, Experience, Performance</li>
<li>Train, Validation, Test</li>
<li>지도, 준지도, 비지도학습</li>
</ul>
</li>
<li><strong>ML Algorithm</strong><ul>
<li>알고리즘과 모델의 차이</li>
<li>Decision Tree, Random Forest, Boosting</li>
<li>SVM</li>
<li>LASSO, Ridge와 규제</li>
<li>상황별 알고리즘 선택</li>
</ul>
</li>
</ol>
<p>큰 그림을 한 문장으로 줄이면 다음과 같습니다.</p>
<blockquote>
<p><strong>AI는 지능적인 일을 수행하게 만드는 큰 분야이고, ML은 데이터에서 규칙을 배우는 AI의 한 방법이며, DL은 여러 층의 신경망을 이용하는 ML의 한 방법이다.</strong></p>
</blockquote>
<hr />
<h1 id="part-1-artificial-intelligence">Part 1. Artificial Intelligence</h1>
<h2 id="1-인공지능의-역사를-왜-배울까">1. 인공지능의 역사를 왜 배울까?</h2>
<p>AI의 역사는 기술 이름을 외우기 위한 연대표가 아닙니다. AI가 반복해서 겪은 다음 패턴을 이해하는 데 목적이 있습니다.</p>
<pre><code class="language-text">새로운 아이디어 등장
→ 기대가 크게 상승
→ 현실의 한계 발견
→ 투자와 관심 감소
→ 데이터·연산·알고리즘의 발전
→ 다시 도약</code></pre>
<p>즉, 오늘날의 생성형 AI도 갑자기 나타난 마법이 아니라 오랜 연구가 축적된 결과입니다.</p>
<h2 id="2-주요-ai-역사">2. 주요 AI 역사</h2>
<h3 id="21-초기-신경망과-퍼셉트론">2.1 초기 신경망과 퍼셉트론</h3>
<p>1940년대에는 생물의 신경세포를 계산 구조로 흉내 내려는 연구가 시작됐습니다. 이후 퍼셉트론은 입력마다 가중치를 곱한 뒤 합산하여 결과를 판단하는 방식을 제시했습니다.</p>
<pre><code class="language-text">입력값 × 중요도(가중치)
→ 모두 더하기
→ 기준보다 크면 1, 작으면 0</code></pre>
<p>예를 들어 대출 승인 여부를 판단할 때 소득, 부채, 연체 이력에 서로 다른 중요도를 주는 것과 비슷합니다.</p>
<p>하지만 단순 퍼셉트론은 직선 하나로 나눌 수 없는 문제, 대표적으로 XOR 문제를 풀지 못했습니다. 이를 극복하기 위해 신경망을 여러 층으로 쌓고, 순전파와 역전파로 가중치를 학습하는 방향이 발전했습니다. 이후 Neocognitron과 CNN 같은 구조로 이어졌습니다.</p>
<h3 id="22-1956년-다트머스-회의">2.2 1956년 다트머스 회의</h3>
<p>다트머스 회의에서 <strong>Artificial Intelligence</strong>라는 용어가 공식적으로 사용됐습니다. 지능을 기계로 구현할 수 있다는 연구 분야가 독립적으로 출발한 사건입니다.</p>
<h3 id="23-eliza와-첫-번째-ai-겨울">2.3 ELIZA와 첫 번째 AI 겨울</h3>
<p>ELIZA는 사용자의 문장에서 특정 표현을 찾아 정해진 문장으로 되묻는 초기 대화 프로그램입니다.</p>
<pre><code class="language-text">사용자: 요즘 기분이 우울해요.
ELIZA: 왜 기분이 우울하다고 생각하나요?</code></pre>
<p>겉으로는 상담 내용을 이해하는 것처럼 보이지만 실제로는 의미를 이해하지 않고 <strong>패턴 매칭</strong>을 수행합니다. 사람은 이런 반응에도 감정이나 지능이 있다고 느낄 수 있는데, 이를 <strong>ELIZA 효과</strong>라고 합니다.</p>
<p>초기 시스템이 기대만큼 복잡한 문제를 풀지 못하면서 과도한 기대와 현실 사이의 차이가 드러났고, 연구 지원이 줄어든 첫 번째 AI 겨울로 이어졌습니다.</p>
<h3 id="24-전문가-시스템과-두-번째-ai-겨울">2.4 전문가 시스템과 두 번째 AI 겨울</h3>
<p>1980년대 전문가 시스템은 사람이 정한 지식을 <code>if-then</code> 규칙으로 저장해 기업의 의사결정에 활용했습니다.</p>
<pre><code class="language-text">IF 장비 온도 &gt; 90도 AND 진동이 큼
THEN 고장 위험 경고</code></pre>
<p>실무 활용 가능성을 보여줬지만 규칙이 많아질수록 유지·보수가 어려웠습니다. 새로운 상황마다 사람이 규칙을 추가해야 했기 때문에 확장성이 낮았고, 다시 투자와 관심이 줄어드는 두 번째 AI 겨울이 왔습니다. 이 경험은 사람이 모든 규칙을 작성하는 방식보다 데이터에서 규칙을 배우는 방식이 필요하다는 점을 부각했습니다.</p>
<h3 id="25-deep-blue">2.5 Deep Blue</h3>
<p>1997년 IBM의 Deep Blue는 체스 세계 챔피언을 이겼습니다. Deep Blue는 체스 규칙, 평가 함수, 대규모 탐색을 결합한 강력한 특정 목적 시스템이었습니다.</p>
<p>ELIZA와 Deep Blue를 비교하면 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>ELIZA</th>
<th>Deep Blue</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>상담 대화 모사</td>
<td>체스 경기</td>
</tr>
<tr>
<td>핵심 방식</td>
<td>문장 패턴 매칭</td>
<td>규칙, 평가, 대규모 탐색</td>
</tr>
<tr>
<td>강점</td>
<td>지능적인 대화처럼 보임</td>
<td>제한된 문제에서 인간 최고 수준을 능가</td>
</tr>
<tr>
<td>한계</td>
<td>의미를 이해하지 못함</td>
<td>체스 밖의 문제로 일반화하기 어려움</td>
</tr>
</tbody></table>
<p>둘 다 중요한 초기 AI지만, 오늘날 ML처럼 데이터에서 폭넓게 학습하고 다른 문제에 일반화하는 시스템은 아니었습니다.</p>
<h3 id="26-딥러닝의-부활과-현대-ai">2.6 딥러닝의 부활과 현대 AI</h3>
<table>
<thead>
<tr>
<th>시기</th>
<th>사건</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>2012</td>
<td>AlexNet</td>
<td>CNN과 GPU 학습이 이미지 인식 성능을 크게 높임</td>
</tr>
<tr>
<td>2016</td>
<td>AlphaGo</td>
<td>딥러닝, 강화학습, 탐색을 결합해 바둑에서 성과를 냄</td>
</tr>
<tr>
<td>2018</td>
<td>BERT</td>
<td>Transformer 기반 양방향 문맥 학습으로 자연어 이해를 발전시킴</td>
</tr>
<tr>
<td>2022</td>
<td>ChatGPT 공개</td>
<td>대규모 언어 모델을 일반 사용자가 대화형으로 이용하기 시작</td>
</tr>
<tr>
<td>2023 이후</td>
<td>멀티모달</td>
<td>텍스트뿐 아니라 이미지, 음성 등 여러 데이터 유형을 함께 처리</td>
</tr>
<tr>
<td>2024 이후</td>
<td>RAG 확산</td>
<td>외부 지식을 검색해 생성 결과의 근거와 최신성을 보완</td>
</tr>
<tr>
<td>2025 이후</td>
<td>AI Agent 확산</td>
<td>사용자의 의도를 이해하고 도구를 사용해 작업까지 수행하는 방향</td>
</tr>
</tbody></table>
<p>게임 AI도 체커, 체스, 바둑, Dota 2, StarCraft II처럼 더 복잡한 환경으로 발전했습니다. 다만 게임을 잘하는 AI가 모든 현실 문제를 잘 푼다는 뜻은 아닙니다. 특정 과제에서 뛰어난 능력과 범용 지능은 구분해야 합니다.</p>
<h2 id="3-ani-생성형-ai-agi">3. ANI, 생성형 AI, AGI</h2>
<p>AI의 능력 범위를 기준으로 다음처럼 생각할 수 있습니다.</p>
<ul>
<li><strong>ANI(Artificial Narrow Intelligence)</strong>: 한정된 과제를 잘 수행하는 AI. 추천, 번역, 불량 검출 등이 해당합니다.</li>
<li><strong>생성형 AI(Generative AI)</strong>: 학습한 데이터의 패턴을 바탕으로 텍스트, 이미지, 코드 같은 새 결과물을 생성합니다.</li>
<li><strong>AGI(Artificial General Intelligence)</strong>: 인간처럼 여러 영역의 문제를 폭넓게 이해하고 학습하는 개념적 목표입니다.</li>
</ul>
<p>생성형 AI는 한 종류의 입력만 다루는 단일 모달에서 텍스트·이미지·음성을 함께 다루는 멀티모달로 발전했습니다. 여기에 로봇이나 소프트웨어 도구를 연결하면 인식만 하는 데서 벗어나 행동까지 수행할 수 있습니다.</p>
<p>주의할 점은 현재의 뛰어난 생성형 AI를 곧바로 완성된 AGI와 같다고 볼 수 없다는 것입니다. 유창한 답변과 안정적인 이해·추론·책임 있는 실행은 서로 다른 문제입니다.</p>
<h2 id="4-ai-ml-dl의-관계">4. AI, ML, DL의 관계</h2>
<p>세 개념은 서로 경쟁하는 용어가 아니라 포함 관계입니다.</p>
<pre><code class="language-text">Artificial Intelligence
└── Machine Learning
    └── Deep Learning</code></pre>
<ul>
<li><strong>AI</strong>: 지각, 언어 처리, 학습, 판단, 로봇 행동 등 지능적 기능 전체</li>
<li><strong>ML</strong>: 사람이 규칙을 모두 작성하지 않고 데이터에서 패턴을 학습하는 방법</li>
<li><strong>DL</strong>: 여러 층의 인공신경망을 사용하는 ML 방법</li>
</ul>
<p>Random Forest, SVM, AdaBoost, 규제 회귀는 머신러닝이지만 보통 딥러닝이라고 부르지 않습니다. CNN, RNN, GAN은 대표적인 딥러닝 구조입니다.</p>
<h3 id="실생활-비유">실생활 비유</h3>
<p>AI를 <strong>요리 전체</strong>, ML을 <strong>여러 요리법 가운데 데이터를 보고 맛을 조정하는 방식</strong>, DL을 <strong>매우 많은 조리 단계를 가진 특정 요리법</strong>이라고 생각할 수 있습니다.</p>
<h2 id="5-통계와-머신러닝">5. 통계와 머신러닝</h2>
<p>교재는 통계와 ML을 우열 관계가 아니라 분석 영역이 확장되는 과정으로 설명합니다.</p>
<table>
<thead>
<tr>
<th>관점</th>
<th>전통적 통계 접근</th>
<th>머신러닝 접근</th>
</tr>
</thead>
<tbody><tr>
<td>출발점</td>
<td>전문가 경험과 가설</td>
<td>데이터에 나타난 패턴</td>
</tr>
<tr>
<td>주된 목적</td>
<td>관계 설명, 가설 확인</td>
<td>예측, 패턴 발견</td>
</tr>
<tr>
<td>자주 쓰는 데이터</td>
<td>표본, 정형 데이터</td>
<td>대규모 정형·비정형 데이터</td>
</tr>
<tr>
<td>모델 형태</td>
<td>비교적 단순하고 해석 가능한 모델</td>
<td>복잡한 비선형 모델도 적극 활용</td>
</tr>
<tr>
<td>중요 평가</td>
<td>추정치, 신뢰구간, 유의성, 가정 진단</td>
<td>검증 데이터에서의 일반화 성능</td>
</tr>
</tbody></table>
<h3 id="51-human-driven과-data-driven">5.1 Human-driven과 Data-driven</h3>
<p><strong>Human-driven</strong> 접근에서는 먼저 가설을 세웁니다.</p>
<pre><code class="language-text">가설: 할인율이 높을수록 구매율이 증가할 것이다.
→ 필요한 데이터를 수집
→ 회귀분석 등으로 관계를 검정</code></pre>
<p><strong>Data-driven</strong> 접근에서는 다양한 변수를 학습시켜 사람이 미리 생각하지 못한 관계도 찾습니다.</p>
<pre><code class="language-text">고객 행동 데이터 입력
→ 여러 특성 조합을 학습
→ 구매 가능성이 높은 패턴 발견</code></pre>
<p>실무에서는 둘을 함께 쓰는 것이 좋습니다. 도메인 지식 없이 데이터만 보면 우연한 상관관계를 원인으로 착각할 수 있고, 가설만 고집하면 데이터가 보여주는 새로운 패턴을 놓칠 수 있습니다.</p>
<h3 id="52-방법론의-차이">5.2 방법론의 차이</h3>
<p>공통적인 분석 과정은 다음과 같습니다.</p>
<ol>
<li>비즈니스 문제 이해</li>
<li>데이터 이해와 탐색적 데이터 분석(EDA)</li>
<li>데이터 준비</li>
<li>모델링</li>
<li>평가</li>
<li>시스템 적용</li>
</ol>
<p>통계 모델링은 가정과 변수의 해석을 중요하게 보고, ML은 학습·검증 데이터에서 여러 모델을 비교해 일반화 성능이 좋은 모델을 찾는 데 더 큰 비중을 둡니다.</p>
<h3 id="53-모델링-관점의-차이">5.3 모델링 관점의 차이</h3>
<p>통계 모델은 변수 사이의 관계를 수식으로 설명하고 모델 가정을 진단하는 경우가 많습니다. ML은 관계의 모양을 미리 단순하게 가정하지 않고 학습 데이터에 맞춰 패턴을 찾을 수 있습니다.</p>
<p>그렇다고 ML에서 통계적 진단이 불필요한 것은 아닙니다. 데이터 편향, 표본 설계, 불확실성, 인과관계가 중요한 문제에서는 통계적 사고가 반드시 필요합니다.</p>
<h2 id="6-분류와-회귀">6. 분류와 회귀</h2>
<h3 id="61-분류classification">6.1 분류(Classification)</h3>
<p>분류는 입력 <code>X</code>를 보고 <strong>어느 범주에 속하는지</strong> 예측합니다.</p>
<ul>
<li>스팸 / 정상 메일</li>
<li>정상 거래 / 이상 거래</li>
<li>고장 / 정상</li>
<li>고양이 / 강아지 / 새</li>
</ul>
<p>모델은 클래스 사이의 <strong>결정 경계(Decision Boundary)</strong>를 찾습니다. 출력은 이산적인 클래스 라벨 또는 클래스별 확률입니다.</p>
<p>대표 평가지표에는 Accuracy, Precision, Recall, F1-score, ROC-AUC 등이 있습니다. 클래스 불균형이 심한 경우 정확도만 보면 안 됩니다.</p>
<h3 id="62-회귀regression">6.2 회귀(Regression)</h3>
<p>회귀는 입력 <code>X</code>를 보고 <strong>연속적인 숫자</strong>를 예측합니다.</p>
<ul>
<li>다음 달 매출 3,250만 원</li>
<li>주택 가격 6억 2천만 원</li>
<li>장비의 남은 수명 37시간</li>
</ul>
<p>모델은 실제값과 예측값의 차이가 작아지는 <strong>최적 적합선(Best Fit Line)</strong> 또는 함수를 찾습니다.</p>
<p>대표 평가지표에는 MAE, MSE, RMSE, <code>R²</code>가 있습니다.</p>
<blockquote>
<p><strong>용어 보충:</strong> SSE는 오차 제곱합이므로 작을수록 좋습니다. <code>R²</code>는 모델이 변동을 얼마나 설명하는지 나타내며 일반적으로 클수록 좋습니다. 둘은 관련은 있지만 같은 지표가 아닙니다.</p>
</blockquote>
<h3 id="63-가장-쉬운-구분법">6.3 가장 쉬운 구분법</h3>
<p>질문의 답이 종류라면 분류, 수치라면 회귀입니다.</p>
<pre><code class="language-text">“이 고객이 이탈할까?”      → 분류
“이 고객은 며칠 뒤 이탈할까?” → 회귀</code></pre>
<h2 id="7-어떤-알고리즘이-좋은가">7. 어떤 알고리즘이 좋은가?</h2>
<p>모든 데이터에 항상 가장 좋은 알고리즘은 없습니다. 데이터의 경계가 직선인지, 곡선인지, 원형인지에 따라 잘 맞는 모델 구조가 달라집니다.</p>
<p>이를 <strong>귀납적 편향(Inductive Bias)</strong> 관점으로 이해할 수 있습니다. 각 알고리즘에는 패턴을 바라보는 기본 방식이 있습니다.</p>
<ul>
<li>선형 모델: 관계가 직선에 가깝다고 봄</li>
<li>Decision Tree: 여러 <code>if-then</code> 조건으로 나눔</li>
<li>SVM: 클래스 사이의 여백이 큰 경계를 찾음</li>
<li>Kernel 모델: 입력 공간을 바꿔 비선형 관계를 표현</li>
</ul>
<p>따라서 알고리즘 이름보다 <strong>모델 구조와 데이터 모양의 적합성</strong>이 중요합니다.</p>
<h2 id="8-모델-성능에-영향을-주는-요인">8. 모델 성능에 영향을 주는 요인</h2>
<p>교재의 핵심 메시지는 모델 성능이 알고리즘 선택만으로 결정되지 않는다는 것입니다. 실제 프로젝트에서는 모델을 적용하기 전 과정에 대부분의 노력이 들어갑니다.</p>
<h3 id="81-모델-설계">8.1 모델 설계</h3>
<ul>
<li>무엇을 예측할지 Target을 명확히 정의</li>
<li>어떤 데이터를 모집단으로 볼지 결정</li>
<li>고객군이나 상황별로 모델을 분리할지 결정</li>
<li>분석 방법과 데이터 마트를 설계</li>
</ul>
<p>Target을 잘못 정의하면 정확한 모델도 쓸모가 없습니다. 예를 들어 “이탈 고객”을 30일 미접속으로 정의할지, 계약 해지로 정의할지에 따라 문제가 완전히 달라집니다.</p>
<h3 id="82-데이터-준비">8.2 데이터 준비</h3>
<ul>
<li>데이터 추출, 정제, 형식 변환</li>
<li>결측치와 이상치 처리</li>
<li>중복·오류·단위 불일치 확인</li>
</ul>
<p>현업에서 가장 많은 시간이 드는 구간입니다. 쓰레기가 들어가면 쓰레기가 나오는 <strong>GIGO(Garbage In, Garbage Out)</strong>를 기억해야 합니다.</p>
<h3 id="83-분석-준비와-특성-공학">8.3 분석 준비와 특성 공학</h3>
<ul>
<li>표준화, 구간화, 범주 통합</li>
<li>Random Sampling, Over-sampling</li>
<li>Train/Validation/Test 분할</li>
<li>파생변수 생성</li>
<li>안정성과 예측력을 기준으로 변수 선택</li>
</ul>
<p>예를 들어 구매 횟수 원본만 쓰는 대신 <code>최근 30일 구매 횟수</code>, <code>평균 구매 간격</code>, <code>마지막 구매 후 경과일</code>을 만들면 행동 패턴을 더 잘 표현할 수 있습니다.</p>
<h3 id="84-모델링과-평가">8.4 모델링과 평가</h3>
<ul>
<li>알고리즘 적용</li>
<li>하이퍼파라미터 최적화</li>
<li>성능 비교</li>
<li>해석과 현업 적용 가능성 검토</li>
</ul>
<p>대회에서는 최고 점수가 목적일 수 있지만 비즈니스에서는 다음 질문이 더 중요합니다.</p>
<ul>
<li>이 결과로 실제 의사결정을 개선할 수 있는가?</li>
<li>현업이 결과를 이해하고 수용할 수 있는가?</li>
<li>운영 환경에서도 안정적으로 작동하는가?</li>
<li>잘못된 예측의 비용은 얼마인가?</li>
</ul>
<blockquote>
<p>좋은 비즈니스 모델은 점수만 높은 모델이 아니라 <strong>현장의 문제를 실제로 해결하는 모델</strong>입니다.</p>
</blockquote>
<h2 id="part-1-핵심-정리">Part 1 핵심 정리</h2>
<ul>
<li>AI 역사는 기대, 한계, 침체, 재도약을 반복해 왔습니다.</li>
<li>AI 안에 ML이 있고, ML 안에 DL이 있습니다.</li>
<li>통계는 가설과 설명, ML은 예측과 패턴 발견에 상대적으로 더 집중하지만 둘은 상호 보완적입니다.</li>
<li>분류는 범주, 회귀는 연속값을 예측합니다.</li>
<li>좋은 알고리즘은 데이터 모양과 목적에 맞는 알고리즘입니다.</li>
<li>데이터 정의와 준비가 모델 성능 및 활용성에 큰 영향을 줍니다.</li>
</ul>
<hr />
<h1 id="part-2-machine-learning">Part 2. Machine Learning</h1>
<h2 id="1-머신러닝이란">1. 머신러닝이란?</h2>
<ul>
<li>**컴퓨터가 학습할 수 있도록 하는 알고리즘과 기술을 개발하는 분야</li>
<li>*
전통적인 프로그램은 사람이 규칙을 직접 작성합니다.</li>
</ul>
<pre><code class="language-text">규칙 + 데이터 → 결과</code></pre>
<p>머신러닝은 데이터와 정답 또는 평가 기준을 이용해 규칙을 학습합니다.</p>
<pre><code class="language-text">데이터 + 경험 → 학습된 모델
학습된 모델 + 새 데이터 → 예측</code></pre>
<p>그래서 추천 시스템은 사용자가 일일이 취향 규칙을 입력하지 않아도 시청·검색·클릭 기록에서 패턴을 학습합니다. 자율주행 시스템은 센서 데이터에서 차선, 보행자, 장애물 패턴을 인식합니다.</p>
<p>Arthur Samuel의 정의를 쉬운 말로 바꾸면 다음과 같습니다.</p>
<blockquote>
<p>컴퓨터가 모든 규칙을 명시적으로 지시받지 않아도 경험을 이용해 과제를 더 잘 수행하도록 만드는 분야</p>
</blockquote>
<h2 id="2-t-e-p로-이해하는-학습">2. T, E, P로 이해하는 학습</h2>
<p>Tom Mitchell의 정의는 머신러닝을 세 요소로 설명합니다.</p>
<ul>
<li><strong>T(Task)</strong>: 무엇을 할 것인가?</li>
<li><strong>E(Experience)</strong>: 무엇을 경험하며 배울 것인가?</li>
<li><strong>P(Performance)</strong>: 잘하고 있는지 어떻게 측정할 것인가?</li>
</ul>
<h3 id="예시-스팸-메일-분류">예시: 스팸 메일 분류</h3>
<table>
<thead>
<tr>
<th>요소</th>
<th>내용</th>
</tr>
</thead>
<tbody><tr>
<td>Task</td>
<td>메일을 스팸과 정상으로 분류</td>
</tr>
<tr>
<td>Experience</td>
<td>과거에 스팸 여부가 표시된 메일 데이터</td>
</tr>
<tr>
<td>Performance</td>
<td>정확도, 정밀도, 재현율, F1-score</td>
</tr>
</tbody></table>
<p>데이터를 더 학습하거나 전략을 바꾼 뒤 P가 향상됐다면 모델이 과제를 더 잘 학습했다고 볼 수 있습니다.</p>
<h3 id="예시-상품-추천">예시: 상품 추천</h3>
<table>
<thead>
<tr>
<th>요소</th>
<th>내용</th>
</tr>
</thead>
<tbody><tr>
<td>Task</td>
<td>사용자에게 상품 순위 추천</td>
</tr>
<tr>
<td>Experience</td>
<td>조회, 클릭, 장바구니, 구매 기록</td>
</tr>
<tr>
<td>Performance</td>
<td>클릭률, 구매 전환율, 장기 재방문율</td>
</tr>
</tbody></table>
<p>여기서 클릭률만 높이면 자극적인 추천이 늘 수 있습니다. 따라서 P는 비즈니스 목적과 부작용까지 고려해 정해야 합니다.</p>
<h2 id="3-머신러닝의-접근-방식">3. 머신러닝의 접근 방식</h2>
<p>지도학습에서는 일반적으로 사전에 Target 또는 Label이 필요합니다. 학습은 한 번 실행하고 끝나는 것이 아니라 여러 조건을 반복 실험하는 <strong>Multi-run, Data-driven 과정</strong>입니다.</p>
<pre><code class="language-text">데이터 준비
→ 기본 모델 학습
→ 성능 측정
→ 특성·하이퍼파라미터 조정
→ 다시 학습
→ 성능과 안정성 비교</code></pre>
<p>여기서 사람이 하는 일은 사라지지 않습니다. 문제를 정의하고, 올바른 데이터를 고르고, 평가 기준을 정하고, 결과를 책임 있게 해석하는 일은 여전히 사람의 역할입니다.</p>
<h2 id="4-train-validation-test">4. Train, Validation, Test</h2>
<p>전체 데이터를 보통 세 용도로 나눕니다.</p>
<table>
<thead>
<tr>
<th>데이터</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>Train</td>
<td>모델의 파라미터 학습</td>
<td>교과서와 연습문제</td>
</tr>
<tr>
<td>Validation</td>
<td>하이퍼파라미터 조정, 모델 선택</td>
<td>모의고사</td>
</tr>
<tr>
<td>Test</td>
<td>최종 일반화 성능 확인</td>
<td>실제 시험</td>
</tr>
</tbody></table>
<h3 id="41-기본-흐름">4.1 기본 흐름</h3>
<ol>
<li>Train 세트로 기본 모델을 학습합니다.</li>
<li>Validation 세트로 모델 종류와 하이퍼파라미터를 선택합니다.</li>
<li>모든 선택이 끝난 뒤 Test 세트로 최종 성능을 한 번 확인합니다.</li>
</ol>
<p>분류는 Accuracy, Precision, Recall, F1 등을, 회귀는 <code>R²</code>, RMSE, MAE 등을 비교할 수 있습니다.</p>
<h3 id="42-데이터-누수-주의">4.2 데이터 누수 주의</h3>
<p>Test 결과를 보고 계속 모델을 수정하면 Test도 사실상 Validation이 됩니다. 그러면 최종 성능을 과대평가할 수 있습니다.</p>
<p>또한 표준화 평균, 결측치 대체값, 변수 선택 기준도 Train에서만 학습해야 합니다. 전체 데이터로 전처리한 뒤 나누면 미래 정보가 학습 과정에 섞일 수 있습니다.</p>
<h3 id="43-단일-분할의-한계">4.3 단일 분할의 한계</h3>
<p>교재에서는 Train-Validation-Test의 단일 분할을 설명합니다. 데이터가 적으면 한 번의 분할 결과가 우연에 민감할 수 있으므로 실무에서는 교차검증(Cross-validation)을 함께 사용하기도 합니다.</p>
<h2 id="5-머신러닝의-학습-유형">5. 머신러닝의 학습 유형</h2>
<h3 id="51-지도학습supervised-learning">5.1 지도학습(Supervised Learning)</h3>
<p>입력 <code>X</code>와 정답 <code>Y</code>가 모두 있습니다. 정답과 예측을 비교하며 오차를 줄입니다.</p>
<ul>
<li>분류: 고객 이탈 여부, 질병 여부, 문서 유형</li>
<li>회귀: 매출, 가격, 수요, 남은 수명</li>
</ul>
<pre><code class="language-text">문제와 정답이 있는 문제집으로 공부하는 학생</code></pre>
<h3 id="52-준지도학습semi-supervised-learning">5.2 준지도학습(Semi-supervised Learning)</h3>
<p>소량의 라벨 데이터와 대량의 비라벨 데이터를 함께 사용합니다.</p>
<p>예를 들어 의료 영상 10만 장은 확보했지만 전문의가 진단 라벨을 붙인 영상은 2천 장뿐일 수 있습니다. 라벨 데이터로 기본적인 구분을 배우고, 비라벨 데이터의 전체 분포와 구조를 함께 활용합니다.</p>
<p>준지도학습이 기대하는 대표 가정은 다음과 같습니다.</p>
<ul>
<li><strong>Smoothness 가정</strong>: 가까운 데이터는 같은 라벨일 가능성이 높다.</li>
<li><strong>Low-density 가정</strong>: 좋은 결정 경계는 데이터가 빽빽한 곳보다 드문 곳을 지나갈 가능성이 높다.</li>
</ul>
<p>비유하면 교사가 일부 문제만 답을 알려주고, 학생은 답이 없는 유사 문제의 배치와 공통점을 보며 학습하는 방식입니다.</p>
<p>다만 비라벨 데이터의 분포가 실제 운영 환경과 다르거나 잘못된 의사 라벨을 반복 학습하면 성능이 오히려 나빠질 수 있습니다.</p>
<h3 id="53-비지도학습unsupervised-learning">5.3 비지도학습(Unsupervised Learning)</h3>
<p>정답 <code>Y</code> 없이 입력 <code>X</code>만 제공합니다. 데이터 안의 구조를 발견하는 것이 목적입니다.</p>
<ul>
<li><strong>Clustering</strong>: 비슷한 고객끼리 그룹화</li>
<li><strong>Dimension Reduction</strong>: 수백 개 변수를 핵심 축 몇 개로 압축</li>
<li><strong>Estimation/Pattern discovery</strong>: 데이터 분포나 이상 패턴 탐색</li>
</ul>
<pre><code class="language-text">정답표 없이 물건의 모양과 쓰임새를 보고 스스로 묶는 학생</code></pre>
<p>비지도학습의 결과에는 정해진 정답이 없으므로, 발견된 군집이 실제 업무에서 의미가 있는지 해석해야 합니다.</p>
<h2 id="6-세-학습-유형-비교">6. 세 학습 유형 비교</h2>
<table>
<thead>
<tr>
<th>유형</th>
<th>라벨</th>
<th>대표 출력</th>
<th>대표 사례</th>
</tr>
</thead>
<tbody><tr>
<td>지도학습</td>
<td>전체 또는 충분한 라벨</td>
<td>분류, 회귀</td>
<td>스팸 분류, 가격 예측</td>
</tr>
<tr>
<td>준지도학습</td>
<td>소량 라벨 + 대량 비라벨</td>
<td>분류, 회귀</td>
<td>의료 영상, 음성 인식</td>
</tr>
<tr>
<td>비지도학습</td>
<td>없음</td>
<td>군집, 차원 축소, 구조 발견</td>
<td>고객 세분화, 시각화</td>
</tr>
</tbody></table>
<h2 id="part-2-핵심-정리">Part 2 핵심 정리</h2>
<ul>
<li>머신러닝은 데이터 경험을 통해 과제의 성능을 높이는 과정입니다.</li>
<li>문제를 T, E, P로 정의하면 학습 목표가 명확해집니다.</li>
<li>Train은 학습, Validation은 선택과 조정, Test는 최종 평가에 사용합니다.</li>
<li>지도·준지도·비지도학습은 라벨의 양과 학습 목적이 다릅니다.</li>
<li>반복 실험보다 먼저 데이터 누수 없는 평가 구조를 만들어야 합니다.</li>
</ul>
<hr />
<h1 id="part-3-ml-algorithm">Part 3. ML Algorithm</h1>
<h2 id="1-알고리즘과-모델의-차이">1. 알고리즘과 모델의 차이</h2>
<p>알고리즘은 문제를 해결하기 위한 명확하고 유한한 절차입니다. 일반적으로 입력과 출력이 있고, 각 단계가 모호하지 않으며, 실제 수행 가능해야 합니다.</p>
<p>머신러닝에서 둘을 구분하면 다음과 같습니다.</p>
<ul>
<li><strong>Learning Algorithm</strong>: 데이터로부터 학습하는 절차</li>
<li><strong>Model</strong>: 알고리즘이 특정 데이터를 학습한 뒤 얻은 규칙과 파라미터</li>
</ul>
<pre><code class="language-text">데이터 + 학습 알고리즘 → 학습된 모델</code></pre>
<p>같은 Decision Tree 알고리즘을 사용해도 데이터가 다르면 서로 다른 트리 모델이 만들어집니다. 요리법이 알고리즘이라면 실제로 완성된 한 접시의 음식이 모델입니다.</p>
<h2 id="2-알고리즘-전체-지도">2. 알고리즘 전체 지도</h2>
<table>
<thead>
<tr>
<th>계열</th>
<th>대표 알고리즘</th>
<th>핵심 아이디어</th>
</tr>
</thead>
<tbody><tr>
<td>Tree/CART</td>
<td>Decision Tree</td>
<td>조건을 반복해 데이터를 나눔</td>
</tr>
<tr>
<td>Bagging</td>
<td>Random Forest</td>
<td>무작위성이 있는 여러 트리의 결과를 결합</td>
</tr>
<tr>
<td>Boosting</td>
<td>XGBoost, LightGBM, CatBoost</td>
<td>이전 모델의 실수를 다음 모델이 보완</td>
</tr>
<tr>
<td>Kernel</td>
<td>SVM</td>
<td>클래스 사이의 최대 마진 경계를 찾음</td>
</tr>
<tr>
<td>Regularization</td>
<td>LASSO, Ridge</td>
<td>계수에 패널티를 줘 복잡도를 제한</td>
</tr>
</tbody></table>
<hr />
<h2 id="3-decision-tree">3. Decision Tree</h2>
<h3 id="31-기본-개념">3.1 기본 개념</h3>
<p>Decision Tree는 질문을 차례로 던져 데이터를 작은 집단으로 나눕니다. 분류와 회귀 모두 가능해서 CART(Classification and Regression Trees) 계열이라고 부릅니다.</p>
<pre><code class="language-text">비가 오는가?
├── 예: 외출하지 않음
└── 아니오: 기온이 30도 이상인가?
    ├── 예: 실내 활동
    └── 아니오: 야외 활동</code></pre>
<p>사람이 규칙을 직접 작성하는 전문가 시스템과 달리, Decision Tree는 데이터에서 어떤 변수와 기준값을 사용할지 학습합니다.</p>
<h3 id="32-트리의-구조">3.2 트리의 구조</h3>
<ul>
<li><strong>Root node</strong>: 모든 데이터가 시작하는 첫 질문</li>
<li><strong>Internal node</strong>: 중간 분기 질문</li>
<li><strong>Branch</strong>: 질문의 결과에 따른 경로</li>
<li><strong>Leaf node</strong>: 최종 예측이 나오는 끝 지점</li>
</ul>
<p>연속형 변수는 보통 <code>소득 &lt; 500만 원</code>처럼 임계값으로 나눕니다. 범주형 변수는 <code>{서울, 경기}</code>와 <code>{기타 지역}</code>처럼 범주 집합을 나눌 수 있습니다.</p>
<h3 id="33-고객-세분화-예시">3.3 고객 세분화 예시</h3>
<pre><code class="language-text">성별?
├── 남성 → 월 소득이 500만 원 이상인가?
│   ├── 예: 구매 가능성 높음
│   └── 아니오: 구매 가능성 낮음
└── 여성 → 신용도가 좋은가?
    ├── 예: 연령으로 추가 분기
    └── 아니오: 구매 가능성 매우 낮음</code></pre>
<p>이 트리에서 읽을 수 있는 것은 다음과 같습니다.</p>
<ul>
<li>어떤 <code>if-then</code> 규칙으로 예측했는가?</li>
<li>어떤 변수가 상단에서 큰 분리를 만들었는가?</li>
<li>변수들이 어떤 순서와 조합으로 작용했는가?</li>
</ul>
<p>반대로 트리만 보고 단정할 수 없는 것도 있습니다.</p>
<ul>
<li>각 Leaf의 표본 수가 충분한가?</li>
<li>예측 비율이 다른 데이터에서도 안정적인가?</li>
<li>성별이나 소득이 결과의 <strong>원인</strong>인가?</li>
</ul>
<p>트리는 연관 패턴을 보여주지만 인과관계를 자동으로 증명하지 않습니다.</p>
<h3 id="34-불순도">3.4 불순도</h3>
<p>좋은 분할은 섞여 있는 집단을 같은 클래스끼리 모이게 합니다. 이 섞인 정도를 <strong>불순도(Impurity)</strong>라고 합니다.</p>
<h4 id="gini-index">Gini Index</h4>
<p>$$
Gini = 1 - \sum_{k=1}^{K} p_k^2
$$</p>
<p>한 집단에서 두 개를 무작위로 뽑았을 때 서로 다른 클래스일 가능성과 연결해 이해할 수 있습니다. 값이 0에 가까우면 한 클래스만 모여 있어 순수합니다.</p>
<h4 id="entropy">Entropy</h4>
<p>$$
Entropy = -\sum_{k=1}^{K} p_k \log_2 p_k
$$</p>
<p>Entropy는 결과를 알기 전의 불확실성을 나타냅니다.</p>
<ul>
<li>빨간 공만 있는 바구니: 색을 묻지 않아도 됨 → Entropy 0</li>
<li>빨간 공과 파란 공이 반반: 무엇이 나올지 가장 불확실 → Entropy 최대</li>
</ul>
<p>Decision Tree는 분할 전보다 분할 후 불순도가 가장 많이 줄어드는 질문을 선택합니다. Entropy 감소량은 <strong>Information Gain</strong>이라고 부릅니다.</p>
<h3 id="35-growing과-pruning">3.5 Growing과 Pruning</h3>
<ul>
<li><strong>Growing</strong>: 불순도를 줄이는 분할을 반복하며 나무를 성장</li>
<li><strong>Stopping rule</strong>: 최대 깊이, 최소 샘플 수 등으로 성장을 중단</li>
<li><strong>Pruning</strong>: 성능 개선에 도움이 적거나 과적합 위험이 큰 가지를 제거</li>
</ul>
<p>질문을 계속 추가하면 Train 데이터는 거의 완벽하게 맞출 수 있지만 새 데이터에는 약할 수 있습니다. 그래서 깊이를 제한하거나 가지치기를 합니다.</p>
<h3 id="36-feature-importance">3.6 Feature Importance</h3>
<p>트리 기반 중요도는 보통 다음 두 요소의 누적으로 이해할 수 있습니다.</p>
<pre><code class="language-text">해당 특성이 줄인 불순도 × 그 분할에 도달한 샘플 비중</code></pre>
<p>상단 특성은 많은 샘플에 적용되기 때문에 중요도가 커지기 쉽습니다. 그러나 “트리 위에 있으니 원인이다”라고 해석하면 안 됩니다. 중요도는 모델 안에서 예측에 기여한 정도이지 인과 효과가 아닙니다.</p>
<h3 id="37-장점과-단점">3.7 장점과 단점</h3>
<p><strong>장점</strong></p>
<ul>
<li>규칙과 그림이 직관적이고 설명하기 쉬움</li>
<li>비선형 관계와 변수 상호작용을 표현</li>
<li>표준화가 거의 필요 없음</li>
<li>연속형·범주형 특성을 다룰 수 있음</li>
<li>작은 데이터에서도 빠르게 기준 모델을 만들 수 있음</li>
</ul>
<p><strong>단점</strong></p>
<ul>
<li>깊게 자라면 과적합하기 쉬움</li>
<li>데이터가 조금만 바뀌어도 트리 구조가 크게 달라질 수 있음</li>
<li>단일 트리의 일반화 성능이 낮을 수 있음</li>
<li>큰 데이터에서는 좋은 분할 탐색 비용이 커질 수 있음</li>
<li>클래스 불균형의 영향을 받을 수 있음</li>
</ul>
<h3 id="38-overfitting">3.8 Overfitting</h3>
<p>과적합은 모델이 학습 데이터의 본질적 패턴뿐 아니라 우연한 잡음까지 외운 상태입니다.</p>
<pre><code class="language-text">모의고사 문제와 답을 통째로 외움
→ 모의고사 100점
→ 새로운 유형의 실제 시험 50점</code></pre>
<p>과적합 위험은 다음 상황에서 커집니다.</p>
<ul>
<li>모델이 지나치게 복잡함</li>
<li>데이터가 적음</li>
<li>변수 수가 지나치게 많음</li>
<li>노이즈와 잘못된 라벨이 많음</li>
<li>같은 검증 데이터를 반복해서 보고 튜닝함</li>
</ul>
<p>대응 방법은 트리 깊이 제한, 가지치기, 더 많은 데이터, 변수 선택, 규제, 교차검증, 앙상블 등이 있습니다. 무엇보다 Train과 Validation/Test 성능 차이를 확인해야 합니다.</p>
<hr />
<h2 id="4-random-forest">4. Random Forest</h2>
<p>단일 트리는 이해하기 쉽지만 불안정합니다. Random Forest는 서로 조금씩 다른 트리를 여러 개 만들고 결과를 합칩니다.</p>
<h3 id="41-동작-원리">4.1 동작 원리</h3>
<ol>
<li>원본 데이터에서 복원 추출한 여러 샘플을 만듭니다.</li>
<li>각 분할에서 사용할 특성 일부를 무작위로 고릅니다.</li>
<li>서로 다른 여러 Decision Tree를 학습합니다.</li>
<li>분류는 다수결 투표, 회귀는 평균으로 최종 결과를 냅니다.</li>
</ol>
<p>이 방식은 <strong>Bagging</strong>의 대표 사례입니다.</p>
<h3 id="42-비유">4.2 비유</h3>
<p>한 명의 전문가에게만 묻지 않고, 서로 다른 자료를 본 전문가 100명에게 물어 다수 의견을 채택하는 것과 같습니다. 전문가들이 모두 똑같은 자료와 사고방식을 가지면 실수도 같아지므로 데이터와 특성을 무작위화해 다양성을 만듭니다.</p>
<h3 id="43-특징">4.3 특징</h3>
<ul>
<li>단일 트리보다 과적합과 변동성이 줄어드는 경우가 많음</li>
<li>튜닝을 많이 하지 않아도 안정적인 기준 성능을 내기 쉬움</li>
<li>변수 중요도를 제공할 수 있음</li>
<li>트리가 많아 전체 모델을 한 장으로 설명하기는 어려움</li>
<li>메모리와 예측 시간이 단일 트리보다 증가</li>
</ul>
<hr />
<h2 id="5-boosting">5. Boosting</h2>
<p>Random Forest가 여러 트리를 비교적 독립적으로 만들어 평균내는 방식이라면, Boosting은 <strong>이전 모델의 실수를 다음 모델이 순차적으로 보완</strong>합니다.</p>
<pre><code class="language-text">약한 모델 1 학습
→ 틀린 부분 확인
→ 그 부분을 더 잘 맞추는 모델 2 학습
→ 남은 오차를 줄이는 모델 3 학습
→ 여러 모델을 합쳐 강한 모델 생성</code></pre>
<p>회귀에서는 앞 모델이 남긴 잔차를 다음 모델이 줄이는 방향으로 생각할 수 있습니다. 성능이 좋지만 순차적 학습과 복잡한 구조 때문에 해석과 튜닝이 어려울 수 있으며 과적합도 관리해야 합니다.</p>
<h3 id="51-xgboost">5.1 XGBoost</h3>
<ul>
<li>Gradient Boosting을 효율적이고 강력하게 구현</li>
<li>규제, 결측치 처리, 병렬 계산 등 실용적 기능 제공</li>
<li>정형 데이터에서 높은 예측 성능을 내는 경우가 많음</li>
<li>깊이 중심(Depth-wise) 성장 방식이 대표적</li>
<li>하이퍼파라미터가 많아 튜닝과 해석이 필요</li>
</ul>
<h3 id="52-lightgbm">5.2 LightGBM</h3>
<ul>
<li>대용량 정형 데이터에서 빠른 학습과 메모리 효율을 지향</li>
<li>손실을 가장 크게 줄일 Leaf를 우선 분할하는 Leaf-wise 방식</li>
<li>같은 분할 수에서 성능이 빠르게 좋아질 수 있음</li>
<li>데이터가 작을 때 Leaf가 지나치게 깊어지면 과적합할 수 있으므로 제한 필요</li>
</ul>
<h3 id="53-catboost">5.3 CatBoost</h3>
<ul>
<li>범주형 특성을 효율적으로 다루도록 설계</li>
<li>복잡한 수동 인코딩 부담을 줄일 수 있음</li>
<li>Ordered Boosting을 사용해 현재 샘플의 정답 정보를 부적절하게 미리 보는 누수를 줄임</li>
<li>기본 설정으로도 안정적인 성능을 보이는 경우가 많음</li>
</ul>
<h3 id="54-depth-wise와-leaf-wise">5.4 Depth-wise와 Leaf-wise</h3>
<ul>
<li><strong>Depth-wise</strong>: 같은 깊이의 노드를 비교적 균형 있게 확장</li>
<li><strong>Leaf-wise</strong>: 현재 가장 큰 손실 감소가 가능한 Leaf를 우선 확장</li>
</ul>
<p>Leaf-wise는 효율적으로 손실을 줄일 수 있지만 트리가 한쪽으로 깊어질 수 있습니다.</p>
<hr />
<h2 id="6-support-vector-machine">6. Support Vector Machine</h2>
<h3 id="61-maximum-margin">6.1 Maximum Margin</h3>
<p>SVM은 두 클래스를 나누는 선 가운데 두 집단과의 <strong>여백(Margin)</strong>이 가장 큰 경계를 찾습니다.</p>
<p>도로 중앙선을 긋는다고 생각해 봅시다. 양쪽 차량에 너무 가까운 선보다 양쪽과 최대한 멀리 떨어진 선이 작은 흔들림에도 안전합니다. SVM도 이런 안정적인 경계를 찾습니다.</p>
<p>경계에 가장 가까워 위치와 방향을 결정하는 데이터 포인트를 <strong>Support Vector</strong>라고 합니다.</p>
<h3 id="62-kernel-trick">6.2 Kernel Trick</h3>
<p>원래 공간에서 직선으로 나눌 수 없는 데이터는 더 높은 차원의 Feature Space로 옮기면 평면으로 나눌 수 있습니다.</p>
<p>예를 들어 종이 위에 원형으로 섞인 점들은 직선으로 나누기 어렵습니다. 가운데를 위로 들어 올려 3차원으로 바꾸면 평면 하나로 나눌 수 있습니다. Kernel은 모든 고차원 좌표를 직접 계산하지 않고도 그 공간에서의 유사도를 계산하도록 돕습니다.</p>
<p>대표 Kernel에는 Linear, Polynomial, RBF가 있습니다.</p>
<h3 id="63-soft-margin과-c">6.3 Soft Margin과 C</h3>
<p>현실 데이터에는 완벽히 나뉘지 않는 점이 있습니다. Soft Margin SVM은 일부 오분류를 허용하면서 넓은 Margin을 찾습니다.</p>
<ul>
<li><code>C</code>가 큼: 오분류에 큰 벌점 → 학습 데이터를 더 맞추고 Margin이 좁아질 수 있음 → 과적합 위험</li>
<li><code>C</code>가 작음: 일부 오분류 허용 → Margin이 넓어짐 → 지나치게 작으면 과소적합 위험</li>
</ul>
<h3 id="64-gamma">6.4 Gamma</h3>
<p>RBF Kernel에서 <code>gamma</code>는 한 데이터 포인트의 영향 범위를 조절합니다.</p>
<ul>
<li><code>gamma</code>가 큼: 가까운 점에만 민감 → 경계가 복잡 → 과적합 위험</li>
<li><code>gamma</code>가 작음: 넓은 범위에 완만하게 영향 → 경계가 단순 → 과소적합 위험</li>
</ul>
<p>보통 <code>C</code>와 <code>gamma</code>를 함께 조정합니다.</p>
<h3 id="65-svm이-잘-맞는-상황">6.5 SVM이 잘 맞는 상황</h3>
<ul>
<li>표본 수에 비해 변수 수가 많음</li>
<li>텍스트, 유전자 발현처럼 고차원 데이터</li>
<li>클래스 사이에 비교적 명확한 Margin이 있음</li>
<li>Kernel로 비선형 경계를 표현할 필요가 있음</li>
</ul>
<p>대규모 데이터에서는 학습 비용이 커질 수 있고, 확률 해석과 설명이 트리보다 어려울 수 있습니다.</p>
<hr />
<h2 id="7-regularization">7. Regularization</h2>
<p>규제는 모델이 지나치게 복잡해지지 않도록 파라미터에 패널티를 주는 방법입니다.</p>
<pre><code class="language-text">기존 손실 함수 + 복잡도 패널티</code></pre>
<p>Train 오차만 줄이면 계수가 지나치게 커져 잡음에 민감해질 수 있습니다. 규제는 약간의 Train 성능을 양보하고 새 데이터에서의 안정성을 얻는 전략입니다.</p>
<h3 id="71-발표-슬라이드-비유">7.1 발표 슬라이드 비유</h3>
<p>발표 자료가 30장인데 제출 한도가 5장이라고 가정해 봅시다. <code>5장 제한</code>이 규제입니다.</p>
<ul>
<li>모든 내용을 조금씩 압축해 남김 → Ridge와 비슷</li>
<li>중요하지 않은 내용을 완전히 삭제 → LASSO와 비슷</li>
</ul>
<h3 id="72-ridge-regression-l2-규제">7.2 Ridge Regression: L2 규제</h3>
<p>$$
Loss_{Ridge} = Loss + \lambda \sum_j w_j^2
$$</p>
<ul>
<li>큰 계수에 더 큰 패널티를 줌</li>
<li>모든 계수를 0에 가깝게 축소하지만 보통 정확히 0으로 만들지는 않음</li>
<li>서로 강하게 상관된 변수가 있을 때 계수를 안정화하는 데 유용</li>
<li>모든 변수를 어느 정도 유지하고 싶을 때 적합</li>
</ul>
<h3 id="73-lasso-regression-l1-규제">7.3 LASSO Regression: L1 규제</h3>
<p>$$
Loss_{LASSO} = Loss + \lambda \sum_j |w_j|
$$</p>
<ul>
<li>일부 계수를 정확히 0으로 만들 수 있음</li>
<li>변수 선택 효과가 생김</li>
<li>간결하고 해석 가능한 모델이 필요할 때 유용</li>
<li>강하게 상관된 변수들 가운데 일부만 선택할 수 있어 결과 안정성을 확인해야 함</li>
</ul>
<h3 id="74-규제-강도">7.4 규제 강도</h3>
<p><code>λ</code> 또는 라이브러리에 따라 <code>alpha</code>가 규제 강도를 조절합니다.</p>
<ul>
<li>너무 작음: 규제가 약해 과적합 가능</li>
<li>너무 큼: 계수를 지나치게 줄여 과소적합 가능</li>
</ul>
<p>적절한 값은 Validation 또는 교차검증으로 선택합니다. 입력 변수의 크기가 크게 다르면 계수 패널티 비교가 불공정해지므로 LASSO와 Ridge 전에는 보통 표준화가 필요합니다.</p>
<h3 id="75-다중공선성">7.5 다중공선성</h3>
<p>서로 비슷한 변수가 많으면 회귀계수가 불안정해질 수 있습니다. 예를 들어 <code>키(cm)</code>와 <code>키(m)</code>를 동시에 넣으면 정보가 거의 같습니다. Ridge는 이런 상관된 변수들의 계수를 함께 축소해 안정성을 높이는 데 도움이 됩니다.</p>
<h3 id="76-dropout과-batch-normalization">7.6 Dropout과 Batch Normalization</h3>
<p>교재는 규제 관련 대표 방법으로 Dropout과 Batch Normalization도 언급합니다.</p>
<ul>
<li><strong>Dropout</strong>: 신경망 학습 중 일부 뉴런을 무작위로 끄며 특정 경로에 과도하게 의존하지 않도록 함</li>
<li><strong>Batch Normalization</strong>: 중간 활성값의 분포를 안정화해 학습을 돕는 기법. 경우에 따라 규제와 비슷한 효과가 나타날 수 있지만 주목적은 학습 안정화입니다.</li>
</ul>
<p>LASSO와 Ridge는 선형 모델의 계수에 직접 패널티를 주는 방식이고, Dropout과 Batch Normalization은 주로 신경망 학습에서 사용하는 방식이라는 차이가 있습니다.</p>
<hr />
<h2 id="8-알고리즘은-어떻게-확장됐나">8. 알고리즘은 어떻게 확장됐나?</h2>
<p>트리 계열의 발전을 한 줄로 보면 다음과 같습니다.</p>
<pre><code class="language-text">Decision Tree
→ 여러 트리를 독립적으로 결합: Random Forest
→ 이전 실수를 순차적으로 보완: Gradient Boosting
→ 속도·정확도·규제 개선: XGBoost
→ 대용량 효율 개선: LightGBM
→ 범주형 처리와 누수 완화: CatBoost</code></pre>
<p>이 순서가 무조건적인 성능 순위는 아닙니다. 데이터 크기, 변수 유형, 설명 가능성, 학습 시간, 운영 비용에 따라 선택이 달라집니다.</p>
<h2 id="9-상황별-1순위-후보">9. 상황별 1순위 후보</h2>
<table>
<thead>
<tr>
<th>상황</th>
<th>먼저 고려할 후보</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>규제 담당자에게 거절 이유를 한 장의 그림으로 설명</td>
<td>Decision Tree</td>
<td>규칙과 경로가 직관적</td>
</tr>
<tr>
<td>튜닝을 많이 하지 않고 안정적인 기준 성능 필요</td>
<td>Random Forest</td>
<td>단일 트리의 변동성과 과적합을 완화</td>
</tr>
<tr>
<td>정형 데이터 대회에서 최고 성능 추구</td>
<td>XGBoost</td>
<td>강력한 Boosting 성능과 세밀한 튜닝 가능</td>
</tr>
<tr>
<td>수천만 행, 속도와 메모리가 중요</td>
<td>LightGBM</td>
<td>대규모 데이터 효율을 지향</td>
</tr>
<tr>
<td>고유값이 많은 범주형 변수가 다수</td>
<td>CatBoost</td>
<td>범주형 특성을 직접 다루는 기능</td>
</tr>
<tr>
<td>표본은 수백 개, 변수는 수천 개, 비선형 경계 필요</td>
<td>Kernel SVM</td>
<td>고차원·소표본 및 비선형 경계에 적합할 수 있음</td>
</tr>
<tr>
<td>변수가 많지만 대체로 선형, 중요한 변수만 남기고 싶음</td>
<td>LASSO</td>
<td>계수를 0으로 만들어 변수 선택 효과</td>
</tr>
</tbody></table>
<p>이 표는 자동 정답표가 아니라 <strong>첫 번째 실험 후보</strong>입니다. 실제 선택은 검증 성능, 데이터 품질, 계산 자원, 설명 가능성, 운영 제약을 함께 보고 결정해야 합니다.</p>
<h2 id="10-교재-퀴즈-풀이">10. 교재 퀴즈 풀이</h2>
<h3 id="q1-정확도보다-한-장의-설명이-중요한-대출-심사">Q1. 정확도보다 한 장의 설명이 중요한 대출 심사</h3>
<p><strong>정답: A. Decision Tree</strong></p>
<p>거절로 이어진 조건 경로를 그림과 <code>if-then</code> 규칙으로 설명하기 쉽습니다. 단, 실제 금융 의사결정에서는 공정성, 안정성, 규제 요건을 별도로 검증해야 합니다.</p>
<h3 id="q2-빠르고-안정적인-기준-성능-과적합-완화">Q2. 빠르고 안정적인 기준 성능, 과적합 완화</h3>
<p><strong>정답: B. Random Forest</strong></p>
<p>여러 트리의 결과를 평균내 단일 트리의 변동성을 줄입니다. 비교적 적은 튜닝으로 강한 기준 모델을 얻기 좋습니다.</p>
<h3 id="q3-정형-데이터-대회에서-최대-정확도">Q3. 정형 데이터 대회에서 최대 정확도</h3>
<p><strong>정답: D. XGBoost</strong></p>
<p>정형 데이터에서 강력하며 튜닝 여지가 많습니다. 다만 항상 우승을 보장하는 것은 아니므로 교차검증과 다른 Boosting 모델 비교가 필요합니다.</p>
<h3 id="q4-수천만-행-속도·메모리·성능-모두-중요">Q4. 수천만 행, 속도·메모리·성능 모두 중요</h3>
<p><strong>정답: C. LightGBM</strong></p>
<p>대용량 데이터에서 효율적인 학습을 목표로 설계됐습니다. Leaf 수와 깊이를 관리해 과적합을 방지해야 합니다.</p>
<h3 id="q5-범주형-변수가-많고-인코딩-부담을-줄이고-싶음">Q5. 범주형 변수가 많고 인코딩 부담을 줄이고 싶음</h3>
<p><strong>정답: C. CatBoost</strong></p>
<p>범주형 특성을 효율적으로 처리하고 Target 정보 누수를 줄이는 학습 전략을 제공합니다.</p>
<h3 id="q6-소표본·고차원·비선형-경계">Q6. 소표본·고차원·비선형 경계</h3>
<p><strong>정답: D. SVM</strong></p>
<p>Kernel SVM은 고차원 공간에서 비선형 경계를 만들 수 있습니다. <code>C</code>와 <code>gamma</code>, 표준화를 함께 조정해야 합니다.</p>
<h3 id="q7-선형-관계이며-중요한-변수만-남겨야-함">Q7. 선형 관계이며 중요한 변수만 남겨야 함</h3>
<p><strong>정답: A. LASSO</strong></p>
<p>L1 규제가 일부 계수를 0으로 만들어 변수 선택 효과와 간결한 해석을 제공합니다.</p>
<h2 id="part-3-핵심-정리">Part 3 핵심 정리</h2>
<ul>
<li>알고리즘은 학습 절차, 모델은 특정 데이터를 학습한 결과입니다.</li>
<li>Decision Tree는 설명력이 좋지만 과적합과 불안정성이 있습니다.</li>
<li>Random Forest는 무작위화된 여러 트리를 결합해 안정성을 높입니다.</li>
<li>Boosting은 이전 모델의 실수를 다음 모델이 보완합니다.</li>
<li>SVM은 최대 Margin과 Kernel을 이용해 선형·비선형 경계를 찾습니다.</li>
<li>LASSO는 변수 선택, Ridge는 계수 안정화와 모든 변수 유지에 강점이 있습니다.</li>
<li>최고 알고리즘 하나를 찾기보다 목적과 제약에 맞는 후보를 검증해야 합니다.</li>
</ul>
<hr />
<h1 id="최종-실무-체크리스트">최종 실무 체크리스트</h1>
<p>머신러닝 프로젝트를 시작할 때 다음 순서를 확인하면 좋습니다.</p>
<h2 id="1-문제-정의">1. 문제 정의</h2>
<ul>
<li>분류인가, 회귀인가, 구조 발견인가?</li>
<li>Target과 예측 시점은 무엇인가?</li>
<li>잘못된 예측의 비용은 무엇인가?</li>
</ul>
<h2 id="2-t-e-p-정의">2. T, E, P 정의</h2>
<ul>
<li>Task는 구체적인가?</li>
<li>Experience가 실제 운영 상황을 대표하는가?</li>
<li>Performance 지표가 비즈니스 목표와 일치하는가?</li>
</ul>
<h2 id="3-데이터-설계">3. 데이터 설계</h2>
<ul>
<li>학습 시점에 실제로 사용할 수 있는 변수인가?</li>
<li>개인정보와 민감정보를 최소화했는가?</li>
<li>결측치, 이상치, 중복, 라벨 오류를 확인했는가?</li>
<li>시간 순서와 집단 중복을 고려해 데이터를 나눴는가?</li>
</ul>
<h2 id="4-평가-설계">4. 평가 설계</h2>
<ul>
<li>Train, Validation, Test의 역할이 분리됐는가?</li>
<li>전처리 과정에 데이터 누수가 없는가?</li>
<li>클래스 불균형에 맞는 지표를 사용했는가?</li>
<li>여러 분할이나 교차검증에서도 성능이 안정적인가?</li>
</ul>
<h2 id="5-모델-선택">5. 모델 선택</h2>
<ul>
<li>설명 가능성이 최우선이면 단순 모델이나 Decision Tree</li>
<li>안정적인 기준 성능이면 Random Forest</li>
<li>정형 데이터 최고 성능 후보로 Boosting 계열</li>
<li>고차원·소표본·비선형 문제라면 Kernel SVM 검토</li>
<li>선형 관계에서 변수 선택은 LASSO, 계수 안정화는 Ridge</li>
</ul>
<h2 id="6-운영-가능성">6. 운영 가능성</h2>
<ul>
<li>현업이 결과를 이해하고 행동으로 옮길 수 있는가?</li>
<li>예측 시간, 메모리, 비용이 허용 범위인가?</li>
<li>데이터 분포 변화와 성능 저하를 감시할 수 있는가?</li>
<li>개인정보, 공정성, 보안, 규제 요구사항을 충족하는가?</li>
</ul>
<hr />
<h1 id="마무리">마무리</h1>
<p>이 교재의 가장 중요한 메시지는 알고리즘 이름을 많이 외우는 것이 아닙니다.</p>
<blockquote>
<p><strong>좋은 머신러닝은 문제를 정확히 정의하고, 올바른 데이터를 준비하고, 누수 없이 평가한 뒤, 목적과 제약에 맞는 모델을 선택하는 과정이다.</strong></p>
</blockquote>
<p>처음에는 다음 네 문장만 확실히 기억해도 충분합니다.</p>
<ol>
<li>분류는 범주, 회귀는 숫자를 예측한다.</li>
<li>Train으로 배우고, Validation으로 선택하고, Test로 최종 확인한다.</li>
<li>복잡한 모델은 강력하지만 과적합과 설명 문제를 함께 가진다.</li>
<li>실무에서 모델의 가치는 최고 점수가 아니라 실제 문제 해결로 결정된다.</li>
</ol>