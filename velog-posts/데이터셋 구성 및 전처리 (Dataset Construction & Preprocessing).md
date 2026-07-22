<p>데이터셋 구성 및 전처리란,</p>
<blockquote>
<p><strong>원본 데이터를 머신러닝이나 통계 모델이 학습할 수 있는 형태로 가공하는 과정</strong>입니다.</p>
</blockquote>
<p>실제 데이터에는 결측치, 이상치, 서로 다른 단위의 값 등이 존재하기 때문에 그대로 모델에 입력하면 성능이 저하될 수 있습니다.</p>
<p>따라서 전처리를 통해 데이터를 정리하고 변환하여 모델이 데이터를 더 잘 학습할 수 있도록 만듭니다.</p>
<hr />
<h3 id="1-결측치-처리-missing-value">1. 결측치 처리 (Missing Value)</h3>
<p>결측치(Missing Value)는 <strong>비어 있는 데이터</strong>를 의미합니다.</p>
<p>예를 들어,</p>
<table>
<thead>
<tr>
<th align="right">나이</th>
<th align="right">연봉</th>
</tr>
</thead>
<tbody><tr>
<td align="right">25</td>
<td align="right">3000</td>
</tr>
<tr>
<td align="right">30</td>
<td align="right">❌</td>
</tr>
<tr>
<td align="right">28</td>
<td align="right">3500</td>
</tr>
</tbody></table>
<p>처럼 일부 값이 없는 경우입니다.</p>
<p>결측치는 그대로 사용할 수 없는 경우가 많기 때문에 다음과 같은 방법으로 처리합니다.</p>
<ul>
<li>평균값 또는 중앙값으로 대체</li>
<li>최빈값으로 대체</li>
<li>해당 행 또는 열 삭제</li>
<li>머신러닝을 이용하여 예측 후 대체</li>
</ul>
<blockquote>
<p><strong>목적:</strong> 데이터의 누락으로 인해 모델 학습이 어려워지는 것을 방지</p>
</blockquote>
<hr />
<h3 id="2-이상치-처리-outlier">2. 이상치 처리 (Outlier)</h3>
<p>이상치란 <strong>다른 데이터와 비교했을 때 지나치게 크거나 작은 값</strong>입니다.</p>
<p>예를 들어,</p>
<pre><code class="language-text">25, 27, 30, 28, 26, 500</code></pre>
<p>에서 <strong>500</strong>은 이상치입니다.</p>
<p>이상치는 회귀모델이나 평균값에 큰 영향을 줄 수 있으므로 제거하거나 수정하는 경우가 많습니다.</p>
<blockquote>
<p><strong>목적:</strong> 모델이 극단적인 데이터에 영향을 받지 않도록 하기 위함</p>
</blockquote>
<hr />
<h3 id="3-스케일링-scaling">3. 스케일링 (Scaling)</h3>
<p>스케일링은 <strong>변수들의 크기(단위)를 맞추는 작업</strong>입니다.</p>
<p>예를 들어,</p>
<table>
<thead>
<tr>
<th>변수</th>
<th align="right">값</th>
</tr>
</thead>
<tbody><tr>
<td>나이</td>
<td align="right">25</td>
</tr>
<tr>
<td>연봉</td>
<td align="right">50,000,000</td>
</tr>
</tbody></table>
<p>처럼 변수마다 값의 크기가 크게 다르면 모델이 큰 값에 더 많은 영향을 받을 수 있습니다.</p>
<p>따라서 변수들의 범위를 비슷하게 맞춰줍니다.</p>
<p>대표적인 방법은 다음과 같습니다.</p>
<ul>
<li><strong>Standard Scaling</strong> : 평균 0, 표준편차 1로 변환</li>
<li><strong>Min-Max Scaling</strong> : 0~1 범위로 변환</li>
</ul>
<blockquote>
<p><strong>목적:</strong> 변수 간 단위 차이를 줄여 모델이 공정하게 학습하도록 함</p>
</blockquote>
<hr />
<h3 id="4-데이터-변환-transformation">4. 데이터 변환 (Transformation)</h3>
<p>데이터 변환은 <strong>데이터의 분포를 모델이 학습하기 좋은 형태로 바꾸는 과정</strong>입니다.</p>
<p>대표적인 방법은 다음과 같습니다.</p>
<ul>
<li>로그 변환(Log Transformation)</li>
<li>제곱근 변환(Square Root Transformation)</li>
<li>Box-Cox 변환</li>
<li>Yeo-Johnson 변환</li>
</ul>
<p>예를 들어,</p>
<p>매출 데이터처럼 일부 값이 매우 큰 경우 로그 변환을 적용하면 데이터의 편향을 줄일 수 있습니다.</p>
<blockquote>
<p><strong>목적:</strong> 데이터의 분포를 안정화하고 모델 성능을 향상</p>
</blockquote>
<hr />
<h3 id="5-인코딩-encoding">5. 인코딩 (Encoding)</h3>
<p>머신러닝 모델은 대부분 <strong>문자 데이터를 직접 처리하지 못합니다.</strong></p>
<p>예를 들어,</p>
<pre><code class="language-text">남자
여자</code></pre>
<p>와 같은 데이터를 숫자로 변환해야 합니다.</p>
<p>대표적인 방법은</p>
<ul>
<li>Label Encoding</li>
<li>One-Hot Encoding</li>
</ul>
<p>입니다.</p>
<blockquote>
<p><strong>목적:</strong> 범주형 데이터를 모델이 이해할 수 있는 숫자 형태로 변환</p>
</blockquote>
<hr />
<h3 id="6-데이터-분할-traintest-split">6. 데이터 분할 (Train/Test Split)</h3>
<p>전처리가 끝난 데이터는 보통</p>
<ul>
<li><strong>학습용(Train Set)</strong></li>
<li><strong>검증용(Test Set)</strong></li>
</ul>
<p>으로 나누어 사용합니다.</p>
<p>이는 모델이 새로운 데이터에서도 잘 동작하는지 평가하기 위함입니다.</p>
<blockquote>
<p><strong>목적:</strong> 모델의 일반화 성능 평가</p>
</blockquote>
<hr />
<h2 id="핵심-정리">핵심 정리</h2>
<table>
<thead>
<tr>
<th>전처리 기법</th>
<th>목적</th>
</tr>
</thead>
<tbody><tr>
<td>결측치 처리</td>
<td>누락된 데이터를 보완하거나 제거</td>
</tr>
<tr>
<td>이상치 처리</td>
<td>극단적인 데이터의 영향 감소</td>
</tr>
<tr>
<td>스케일링</td>
<td>변수들의 크기와 단위를 맞춤</td>
</tr>
<tr>
<td>데이터 변환</td>
<td>데이터 분포를 안정화</td>
</tr>
<tr>
<td>인코딩</td>
<td>문자 데이터를 숫자로 변환</td>
</tr>
<tr>
<td>데이터 분할</td>
<td>학습과 성능 평가를 위한 데이터 분리</td>
</tr>
</tbody></table>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><strong>데이터 전처리는 원본 데이터를 모델이 학습하기 좋은 형태로 정리·변환하는 과정이며, 모델 성능을 높이기 위한 필수 단계입니다.</strong></p>
</blockquote>