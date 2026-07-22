<h2 id="개념">개념</h2>
<p><strong>Transformer</strong>는 현대 생성형 AI와 LLM(Large Language Model)의 핵심이 되는 딥러닝 아키텍처이다.</p>
<p>2017년 구글이 발표한 논문 <strong>「Attention Is All You Need」</strong>에서 처음 제안되었으며, 현재 GPT, BERT, Gemini, Claude, Llama 등 대부분의 최신 언어 모델이 Transformer 구조를 기반으로 만들어졌다.</p>
<p>기존의 <strong>RNN, LSTM, GRU</strong>는 문장을 <strong>앞에서부터 순서대로</strong> 읽으며 학습했다. 따라서</p>
<ul>
<li>문장이 길어질수록 앞부분 정보를 잊기 쉽고(Long-Term Dependency Problem)</li>
<li>순차적으로 계산해야 하므로 병렬 처리가 어렵고</li>
<li>학습 속도가 느리다는 한계가 있었다.</li>
</ul>
<p>Transformer는 이러한 문제를 <strong>Self-Attention</strong>이라는 메커니즘으로 해결하였다.</p>
<p>Self-Attention은</p>
<blockquote>
<p><strong>현재 단어를 이해하기 위해 문장 속 모든 단어가 얼마나 중요한지를 동시에 계산하는 방식</strong>이다.</p>
</blockquote>
<p>즉, 문장을 순서대로 읽지 않고 <strong>모든 단어 간의 관계를 한 번에 계산</strong>하기 때문에 긴 문맥도 효과적으로 이해할 수 있으며 GPU를 활용한 병렬 처리도 가능하다.</p>
<hr />
<h2 id="transformer의-전체-구조">Transformer의 전체 구조</h2>
<p>Transformer는 크게 두 부분으로 구성된다.</p>
<pre><code>    입력 문장
      │
      ▼
+------------------+
|     Encoder      |
| 문장을 이해하는 역할 |
+------------------+
      │
      ▼
 문맥(Context) 표현
      │
      ▼
+------------------+
|     Decoder      |
| 다음 단어 생성 역할 |
+------------------+
      │
      ▼
    출력 문장</code></pre><ul>
<li><strong>Encoder</strong> : 입력 문장의 의미와 문맥을 이해하는 역할</li>
<li><strong>Decoder</strong> : 이해한 문맥을 바탕으로 다음 단어를 생성하는 역할</li>
</ul>
<p>대표적인 모델은 다음과 같다.</p>
<table>
<thead>
<tr>
<th>모델</th>
<th>사용하는 구조</th>
</tr>
</thead>
<tbody><tr>
<td>BERT</td>
<td>Encoder</td>
</tr>
<tr>
<td>GPT</td>
<td>Decoder</td>
</tr>
<tr>
<td>T5</td>
<td>Encoder + Decoder</td>
</tr>
<tr>
<td>Transformer(원본)</td>
<td>Encoder + Decoder</td>
</tr>
</tbody></table>
<hr />
<h2 id="transformer의-핵심-구성-요소">Transformer의 핵심 구성 요소</h2>
<h3 id="1-query-q">1. Query (Q)</h3>
<p>현재 단어가 <strong>찾고 싶은 정보</strong>를 의미한다.</p>
<p>예를 들어</p>
<pre><code>나는 AI를 공부한다.</code></pre><p>&quot;공부한다&quot;는</p>
<ul>
<li>누가 공부하는가?</li>
<li>무엇을 공부하는가?</li>
</ul>
<p>를 찾으려고 한다.</p>
<p>이처럼 현재 단어가 찾으려는 정보가 Query이다.</p>
<hr />
<h3 id="2-key-k">2. Key (K)</h3>
<p>각 단어가 가지고 있는 <strong>검색용 표지(Label)</strong> 이다.</p>
<p>Query는 모든 Key와 비교하여</p>
<blockquote>
<p>&quot;어떤 단어가 나와 가장 관련 있는가?&quot;</p>
</blockquote>
<p>를 계산한다.</p>
<p>-&gt; Query 토큰들로부터 참고될 수 있는 정보</p>
<hr />
<h3 id="3-value-v">3. Value (V)</h3>
<p>실제로 가져올 <strong>의미 정보</strong>이다.</p>
<p>Attention은</p>
<ul>
<li>Query</li>
<li>Key</li>
</ul>
<p>를 비교하여 중요도를 계산한 뒤,</p>
<p>가장 중요한 Value를 많이 가져온다.</p>
<hr />
<h3 id="4-scaled-dot-product-attention">4. Scaled Dot-Product Attention</h3>
<p>Transformer의 핵심 계산 과정이다.</p>
<p>순서는 다음과 같다.</p>
<pre><code>Query
   │
