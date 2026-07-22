<blockquote>
<p><strong>학습일:</strong> 2026.07.18
<strong>주제:</strong> Python Matplotlib &amp; Seaborn 기초부터 실무 활용까지</p>
</blockquote>
<hr />
<h1 id="데이터-시각화data-visualization란">데이터 시각화(Data Visualization)란?</h1>
<p>데이터 시각화(Data Visualization)는 <strong>숫자와 데이터를 그래프나 차트로 표현하여 쉽게 이해할 수 있도록 만드는 과정</strong>입니다.</p>
<p>데이터가 많아질수록 숫자만 보고는 의미를 파악하기 어렵습니다.</p>
<p>그래프를 사용하면 <strong>패턴, 추세, 이상치(Outlier), 관계</strong>를 한눈에 확인할 수 있습니다.</p>
<hr />
<h1 id="왜-데이터-시각화를-사용할까">왜 데이터 시각화를 사용할까?</h1>
<p>다음과 같은 월별 매출 데이터가 있다고 가정해 봅시다.</p>
<table>
<thead>
<tr>
<th>월</th>
<th align="right">매출(만원)</th>
</tr>
</thead>
<tbody><tr>
<td>1월</td>
<td align="right">120</td>
</tr>
<tr>
<td>2월</td>
<td align="right">135</td>
</tr>
<tr>
<td>3월</td>
<td align="right">160</td>
</tr>
<tr>
<td>4월</td>
<td align="right">210</td>
</tr>
<tr>
<td>5월</td>
<td align="right">240</td>
</tr>
</tbody></table>
<p>숫자만 보면 증가하는지 바로 알기 어렵습니다.</p>
<p>하지만 선 그래프로 표현하면 <strong>매출이 꾸준히 증가하고 있다는 사실</strong>을 즉시 알 수 있습니다.</p>
<hr />
<h2 id="쉽게-설명">쉽게 설명</h2>
<blockquote>
<p><strong>표(Table)는 책이다.</strong></p>
<p><strong>그래프(Graph)는 사진이다.</strong></p>
</blockquote>
<p>책은 자세하게 설명해 주지만 읽는 데 시간이 걸립니다.</p>
<p>사진은 보는 순간 내용을 이해할 수 있습니다.</p>
<p>그래프도 마찬가지입니다.</p>
<hr />
<h1 id="matplotlib란">Matplotlib란?</h1>
<p>Matplotlib는 <strong>Python에서 가장 많이 사용하는 그래프 라이브러리</strong>입니다.</p>
<p>거의 모든 데이터 시각화의 기본이 되는 라이브러리이며,</p>
<p>Seaborn 역시 내부적으로 Matplotlib를 기반으로 만들어졌습니다.</p>
<hr />
<h1 id="seaborn이란">Seaborn이란?</h1>
<p>Seaborn은 <strong>Matplotlib를 더욱 보기 좋고 쉽게 사용할 수 있도록 만든 라이브러리</strong>입니다.</p>
<p>기본 디자인이 깔끔하고,</p>
<p>통계 그래프를 쉽게 그릴 수 있습니다.</p>
<hr />
<h1 id="matplotlib와-seaborn-비교">Matplotlib와 Seaborn 비교</h1>
<table>
<thead>
<tr>
<th>항목</th>
<th>Matplotlib</th>
<th>Seaborn</th>
</tr>
</thead>
<tbody><tr>
<td>난이도</td>
<td>조금 높음</td>
<td>쉬움</td>
</tr>
<tr>
<td>디자인</td>
<td>기본 스타일</td>
<td>세련된 스타일</td>
</tr>
<tr>
<td>세부 설정</td>
<td>매우 자유로움</td>
<td>간단함</td>
</tr>
<tr>
<td>통계 그래프</td>
<td>직접 구현</td>
<td>기본 제공</td>
</tr>
<tr>
<td>실무 사용</td>
<td>★★★★★</td>
<td>★★★★★</td>
</tr>
</tbody></table>
<hr />
<h1 id="설치">설치</h1>
<pre><code class="language-bash">pip install matplotlib seaborn</code></pre>
<hr />
<h1 id="라이브러리-불러오기">라이브러리 불러오기</h1>
<pre><code class="language-python">import matplotlib.pyplot as plt
import seaborn as sns</code></pre>
<hr />
<h1 id="가장-기본적인-그래프">가장 기본적인 그래프</h1>
<h2 id="선-그래프line-plot">선 그래프(Line Plot)</h2>
<p>가장 많이 사용하는 그래프입니다.</p>
<pre><code class="language-python">import matplotlib.pyplot as plt

