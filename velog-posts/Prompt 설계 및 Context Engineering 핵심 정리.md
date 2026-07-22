<blockquote>
<p><strong>학습일:</strong> 2026.07.15
<strong>주제:</strong> Prompt Engineering &amp; Context Engineering</p>
</blockquote>
<hr />
<h1 id="오늘의-메모">오늘의 메모</h1>
<p>** AI 엔지니어링 간단 정리**</p>
<ol>
<li><strong>Prompt Engineering</strong><ul>
<li>의도를 모델에게 정확히 전달한다 / 어떻게 말해야 하는가</li>
<li>Persona, few-shot, CoT, 구조화(Markdown, JSON) 등</li>
</ul>
</li>
<li><strong>Context Engineering</strong><ul>
<li>판단에 필요한 자료를 주입한다 / 무엇을, 얼마나 제공할 것인가</li>
<li>RAG, Memory, 문서 주입, 컨텍스트 윈도우 관리 등</li>
</ul>
</li>
<li><strong>Harness Engineering</strong><ul>
<li>모델이 작업할 환경과 권한을 제어한다 / 어떤 환경에서 작업하게 할 것인가</li>
<li>Tools, MCP, Skills, Agents, 권한 설계 등</li>
</ul>
</li>
<li><strong>Loop Engineering</strong><ul>
<li>실행 &gt;&gt; 검증 &gt;&gt; 수정이 반복하도록 설계한다 / 언제 멈추고, 어디서 사람이 개입하는가</li>
<li>테스트/자동 검증, Hooks, Eval, 종료 조건, HITL(Human in the loop) 등</li>
</ul>
</li>
</ol>
<p><strong>정합성(Consistency):</strong> 데이터나 시스템 등의 상태가 서로 <strong>모순 없이 일관되게</strong> 일치하는 것</p>
<p><strong>무결성(Integrity):</strong> 정보나 데이터가 <strong>정확하고 완전한</strong> 상태로 유지되는 것</p>
<h1 id="오늘-학습한-내용">오늘 학습한 내용</h1>
<p>오늘 학습에서는 단순히 프롬프트를 잘 작성하는 것이 아니라 <strong>LLM이 더 정확한 결과를 생성하도록 정보를 설계하는 방법(Context Engineering)</strong> 을 중심으로 학습했다.</p>
<p>프롬프트는 AI에게 질문을 던지는 것이 아니라 <strong>문제를 해결하기 위한 환경(Context)을 설계하는 과정</strong>이라는 점이 가장 중요한 내용이었다.</p>
<hr />
<h1 id="1-prompt-engineering과-context-engineering의-차이">1. Prompt Engineering과 Context Engineering의 차이</h1>
<h2 id="prompt-engineering">Prompt Engineering</h2>
<ul>
<li>AI에게 원하는 작업을 수행하도록 프롬프트를 작성하는 기술</li>
<li>질문의 표현이나 역할(Role), 출력 형식을 조정하여 결과를 개선</li>
</ul>
<p>예시</p>
<pre><code class="language-text">너는 데이터 분석 전문가다.
다음 데이터를 분석하고 표 형태로 정리해줘.</code></pre>
<hr />
<h2 id="context-engineering">Context Engineering</h2>
<p>Prompt보다 더 넓은 개념이다.</p>
<p>AI가 올바른 답을 생성하기 위해 필요한 <strong>모든 정보와 환경을 설계하는 과정</strong>이다.</p>
<p>즉,</p>
<blockquote>
<p>Prompt는 Context의 일부일 뿐이다.</p>
</blockquote>
<p>Context에는 다음과 같은 요소들이 포함된다.</p>
<ul>
<li>시스템 프롬프트</li>
<li>사용자 입력</li>
<li>이전 대화(History)</li>
<li>메모리</li>
<li>외부 문서(RAG)</li>
<li>Tool 사용 결과</li>
<li>출력 형식</li>
</ul>
<p>즉 AI에게</p>
<blockquote>
<p>&quot;무엇을 말할 것인가?&quot;</p>
</blockquote>
<p>보다</p>
<blockquote>
<p>&quot;무엇을 알고 답해야 하는가?&quot;</p>
</blockquote>
<p>를 설계하는 것이 Context Engineering이다.</p>
<hr />
<h1 id="2-좋은-context의-구성-요소">2. 좋은 Context의 구성 요소</h1>
<p>효율적인 Context는 다음 요소들로 구성된다.</p>
<ul>
<li>역할(Role)</li>
<li>목표(Task)</li>
<li>필요한 배경 정보(Context)</li>
<li>제약 조건(Constraint)</li>
<li>출력 형식(Output Format)</li>
<li>예시(Few-shot)</li>
</ul>
<p>이 요소들이 명확할수록 AI의 응답 품질이 높아진다.</p>
<hr />
<h1 id="3-prompt보다-중요한-것은-context">3. Prompt보다 중요한 것은 Context</h1>
<p>같은 질문이라도 Context가 달라지면 결과가 크게 달라진다.</p>
<p>예를 들어</p>
<pre><code class="language-text">파이썬 알려줘</code></pre>
<p>보다</p>
<pre><code class="language-text">나는 Python 초보자다.

