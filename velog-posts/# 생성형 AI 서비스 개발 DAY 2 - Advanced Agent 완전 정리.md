<h2 id="학습-범위">학습 범위</h2>
<table>
<thead>
<tr>
<th>순서</th>
<th>파트</th>
<th>핵심 질문</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>Advanced Agent 개요</td>
<td>Basic Agent를 왜 고도화해야 할까?</td>
</tr>
<tr>
<td>2</td>
<td>Runtime &amp; State</td>
<td>실행 환경과 변화하는 데이터를 어떻게 구분할까?</td>
</tr>
<tr>
<td>3</td>
<td>Middleware</td>
<td>Agent 실행 중간에 어떻게 개입할까?</td>
</tr>
<tr>
<td>4</td>
<td>Guardrails</td>
<td>위험한 입력·도구 실행·출력을 어떻게 통제할까?</td>
</tr>
<tr>
<td>5</td>
<td>Long-term Memory</td>
<td>여러 대화에 걸쳐 필요한 정보를 어떻게 기억할까?</td>
</tr>
<tr>
<td>6</td>
<td>과정 마무리</td>
<td>LangChain, RAG, LangGraph는 어떻게 연결될까?</td>
</tr>
</tbody></table>
<blockquote>
<p><strong>Advanced Agent는 Tool을 잘 쓰는 Agent를 넘어, 실행 환경·상태·안전 정책·기억을 개발자가 통제할 수 있게 만든 운영 수준의 Agent입니다.</strong></p>
</blockquote>
<hr />
<h1 id="1-advanced-agent란">1. Advanced Agent란?</h1>
<h2 id="11-basic-agent만으로-부족한-이유">1.1 Basic Agent만으로 부족한 이유</h2>
<p>Basic Agent는 질문을 이해하고 Tool을 선택해 실행할 수 있습니다. 실제 서비스에서는 여기에 다음 통제가 필요합니다.</p>
<ul>
<li>현재 사용자가 Tool을 실행할 권한이 있는가?</li>
<li>모델에 개인정보가 그대로 전달되지는 않는가?</li>
<li>결제·메일 발송·삭제 전에 사람의 승인을 받는가?</li>
<li>긴 대화 때문에 토큰과 비용이 계속 증가하지 않는가?</li>
<li>위험하거나 잘못된 답변을 어떻게 막는가?</li>
<li>새 채팅에서도 사용자의 설정을 기억할 것인가?</li>
</ul>
<p>Advanced Agent는 이런 문제를 해결하기 위해 Agent의 실행 과정에 통제 장치를 추가합니다.</p>
<h2 id="12-컨텍스트-엔지니어링">1.2 컨텍스트 엔지니어링</h2>
<p>Advanced Agent의 중심에는 <strong>컨텍스트 엔지니어링(Context Engineering)</strong>이 있습니다.</p>
<p>프롬프트 엔지니어링이 모델에게 어떻게 지시할지에 집중한다면, 컨텍스트 엔지니어링은 다음 정보를 <strong>언제, 얼마나, 어떤 형태로 제공할지</strong> 설계합니다.</p>
<ul>
<li>현재 사용자와 권한</li>
<li>최근 대화와 Tool 결과</li>
<li>시스템 규칙</li>
<li>장기 메모리</li>
<li>검색한 문서</li>
<li>출력 형식과 안전 정책</li>
</ul>
<p>정보를 많이 넣는 것이 항상 좋은 것은 아닙니다. 현재 작업에 필요한 정보만 골라 제공해야 비용, 속도, 정확도, 개인정보 보호 측면에서 유리합니다.</p>
<h2 id="13-네-가지-핵심-요소">1.3 네 가지 핵심 요소</h2>
<ol>
<li><strong>Runtime &amp; State</strong>: 실행 환경과 실행 중 변하는 데이터를 분리합니다.</li>
<li><strong>Middleware</strong>: 실행 전후에 개입해 검사·수정·기록·흐름 제어를 수행합니다.</li>
<li><strong>Guardrails</strong>: 입력, Tool, 출력 단계에 안전 정책을 적용합니다.</li>
<li><strong>Long-term Memory</strong>: 세션을 넘어 필요한 정보를 저장하고 다시 사용합니다.</li>
</ol>
<h2 id="14-agent-실행-지점과-hook">1.4 Agent 실행 지점과 Hook</h2>
<pre><code class="language-text">사용자 요청
   ↓
before_agent
   ↓
before_model
   ↓
wrap_model_call ── 모델 호출 전체를 감쌈
   ↓
after_model
   ↓
wrap_tool_call ─── Tool 호출 전체를 감쌈
   ↓
after_agent
   ↓