month = [1,2,3,4,5]
sales = [120,135,160,210,240]

plt.plot(month, sales)

plt.show()</code></pre>
<hr />
<h2 id="언제-사용할까">언제 사용할까?</h2>
<ul>
<li>월별 매출</li>
<li>주가 변화</li>
<li>기온 변화</li>
<li>방문자 수 변화</li>
</ul>
<p>처럼 <strong>시간에 따른 변화</strong>를 볼 때 사용합니다.</p>
<hr />
<h2 id="비유">비유</h2>
<p>심전도 그래프처럼</p>
<p>시간이 흐르면서 값이 어떻게 변하는지 보여줍니다.</p>
<hr />
<h1 id="제목-넣기">제목 넣기</h1>
<pre><code class="language-python">plt.title(&quot;Monthly Sales&quot;)</code></pre>
<hr />
<h1 id="축-이름-넣기">축 이름 넣기</h1>
<pre><code class="language-python">plt.xlabel(&quot;Month&quot;)

plt.ylabel(&quot;Sales&quot;)</code></pre>
<hr />
<h1 id="색상-변경">색상 변경</h1>
<pre><code class="language-python">plt.plot(month, sales, color=&quot;red&quot;)</code></pre>
<hr />
<h1 id="선-스타일">선 스타일</h1>
<pre><code class="language-python">plt.plot(month, sales, linestyle=&quot;--&quot;)</code></pre>
<hr />
<h1 id="마커-추가">마커 추가</h1>
<pre><code class="language-python">plt.plot(month, sales, marker=&quot;o&quot;)</code></pre>
<hr />
<h1 id="막대-그래프bar-chart">막대 그래프(Bar Chart)</h1>
<pre><code class="language-python">name = [&quot;A&quot;,&quot;B&quot;,&quot;C&quot;,&quot;D&quot;]

score = [80,95,70,90]

plt.bar(name, score)

plt.show()</code></pre>
<hr />
<h2 id="언제-사용할까-1">언제 사용할까?</h2>
<ul>
<li>상품별 판매량</li>
<li>부서별 매출</li>
<li>학생 성적 비교</li>
</ul>
<hr />
<h2 id="비유-1">비유</h2>
<p>막대그래프는</p>
<blockquote>
<p>&quot;누가 더 큰가?&quot;</p>
</blockquote>
<p>를 비교하는 그래프입니다.</p>
<hr />
<h1 id="가로-막대-그래프">가로 막대 그래프</h1>
<pre><code class="language-python">plt.barh(name, score)</code></pre>
<hr />
<h1 id="히스토그램histogram">히스토그램(Histogram)</h1>
<p>데이터 분포를 확인합니다.</p>
<pre><code class="language-python">import numpy as np

data = np.random.randn(1000)

plt.hist(data)

plt.show()</code></pre>
<hr />
<h2 id="언제-사용할까-2">언제 사용할까?</h2>
<ul>
<li>시험 점수 분포</li>
<li>고객 연령 분포</li>
<li>월급 분포</li>
</ul>
<hr />
<h2 id="비유-2">비유</h2>
<p>사람들이 어느 구간에 가장 많이 모여 있는지 보는 그래프입니다.</p>
<hr />
<h1 id="산점도scatter-plot">산점도(Scatter Plot)</h1>
<pre><code class="language-python">x = [1,2,3,4,5]

y = [2,3,5,4,6]

plt.scatter(x,y)