for문을 처음 배우고 있다.

예제를 이용해서 쉽게 설명해줘.

답변은 단계별로 작성해줘.</code></pre>
<p>처럼 필요한 정보를 충분히 제공하는 것이 훨씬 좋은 결과를 만든다.</p>
<hr />
<h1 id="4-context를-구성하는-대표-요소">4. Context를 구성하는 대표 요소</h1>
<h2 id="①-memory">① Memory</h2>
<p>이전 대화나 사용자 정보를 기억하여 답변에 반영</p>
<p>예)</p>
<ul>
<li>사용자의 직업</li>
<li>관심 분야</li>
<li>선호하는 답변 스타일</li>
</ul>
<hr />
<h2 id="②-history">② History</h2>
<p>직전 대화 내용을 이어받아 문맥을 유지</p>
<p>예)</p>
<pre><code>사용자 :
이 코드 오류가 뭐야?

↓

사용자 :
그럼 어떻게 수정해?</code></pre><p>History가 없으면 두 번째 질문을 이해하기 어렵다.</p>
<hr />
<h2 id="③-external-knowledge">③ External Knowledge</h2>
<p>모델이 학습하지 않은 최신 정보를 제공</p>
<p>대표적으로</p>
<ul>
<li>PDF</li>
<li>데이터베이스</li>
<li>검색(RAG)</li>
<li>API</li>
</ul>
<p>등이 활용된다.</p>
<hr />
<h2 id="④-tool">④ Tool</h2>
<p>LLM이 직접 수행하지 못하는 작업을 외부 도구가 수행한다.</p>
<p>예)</p>
<ul>
<li>계산기</li>
<li>웹 검색</li>
<li>데이터베이스 조회</li>
<li>Python 실행</li>
<li>API 호출</li>
</ul>
<hr />
<h1 id="5-prompt-설계-원칙">5. Prompt 설계 원칙</h1>
<p>좋은 Prompt는 아래 요소를 최대한 포함한다.</p>
<h3 id="role">Role</h3>
<pre><code>너는 금융 전문가다.</code></pre><hr />
<h3 id="task">Task</h3>
<pre><code>투자 리스크를 분석해줘.</code></pre><hr />
<h3 id="constraint">Constraint</h3>
<pre><code>초보자도 이해할 수 있도록 설명해줘.</code></pre><hr />
<h3 id="output-format">Output Format</h3>
<pre><code>표 형태로 작성해줘.</code></pre><hr />
<h3 id="example">Example</h3>
<pre><code>예시를 하나 포함해줘.</code></pre><hr />
<h1 id="6-markdown-활용">6. Markdown 활용</h1>
<p>LLM이 Markdown을 잘 이해하기 때문에 출력 형식을 지정할 때 매우 유용하다.</p>
<p>대표 문법</p>
<h2 id="제목">제목</h2>
<pre><code class="language-markdown"># 제목
## 소제목
### 세부 제목</code></pre>
<hr />
<h2 id="리스트">리스트</h2>
<pre><code class="language-markdown">- 항목
- 항목</code></pre>
<hr />
<h2 id="번호-리스트">번호 리스트</h2>
<pre><code class="language-markdown">1.
2.
3.</code></pre>
<hr />
<h2 id="코드블록">코드블록</h2>
<pre><code class="language-markdown">```python
print(&quot;Hello&quot;)
```</code></pre>
<hr />
<h2 id="표">표</h2>
<pre><code class="language-markdown">|항목|설명|
|---|---|
|Prompt|질문|
|Context|환경|</code></pre>
<p>Markdown을 활용하면 AI가 원하는 형식으로 답변을 생성하기 쉬워진다.</p>
<hr />
<h1 id="7-ai-프로젝트에서-context-engineering">7. AI 프로젝트에서 Context Engineering</h1>
<p>실제 AI 프로젝트에서는 Prompt 하나만 관리하지 않는다.</p>
<p>다음과 같은 구조로 관리하는 경우가 많다.</p>
<pre><code>project/
├── AGENTS.md
├── docs/
├── templates/
├── outputs/
├── logs/</code></pre><p>각 폴더는 프롬프트, 규칙, 템플릿, 생성 결과, 수정 이력 등을 관리하여 AI가 일관된 결과를 생성하도록 돕는다.</p>
<hr />
<h1 id="추가로-알아두면-좋은-개념들">추가로 알아두면 좋은 개념들</h1>
<hr />
<h1 id="1-token토큰">1. Token(토큰)</h1>
<p>LLM은 문장을 한 글자씩 읽는 것이 아니라 <strong>Token 단위</strong>로 처리한다.</p>
<p>토큰은 단어나 문장의 일부를 의미하며, 영어와 한국어에서 토큰 개수가 다르게 계산된다.</p>
<p>예를 들어</p>
<pre><code class="language-text">안녕하세요.</code></pre>
<p>와</p>
<pre><code class="language-text">Hello.</code></pre>
<p>는 글자 수는 비슷하지만 토큰 수는 다를 수 있다.</p>
<hr />
<h2 id="왜-중요한가">왜 중요한가?</h2>
<p>모델마다 사용할 수 있는 최대 Context Window가 존재한다.</p>
<p>예를 들어</p>
<ul>
<li>GPT-4o</li>
<li>Claude</li>
<li>Gemini</li>
</ul>
<p>모두 사용할 수 있는 최대 Token 개수가 정해져 있다.</p>
<p>Token을 초과하면</p>
<ul>
<li>앞부분이 잘리거나</li>
<li>중요한 정보가 제거되거나</li>
<li>비용이 증가할 수 있다.</li>
</ul>
<p>즉,</p>
<blockquote>
<p>Context를 길게 넣는 것보다 <strong>필요한 정보만 효율적으로 넣는 것</strong>이 더 중요하다.</p>
</blockquote>
<hr />
<h1 id="2-context-window">2. Context Window</h1>
<p>Context Window는</p>
<blockquote>
<p>AI가 한 번에 기억하고 처리할 수 있는 정보의 최대 크기</p>
</blockquote>
<p>를 의미한다.</p>
<p>여기에는</p>
<ul>
<li>시스템 프롬프트</li>
<li>사용자 입력</li>
<li>이전 대화</li>
<li>RAG 검색 결과</li>
<li>AI의 답변</li>
</ul>
<p>모두 포함된다.</p>
<p>즉,</p>
<pre><code>System Prompt
+
History
+
User Prompt
+
Retrieved Documents
+
AI Response</code></pre><p>전부 Context Window를 차지한다.</p>
<hr />
<h1 id="3-hallucination환각">3. Hallucination(환각)</h1>
<p>Hallucination은</p>
<blockquote>
<p>AI가 사실이 아닌 내용을 마치 사실처럼 생성하는 현상</p>
</blockquote>
<p>이다.</p>
<p>예를 들어</p>
<ul>
<li>존재하지 않는 논문</li>
<li>가짜 API</li>
<li>없는 함수</li>
<li>잘못된 통계</li>
</ul>
<p>등을 자연스럽게 만들어낼 수 있다.</p>
<hr />
<h2 id="hallucination을-줄이는-방법">Hallucination을 줄이는 방법</h2>
<ul>
<li>충분한 Context 제공</li>
<li>최신 자료(RAG) 사용</li>
<li>출처를 함께 요청</li>
<li>AI가 모르면 모른다고 답하도록 지시</li>
</ul>
<p>예시</p>
<pre><code class="language-text">모르는 내용은 추측하지 말고
&quot;정보가 부족하다&quot;고 답변해.</code></pre>
<hr />
<h1 id="4-few-shot-prompting">4. Few-shot Prompting</h1>
<p>Few-shot Prompting은</p>
<blockquote>
<p>원하는 답변의 예시를 함께 제공하는 기법</p>
</blockquote>
<p>이다.</p>
<p>예를 들어</p>
<pre><code class="language-text">질문