최종 결과</code></pre>
<p>Hook을 이용하면 모델이나 Tool의 핵심 코드를 직접 수정하지 않고도 보안, 비용 관리, 로깅, 승인 절차를 추가할 수 있습니다.</p>
<hr />
<h1 id="2-runtime--state">2. Runtime &amp; State</h1>
<h2 id="21-runtime">2.1 Runtime</h2>
<p>Runtime은 Agent가 작업하는 <strong>실행 환경과 배경</strong>입니다. 한 번의 실행에서 참고하는 비교적 고정된 정보를 담습니다.</p>
<table>
<thead>
<tr>
<th>요소</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>context</code></td>
<td>현재 실행의 사용자·권한·설정</td>
<td>사용자 ID, 역할, 앱 이름</td>
</tr>
<tr>
<td><code>store</code></td>
<td>세션을 넘어 정보를 보관하는 저장소</td>
<td>사용자 선호, 장기 설정</td>
</tr>
<tr>
<td><code>stream writer</code></td>
<td>실행 중 발생하는 이벤트를 전달</td>
<td>진행 상황 실시간 표시</td>
</tr>
</tbody></table>
<p>회사에 비유하면 Context는 사원증, Store는 문서 보관함, Stream Writer는 업무 상황판에 가깝습니다.</p>
<h2 id="22-state">2.2 State</h2>
<p>State는 Agent가 실행되면서 변하는 <strong>현재 작업 상태</strong>입니다.</p>
<ul>
<li>사용자가 보낸 메시지</li>
<li>모델이 만든 답변</li>
<li>모델이 요청한 Tool Call</li>
<li>Tool이 반환한 결과</li>
<li>Middleware가 추가하거나 수정한 값</li>
</ul>
<p>특히 <code>messages</code>에는 대화와 Tool 실행 기록이 순서대로 쌓입니다. 언어 모델은 호출될 때 전달받은 메시지를 읽고 다음 답을 만듭니다.</p>
<h2 id="23-runtime과-state-비교">2.3 Runtime과 State 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Runtime</th>
<th>State</th>
</tr>
</thead>
<tbody><tr>
<td>의미</td>
<td>Agent가 일하는 환경</td>
<td>Agent가 일하면서 만든 현재 기록</td>
</tr>
<tr>
<td>변화</td>
<td>실행 중 대체로 고정</td>
<td>단계가 진행될수록 변화</td>
</tr>
<tr>
<td>예시</td>
<td>사용자 ID, 권한, Store</td>
<td>Messages, Tool 결과</td>
</tr>
<tr>
<td>비유</td>
<td>사원증과 사무 환경</td>
<td>현재 작성 중인 업무 노트</td>
</tr>
</tbody></table>
<blockquote>
<p><code>누가 어떤 환경에서 실행하는가</code>는 Runtime, <code>지금까지 무엇이 일어났는가</code>는 State입니다.</p>
</blockquote>
<h2 id="24-dataclass로-context-정의">2.4 <code>dataclass</code>로 Context 정의</h2>
<pre><code class="language-python">from dataclasses import dataclass

@dataclass
class Context:
    user_name: str
    user_role: str</code></pre>
<p><code>dataclass</code>는 생성자와 객체 표현 등을 자동으로 만들어 주므로 Context처럼 데이터를 담는 객체에 적합합니다.</p>
<pre><code class="language-python">agent = create_agent(
    model=model,
    tools=[],
    context_schema=Context,
)</code></pre>
<p><code>context_schema</code>는 Agent가 어떤 형태의 Context를 받을지 알려 줍니다.</p>
<h2 id="25-node-style에서-runtime-확인">2.5 Node-style에서 Runtime 확인</h2>
<pre><code class="language-python">from langchain.agents.middleware import before_model

@before_model
def log_before_model(state, runtime):
    print(&quot;현재 메시지 수:&quot;, len(state[&quot;messages&quot;]))
    print(&quot;현재 사용자:&quot;, runtime.context.user_name)
    return None</code></pre>
<ul>
<li><code>state</code>: 지금까지 쌓인 대화와 실행 기록</li>
<li><code>runtime</code>: 사용자 Context와 Store 같은 실행 환경</li>
</ul>
<p>로그에 메시지 전문이나 개인정보를 무분별하게 남겨서는 안 됩니다.</p>
<h2 id="26-wrap-style로-요청-변경">2.6 Wrap-style로 요청 변경</h2>
<pre><code class="language-python">from langchain.agents.middleware import wrap_model_call

@wrap_model_call
def inject_user_name(request, handler):
    user_name = request.runtime.context.user_name
    prompt = f&quot;사용자 이름은 {user_name}입니다. 필요한 경우에만 활용하세요.&quot;
    new_request = request.override(system_prompt=prompt)
    return handler(new_request)</code></pre>
