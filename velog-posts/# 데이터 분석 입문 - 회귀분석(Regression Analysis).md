<h1 id="회귀분석regression-analysis이란">회귀분석(Regression Analysis)이란?</h1>
<p>데이터를 분석하다 보면 다음과 같은 궁금증이 생깁니다.</p>
<ul>
<li>광고비를 늘리면 매출은 얼마나 증가할까?</li>
<li>공부시간이 1시간 늘어나면 시험 점수는 몇 점 오를까?</li>
<li>집의 면적이 커질수록 집값은 얼마나 상승할까?</li>
</ul>
<p>이처럼 <strong>하나 이상의 변수(X)를 이용하여 다른 변수(Y)를 설명하거나 예측하는 통계 기법</strong>을 <strong>회귀분석(Regression Analysis)</strong>이라고 합니다.</p>
<p>즉,</p>
<blockquote>
<p><strong>독립변수(X)를 이용하여 종속변수(Y)를 가장 잘 예측하는 직선을 찾는 과정</strong>입니다.</p>
</blockquote>
<hr />
<h1 id="상관분석과-회귀분석의-차이">상관분석과 회귀분석의 차이</h1>
<p>많은 사람들이 두 분석을 헷갈려 합니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>상관분석</th>
<th>회귀분석</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>두 변수의 관계 확인</td>
<td>값을 예측</td>
</tr>
<tr>
<td>결과</td>
<td>상관계수(r)</td>
<td>회귀식</td>
</tr>
<tr>
<td>방향성</td>
<td>없음</td>
<td>X → Y</td>
</tr>
<tr>
<td>예측 가능</td>
<td>❌</td>
<td>⭕</td>
</tr>
</tbody></table>
<p>예를 들어,</p>
<ul>
<li><strong>상관분석</strong>은 &quot;공부시간과 시험점수는 관련이 있는가?&quot;를 확인합니다.</li>
<li><strong>회귀분석</strong>은 &quot;공부시간이 5시간이면 시험점수는 몇 점일까?&quot;를 예측합니다.</li>
</ul>
<hr />
<h1 id="독립변수와-종속변수">독립변수와 종속변수</h1>
<p>회귀분석에서는 두 가지 변수를 사용합니다.</p>
<ul>
<li><strong>독립변수(X)</strong> : 원인 또는 입력값</li>
<li><strong>종속변수(Y)</strong> : 결과 또는 예측 대상</li>
</ul>
<h3 id="예시">예시</h3>
<table>
<thead>
<tr>
<th>독립변수(X)</th>
<th>종속변수(Y)</th>
</tr>
</thead>
<tbody><tr>
<td>공부시간</td>
<td>시험점수</td>
</tr>
<tr>
<td>광고비</td>
<td>매출</td>
</tr>
<tr>
<td>면적</td>
<td>집값</td>
</tr>
<tr>
<td>운동시간</td>
<td>체지방률</td>
</tr>
</tbody></table>
<hr />
<h1 id="회귀식regression-equation">회귀식(Regression Equation)</h1>
<p>회귀분석은 데이터를 가장 잘 설명하는 직선을 찾습니다.</p>
<p>회귀식은 다음과 같습니다.</p>
<p>$$
\hat{Y}=b_0+b_1X
$$</p>
<p>각 기호의 의미는 다음과 같습니다.</p>
<ul>
<li><strong>$\hat{Y}$</strong> : 예측값</li>
<li><strong>$b_0$</strong> : 절편(Intercept)</li>
<li><strong>$b_1$</strong> : 기울기(Slope)</li>
<li><strong>$X$</strong> : 독립변수</li>
</ul>
<p>예를 들어</p>
<p>$$
\hat{Y}=40+10X
$$</p>
<p>이라면</p>
<p>공부시간이 3시간일 때</p>
<p>$$
40+10\times3=70
$$</p>
<p>즉,</p>
<p>예상 시험점수는 <strong>70점</strong>입니다.</p>
<hr />
<h1 id="기울기와-절편">기울기와 절편</h1>
<h3 id="기울기slope">기울기(Slope)</h3>
<p>기울기는 <strong>X가 1 증가할 때 Y가 얼마나 변하는지</strong>를 의미합니다.</p>
<p>예를 들어</p>
<p>$$
\hat{Y}=40+10X
$$</p>
<p>에서는</p>
<p>기울기가 10이므로</p>
<blockquote>
<p>공부시간이 1시간 증가하면 시험점수는 평균적으로 10점 증가합니다.</p>
</blockquote>
<hr />
<h3 id="절편intercept">절편(Intercept)</h3>
<p>절편은</p>
<blockquote>
<p><strong>X가 0일 때 Y의 값</strong></p>
</blockquote>
<p>입니다.</p>
<p>위 식에서는</p>
<p>공부시간이 0시간이면</p>
<p>예상 점수는 40점입니다.</p>
<hr />
<h1 id="최소제곱법least-squares">최소제곱법(Least Squares)</h1>
<p>실제 데이터는 하나의 직선 위에 정확히 놓이지 않습니다.</p>
<p>따라서 수많은 직선 중에서</p>
<p><strong>데이터를 가장 잘 설명하는 직선</strong>을 선택해야 합니다.</p>
<p>이때 사용하는 방법이 <strong>최소제곱법</strong>입니다.</p>
<p>최소제곱법은</p>
<blockquote>
<p><strong>실제값과 예측값의 차이(잔차)의 제곱합이 가장 작은 직선</strong>을 선택하는 방법입니다.</p>
</blockquote>
<p>즉,</p>
<p>가장 오차가 작은 직선을 찾는 과정입니다.</p>
<hr />
<h1 id="잔차residual와-rmse">잔차(Residual)와 RMSE</h1>
<p>회귀분석에서는 실제 데이터와 예측값이 완전히 일치하는 경우는 거의 없습니다.</p>
<p>이때 <strong>실제값과 예측값의 차이</strong>를 <strong>잔차(Residual)</strong>라고 합니다.</p>
<p>$$
\text{잔차} = \text{실제값} - \text{예측값}
$$</p>
<p>예를 들어,</p>
<table>
<thead>
<tr>
<th align="right">실제 점수</th>
<th align="right">예측 점수</th>
<th align="right">잔차</th>
</tr>
</thead>
<tbody><tr>
<td align="right">80</td>
<td align="right">78</td>
<td align="right">2</td>
</tr>
<tr>
<td align="right">70</td>
<td align="right">72</td>
<td align="right">-2</td>
</tr>
<tr>
<td align="right">90</td>
<td align="right">88</td>
<td align="right">2</td>
</tr>
</tbody></table>
<ul>
<li><strong>잔차가 0</strong>이면 예측이 정확합니다.</li>
<li><strong>잔차의 절댓값이 작을수록</strong> 예측이 실제값에 가깝습니다.</li>
<li><strong>잔차의 절댓값이 클수록</strong> 예측 오차가 크다는 의미입니다.</li>
</ul>
<p>하지만 잔차는 <strong>개별 데이터의 오차</strong>만 보여주기 때문에, 모델 전체의 성능을 판단하기는 어렵습니다.</p>
<p>그래서 사용하는 지표가 <strong>RMSE(Root Mean Squared Error)</strong>입니다.</p>
<p>RMSE는 <strong>모든 데이터의 잔차를 이용하여 평균적인 예측 오차를 계산한 값</strong>입니다.</p>
<p>$$
RMSE=
\sqrt{
\frac{
\sum(실제값-예측값)^2
}{n}
}
$$</p>
<p>즉,</p>
<ol>
<li>잔차를 구한다.</li>
<li>잔차를 제곱한다.</li>
<li>제곱한 값의 평균을 구한다.</li>
<li>마지막으로 제곱근을 취한다.</li>
</ol>
<p>예를 들어 RMSE가 <strong>3</strong>이라면,</p>
<blockquote>
<p><strong>모델이 평균적으로 약 3점 정도의 오차를 가지고 예측한다</strong>는 의미입니다.</p>
</blockquote>
<p>따라서 <strong>RMSE는 작을수록 예측 성능이 좋은 회귀모델</strong>이라고 판단합니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>잔차(Residual)</td>
<td>개별 데이터의 예측 오차</td>
</tr>
<tr>
<td>RMSE</td>
<td>모든 잔차를 종합한 평균적인 예측 오차</td>
</tr>
</tbody></table>
<blockquote>
<p><strong>한 줄 정리</strong><br /><strong>잔차는 개별 데이터의 오차이고, RMSE는 모든 잔차를 종합하여 모델의 평균적인 예측 성능을 나타내는 지표입니다.</strong></p>
</blockquote>
<p>*<em>예시- 초보자가 쉽게 기억하는 방법
*</em>잔차 = &quot;한 학생의 오답&quot;
RMSE = &quot;반 전체 학생들이 평균적으로 몇 점 정도 틀렸는지&quot;</p>
<hr />
<h1 id="결정계수r²-or-r-squared">결정계수(R² or R-SQUARED)</h1>
<p>R²(결정계수)는 회귀모델이 실제 데이터의 패턴을 얼마나 잘 설명(재현)하는지를 나타내는 지표이며, 값이 1에 가까울수록 예측값이 실제값에 더 가깝다는 의미입니다.</p>
<p>$$
0 \le R^2 \le 1
$$</p>
<table>
<thead>
<tr>
<th align="right">R²</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1</td>
<td>완벽한 예측</td>
</tr>
<tr>
<td align="right">0.8</td>
<td>80% 설명</td>
</tr>
<tr>
<td align="right">0.5</td>
<td>50% 설명</td>
</tr>
<tr>
<td align="right">0</td>
<td>설명하지 못함</td>
</tr>
</tbody></table>
<p>예를 들어</p>
<p>$$
R^2=0.92
$$</p>
<p>라면</p>
<blockquote>
<p><strong>시험점수의 변동 중 92%를 공부시간으로 설명할 수 있다.</strong>
8% 는 나머지 다른 요인(예: 취침 시간,컨디션...)으로 설명할 수 있다</p>
</blockquote>
<p> <strong>주의:</strong> R²가 높다고 해서 반드시 좋은 모델은 아닙니다. 새로운 데이터에서도 성능이 좋은지 확인하기 위해 <strong>RMSE, MAE</strong> 등의 오차 지표와 함께 평가해야 합니다.</p>
<hr />
<h2 id="회귀효과regression-to-the-mean">회귀효과(Regression to the Mean)</h2>
<p>회귀효과란 <strong>우연한 요인으로 평균보다 매우 높거나 낮은 결과가 나왔을 때, 다음 측정에서는 평균에 가까운 값으로 돌아오는 현상</strong>을 말합니다.</p>
<h3 id="예시-1">예시</h3>
<p>중간고사에서 평소보다 운이 좋아 <strong>100점</strong>을 받은 학생이 있다고 가정해 보겠습니다.</p>
<p>이 학생의 실제 실력이 90점 수준이라면, 기말고사에서는 운의 영향이 줄어들면서 <strong>90점 안팎</strong>의 점수를 받을 가능성이 높습니다.</p>
<p>반대로, 평소 실력이 90점인 학생이 중간고사에서 컨디션이 좋지 않아 <strong>70점</strong>을 받았다면, 기말고사에서는 다시 <strong>평균에 가까운 90점</strong> 정도로 회복될 가능성이 높습니다.</p>
<p>즉,</p>
<ul>
<li><strong>평소보다 매우 높은 성적</strong> → 다음 시험에서는 평균에 가까워질 가능성이 높음</li>
<li><strong>평소보다 매우 낮은 성적</strong> → 다음 시험에서는 평균에 가까워질 가능성이 높음</li>
</ul>
<p>이는 실력이 갑자기 변해서가 아니라, <strong>우연한 요인(운, 컨디션, 문제 난이도 등)의 영향이 줄어들기 때문</strong>입니다.</p>
<blockquote>
<p><strong>회귀효과는 극단적인 결과일수록 다음 측정에서는 평균에 가까워지는 통계적 현상을 의미합니다.</strong></p>
</blockquote>
<hr />
<h1 id="회귀분석의-가정">회귀분석의 가정</h1>
<p>회귀분석은 다음 조건을 만족할 때 신뢰할 수 있는 결과를 얻을 수 있습니다.</p>
<ul>
<li><strong>선형성</strong> : X와 Y는 선형 관계를 가진다.</li>
<li><strong>독립성</strong> : 데이터는 서로 영향을 주지 않는다.</li>
<li><strong>등분산성</strong> : 잔차의 분산이 일정하다.</li>
<li><strong>정규성</strong> : 잔차는 정규분포를 따른다.</li>
<li><strong>다중공선성 없음</strong> : 독립변수끼리 지나치게 높은 상관관계를 가지지 않는다.</li>
</ul>
<hr />
<h2 id="회귀분석-결과의-해석">회귀분석 결과의 해석</h2>
<p>회귀분석을 수행하면 다양한 지표가 출력됩니다. 각 지표는 <strong>모델의 성능</strong>, <strong>변수의 영향력</strong>, <strong>통계적 유의성</strong> 등을 평가하는 역할을 합니다.</p>
<table>
<thead>
<tr>
<th>지표</th>
<th>핵심 질문</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><strong>R² (결정계수)</strong></td>
<td>모델이 데이터를 얼마나 잘 설명하는가?</td>
<td>0~1 사이의 값으로, 1에 가까울수록 모델의 설명력이 높다.</td>
</tr>
<tr>
<td><strong>P-value</strong></td>
<td>어떤 변수가 실제로 영향을 미치는가?</td>
<td>일반적으로 <strong>0.05 미만</strong>이면 해당 변수가 통계적으로 유의한 영향을 미친다고 판단한다.</td>
</tr>
<tr>
<td><strong>Coefficient (회귀계수)</strong></td>
<td>변수의 영향력과 방향은 무엇인가?</td>
<td>계수의 <strong>절댓값</strong>은 영향력의 크기, <strong>부호(+/-)</strong>는 영향의 방향을 의미한다.</td>
</tr>
<tr>
<td><strong>RMSE (Root Mean Squared Error)</strong></td>
<td>평균적으로 얼마나 틀리는가?</td>
<td>예측값과 실제값의 평균적인 오차를 나타내며, <strong>작을수록 좋은 모델</strong>이다.</td>
</tr>
<tr>
<td><strong>F-test (F-검정)</strong></td>
<td>모델 전체가 유의한가?</td>
<td>회귀모델 전체가 통계적으로 의미가 있는지 검정한다. 일반적으로 <strong>P-value &lt; 0.05</strong>이면 모델이 유의하다고 판단한다.</td>
</tr>
</tbody></table>
<blockquote>
<p><strong>한 줄 정리</strong></p>
<ul>
<li><strong>R²</strong> → 모델의 설명력</li>
<li><strong>P-value</strong> → 변수의 유의성</li>
<li><strong>Coefficient</strong> → 변수의 영향력과 방향</li>
<li><strong>RMSE</strong> → 예측 오차</li>
<li><strong>F-test</strong> → 모델 전체의 유의성</li>
</ul>
</blockquote>
<h1 id="실무-활용">실무 활용</h1>
<h3 id="1-매출-예측">1. 매출 예측</h3>
<p>광고비를 입력하여</p>
<p>예상 매출을 예측합니다.</p>
<hr />
<h3 id="2-부동산-가격-예측">2. 부동산 가격 예측</h3>
<p>면적</p>
<p>방 개수</p>
<p>위치 등을 이용하여</p>
<p>집값을 예측합니다.</p>
<hr />
<h3 id="3-수요-예측">3. 수요 예측</h3>
<p>과거 판매량을 이용하여</p>
<p>다음 달 판매량을 예측합니다.</p>
<hr />
<h3 id="4-ai-프로젝트">4. AI 프로젝트</h3>
<p>머신러닝에서는</p>
<p>회귀모델을 이용하여</p>
<ul>
<li>주가 예측</li>
<li>전력 사용량 예측</li>
<li>매출 예측</li>
<li>생산량 예측</li>
</ul>
<p>등 다양한 문제를 해결합니다.</p>
<hr />
<h1 id="python-실습">Python 실습</h1>
<pre><code class="language-python">import pandas as pd
from sklearn.linear_model import LinearRegression

