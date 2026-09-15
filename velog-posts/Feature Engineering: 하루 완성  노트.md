<h1 id="orientation--intro">Orientation &amp; Intro</h1>
<h2 id="1-분석은-모델-학습부터-시작하지-않는다">1. 분석은 모델 학습부터 시작하지 않는다</h2>
<p>데이터 분석 프로젝트의 전체 흐름은 다음과 같이 볼 수 있습니다.</p>
<pre><code class="language-text">비즈니스 이해 → 가설 수립 → 데이터 준비 → 모델 정의 → 모델 평가 → 시스템 적용</code></pre>
<p>좋은 분석은 “어떤 알고리즘을 쓸까?”보다 “무엇을 해결해야 할까?”에서 시작합니다. 예를 들어 고객 이탈을 예측한다고 해봅시다.</p>
<ul>
<li>비즈니스 문제: 다음 달에 서비스를 떠날 가능성이 큰 고객을 미리 찾고 싶다.</li>
<li>가설: 최근 접속이 줄고 문의가 늘어난 고객은 이탈할 가능성이 높을 것이다.</li>
<li>필요한 데이터: 접속일, 이용 횟수, 문의 횟수, 가입 기간, 요금제 등</li>
<li>모델의 역할: 각 고객의 이탈 가능성을 점수로 계산한다.</li>
<li>실제 적용: 점수가 높은 고객에게 적절한 유지 프로그램을 제안한다.</li>
</ul>
<p>모델의 정확도가 높아도 현업에서 사용할 수 없다면 프로젝트의 목적을 달성한 것이 아닙니다. 따라서 분석 방법론은 단순한 코딩 순서가 아니라 <strong>활동, 절차, 기법, 산출물을 연결하는 업무 가이드</strong>입니다.</p>
<h2 id="2-단계별로-반드시-물어야-할-질문">2. 단계별로 반드시 물어야 할 질문</h2>
<h3 id="21-비즈니스-이해">2.1 비즈니스 이해</h3>
<ul>
<li>해결하려는 문제는 무엇인가?</li>
<li>성공을 어떤 지표로 판단할 것인가?</li>
<li>경쟁 환경이나 업무 제약은 무엇인가?</li>
<li>분석 결과가 기업 가치, 비용 절감, 공정 효율에 어떻게 연결되는가?</li>
</ul>
<p>예를 들어 제조 불량 탐지에서 “정확도 95%”만 목표로 삼으면 부족합니다. 불량품을 놓쳤을 때의 비용과 정상 제품을 불량으로 잘못 판정했을 때의 비용이 다르기 때문입니다.</p>
<h3 id="22-데이터와-가설-이해">2.2 데이터와 가설 이해</h3>
<ul>
<li>어떤 분석에 어떤 데이터가 필요한가?</li>
<li>현재 수집 중인 항목, 양, 품질은 충분한가?</li>
<li>외부에서 추가로 확보해야 할 데이터가 있는가?</li>
<li>데이터의 의미를 설명하는 메타데이터, 분류 체계, 기준 정보가 있는가?</li>
<li>업무 전문가와 같은 뜻으로 변수를 이해하고 있는가?</li>
</ul>
<p><code>status = 0</code>이 “정상”인지 “탈퇴”인지 모른다면 숫자는 있어도 정보는 없는 것과 같습니다.</p>
<h3 id="23-분석-환경-이해">2.3 분석 환경 이해</h3>
<ul>
<li>데이터 크기에 맞는 CPU, GPU, 메모리가 있는가?</li>
<li>저장 공간은 충분한가?</li>
<li>개발 환경과 운영 환경의 패키지 버전은 호환되는가?</li>
</ul>
<p>작은 노트북 데이터로 만든 코드가 수억 건의 운영 데이터에서도 그대로 동작한다고 보장할 수 없습니다.</p>
<h3 id="24-데이터-준비와-feature-engineering">2.4 데이터 준비와 Feature Engineering</h3>
<ul>
<li>결측치는 왜 생겼으며 삭제할지, 대체할지 결정했는가?</li>
<li>이상치는 오류인가, 중요한 사건인가?</li>
<li>업무 규칙으로 계산한 항목의 로직과 갱신 주기는 무엇인가?</li>
<li>분석 목적에 필요한 핵심 변수는 무엇인가?</li>
<li>변수의 분포를 바꾸거나 새 변수를 만들 필요가 있는가?</li>
<li>전통적 머신러닝과 딥러닝 중 어떤 접근이 더 적합한가?</li>
</ul>
<h3 id="25-모델-설계와-평가">2.5 모델 설계와 평가</h3>
<ul>
<li>선형 모델, 트리·부스팅 모델, 앙상블 모델 중 무엇을 왜 선택했는가?</li>
<li>학습·검증·테스트 데이터를 어떻게 나눌 것인가?</li>
<li>정확도가 높지만 재현율이 낮아도 사용할 수 있는가?</li>
<li>회귀 모델의 오차를 어떻게 줄일 것인가?</li>
</ul>
<p>정확도는 언제나 충분한 지표가 아닙니다. 질병 환자가 1%인 데이터에서 모두 정상이라고 답해도 정확도는 99%지만 환자를 한 명도 찾지 못합니다.</p>
<h3 id="26-시스템-적용과-유지보수">2.6 시스템 적용과 유지보수</h3>
<ul>
<li>예측 결과를 업무 시스템과 어떻게 결합할 것인가?</li>
<li>지나치게 많은 이상 알림을 후처리 규칙으로 줄여야 하는가?</li>
<li>모델을 어떤 주기와 흐름으로 실행할 것인가?</li>
<li>운영 데이터의 변화와 모델 성능 저하를 어떻게 감시할 것인가?</li>
<li>재학습 주기와 배포 절차는 무엇인가?</li>
</ul>
<p>이 단계가 AIOps 또는 MLOps와 만나는 지점입니다. 모델은 한 번 만들어 끝나는 파일이 아니라 계속 관찰하고 갱신해야 하는 운영 대상입니다.</p>
<h2 id="3-feature-engineering이란">3. Feature Engineering이란?</h2>
<p>Feature는 모델이 입력으로 받는 <strong>문제의 단서</strong>입니다. Feature Engineering은 원본 데이터를 모델이 학습하기 좋은, 의미 있는 표현으로 바꾸는 과정입니다.</p>
<pre><code class="language-text">나쁜 데이터 → 나쁜 예측
깨끗하기만 한 데이터 → 제한적인 예측
의미 있는 Feature → 더 좋은 예측 가능성</code></pre>
<p>예를 들어 쇼핑몰 데이터에 <code>마지막 구매일</code>만 있다면 모델이 날짜 자체의 의미를 알아내기 어렵습니다. 이를 <code>오늘 - 마지막 구매일 = 최근 구매 후 경과일</code>로 바꾸면 고객의 최근 활동성을 직접 표현할 수 있습니다.</p>
<blockquote>
<p>핵심: 전처리가 데이터를 사용할 수 있게 만드는 일이라면, Feature Engineering은 문제를 더 잘 설명하게 만드는 일입니다. 실제로 두 영역은 서로 겹치며 하나의 연속된 과정으로 보는 편이 좋습니다.</p>
</blockquote>
<hr />
<h1 id="eda">EDA</h1>
<h2 id="1-eda란">1. EDA란?</h2>
<p>EDA(Exploratory Data Analysis, 탐색적 데이터 분석)는 데이터를 탐색해 구조와 특성을 이해하고, 문제와 가설을 발견하여 다음 분석 방향을 결정하는 과정입니다.</p>
<p>단순히 그래프를 많이 그리는 활동이 아닙니다.</p>
<pre><code class="language-text">데이터 구조 이해
    ↓
데이터 품질 확인
    ↓
변수의 특성 이해
    ↓
변수 사이의 관계 탐색
    ↓
