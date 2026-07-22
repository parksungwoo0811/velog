<blockquote>
<p><strong>학습일:</strong> 2026.07.18
<strong>주제:</strong> Python NumPy 기초부터 실무 활용까지</p>
</blockquote>
<hr />
<h1 id="numpy란">NumPy란?</h1>
<p>NumPy(Numerical Python)는 <strong>파이썬에서 숫자 데이터를 빠르고 효율적으로 계산하기 위한 라이브러리</strong>입니다.</p>
<p>데이터 분석, 머신러닝, 인공지능에서는 거의 필수적으로 사용됩니다.</p>
<p>Pandas, Scikit-Learn, TensorFlow, PyTorch 등 대부분의 데이터 분석 및 AI 라이브러리도 내부적으로 NumPy를 기반으로 동작합니다.</p>
<hr />
<h2 id="왜-numpy를-사용할까">왜 NumPy를 사용할까?</h2>
<p>Python의 기본 리스트(list)도 숫자를 저장할 수 있습니다.</p>
<p>하지만 데이터가 수천 개, 수만 개, 수백만 개가 되면 계산 속도가 매우 느려집니다.</p>
<p>NumPy는 이러한 문제를 해결하기 위해 만들어졌습니다.</p>
<h3 id="쉽게-설명">쉽게 설명</h3>
<blockquote>
<p><strong>Python List = 종이에 숫자를 하나씩 적는 것</strong></p>
</blockquote>
<blockquote>
<p><strong>NumPy Array = 엑셀에서 한 번에 계산하는 것</strong></p>
</blockquote>
<p>리스트는 하나씩 계산하지만,</p>
<p>NumPy는 여러 데이터를 한 번에 계산합니다.</p>
<p>이것을 <strong>벡터화(Vectorization)</strong> 라고 합니다.</p>
<hr />
<h2 id="비유">비유</h2>
<p>마트에서 사과 100개 가격을 계산한다고 생각해 봅시다.</p>
<h3 id="python-list">Python List</h3>
<pre><code>사과 하나 계산
↓

또 하나 계산
↓

또 하나 계산
↓

100번 반복</code></pre><h3 id="numpy">NumPy</h3>
<pre><code>사과 100개

↓

한 번에 계산</code></pre><p>그래서 훨씬 빠릅니다.</p>
<hr />
<h1 id="numpy의-핵심-자료형">NumPy의 핵심 자료형</h1>
<p>NumPy에서는 데이터를 <strong>Array(배열)</strong> 형태로 저장합니다.</p>
<pre><code>Python

[1,2,3,4]

↓

NumPy

array([1,2,3,4])</code></pre><hr />
<h2 id="array란">Array란?</h2>
<p>같은 자료형의 데이터를 연속된 메모리에 저장하는 자료구조입니다.</p>
<p>쉽게 말하면</p>
<blockquote>
<p>&quot;숫자 전용 리스트&quot;</p>
</blockquote>
<p>라고 생각하면 됩니다.</p>
<hr />
<h2 id="왜-array를-사용할까">왜 Array를 사용할까?</h2>
<table>
<thead>
<tr>
<th>Python List</th>
<th>NumPy Array</th>
</tr>
</thead>
<tbody><tr>
<td>자료형 섞임 가능</td>
<td>같은 자료형만 저장</td>
</tr>
<tr>
<td>속도 느림</td>
<td>매우 빠름</td>
</tr>
<tr>
<td>메모리 많이 사용</td>
<td>메모리 효율적</td>
</tr>
<tr>
<td>대용량 계산 부적합</td>
<td>대용량 계산 최적화</td>
</tr>
</tbody></table>
<hr />
<h1 id="numpy-설치">NumPy 설치</h1>
<pre><code class="language-bash">pip install numpy</code></pre>
<hr />
<h1 id="numpy-불러오기">NumPy 불러오기</h1>
<pre><code class="language-python">import numpy as np</code></pre>
<h3 id="왜-np라고-쓰나요">왜 np라고 쓰나요?</h3>
<p>관례(convention)입니다.</p>
<p>거의 모든 데이터 분석 코드에서 이렇게 사용합니다.</p>
<hr />
<h1 id="배열array-만들기">배열(Array) 만들기</h1>
<h2 id="1-리스트로-생성">1. 리스트로 생성</h2>
<pre><code class="language-python">import numpy as np

