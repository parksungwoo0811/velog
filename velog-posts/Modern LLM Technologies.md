<h2 id="개념">개념</h2>
<h3 id="1-rlhf">1. RLHF</h3>
<p><strong>RLHF(Reinforcement Learning from Human Feedback)는 사람이 선호하는 답변을 보상(Reward)으로 학습하여, LLM이 더 유용하고 안전하며 자연스러운 답변을 생성하도록 만드는 강화학습 기법이다.</strong></p>
<p>LLM은 기본적으로 “다음 단어를 잘 예측하는 모델”입니다. 그래서 사전학습만으로는 지식은 많아도 답변이 무례하거나, 너무 장황하거나, 사용자의 의도와 어긋날 수 있습니다.</p>
<p>RLHF는 모델을 단순히 “정답을 맞히는 기계”에서 “사람이 선호하는 방식으로 답하는 AI”로 맞추는 과정입니다.</p>
<p>핵심 흐름은 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Supervised Fine-Tuning</td>
<td>사람이 만든 모범 답변으로 모델을 먼저 학습</td>
</tr>
<tr>
<td>Reward Model</td>
<td>여러 답변 중 사람이 더 선호하는 답변에 높은 점수를 주는 모델 학습</td>
</tr>
<tr>
<td>Reinforcement Learning</td>
<td>보상 점수가 높아지는 방향으로 LLM을 추가 학습</td>
</tr>
</tbody></table>
<p>단점도 있습니다. 사람이 좋아할 만한 말투만 과하게 학습하면 모델이 지나치게 아첨하거나, 비판 없이 동조하는 문제가 생길 수 있습니다.</p>
<p>즉, 사람의 평가 데이터 품질이 낮거나 편향되어 있으면
그 편향까지 함께 학습할 수 있다.</p>
<h3 id="2-scaling-law">2. Scaling Law</h3>
<p>*<em>Scaling Law는
*</em>
모델 크기(Parameter),
학습 데이터(Token),
연산량(Compute)</p>
<p>세 요소가 증가하면
*<em>모델 성능도 일정한 패턴으로 향상된다는 경험적 법칙이다.
*</em>
LLM 성능은 크게 세 가지에 의해 결정됩니다.</p>
<table>
<thead>
<tr>
<th>요소</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Parameter</td>
<td>모델 안에 있는 학습 가능한 값의 개수</td>
</tr>
<tr>
<td>Data</td>
<td>학습에 사용하는 토큰 수</td>
</tr>
<tr>
<td>Compute</td>
<td>학습에 투입되는 연산량</td>
</tr>
</tbody></table>
<p>초기에는 “모델을 크게 만들수록 더 똑똑해진다”는 흐름이 강했습니다. GPT-2에서 GPT-3로 넘어가며 모델 크기와 데이터 규모가 커졌고, 성능도 크게 향상되었습니다.</p>
<p>하지만 <code>Chinchilla Scaling Law</code>는 무조건 모델만 키우는 것이 답은 아니라고 말합니다. 모델 크기와 데이터 양, 연산량의 균형이 중요합니다. 큰 모델에 데이터가 부족하면 오히려 비효율적일 수 있습니다.</p>
<p>같은 연산량이라면 작은 모델에 더 많은 데이터를 학습시키는 것이 큰 모델에
적은 데이터를 학습시키는 것보다 더 효율적이라는 사실을 보여주었다.</p>
<h3 id="3-knowledge-distillation">3. Knowledge Distillation</h3>
<p><code>Knowledge Distillation</code>, 즉 지식 증류는 큰 모델의 능력을 작은 모델에게 전달하는 방법입니다.</p>
<ul>
<li>큰 모델(Teacher)이 출력한 확률 분포와 추론 결과를 이용하여 작은 모델(Student)이 Teacher의 예측 패턴을 학습하는 기법이다.</li>
</ul>
<p>큰 모델은 성능이 좋지만 학습과 추론 비용이 큽니다. 모든 회사가 수천 개 GPU를 사용해 거대 모델을 직접 학습시키기는 어렵습니다. 그래서 강력한 Teacher Model이 만든 답변, 추론 과정, 설명 데이터를 이용해 작은 Student Model을 학습시킵니다.</p>
<p>핵심은 단순 압축이 아닙니다.<br />큰 모델의 “생각하는 방식”을 작은 모델에게 전달하는 과정입니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Teacher Model</td>
<td>성능이 뛰어난 큰 모델</td>
</tr>
<tr>
<td>Student Model</td>
<td>더 작고 빠른 모델</td>
</tr>
<tr>
<td>Reasoning Data</td>
<td>Teacher가 만든 추론 과정, 풀이, 설명</td>
</tr>
<tr>
<td>Soft Label</td>
<td>정답 하나만이 아니라 가능성 분포(얼마나 다른 답도 가능성이 있는지 까지 함께 알려주는 학습 데이터</td>
</tr>
</tbody></table>
<p>예를 들어 <code>고양이=1, 개=0</code>처럼 딱 잘라 알려주는 것이 <code>Hard Label</code>이라면, <code>고양이 0.7, 개 0.2, 여우 0.1</code>처럼 유사성 정보까지 알려주는 것이 <code>Soft Label</code>입니다. Student Model은 Soft Label을 통해 더 부드럽고 일반화된 방식으로 학습할 수 있습니다.</p>
<h3 id="4-moe">4. MoE</h3>
<p><code>MoE</code>는 <code>Mixture of Experts</code>의 약자입니다.<br />모든 파라미터를 매번 다 쓰는 대신, 입력에 맞는 전문가 일부만 활성화하는 구조입니다.</p>
<p>일반적인 Dense Transformer는 모든 토큰에 대해 모든 파라미터를 사용합니다. 반면 MoE는 Router가 입력 토큰을 보고 어떤 Expert를 사용할지 고릅니다.</p>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Router</td>
<td>어떤 Expert를 쓸지 결정하는 작은 신경망</td>
</tr>
<tr>
<td>Expert</td>
<td>특정 유형의 입력을 처리하는 전문 모듈</td>
</tr>
<tr>
<td>Gating</td>
<td>Expert별 선택 확률을 계산하는 과정</td>
</tr>
<tr>
<td>Top-k Selection</td>
<td>여러 Expert 중 일부만 활성화하는 방식</td>
</tr>
</tbody></table>
<p>예를 들어 어떤 토큰이 수학 문제와 관련 있으면 수학에 강한 Expert가 선택되고, 코드와 관련 있으면 코드에 강한 Expert가 선택되는 식입니다.</p>
<p>MoE의 장점은 모델 전체 크기는 크게 유지하면서도 실제 계산에는 일부 Expert만(모델 전체는 매우 크지만 추론 시에Top-k Expert만 활성화) 사용한다는 점입니다. 그래서 성능과 비용 효율을 동시에 노릴 수 있습니다.</p>
<hr />
<blockquote>
<pre><code class="language-__">Pretraining
      │
      ▼
Scaling Law
(큰 모델 학습)
      │
      ▼
    RLHF
(사람처럼 답하기)
      │
      ▼
Knowledge Distillation
(작은 모델 만들기)
      │
      ▼
    MoE
(필요한 Expert만 사용)
      │
      ▼
효율적인 최신 LLM</code></pre>
</blockquote>
<p><del>~</del></p>
<h2 id="예시">예시</h2>
<h3 id="rlhf-예시">RLHF 예시</h3>
<p>질문: “파이썬에서 리스트를 뒤집는 방법을 알려줘.”</p>
<p>나쁜 답변:</p>
<pre><code class="language-text">reverse() 쓰면 됩니다.</code></pre>
<p>더 선호되는 답변:</p>
<pre><code class="language-text">리스트를 뒤집는 방법은 두 가지가 있습니다.

1. reverse(): 원본 리스트를 직접 뒤집습니다.
2. a[::-1]: 뒤집힌 새 리스트를 만듭니다.

상황에 따라 적절한 방법을 선택하면 됩니다.</code></pre>
<p>RLHF는 두 번째처럼 친절하고 실용적인 답변을 더 선호하도록 모델을 훈련합니다.</p>
<h3 id="scaling-law-예시">Scaling Law 예시</h3>
<p>학생이 공부를 잘하려면 세 가지가 필요합니다.</p>
<table>
<thead>
<tr>
<th>LLM 요소</th>
<th>학생 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Parameter</td>
<td>머리 용량</td>
</tr>
<tr>
<td>Data</td>
<td>공부한 책의 양</td>
</tr>
<tr>
<td>Compute</td>
<td>공부한 시간과 에너지</td>
</tr>
</tbody></table>
<p>머리만 좋고 공부한 책이 적으면 부족합니다.<br />책만 많고 공부할 시간이 부족해도 부족합니다.<br />*<em>Scaling Law의 핵심은 “크기, 데이터, 연산량을 균형 있게 키워야 한다”는 것입니다.
*</em></p>
<h3 id="distillation-예시">Distillation 예시</h3>
<p>유명한 수학 선생님이 어려운 문제를 푸는 과정을 자세히 설명합니다.<br />그 설명을 듣고 작은 AI 튜터가 풀이 방식을 배웁니다.</p>
<p>이때 작은 AI는 단순히 정답만 외우는 것이 아니라,</p>
<pre><code class="language-text">문제를 어떻게 읽는지
중간 과정을 어떻게 세우는지
어떤 실수를 피해야 하는지</code></pre>
<p>까지 배웁니다. 이것이 Knowledge Distillation입니다.</p>
<h3 id="moe-예시">MoE 예시</h3>
<p>회사에 여러 전문가가 있다고 생각하면 됩니다.</p>
<table>
<thead>
<tr>
<th>질문 유형</th>
<th>선택되는 전문가</th>
</tr>
</thead>
<tbody><tr>
<td>법률 질문</td>
<td>법률 Expert</td>
</tr>
<tr>
<td>코드 질문</td>
<td>개발 Expert</td>
</tr>
<tr>
<td>수학 문제</td>
<td>수학 Expert</td>
</tr>
<tr>
<td>글쓰기 요청</td>
<td>문장 Expert</td>
</tr>
</tbody></table>
<p>모든 전문가가 매번 회의에 들어오면 비용이 큽니다.<br />MoE는 질문에 맞는 전문가 몇 명만 불러서 효율적으로 답변하는 구조입니다.</p>
<h2 id="실무-활용">실무 활용</h2>
<table>
<thead>
<tr>
<th>기술</th>
<th>실무 활용</th>
</tr>
</thead>
<tbody><tr>
<td>RLHF</td>
<td>챗봇 답변 품질 개선, 안전한 응답 생성, 고객 응대 톤 조정</td>
</tr>
<tr>
<td>Scaling Law</td>
<td>모델 학습 예산 설계, 데이터와 GPU 자원 배분, 모델 크기 결정</td>
</tr>
<tr>
<td>Knowledge Distillation</td>
<td>큰 모델을 작은 모델로 경량화, 모바일/사내 서버 배포, 추론 비용 절감</td>
</tr>
<tr>
<td>MoE</td>
<td>대규모 모델의 계산 비용 절감, 전문 영역별 성능 강화, 빠른 추론 구현</td>
</tr>
</tbody></table>
<p>실제 AI 프로젝트에서는 네 기술이 함께 쓰입니다.</p>
<p>예를 들어 기업용 AI 챗봇을 만든다면 다음처럼 활용할 수 있습니다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>적용 기술</th>
</tr>
</thead>
<tbody><tr>
<td>기본 LLM 선정</td>
<td>Scaling Law를 고려해 성능과 비용 균형 확인</td>
</tr>
<tr>
<td>답변 품질 개선</td>
<td>RLHF 또는 선호 데이터 기반 튜닝</td>
</tr>
<tr>
<td>사내 배포 최적화</td>
<td>Distillation으로 작은 모델 생성</td>
</tr>
<tr>
<td>대규모 처리 효율화</td>
<td>MoE 구조로 필요한 전문가만 활성화</td>
</tr>
</tbody></table>
<h2 id="핵심-정리">핵심 정리</h2>
<table>
<thead>
<tr>
<th>파트</th>
<th>핵심 개념</th>
<th>쉽게 말하면</th>
<th>장점</th>
<th>주의점</th>
</tr>
</thead>
<tbody><tr>
<td>RLHF</td>
<td>사람 피드백 기반 강화학습</td>
<td>사람이 좋아하는 답변을 배우게 함</td>
<td>답변 품질, 친절함, 안전성 향상</td>
<td>과한 아첨, 편향, 비용 문제</td>
</tr>
<tr>
<td>Scaling Law</td>
<td>모델, 데이터, 연산량 확장 법칙</td>
<td>크게 잘 키우면 성능이 좋아짐</td>
<td>성능 향상 예측 가능</td>
<td>무조건 큰 모델이 답은 아님</td>
</tr>
<tr>
<td>Chinchilla Law</td>
<td>모델과 데이터의 균형</td>
<td>모델 크기와 학습 데이터 비율이 중요</td>
<td>학습 효율 향상</td>
<td>데이터 부족한 대형 모델은 비효율</td>
</tr>
<tr>
<td>Distillation</td>
<td>큰 모델 지식 전달</td>
<td>큰 선생님이 작은 학생을 가르침</td>
<td>비용 절감, 빠른 추론, 경량화</td>
<td>Teacher 품질에 의존</td>
</tr>
<tr>
<td>Soft Label</td>
<td>확률 분포 기반 학습</td>
<td>정답뿐 아니라 유사성도 배움</td>
<td>일반화 성능 향상</td>
<td>학습 데이터 설계가 중요</td>
</tr>
<tr>
<td>MoE</td>
<td>전문가 혼합 구조</td>
<td>필요한 전문가만 불러서 계산</td>
<td>큰 모델 성능과 비용 효율 동시 확보</td>
<td>Router 학습과 Expert 균형이 어려움</td>
</tr>
<tr>
<td>Router</td>
<td>Expert 선택기</td>
<td>입력에 맞는 전문가를 고름</td>
<td>계산량 절감</td>
<td>특정 Expert에 쏠릴 수 있음</td>
</tr>
</tbody></table>
<h2 id="한-줄-요약">한 줄 요약</h2>
<p>Modern LLM Technologies는 LLM을 더 사람답게 만들고, 더 크게 학습시키며, 더 작게 배포하고, 더 효율적으로 계산하기 위한 핵심 기술들의 묶음입니다.</p>