Feature Engineering 방향 결정</code></pre>
<h2 id="2-eda의-전체-workflow">2. EDA의 전체 Workflow</h2>
<pre><code class="language-text">Raw Data
  → 구조 확인
  → 기술 통계
  → 시각화
  → 질문과 가설
  → 발견한 인사이트와 문제 정리
  → 전처리 및 Feature Engineering
  → 모델링과 평가</code></pre>
<ul>
<li>구조 확인: 행과 열의 크기, 변수명, 타입, 결측치 확인</li>
<li>기술 통계: 중심, 퍼짐, 범위, 분포 형태 파악</li>
<li>시각화: 분포, 패턴, 변수 간 관계 확인</li>
<li>질문하기: 관찰한 현상의 원인과 모델에 미칠 영향을 생각</li>
<li>다음 행동: 삭제, 대체, 변환, 스케일링, 인코딩, 파생변수 생성 등</li>
</ul>
<p>EDA가 <strong>진단</strong>이라면 Feature Engineering은 그 진단에 따른 <strong>처방</strong>입니다.</p>
<h2 id="3-기술-통계-숫자를-짧게-요약하는-법">3. 기술 통계: 숫자를 짧게 요약하는 법</h2>
<h3 id="31-중심-위치">3.1 중심 위치</h3>
<ul>
<li>평균(mean): 모든 값을 더해 개수로 나눈 값</li>
<li>중앙값(median): 정렬했을 때 가운데 값</li>
</ul>
<p>월급이 <code>[300, 320, 340, 360, 3000]</code>만 원이라면 평균은 864만 원이지만 중앙값은 340만 원입니다. 극단적으로 큰 값이 있을 때 중앙값이 일반적인 사람을 더 잘 대표할 수 있습니다.</p>
<h3 id="32-산포">3.2 산포</h3>
<ul>
<li>범위(range): 최댓값 - 최솟값</li>
<li>분산(variance): 값들이 평균에서 얼마나 떨어져 있는지 제곱해 요약</li>
<li>표준편차(standard deviation): 분산에 제곱근을 적용해 원래 단위로 표현</li>
<li>사분위수: 데이터를 25%, 50%, 75% 위치로 나눈 값</li>
<li>IQR: <code>Q3 - Q1</code>, 가운데 50% 데이터가 차지하는 범위</li>
</ul>
<h3 id="33-분포의-형태">3.3 분포의 형태</h3>
<ul>
<li>왜도(skewness): 분포가 한쪽으로 얼마나 치우쳤는가?</li>
<li>첨도(kurtosis): 꼬리와 극단값의 성격이 어떠한가?</li>
</ul>
<p>왜도가 큰 매출 데이터는 대부분 작은 값이고 드물게 매우 큰 값이 나타납니다. 이런 경우 평균, 표준편차만 보고 전형적인 고객을 판단하면 오해할 수 있습니다.</p>
<h2 id="4-기본-eda-도구">4. 기본 EDA 도구</h2>
<table>
<thead>
<tr>
<th>목적</th>
<th>pandas 예시</th>
<th>확인할 내용</th>
</tr>
</thead>
<tbody><tr>
<td>크기</td>
<td><code>df.shape</code></td>
<td>행과 열의 수</td>
</tr>
<tr>
<td>열 이름</td>
<td><code>df.columns</code></td>
<td>오탈자, 예상하지 못한 변수</td>
</tr>
<tr>
<td>타입·결측</td>
<td><code>df.info()</code></td>
<td>숫자가 문자열로 읽히지 않았는지</td>
</tr>
<tr>
<td>앞·뒤 일부</td>
<td><code>df.head()</code>, <code>df.tail()</code></td>
<td>값의 형식과 단위</td>
</tr>
<tr>
<td>무작위 표본</td>
<td><code>df.sample()</code></td>
<td>정렬된 앞부분에 치우치지 않은 확인</td>
</tr>
<tr>
<td>요약 통계</td>
<td><code>df.describe(include=&quot;all&quot;)</code></td>
<td>수치형과 범주형의 전반적 특성</td>
</tr>
<tr>
<td>범주 종류</td>
<td><code>unique()</code>, <code>nunique()</code></td>
<td>값 자체와 고유값 개수</td>
</tr>
<tr>
<td>범주 빈도</td>
<td><code>value_counts()</code></td>
<td>도수와 비율</td>
</tr>
<tr>
<td>결측치</td>
<td><code>isnull().sum()</code>, <code>isnull().mean()</code></td>
<td>개수와 비율</td>
</tr>
<tr>
<td>상관관계</td>
<td><code>corr(numeric_only=True)</code></td>
<td>수치형 변수의 선형 관계</td>
</tr>
</tbody></table>
<p><code>head()</code>만 보면 데이터가 날짜순으로 정렬되어 특정 기간만 관찰할 수 있습니다. <code>sample(random_state=42)</code>를 함께 보면 더 균형 있게 확인할 수 있습니다.</p>
<h2 id="5-범주형-변수와-cardinality">5. 범주형 변수와 Cardinality</h2>
<p>Cardinality는 범주형 변수에 있는 서로 다른 값의 개수입니다.</p>
<ul>
<li>성별 코드 2개: 낮은 Cardinality</li>
<li>지역 5개: 비교적 낮은 Cardinality</li>
<li>고객 ID 100,000개: 높은 Cardinality</li>
</ul>
<p>고유값이 많다고 무조건 쓸모없는 것은 아닙니다. 다만 고객 ID를 One-Hot Encoding하면 열이 지나치게 늘어날 수 있고, 새로운 고객에게 일반화하기 어렵습니다. 반대로 지역이나 상품군처럼 반복되는 범주는 유용한 정보가 될 수 있습니다.</p>
<p><code>nunique()</code>만으로는 빈도 불균형, 결측치, 오탈자, Target과의 관계를 알 수 없습니다. 반드시 <code>value_counts()</code>와 업무 의미를 함께 봐야 합니다.</p>
<h2 id="6-결측-패턴-탐색">6. 결측 패턴 탐색</h2>
<p>결측치의 총개수뿐 아니라 <strong>어디에 함께 나타나는지</strong>가 중요합니다.</p>
<ul>
<li>특정 기간에만 결측: 수집 시스템 장애 가능성</li>
<li>두 센서가 함께 결측: 같은 장비나 통신망 문제 가능성</li>
<li>소득이 높은 집단에서만 응답 누락: 비무작위 결측 가능성</li>
</ul>
<p><code>missingno</code>의 matrix는 행별 결측 위치를, heatmap은 변수들의 결측 동시 발생 관계를 시각적으로 확인할 때 유용합니다.</p>
<h2 id="7-상관관계-탐색">7. 상관관계 탐색</h2>
<p>상관계수는 두 수치형 변수가 함께 움직이는 <strong>선형 관계의 정도</strong>입니다. 보통 -1에서 1 사이입니다.</p>
<ul>
<li>1에 가까움: 함께 증가하는 경향</li>
<li>-1에 가까움: 하나가 증가할 때 다른 하나는 감소하는 경향</li>
<li>0에 가까움: 뚜렷한 선형 관계가 약함</li>
</ul>
<p>주의할 점은 세 가지입니다.</p>
<ol>
<li>상관관계는 인과관계가 아닙니다.</li>
<li>U자형 같은 비선형 관계는 상관계수가 0에 가까울 수 있습니다.</li>
<li>극단값 하나가 상관계수를 크게 바꿀 수 있습니다.</li>
</ol>
<p>변수가 많으면 correlation matrix를 heatmap으로 표현해 강한 관계를 빠르게 찾을 수 있습니다.</p>
<h2 id="8-변수-조합에-맞는-시각화">8. 변수 조합에 맞는 시각화</h2>
<h3 id="81-수치형-하나">8.1 수치형 하나</h3>
<p>기본 도구는 Histogram입니다. 필요에 따라 Frequency Polygon이나 KDE로 부드러운 밀도 형태를 봅니다.</p>
<p>확인할 질문:</p>
<ul>
<li>이상점이 있는가?</li>
<li>종 모양인가, 한쪽으로 치우쳤는가?</li>
<li>봉우리가 하나인가, 여러 개인가?</li>
<li>bin 개수에 따라 해석이 달라지는가?</li>
</ul>
<h3 id="82-범주형-하나">8.2 범주형 하나</h3>
<p>Bar Chart나 도수표가 기본입니다. Pie Chart도 가능하지만 범주가 많으면 각도를 비교하기 어려워 Bar Chart가 대체로 읽기 쉽습니다.</p>
<h3 id="83-수치형과-수치형">8.3 수치형과 수치형</h3>
<p>Scatter Plot을 사용합니다.</p>
<ul>
<li>선형인가, 비선형인가?</li>
<li>관계가 강한가, 약한가?</li>
<li>이상점이 있는가?</li>
<li>로그 변환이 필요한가?</li>
<li>점이 너무 많다면 표본 추출이나 투명도 조절이 필요한가?</li>
</ul>
<p>원인으로 가정한 변수를 x축, 결과 변수를 y축에 두면 설명하기 쉽습니다. 다만 축의 배치가 인과관계를 증명하지는 않습니다.</p>
<h3 id="84-수치형과-범주형">8.4 수치형과 범주형</h3>
<p>Box Plot으로 범주별 수치 분포를 비교합니다. 관측치가 적은 범주는 점을 함께 표시해야 실제 표본 수를 오해하지 않습니다.</p>
<h3 id="85-범주형과-범주형">8.5 범주형과 범주형</h3>
<p>교차표 또는 Mosaic Plot을 사용합니다. 성별과 객실 등급에 따른 생존 여부처럼 제3의 결과 변수를 비율로 추가하면 더 의미 있는 질문을 만들 수 있습니다.</p>
<h2 id="9-question-driven-eda">9. Question-driven EDA</h2>
<p>EDA의 핵심은 관찰을 다음 행동으로 연결하는 것입니다.</p>
<table>
<thead>
<tr>
<th>질문</th>
<th>분석 방법</th>
<th>가능한 다음 행동</th>
</tr>
</thead>
<tbody><tr>
<td>결측치는 어디에 얼마나 있는가?</td>
<td>결측 개수·비율·패턴</td>
<td>삭제, 대체, 결측 여부 변수 생성</td>
</tr>
<tr>
<td>중복·유효하지 않은 값이 있는가?</td>
<td>중복, 타입, 범위, 업무 규칙</td>
<td>중복 제거, 타입 변환, 오류 수정</td>
</tr>
<tr>
<td>수치형 분포는 어떠한가?</td>
<td>기술 통계, Histogram, KDE, 왜도</td>
<td>변환, 스케일링, 구간화</td>
</tr>
<tr>
<td>이상치는 있는가?</td>
<td>Box Plot, IQR, Z-score</td>
<td>원인 조사, 제거, 대체, 영향 완화</td>
</tr>
<tr>
<td>범주 구성은 적절한가?</td>
<td><code>nunique()</code>, <code>value_counts()</code></td>
<td>범주 통합, 인코딩 결정</td>
</tr>
<tr>
<td>비슷한 정보를 가진 변수가 있는가?</td>
<td>상관행렬, Heatmap</td>
<td>중복 변수 제거, 다중공선성 검토</td>
</tr>
</tbody></table>
<p>Target이 수치형인 회귀 문제에서는 Scatter Plot, 범주별 Box Plot, 그룹 평균을 봅니다. Target이 범주형인 분류 문제에서는 Target별 수치 분포, 범주별 Target 비율을 봅니다.</p>
<hr />
<h1 id="eda-실습">EDA 실습</h1>
<p>아래 코드는 가상의 구매 데이터로 전체 흐름을 연습하는 예입니다.</p>
<pre><code class="language-python">import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.DataFrame({
    &quot;age&quot;: [25, 32, None, 41, 29, 35, 46, 50, 22, 27],
    &quot;purchase_amount&quot;: [32000, 45000, 39000, 68000, 42000,
                        61000, 55000, 72000, 30000, 44000],
    &quot;department&quot;: [&quot;HR&quot;, &quot;IT&quot;, &quot;Sales&quot;, &quot;IT&quot;, &quot;HR&quot;,
                   &quot;IT&quot;, &quot;Sales&quot;, &quot;IT&quot;, &quot;HR&quot;, &quot;Sales&quot;],
    &quot;purchased&quot;: [0, 1, 0, 1, 1, 1, 0, 1, 0, 1]
})</code></pre>
<h2 id="step-1-구조를-확인한다">Step 1. 구조를 확인한다</h2>
<pre><code class="language-python">print(df.shape)
print(df.columns)
df.info()</code></pre>
<p>생각할 점:</p>
<ul>
<li>행이 10개뿐이므로 결과를 일반화하면 안 됩니다.</li>
<li><code>age</code>에는 결측치가 있습니다.</li>
<li><code>department</code>는 문자열 범주형 변수입니다.</li>
<li><code>purchased</code>는 0과 1로 표현된 Target 후보입니다.</li>
</ul>
<h2 id="step-2-실제-값과-요약-통계를-함께-본다">Step 2. 실제 값과 요약 통계를 함께 본다</h2>
<pre><code class="language-python">print(df.head())
print(df.sample(5, random_state=42))
print(df.describe(include=&quot;all&quot;))</code></pre>
<p>평균만 보지 말고 중앙값, 표준편차, 최소·최대, 사분위수를 함께 봅니다. 범주형 변수에서는 <code>unique</code>, <code>top</code>, <code>freq</code>도 확인합니다.</p>
<h2 id="step-3-범주와-결측을-확인한다">Step 3. 범주와 결측을 확인한다</h2>
<pre><code class="language-python">print(df[&quot;department&quot;].unique())
print(df[&quot;department&quot;].nunique())
print(df[&quot;department&quot;].value_counts())
print(df[&quot;department&quot;].value_counts(normalize=True))