<p>Wrap-style은 실제 모델 호출 전체를 감쌉니다. <code>handler()</code>를 호출해야 다음 실행으로 넘어갑니다.</p>
<h2 id="27-modelrequest의-주요-정보">2.7 ModelRequest의 주요 정보</h2>
<p><code>wrap_model_call</code>의 <code>request</code>에는 다음 정보가 들어갈 수 있습니다.</p>
<ul>
<li>사용할 모델</li>
<li>시스템 프롬프트</li>
<li>현재 메시지 목록</li>
<li>사용 가능한 Tool과 Tool 선택 정책</li>
<li>구조화된 출력 형식</li>
<li>Runtime</li>
<li>호출 설정과 메타데이터</li>
</ul>
<p>따라서 사용자 등급에 따른 Tool 제한, 메시지 길이에 따른 모델 변경, 언어 설정에 따른 프롬프트 변경 등이 가능합니다.</p>
<hr />
<h1 id="3-middleware">3. Middleware</h1>
<h2 id="31-middleware란">3.1 Middleware란?</h2>
<p>Middleware는 Agent의 메인 로직을 바꾸지 않고 실행 중간에 개입해 공통 기능을 처리합니다.</p>
<ul>
<li>위험한 요청을 모델 호출 전에 차단해 비용 절감</li>
<li>개인정보와 권한 검사</li>
<li>공통 기능의 모듈화와 재사용</li>
<li>상황에 따른 모델·Tool 변경</li>
<li>실행 기록과 오류 추적</li>
</ul>
<h2 id="32-node-style과-wrap-style">3.2 Node-style과 Wrap-style</h2>
<h3 id="node-style">Node-style</h3>
<p>특정 실행 지점의 앞이나 뒤에서 순차적으로 작동합니다.</p>
<ul>
<li><code>before_agent</code></li>
<li><code>before_model</code></li>
<li><code>after_model</code></li>
<li><code>after_agent</code></li>
</ul>
<p>보통 <code>state</code>, <code>runtime</code>을 받으며 검사, 로깅, State 수정, 조기 종료에 적합합니다.</p>
<h3 id="wrap-style">Wrap-style</h3>
<p>모델 또는 Tool 호출 전체를 감쌉니다.</p>
<ul>
<li><code>wrap_model_call</code></li>
<li><code>wrap_tool_call</code></li>
</ul>
<p>보통 <code>request</code>, <code>handler</code>를 받으며 요청 변경, 동적 모델 선택, 재시도, 예외 처리에 적합합니다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>Node-style</th>
<th>Wrap-style</th>
</tr>
</thead>
<tbody><tr>
<td>개입</td>
<td>특정 지점에서 실행</td>
<td>호출 전체를 감싸 실행</td>
</tr>
<tr>
<td>주요 인자</td>
<td><code>state</code>, <code>runtime</code></td>
<td><code>request</code>, <code>handler</code></td>
</tr>
<tr>
<td>용도</td>
<td>검사, 로깅, 상태 수정, 종료</td>
<td>요청 변형, 모델 교체, 재시도</td>
</tr>
</tbody></table>
<h2 id="33-built-in-middleware">3.3 Built-in Middleware</h2>
<h3 id="331-llm-tool-emulator">3.3.1 LLM Tool Emulator</h3>
<p>실제 Tool 대신 LLM이 Tool 응답을 흉내 냅니다.</p>
<p>적합한 경우:</p>
<ul>
<li>API가 아직 개발 중인 프로토타입</li>
<li>실제 API 호출 비용이 비싼 테스트</li>
<li>전체 서비스 흐름을 먼저 검증할 때</li>
</ul>
<pre><code class="language-python">agent = create_agent(
    model=model,
    tools=[read_message_tool, send_message_tool],
    middleware=[LLMToolEmulator(model=emulator_model)],
)</code></pre>
<p>Emulator 결과는 실제 시스템 데이터가 아니라 모델이 만든 가상 결과입니다. 실제 조회·결제·발송 성공의 증거로 사용하면 안 됩니다.</p>
<h3 id="332-todo-list-middleware">3.3.2 Todo List Middleware</h3>
<p>복잡한 요청을 여러 작업으로 나누고 상태를 추적합니다.</p>
<pre><code class="language-text">[진행 중] 새 메시지 조회
[대기] 핵심 내용 요약
[대기] 회신 초안 작성
[대기] 사용자 승인
[대기] 전송 및 결과 보고</code></pre>
<p>장기 작업에서 누락을 줄일 수 있지만, 계획 자체가 항상 옳은 것은 아니므로 중요한 업무 규칙은 별도로 검증해야 합니다.</p>
<h3 id="333-human-in-the-loop">3.3.3 Human-in-the-loop</h3>
<p>중요한 행동 전에 사람의 결정을 받습니다.</p>
<ul>
<li><code>approve</code>: 그대로 승인</li>
<li><code>edit</code>: 수정 후 승인</li>
<li><code>reject</code>: 실행 거절</li>
</ul>
<p>이메일 발송, 결제·환불, 데이터 삭제, 외부 공개, 권한 변경 등에 적합합니다. 실행을 멈췄다가 승인 후 이어 가야 하므로 Checkpointer와 함께 사용합니다.</p>
<blockquote>
<p>Agent가 실행할 수 있다는 것과 실행해도 된다는 것은 다릅니다.</p>
</blockquote>
<h3 id="334-pii-detection">3.3.4 PII Detection</h3>
<p>PII는 개인을 식별할 수 있는 정보입니다. 이메일 주소, 카드 번호, IP 주소 등을 탐지해 정책에 따라 처리합니다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>block</code></td>
<td>처리 중단</td>
</tr>
<tr>
<td><code>redact</code></td>
<td>민감한 값을 완전히 가림</td>
</tr>
<tr>
<td><code>mask</code></td>
<td>일부 문자만 남기고 마스킹</td>
</tr>
<tr>
<td><code>hash</code></td>
<td>원문 대신 해시값 사용</td>
</tr>
</tbody></table>
<p>적용 위치도 구분할 수 있습니다.</p>
<ul>
<li>사용자 입력</li>
<li>모델 입력</li>
<li>Tool 결과</li>
<li>모델 출력</li>
</ul>
<p>정규식은 빠르지만 변형된 표현을 놓칠 수 있습니다. 탐지뿐 아니라 접근 권한, 암호화, 보관 기간, 삭제 정책까지 함께 설계해야 합니다.</p>
<h3 id="335-summarization-middleware">3.3.5 Summarization Middleware</h3>
<p>긴 대화의 오래된 부분을 요약하고 최근 대화는 원문으로 유지합니다.</p>
<pre><code class="language-text">요약 전: 오래된 대화 전체 + 최근 대화 전체
요약 후: 오래된 대화 요약 + 최근 대화 원문</code></pre>
<p>주요 설정:</p>
<ul>
<li><code>model</code>: 요약에 사용할 모델</li>
<li><code>trigger</code>: 메시지 수나 토큰 수 등 시작 조건</li>
<li><code>keep</code>: 원문으로 보존할 최신 대화 범위</li>
<li><code>summary_prompt</code>: 요약 방식</li>
<li>요약 대상과 목표 토큰 크기</li>
</ul>
<p>요약은 비용을 줄이지만 세부 내용이 손실될 수 있습니다. 최신 대화, 확정된 결정, 안전 관련 정보는 별도로 보존하는 것이 좋습니다.</p>
<h2 id="34-custom-middleware">3.4 Custom Middleware</h2>
<h3 id="node-style-입력-검사-후-종료">Node-style: 입력 검사 후 종료</h3>
<pre><code class="language-python">from langchain.agents.middleware import before_agent
from langchain.messages import AIMessage