plt.show()</code></pre>
<hr />
<h2 id="언제-사용할까-3">언제 사용할까?</h2>
<ul>
<li>공부 시간과 성적</li>
<li>키와 몸무게</li>
<li>광고비와 매출</li>
</ul>
<hr />
<h2 id="비유-3">비유</h2>
<p>지도 위에 점을 찍어</p>
<p>패턴을 찾는 것과 같습니다.</p>
<hr />
<h1 id="원-그래프pie-chart">원 그래프(Pie Chart)</h1>
<pre><code class="language-python">label = [&quot;A&quot;,&quot;B&quot;,&quot;C&quot;]

value = [30,45,25]

plt.pie(value, labels=label)

plt.show()</code></pre>
<hr />
<h2 id="언제-사용할까-4">언제 사용할까?</h2>
<ul>
<li>시장 점유율</li>
<li>예산 비율</li>
<li>고객 비율</li>
</ul>
<hr />
<h2 id="주의">주의</h2>
<p>항목이 너무 많으면</p>
<p>원 그래프보다 막대그래프가 더 좋습니다.</p>
<hr />
<h1 id="여러-그래프-그리기">여러 그래프 그리기</h1>
<pre><code class="language-python">plt.figure(figsize=(8,5))

plt.plot(month,sales)

plt.show()</code></pre>
<hr />
<h1 id="그래프-저장하기">그래프 저장하기</h1>
<pre><code class="language-python">plt.savefig(&quot;sales.png&quot;)</code></pre>
<hr />
<h1 id="seaborn-시작하기">Seaborn 시작하기</h1>
<p>기본 스타일 적용</p>
<pre><code class="language-python">import seaborn as sns

sns.set_theme()</code></pre>
<hr />
<h1 id="seaborn-선-그래프">Seaborn 선 그래프</h1>
<pre><code class="language-python">sns.lineplot(x=month, y=sales)</code></pre>
<hr />
<h1 id="seaborn-막대그래프">Seaborn 막대그래프</h1>
<pre><code class="language-python">import pandas as pd

df = pd.DataFrame({
    &quot;직업&quot;:[&quot;개발&quot;,&quot;기획&quot;,&quot;디자인&quot;],
    &quot;연봉&quot;:[6000,5000,4500]
})

sns.barplot(data=df, x=&quot;직업&quot;, y=&quot;연봉&quot;)</code></pre>
<hr />
<h1 id="seaborn-히스토그램">Seaborn 히스토그램</h1>
<pre><code class="language-python">sns.histplot(data)</code></pre>
<hr />
<h1 id="box-plot">Box Plot</h1>
<p>가장 많이 사용하는 통계 그래프입니다.</p>
<pre><code class="language-python">sns.boxplot(data=data)</code></pre>
<hr />
<h2 id="언제-사용할까-5">언제 사용할까?</h2>
<ul>
<li>이상치 탐지</li>
<li>데이터 분포 확인</li>
<li>중앙값 확인</li>
</ul>
<hr />
<h2 id="쉽게-설명-1">쉽게 설명</h2>
<p>시험 점수를 분석한다고 생각해 봅시다.</p>
<p>대부분 학생은 60~90점 사이입니다.</p>
<p>그런데</p>
<p>한 학생만 5점을 받았습니다.</p>
<p>이 학생을 <strong>이상치(Outlier)</strong> 라고 합니다.</p>
<p>Box Plot은</p>
<p>이러한 이상치를 쉽게 찾습니다.</p>
<hr />
<h1 id="box-plot-구성">Box Plot 구성</h1>
<table>
<thead>
<tr>
<th>구성</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>최솟값</td>
<td>가장 작은 값</td>
</tr>
<tr>
<td>Q1</td>
<td>25%</td>
</tr>
<tr>
<td>Median</td>
<td>중앙값</td>
</tr>
<tr>
<td>Q3</td>
<td>75%</td>
</tr>
<tr>
<td>최댓값</td>
<td>가장 큰 값</td>
</tr>
<tr>
<td>점</td>
<td>이상치</td>
</tr>
</tbody></table>
<hr />
<h1 id="heatmap">Heatmap</h1>
<p>상관관계를 보기 위한 대표 그래프입니다.</p>
<pre><code class="language-python">import pandas as pd

corr = df.corr(numeric_only=True)