print(df.isnull().sum())
print(df.isnull().mean())</code></pre>
<p>여기서 “나이가 비어 있다”는 사실만 알 수 있습니다. 왜 비었는지는 데이터 수집 과정이나 업무 담당자에게 확인해야 합니다.</p>
<h2 id="step-4-분포와-관계를-시각화한다">Step 4. 분포와 관계를 시각화한다</h2>
<pre><code class="language-python">sns.histplot(data=df, x=&quot;purchase_amount&quot;, kde=True)
plt.show()

sns.boxplot(data=df, x=&quot;purchased&quot;, y=&quot;purchase_amount&quot;)
plt.show()

sns.scatterplot(data=df, x=&quot;age&quot;, y=&quot;purchase_amount&quot;, hue=&quot;purchased&quot;)
plt.show()

corr = df.corr(numeric_only=True)
sns.heatmap(corr, annot=True, cmap=&quot;Blues&quot;, vmin=-1, vmax=1)
plt.show()</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/xxtimrecht/post/ac273b80-a8b7-4284-aa47-414b58fe4cc6/image.png" /></p>
<h2 id="step-5-관찰을-질문으로-바꾼다">Step 5. 관찰을 질문으로 바꾼다</h2>
<p>단순 관찰: 구매한 집단의 구매금액이 더 커 보인다.<br />좋은 질문: 구매금액이 커서 구매했다고 표시된 것인가, 구매 완료 후 금액이 기록된 것인가?</p>
<p>두 경우의 의미는 완전히 다릅니다. 두 번째라면 <code>purchase_amount</code>는 예측 시점에 알 수 없는 정보여서 <strong>Target Leakage</strong>가 될 수 있습니다.</p>
<h2 id="step-6-eda-결과표를-만든다">Step 6. EDA 결과표를 만든다</h2>
<table>
<thead>
<tr>
<th>발견</th>
<th>가능한 원인</th>
<th>모델 영향</th>
<th>다음 행동</th>
</tr>
</thead>
<tbody><tr>
<td><code>age</code> 결측 10%</td>
<td>미입력 또는 수집 오류</td>
<td>일부 모델이 학습 불가</td>
<td>발생 원인 확인 후 대체·삭제 검토</td>
</tr>
<tr>
<td>부서별 표본 수 차이</td>
<td>실제 구성 또는 표본 편향</td>
<td>소수 범주 예측 불안정</td>
<td>더 많은 데이터 확인</td>
</tr>
<tr>
<td>구매금액과 Target 관계가 큼</td>
<td>실제 신호 또는 Leakage</td>
<td>과도하게 높은 성능 가능</td>
<td>변수 생성 시점 확인</td>
</tr>
</tbody></table>
<blockquote>
<p>실습의 정답은 그래프가 아니라, “발견 → 원인 가설 → 모델 영향 → 다음 행동”의 연결입니다.</p>
</blockquote>
<hr />
<h1 id="pre-processing-basic-1">Pre-processing Basic 1</h1>
<h2 id="1-결측치missing-value">1. 결측치(Missing Value)</h2>
<p>결측치는 값이 존재하지 않는 상태입니다. 데이터에서는 <code>Null</code>, <code>NA</code>, <code>NaN</code> 등으로 표현됩니다.</p>
<h3 id="11-결측치가-문제인-이유">1.1 결측치가 문제인 이유</h3>
<ul>
<li>알고리즘이나 연산이 결측값을 처리하지 못할 수 있습니다.</li>
<li>결측 행을 모두 제거하면 중요한 집단이 더 많이 사라질 수 있습니다.</li>
<li>잘못 대체하면 원래 분포와 변수 관계가 왜곡됩니다.</li>
<li>결측 자체가 업무적으로 중요한 신호일 수 있습니다.</li>
</ul>
<p>예를 들어 대출 신청서에서 소득 미기재는 단순 실수가 아니라 특정 고객 행동과 연결될 수 있습니다. <code>소득을 평균으로 채우기</code> 전에 <code>소득_결측여부</code>라는 변수를 만드는 이유입니다.</p>
<h3 id="12-결측-메커니즘">1.2 결측 메커니즘</h3>
<ul>
<li>무작위 결측: 다른 관측 변수나 실제 값과 무관하게 우연히 누락</li>
<li>조건부 결측: 관측된 다른 변수와 관련되어 누락</li>
<li>비무작위 결측: 누락된 값 자체와 관련되어 누락</li>
</ul>
<p>예를 들어 체중이 높은 사람이 체중 공개를 더 자주 거부한다면 결측이 무작위가 아닙니다. 이런 값을 단순 평균으로 대체하면 전체 체중을 낮게 추정할 수 있습니다.</p>
<h3 id="13-제거">1.3 제거</h3>
<ul>
<li>결측 비율이 작고 무작위에 가깝다면 행 제거를 검토합니다.</li>
<li>한 열의 대부분이 비어 있고 업무 가치도 낮다면 열 제거를 검토합니다.</li>
<li>흔히 “행 결측 10% 이하”, “열 결측 50% 이상” 같은 수치를 참고하지만 절대 규칙은 아닙니다.</li>
</ul>
<p>100명 중 5명의 질병 환자 데이터가 모두 결측이라면 전체의 5%밖에 안 되어도 삭제하면 안 됩니다. 비율보다 <strong>누가 사라지는지</strong>가 중요합니다.</p>
<h3 id="14-단순-대체">1.4 단순 대체</h3>
<ul>
<li>범주형: 최빈값 또는 <code>Unknown</code> 범주</li>
<li>대칭적인 수치형: 평균</li>
<li>치우침과 이상치가 큰 수치형: 중앙값</li>
<li>그룹 차이가 명확함: 그룹별 평균·중앙값 같은 조건부 대체</li>
</ul>
<pre><code class="language-python">df[&quot;age_missing&quot;] = df[&quot;age&quot;].isna().astype(int)
df[&quot;age&quot;] = df[&quot;age&quot;].fillna(df[&quot;age&quot;].median())</code></pre>
<h3 id="15-시계열-대체">1.5 시계열 대체</h3>
<p>시계열은 시간 구간마다 평균과 분산이 달라질 수 있으므로 전체 평균 대체가 위험합니다.</p>
<ul>
<li>직전 관측값 사용(Forward Fill)</li>
<li>이동 평균 또는 이동 중앙값</li>
<li>선형 보간</li>
<li>Spline 보간</li>
</ul>
<p>Cubic Spline은 곡선을 부드럽게 만들지만 실제로 존재하지 않은 과도한 값까지 만들 수 있습니다. 센서의 물리적 범위와 시간 간격을 반드시 확인해야 합니다.</p>
<h3 id="16-예측-기반-대체">1.6 예측 기반 대체</h3>
<p>결측이 없는 행으로 회귀, KNN 등의 모델을 학습해 결측값을 예측할 수 있습니다. 변수 사이 관계를 활용한다는 장점이 있지만, 예측값을 사실처럼 취급해 불확실성을 감추거나 업무상 불가능한 값을 만들 수 있습니다.</p>
<h2 id="2-이상치outlier">2. 이상치(Outlier)</h2>
<p>이상치는 전체 범위에서 크게 벗어난 매우 크거나 작은 관측값입니다.</p>
<h3 id="21-왜-문제가-되는가">2.1 왜 문제가 되는가?</h3>
<ul>
<li>평균과 표준편차가 크게 흔들립니다.</li>
<li>분산이 과도하게 커집니다.</li>
<li>선형 모델이 일부 극단값을 맞추는 데 끌려갈 수 있습니다.</li>
<li>작은 데이터일수록 영향이 큽니다.</li>
</ul>
<p>하지만 이상치는 반드시 오류가 아닙니다. 금융 사기, 설비 고장, 중환자처럼 우리가 찾고 싶은 사건일 수도 있습니다.</p>
<h3 id="22-iqr-방식">2.2 IQR 방식</h3>
<pre><code class="language-text">IQR = Q3 - Q1
하한 = Q1 - 1.5 × IQR
상한 = Q3 + 1.5 × IQR</code></pre>
<p>하한보다 작거나 상한보다 큰 값을 이상치 후보로 표시합니다. 정규분포를 강하게 가정하지 않는 장점이 있지만, 후보를 찾는 통계 규칙일 뿐 오류 판정 규칙은 아닙니다.</p>
<h3 id="23-z-score-방식">2.3 Z-score 방식</h3>
<pre><code class="language-text">z = (개별값 - 평균) / 표준편차</code></pre>
<p>정규분포에 가까운 데이터에서 보통 <code>|z| &gt; 3</code>인 값을 이상치 후보로 봅니다. 분포가 심하게 치우쳤거나 이상치 때문에 평균과 표준편차가 이미 왜곡되었다면 적합하지 않을 수 있습니다.</p>
<h3 id="24-처리-방법">2.4 처리 방법</h3>
<ul>
<li>제거: 명백한 입력 오류이며 원래 값을 복구할 수 없을 때</li>
<li>수정: 나이 <code>355</code>가 <code>35</code>의 오타임을 근거로 확인했을 때</li>
<li>상·하한 대체(Winsorization): 극단값을 정한 경계값으로 제한</li>
<li>가중치 조정: 극단값이 학습에 미치는 영향 축소</li>
<li>변환: 로그 등으로 큰 값의 영향 완화</li>
<li>유지: 희귀하지만 실제로 중요한 사건일 때</li>
</ul>
<p>무조건 많은 이상치를 제거하면 데이터가 인위적으로 깨끗해지고 테스트 성능이 과장될 수 있습니다.</p>
<blockquote>
<p>이상치 처리의 첫 질문은 “얼마나 멀리 있는가?”가 아니라 “왜 생겼는가?”입니다.</p>
</blockquote>
<hr />
<h1 id="pre-processing-basic-2">Pre-processing Basic 2</h1>
<h2 id="1-data-scaling">1. Data Scaling</h2>
<p>Scaling은 서로 다른 변수의 값 범위 또는 분포 규모를 비슷하게 조정하는 작업입니다.</p>
<p>예를 들어 나이는 20~60, 연봉은 수천만 단위라면 거리 기반 알고리즘은 연봉 차이를 훨씬 크게 느낍니다. 값이 크다는 이유만으로 더 중요한 변수가 되어서는 안 됩니다.</p>
<h3 id="11-scaling이-필요한-이유">1.1 Scaling이 필요한 이유</h3>
<ul>
<li>거리 계산에서 단위가 큰 변수의 지배를 막습니다.</li>
<li>Gradient 기반 학습의 안정성과 수렴 속도를 높일 수 있습니다.</li>
<li>매우 큰 값과 작은 값으로 인한 수치적 overflow·underflow 위험을 줄입니다.</li>
<li>변수 단위 차이 때문에 행렬의 condition number가 커지는 문제를 완화할 수 있습니다.</li>
</ul>
<p>Scaling이 특히 중요한 모델은 KNN, K-means, SVM, 선형·로지스틱 회귀, 신경망 등입니다. 의사결정나무와 트리 기반 앙상블은 변수의 순서와 분기점을 사용하므로 일반적으로 영향이 작습니다.</p>
<h3 id="12-min-max-scaling">1.2 Min-Max Scaling</h3>
<pre><code class="language-text">x_scaled = (x - min) / (max - min)</code></pre>
<ul>
<li>보통 0~1 범위로 변환</li>
<li>최솟값은 0, 최댓값은 1</li>
<li>범위가 명확하다는 장점</li>
<li>이상치가 있으면 대부분의 정상값이 좁은 구간에 압축됨</li>
</ul>
<h3 id="13-standard-scaling">1.3 Standard Scaling</h3>
<pre><code class="language-text">x_scaled = (x - 평균) / 표준편차</code></pre>
<ul>
<li>평균 0, 표준편차 1이 되도록 조정</li>
<li>결과 범위는 제한되지 않음</li>
<li>평균과 표준편차를 사용하므로 이상치에 민감</li>
<li>“정규분포로 만든다”기보다 평균과 표준편차를 맞춘다고 이해하는 것이 정확함</li>
</ul>
<h3 id="14-robust-scaling">1.4 Robust Scaling</h3>
<pre><code class="language-text">x_scaled = (x - 중앙값) / IQR</code></pre>
<ul>
<li>중앙값을 0 부근으로, IQR을 1 규모로 조정</li>
<li>평균 대신 중앙값, 표준편차 대신 IQR을 사용해 이상치에 비교적 강함</li>
<li>변환값의 최솟값과 최댓값은 제한되지 않음</li>
</ul>
<h3 id="15-normalization">1.5 Normalization</h3>
<p>각 행의 벡터 길이가 1이 되도록 조정합니다. 열 단위 통계를 맞추는 Scaling과 달리 <strong>행 단위</strong>로 적용됩니다. 문서의 단어 빈도나 방향의 유사성이 중요한 문제에서 자주 사용합니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/xxtimrecht/post/a880198f-f22c-433c-92a2-664c0625cdaa/image.png" /></p>
<h3 id="16-가장-중요한-원칙-학습-데이터로만-기준을-구한다">1.6 가장 중요한 원칙: 학습 데이터로만 기준을 구한다</h3>
<pre><code class="language-python">from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X_train, X_test = train_test_split(X, test_size=0.2, random_state=42)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # 기준 학습
X_test_scaled = scaler.transform(X_test)        # 같은 기준 적용</code></pre>
<p>전체 데이터에 먼저 <code>fit</code>하면 테스트 데이터의 평균과 표준편차가 학습 과정에 들어갑니다. 이것도 Data Leakage입니다.</p>
<h2 id="2-data-encoding">2. Data Encoding</h2>
<p>범주형 변수는 서로 다른 그룹을 나타냅니다. 숫자로 적혀 있어도 값의 차이나 순서가 의미 없을 수 있습니다. 대부분의 머신러닝 알고리즘은 문자열을 그대로 계산할 수 없으므로 수치 표현으로 바꿔야 합니다.</p>
<h3 id="21-label-encoding">2.1 Label Encoding</h3>
<pre><code class="language-text">서울 → 1
부산 → 2
제주 → 3</code></pre>
<ul>
<li>간단하고 열이 늘어나지 않음</li>
<li>모델이 <code>제주 &gt; 부산 &gt; 서울</code> 같은 가짜 순서를 학습할 수 있음</li>
<li>순서형 범주나 Target label에 주로 사용</li>
<li>명목형 입력 변수에는 신중해야 함</li>
</ul>
<h3 id="22-one-hot-encoding">2.2 One-Hot Encoding</h3>
<pre><code class="language-text">도시  서울  부산  제주
서울   1     0     0
부산   0     1     0
제주   0     0     1</code></pre>
<ul>
<li>각 범주가 독립적인 열이 되므로 가짜 순서가 없음</li>
<li>범주가 <code>k</code>개면 보통 <code>k</code>개의 열 생성</li>
<li>열 수와 메모리 사용량이 늘어남</li>
<li>선형 모델에서는 모든 열의 합이 항상 1이 되어 완전한 다중공선성이 생길 수 있음</li>
</ul>
<h3 id="23-dummy-coding">2.3 Dummy Coding</h3>
<p>One-Hot 열 중 하나를 기준 범주로 제외해 <code>k-1</code>개 열만 만듭니다. 제외된 범주는 모든 Dummy 값이 0인 상태로 표현됩니다.</p>
<ul>
<li>선형 모델의 Dummy Variable Trap을 피할 수 있음</li>
<li>다른 범주의 계수를 기준 범주와 비교해 해석 가능</li>
<li>미지 범주 또는 결측도 모두 0으로 처리하면 기준 범주와 구분되지 않을 수 있음
<img alt="" src="https://velog.velcdn.com/images/xxtimrecht/post/6b76dd4c-cdd8-4cb1-b556-8e9220551c84/image.png" /></li>
</ul>
<h3 id="24-high-cardinality-문제">2.4 High Cardinality 문제</h3>
<p>사용자 ID처럼 범주가 100,000개라면 One-Hot Encoding은 열을 폭발적으로 늘립니다.</p>
<p>대안:</p>
<ul>
<li>의미가 비슷한 범주를 그룹화</li>
<li>드문 범주를 <code>기타</code>로 통합</li>
<li>빈도 또는 횟수로 Encoding</li>
<li>Target의 조건부 통계량으로 Encoding</li>
</ul>
<p>Target Encoding은 강력하지만 전체 데이터의 Target 평균을 그대로 사용하면 Leakage가 발생합니다. 학습 fold 안에서만 통계량을 계산하고, 작은 범주는 전체 평균 쪽으로 완화하는 방법이 필요합니다.</p>
<hr />
<h1 id="pre-processing-advanced">Pre-processing Advanced</h1>
<h2 id="1-variable-transformation">1. Variable Transformation</h2>
<p>변수 변환은 정보와 순서를 가능한 한 유지하면서 <strong>표현 방식</strong>을 바꾸는 과정입니다. 변환 후 분포, 척도, 변수 관계, 분산이 달라져 모델이 패턴을 더 쉽게 학습할 수 있습니다.</p>
<h3 id="11-변환하지-않았을-때-생길-수-있는-문제">1.1 변환하지 않았을 때 생길 수 있는 문제</h3>
<ul>
<li>큰 값과 이상치의 영향이 지나치게 커짐</li>
<li>곡선 관계를 선형 모델이 충분히 표현하지 못함</li>
<li>큰 값 구간에만 학습이 집중됨</li>
<li>값이 커질수록 오차 분산이 커져 회귀 추정이 불안정해짐</li>
</ul>
<h3 id="12-기대-효과">1.2 기대 효과</h3>
<ul>
<li>왜도 감소</li>
<li>극단값의 상대적 영향 감소</li>
<li>비선형 관계를 선형에 가깝게 표현</li>
<li>값의 크기에 따라 달라지는 분산을 안정화</li>
</ul>
<h2 id="2-log-transformation">2. Log Transformation</h2>
<p>로그는 “밑을 몇 번 거듭제곱하면 이 값이 되는가?”를 나타냅니다. 원래 값의 순서를 유지하면서 큰 범위를 압축합니다.</p>
<pre><code class="language-text">원래 값: 10 → 100 → 1,000
log10:    1 →   2 →     3</code></pre>
<p>원래 값이 10배가 될 때 로그값은 1씩 증가합니다. 매출, 조회수, 소득처럼 오른쪽 꼬리가 긴 데이터에 유용합니다.</p>
<pre><code class="language-python">import numpy as np