Key와 내적(Dot Product)
   │
관련도 점수 계산
   │
√dk 로 스케일링
   │
Softmax
   │
Value와 가중합
   │
최종 문맥 벡터 생성</code></pre><p>수식은 다음과 같다.</p>
<p>$$
Attention(Q,K,V)
=
Softmax
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)
V
$$</p>
<p>여기서</p>
<ul>
<li><strong>Q</strong> : Query</li>
<li><strong>K</strong> : Key</li>
<li><strong>V</strong> : Value</li>
<li><strong>dk</strong> : Key 벡터의 차원</li>
</ul>
<p>이다.</p>
<blockquote>
<p>** √dk로 나누는 이유는 Q,K 벡터 차원이 커질수록 내적 값이 너무 커지서  Softmax함수의 기울기가 매우 작아지는 문제를 하기 위함이다. 이후 내적 값의 크기 범위가 안정화되어 적당한 softmax 기울기를 유지**</p>
</blockquote>
<hr />
<h3 id="5-softmax">5. Softmax</h3>
<p>Attention Score를</p>
<p><strong>0~1 사이의 확률 값</strong>으로 변환한다.</p>
<p>예를 들어</p>
<table>
<thead>
<tr>
<th>단어</th>
<th align="right">점수</th>
</tr>
</thead>
<tbody><tr>
<td>AI</td>
<td align="right">8.2</td>
</tr>
<tr>
<td>공부</td>
<td align="right">7.5</td>
</tr>
<tr>
<td>나는</td>
<td align="right">1.4</td>
</tr>
</tbody></table>
<p>Softmax를 적용하면</p>
<table>
<thead>
<tr>
<th>단어</th>
<th align="right">중요도</th>
</tr>
</thead>
<tbody><tr>
<td>AI</td>
<td align="right">0.47</td>
</tr>
<tr>
<td>공부</td>
<td align="right">0.40</td>
</tr>
<tr>
<td>나는</td>
<td align="right">0.13</td>
</tr>
</tbody></table>
<p>처럼 어떤 단어에 얼마나 집중할지를 결정한다.</p>
<hr />
<h3 id="6-multi-head-attention">6. Multi-Head Attention</h3>
<p>Transformer는 Attention을 <strong>한 번만 계산하지 않는다.</strong></p>
<p>여러 개의 Attention을 동시에 수행하여</p>
<p>각각 다른 관점에서 문장을 이해한다.</p>
<p>예를 들어</p>
<table>
<thead>
<tr>
<th>Head</th>
<th>집중하는 정보</th>
</tr>
</thead>
<tbody><tr>
<td>Head 1</td>
<td>문법</td>
</tr>
<tr>
<td>Head 2</td>
<td>의미</td>
</tr>
<tr>
<td>Head 3</td>
<td>문맥</td>
</tr>
<tr>
<td>Head 4</td>
<td>핵심 키워드</td>
</tr>
</tbody></table>
<p>이 결과들을 합쳐 더 풍부한 문장 표현을 만든다.</p>
<hr />
<h3 id="7-linear-layer">7. Linear Layer</h3>
<p>Multi-Head Attention에서 나온 여러 결과를</p>
<p>다시 하나의 벡터로 합치는 층이다.</p>
<p>즉,</p>
<p>여러 전문가의 의견을 종합하여</p>
<p>하나의 최종 의견을 만드는 과정이라고 생각하면 된다.</p>
<hr />
<h3 id="8-residual-connection">8. Residual Connection</h3>
<p>Attention을 여러 번 수행하면</p>
<p>기존 정보가 사라질 수 있다.</p>
<p>Residual Connection은</p>
<p><strong>원래 입력을 출력에 다시 더해주는 구조</strong>이다.</p>
<pre><code>입력
 │