@before_agent(can_jump_to=[&quot;end&quot;])
def validate_input(state, runtime):
    text = state[&quot;messages&quot;][-1].content
    if &quot;차단할 표현&quot; in text:
        return {
            &quot;messages&quot;: [AIMessage(content=&quot;이 요청은 처리할 수 없습니다.&quot;)],
            &quot;jump_to&quot;: &quot;end&quot;,
        }
    return None</code></pre>
<p>LLM 호출 전에 끝내므로 빠르고 비용이 적게 듭니다.</p>
<h3 id="wrap-style-메시지-길이에-따른-모델-선택">Wrap-style: 메시지 길이에 따른 모델 선택</h3>
<pre><code class="language-python">@wrap_model_call
def dynamic_model_selector(request, handler):
    total_length = sum(len(message.content) for message in request.messages)
    selected_model = small_model if total_length &lt; 1000 else large_model
    return handler(request.override(model=selected_model))</code></pre>
<p>짧은 요청에는 빠르고 저렴한 모델을, 복잡한 요청에는 성능이 높은 모델을 사용할 수 있습니다.</p>
<h3 id="반복적인-agent-실행-예시">반복적인 Agent 실행 예시</h3>
<pre><code class="language-text">사용자: 42 + 3 × 23은?
→ 모델: 곱셈을 먼저 판단
→ multiply(3, 23) → 69
→ 모델: 덧셈 판단
→ add(42, 69) → 111
→ 모델: 최종 답변 생성</code></pre>
<p>Middleware는 이 반복 과정의 모델 호출과 Tool 호출 전후에 개입할 수 있습니다.</p>
<hr />
<h1 id="4-guardrails">4. Guardrails</h1>
<h2 id="41-guardrail이란">4.1 Guardrail이란?</h2>
<p>Guardrail은 Agent가 위험한 방향으로 이탈하지 않도록 입력, Tool 실행, 출력을 검사하는 안전장치입니다.</p>
<pre><code class="language-text">입력 Guardrail → Model → Tool Guardrail → 출력 Guardrail</code></pre>
<p>Middleware가 실행 흐름에 개입하는 <strong>기술적 장치</strong>라면, Guardrail은 그 장치를 이용해 적용하는 <strong>안전 정책과 검증 규칙</strong>입니다.</p>
<h2 id="42-built-in과-custom">4.2 Built-in과 Custom</h2>
<ul>
<li>Built-in: 개인정보 탐지, 사람 승인처럼 자주 쓰는 기능</li>
<li>Custom: 서비스 목적에 맞게 직접 만든 입력·출력 검사 규칙</li>
</ul>
<p>Custom Guardrail은 Before Agent, After Agent, 여러 Guardrail 결합으로 구성할 수 있습니다.</p>
<h2 id="43-결정론적-방식과-모델-기반-방식">4.3 결정론적 방식과 모델 기반 방식</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>결정론적</th>
<th>모델 기반</th>
</tr>
</thead>
<tbody><tr>
<td>방법</td>
<td>정규식, 키워드, 스키마, 코드</td>
<td>별도 LLM이 의미 평가</td>
</tr>
<tr>
<td>장점</td>
<td>빠르고 저렴하며 예측 가능</td>
<td>미묘한 의미와 우회 표현 파악</td>
</tr>
<tr>
<td>단점</td>
<td>복잡한 문맥을 놓칠 수 있음</td>
<td>추가 비용·지연·평가 오류 가능</td>
</tr>
<tr>
<td>예시</td>
<td>카드 번호 패턴 탐지</td>
<td>답변의 정책 위반 평가</td>
</tr>
</tbody></table>
<p>명확한 규칙은 코드로 먼저 처리하고, 의미 판단이 필요할 때 모델 기반 검사를 추가하는 방식이 효율적입니다.</p>
<h2 id="44-before-agent-guardrail">4.4 Before Agent Guardrail</h2>
<p>사용자 입력이 LLM에 전달되기 전에 검사합니다.</p>
<ul>
<li>부정행위 요청 감지</li>
<li>서비스 범위를 벗어난 질문 안내</li>
<li>유해 요청 차단</li>
<li>개인정보 마스킹</li>
<li>위기 상황을 사람 담당자에게 이관</li>
</ul>
<pre><code class="language-python">@before_agent(can_jump_to=[&quot;end&quot;])
def education_guardrail(state, runtime):
    user_text = state[&quot;messages&quot;][-1].content
    if contains_disallowed_request(user_text):
        return {
            &quot;messages&quot;: [{
                &quot;role&quot;: &quot;assistant&quot;,
                &quot;content&quot;: &quot;정답 대신 스스로 풀 수 있도록 힌트를 제공하겠습니다.&quot;,
            }],
            &quot;jump_to&quot;: &quot;end&quot;,
        }
    return None</code></pre>
