<h3 id="--transformer-입력-데이터에서-중요한-정보의-관계를-어텐션attention-으로-학습하는-딥러닝-신경망-구조이다">- <strong>Transformer</strong>: 입력 데이터에서 중요한 정보의 관계를 <strong>어텐션(Attention)</strong> 으로 학습하는 딥러닝 신경망 구조이다.</h3>
<h3 id="--llm-large-language-model-방대한-텍스트를-학습해-사람처럼-언어를-이해하고-생성하는-대규모-ai-언어-모델이다">- <strong>LLM (Large Language Model)</strong>: 방대한 텍스트를 학습해 <strong>사람처럼 언어를 이해하고 생성하는 대규모 AI 언어 모델</strong>이다.</h3>
<h3 id="--어텐션attention-문장을-처리할-때-모든-단어를-똑같이-보지-않고-현재-단어와-가장-관련-있는-단어에-더-집중하는-메커니즘이다">- <strong>어텐션(Attention)</strong>: 문장을 처리할 때 모든 단어를 똑같이 보지 않고, 현재 단어와 가장 관련 있는 단어에 더 집중하는 메커니즘이다.</h3>
<h1 id="llm의-한계">LLM의 한계</h1>
<h2 id="1-black-box-블랙박스">1. Black Box (블랙박스)</h2>
<h3 id="개념">개념</h3>
<ul>
<li>LLM은 <strong>수십억 개의 파라미터</strong>를 기반으로 동작하기 때문에, 어떤 과정을 거쳐 특정 답변을 생성했는지 설명하기 어렵다.</li>
<li>Software 2.0의 한계가 그대로 이어진다.</li>
</ul>
<h3 id="특징">특징</h3>
<ul>
<li>특정 답변이 나온 이유를 명확하게 설명하기 어려움</li>
<li>잘못된 결과가 발생해도 원인을 추적하기 쉽지 않음</li>
<li>모델 내부 의사결정 과정을 사람이 직접 해석하기 어려움</li>
</ul>
<h3 id="예시">예시</h3>
<blockquote>
<p>AI가 대출을 거절했지만, 왜 거절했는지 정확한 근거를 설명하지 못하는 경우</p>
</blockquote>
<hr />
<h2 id="2-bias-편향">2. Bias (편향)</h2>
<h3 id="개념-1">개념</h3>
<ul>
<li>LLM은 인터넷, 책, 뉴스 등 <strong>웹에서 수집한 방대한 데이터</strong>를 학습한다.</li>
<li>따라서 데이터에 존재하는 편향까지 함께 학습하게 된다.</li>
</ul>
<h3 id="학습되는-편향의-예">학습되는 편향의 예</h3>
<ul>
<li>데이터 편향(Data Bias)</li>
<li>문화적 편향(Cultural Bias)</li>
<li>오래된 정보(Outdated Information)</li>
<li>잘못된 정보(Misinformation)</li>
</ul>
<h3 id="특징-1">특징</h3>
<ul>
<li>Prompt를 잘 작성하더라도 학습된 편향을 완전히 제거할 수는 없다.</li>
<li>겉으로는 자연스러운 답변을 하지만 잘못된 정보를 포함하는 <strong>Silent Failure</strong>가 발생할 수 있다.</li>
</ul>
<h3 id="예시-1">예시</h3>
<blockquote>
<p>특정 직업을 특정 성별과 연결하여 답변하거나, 오래된 정보를 최신 정보처럼 설명하는 경우</p>
</blockquote>
<hr />
<h2 id="3-hallucination-환각">3. Hallucination (환각)</h2>
<h3 id="개념-2">개념</h3>
<ul>
<li>LLM은 <strong>정답을 찾는 모델이 아니라 다음 토큰을 가장 그럴듯하게 예측하는 모델</strong>이다.</li>
<li>따라서 실제 사실이 아닌 내용을 매우 자연스럽게 생성할 수 있다.</li>
</ul>
<h3 id="특징-2">특징</h3>
<ul>
<li>존재하지 않는 논문 생성</li>
<li>가짜 출처 생성</li>
<li>잘못된 수치 제시</li>
<li>틀린 코드 작성</li>
<li>사실이 아닌 내용을 확신에 찬 표현으로 설명</li>
</ul>
<h3 id="예시-2">예시</h3>
<blockquote>
<p>실제 존재하지 않는 논문을 APA 형식까지 갖춰 인용하는 경우</p>
</blockquote>
<ul>
<li><strong>휴먼 인 더 루프(HITL, Human-in-the-Loop)</strong>는 인공지능(AI)이나 자동화 시스템의 학습, 의사결정, 감독 과정에 사람이 적극적으로 개입하여 시스템의 정확성, 안전성, 윤리성을 확보하는 협업 프로세스 및 설계 철학입니다</li>
</ul>
<hr />
<h2 id="4-non-deterministic-비결정성">4. Non-deterministic (비결정성)</h2>
<h3 id="개념-3">개념</h3>
<ul>
<li>같은 Prompt와 동일한 입력이라도 항상 동일한 결과를 생성하지 않는다.</li>
</ul>
<h3 id="특징-3">특징</h3>
<ul>
<li>실행할 때마다 표현이나 내용이 조금씩 달라질 수 있음</li>
<li>일반 프로그램처럼 항상 같은 입력에 같은 출력이 보장되지 않음</li>
<li>Temperature 등의 설정에 따라 결과 차이가 더욱 커질 수 있음</li>
</ul>
<h3 id="예시-3">예시</h3>
<p>같은 질문을 여러 번 입력하면</p>
<pre><code>질문: Transformer를 설명해줘.</code></pre><p>1번째 답변</p>
<blockquote>
<p>Transformer는 Attention 기반의 딥러닝 모델입니다.</p>
</blockquote>
<p>2번째 답변</p>
<blockquote>
<p>Transformer는 Self-Attention을 활용하여 문맥을 학습하는 신경망 구조입니다.</p>
</blockquote>
<p>→ 의미는 비슷하지만 표현과 설명 방식이 달라질 수 있다.</p>
<hr />
<h1 id="한눈에-정리">한눈에 정리</h1>
<table>
<thead>
<tr>
<th>한계</th>
<th>핵심 내용</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Black Box</strong></td>
<td>왜 그런 답을 생성했는지 설명하기 어려움</td>
</tr>
<tr>
<td><strong>Bias</strong></td>
<td>학습 데이터의 편향을 그대로 학습함</td>
</tr>
<tr>
<td><strong>Hallucination</strong></td>
<td>사실이 아닌 내용을 그럴듯하게 생성함</td>
</tr>
<tr>
<td><strong>Non-deterministic</strong></td>
<td>같은 입력이라도 항상 같은 출력이 나오지 않음</td>
</tr>
</tbody></table>