df[&quot;amount_log&quot;] = np.log1p(df[&quot;purchase_amount&quot;])</code></pre>
<h3 id="21-유의할-점">2.1 유의할 점</h3>
<ul>
<li>일반적인 로그는 양수에서 정의됩니다.</li>
<li>0이 포함되면 <code>log(x + 1)</code>인 <code>np.log1p(x)</code>를 고려합니다.</li>
<li>음수가 포함되면 Yeo-Johnson 변환을 고려합니다.</li>
<li>변환 후 해석 단위가 원래 단위가 아니라 로그 단위가 됩니다.</li>
<li>자연로그와 상용로그는 척도만 달라지고 압축 효과의 본질은 같습니다.</li>
</ul>
<h3 id="22-다른-변환">2.2 다른 변환</h3>
<table>
<thead>
<tr>
<th>변환</th>
<th>적합한 상황</th>
<th>핵심 효과</th>
</tr>
</thead>
<tbody><tr>
<td>Log</td>
<td>오른쪽 꼬리가 길고 값 범위가 큼</td>
<td>큰 값을 강하게 압축</td>
</tr>
<tr>
<td>Square Root</td>
<td>횟수·건수 같은 Count 데이터</td>
<td>로그보다 완만하게 압축</td>
</tr>
<tr>
<td>Box-Cox</td>
<td>모든 값이 양수</td>
<td>적절한 변환 강도 <code>λ</code> 탐색</td>
</tr>
<tr>
<td>Yeo-Johnson</td>
<td>0이나 음수가 포함</td>
<td>Box-Cox와 비슷한 목적을 더 넓은 값에 적용</td>
</tr>
</tbody></table>
<h2 id="3-imbalanced-data">3. Imbalanced Data</h2>
<p>불균형 데이터는 분류 Target의 클래스별 관측치 수가 크게 다른 데이터입니다.</p>
<ul>
<li>제조: 정상 제품 vs 불량품</li>
<li>금융: 정상 거래 vs 사기 거래</li>
<li>의료: 정상 상태 vs 특정 질병</li>
<li>플랫폼: 유지 고객 vs 이탈 고객</li>
</ul>
<p>정상 99%, 이상 1%인 문제에서 모두 정상이라고 예측하면 정확도는 99%입니다. 그러나 이상을 한 건도 찾지 못하므로 목적에 맞지 않습니다. Precision, Recall, F1-score, PR-AUC, 비용 기반 평가를 함께 봐야 합니다.</p>
<h2 id="4-undersampling">4. Undersampling</h2>
<p>다수 클래스의 표본을 줄여 균형을 맞추는 방법입니다.</p>
<h3 id="41-random-undersampling">4.1 Random Undersampling</h3>
<ul>
<li>다수 클래스를 무작위 제거</li>
<li>빠르고 단순함</li>
<li>시행마다 결과가 달라질 수 있음</li>
<li>유용한 정보를 함께 버릴 위험</li>
</ul>
<h3 id="42-tomek-links">4.2 Tomek Links</h3>
<p>서로 다른 클래스의 두 점이 매우 가까워 서로의 가장 가까운 이웃이 되는 경계 쌍을 찾고, 보통 다수 클래스 점을 제거합니다.</p>
<ul>
<li>결정 경계를 더 선명하게 만들 수 있음</li>
<li>경계의 노이즈를 정리하는 데 유용</li>
<li>제거되는 표본 수가 제한적이어서 균형 자체를 크게 바꾸지 못할 수 있음</li>
</ul>
<h3 id="43-condensed-nearest-neighbor">4.3 Condensed Nearest Neighbor</h3>
<p>소수 클래스와 대표적인 다수 클래스만 남긴 뒤, 1-NN으로 분류에 필요한 다수 표본을 점차 추가하는 접근입니다.</p>
<ul>
<li>밀집된 다수 클래스의 중복 표본을 크게 줄일 수 있음</li>
<li>선택된 초기 표본과 노이즈에 영향을 받을 수 있음</li>
</ul>
<h2 id="5-oversampling">5. Oversampling</h2>
<p>소수 클래스 표본을 늘리는 방법입니다.</p>
<h3 id="51-random-oversampling">5.1 Random Oversampling</h3>
<ul>
<li>소수 클래스 데이터를 무작위 복제</li>
<li>원본 정보를 잃지 않음</li>
<li>같은 점을 반복 학습해 과적합할 위험</li>
<li>특정 표본이 지나치게 많이 복제될 수 있음</li>
</ul>
<h3 id="52-smote">5.2 SMOTE</h3>
<p>SMOTE(Synthetic Minority Oversampling Technique)는 소수 클래스 점과 가까운 소수 이웃을 연결한 선 위에 새로운 합성 표본을 만듭니다.</p>
<pre><code class="language-text">소수 점 A ●──────×──────● 소수 이웃 B
                새 점</code></pre>