Attention
 │
출력
 │
＋
 │
원래 입력
 │
최종 출력</code></pre><p>덕분에</p>
<ul>
<li>정보 손실 감소</li>
<li>Gradient Vanishing 완화</li>
<li>깊은 모델 학습 가능</li>
</ul>
<p>이라는 장점이 있다.</p>
<hr />
<h3 id="9-layer-normalization">9. Layer Normalization</h3>
<p>LayerNorm은</p>
<p>각 층의 출력 분포를 일정하게 만들어</p>
<p>학습이 안정적으로 이루어지도록 한다.</p>
<p>효과는</p>
<ul>
<li>학습 속도 향상</li>
<li>값의 폭주 방지</li>
<li>모델 안정성 향상</li>
</ul>
<p>이다.</p>
<hr />
<h3 id="10-feed-forward-network-ffn">10. Feed Forward Network (FFN)</h3>
<p>Attention으로 계산한 결과를</p>
<p>비선형 신경망을 이용하여</p>
<p>더 풍부한 표현으로 변환한다.</p>
<p>즉,</p>
<p>Attention이</p>
<p>&quot;무엇이 중요한가&quot;</p>
<p>를 찾는 과정이라면</p>
<p>Feed Forward는</p>
<p>&quot;그 정보를 더 잘 표현하도록 가공하는 과정&quot;</p>
<p>이라고 볼 수 있다.</p>
<hr />
<h3 id="11-positional-encoding">11. Positional Encoding</h3>
<p>Transformer는 문장을 동시에 보기 때문에</p>
<p>단어 순서를 알 수 없다.</p>
<p>그래서</p>
<p>각 단어에 위치 정보를 추가한다.</p>
<p>예를 들어</p>
<pre><code>&gt; 나는 밥을 먹었다.</code></pre><p>과</p>
<pre><code>&gt; 밥이 나를 먹었다.</code></pre><p>는</p>
<p>단어는 거의 같지만</p>
<p>순서가 다르므로 의미가 완전히 달라진다.</p>
<p>Positional Encoding 덕분에</p>
<p>Transformer는 단어의 위치까지 함께 학습할 수 있다.</p>
<hr />
<h3 id="12-masked-self-attention">12. Masked Self-Attention</h3>
<p>Decoder에서는</p>
<p>미래 단어를 보면 안 된다.</p>
<p>예를 들어</p>
<pre><code>나는 오늘 점심을 먹었다.</code></pre><p>에서</p>
<p>&quot;오늘&quot;을 예측할 때</p>
<p>뒤의</p>
<p>&quot;점심을&quot;</p>
<p>을 미리 보면 정답을 알게 된다.</p>
<p>그래서 미래 단어를 <strong>Mask</strong>하여</p>
<p>현재까지의 단어만 보고 다음 단어를 예측하도록 만든다.</p>
<p>이 구조 덕분에 GPT는 자연스럽게 문장을 생성할 수 있다.</p>
<hr />
<h2 id="예시">예시</h2>
<h3 id="예시-1-self-attention">예시 1. Self-Attention</h3>
<p>문장</p>
<pre><code>I study AI hard.</code></pre><p>Self-Attention은</p>
<p>&quot;study&quot;를 이해하기 위해</p>
<ul>
<li>I</li>
<li>AI</li>
<li>hard</li>
</ul>
<p>와의 관련성을 동시에 계산한다.</p>
<p>관련성이 높은 단어에는 높은 Attention Score를 부여한다.</p>
<hr />
<h3 id="예시-2-positional-encoding">예시 2. Positional Encoding</h3>
<p>다음 두 문장을 비교해보자.</p>
<table>
<thead>
<tr>
<th>문장</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>나는 밥을 먹었다.</td>
<td>사람이 밥을 먹음</td>
</tr>
<tr>
<td>밥이 나를 먹었다.</td>
<td>말이 되지 않는 문장</td>
</tr>
</tbody></table>
<p>Transformer는 Positional Encoding을 이용해</p>
<p>단어 순서까지 함께 학습한다.</p>
<hr />
<h3 id="예시-3-multi-head-attention">예시 3. Multi-Head Attention</h3>
<p>회의에서 여러 전문가가 동시에 문서를 검토한다고 생각해보자.</p>
<table>
<thead>
<tr>
<th>전문가</th>
<th>보는 관점</th>
</tr>
</thead>
<tbody><tr>
<td>전문가 1</td>
<td>문법</td>
</tr>
<tr>
<td>전문가 2</td>
<td>의미</td>
</tr>
<tr>
<td>전문가 3</td>
<td>핵심 키워드</td>
</tr>
<tr>
<td>전문가 4</td>
<td>전체 문맥</td>
</tr>
</tbody></table>
<p>각 전문가의 의견을 종합하면</p>
<p>한 사람이 보는 것보다 더 정확한 결론을 얻을 수 있다.</p>
<p>이것이 Multi-Head Attention과 매우 비슷한 방식이다.</p>
<hr />
<h2 id="실무-활용">실무 활용</h2>
<p>Transformer는 현재 거의 모든 최신 NLP 및 생성형 AI 서비스의 기반 기술이다.</p>
<table>
<thead>
<tr>
<th>활용 분야</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>챗봇</td>
<td>자연스러운 질문 이해 및 답변 생성</td>
</tr>
<tr>
<td>문서 요약</td>
<td>긴 보고서, 논문, 회의록 요약</td>
</tr>
<tr>
<td>번역</td>
<td>문맥을 고려한 자연스러운 기계 번역</td>
</tr>
<tr>
<td>검색(RAG)</td>
<td>의미 기반 문서 검색 및 답변 생성</td>
</tr>
<tr>
<td>감성 분석</td>
<td>리뷰나 고객 의견의 긍정·부정 판단</td>
</tr>
<tr>
<td>문서 분류</td>
<td>뉴스, 이메일, 문의 유형 자동 분류</td>
</tr>
<tr>
<td>코드 생성</td>
<td>자연어를 Python, Java 등 코드로 변환</td>
</tr>
<tr>
<td>AI Agent</td>
<td>계획 수립, 추론, 도구 호출 등 에이전트의 두뇌 역할</td>
</tr>
</tbody></table>
<p>데이터 분석 프로젝트에서는 Transformer를 처음부터 학습하기보다 <strong>사전학습된 모델(Pre-trained Model)</strong> 을 활용하는 경우가 대부분이다.</p>
<p>대표적인 모델은</p>
<ul>
<li>BERT</li>
<li>GPT</li>
<li>Sentence Transformer</li>
<li>RoBERTa</li>
<li>T5</li>
</ul>
<p>등이 있다.</p>
<p>예를 들어 고객 문의 데이터를 분석하는 경우</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>활용 방식</th>
</tr>
</thead>
<tbody><tr>
<td>데이터 수집</td>
<td>고객 문의, 상담 로그, 리뷰 수집</td>
</tr>
<tr>
<td>임베딩 생성</td>
<td>문장을 의미 벡터로 변환</td>
</tr>
<tr>
<td>유사도 검색</td>
<td>비슷한 문의 자동 그룹화</td>
</tr>
<tr>
<td>자동 분류</td>
<td>환불, 배송, 장애 등으로 분류</td>
</tr>
<tr>
<td>요약</td>
<td>문의 내용을 한 줄로 요약</td>
</tr>
<tr>
<td>답변 생성</td>
<td>FAQ와 매뉴얼을 참고하여 답변 생성</td>
</tr>
</tbody></table>
<hr />
<h2 id="핵심-정리">핵심 정리</h2>
<table>
<thead>
<tr>
<th>핵심 개념</th>
<th>쉬운 설명</th>
<th>왜 중요한가</th>
</tr>
</thead>
<tbody><tr>
<td>Transformer</td>
<td>Attention 기반의 딥러닝 언어 모델 구조</td>
<td>GPT, BERT, LLM의 기반</td>
</tr>
<tr>
<td>Attention</td>
<td>중요한 단어에 더 집중하는 메커니즘</td>
<td>긴 문맥을 효과적으로 이해</td>
</tr>
<tr>
<td>Self-Attention</td>
<td>문장 내부 단어 간 관계를 계산</td>
<td>문맥과 의미를 정확하게 파악</td>
</tr>
<tr>
<td>Query</td>
<td>현재 단어가 찾고 싶은 정보</td>
<td>Attention 계산의 기준</td>
</tr>
<tr>
<td>Key</td>
<td>비교를 위한 기준 정보</td>
<td>Query와 관련성을 계산</td>
</tr>
<tr>
<td>Value</td>
<td>실제 의미 정보</td>
<td>최종 문맥 표현 생성</td>
</tr>
<tr>
<td>Scaled Dot-Product Attention</td>
<td>Q, K, V를 이용한 Attention 계산 방식</td>
<td>Transformer의 핵심 연산</td>
</tr>
<tr>
<td>Softmax</td>
<td>Attention Score를 확률로 변환</td>
<td>중요한 단어에 집중</td>
</tr>
<tr>
<td>Multi-Head Attention</td>
<td>여러 관점에서 동시에 Attention 수행</td>
<td>다양한 의미를 함께 학습</td>
</tr>
<tr>
<td>Linear Layer</td>
<td>여러 Head 결과를 하나로 통합</td>
<td>풍부한 표현 생성</td>
</tr>
<tr>
<td>Residual Connection</td>
<td>입력을 출력에 다시 더하는 구조</td>
<td>정보 손실과 Gradient Vanishing 완화</td>
</tr>
<tr>
<td>Layer Normalization</td>
<td>출력 분포를 일정하게 유지</td>
<td>안정적인 학습</td>
</tr>
<tr>
<td>Feed Forward Network</td>
<td>Attention 결과를 비선형 변환</td>
<td>표현력 향상</td>
</tr>
<tr>
<td>Positional Encoding</td>
<td>단어 위치 정보 추가</td>
<td>단어 순서를 이해</td>
</tr>
<tr>
<td>Encoder</td>
<td>입력 문장을 이해</td>
<td>BERT 기반 모델</td>
</tr>
<tr>
<td>Decoder</td>
<td>다음 단어 생성</td>
<td>GPT 기반 모델</td>
</tr>
<tr>
<td>Masked Self-Attention</td>
<td>미래 단어를 가려 다음 단어 예측</td>
<td>생성형 AI의 핵심</td>
</tr>
</tbody></table>
<hr />
<h2 id="흐름으로-이해하기">흐름으로 이해하기</h2>
<pre><code>입력 문장
      │
      ▼
Embedding
      │
      ▼
Positional Encoding
      │
      ▼
Multi-Head Self-Attention
      │
      ▼
Residual Connection + LayerNorm
      │
      ▼
Feed Forward Network
      │
      ▼
Residual Connection + LayerNorm
      │
      ▼
(Encoder 여러 층 반복)
      │
      ▼
Decoder
      │
      ▼
Masked Self-Attention
      │
      ▼
다음 토큰 예측
      │
      ▼
최종 문장 생성</code></pre><hr />
<h2 id="한-줄-요약">한 줄 요약</h2>
<blockquote>
<p><strong>Transformer는 Self-Attention을 이용해 문장 속 모든 단어의 관계를 동시에 이해하고, 문맥을 효과적으로 학습하여 GPT, BERT, LLM, RAG, 챗봇 등 현대 생성형 AI를 가능하게 만든 핵심 딥러닝 아키텍처이다.</strong></p>
</blockquote>