# 리스트를 NumPy 배열로 변환
arr = np.array([1,2,3,4,5])

print(arr)</code></pre>
<p>출력</p>
<pre><code>[1 2 3 4 5]</code></pre><hr />
<h2 id="2차원-배열-만들기">2차원 배열 만들기</h2>
<pre><code class="language-python">arr = np.array([
    [1,2,3],
    [4,5,6]
])

print(arr)</code></pre>
<p>출력</p>
<pre><code>[[1 2 3]
 [4 5 6]]</code></pre><hr />
<h2 id="비유-1">비유</h2>
<p>1차원 배열</p>
<pre><code>책 한 줄</code></pre><p>2차원 배열</p>
<pre><code>엑셀 표</code></pre><p>3차원 배열</p>
<pre><code>엑셀 파일 여러 장</code></pre><hr />
<h1 id="배열-정보-확인">배열 정보 확인</h1>
<pre><code class="language-python">import numpy as np

arr = np.array([
    [1,2,3],
    [4,5,6]
])

print(arr.shape)</code></pre>
<p>출력</p>
<pre><code>(2,3)</code></pre><p>의미</p>
<pre><code>2행

3열</code></pre><hr />
<h2 id="차원-확인">차원 확인</h2>
<pre><code class="language-python">print(arr.ndim)</code></pre>
<p>출력</p>
<pre><code>2</code></pre><hr />
<h2 id="데이터-개수">데이터 개수</h2>
<pre><code class="language-python">print(arr.size)</code></pre>
<p>출력</p>
<pre><code>6</code></pre><hr />
<h2 id="데이터-타입">데이터 타입</h2>
<pre><code class="language-python">print(arr.dtype)</code></pre>
<p>출력</p>
<pre><code>int64</code></pre><hr />
<h1 id="자주-사용하는-배열-생성-함수">자주 사용하는 배열 생성 함수</h1>
<h2 id="①-0으로-채우기">① 0으로 채우기</h2>
<pre><code class="language-python">np.zeros((3,4))</code></pre>
<p>결과</p>
<pre><code>[[0. 0. 0. 0.]
 [0. 0. 0. 0.]
 [0. 0. 0. 0.]]</code></pre><hr />
<h2 id="②-1로-채우기">② 1로 채우기</h2>
<pre><code class="language-python">np.ones((2,3))</code></pre>
<hr />
<h2 id="③-원하는-숫자로-채우기">③ 원하는 숫자로 채우기</h2>
<pre><code class="language-python">np.full((2,2),100)</code></pre>
<p>결과</p>
<pre><code>[[100 100]
 [100 100]]</code></pre><hr />
<h2 id="④-연속된-숫자">④ 연속된 숫자</h2>
<pre><code class="language-python">np.arange(1,11)</code></pre>
<p>결과</p>
<pre><code>[1 2 3 4 5 6 7 8 9 10]</code></pre><hr />
<h2 id="⑤-일정-간격">⑤ 일정 간격</h2>
<pre><code class="language-python">np.linspace(0,100,5)</code></pre>
<p>결과</p>
<pre><code>[0. 25. 50. 75. 100.]</code></pre><hr />
<h2 id="언제-사용할까">언제 사용할까?</h2>
<table>
<thead>
<tr>
<th>함수</th>
<th>사용 상황</th>
</tr>
</thead>
<tbody><tr>
<td>zeros()</td>
<td>빈 이미지 생성</td>
</tr>
<tr>
<td>ones()</td>
<td>초기값 설정</td>
</tr>
<tr>
<td>full()</td>
<td>기본값 채우기</td>
</tr>
<tr>
<td>arange()</td>
<td>반복 숫자</td>
</tr>
<tr>
<td>linspace()</td>
<td>그래프 축 생성</td>
</tr>
</tbody></table>
<hr />
<h1 id="배열-연산">배열 연산</h1>
<p>Python List</p>
<pre><code class="language-python">a=[1,2,3]
b=[4,5,6]