<ul>
<li>단순 복제가 아니라 새로운 점을 생성</li>
<li>소수 클래스 영역을 부드럽게 확장</li>
<li>노이즈 점 주변에서도 가짜 표본을 만들 수 있음</li>
<li>서로 다른 소수 집단 사이에 부적절한 점을 만들 가능성</li>
</ul>
<h3 id="53-borderline-smote">5.3 Borderline-SMOTE</h3>
<p>주변 이웃의 절반 이상이 다수 클래스인 소수 표본을 <code>danger set</code>으로 보고, 결정 경계 근처에 집중해 SMOTE를 적용합니다. 분류하기 어려운 구간을 보강하지만 경계의 노이즈도 키울 수 있습니다.</p>
<h3 id="54-adasyn">5.4 ADASYN</h3>
<p>소수 표본마다 주변 다수 클래스 비율을 계산해 어려운 곳일수록 더 많은 합성 표본을 생성합니다. 학습이 어려운 영역에 집중한다는 장점이 있지만, 이상치 주변을 어려운 영역으로 오해하면 노이즈를 증폭할 수 있습니다.</p>
<h2 id="6-sampling-실무-원칙">6. Sampling 실무 원칙</h2>
<ol>
<li>Train과 Test를 먼저 분리합니다.</li>
<li>Sampling은 Train에만 적용합니다.</li>
<li>Validation도 원래 현실의 클래스 비율을 유지하는 것이 일반적입니다.</li>
<li>교차검증에서는 각 학습 fold 내부에서만 Sampling합니다.</li>
<li>정확도보다 문제에 맞는 지표를 사용합니다.</li>
<li>Sampling 전후의 분포와 결정 경계를 시각적으로 비교합니다.</li>
</ol>
<pre><code class="language-python">from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_train_resampled, y_train_resampled = smote.fit_resample(
    X_train, y_train
)
# X_test, y_test에는 SMOTE를 적용하지 않는다.</code></pre>
<p>극단적 불균형에서는 다수 클래스가 전체 공간에 충분히 넓게 분포하므로 일부를 줄여도 결정 경계를 유지하는 경우가 있습니다. Undersampling이 항상 위험하고 Oversampling이 항상 안전한 것은 아닙니다. 선택은 실험과 검증의 대상입니다.</p>
<hr />
<h1 id="feature-engineering-실습">Feature Engineering 실습</h1>
<h2 id="1-좋은-feature를-만드는-여섯-가지-전략">1. 좋은 Feature를 만드는 여섯 가지 전략</h2>
<table>
<thead>
<tr>
<th>전략</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>비율(Ratio)</td>
<td>변수 간 상대적 크기</td>
<td>구매금액 / 구매횟수</td>
</tr>
<tr>
<td>차이(Difference)</td>
<td>간격이나 변화</td>
<td>배송일 - 주문일</td>
</tr>
<tr>
<td>상호작용(Interaction)</td>
<td>두 변수의 결합 효과</td>
<td>면적 × 층수</td>
</tr>
<tr>
<td>시간 Feature</td>
<td>날짜에서 시간 정보 추출</td>
<td>요일, 월, 가입 경과일</td>
</tr>
<tr>
<td>구간화(Binning)</td>
<td>연속값을 의미 있는 구간으로 변환</td>
<td>나이 → 연령대</td>
</tr>
<tr>
<td>집계(Aggregation)</td>
<td>여러 행을 하나의 요약 정보로 변환</td>
<td>고객별 평균 구매액</td>
</tr>
</tbody></table>
<h2 id="2-계산식보다-질문에서-시작한다">2. 계산식보다 질문에서 시작한다</h2>
<table>
<thead>
<tr>
<th>원본 Feature</th>
<th>질문</th>
<th>새 Feature</th>
<th>전략</th>
</tr>
</thead>
<tbody><tr>
<td>키, 몸무게</td>
<td>체형을 더 잘 표현할 수 있을까?</td>
<td>BMI</td>
<td>비율</td>
</tr>
<tr>
<td>주문일, 배송일</td>
<td>배송은 얼마나 걸렸는가?</td>
<td>배송 소요일</td>
<td>차이·시간</td>
</tr>
<tr>
<td>구매금액, 구매횟수</td>
<td>한 번에 얼마나 구매하는가?</td>
<td>평균 구매금액</td>
<td>비율·집계</td>
</tr>
<tr>
<td>가입일</td>
<td>얼마나 오래 거래했는가?</td>
<td>가입 경과일</td>
<td>시간</td>
</tr>
<tr>
<td>생년월일</td>
<td>나이를 더 쉽게 활용할 수 있을까?</td>
<td>연령대</td>
<td>시간·구간화</td>
</tr>
<tr>
<td>구매 이력</td>
<td>얼마나 자주 구매하는가?</td>
<td>구매 빈도</td>
<td>집계</td>
</tr>
<tr>
<td>건물 면적, 층수</td>
<td>면적에 따라 층수 영향이 달라지는가?</td>
<td>면적 × 층수</td>
<td>상호작용</td>
</tr>
<tr>
<td>대출금, 소득</td>
<td>상환 부담은 어느 정도인가?</td>
<td>DTI</td>
<td>비율</td>
</tr>
<tr>
<td>거리, 이동시간</td>
<td>평균 이동 속도는 얼마인가?</td>
<td>거리 / 시간</td>
<td>비율</td>
</tr>
<tr>
<td>최근 로그인일</td>
<td>최근 활동했는가?</td>
<td>마지막 로그인 후 경과일</td>
<td>시간</td>
</tr>
</tbody></table>
<blockquote>
<p>파생변수는 단순한 계산식이 아니라 실제 문제를 더 잘 표현하는 방법입니다.</p>
</blockquote>
<h2 id="3-고객-구매-데이터-실습">3. 고객 구매 데이터 실습</h2>
<pre><code class="language-python">import numpy as np
import pandas as pd