<p>장점은 모델 호출 전에 차단해 비용과 정보 노출을 줄인다는 것입니다. 단순 키워드 검사는 정상 요청을 잘못 막거나 우회 표현을 놓칠 수 있습니다.</p>
<h2 id="45-after-agent-guardrail">4.5 After Agent Guardrail</h2>
<p>모델이 답변을 만든 뒤 사용자에게 보여 주기 전에 검사합니다.</p>
<ol>
<li>마지막 메시지가 AI 답변인지 확인합니다.</li>
<li>규칙 또는 평가 모델로 검사합니다.</li>
<li>안전하면 그대로 반환합니다.</li>
<li>문제가 있으면 차단하거나 수정합니다.</li>
<li>필요하면 안전 조건으로 다시 생성합니다.</li>
</ol>
<pre><code class="language-python">@after_agent
def output_guardrail(state, runtime):
    last_message = state[&quot;messages&quot;][-1]
    if evaluate_answer(last_message.content) == &quot;UNSAFE&quot;:
        last_message.content = &quot;안전 기준에 따라 답변을 제공할 수 없습니다.&quot;
    return None</code></pre>
<h3 id="intervention">Intervention</h3>
<p>문제가 된 답을 미리 정한 안전 문구로 교체합니다. 빠르고 예측 가능하지만 상황에 맞는 자연스러운 안내는 어려울 수 있습니다.</p>
<h3 id="correction--regeneration">Correction / Regeneration</h3>
<p>안전 조건을 추가해 답변을 다시 생성합니다. 자연스럽지만 추가 모델 호출로 비용과 시간이 늘며, 재생성 결과도 다시 검증해야 할 수 있습니다.</p>
<h2 id="46-before와-after-비교">4.6 Before와 After 비교</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>Before Agent</th>
<th>After Agent</th>
</tr>
</thead>
<tbody><tr>
<td>시점</td>
<td>LLM 호출 전</td>
<td>답변 생성 후</td>
</tr>
<tr>
<td>목적</td>
<td>입력 차단, 보안, 비용 절감</td>
<td>출력 품질과 안전성 검사</td>
</tr>
<tr>
<td>방식</td>
<td>규칙, 키워드, 정규식</td>
<td>규칙 또는 평가 모델</td>
</tr>
<tr>
<td>비용</td>
<td>호출 전에 막아 절약 가능</td>
<td>추가 검증 비용 발생 가능</td>
</tr>
<tr>
<td>한계</td>
<td>최종 출력을 검사하지 못함</td>
<td>메인 모델 비용은 이미 발생</td>
</tr>
</tbody></table>
<h2 id="47-여러-guardrail-결합">4.7 여러 Guardrail 결합</h2>
<pre><code class="language-text">Layer 1: 서비스 범위와 부정행위 입력 검사
Layer 2: 개인정보 탐지와 마스킹
Layer 3: 위험 상황의 사람 담당자 이관
Layer 4: 모델 출력의 안전성과 품질 검사</code></pre>
<pre><code class="language-python">agent = create_agent(
    model=model,
    tools=[],
    middleware=[
        education_guardrail,
        privacy_guardrail,
        human_escalation_guardrail,
        output_guardrail,
    ],
)</code></pre>
<p>빠르고 저렴한 검사를 앞에 두고, 개인정보는 모델 전달 전에 처리하며, 위기 상황은 자동 답변보다 사람 이관을 우선합니다. 차단 이유는 기록하되 로그에 개인정보를 남기지 않습니다.</p>
<hr />
<h1 id="5-long-term-memory">5. Long-term Memory</h1>
<h2 id="51-장기-메모리란">5.1 장기 메모리란?</h2>
<p>장기 메모리는 현재 대화 스레드를 넘어 유지할 정보입니다.</p>
<ul>
<li>사용자가 선호하는 설명 난이도</li>
<li>기본 언어 설정</li>
<li>알림 수신 방식</li>
<li>사용자가 저장을 요청한 업무 설정</li>
<li>이전에 확정한 중요한 결정</li>
</ul>
<table>
<thead>
<tr>
<th>구분</th>
<th>단기 메모리</th>
<th>장기 메모리</th>
</tr>
</thead>
<tbody><tr>
<td>범위</td>
<td>하나의 대화 스레드</td>
<td>여러 대화와 세션</td>
</tr>
<tr>
<td>저장 수단</td>
<td>Checkpointer</td>
<td>Store 또는 영구 DB</td>
</tr>
<tr>
<td>정보</td>
<td>최근 메시지, Tool 결과</td>
<td>설정, 선호, 확정된 정보</td>
</tr>
</tbody></table>
<p>모든 대화를 무조건 저장하는 것은 올바른 장기 메모리가 아닙니다. 필요한 정보만 동의와 정책에 따라 저장해야 합니다.</p>
<h2 id="52-store-기본-구조">5.2 Store 기본 구조</h2>
<pre><code class="language-python">from langchain_core.stores import InMemoryStore