print(a+b)</code></pre>
<p>결과</p>
<pre><code>[1,2,3,4,5,6]</code></pre><p>리스트는 이어붙입니다.</p>
<hr />
<p>NumPy</p>
<pre><code class="language-python">import numpy as np

a=np.array([1,2,3])
b=np.array([4,5,6])

print(a+b)</code></pre>
<p>결과</p>
<pre><code>[5 7 9]</code></pre><p>각 원소끼리 계산합니다.</p>
<hr />
<h2 id="사칙연산">사칙연산</h2>
<pre><code class="language-python">print(a+b)

print(a-b)

print(a*2)

print(a/2)</code></pre>
<hr />
<h2 id="비유-2">비유</h2>
<pre><code>학생 성적

국어

영어

수학

↓

모든 학생 점수에

+5점

한 번에 적용</code></pre><hr />
<h1 id="브로드캐스팅broadcasting">브로드캐스팅(Broadcasting)</h1>
<p>NumPy의 가장 강력한 기능 중 하나입니다.</p>
<pre><code class="language-python">scores=np.array([80,90,70])

scores+10</code></pre>
<p>결과</p>
<pre><code>[90 100 80]</code></pre><p>모든 원소에 자동으로 더합니다.</p>
<hr />
<h2 id="왜-사용할까">왜 사용할까?</h2>
<p>반복문 없이 계산하기 위해서입니다.</p>
<p>속도가 훨씬 빠릅니다.</p>
<hr />
<h1 id="인덱싱">인덱싱</h1>
<pre><code class="language-python">arr=np.array([10,20,30,40])

print(arr[0])</code></pre>
<p>출력</p>
<pre><code>10</code></pre><hr />
<p>2차원</p>
<pre><code class="language-python">arr=np.array([
    [1,2],
    [3,4]
])

print(arr[1][0])</code></pre>
<p>출력</p>
<pre><code>3</code></pre><hr />
<h1 id="슬라이싱">슬라이싱</h1>
<pre><code class="language-python">arr=np.array([1,2,3,4,5])

print(arr[1:4])</code></pre>
<p>결과</p>
<pre><code>[2 3 4]</code></pre><hr />
<h1 id="boolean-indexing">Boolean Indexing</h1>
<p>조건에 맞는 데이터만 선택합니다.</p>
<pre><code class="language-python">score=np.array([80,90,65,40])

