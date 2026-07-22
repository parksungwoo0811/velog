<blockquote>
<p><strong>학습일:</strong> 2026.07.18
<strong>주제:</strong> Python Pandas 기초부터 실무 활용까지</p>
</blockquote>
<hr />
<h1 id="pandas란">Pandas란?</h1>
<p>Pandas는 <strong>데이터를 표(Table) 형태로 쉽고 빠르게 다룰 수 있도록 만들어진 Python 라이브러리</strong>입니다.</p>
<p>데이터 분석에서는 CSV, Excel, 데이터베이스(DB) 등 대부분의 데이터를 표 형태로 다루기 때문에 Pandas는 거의 필수적으로 사용됩니다.</p>
<p>NumPy가 <strong>숫자 계산</strong>에 특화되어 있다면,</p>
<p>Pandas는 <strong>데이터 관리와 분석</strong>에 특화되어 있습니다.</p>
<hr />
<h1 id="왜-pandas를-사용할까">왜 Pandas를 사용할까?</h1>
<p>실무에서 다루는 데이터는 대부분 아래와 같은 형태입니다.</p>
<table>
<thead>
<tr>
<th>이름</th>
<th align="right">나이</th>
<th>직업</th>
</tr>
</thead>
<tbody><tr>
<td>김철수</td>
<td align="right">25</td>
<td>개발자</td>
</tr>
<tr>
<td>이영희</td>
<td align="right">30</td>
<td>디자이너</td>
</tr>
<tr>
<td>박민수</td>
<td align="right">28</td>
<td>기획자</td>
</tr>
</tbody></table>
<p>Python의 리스트만으로 이러한 데이터를 다루려면 매우 복잡합니다.</p>
<p>Pandas는 엑셀처럼 데이터를 다룰 수 있도록 만들어졌습니다.</p>
<h2 id="쉽게-설명">쉽게 설명</h2>
<blockquote>
<p><strong>Python List = 메모장</strong></p>
<p><strong>NumPy = 계산기</strong></p>
<p><strong>Pandas = Excel</strong></p>
</blockquote>
<p>즉,</p>
<ul>
<li>데이터를 보기 쉽고</li>
<li>수정하기 쉽고</li>
<li>분석하기 쉽도록 만들어 줍니다.</li>
</ul>
<hr />
<h1 id="비유">비유</h1>
<p>회사에서 직원 명단을 관리한다고 생각해봅시다.</p>
<h3 id="python-list">Python List</h3>
<p>직원 정보를 하나씩 찾아야 합니다.</p>
<h3 id="pandas">Pandas</h3>
<p>엑셀에서 필터를 누르거나 정렬하는 것처럼 한 줄의 코드만으로 데이터를 관리할 수 있습니다.</p>
<hr />
<h1 id="pandas-설치">Pandas 설치</h1>
<pre><code class="language-bash">pip install pandas</code></pre>
<hr />
<h1 id="pandas-불러오기">Pandas 불러오기</h1>
<pre><code class="language-python">import pandas as pd</code></pre>
<p>대부분의 예제에서는 <code>pd</code>라는 별칭을 사용합니다.</p>
<hr />
<h1 id="pandas의-핵심-자료구조">Pandas의 핵심 자료구조</h1>
<p>Pandas에는 대표적으로 두 가지 자료구조가 있습니다.</p>
<table>
<thead>
<tr>
<th>자료구조</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Series</td>
<td>1차원 데이터</td>
</tr>
<tr>
<td>DataFrame</td>
<td>2차원 표 형태 데이터</td>
</tr>
</tbody></table>
<hr />
<h1 id="series">Series</h1>
<p>Series는 <strong>한 개의 열(Column)</strong> 을 의미합니다.</p>
<p>예를 들어 학생들의 점수만 저장하는 경우입니다.</p>
<pre><code class="language-python">import pandas as pd

score = pd.Series([90, 85, 100, 70])