store = InMemoryStore()</code></pre>
<p>Store는 Key-Value 방식으로 정보를 저장합니다.</p>
<ul>
<li>Key: 어느 사용자·앱의 어떤 기억인지 식별</li>
<li>Value: 기억이 담긴 <code>Document</code></li>
<li><code>mset</code>: 여러 항목 저장</li>
<li><code>mget</code>: 여러 Key 조회</li>
</ul>
<h2 id="53-namespace와-id">5.3 Namespace와 ID</h2>
<pre><code class="language-python">namespace = f&quot;{user_id}::{app_name}&quot;
memory_id = &quot;item_001&quot;
store_key = f&quot;{namespace}::{memory_id}&quot;</code></pre>
<pre><code class="language-text">사용자 + 앱 = 폴더(namespace)
기억 ID = 파일 이름
Document = 파일 내용</code></pre>
<p>실제 서비스의 <code>user_id</code>는 인증 시스템에서 가져와야 합니다. 사용자가 메시지에 적은 ID를 믿으면 다른 사용자의 기억에 접근하는 보안 문제가 생길 수 있습니다.</p>
<h2 id="54-document-저장과-조회">5.4 Document 저장과 조회</h2>
<pre><code class="language-python">from langchain_core.documents import Document

document = Document(
    page_content=&quot;사용자는 초보자용 단계별 설명을 선호함&quot;,
    metadata={&quot;category&quot;: &quot;preference&quot;, &quot;source&quot;: &quot;user_setting&quot;},
)

store.mset([(store_key, document)])
documents = store.mget([store_key])</code></pre>
<ul>
<li><code>page_content</code>: 모델이 참고할 핵심 기억</li>
<li><code>metadata</code>: 분류, 출처, 생성 시각 같은 관리 정보</li>
</ul>
<p>교재에서는 Store Key 목록을 별도 리스트나 사용자·앱별 딕셔너리로 관리합니다. 조회할 Key를 추적해야 하기 때문입니다.</p>
<h2 id="55-use-case-1-미리-저장한-기억-주입">5.5 Use Case 1: 미리 저장한 기억 주입</h2>
<p>개발자가 기억과 Key를 미리 저장하고, Middleware가 현재 사용자의 기억을 조회해 시스템 프롬프트에 넣습니다.</p>
<h3 id="흐름">흐름</h3>
<ol>
<li>Store 생성</li>
<li>사용자·앱별 Namespace 정의</li>
<li>고정 Key와 Document 저장</li>
<li>조회할 Key 목록 관리</li>
<li>Context에 사용자 ID와 앱 이름 정의</li>
<li><code>wrap_model_call</code>에서 사용자 확인</li>
<li>해당 사용자의 기억만 조회</li>
<li>시스템 프롬프트에 기억 주입</li>
<li>Agent 실행</li>
</ol>
<pre><code class="language-python">from dataclasses import dataclass

@dataclass
class Context:
    user_id: str
    app_name: str

@wrap_model_call
def inject_memory(request, handler):
    user_id = request.runtime.context.user_id
    app_name = request.runtime.context.app_name
    memory_text = load_memory_for(user_id, app_name)

    prompt = f&quot;필요한 경우에만 참고할 사용자 설정:\n{memory_text}&quot;
    return handler(request.override(system_prompt=prompt))</code></pre>
<pre><code class="language-python">agent = create_agent(
    model=model,
    store=store,
    context_schema=Context,
    middleware=[inject_memory],
)</code></pre>
<p>통제하기 쉽지만, 모든 기억을 매번 넣으면 토큰이 낭비되고 관련 없는 정보가 판단을 방해할 수 있습니다.</p>
<h2 id="56-use-case-2-tool로-기억-저장·조회">5.6 Use Case 2: Tool로 기억 저장·조회</h2>
<p>Agent에게 사용자 정보 저장 Tool과 조회 Tool을 제공합니다.</p>
<h3 id="흐름-1">흐름</h3>
<ol>
<li>Store와 사용자별 Key 관리 구조 생성</li>
<li><code>Context</code>로 사용자와 앱 구분</li>
<li><code>UserInfo</code>로 저장할 정보 형식 정의</li>
<li>조회 Tool 생성</li>
<li>저장 Tool 생성</li>
<li>Middleware로 Tool 사용 기준 추가</li>
<li>Agent에 Store, Context, Tool, Middleware 연결</li>
<li>대화 중 Agent가 필요한 Tool 호출</li>
</ol>
<h3 id="저장-정보-구조">저장 정보 구조</h3>
<pre><code class="language-python">from typing_extensions import TypedDict

class UserInfo(TypedDict, total=False):
    preferred_language: str
    explanation_level: str
    notification_preference: str</code></pre>
<p>허용 필드를 제한하면 Agent가 임의의 정보를 무분별하게 저장하는 위험을 줄일 수 있습니다.</p>
<h3 id="조회-tool">조회 Tool</h3>
<pre><code class="language-python">@tool
def get_user_info(runtime) -&gt; str:
    &quot;&quot;&quot;현재 사용자의 저장된 설정을 조회한다.&quot;&quot;&quot;
    user_id = runtime.context.user_id
    app_name = runtime.context.app_name
    keys = get_managed_keys(user_id, app_name)

    if not keys:
        return &quot;기록된 설정이 없습니다.&quot;

    return format_documents(runtime.store.mget(keys))</code></pre>
<p>현재 Runtime의 인증된 사용자와 앱 Namespace만 조회해야 합니다.</p>
<h3 id="저장-tool">저장 Tool</h3>
<pre><code class="language-python">import uuid