사과

출력

과일

질문

자동차

출력

탈것

질문

고양이

출력</code></pre>
<p>처럼 예시를 먼저 보여주면</p>
<p>AI는 동일한 패턴으로 답변을 생성한다.</p>
<hr />
<h1 id="5-zero-shot--one-shot--few-shot">5. Zero-shot / One-shot / Few-shot</h1>
<h3 id="zero-shot">Zero-shot</h3>
<p>예시 없이 바로 요청</p>
<pre><code class="language-text">감정을 분석해줘.</code></pre>
<hr />
<h3 id="one-shot">One-shot</h3>
<p>예시 1개 제공</p>
<pre><code class="language-text">행복합니다 → 긍정

슬퍼요 →</code></pre>
<hr />
<h3 id="few-shot">Few-shot</h3>
<p>여러 개의 예시 제공</p>
<p>예시가 많을수록 원하는 결과를 얻기 쉽다.</p>
<hr />
<h1 id="6-chain-of-thoughtcot">6. Chain of Thought(CoT)</h1>
<p>복잡한 문제는</p>
<blockquote>
<p>한 번에 답을 요구하는 것보다</p>
</blockquote>
<p>생각 과정을 단계적으로 수행하도록 하면 정확도가 높아진다.</p>
<p>예를 들어</p>
<pre><code class="language-text">문제를 해결하기 전에