reference_date = pd.Timestamp(&quot;2026-01-01&quot;)

# 1) 비율: 한 번 구매할 때 평균적으로 쓴 금액
df[&quot;avg_purchase_amount&quot;] = np.where(
    df[&quot;purchase_count&quot;] &gt; 0,
    df[&quot;total_purchase_amount&quot;] / df[&quot;purchase_count&quot;],
    0
)

# 2) 시간: 최근 구매 후 경과일
df[&quot;recency_days&quot;] = (
    reference_date - pd.to_datetime(df[&quot;last_purchase_date&quot;])
).dt.days

# 3) 시간: 가입 후 경과일
df[&quot;membership_days&quot;] = (
    reference_date - pd.to_datetime(df[&quot;join_date&quot;])
).dt.days

# 4) 구간화: 고객 활동 수준
df[&quot;activity_level&quot;] = pd.cut(
    df[&quot;login_count_30d&quot;],
    bins=[-1, 0, 5, 20, float(&quot;inf&quot;)],
    labels=[&quot;inactive&quot;, &quot;low&quot;, &quot;medium&quot;, &quot;high&quot;]
)

# 5) 상호작용: 방문이 구매로 연결되는 정도
df[&quot;purchase_per_visit&quot;] = np.where(
    df[&quot;visit_count&quot;] &gt; 0,
    df[&quot;purchase_count&quot;] / df[&quot;visit_count&quot;],
    0
)</code></pre>
<h3 id="실습에서-확인할-것">실습에서 확인할 것</h3>
<ul>
<li>분모가 0일 때 어떻게 처리할 것인가?</li>
<li>기준일은 예측 시점보다 미래가 아닌가?</li>
<li><code>마지막 구매일</code>이 Target 발생 이후의 정보는 아닌가?</li>
<li>집계 기간은 최근 7일, 30일, 90일 중 무엇이 적절한가?</li>
<li>생성한 Feature가 새 고객에게도 계산 가능한가?</li>
</ul>
<h2 id="4-domain-knowledge가-만드는-feature">4. Domain Knowledge가 만드는 Feature</h2>
<p>같은 원본 데이터라도 도메인 질문에 따라 의미 있는 Feature가 달라집니다.</p>
<table>
<thead>
<tr>
<th>분야</th>
<th>원본 값</th>
<th>도메인 질문</th>
<th>파생 Feature</th>
</tr>
</thead>
<tbody><tr>
<td>헬스케어</td>
<td>키, 몸무게</td>
<td>체형과 비만 정도를 어떻게 표현할까?</td>
<td>BMI</td>
</tr>
<tr>
<td>투자</td>
<td>수익률, 변동성</td>
<td>위험을 감수한 만큼 수익을 냈는가?</td>
<td>Sharpe Ratio</td>
</tr>
<tr>
<td>투자</td>
<td>최고점, 현재값</td>
<td>고점에서 얼마나 하락했는가?</td>
<td>Drawdown</td>
</tr>
<tr>
<td>신용</td>
<td>사용액, 한도</td>
<td>신용한도를 얼마나 사용했는가?</td>
<td>Credit Utilization</td>
</tr>
<tr>
<td>이커머스</td>
<td>조회수, 구매수</td>
<td>조회가 구매로 얼마나 연결되는가?</td>
<td>Conversion Rate</td>
</tr>
</tbody></table>
<p>이 지표들은 단순히 나누거나 빼서 생긴 것이 아닙니다. 각 분야에서 오랫동안 중요한 의미가 있다고 검증된 질문을 수식으로 표현한 것입니다.</p>
<h2 id="5-rfm-예시">5. RFM 예시</h2>
<p>고객 가치를 표현하는 대표적인 방식입니다.</p>
<ul>
<li>Recency: 최근에 구매했는가? → <code>기준일 - 마지막 구매일</code></li>
<li>Frequency: 얼마나 자주 구매했는가? → <code>기간 내 구매 횟수</code></li>
<li>Monetary: 얼마나 많이 구매했는가? → <code>기간 내 총 구매금액</code></li>
</ul>
<p>원본 거래 행 수천 개를 고객당 3개의 의미 있는 정보로 요약할 수 있습니다. 단, 모든 변수는 예측 시점 이전 기록만 사용해야 합니다.</p>
<h2 id="6-좋은-feature의-체크리스트">6. 좋은 Feature의 체크리스트</h2>
<ul>
<li>예측 시점에 실제로 알 수 있는가?</li>
<li>Target의 미래 정보를 몰래 포함하지 않는가?</li>
<li>업무적으로 설명할 수 있는가?</li>
<li>분모 0, 결측, 음수, 무한대를 처리했는가?</li>
<li>학습과 운영에서 같은 방식으로 계산되는가?</li>
<li>새 데이터와 새로운 범주에도 동작하는가?</li>
<li>성능 향상이 교차검증에서도 반복되는가?</li>
</ul>
<hr />
<h1 id="quiz--wrap-up">Quiz &amp; Wrap-up</h1>
<h2 id="1-확인-quiz">1. 확인 Quiz</h2>
<details>
Q1. EDA는 그래프를 많이 그리는 과정이다. O/X