@tool
def save_user_info(user_info: UserInfo, runtime) -&gt; str:
    &quot;&quot;&quot;사용자가 저장을 허용한 설정을 저장한다.&quot;&quot;&quot;
    if not user_info:
        return &quot;저장할 정보가 없습니다.&quot;

    user_id = runtime.context.user_id
    app_name = runtime.context.app_name
    memory_id = str(uuid.uuid4())
    store_key = f&quot;{user_id}::{app_name}::{memory_id}&quot;

    document = convert_to_document(user_info)
    runtime.store.mset([(store_key, document)])
    register_managed_key(user_id, app_name, store_key)
    return &quot;사용자 설정이 저장되었습니다.&quot;</code></pre>
<p>UUID를 사용하면 기억마다 겹치지 않는 ID를 만들기 쉽습니다.</p>
<h3 id="프롬프트-업데이트-middleware">프롬프트 업데이트 Middleware</h3>
<pre><code class="language-python">@wrap_model_call
def prompt_update(request, handler):
    rule = &quot;&quot;&quot;
    사용자 설정은 목적을 확인한 뒤 저장하세요.
    민감 정보는 저장하지 마세요.
    사용자가 설정을 물으면 조회 Tool을 사용하세요.
    &quot;&quot;&quot;
    prompt = (request.system_prompt or &quot;&quot;) + rule
    return handler(request.override(system_prompt=prompt))</code></pre>
<p>프롬프트만으로 보안을 보장할 수는 없습니다. 저장 Tool 안에서도 허용 필드, 권한, 데이터 크기를 다시 검증해야 합니다.</p>
<pre><code class="language-python">agent = create_agent(
    model=model,
    tools=[get_user_info, save_user_info],
    store=store,
    middleware=[prompt_update],
    context_schema=Context,
)</code></pre>
<h3 id="대화-예시">대화 예시</h3>
<pre><code class="language-text">사용자: 앞으로 설명은 초보자 수준으로 해 줘.
→ save_user_info 호출
→ Store에 설정 저장

새 대화: 내가 원하는 설명 수준이 뭐였지?
→ get_user_info 호출
→ 저장된 설정 조회
→ 초보자 수준을 선호한다고 답변</code></pre>
<h2 id="57-두-use-case-비교">5.7 두 Use Case 비교</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>Use Case 1</th>
<th>Use Case 2</th>
</tr>
</thead>
<tbody><tr>
<td>저장</td>
<td>개발자가 미리 저장</td>
<td>Agent가 Tool로 동적 저장</td>
</tr>
<tr>
<td>Key</td>
<td>고정 ID 가능</td>
<td>UUID 등 자동 생성 가능</td>
</tr>
<tr>
<td>조회</td>
<td>Middleware가 프롬프트에 주입</td>
<td>Agent가 조회 Tool 호출</td>
</tr>
<tr>
<td>장점</td>
<td>단순하고 통제하기 쉬움</td>
<td>대화 중 새 정보 기억 가능</td>
</tr>
<tr>
<td>주의</td>
<td>매번 주입하면 토큰 낭비</td>
<td>잘못된 정보까지 저장할 위험</td>
</tr>
</tbody></table>
<p>함께 사용할 수도 있습니다.</p>
<ul>
<li>항상 필요한 설정: 선택적으로 프롬프트에 주입</li>
<li>특정 질문에만 필요한 정보: 조회 Tool 사용</li>
<li>새로운 사용자 설정: 검증 후 저장 Tool 사용</li>
</ul>
<h2 id="58-장기-메모리-안전-수칙">5.8 장기 메모리 안전 수칙</h2>
<ul>
<li>저장 사실과 목적을 사용자가 알 수 있어야 합니다.</li>
<li>꼭 필요한 최소 정보만 저장합니다.</li>
<li>민감 정보는 기본적으로 저장하지 않습니다.</li>
<li>사용자와 앱별 Namespace를 분리합니다.</li>
<li>인증된 사용자 ID만 사용합니다.</li>
<li>조회, 수정, 삭제 기능을 제공합니다.</li>
<li>잘못 저장된 기억을 정정할 수 있어야 합니다.</li>
<li>보관 기간과 자동 삭제 기준을 정합니다.</li>
<li>현재 요청과 관련된 기억만 프롬프트에 넣습니다.</li>
<li>Tool 입력을 서버에서 다시 검증합니다.</li>
</ul>
<p><code>InMemoryStore</code>는 프로세스가 종료되면 내용이 사라질 수 있는 실습용 저장소입니다. 실제 서비스에서는 영구 DB, 접근 제어, 암호화, 감사 로그가 필요합니다.</p>
<hr />
<h1 id="6-advanced-agent-전체-연결">6. Advanced Agent 전체 연결</h1>
<p>고객 지원 Agent를 예로 연결해 봅시다.</p>
<pre><code class="language-text">1. Runtime
   - 인증된 사용자 ID와 상담원 권한 확인

2. Before Guardrail
   - 개인정보 마스킹과 위험 요청 차단

3. State
   - 현재 상담 메시지와 Tool 결과 유지

4. Model
   - 필요한 Tool 판단

5. Middleware
   - Tool 권한 검사와 실행 기록

6. Human-in-the-loop
   - 환불·보상 실행 전 담당자 승인

7. After Guardrail
   - 확인되지 않은 약속과 개인정보 노출 검사

8. Long-term Memory
   - 사용자가 동의한 연락 방식만 저장</code></pre>