# 데이터 준비
X = df[[&quot;StudyTime&quot;]]
y = df[&quot;Score&quot;]

# 모델 생성 및 학습
model = LinearRegression()
model.fit(X, y)

# 기울기와 절편
print(model.coef_)
print(model.intercept_)

# 결정계수(R²)
print(model.score(X, y))</code></pre>
<hr />
<h1 id="핵심-정리">핵심 정리</h1>
<table>
<thead>
<tr>
<th>항목</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>회귀분석</td>
<td>변수를 이용하여 값을 예측하는 분석 방법</td>
</tr>
<tr>
<td>독립변수(X)</td>
<td>원인 또는 입력값</td>
</tr>
<tr>
<td>종속변수(Y)</td>
<td>예측 대상</td>
</tr>
<tr>
<td>회귀식</td>
<td>Y를 예측하는 수식</td>
</tr>
<tr>
<td>기울기</td>
<td>X가 1 증가할 때 Y의 변화량</td>
</tr>
<tr>
<td>절편</td>
<td>X=0일 때 Y값</td>
</tr>
<tr>
<td>최소제곱법</td>
<td>잔차 제곱합이 가장 작은 직선 선택</td>
</tr>
<tr>
<td>잔차</td>
<td>실제값과 예측값의 차이</td>
</tr>
<tr>
<td>결정계수(R²)</td>
<td>모델의 설명력(0~1)</td>
</tr>
</tbody></table>
<hr />
<h1 id="면접에서-자주-나오는-질문">면접에서 자주 나오는 질문</h1>
<h3 id="q1-상관분석과-회귀분석의-차이는">Q1. 상관분석과 회귀분석의 차이는?</h3>
<ul>
<li>상관분석은 변수 간의 <strong>관계</strong>를 확인합니다.</li>
<li>회귀분석은 변수 간의 관계를 이용하여 <strong>미래의 값을 예측</strong>합니다.</li>
</ul>
<hr />
<h3 id="q2-기울기의-의미는">Q2. 기울기의 의미는?</h3>
<p>독립변수가 1 증가할 때 종속변수가 평균적으로 얼마나 변하는지를 의미합니다.</p>
<hr />
<h3 id="q3-결정계수r²가-높을수록-좋은-모델인가">Q3. 결정계수(R²)가 높을수록 좋은 모델인가?</h3>
<p>일반적으로는 그렇지만, <strong>과적합(Overfitting)</strong>으로 인해 R²만 높고 새로운 데이터에는 성능이 낮을 수도 있으므로 다른 평가 지표와 함께 확인해야 합니다.</p>
<hr />
<h1 id="자주-하는-실수">자주 하는 실수</h1>
<ul>
<li>❌ 상관분석과 회귀분석을 같은 개념으로 생각한다.</li>
<li>❌ R²만 보고 모델의 성능을 판단한다.</li>
<li>❌ 이상치가 많은 데이터를 그대로 회귀분석한다.</li>
<li>❌ 독립변수 간 높은 상관관계(다중공선성)를 확인하지 않는다.</li>
</ul>
<hr />
<h1 id="한-줄-요약">한 줄 요약</h1>
<blockquote>
<p><strong>회귀분석은 독립변수(X)를 이용하여 종속변수(Y)를 가장 잘 설명하는 모델을 만들고, 이를 바탕으로 미래의 값을 예측하는 데이터 분석 기법이다.</strong></p>
</blockquote>