<p><strong>정답: X</strong><br />EDA는 구조와 품질을 확인하고, 질문과 가설을 만들어 다음 전처리·Feature Engineering·모델링 행동을 결정하는 과정입니다.</p>
</details>

<details>
Q2. 결측률이 5%면 항상 행을 삭제해도 된다. O/X

<p><strong>정답: X</strong><br />결측률은 참고값입니다. 결측이 특정 집단에 몰렸는지, 결측 자체가 신호인지, 삭제로 클래스 불균형이 심해지는지 확인해야 합니다.</p>
</details>

<details>
Q3. IQR 경계를 벗어나면 무조건 오류 데이터다. O/X

<p><strong>정답: X</strong><br />통계 규칙은 이상치 후보를 찾을 뿐입니다. 실제 사건인지 입력 오류인지 도메인 관점에서 판단해야 합니다.</p>
</details>

<details>
Q4. Standard Scaling을 하면 모든 값이 0과 1 사이가 된다. O/X

<p><strong>정답: X</strong><br />평균 0, 표준편차 1 규모로 맞추며 값의 범위는 제한되지 않습니다. 0~1 범위는 일반적인 Min-Max Scaling의 특징입니다.</p>
</details>

<details>
Q5. Test 데이터에도 scaler.fit_transform()을 따로 실행한다. O/X