<p>핵심은 자동화의 양이 아니라 <strong>통제 가능성</strong>입니다.</p>
<hr />
<h1 id="7-자주-헷갈리는-개념">7. 자주 헷갈리는 개념</h1>
<h2 id="runtime과-state">Runtime과 State</h2>
<ul>
<li>Runtime: 누가 어떤 환경과 권한으로 실행하는가</li>
<li>State: 실행 과정에서 지금까지 무엇이 일어났는가</li>
</ul>
<h2 id="middleware와-guardrail">Middleware와 Guardrail</h2>
<ul>
<li>Middleware: 실행 흐름에 개입하는 구현 장치</li>
<li>Guardrail: 안전과 정책을 지키는 규칙</li>
<li>Guardrail을 Middleware로 구현할 수 있음</li>
</ul>
<h2 id="node-style과-wrap-style">Node-style과 Wrap-style</h2>
<ul>
<li>Node-style: 특정 시점에서 검사하거나 State 수정</li>
<li>Wrap-style: 모델 또는 Tool 호출 전체를 감싸 흐름 제어</li>
</ul>
<h2 id="checkpointer와-store">Checkpointer와 Store</h2>
<ul>
<li>Checkpointer: 한 대화 스레드의 State 저장·복원</li>
<li>Store: 여러 세션에서 재사용할 장기 정보 저장</li>
</ul>
<h2 id="pii-detection과-guardrail">PII Detection과 Guardrail</h2>
<ul>
<li>PII Detection: 개인정보를 찾는 구체적인 기능</li>
<li>Guardrail: 탐지된 정보를 차단·마스킹하는 전체 정책</li>
</ul>
<h2 id="structured-output과-guardrail">Structured Output과 Guardrail</h2>
<ul>
<li>Structured Output: 결과 형식을 일정하게 만듦</li>
<li>Guardrail: 결과의 허용 여부와 안전성을 검사</li>
<li>형식이 맞아도 내용이 안전하거나 사실이라는 보장은 없음</li>
</ul>
<hr />
<h1 id="8-교재-마지막-과정-정리">8. 교재 마지막 과정 정리</h1>
<p>교재 마지막 부분은 학습 내용을 LangChain, RAG, LangGraph로 연결합니다.</p>
<h2 id="langchain">LangChain</h2>
<ul>
<li>LCEL을 이용한 Chain 구성</li>
<li>Prompt와 Structured Output</li>
<li>Tool Calling과 Agent</li>
<li>대화 Memory 관리</li>
</ul>
<h2 id="rag">RAG</h2>
<ul>
<li>문서 수집과 전처리</li>
<li>임베딩과 벡터 저장소</li>
<li>검색과 재정렬</li>
<li>검색 Context를 모델에 전달</li>
<li>근거 기반 응답으로 환각을 줄이는 구조</li>
</ul>
<h2 id="langgraph와-agent-고도화">LangGraph와 Agent 고도화</h2>
<ul>
<li>State 기반 실행 흐름</li>
<li>조건 분기와 반복</li>
<li>Checkpointer를 통한 상태 저장</li>
<li>Middleware와 Guardrail</li>
<li>사람 승인과 장기 메모리</li>
</ul>
<pre><code class="language-text">LangChain: 모델·프롬프트·Tool 연결
    ↓
RAG: 외부 지식을 검색해 근거 제공
    ↓
LangGraph/Advanced Agent: 상태와 실행 흐름 통제</code></pre>
<h2 id="공식-문서-확인-방법">공식 문서 확인 방법</h2>
<p>라이브러리는 업데이트가 빠르므로 실습 코드가 동작하지 않을 때는 다음 순서로 확인합니다.</p>
<ol>
<li>현재 가상환경의 LangChain 버전 확인</li>
<li>Notebook이 같은 Python Kernel을 쓰는지 확인</li>
<li>설치 버전과 맞는 공식 문서 확인</li>
<li>Import 경로와 함수 인자 변경 여부 확인</li>
<li>API 키는 <code>.env</code>에서 불러오고 출력하지 않기</li>
</ol>
<p>참고: <a href="https://docs.langchain.com/">LangChain 공식 문서</a></p>
<hr />
<h1 id="9-day-2-핵심-요약">9. DAY 2 핵심 요약</h1>
<ol>
<li>Advanced Agent는 컨텍스트 엔지니어링으로 안전하고 통제 가능한 Agent를 만듭니다.</li>
<li>Runtime은 사용자·권한·Store 같은 실행 환경입니다.</li>
<li>State는 메시지와 Tool 결과처럼 실행 중 변하는 데이터입니다.</li>
<li>Node-style은 특정 실행 지점에, Wrap-style은 호출 전체에 개입합니다.</li>
<li>Built-in Middleware에는 Tool Emulator, Todo List, Human-in-the-loop, PII Detection, Summarization이 있습니다.</li>
<li>Guardrail은 입력, 처리, 출력 단계의 위험을 막는 안전 정책입니다.</li>
<li>Before Guardrail은 호출 전에, After Guardrail은 답변 생성 후 검사합니다.</li>
<li>장기 메모리는 Store, Namespace, Document, Key 관리 구조를 사용합니다.</li>
<li>장기 메모리에서는 동의, 최소 수집, 권한 분리, 수정·삭제 가능성이 우선입니다.</li>
<li>좋은 Agent는 허용된 일을 정확하고 안전하게 수행하는 Agent입니다.</li>
</ol>
<blockquote>
<p><strong>Advanced Agent 설계의 핵심은 지능을 높이는 것만이 아니라, 그 지능이 언제·어떤 정보로·어떤 권한 안에서 행동할지 통제하는 것입니다.</strong></p>
</blockquote>