1. 문제를 분석하고

2. 필요한 정보를 정리한 후

3. 결론을 내려줘.</code></pre>
<p>처럼 단계를 나누면 더 안정적인 답변을 얻을 수 있다.</p>
<hr />
<h1 id="7-structured-output">7. Structured Output</h1>
<p>LLM은 출력 형식을 지정할수록 결과가 일정해진다.</p>
<p>예를 들어</p>
<pre><code class="language-text">JSON으로 출력해줘.</code></pre>
<p>또는</p>
<pre><code class="language-text">다음 형식으로 작성해.

제목

요약

장점

단점

결론</code></pre>
<p>처럼 형식을 지정하면 후처리도 쉬워진다.</p>
<hr />
<h1 id="8-ragretrieval-augmented-generation">8. RAG(Retrieval-Augmented Generation)</h1>
<p>RAG는</p>
<blockquote>
<p>외부 문서를 검색한 뒤 그 내용을 기반으로 답변을 생성하는 기술</p>
</blockquote>
<p>이다.</p>
<p>동작 과정</p>
<pre><code>질문

↓

문서 검색

↓

관련 내용 추출

↓

LLM에게 전달

↓

최종 답변 생성</code></pre><p>장점</p>
<ul>
<li>최신 정보 활용 가능</li>
<li>Hallucination 감소</li>
<li>기업 내부 문서 활용 가능</li>
</ul>
<hr />
<h1 id="9-agent">9. Agent</h1>
<p>Agent는</p>
<blockquote>
<p>LLM이 여러 도구를 스스로 선택하고 실행하여 목표를 달성하는 시스템</p>
</blockquote>
<p>이다.</p>
<p>예를 들어</p>
<pre><code>사용자 요청