print(score)</code></pre>
<p>출력</p>
<pre><code class="language-text">0     90
1     85
2    100
3     70
dtype: int64</code></pre>
<hr />
<h2 id="언제-사용할까">언제 사용할까?</h2>
<ul>
<li>시험 점수</li>
<li>월별 매출</li>
<li>하루 방문자 수</li>
</ul>
<p>처럼 하나의 데이터만 저장할 때 사용합니다.</p>
<hr />
<h1 id="dataframe">DataFrame</h1>
<p>가장 많이 사용하는 자료구조입니다.</p>
<p>엑셀처럼 행(Row)과 열(Column)로 구성됩니다.</p>
<pre><code class="language-python">import pandas as pd

df = pd.DataFrame({
    &quot;이름&quot;: [&quot;김철수&quot;, &quot;이영희&quot;, &quot;박민수&quot;],
    &quot;나이&quot;: [25, 30, 28],
    &quot;직업&quot;: [&quot;개발자&quot;, &quot;디자이너&quot;, &quot;기획자&quot;]
})

print(df)</code></pre>
<p>출력</p>
<pre><code class="language-text">     이름  나이    직업
0  김철수  25   개발자
1  이영희  30  디자이너
2  박민수  28   기획자</code></pre>
<hr />
<h1 id="dataframe-구조">DataFrame 구조</h1>
<table>
<thead>
<tr>
<th>용어</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Row</td>
<td>행</td>
</tr>
<tr>
<td>Column</td>
<td>열</td>
</tr>
<tr>
<td>Index</td>
<td>행 번호</td>
</tr>
</tbody></table>
<hr />
<h1 id="csv-파일-불러오기">CSV 파일 불러오기</h1>
<p>실무에서는 대부분 CSV 파일을 사용합니다.</p>
<pre><code class="language-python">import pandas as pd