print(score&gt;70)</code></pre>
<p>결과</p>
<pre><code>[ True  True False False]</code></pre><hr />
<p>선택</p>
<pre><code class="language-python">print(score[score&gt;70])</code></pre>
<p>출력</p>
<pre><code>[80 90]</code></pre><hr />
<h2 id="언제-사용할까-1">언제 사용할까?</h2>
<p>학생 성적</p>
<p>70점 이상만 보기</p>
<p>고객 구매금액</p>
<p>10만원 이상만 보기</p>
<p>매출</p>
<p>100만원 이상만 분석</p>
<hr />
<h1 id="집계-함수">집계 함수</h1>
<pre><code class="language-python">arr=np.array([10,20,30,40])</code></pre>
<p>평균</p>
<pre><code class="language-python">np.mean(arr)</code></pre>
<p>합</p>
<pre><code class="language-python">np.sum(arr)</code></pre>
<p>최댓값</p>
<pre><code class="language-python">np.max(arr)</code></pre>
<p>최솟값</p>
<pre><code class="language-python">np.min(arr)</code></pre>
<p>표준편차</p>
<pre><code class="language-python">np.std(arr)</code></pre>
<p>분산</p>
<pre><code class="language-python">np.var(arr)</code></pre>
<hr />
<h2 id="왜-중요할까">왜 중요할까?</h2>
<p>데이터 분석의 대부분은</p>
<p>&quot;전체 데이터의 특징&quot;</p>
<p>을 파악하는 과정입니다.</p>
<hr />
<h1 id="난수-생성">난수 생성</h1>
<pre><code class="language-python">np.random.rand(5)</code></pre>
<p>0~1</p>
<p>랜덤 생성</p>
<hr />
<p>정수</p>
<pre><code class="language-python">np.random.randint(1,100,10)</code></pre>
<hr />
<h2 id="언제-사용할까-2">언제 사용할까?</h2>
<ul>
<li>머신러닝 데이터 생성</li>
<li>테스트 데이터 생성</li>
<li>시뮬레이션</li>
</ul>
<hr />
<h1 id="배열-형태-변경">배열 형태 변경</h1>
<pre><code class="language-python">arr=np.arange(12)</code></pre>
<pre><code>[0~11]</code></pre><p>↓</p>
<pre><code class="language-python">arr.reshape(3,4)</code></pre>
<pre><code>3행

4열</code></pre><hr />
<p>평탄화</p>
<pre><code class="language-python">arr.flatten()</code></pre>
<hr />
<h2 id="비유-3">비유</h2>
<pre><code>종이 접기

↓

다시 펼치기</code></pre><p>reshape는 모양만 바꾸고</p>
<p>데이터는 그대로입니다.</p>
<hr />
<h1 id="실무-활용">실무 활용</h1>
<h2 id="사례-1-쇼핑몰-매출-분석">사례 1. 쇼핑몰 매출 분석</h2>
<pre><code class="language-python">import numpy as np

sales = np.array([120000, 150000, 90000, 210000, 180000])

# 평균 매출 계산
print(np.mean(sales))

# 최고 매출
print(np.max(sales))

# 15만 원 이상 매출만 추출
print(sales[sales &gt;= 150000])</code></pre>
<h3 id="왜-사용할까-1">왜 사용할까?</h3>
<ul>
<li>하루 평균 매출 계산</li>
<li>최대/최소 매출 파악</li>
<li>특정 금액 이상 주문만 분석</li>
</ul>
<hr />
<h2 id="사례-2-학생-성적-분석">사례 2. 학생 성적 분석</h2>
<pre><code class="language-python">import numpy as np

scores = np.array([85, 92, 78, 96, 88])

# 평균 점수
print(np.mean(scores))

# 최고 점수
print(np.max(scores))

# 평균 이상 학생만 추출
avg = np.mean(scores)
print(scores[scores &gt;= avg])</code></pre>
<h3 id="왜-사용할까-2">왜 사용할까?</h3>
<ul>
<li>시험 결과 분석</li>
<li>평균 이상 학생 확인</li>
<li>성적 분포 파악</li>
</ul>
<hr />
<h2 id="사례-3-센서-데이터-분석">사례 3. 센서 데이터 분석</h2>
<p>공장의 온도 센서가 1초마다 데이터를 기록한다고 가정해 보겠습니다.</p>
<pre><code class="language-python">import numpy as np

temperature = np.array([25.1, 25.3, 24.9, 25.7, 26.1])