sns.heatmap(corr, annot=True)</code></pre>
<hr />
<h2 id="언제-사용할까-6">언제 사용할까?</h2>
<ul>
<li>변수 관계 분석</li>
<li>머신러닝 전처리</li>
<li>다중공선성 확인</li>
</ul>
<hr />
<h2 id="비유-4">비유</h2>
<p>친한 친구끼리는 가까운 색</p>
<p>관련 없는 친구는 먼 색</p>
<p>처럼 표현하는 그래프입니다.</p>
<hr />
<h1 id="pair-plot">Pair Plot</h1>
<pre><code class="language-python">sns.pairplot(df)</code></pre>
<hr />
<h2 id="언제-사용할까-7">언제 사용할까?</h2>
<p>모든 변수의 관계를 한 번에 확인할 때</p>
<hr />
<h1 id="실무-활용">실무 활용</h1>
<h2 id="사례-1-쇼핑몰-매출-분석">사례 1. 쇼핑몰 매출 분석</h2>
<pre><code class="language-python">import pandas as pd
import matplotlib.pyplot as plt

sales = pd.read_csv(&quot;sales.csv&quot;)

plt.plot(sales[&quot;Month&quot;], sales[&quot;Sales&quot;])

plt.title(&quot;Monthly Sales&quot;)