df = pd.read_csv(&quot;sales.csv&quot;)</code></pre>
<hr />
<h1 id="excel-파일-불러오기">Excel 파일 불러오기</h1>
<pre><code class="language-python">df = pd.read_excel(&quot;sales.xlsx&quot;)</code></pre>
<hr />
<h1 id="데이터-저장하기">데이터 저장하기</h1>
<p>CSV 저장</p>
<pre><code class="language-python">df.to_csv(&quot;result.csv&quot;, index=False)</code></pre>
<p>Excel 저장</p>
<pre><code class="language-python">df.to_excel(&quot;result.xlsx&quot;, index=False)</code></pre>
<hr />
<h1 id="데이터-확인하기">데이터 확인하기</h1>
<h2 id="상위-데이터">상위 데이터</h2>
<pre><code class="language-python">df.head()</code></pre>
<p>기본적으로 5개를 보여줍니다.</p>
<pre><code class="language-python">df.head(10)</code></pre>
<hr />
<h2 id="하위-데이터">하위 데이터</h2>
<pre><code class="language-python">df.tail()</code></pre>
<hr />
<h2 id="데이터-정보">데이터 정보</h2>
<pre><code class="language-python">df.info()</code></pre>
<p>출력 예시</p>
<pre><code class="language-text">&lt;class 'pandas.core.frame.DataFrame'&gt;
RangeIndex: 100 entries
Data columns (total 4 columns)</code></pre>
<p>확인 가능한 내용</p>
<ul>
<li>행 개수</li>
<li>열 개수</li>
<li>자료형</li>
<li>결측치 여부</li>
</ul>
<hr />
<h1 id="기초-통계-확인">기초 통계 확인</h1>
<pre><code class="language-python">df.describe()</code></pre>
<p>결과</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>count</td>
<td>데이터 개수</td>
</tr>
<tr>
<td>mean</td>
<td>평균</td>
</tr>
<tr>
<td>std</td>
<td>표준편차</td>
</tr>
<tr>
<td>min</td>
<td>최솟값</td>
</tr>
<tr>
<td>max</td>
<td>최댓값</td>
</tr>
</tbody></table>
<hr />
<h1 id="컬럼-선택하기">컬럼 선택하기</h1>
<pre><code class="language-python">df[&quot;나이&quot;]</code></pre>
<p>여러 개 선택</p>
<pre><code class="language-python">df[[&quot;이름&quot;,&quot;나이&quot;]]</code></pre>
<hr />
<h1 id="행-선택하기">행 선택하기</h1>
<h3 id="iloc">iloc</h3>
<p>숫자로 선택</p>
<pre><code class="language-python">df.iloc[0]</code></pre>
<p>첫 번째 행</p>
<hr />
<h3 id="loc">loc</h3>
<p>이름으로 선택</p>
<pre><code class="language-python">df.loc[0]</code></pre>
<hr />
<h1 id="조건-검색">조건 검색</h1>
<p>30세 이상만 보기</p>
<pre><code class="language-python">df[df[&quot;나이&quot;] &gt;= 30]</code></pre>
<p>직업이 개발자인 사람</p>
<pre><code class="language-python">df[df[&quot;직업&quot;] == &quot;개발자&quot;]</code></pre>
<hr />
<h1 id="여러-조건-검색">여러 조건 검색</h1>
<pre><code class="language-python">df[
    (df[&quot;나이&quot;] &gt;= 30) &amp;
    (df[&quot;직업&quot;] == &quot;개발자&quot;)
]</code></pre>
<p>OR 조건</p>
<pre><code class="language-python">df[
    (df[&quot;나이&quot;] &gt;= 30) |
    (df[&quot;직업&quot;] == &quot;디자이너&quot;)
]</code></pre>
<hr />
<h1 id="새로운-컬럼-만들기">새로운 컬럼 만들기</h1>
<pre><code class="language-python">df[&quot;연봉&quot;] = [4000, 5000, 4500]</code></pre>
<p>기존 컬럼으로 생성</p>
<pre><code class="language-python">df[&quot;나이+10&quot;] = df[&quot;나이&quot;] + 10</code></pre>
<hr />
<h1 id="컬럼-삭제">컬럼 삭제</h1>
<pre><code class="language-python">df.drop(columns=[&quot;연봉&quot;])</code></pre>
<hr />
<h1 id="결측치missing-value">결측치(Missing Value)</h1>
<p>결측치란?</p>
<p>비어 있는 데이터입니다.</p>
<p>예시</p>
<table>
<thead>
<tr>
<th>이름</th>
<th>나이</th>
</tr>
</thead>
<tbody><tr>
<td>김철수</td>
<td>25</td>
</tr>
<tr>
<td>이영희</td>
<td>NaN</td>
</tr>
</tbody></table>
<hr />
<h2 id="결측치-확인">결측치 확인</h2>
<pre><code class="language-python">df.isnull()</code></pre>
<p>개수 확인</p>
<pre><code class="language-python">df.isnull().sum()</code></pre>
<hr />
<h2 id="결측치-제거">결측치 제거</h2>
<pre><code class="language-python">df.dropna()</code></pre>
<hr />
<h2 id="결측치-채우기">결측치 채우기</h2>
<pre><code class="language-python">df.fillna(0)</code></pre>
<p>평균으로 채우기</p>
<pre><code class="language-python">df[&quot;나이&quot;].fillna(df[&quot;나이&quot;].mean())</code></pre>
<hr />
<h1 id="정렬하기">정렬하기</h1>
<p>나이순</p>
<pre><code class="language-python">df.sort_values(&quot;나이&quot;)</code></pre>
<p>내림차순</p>
<pre><code class="language-python">df.sort_values(&quot;나이&quot;, ascending=False)</code></pre>
<hr />
<h1 id="그룹화groupby">그룹화(GroupBy)</h1>
<p>가장 많이 사용하는 기능입니다.</p>
<pre><code class="language-python">df.groupby(&quot;직업&quot;)[&quot;연봉&quot;].mean()</code></pre>
<p>결과</p>
<table>
<thead>
<tr>
<th>직업</th>
<th align="right">평균 연봉</th>
</tr>
</thead>
<tbody><tr>
<td>개발자</td>
<td align="right">5000</td>
</tr>
<tr>
<td>기획자</td>
<td align="right">4200</td>
</tr>
<tr>
<td>디자이너</td>
<td align="right">4700</td>
</tr>
</tbody></table>
<hr />
<h2 id="쉽게-설명-1">쉽게 설명</h2>
<p>학교 성적을 반별 평균으로 계산하는 것과 같습니다.</p>
<p>학생 개개인이 아니라</p>
<p>반 단위로 묶어서 계산하는 것입니다.</p>
<hr />
<h1 id="데이터-개수-세기">데이터 개수 세기</h1>
<pre><code class="language-python">df[&quot;직업&quot;].value_counts()</code></pre>
<p>결과</p>
<pre><code class="language-text">개발자      15
기획자      10
디자이너     8</code></pre>
<hr />
<h1 id="중복-제거">중복 제거</h1>
<pre><code class="language-python">df.drop_duplicates()</code></pre>
<hr />
<h1 id="데이터-타입-변경">데이터 타입 변경</h1>
<pre><code class="language-python">df[&quot;나이&quot;] = df[&quot;나이&quot;].astype(int)</code></pre>
<hr />
<h1 id="실무-활용">실무 활용</h1>
<h2 id="사례-1-쇼핑몰-주문-데이터-분석">사례 1. 쇼핑몰 주문 데이터 분석</h2>
<pre><code class="language-python">import pandas as pd