<p><strong>정답: X</strong><br />Train에서 <code>fit</code>한 기준으로 Test에는 <code>transform</code>만 해야 합니다.</p>
</details>

<details>
Q6. 명목형 범주를 Label Encoding하면 가짜 순서가 생길 수 있다. O/X

<p><strong>정답: O</strong><br />서울=1, 부산=2, 제주=3이라는 숫자는 도시 사이의 실제 크기나 거리를 뜻하지 않습니다.</p>
</details>

<details>
Q7. 로그 변환은 극단값을 삭제한다. O/X

<p><strong>정답: X</strong><br />값을 삭제하는 것이 아니라 큰 값의 상대적 크기를 압축합니다.</p>
</details>

<details>
Q8. 정확도 99%면 불균형 분류 모델도 반드시 훌륭하다. O/X

<p><strong>정답: X</strong><br />소수 클래스가 1%라면 전부 다수 클래스로 예측해도 정확도 99%입니다. Recall, Precision, PR-AUC와 실제 비용을 확인해야 합니다.</p>
</details>

<details>
Q9. SMOTE는 Test 데이터에도 적용해 균형을 맞춰야 한다. O/X

<p><strong>정답: X</strong><br />Sampling은 Train에만 적용합니다. Test는 현실의 분포를 유지해야 공정하게 평가할 수 있습니다.</p>
</details>

<details>
Q10. 좋은 파생변수는 복잡한 수식에서 시작한다. O/X

<p><strong>정답: X</strong><br />좋은 파생변수는 “현재 변수로 표현하지 못한 중요한 정보는 무엇인가?”라는 질문에서 시작합니다.</p>
</details>

<h2 id="2-최종-정리">2. 최종 정리</h2>
<pre><code class="language-text">1. 비즈니스 문제와 예측 시점을 정의한다.
2. EDA로 구조, 품질, 분포, 관계를 탐색한다.
3. 관찰을 질문과 가설로 바꾼다.
4. 결측치와 이상치의 원인을 먼저 이해한다.
5. 모델에 맞게 Scaling과 Encoding을 적용한다.
6. 분포와 관계가 어렵다면 변수 변환을 검토한다.
7. 클래스 불균형은 Sampling과 적절한 지표로 다룬다.
8. 도메인 질문을 비율, 차이, 시간, 구간, 집계, 상호작용 Feature로 표현한다.
9. 모든 전처리는 Train에서 학습하고 운영에서도 동일하게 재현한다.
10. Feature의 가치는 교차검증과 실제 업무 효과로 확인한다.</code></pre>
<h2 id="3-오늘의-한-문장">3. 오늘의 한 문장</h2>
<blockquote>
<p>Feature Engineering은 데이터를 복잡하게 만드는 기술이 아니라, 현실의 문제를 모델이 이해할 수 있는 언어로 번역하는 과정이다.</p>
</blockquote>
<hr />
<h2 id="용어-미니-사전">용어 미니 사전</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 뜻</th>
</tr>
</thead>
<tbody><tr>
<td>Feature</td>
<td>모델이 판단에 사용하는 입력 단서</td>
</tr>
<tr>
<td>Target</td>
<td>모델이 맞히려는 정답</td>
</tr>
<tr>
<td>EDA</td>
<td>데이터를 탐색하며 질문과 다음 행동을 찾는 과정</td>
</tr>
<tr>
<td>Imputation</td>
<td>결측값을 다른 값으로 채우는 작업</td>
</tr>
<tr>
<td>Outlier</td>
<td>일반적인 범위에서 크게 벗어난 관측값</td>
</tr>
<tr>
<td>Scaling</td>
<td>변수들의 값 규모를 조정하는 작업</td>
</tr>
<tr>
<td>Encoding</td>
<td>범주를 모델이 계산할 수 있는 숫자로 표현하는 작업</td>
</tr>
<tr>
<td>Cardinality</td>
<td>범주형 변수의 서로 다른 값 개수</td>
</tr>
<tr>
<td>Data Leakage</td>
<td>예측 시 알 수 없는 정보가 학습에 들어가는 문제</td>
</tr>
<tr>
<td>Imbalanced Data</td>
<td>Target 클래스 비율이 크게 다른 데이터</td>
</tr>
<tr>
<td>Derived Feature</td>
<td>기존 변수로 계산해 만든 새 변수</td>
</tr>
<tr>
<td>Domain Knowledge</td>
<td>해당 업무·산업에 관한 지식</td>
</tr>
</tbody></table>
<h2 id="학습-후-스스로-답해볼-질문">학습 후 스스로 답해볼 질문</h2>
<ol>
<li>내가 다루는 데이터에서 결측은 왜 생기는가?</li>
<li>이상치는 제거해야 할 오류인가, 찾아야 할 사건인가?</li>
<li>어떤 변수에 Scaling이 필요하고, 어떤 모델에는 영향이 작은가?</li>
<li>범주가 매우 많을 때 One-Hot Encoding 대신 무엇을 할 수 있는가?</li>
<li>내 Feature는 예측 시점에 실제로 계산 가능한가?</li>
<li>원본 변수들이 아직 표현하지 못한 업무적 정보는 무엇인가?</li>
</ol>