plt.show()</code></pre>
<h3 id="왜-사용할까">왜 사용할까?</h3>
<ul>
<li>월별 매출 변화 확인</li>
<li>성수기 분석</li>
<li>매출 추세 확인</li>
</ul>
<hr />
<h2 id="사례-2-마케팅-광고-분석">사례 2. 마케팅 광고 분석</h2>
<pre><code class="language-python">sns.scatterplot(
    data=df,
    x=&quot;광고비&quot;,
    y=&quot;매출&quot;
)</code></pre>
<h3 id="왜-사용할까-1">왜 사용할까?</h3>
<p>광고비가 증가하면</p>
<p>매출도 증가하는지 확인합니다.</p>
<hr />
<h2 id="사례-3-인사hr-데이터">사례 3. 인사(HR) 데이터</h2>
<pre><code class="language-python">sns.boxplot(
    data=df,
    y=&quot;연봉&quot;
)</code></pre>
<h3 id="왜-사용할까-2">왜 사용할까?</h3>
<ul>
<li>이상 연봉 확인</li>
<li>연봉 분포 확인</li>
</ul>
<hr />
<h1 id="언제-사용하는가">언제 사용하는가?</h1>
<table>
<thead>
<tr>
<th>상황</th>
<th>추천 그래프</th>
</tr>
</thead>
<tbody><tr>
<td>시간 변화</td>
<td>선 그래프</td>
</tr>
<tr>
<td>항목 비교</td>
<td>막대그래프</td>
</tr>
<tr>
<td>분포 확인</td>
<td>히스토그램</td>
</tr>
<tr>
<td>비율 확인</td>
<td>원 그래프</td>
</tr>
<tr>
<td>관계 분석</td>
<td>산점도</td>
</tr>
<tr>
<td>이상치 확인</td>
<td>Box Plot</td>
</tr>
<tr>
<td>상관관계</td>
<td>Heatmap</td>
</tr>
</tbody></table>
<hr />
<h1 id="왜-사용하는가">왜 사용하는가?</h1>
<table>
<thead>
<tr>
<th>이유</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>빠른 이해</td>
<td>숫자보다 그래프가 이해하기 쉽다.</td>
</tr>
<tr>
<td>패턴 발견</td>
<td>추세와 변화 확인</td>
</tr>
<tr>
<td>이상치 탐지</td>
<td>특이한 데이터 확인</td>
</tr>
<tr>
<td>보고서 작성</td>
<td>시각적으로 전달력이 높음</td>
</tr>
<tr>
<td>의사결정 지원</td>
<td>데이터 기반 판단 가능</td>
</tr>
</tbody></table>
<hr />
<h1 id="주의할-점">주의할 점</h1>
<table>
<thead>
<tr>
<th>주의사항</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>그래프 선택</td>
<td>데이터 특성에 맞는 그래프를 선택해야 한다.</td>
</tr>
<tr>
<td>색상 남용 금지</td>
<td>너무 많은 색은 오히려 가독성을 떨어뜨린다.</td>
</tr>
<tr>
<td>축(Label) 표시</td>
<td>축 이름과 제목을 반드시 작성한다.</td>
</tr>
<tr>
<td>왜곡 금지</td>
<td>축 범위를 임의로 조정하면 잘못된 해석을 유도할 수 있다.</td>
</tr>
<tr>
<td>3D 그래프 지양</td>
<td>정보 전달보다 시각 효과가 커질 수 있으므로 필요한 경우만 사용한다.</td>
</tr>
</tbody></table>
<hr />
<h1 id="matplotlib와-seaborn-비교-1">Matplotlib와 Seaborn 비교</h1>
<table>
<thead>
<tr>
<th>항목</th>
<th>Matplotlib</th>
<th>Seaborn</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>기본 그래프</td>
<td>통계 그래프</td>
</tr>
<tr>
<td>사용 난이도</td>
<td>중간</td>
<td>쉬움</td>
</tr>
<tr>
<td>디자인</td>
<td>기본 스타일</td>
<td>세련된 스타일</td>
</tr>
<tr>
<td>커스터마이징</td>
<td>매우 자유로움</td>
<td>상대적으로 제한적</td>
</tr>
<tr>
<td>대표 사용처</td>
<td>보고서, 논문</td>
<td>EDA, 데이터 분석</td>
</tr>
</tbody></table>
<hr />
<h1 id="자주-사용하는-그래프-정리">자주 사용하는 그래프 정리</h1>
<table>
<thead>
<tr>
<th>그래프</th>
<th>언제 사용하는가</th>
<th>대표 함수</th>
</tr>
</thead>
<tbody><tr>
<td>선 그래프</td>
<td>시간 변화</td>
<td><code>plot()</code></td>
</tr>
<tr>
<td>막대 그래프</td>
<td>항목 비교</td>
<td><code>bar()</code></td>
</tr>
<tr>
<td>히스토그램</td>
<td>분포 확인</td>
<td><code>hist()</code></td>
</tr>
<tr>
<td>산점도</td>
<td>상관관계</td>
<td><code>scatter()</code></td>
</tr>
<tr>
<td>원 그래프</td>
<td>비율 비교</td>
<td><code>pie()</code></td>
</tr>
<tr>
<td>Box Plot</td>
<td>이상치 확인</td>
<td><code>boxplot()</code></td>
</tr>
<tr>
<td>Heatmap</td>
<td>상관계수</td>
<td><code>heatmap()</code></td>
</tr>
<tr>
<td>Pair Plot</td>
<td>전체 변수 관계</td>
<td><code>pairplot()</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="핵심-정리">핵심 정리</h1>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Matplotlib</td>
<td>Python의 기본 시각화 라이브러리</td>
</tr>
<tr>
<td>Seaborn</td>
<td>Matplotlib 기반의 통계 시각화 라이브러리</td>
</tr>
<tr>
<td>선 그래프</td>
<td>시간에 따른 변화</td>
</tr>
<tr>
<td>막대 그래프</td>
<td>항목 간 비교</td>
</tr>
<tr>
<td>히스토그램</td>
<td>데이터 분포 확인</td>
</tr>
<tr>
<td>산점도</td>
<td>두 변수의 관계 확인</td>
</tr>
<tr>
<td>Box Plot</td>
<td>이상치 및 사분위수 확인</td>
</tr>
<tr>
<td>Heatmap</td>
<td>상관관계 시각화</td>
</tr>
<tr>
<td>Pair Plot</td>
<td>모든 변수 간 관계 확인</td>
</tr>
</tbody></table>
<hr />
<h1 id="한-줄-요약">한 줄 요약</h1>
<blockquote>
<p><strong>Matplotlib와 Seaborn은 데이터를 그래프로 시각화하여 숫자로는 발견하기 어려운 패턴, 추세, 이상치, 변수 간 관계를 쉽게 이해하도록 도와주는 핵심 라이브러리이다.</strong></p>
</blockquote>