print(np.mean(temperature))   # 평균 온도
print(np.max(temperature))    # 최고 온도
print(temperature[temperature &gt; 25.5])  # 기준 이상 온도</code></pre>
<h3 id="왜-사용할까-3">왜 사용할까?</h3>
<ul>
<li>이상 온도 감지</li>
<li>장비 이상 여부 확인</li>
<li>실시간 모니터링</li>
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
<td>데이터 분석</td>
<td>숫자 계산</td>
</tr>
<tr>
<td>AI</td>
<td>행렬 연산</td>
</tr>
<tr>
<td>머신러닝</td>
<td>데이터 전처리</td>
</tr>
<tr>
<td>이미지 처리</td>
<td>픽셀 계산</td>
</tr>
<tr>
<td>금융</td>
<td>주가 분석</td>
</tr>
<tr>
<td>센서 데이터</td>
<td>실시간 계산</td>
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
<td>빠른 속도</td>
<td>Python 리스트보다 훨씬 빠른 계산</td>
</tr>
<tr>
<td>메모리 효율</td>
<td>대용량 데이터 처리에 적합</td>
</tr>
<tr>
<td>벡터화 연산</td>
<td>반복문 없이 한 번에 계산 가능</td>
</tr>
<tr>
<td>다양한 함수</td>
<td>통계, 난수, 선형대수 등 풍부한 기능 제공</td>
</tr>
<tr>
<td>호환성</td>
<td>Pandas, Scikit-Learn, TensorFlow 등과 함께 사용</td>
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
<td>자료형 통일</td>
<td>한 배열에는 같은 자료형을 사용하는 것이 효율적입니다.</td>
</tr>
<tr>
<td>브로드캐스팅 이해</td>
<td>자동 연산은 편리하지만 배열 크기가 맞지 않으면 오류가 발생할 수 있습니다.</td>
</tr>
<tr>
<td>리스트와 차이</td>
<td><code>+</code> 연산의 의미가 리스트와 NumPy 배열에서 다릅니다.</td>
</tr>
<tr>
<td>메모리 공유</td>
<td>슬라이싱은 원본 데이터를 참조하는 경우가 있으므로 수정 시 주의가 필요합니다. (<code>copy()</code> 활용)</td>
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
<td>배열(Array)</td>
<td>숫자를 효율적으로 저장하는 자료구조</td>
<td><code>np.array()</code></td>
</tr>
<tr>
<td>배열 생성</td>
<td>0, 1, 일정값, 연속값 생성</td>
<td><code>zeros()</code>, <code>ones()</code>, <code>full()</code>, <code>arange()</code>, <code>linspace()</code></td>
</tr>
<tr>
<td>배열 정보</td>
<td>크기, 차원, 자료형 확인</td>
<td><code>shape</code>, <code>ndim</code>, <code>size</code>, <code>dtype</code></td>
</tr>
<tr>
<td>벡터화 연산</td>
<td>반복문 없이 한 번에 계산</td>
<td><code>+</code>, <code>-</code>, <code>*</code>, <code>/</code></td>
</tr>
<tr>
<td>브로드캐스팅</td>
<td>크기가 다른 배열도 자동 계산</td>
<td><code>array + 10</code></td>
</tr>
<tr>
<td>인덱싱/슬라이싱</td>
<td>원하는 데이터 선택</td>
<td><code>[]</code>, <code>:</code></td>
</tr>
<tr>
<td>Boolean Indexing</td>
<td>조건에 맞는 데이터 추출</td>
<td><code>array[array &gt; x]</code></td>
</tr>
<tr>
<td>집계 함수</td>
<td>평균, 합계, 최댓값 등 계산</td>
<td><code>mean()</code>, <code>sum()</code>, <code>max()</code>, <code>min()</code></td>
</tr>
<tr>
<td>난수 생성</td>
<td>랜덤 데이터 생성</td>
<td><code>random.rand()</code>, <code>random.randint()</code></td>
</tr>
<tr>
<td>배열 변형</td>
<td>배열 모양 변경</td>
<td><code>reshape()</code>, <code>flatten()</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="한-줄-요약">한 줄 요약</h1>
<blockquote>
<p><strong>NumPy는 대용량 숫자 데이터를 빠르고 효율적으로 처리하기 위한 핵심 라이브러리이며, 벡터화 연산과 다양한 수학·통계 기능을 통해 데이터 분석과 AI 개발의 기반이 된다.</strong></p>
</blockquote>