sales = pd.read_csv(&quot;sales.csv&quot;)

print(sales.head())

print(sales[&quot;금액&quot;].mean())

print(sales[sales[&quot;금액&quot;] &gt;= 100000])</code></pre>
<h3 id="언제-사용할까-1">언제 사용할까?</h3>
<ul>
<li>하루 매출 분석</li>
<li>VIP 고객 분석</li>
<li>상품별 판매량 분석</li>
</ul>
<hr />
<h2 id="사례-2-인사hr-데이터-분석">사례 2. 인사(HR) 데이터 분석</h2>
<pre><code class="language-python">employee = pd.read_csv(&quot;employee.csv&quot;)

employee.groupby(&quot;부서&quot;)[&quot;연봉&quot;].mean()</code></pre>
<h3 id="언제-사용할까-2">언제 사용할까?</h3>
<ul>
<li>부서별 평균 연봉</li>
<li>부서별 직원 수</li>
<li>퇴사율 분석</li>
</ul>
<hr />
<h2 id="사례-3-학생-성적-분석">사례 3. 학생 성적 분석</h2>
<pre><code class="language-python">score = pd.read_csv(&quot;score.csv&quot;)

print(score.describe())

print(score.sort_values(&quot;수학&quot;))</code></pre>
<h3 id="언제-사용할까-3">언제 사용할까?</h3>
<ul>
<li>과목별 평균</li>
<li>상위권 학생 분석</li>
<li>시험 결과 리포트 작성</li>
</ul>
<hr />
<h1 id="언제-사용하는가">언제 사용하는가?</h1>
<table>
<thead>
<tr>
<th>상황</th>
<th>활용</th>
</tr>
</thead>
<tbody><tr>
<td>CSV 분석</td>
<td>데이터 불러오기</td>
</tr>
<tr>
<td>Excel 분석</td>
<td>엑셀 자동화</td>
</tr>
<tr>
<td>데이터 전처리</td>
<td>결측치 처리</td>
</tr>
<tr>
<td>데이터 정렬</td>
<td>순위 분석</td>
</tr>
<tr>
<td>데이터 집계</td>
<td>GroupBy</td>
</tr>
<tr>
<td>보고서 작성</td>
<td>통계 요약</td>
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
<td>표 형태 데이터 관리</td>
<td>엑셀처럼 쉽게 다룰 수 있음</td>
</tr>
<tr>
<td>데이터 전처리</td>
<td>결측치, 중복 제거가 쉬움</td>
</tr>
<tr>
<td>빠른 분석</td>
<td>몇 줄의 코드로 통계 계산 가능</td>
</tr>
<tr>
<td>다양한 파일 지원</td>
<td>CSV, Excel, SQL 등과 연동 가능</td>
</tr>
<tr>
<td>AI 전처리</td>
<td>머신러닝 입력 데이터 준비에 필수</td>
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
<td>결측치 확인</td>
<td>분석 전에 반드시 <code>isnull()</code>로 확인합니다.</td>
</tr>
<tr>
<td>자료형 확인</td>
<td>숫자가 문자열(String)로 저장될 수 있으므로 <code>info()</code>를 확인합니다.</td>
</tr>
<tr>
<td>원본 변경 여부</td>
<td><code>drop()</code>, <code>fillna()</code> 등은 기본적으로 원본을 변경하지 않습니다. 필요하면 <code>inplace=True</code> 또는 재할당을 사용합니다.</td>
</tr>
<tr>
<td>인덱스 관리</td>
<td>필터링 후 인덱스가 유지되므로 필요 시 <code>reset_index(drop=True)</code>를 사용합니다.</td>
</tr>
</tbody></table>
<hr />
<h1 id="numpy와-pandas-비교">NumPy와 Pandas 비교</h1>
<table>
<thead>
<tr>
<th>항목</th>
<th>NumPy</th>
<th>Pandas</th>
</tr>
</thead>
<tbody><tr>
<td>주요 목적</td>
<td>숫자 계산</td>
<td>데이터 분석</td>
</tr>
<tr>
<td>자료구조</td>
<td>Array</td>
<td>Series, DataFrame</td>
</tr>
<tr>
<td>데이터 형태</td>
<td>행렬</td>
<td>표(Table)</td>
</tr>
<tr>
<td>속도</td>
<td>매우 빠름</td>
<td>분석에 최적화</td>
</tr>
<tr>
<td>대표 활용</td>
<td>수학 연산, AI</td>
<td>전처리, 분석, 집계</td>
</tr>
</tbody></table>
<hr />
<h1 id="핵심-정리">핵심 정리</h1>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
<th>대표 함수</th>
</tr>
</thead>
<tbody><tr>
<td>Series</td>
<td>1차원 데이터</td>
<td><code>pd.Series()</code></td>
</tr>
<tr>
<td>DataFrame</td>
<td>2차원 표</td>
<td><code>pd.DataFrame()</code></td>
</tr>
<tr>
<td>데이터 불러오기</td>
<td>CSV, Excel</td>
<td><code>read_csv()</code>, <code>read_excel()</code></td>
</tr>
<tr>
<td>데이터 확인</td>
<td>상위 데이터</td>
<td><code>head()</code></td>
</tr>
<tr>
<td>정보 확인</td>
<td>자료형, 결측치</td>
<td><code>info()</code></td>
</tr>
<tr>
<td>통계 확인</td>
<td>기초 통계</td>
<td><code>describe()</code></td>
</tr>
<tr>
<td>조건 검색</td>
<td>필터링</td>
<td><code>loc</code>, <code>iloc</code></td>
</tr>
<tr>
<td>결측치 처리</td>
<td>삭제 및 채우기</td>
<td><code>dropna()</code>, <code>fillna()</code></td>
</tr>
<tr>
<td>정렬</td>
<td>오름차순, 내림차순</td>
<td><code>sort_values()</code></td>
</tr>
<tr>
<td>그룹 분석</td>
<td>그룹별 통계</td>
<td><code>groupby()</code></td>
</tr>
<tr>
<td>빈도 계산</td>
<td>개수 세기</td>
<td><code>value_counts()</code></td>
</tr>
<tr>
<td>중복 제거</td>
<td>중복 데이터 삭제</td>
<td><code>drop_duplicates()</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="한-줄-요약">한 줄 요약</h1>
<blockquote>
<p><strong>Pandas는 표 형태의 데이터를 쉽고 효율적으로 관리·가공·분석하기 위한 핵심 라이브러리이며, 데이터 분석과 머신러닝 전처리 과정에서 가장 많이 사용된다.</strong></p>
</blockquote>