↓

검색

↓

API 호출

↓

Python 실행

↓

결과 분석

↓

최종 답변</code></pre><p>처럼 여러 작업을 연결하여 수행한다.</p>
<p>최근 AI 서비스 대부분은 단순한 챗봇보다 이러한 Agent 구조를 사용한다.</p>
<hr />
<h1 id="10-mcpmodel-context-protocol">10. MCP(Model Context Protocol)</h1>
<p>MCP는</p>
<blockquote>
<p>AI와 외부 도구를 연결하기 위한 표준 프로토콜</p>
</blockquote>
<p>이다.</p>
<p>MCP를 사용하면</p>
<ul>
<li>데이터베이스</li>
<li>GitHub</li>
<li>Slack</li>
<li>Notion</li>
<li>Google Drive</li>
</ul>
<p>등 다양한 서비스와 AI를 쉽게 연동할 수 있다.</p>
<p>최근 AI 개발에서 가장 주목받는 기술 중 하나이다.</p>
<hr />
<h1 id="오늘-추가로-기억할-핵심">오늘 추가로 기억할 핵심</h1>
<ul>
<li>Token은 AI가 이해하는 최소 단위이다.</li>
<li>Context Window를 초과하면 중요한 정보가 잘릴 수 있다.</li>
<li>Hallucination을 줄이려면 충분한 Context와 신뢰할 수 있는 자료를 제공해야 한다.</li>
<li>Few-shot Prompting은 예시를 제공하여 원하는 결과를 유도하는 기법이다.</li>
<li>Structured Output을 활용하면 일정한 형식의 결과를 얻을 수 있다.</li>
<li>RAG는 외부 문서를 검색하여 답변의 정확성을 높인다.</li>
<li>Agent는 여러 도구를 활용해 복합적인 작업을 수행하는 AI 시스템이다.</li>
<li>MCP는 AI와 외부 서비스를 연결하는 표준 인터페이스로, 앞으로 AI 개발에서 매우 중요한 기술이다.</li>
<li>Prompt Engineering은 질문을 잘 작성하는 기술이다.</li>
<li>Context Engineering은 AI가 답변하기 위한 전체 환경을 설계하는 개념이다.</li>
<li>좋은 결과를 얻으려면 Prompt보다 Context가 더 중요하다.</li>
<li>Context는 Memory, History, External Knowledge, Tool 등으로 구성된다.</li>
<li>Prompt에는 Role, Task, Constraint, Output Format, Example을 포함하는 것이 좋다.</li>
<li>Markdown을 활용하면 원하는 형식의 답변을 안정적으로 생성할 수 있다.</li>
<li>실제 AI 서비스에서는 Prompt뿐 아니라 프로젝트 구조와 문서 관리까지 포함한 Context Engineering이 중요하다.</li>
</ul>
<hr />
<h1 id="개인-메모">개인 메모</h1>
<p>Prompt Engineering은 <strong>질문을 잘 만드는 기술</strong>, Context Engineering은 <strong>AI가 문제를 해결할 수 있는 환경을 설계하는 기술</strong>이다.</p>
<p>앞으로는 단순히 프롬프트를 수정하기보다 <strong>RAG, Memory, Tool, Agent, MCP까지 포함한 전체 AI 시스템 관점에서 설계하는 능력</strong>을 키우는 것이 중요하다.</p>
<hr />
<h1 id="느낀-점">느낀 점</h1>
<p>이번 학습을 통해 단순히 프롬프트를 잘 작성하는 것보다 <strong>AI가 문제를 해결하는 데 필요한 정보와 환경(Context)을 설계하는 것이 더 중요하다</strong>는 점을 이해했다.</p>
<p>앞으로는 프롬프트 자체를 수정하기보다 <strong>역할, 배경 정보, 제약 조건, 출력 형식, 외부 지식 등을 함께 설계하는 습관</strong>을 들여 더 정확하고 일관된 AI 결과를 만들어보려고 한다.</p>