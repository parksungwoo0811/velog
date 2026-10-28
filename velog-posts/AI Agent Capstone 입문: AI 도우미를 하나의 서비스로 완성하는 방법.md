<blockquote>
<p><strong>대상:</strong> AI Agent 프로젝트를 처음 배우는 사람. 프로그래밍을 잘 몰라도 역할과 흐름부터 이해할 수 있도록 설명합니다.</p>
<p><strong>글의 범위:</strong> 제공된 교재의 본문 4개 파트와 각 파트의 요약·실습·연결도·서비스 흐름을 학습 목적에 맞게 다시 설명했습니다. 목차에만 있는 ‘Agent 확장’은 별도 본문이 없으므로, 관련 내용을 모은 <strong>보충 파트</strong>로 표시했습니다.</p>
<p>원본 슬라이드·이미지·내부 실습 문서는 재게시하지 않습니다. 사람을 식별하는 정보, 내부 주소·계정·접속 정보도 담지 않았습니다. 아래 가상의 사례, 설계안, 숫자, 데이터, 코드는 이해를 돕기 위해 새로 만든 예시입니다. 별도 내부 실습 가이드의 코드를 재현한 것이 아닙니다.</p>
</blockquote>
<hr />
<h2 id="먼저-읽기-이번-프로젝트는-무엇을-만드는-걸까">먼저 읽기: 이번 프로젝트는 무엇을 만드는 걸까?</h2>
<p>AI에게 질문하면 글로 답해 주는 채팅을 사용해 본 적이 있을 것입니다. 이번에는 한 걸음 더 나아갑니다.</p>
<p><strong>질문을 받고 → 필요한 자료를 찾고 → 도구로 현재 상황을 확인하고 → 결과를 설명하며 → 다음 대화까지 이어가는 서비스</strong>를 만듭니다.</p>
<p>Capstone은 여러 시간에 배운 내용을 연결하는 종합 프로젝트라는 뜻입니다. 블록 하나의 모양을 배우는 수업이 끝났다면, 이번에는 그 블록으로 집을 짓는 것입니다.</p>
<p>이 글에서는 가상의 <strong>학교 도서관 AI 도우미</strong>를 공통 예시로 사용합니다.</p>
<pre><code class="language-text">학생: 공룡 책을 빌리고 싶어요.
AI: 공룡 책을 검색하고 있어요.
AI: ‘공룡의 하루’가 있어요. 현재 한 권을 빌릴 수 있어요.
학생: 그 책을 예약해 줘요.
AI: 예약할 책과 내용을 확인해 주세요.
학생이 확인하면 예약 도구를 실행하고, 실제 결과를 알려 줍니다.</code></pre>
<p>실무에서는 이를 ‘운영 매뉴얼 검색 → 시스템 상태 확인 → 관련 이슈 조회 → 작업 기록 생성’으로 바꿀 수 있습니다. 회사의 서버를 관리하는 도우미와 도서관 도우미는 다루는 자료가 다르지만, 연결 원리는 비슷합니다.</p>
<h3 id="전체-구성-요소를-식당에-비유해-보기">전체 구성 요소를 식당에 비유해 보기</h3>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>쉬운 뜻</th>
<th>식당 비유</th>
<th>도서관 프로젝트에서 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>Frontend</td>
<td>사용자가 보는 화면</td>
<td>주문판과 주문 화면</td>
<td>질문 입력, 검색 진행 상황 표시</td>
</tr>
<tr>
<td>Backend</td>
<td>요청을 접수하고 처리하는 서버 쪽 프로그램</td>
<td>주문 접수 창구</td>
<td>입력 검사, 사용자 확인, 에이전트 실행</td>
</tr>
<tr>
<td>AI Agent</td>
<td>목표를 위해 다음 행동을 선택하는 프로그램</td>
<td>주문에 맞게 일하는 조리 담당자</td>
<td>검색할지, 재고를 확인할지 판단</td>
</tr>
<tr>
<td>LLM</td>
<td>문장을 이해하고 생성하는 언어 모델</td>
<td>설명하고 판단하는 능력</td>
<td>질문 해석, 도구 선택 제안, 답변 작성</td>
</tr>
<tr>
<td>MCP Client</td>
<td>MCP 서버와 연결하는 구성 요소</td>
<td>부서별 연락 담당자</td>
<td>사용 가능한 기능 조회와 호출 전달</td>
</tr>
<tr>
<td>MCP Server</td>
<td>정해진 규칙으로 데이터와 도구를 제공하는 프로그램</td>
<td>식재료·창고 담당 부서</td>
<td>책 검색, 대출 상태 조회 기능 제공</td>
</tr>
<tr>
<td>Vector DB</td>
<td>의미가 비슷한 자료를 찾는 데이터베이스</td>
<td>비슷한 재료를 찾아 주는 창고</td>
<td>‘무서운 동물 책’과 관련된 책 설명 검색</td>
</tr>
<tr>
<td>State Store</td>
<td>대화와 진행 상태를 저장하는 곳</td>
<td>주문 기록장</td>
<td>방금 이야기한 책, 실행 단계 저장</td>
</tr>
<tr>
<td>외부 API</td>
<td>다른 서비스에 일을 요청하는 창구</td>
<td>외부 배송 업체 연락처</td>
<td>실제 대출 시스템에 예약 요청</td>
</tr>
</tbody></table>
<p>Agent는 LLM 하나만을 가리키는 말이 아닙니다. 모델, 도구, 상태, 실행 규칙을 묶은 프로그램으로 이해하면 좋습니다. 또한 LLM이 ‘예약하자’고 판단해도 <strong>서버가 권한과 입력을 검사한 뒤</strong> 실제 도구를 실행해야 합니다.</p>
<h3 id="이-글의-순서">이 글의 순서</h3>
<ol>
<li>MCP 설계 및 개발: AI가 사용할 도구 상자를 설계하기</li>
<li>통합 아키텍처: 화면·서버·AI·저장소의 역할 나누기</li>
<li>Streaming: 기다리는 동안 진행 상황과 답변 전달하기</li>
<li>통합 환경의 운영: 관찰·평가·오류·비용·기억 관리하기</li>
<li>보충: Agent 확장을 계획하고 프로젝트 완료 여부 확인하기</li>
</ol>
<hr />
<h2 id="part-1-mcp-설계-및-개발-ai의-도구-상자-만들기">Part 1. MCP 설계 및 개발: AI의 도구 상자 만들기</h2>
<h3 id="1-1-왜-mcp가-필요할까">1-1. 왜 MCP가 필요할까?</h3>
<p>AI가 책에 대해 설명하는 능력과, <strong>지금 도서관에 책이 남아 있는지 확인하는 능력</strong>은 다릅니다. 현재 재고는 도서관 시스템에 물어봐야 합니다.</p>
<p>외부 시스템마다 연결 방식이 전부 다르면, 새 기능을 연결할 때마다 새로운 방식으로 개발해야 합니다. MCP(Model Context Protocol)는 AI 애플리케이션과 외부 데이터·도구를 연결하는 공통 약속입니다.</p>
<p>USB 규격이 여러 기기의 연결을 돕는 것처럼 생각할 수 있습니다. 다만 MCP를 연결했다고 내용의 정확성, 권한 관리, 업무의 성공까지 자동으로 보장되지는 않습니다.</p>
<pre><code class="language-text">AI 애플리케이션
    ↓ MCP 규칙으로 요청
도서관 MCP 서버
    ↓ 실제 시스템에 맞는 처리
책 데이터 / 대출 API</code></pre>
<p>MCP 서버는 AI 모델 그 자체가 아닙니다. 모델이 이용할 수 있는 기능을 제공하는 쪽입니다.</p>
<h3 id="1-2-host-client-server는-누구일까">1-2. Host, Client, Server는 누구일까?</h3>
<table>
<thead>
<tr>
<th>이름</th>
<th>역할</th>
<th>도서관 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Host</td>
<td>사용자와 상호작용하며 모델과 여러 연결을 조율하는 애플리케이션</td>
<td>안내 데스크 전체</td>
</tr>
<tr>
<td>Client</td>
<td>Host 안에서 특정 MCP 서버와의 연결·메시지를 관리하는 구성 요소</td>
<td>부서별 전담 연락 담당자</td>
</tr>
<tr>
<td>Server</td>
<td>데이터·도구·프롬프트를 제공하는 프로그램</td>
<td>도서 검색 부서, 대출 부서</td>
</tr>
</tbody></table>
<p>하나의 Host가 여러 Client를 가질 수 있고, 각 Client가 특정 Server와 연결됩니다. <strong>Client 하나가 반드시 독립 프로그램 파일이나 별도 프로세스여야 하는 것은 아닙니다.</strong> 역할의 구분과 실행 프로그램의 구분은 다릅니다.</p>
<p>일반적인 연결 흐름은 다음과 같습니다.</p>
<ol>
<li>Client가 Server와 연결합니다.</li>
<li>서로 지원하는 기능을 확인합니다.</li>
<li>Server가 제공하는 도구 등의 목록과 설명을 받습니다.</li>
<li>Host가 필요한 정보를 모델에 제공합니다.</li>
<li>모델의 도구 호출 요청을 애플리케이션이 검사하고 실행합니다.</li>
<li>실행 결과를 받아 최종 답변에 사용합니다.</li>
</ol>
<p>JSON-RPC는 이 과정의 요청·응답 메시지를 표현하는 규칙입니다. ‘무슨 기능을, 어떤 입력으로 호출하고, 어느 요청에 대한 결과인지’를 서로 알아보도록 쓰는 공통 양식이라고 생각하면 됩니다.</p>
<h3 id="1-3-resource-tool-prompt-세-칸으로-정리하기">1-3. Resource, Tool, Prompt: 세 칸으로 정리하기</h3>
<p>도구 상자 안에는 성격이 다른 물건이 들어갑니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>핵심 의미</th>
<th>주로 사용을 조율하는 쪽</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>Resource</td>
<td>읽을 수 있는 데이터와 문맥</td>
<td>애플리케이션</td>
<td>도서관 이용 안내문</td>
</tr>
<tr>
<td>Tool</td>
<td>입력을 받아 실행하는 기능</td>
<td>모델의 선택을 애플리케이션이 실행</td>
<td>조건별 책 검색, 대출 가능 여부 조회, 예약 생성</td>
</tr>
<tr>
<td>Prompt</td>
<td>반복해서 사용할 작업 요청 양식</td>
<td>사용자</td>
<td>‘책 추천하기’ 질문 템플릿</td>
</tr>
</tbody></table>
<p><strong>Resource는 참고 자료, Tool은 기능 버튼, Prompt는 작업 요청 양식</strong>이라고 기억해 봅시다.</p>
<h4 id="resource-예시">Resource 예시</h4>
<pre><code class="language-text">URI: library://guides/borrowing
내용: 대출 기간과 반납 방법을 설명한 안내문</code></pre>
<p>URI는 자원의 이름표입니다. 반드시 브라우저로 여는 웹 주소일 필요는 없습니다.</p>
<h4 id="tool-예시">Tool 예시</h4>
<pre><code class="language-text">기능 이름: search_books
입력: 검색어와 최대 결과 수
출력: 책 목록, 책 식별자, 검색 결과 수</code></pre>
<p>조회만 하는 기능도 Tool이 될 수 있습니다. ‘실행’은 반드시 데이터를 바꾼다는 뜻이 아닙니다. 계산이나 검색도 함수 실행입니다.</p>
<h4 id="prompt-예시">Prompt 예시</h4>
<pre><code class="language-text">양식 이름: recommend_books
입력: 독자의 나이, 좋아하는 주제
결과: 해당 조건에 맞는 책을 찾고 추천 이유를 설명하도록 구성한 요청문</code></pre>
<p>Prompt를 가져오는 것과 그 요청문을 모델에 보내 실행하는 것은 별도 단계입니다. 양식을 꺼냈다고 추천이 이미 완료된 것은 아닙니다.</p>
<blockquote>
<p><strong>정확히 이해하기:</strong> ‘정적 자료는 Resource, 동적 결과는 Tool’은 처음 분류할 때 도움이 되는 기준입니다. 하지만 Resource도 변경되거나 동적으로 제공될 수 있습니다. 결정할 때는 <strong>데이터를 자원으로 제공하는지, 모델이 입력을 정해 실행할 기능으로 제공하는지</strong>를 함께 봅니다. MCP의 기본 구분은 <a href="https://modelcontextprotocol.io/specification/2025-06-18/server/resources">Resources 공식 명세</a>와 <a href="https://modelcontextprotocol.io/specification/2025-06-18/server/tools">Tools 공식 명세</a>에서 확인할 수 있습니다.</p>
</blockquote>
<h3 id="1-4-무엇이든-tool로-만들면-왜-어려워질까">1-4. 무엇이든 Tool로 만들면 왜 어려워질까?</h3>
<p>리모컨에 ‘전원 켜기’, ‘켜기’, ‘시작하기’라는 버튼이 따로 있는데 모두 같은 일을 한다고 상상해 보세요. 어떤 버튼을 눌러야 할지 헷갈립니다.</p>
<p>AI에게도 도구 이름과 설명이 중요합니다.</p>
<pre><code class="language-text">헷갈리는 예
search_books / find_books / query_books
→ 기능이 거의 같고 구분 기준이 없음

개선한 예
search_books(query, limit)
get_book_availability(book_id)
→ 책을 찾는 일과 현재 대출 가능 여부를 확인하는 일을 구분</code></pre>
<p>모호한 도구가 많으면 잘못된 도구 선택과 불필요한 호출이 늘 수 있습니다. 도구 설명을 모델 입력으로 보내는 과정에도 토큰이 사용됩니다. 다만 비용이 도구 수에 정확히 비례하는 것은 아니며, 설명 길이·전달 방식·캐시 사용 등에 따라 달라집니다.</p>
<p><strong>도구를 적게 만드는 것 자체가 목표가 아니라, 서로 구분하기 쉽게 만드는 것이 목표입니다.</strong></p>
<h3 id="1-5-기능-→-분류-→-책임-점검의-세-단계">1-5. 기능 → 분류 → 책임 점검의 세 단계</h3>
<h4 id="1단계-필요한-일을-쉬운-말로-전부-적기">1단계: 필요한 일을 쉬운 말로 전부 적기</h4>
<p>처음부터 기술 용어로 쓰지 않아도 됩니다.</p>
<ul>
<li>어떤 책이 있는지 찾기</li>
<li>도서관 이용 규칙 읽기</li>
<li>현재 빌릴 수 있는지 확인하기</li>
<li>예약하기</li>
<li>이용자에게 취향을 묻는 추천 양식 제공하기</li>
</ul>
<h4 id="2단계-resource·tool·prompt로-분류하기">2단계: Resource·Tool·Prompt로 분류하기</h4>
<table>
<thead>
<tr>
<th>필요한 기능</th>
<th>설계 예시</th>
<th>분류 이유</th>
</tr>
</thead>
<tbody><tr>
<td>이용 규칙 읽기</td>
<td>Resource</td>
<td>문서 자체를 문맥으로 제공</td>
</tr>
<tr>
<td>조건에 맞는 책 찾기</td>
<td>Tool</td>
<td>검색 조건을 입력받아 실행</td>
</tr>
<tr>
<td>현재 대출 가능 여부 확인</td>
<td>조회 Tool</td>
<td>책 식별자를 받아 현재 상태 조회</td>
</tr>
<tr>
<td>책 예약</td>
<td>쓰기 Tool</td>
<td>실제 시스템의 예약 상태 변경</td>
</tr>
<tr>
<td>추천 요청 양식</td>
<td>Prompt</td>
<td>반복할 요청의 틀 제공</td>
</tr>
</tbody></table>
<p>실무의 운영 지원 도우미에도 같은 방법을 적용할 수 있습니다.</p>
<table>
<thead>
<tr>
<th>필요한 기능</th>
<th>분류 예시</th>
<th>설계할 때 볼 점</th>
</tr>
</thead>
<tbody><tr>
<td>운영 문서 검색</td>
<td>조회 Tool</td>
<td>질문에 맞는 문서를 찾는 기능</td>
</tr>
<tr>
<td>특정 장애 대응 절차 읽기</td>
<td>Resource</td>
<td>주소로 식별되는 매뉴얼 제공</td>
</tr>
<tr>
<td>기존 작업 기록 검색</td>
<td>조회 Tool</td>
<td>검색 조건을 받아 외부 시스템 조회</td>
</tr>
<tr>
<td>새 작업 기록 생성</td>
<td>쓰기 Tool</td>
<td>중복 생성과 권한 확인 필요</td>
</tr>
<tr>
<td>모니터링 상태 확인</td>
<td>조회 Tool</td>
<td>최신 상태와 조회 시각 반환</td>
</tr>
<tr>
<td>표준 진단 요청 양식</td>
<td>Prompt</td>
<td>사용자가 재사용할 작업 순서 제공</td>
</tr>
</tbody></table>
<p>이 표는 가능한 설계안입니다. 시스템의 사용자와 제공 방식에 따라 다른 분류도 가능합니다.</p>
<h4 id="3단계-한-기능의-책임이-너무-크거나-겹치지-않는지-보기">3단계: 한 기능의 책임이 너무 크거나 겹치지 않는지 보기</h4>
<p>‘검색하고, 예약하고, 안내 메시지까지 전송’하는 도구 하나는 편해 보입니다. 그러나 예약만 실패했다면 어디까지 성공했는지 알기 어렵습니다.</p>
<p>기능을 나누면 실패 지점을 알기 쉽고, 쓰기 작업 전에 확인을 받을 수 있습니다. 반대로 언제나 함께 실행되어야 하는 작은 조회들을 적절히 묶으면 불필요한 왕복 호출을 줄일 수 있습니다.</p>
<p><strong>단일 책임 원칙</strong>은 ‘한 함수가 한 줄이어야 한다’는 뜻이 아닙니다. 무엇을 책임지는 기능인지 명확해야 한다는 뜻입니다.</p>
<h3 id="1-6-mcp-server-구현의-뼈대-이해하기">1-6. MCP Server 구현의 뼈대 이해하기</h3>
<p>구현 코드를 처음 보면 다음 단어부터 이해하면 됩니다.</p>
<ul>
<li><strong>인스턴스:</strong> 설계도를 바탕으로 실제 만든 객체 하나입니다. 붕어빵 틀과 붕어빵의 관계입니다.</li>
<li><strong>데코레이터:</strong> 함수 위에 붙여 ‘이 함수를 어떤 기능으로 등록하자’고 표시하는 장치입니다.</li>
<li><strong>스키마:</strong> 입력과 출력에 어떤 항목이 들어가야 하는지 정한 양식입니다.</li>
<li><strong>Docstring/description:</strong> 기능을 언제 어떻게 사용할지 설명하는 안내문입니다.</li>
</ul>
<p>다음은 라이브러리에 종속되지 않는 <strong>설계용 의사코드</strong>입니다. 그대로 실행하는 완성 코드가 아닙니다.</p>
<pre><code class="language-text">서버 = MCP 서버 생성(&quot;library-demo&quot;)

Resource 등록:
    주소 = library://guides/borrowing
    반환 = 대출 안내문

Tool 등록:
    이름 = get_book_availability
    설명 = 책 식별자를 알고 있을 때 현재 대출 가능 여부를 확인한다.
    입력 = book_id: 비어 있지 않은 문자열
    출력 = book_id, available, checked_at

Prompt 등록:
    이름 = recommend_books
    입력 = age_group, topic
    반환 = 추천 작업을 시작할 요청문

서버 실행</code></pre>
<p>이렇게 정의하면 모델이 ‘현재 상태가 필요하구나’라는 상황에서 어떤 도구를 선택할지 이해하기 쉬워집니다.</p>
<h3 id="1-7-mcp-inspector-ai를-연결하기-전에-직접-눌러-보기">1-7. MCP Inspector: AI를 연결하기 전에 직접 눌러 보기</h3>
<p>Inspector는 MCP 서버를 사람이 직접 확인하는 점검 도구입니다. AI의 선택이 맞는지 보기 전에, <strong>기능 자체가 제대로 되는지</strong> 확인합니다.</p>
<p>예를 들어 자판기에서 음료가 안 나온다면, 음성 주문 AI를 고치기 전에 음료 버튼부터 눌러 보는 것입니다.</p>
<p>점검 순서는 다음과 같습니다.</p>
<ol>
<li>서버와 연결되는지 확인합니다.</li>
<li>등록한 Tool·Resource·Prompt 목록을 확인합니다.</li>
<li>도구에 입력을 넣고 직접 실행합니다.</li>
<li>Resource의 구체적인 URI로 내용을 읽습니다.</li>
<li>Prompt 인자를 넣어 완성된 요청문을 확인합니다.</li>
<li>잘못된 입력과 없는 자원을 넣어 실패 처리도 확인합니다.</li>
</ol>
<p><strong>Tool 점검:</strong> 설명만 읽고 사용 시점을 알 수 있는가? 필수 입력과 범위가 명확한가? 결과 구조가 일정한가?</p>
<p><strong>Resource 점검:</strong> 주소가 맞는가? 문서가 읽히는가? 없는 문서와 접근할 수 없는 문서의 오류가 구분되는가?</p>
<p><strong>Prompt 점검:</strong> 필요한 인자가 명확한가? 인자를 빠뜨리면 이해할 수 있는 오류가 나오는가? 완성된 문장에 의도한 작업 순서가 있는가?</p>
<h3 id="1-8-실습-가이드-도구-상자만-먼저-완성하기">1-8. 실습 가이드: 도구 상자만 먼저 완성하기</h3>
<p><strong>목표:</strong> 화면과 AI 없이도 도구를 직접 시험할 수 있게 만듭니다.</p>
<pre><code class="language-text">Inspector → MCP 서버 → 가상 책 데이터
                       → 가상 대출 API</code></pre>
<p>처음에는 실제 외부 서비스 대신 Mock을 사용해도 좋습니다. Mock은 ‘연습용 가짜 시스템’입니다. 실제 예약을 만들지 않고 예약 결과의 모양을 시험할 수 있습니다.</p>
<p><strong>진행 방법</strong></p>
<ol>
<li>공개 가능한 가상 책 데이터와 이용 안내문을 만듭니다.</li>
<li>Resource 1개, 조회 Tool 1개, 추천 Prompt 1개부터 등록합니다.</li>
<li>Inspector에서 각각 직접 확인합니다.</li>
<li>책이 없는 경우와 잘못된 식별자를 전달하는 경우를 시험합니다.</li>
<li>쓰기 Tool을 추가한다면 중복 실행과 사용자 확인을 별도로 설계합니다.</li>
</ol>
<p><strong>성공 확인:</strong> ‘서버가 실행됐다’만으로 끝내지 않습니다. 실제 목록 조회, 정상 호출, 잘못된 입력, 없는 데이터까지 확인해야 합니다. Mock으로 통과한 결과와 실제 API 연동 결과도 구분합니다.</p>
<p><strong>파트 복습:</strong> MCP는 연결 약속, Resource는 자료, Tool은 실행 기능, Prompt는 요청 양식, Inspector는 직접 시험하는 점검 도구입니다.</p>
<hr />
<h2 id="part-2-통합-아키텍처-누가-무슨-일을-맡을까">Part 2. 통합 아키텍처: 누가 무슨 일을 맡을까?</h2>
<h3 id="2-1-아키텍처는-프로그램의-자리와-역할을-정하는-일">2-1. 아키텍처는 프로그램의 자리와 역할을 정하는 일</h3>
<p>같은 도서관 안내 서비스를 만들어도 설계는 달라질 수 있습니다.</p>
<ul>
<li>누가 책을 검색할까요?</li>
<li>예약 시스템은 어느 프로그램에서 연결할까요?</li>
<li>대화는 어디에 저장할까요?</li>
<li>검색 실패는 누가 처리할까요?</li>
</ul>
<p>이 결정을 모은 것이 통합 아키텍처입니다. 집을 짓기 전에 방과 문, 수도관의 위치를 정하는 것과 같습니다.</p>
<p><strong>시스템 경계</strong>는 각 부분이 책임지는 범위를 나누는 선입니다.</p>
<table>
<thead>
<tr>
<th>경계</th>
<th>정할 질문</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>책임 경계</td>
<td>누가 어떤 일을 맡는가?</td>
<td>Backend는 입력 검사, Agent는 작업 선택</td>
</tr>
<tr>
<td>인터페이스 경계</td>
<td>어떤 양식으로 주고받는가?</td>
<td>요청은 JSON, 진행 상황은 이벤트</td>
</tr>
<tr>
<td>데이터 경계</td>
<td>누가 데이터를 소유하고 읽고 바꾸는가?</td>
<td>예약 변경은 대출 시스템만 수행</td>
</tr>
</tbody></table>
<p>컴포넌트는 기능 단위, 모듈은 관련 코드를 묶은 단위, 서비스는 독립 실행·배포할 수 있는 프로그램 단위로 생각하면 됩니다. 예를 들어 ‘대출 기능’이라는 컴포넌트를 여러 코드 모듈로 구현하고, 이를 대출 서비스로 운영할 수 있습니다. 팀에 따라 용어 사용 범위는 조금씩 다릅니다.</p>
<h3 id="2-2-결정-①-검색은-backend가-할까-agent가-할까">2-2. 결정 ① 검색은 Backend가 할까, Agent가 할까?</h3>
<p>Retrieval은 필요한 정보를 찾아 가져오는 일입니다.</p>
<p>RAG는 자료를 검색한 뒤, 그 자료를 답변 생성에 활용하는 방식입니다. 오픈북 시험처럼 참고 자료를 보고 답하는 것입니다. Vector DB는 검색을 돕는 수단 중 하나이고, 그 자체가 답변의 정확성을 보장하지는 않습니다.</p>
<h4 id="방식-a-backend가-먼저-검색">방식 A: Backend가 먼저 검색</h4>
<pre><code class="language-text">질문 → Backend가 자료 검색 → 질문과 검색 자료를 Agent에 전달 → 답변</code></pre>
<p>선생님이 참고 자료를 미리 나눠 주고 학생에게 설명하게 하는 방식입니다. 항상 필요한 검색을 일관되게 적용하기 쉽지만, 검색이 필요 없는 인사말에도 검색한다면 낭비가 생길 수 있습니다.</p>
<h4 id="방식-b-agent가-필요에-따라-검색-tool-호출">방식 B: Agent가 필요에 따라 검색 Tool 호출</h4>
<pre><code class="language-text">질문 → Agent가 필요한 정보 판단 → 검색 Tool 호출 → 결과 확인 → 답변</code></pre>
<p>학생이 문제를 읽고 필요한 책을 찾아보는 방식입니다. 여러 자료를 상황에 맞게 조합하기 좋지만, 검색을 빠뜨리거나 지나치게 반복할 수 있어 실행 규칙과 평가가 필요합니다.</p>
<table>
<thead>
<tr>
<th>기준</th>
<th>Backend 선검색</th>
<th>Agent가 검색 선택</th>
</tr>
</thead>
<tbody><tr>
<td>흐름</td>
<td>비교적 일정</td>
<td>질문에 따라 달라짐</td>
</tr>
<tr>
<td>유연성</td>
<td>정해진 흐름 중심</td>
<td>추가 검색·여러 자료 조합 가능</td>
</tr>
<tr>
<td>점검할 위험</td>
<td>불필요한 검색</td>
<td>검색 누락·반복 호출</td>
</tr>
<tr>
<td>적합한 설계 예</td>
<td>정해진 문서 조회 절차</td>
<td>질문에 맞춰 여러 시스템 조사</td>
</tr>
</tbody></table>
<p>어느 방식을 선택해도 <strong>접근 권한 검사는 서버 쪽에서 강제</strong>해야 합니다. 모델에게 ‘다른 사람 자료를 보지 마’라고 말하는 것만으로는 충분하지 않습니다.</p>
<h3 id="2-3-결정-②-mcp-기능을-어디에서-실행할까">2-3. 결정 ② MCP 기능을 어디에서 실행할까?</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>쉬운 설명</th>
<th>장점</th>
<th>주의할 점</th>
</tr>
</thead>
<tbody><tr>
<td>같은 프로세스의 함수 직접 호출</td>
<td>같은 작업 공간 안에서 함수를 바로 부름</td>
<td>단순하고 통신 단계가 적음</td>
<td>직접 호출만 하면 MCP 프로토콜을 사용한 연결은 아님</td>
</tr>
<tr>
<td>같은 머신의 별도 프로세스</td>
<td>옆 작업실의 프로그램과 연결</td>
<td>실행 환경과 역할을 나눌 수 있음</td>
<td>시작·종료·오류·표준 입출력 관리 필요</td>
</tr>
<tr>
<td>네트워크의 별도 서비스</td>
<td>다른 건물의 부서에 요청</td>
<td>여러 애플리케이션이 공유하고 독립 배포 가능</td>
<td>인증·네트워크·배포 관리 필요</td>
</tr>
</tbody></table>
<p>프로세스는 실행 중인 프로그램의 단위입니다. 별도 프로세스가 곧 별도 컴퓨터라는 뜻은 아닙니다.</p>
<p><code>stdio</code>는 표준 입력과 표준 출력으로 메시지를 주고받는 방식입니다. 이 방식을 쓸 때 디버그 문장을 프로토콜 메시지와 같은 표준 출력에 섞으면 통신을 방해할 수 있으므로 로그 출력 경로를 분리해야 합니다.</p>
<p>네트워크 MCP는 채택한 프로토콜 버전에 맞는 HTTP 전송 방식을 확인해야 합니다. <strong>사용자 화면으로 답변을 보내는 SSE와 MCP의 전송 방식은 다른 연결</strong>이므로 혼동하지 마세요.</p>
<h3 id="2-4-결정-③-대화의-기억을-어디에-둘까">2-4. 결정 ③ 대화의 기억을 어디에 둘까?</h3>
<pre><code class="language-text">학생: 공룡 책을 찾아 줘.
AI: ‘공룡의 하루’가 있어.
학생: 그 책은 빌릴 수 있어?</code></pre>
<p>두 번째 질문의 ‘그 책’을 알려면 이전 대화를 기억해야 합니다.</p>
<table>
<thead>
<tr>
<th>저장 위치</th>
<th>비유</th>
<th>장점</th>
<th>주의할 점</th>
</tr>
</thead>
<tbody><tr>
<td>Frontend</td>
<td>학생이 자기 공책을 들고 옴</td>
<td>작은 시제품에서 이해하기 쉬움</td>
<td>매번 이력 전송, 기기 간 공유·변조·유실 고려</td>
</tr>
<tr>
<td>Backend 메모리</td>
<td>담당 직원 머릿속에 기억</td>
<td>빠르고 개발이 간단</td>
<td>프로세스 종료 시 유실, 다른 인스턴스와 공유 어려움</td>
</tr>
<tr>
<td>외부 DB</td>
<td>공동 기록장</td>
<td>재시작·여러 인스턴스 대응 가능</td>
<td>연결 관리·권한·저장 정책 필요</td>
</tr>
</tbody></table>
<p>브라우저의 저장소에 대화 ID를 남기는 일과, 서버의 DB에 실제 대화 상태를 저장하는 일은 다릅니다. 번호표만 보관한다고 상담 기록까지 보관되는 것은 아닙니다.</p>
<h3 id="2-5-결정-④-실패는-어디에서-처리할까">2-5. 결정 ④ 실패는 어디에서 처리할까?</h3>
<p>책 검색이 실패했다고 가정해 봅시다.</p>
<ol>
<li>도구가 자체적으로 제한된 재시도를 할 수 있습니다.</li>
<li>해결하지 못하면 ‘검색 시스템 연결 실패’라는 원인을 상위 계층에 전달합니다.</li>
<li>Agent가 다른 자료를 쓸 수 있는지 판단합니다.</li>
<li>Backend가 응답 양식을 정리합니다.</li>
<li>Frontend가 사용자에게 상태와 다음 행동을 보여 줍니다.</li>
</ol>
<p><strong>‘검색 결과 0건’과 ‘검색 시스템 고장’은 다른 결과입니다.</strong> 고장을 ‘없음’으로 바꾸면 AI가 책이 없다고 잘못 설명할 수 있습니다.</p>
<p>또한 운영 중 실제 데이터 조회가 실패했다고 Mock 데이터를 진짜 결과처럼 보여 주면 안 됩니다. Mock은 학습·시험 환경용입니다. 대체 데이터나 오래된 자료를 제공한다면 그 한계를 표시해야 합니다.</p>
<h3 id="2-6-이번-프로젝트의-연결-흐름-이해하기">2-6. 이번 프로젝트의 연결 흐름 이해하기</h3>
<p>다음은 본문의 연결 구조를 학습용으로 단순화한 것입니다.</p>
<pre><code class="language-text">[사용자 화면]
      ↓ 질문과 대화 ID
[FastAPI Backend]
      ↓ 입력 검사 후 실행
[LangGraph Agent] ←→ [대화 상태 저장소]
      ↓ 필요한 도구 선택
[MCP Client]
      ↓ MCP 요청
[MCP Server]
      ↓
[검색 데이터 / 외부 API]</code></pre>
<p>LangGraph는 작업 단계를 노드와 연결로 표현하고 상태를 전달하는 도구로 이해하면 됩니다. 모든 프로젝트가 반드시 이 조합을 써야 하는 것은 아닙니다. 여기서는 여러 부분이 연결되는 원리를 배우는 데 집중합니다.</p>
<h3 id="2-7-thread와-checkpointer-대화방과-자동-저장-담당자">2-7. Thread와 Checkpointer: 대화방과 자동 저장 담당자</h3>
<p><strong>Thread는 하나의 대화 흐름을 구분하는 단위</strong>입니다. <code>thread_id</code>는 대화방 번호처럼 사용할 수 있습니다.</p>
<p><strong>State는 그 대화방의 현재 기록</strong>입니다. 메시지, 선택한 책, 도구 결과, 남은 작업 등을 담을 수 있습니다.</p>
<p><strong>Checkpointer는 상태를 저장하고 다시 읽는 역할</strong>입니다. <strong>Checkpoint는 저장된 특정 시점의 상태</strong>입니다.</p>
<table>
<thead>
<tr>
<th>용어</th>
<th>게임 비유</th>
</tr>
</thead>
<tbody><tr>
<td>thread_id</td>
<td>게임 진행 파일의 이름</td>
</tr>
<tr>
<td>State</td>
<td>지금 가진 아이템과 진행 상황</td>
</tr>
<tr>
<td>Checkpointer</td>
<td>자동 저장 기능</td>
</tr>
<tr>
<td>Checkpoint</td>
<td>특정 시점의 저장본</td>
</tr>
<tr>
<td>checkpoint_id</td>
<td>그 저장본의 식별 번호</td>
</tr>
</tbody></table>
<p>LangGraph의 체크포인트는 실행 단계인 super-step 경계에서 저장됩니다. 병렬 실행에서는 노드 하나와 super-step 하나가 항상 같지는 않습니다. 저장소 구현이 메모리라면 프로세스 종료 후에는 남지 않습니다. 관련 구분은 <a href="https://docs.langchain.com/oss/python/langgraph/checkpointers">LangGraph Checkpointers</a>를 참고할 수 있습니다.</p>
<p>설정 모양은 다음과 같습니다.</p>
<pre><code class="language-python"># 설정 구조를 보여 주는 예시입니다.
config = {&quot;configurable&quot;: {&quot;thread_id&quot;: &quot;demo-library-thread&quot;}}</code></pre>
<p>대화마다 다른 ID를 사용하면 기록을 분리할 수 있지만, <strong>ID 분리만으로 다른 사용자의 접근까지 차단되는 것은 아닙니다.</strong> 서버가 소유권을 확인해야 합니다.</p>
<h3 id="2-8-thread-id는-누가-만들까">2-8. Thread ID는 누가 만들까?</h3>
<h4 id="backend가-발급하는-방식">Backend가 발급하는 방식</h4>
<ol>
<li>사용자가 새 대화를 요청합니다.</li>
<li>Backend가 ID를 만듭니다.</li>
<li>Backend가 ID를 화면에 알려 줍니다.</li>
<li>화면이 다음 요청부터 같은 ID를 보냅니다.</li>
</ol>
<p>서버가 대화 생성·목록·소유권을 함께 관리하기 좋습니다. 다만 ‘ID를 서버가 만들었다’는 이유만으로 요청마다 서버 메모리에 현재 대화를 기억해야 하는 것은 아닙니다. ID를 요청에 포함하고 상태를 외부 저장소에서 읽으면 서버도 상태를 분리해 운영할 수 있습니다.</p>
<h4 id="frontend가-발급하는-방식">Frontend가 발급하는 방식</h4>
<ol>
<li>사용자가 ‘새 대화’를 누릅니다.</li>
<li>화면이 ID를 만듭니다.</li>
<li>요청에 ID를 함께 보냅니다.</li>
<li>Backend가 사용자와 ID의 사용 권한을 검증합니다.</li>
</ol>
<p>화면의 새 대화 동작과 연결하기 쉽지만, 다른 기기와 목록을 맞추려면 별도 저장과 동기화가 필요합니다.</p>
<p><strong>ID 발급 위치와 상태 저장 위치는 별도로 결정하는 문제입니다.</strong></p>
<h3 id="2-9-uuid-겹칠-가능성이-매우-낮은-이름표">2-9. UUID: 겹칠 가능성이 매우 낮은 이름표</h3>
<p>UUID는 식별자를 만드는 표준 형식입니다. UUID v4는 무작위 값을 사용합니다. 일반적인 하이픈 표기는 36자입니다.</p>
<pre><code class="language-python">import uuid

thread_id = str(uuid.uuid4())</code></pre>
<pre><code class="language-javascript">// 지원되는 보안 환경의 브라우저에서 사용하는 예시
const threadId = crypto.randomUUID();</code></pre>
<p>Python 코드는 UUID 객체를 문자열로 바꾸어 전달합니다. 브라우저의 <code>crypto.randomUUID()</code>는 HTTPS나 localhost 같은 보안 환경과 브라우저 지원 조건을 확인해야 합니다.</p>
<p>UUID는 비밀번호가 아닙니다. 번호를 알아냈다고 다른 사람의 대화에 접근할 수 있어서는 안 됩니다.</p>
<h3 id="2-10-fastapi-요청을-받는-접수-창구">2-10. FastAPI: 요청을 받는 접수 창구</h3>
<p>FastAPI는 Python으로 API를 만드는 프레임워크입니다.</p>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 설명</th>
</tr>
</thead>
<tbody><tr>
<td>Request</td>
<td>사용자가 보낸 요청</td>
</tr>
<tr>
<td>Response</td>
<td>서버가 돌려주는 결과</td>
</tr>
<tr>
<td>Endpoint</td>
<td>업무별 접수 주소</td>
</tr>
<tr>
<td>JSON</td>
<td>프로그램끼리 데이터를 주고받는 표기법</td>
</tr>
<tr>
<td>Schema</td>
<td>입력·출력의 양식과 타입</td>
</tr>
<tr>
<td>GET</td>
<td>주로 자료를 조회할 때 사용하는 HTTP 메서드</td>
</tr>
<tr>
<td>POST</td>
<td>데이터를 보내 처리나 생성을 요청할 때 사용하는 HTTP 메서드</td>
</tr>
</tbody></table>
<pre><code class="language-json">{
  &quot;message&quot;: &quot;공룡 책을 찾아 줘&quot;,
  &quot;thread_id&quot;: &quot;demo-library-thread&quot;
}</code></pre>
<p>위 JSON은 요청 형태의 예시입니다. 실제 API에서는 서버가 인증 정보로 사용자를 확인하고, ID 소유권과 메시지 길이 등을 검사합니다.</p>
<h3 id="2-11-실습-가이드-화면-없이-api부터-연결하기">2-11. 실습 가이드: 화면 없이 API부터 연결하기</h3>
<p><strong>목표:</strong> HTTP 요청 한 번이 Agent와 MCP 도구까지 전달되어 응답하는지 확인합니다.</p>
<ol>
<li>상태 확인용 접수 주소와 채팅 접수 주소를 준비합니다.</li>
<li>요청 스키마를 정하고 빈 질문을 거절하도록 합니다.</li>
<li>Agent와 MCP Client를 연결합니다.</li>
<li>같은 대화 ID로 이전 질문을 이어서 물어봅니다.</li>
<li>다른 대화 ID에서는 이전 대화가 섞이지 않는지 확인합니다.</li>
<li>도구 서버를 중단해 오류가 어느 단계에서 발생하는지 확인합니다.</li>
</ol>
<pre><code class="language-text">시험 A: 책을 찾아 줘 → 검색 결과가 있는가?
시험 B: 그 책을 설명해 줘 → 같은 대화에서 앞의 책을 기억하는가?
시험 C: 새 대화에서 ‘그 책’을 물음 → 근거 없이 앞 대화 내용을 가져오지 않는가?
시험 D: 검색 서버 중단 → ‘검색 고장’을 ‘책 없음’과 구분하는가?</code></pre>
<p><strong>성공 확인:</strong> 상태 확인 API가 응답하는 것과 전체 채팅 기능이 동작하는 것은 다릅니다. 실제 채팅 요청으로 도구 호출과 대화 분리까지 확인합니다.</p>
<p><strong>파트 복습:</strong> 검색 담당, 도구 실행 위치, 기억 저장 위치, 오류 책임을 먼저 정하면 연결과 문제 해결이 쉬워집니다.</p>
<hr />
<h2 id="part-3-streaming-기다리는-동안-무슨-일이-일어나는지-보여-주기">Part 3. Streaming: 기다리는 동안 무슨 일이 일어나는지 보여 주기</h2>
<h3 id="3-1-한-번에-주기와-조금씩-보내기의-차이">3-1. 한 번에 주기와 조금씩 보내기의 차이</h3>
<p>식당에서 1분 동안 아무 말도 없다면 주문이 들어갔는지 불안할 수 있습니다. ‘주문 접수 → 조리 중 → 포장 중’이라고 보여 주면 상황을 이해할 수 있습니다.</p>
<p>Streaming은 데이터를 완성될 때까지 기다렸다 한 번에 보내는 대신, <strong>준비되는 대로 나누어 전달하는 방식</strong>입니다.</p>
<pre><code class="language-text">일반 응답:
질문 → 오래 기다림 → 완성된 답변

스트리밍 응답:
질문 → 검색 시작 → 검색 완료 → 답변 일부 → 나머지 답변 → 완료</code></pre>
<p>스트리밍은 사용자가 기다리는 경험을 개선합니다. 하지만 모델이 계산해야 하는 총시간이나 총비용이 반드시 줄어드는 것은 아닙니다.</p>
<h3 id="3-2-langgraph의-stream과-브라우저-streaming은-다르다">3-2. LangGraph의 stream과 브라우저 Streaming은 다르다</h3>
<p>서버 코드가 <code>stream()</code> 또는 <code>astream()</code>으로 결과를 조금씩 받더라도, 브라우저에 자동으로 전달되지는 않습니다.</p>
<p>주방에서 음식이 조금씩 나오는 것과, 직원이 손님에게 접시를 가져가는 것은 다른 일입니다.</p>
<pre><code class="language-text">Agent 내부 이벤트
    ↓
화면에 필요한 이벤트로 변환
    ↓
HTTP 스트림 형식으로 포장
    ↓
네트워크 전송
    ↓
브라우저에서 해석
    ↓
화면 갱신</code></pre>
<p><code>invoke()</code>는 보통 완료 결과를 받는 호출로, <code>stream()</code>/<code>astream()</code>은 점진적으로 나오는 결과를 받는 통로로 이해하면 됩니다. 정확히 어떤 데이터가 나오는지는 설정한 스트림 모드에 따라 달라집니다.</p>
<h3 id="3-3-streaming을-설계하는-세-가지-질문">3-3. Streaming을 설계하는 세 가지 질문</h3>
<table>
<thead>
<tr>
<th>차원</th>
<th>질문</th>
<th>택배 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Transport</td>
<td>어떤 통로로 보낼까?</td>
<td>자동차, 기차 중 배송 수단 선택</td>
</tr>
<tr>
<td>Content</td>
<td>무엇을 보낼까?</td>
<td>상자에 담을 물건 선택</td>
</tr>
<tr>
<td>Lifecycle</td>
<td>언제 시작하고 어떻게 끝낼까?</td>
<td>접수·배송·수령·분실 처리</td>
</tr>
</tbody></table>
<p>통로만 만들고 보낼 내용과 종료 규칙을 정하지 않으면, 화면은 무엇을 표시할지 알 수 없습니다.</p>
<h3 id="3-4-transport-비교하기">3-4. Transport 비교하기</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>개념</th>
<th>활용 예</th>
<th>고려할 점</th>
</tr>
</thead>
<tbody><tr>
<td>HTTP 응답 스트리밍</td>
<td>응답 본문을 나누어 전달</td>
<td>점진적인 텍스트 응답</td>
<td>청크와 논리적 메시지는 다를 수 있음</td>
</tr>
<tr>
<td>SSE</td>
<td>HTTP로 텍스트 이벤트를 서버에서 클라이언트로 전달</td>
<td>채팅 응답과 진행 알림</td>
<td>응답 채널은 단방향, 프록시 버퍼링 확인</td>
</tr>
<tr>
<td>WebSocket</td>
<td>연결 후 양방향으로 메시지 교환</td>
<td>실시간 협업·게임</td>
<td>연결 상태와 재연결 관리 필요</td>
</tr>
<tr>
<td>gRPC Streaming</td>
<td>서비스 간 타입 기반 스트리밍 RPC</td>
<td>백엔드 서비스 간 통신</td>
<td>브라우저 직접 사용에 제약</td>
</tr>
</tbody></table>
<p>SSE는 HTTP 스트리밍 위에 이벤트 형식을 더한 방식으로 볼 수 있습니다. 둘은 완전히 별개의 종류라기보다 관련된 개념입니다. HTTP 버전에 따라 실제 전송 처리도 달라지므로 모든 스트림을 HTTP/1.1의 chunked 전송과 동일시하지 않습니다.</p>
<p>일반적인 질문·응답형 채팅은 사용자가 질문을 보내고 서버가 진행 상황을 계속 돌려주는 구조라 SSE로 구현하기 좋습니다.</p>
<h3 id="3-5-sse-메시지는-어떻게-생겼을까">3-5. SSE 메시지는 어떻게 생겼을까?</h3>
<pre><code class="language-text">event: tool_start
data: {&quot;tool_call_id&quot;:&quot;call-1&quot;,&quot;name&quot;:&quot;search_books&quot;}

event: tool_end
data: {&quot;tool_call_id&quot;:&quot;call-1&quot;,&quot;result_count&quot;:2}

event: token
data: {&quot;text&quot;:&quot;공룡 책을 찾았어요.&quot;}

event: done
data: {&quot;status&quot;:&quot;completed&quot;}
</code></pre>
<p><code>event</code>는 소식의 종류, <code>data</code>는 실제 내용입니다. <strong>메시지 사이의 빈 줄</strong>이 이벤트 경계를 표시합니다. 응답의 미디어 타입은 <code>text/event-stream</code>입니다.</p>
<p>이 예시는 직접 설계한 이벤트 규약입니다. 특정 SDK가 그대로 읽을 수 있다고 가정하지 않습니다.</p>
<p>또한 네트워크에서 한 번 읽은 데이터 조각이 SSE 이벤트 한 개와 일치하지 않을 수 있습니다. JSON 중간이 잘리거나 이벤트 여러 개가 함께 도착할 수 있으므로, 수신 측에서 남은 문자열을 보관하고 완성된 이벤트를 구분해야 합니다.</p>
<h4 id="get과-post의-차이도-확인하기">GET과 POST의 차이도 확인하기</h4>
<p>브라우저의 기본 <code>EventSource</code>는 GET 기반으로 연결합니다. 질문을 JSON으로 보내는 POST 스트리밍 API를 쓴다면 <code>fetch()</code>의 응답 스트림을 읽거나, 그 방식을 지원하는 SDK transport를 사용해야 합니다.</p>
<p>SSE 응답 채널이 단방향이어도 별도 요청으로 ‘작업 취소’를 보내는 설계는 가능합니다. 다만 취소 요청을 받는 API와 서버의 작업 취소 처리가 따로 필요합니다.</p>
<h3 id="3-6-content-답변-글자-이외에-무엇을-보낼까">3-6. Content: 답변 글자 이외에 무엇을 보낼까?</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>화면에 보여 줄 내용</th>
</tr>
</thead>
<tbody><tr>
<td>답변 조각</td>
<td>완성 중인 답변</td>
</tr>
<tr>
<td>도구 시작·종료</td>
<td>책 검색 중, 상태 조회 완료</td>
</tr>
<tr>
<td>작업 상태</td>
<td>대기·진행·성공·실패</td>
</tr>
<tr>
<td>메타데이터</td>
<td>요청 식별자, 처리 시간, 필요한 사용량 정보</td>
</tr>
<tr>
<td>오류</td>
<td>실패한 기능과 다시 시도할 수 있는지 여부</td>
</tr>
<tr>
<td>전체 완료</td>
<td>더 이상 기다릴 필요가 없다는 신호</td>
</tr>
</tbody></table>
<p><strong>보여 줄 것은 사용자가 이해할 수 있는 진행 상태와 근거입니다.</strong> 모델의 비공개 추론 원문이나 전체 내부 상태를 그대로 노출할 필요는 없습니다. 도구 입력과 결과에도 개인정보가 포함될 수 있어 공개할 필드를 골라야 합니다.</p>
<h3 id="3-7-event-schema-소식에-공통-양식-붙이기">3-7. Event Schema: 소식에 공통 양식 붙이기</h3>
<p>아래처럼 모두 <code>data</code>라는 이름으로 보내면 내용이 텍스트인지 오류인지 구분하기 어렵습니다.</p>
<pre><code class="language-json">{&quot;data&quot;: &quot;책을 찾았어요&quot;}</code></pre>
<p>종류와 식별자를 명확히 정해 봅시다.</p>
<pre><code class="language-json">{
  &quot;type&quot;: &quot;task_completed&quot;,
  &quot;schema_version&quot;: 1,
  &quot;request_id&quot;: &quot;demo-request&quot;,
  &quot;task_id&quot;: &quot;search-task&quot;,
  &quot;status&quot;: &quot;succeeded&quot;,
  &quot;result_count&quot;: 2
}</code></pre>
<p>다음은 설계용 TypeScript 타입 예시입니다.</p>
<pre><code class="language-typescript">type ChatEvent =
  | { type: &quot;token&quot;; text: string }
  | { type: &quot;tool_start&quot;; toolCallId: string; name: string }
  | { type: &quot;tool_end&quot;; toolCallId: string; ok: boolean }
  | { type: &quot;error&quot;; code: string; message: string }
  | { type: &quot;done&quot;; status: &quot;completed&quot; | &quot;partial&quot; | &quot;failed&quot; };</code></pre>
<p>이벤트마다 필요한 필드를 정하면 화면 코드가 간단해집니다. 다만 TypeScript 타입은 실행 중 수신한 JSON을 자동 검증하지 않습니다. 실제 입력 경계에서는 별도의 스키마 검증도 필요합니다.</p>
<p>알 수 없는 이벤트가 추가되었을 때 화면 전체가 멈추지 않도록 확장 정책을 정하고, 동시에 파싱 오류를 기록해 잘못된 데이터가 조용히 사라지지 않게 합니다.</p>
<h3 id="3-8-lifecycle-시작·진행·종료·실패를-정하기">3-8. Lifecycle: 시작·진행·종료·실패를 정하기</h3>
<table>
<thead>
<tr>
<th>단계</th>
<th>정할 것</th>
<th>화면 예시</th>
</tr>
</thead>
<tbody><tr>
<td>시작</td>
<td>요청/실행 ID, 대화 ID, 초기 상태</td>
<td>요청을 받았어요</td>
</tr>
<tr>
<td>진행</td>
<td>도구 이벤트, 하트비트, 시간 제한</td>
<td>책을 찾고 있어요</td>
</tr>
<tr>
<td>정상 완료</td>
<td>완료 이벤트와 최종 상태</td>
<td>답변이 완료됐어요</td>
</tr>
<tr>
<td>부분 완료</td>
<td>성공한 결과와 부족한 정보</td>
<td>책 목록만 확인했어요</td>
</tr>
<tr>
<td>실패</td>
<td>오류 코드, 재시도 가능 여부</td>
<td>상태 조회에 실패했어요</td>
</tr>
<tr>
<td>연결 끊김</td>
<td>중단·계속 실행·부분 저장 정책</td>
<td>연결이 끊겼어요</td>
</tr>
</tbody></table>
<p>연결이 닫혔다고 항상 정상 완료는 아닙니다. <code>done</code> 같은 명시적인 완료 신호가 있으면 정상 종료와 중간 단절을 구분하기 쉽습니다.</p>
<p>HTTP 응답을 시작하기 전 오류는 일반 HTTP 오류 상태로 보낼 수 있습니다. 이미 스트림 응답을 시작했다면 상태 코드를 새로 바꾸기 어려우므로, 스트림 안의 오류 이벤트와 종료 정책을 사용해야 합니다.</p>
<h3 id="3-9-긴-도구-실행-heartbeat로-연결-유지하기">3-9. 긴 도구 실행: Heartbeat로 연결 유지하기</h3>
<p>외부 조회가 오래 걸려 한동안 아무 데이터도 보내지 않으면, 사용자나 중간 프록시가 멈췄다고 판단할 수 있습니다.</p>
<p>Heartbeat는 ‘연결이 살아 있어요’라는 신호입니다.</p>
<pre><code class="language-text">: keepalive
</code></pre>
<p>SSE에서는 위와 같은 주석을 전송할 수 있습니다. 전송 주기는 실제 프록시·서버의 idle timeout보다 충분히 짧게 정합니다. 고정된 만능 주기는 없습니다.</p>
<p>도구 시작 이벤트 한 번을 보냈다고 긴 작업 내내 연결이 유지되는 것은 아닙니다. 주기적인 신호, 실제 flush, 프록시 버퍼링 설정을 함께 확인해야 합니다. 또한 Heartbeat는 연결의 생존 신호이지, 업무가 실제로 진전하고 있다는 증거는 아닙니다.</p>
<h3 id="3-10-사용자가-창을-닫으면-어떻게-할까">3-10. 사용자가 창을 닫으면 어떻게 할까?</h3>
<table>
<thead>
<tr>
<th>정책</th>
<th>장점</th>
<th>고려할 점</th>
</tr>
</thead>
<tbody><tr>
<td>즉시 취소</td>
<td>불필요한 실행과 비용 감소</td>
<td>부분 결과 처리 필요</td>
</tr>
<tr>
<td>끝까지 실행</td>
<td>돌아왔을 때 결과 확인 가능</td>
<td>백그라운드 작업·저장·비용 정책 필요</td>
</tr>
<tr>
<td>부분 저장 후 취소</td>
<td>진행 상태를 남기면서 실행 중단</td>
<td>구현과 재개 규칙이 복잡</td>
</tr>
</tbody></table>
<p>‘화면 연결 끊기’, ‘Agent 작업 취소’, ‘외부 API 실행 취소’는 서로 다른 일입니다. 이미 처리된 예약은 화면을 닫았다고 자동으로 취소되지 않습니다. 작업마다 취소 가능 여부와 실제 처리 결과를 확인해야 합니다.</p>
<h3 id="3-11-backpressure-받는-속도가-느리면-어떻게-할까">3-11. Backpressure: 받는 속도가 느리면 어떻게 할까?</h3>
<p>공을 1초에 100개 던지는데 친구는 10개만 받을 수 있다면 공이 쌓입니다. 서버가 이벤트를 보내는 속도보다 브라우저가 처리하는 속도가 느려도 비슷한 일이 생깁니다.</p>
<p>대응 방법은 다음과 같습니다.</p>
<ul>
<li>작은 텍스트 조각을 잠깐 모아서 전달하거나 화면을 묶어서 갱신합니다.</li>
<li>메모리 큐에 상한을 두고, 한도를 넘을 때의 정책을 정합니다.</li>
<li>TCP 흐름 제어와 별개로 애플리케이션 내부 큐가 계속 쌓이지 않는지 봅니다.</li>
<li>필요하면 처리 확인 신호인 ACK를 설계합니다. SSE 자체의 역방향 메시지는 아니므로 별도 요청이 필요합니다.</li>
</ul>
<p>텍스트를 조금씩 묶는 것은 가능하지만, 도구 실패나 완료 같은 중요한 제어 이벤트를 임의로 버리면 안 됩니다.</p>
<h3 id="3-12-frontend-선택-streamlit-react-nextjs">3-12. Frontend 선택: Streamlit, React, Next.js</h3>
<table>
<thead>
<tr>
<th>도구</th>
<th>쉬운 설명</th>
<th>유리한 점</th>
<th>추가로 고민할 점</th>
</tr>
</thead>
<tbody><tr>
<td>Streamlit</td>
<td>Python으로 화면을 빠르게 구성하는 도구</td>
<td>분석 데모와 시제품 작성이 쉬움</td>
<td>세밀한 UI·상태·배포 요구사항</td>
</tr>
<tr>
<td>React</td>
<td>화면을 작은 부품으로 구성하는 라이브러리</td>
<td>복잡한 화면과 상호작용을 자유롭게 구성</td>
<td>라우팅·서버 연결·상태 관리 선택</td>
</tr>
<tr>
<td>Next.js</td>
<td>React 기반 앱 구성을 돕는 프레임워크</td>
<td>화면·라우팅·서버 기능을 함께 구성</td>
<td>클라이언트/서버 경계와 배포 방식 이해</td>
</tr>
</tbody></table>
<p>초보자는 ‘가장 유명한 도구’보다 만들 화면의 복잡도와 팀의 익숙함을 기준으로 선택하면 좋습니다. 이 프로젝트는 Python Agent와 별도 웹 화면을 연결하는 구조를 학습합니다.</p>
<h3 id="3-13-nextjs의-주요-개념을-메뉴판에-비유하기">3-13. Next.js의 주요 개념을 메뉴판에 비유하기</h3>
<ul>
<li><strong>Component:</strong> 입력창, 메시지, 작업 카드처럼 재사용하는 화면 부품입니다.</li>
<li><strong>Router:</strong> 주소마다 어느 화면을 보여 줄지 정합니다.</li>
<li><strong>SSR:</strong> 주문이 들어올 때마다 HTML을 만드는 방식입니다.</li>
<li><strong>SSG:</strong> 미리 만들어 둔 HTML을 제공하는 방식입니다.</li>
<li><strong>ISR:</strong> 미리 만든 결과를 사용하면서 정해진 조건에 따라 갱신하는 방식입니다.</li>
<li><strong>CSR:</strong> 브라우저의 JavaScript로 화면을 구성하는 방식입니다.</li>
<li><strong>RSC:</strong> 서버에서 실행되는 React 컴포넌트 모델입니다. SSR과 같은 뜻은 아닙니다.</li>
<li><strong>Hydration:</strong> 서버가 보낸 화면에 브라우저 쪽 동작을 연결하는 과정입니다.</li>
<li><strong>Route Handler/API 기능:</strong> 서버에서 요청을 처리하는 창구입니다.</li>
</ul>
<p>Next.js가 풀스택 기능을 제공하므로 모든 AI 앱에 별도 Backend가 필수인 것은 아닙니다. 여기서는 기존 Python Agent를 활용하기 위해 FastAPI를 별도로 연결하는 것입니다.</p>
<h3 id="3-14-vercel-ai-sdk-채팅을-만드는-공통-도구-모음">3-14. Vercel AI SDK: 채팅을 만드는 공통 도구 모음</h3>
<p>SDK는 자주 쓰는 기능을 미리 묶어 놓은 개발 도구입니다.</p>
<p>AI SDK의 역할은 크게 모델 호출을 다루는 Core, 채팅 화면 상태를 돕는 UI 기능, 모델 제공자와 연결하는 Provider로 이해하면 됩니다. Hook은 React에서 상태나 동작을 재사용하는 함수 형태입니다. <code>useChat</code> 같은 기능이 이에 해당합니다.</p>
<p>중요한 연결 규칙이 있습니다.</p>
<p><strong>직접 만든 SSE 이벤트가 AI SDK의 기본 채팅 프로토콜과 자동으로 호환되는 것은 아닙니다.</strong></p>
<p>다음 중 한 방식을 선택합니다.</p>
<ol>
<li>자체 SSE 규약을 사용하고 화면에서 직접 파싱합니다.</li>
<li>Backend가 AI SDK의 메시지 스트림 규약을 따르도록 변환합니다.</li>
<li>지원되는 사용자 정의 transport를 구현합니다.</li>
</ol>
<p>예를 들어 <code>{ &quot;type&quot;: &quot;token&quot;, &quot;text&quot;: &quot;안녕&quot; }</code>을 보낸다고 기본 <code>useChat</code>이 반드시 처리하는 것은 아닙니다. 설치한 SDK 버전의 메시지 조각·종료 규칙·헤더 요구사항을 함께 맞춰야 합니다. 자세한 규약은 <a href="https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol">AI SDK Stream Protocols</a>에서 확인합니다.</p>
<p>모델 제공자를 쉽게 바꾸도록 도와주더라도 지원 기능, 도구 호출 결과, 지연, 비용까지 같아지는 것은 아니므로 변경 후 평가가 필요합니다.</p>
<h3 id="3-15-실습-가이드-①-단일-agent-streaming">3-15. 실습 가이드 ① 단일 Agent Streaming</h3>
<p><strong>목표:</strong> 검색 진행 상황과 답변이 화면에 점진적으로 나타나게 합니다.</p>
<pre><code class="language-text">Agent 이벤트
   ↓ 의미를 고르는 변환기
화면용 이벤트
   ↓ SSE 형식으로 포장
전송 스트림
   ↓ 브라우저에서 파싱
도구 상태 카드 / 답변 영역</code></pre>
<p><strong>구현 순서</strong></p>
<ol>
<li><code>token</code>, <code>tool_start</code>, <code>tool_end</code>, <code>error</code>, <code>done</code>부터 정의합니다.</li>
<li>Backend에서 Agent 이벤트를 화면용 이벤트로 바꿉니다.</li>
<li>SSE로 포장하고 즉시 전달되도록 합니다.</li>
<li>화면에서 진행 중인 도구와 답변을 따로 표시합니다.</li>
<li>성공·실패·연결 끊김 시 입력창과 로딩 표시를 정리합니다.</li>
<li>긴 작업, 빈 답변, 도구 실패, 잘린 네트워크 청크를 시험합니다.</li>
</ol>
<p><strong>성공 확인:</strong> 글자가 조금씩 나오는 것뿐 아니라, 도구 상태와 종료 표시가 정확한지 확인합니다. 내부 상태나 비밀 정보가 그대로 전송되지 않는지도 봅니다.</p>
<h3 id="3-16-실습-가이드-②-multi-agent-streaming">3-16. 실습 가이드 ② Multi-Agent Streaming</h3>
<p>멀티에이전트는 여러 역할의 에이전트가 협력하는 구조입니다. 학교 조별 활동을 생각해 봅시다.</p>
<pre><code class="language-text">총괄 담당: 질문을 보고 해야 할 작업을 나눔
    ├─ 자료 조사 담당: 관련 책과 설명 검색
    ├─ 상태 확인 담당: 현재 대출 상태 조회
    └─ 정리 담당: 결과를 읽기 쉬운 답변으로 작성</code></pre>
<p>총괄 역할을 Supervisor라고 부를 수 있습니다. 계획을 세우고 작업을 배정하며 완료 여부를 확인합니다.</p>
<p>화면에는 다음 상태를 구분해 보여 줍니다.</p>
<pre><code class="language-text">계획 생성 완료
작업 1: 관련 책 검색 중
작업 1: 성공
작업 2: 대출 상태 확인 중
작업 2: 실패
최종 답변: 책 추천은 가능하지만 대출 상태는 확인하지 못함</code></pre>
<p>이벤트에 <code>task_id</code>, <code>agent_id</code>, <code>tool_call_id</code> 같은 식별자를 붙이면 동시에 실행되는 작업을 구분하기 쉽습니다. 작업 종료와 전체 요청 종료도 구분합니다.</p>
<p>단, 상태 확인이 검색된 책의 ID를 필요로 한다면 검색 전에 시작할 수 없습니다. <strong>여러 에이전트가 있다는 이유로 모든 작업을 병렬로 실행하지 않습니다.</strong></p>
<p><strong>성공 확인:</strong> 작업별 상태, 실패한 작업, 최종 통합 결과가 맞는지 확인합니다. 한 작업의 실패가 전체 성공으로 잘못 표시되지 않아야 합니다.</p>
<p><strong>파트 복습:</strong> 스트리밍은 통로·내용·생명주기의 설계입니다. 내부 이벤트를 사용자가 이해할 소식으로 바꿔서 전달해야 합니다.</p>
<hr />
<h2 id="part-4-통합-환경의-운영-계속-믿고-쓸-수-있게-만들기">Part 4. 통합 환경의 운영: 계속 믿고 쓸 수 있게 만들기</h2>
<h3 id="4-1-개발할-때-한-번-성공하는-것과-운영은-다르다">4-1. 개발할 때 한 번 성공하는 것과 운영은 다르다</h3>
<p>내 컴퓨터에서 한 명이 사용할 때와, 여러 사람이 동시에 오래 사용할 때는 문제가 달라집니다.</p>
<ul>
<li>어떤 요청만 유난히 느립니다.</li>
<li>답변은 나오지만 근거가 틀립니다.</li>
<li>외부 API가 가끔 실패합니다.</li>
<li>대화가 길어지면서 비용이 늘어납니다.</li>
<li>서버를 재시작하자 대화 기록이 사라집니다.</li>
</ul>
<p>운영은 이런 문제를 발견하고, 원인을 찾고, 개선한 뒤 다시 확인하는 과정입니다.</p>
<h3 id="4-2-observability-겉의-기록으로-속을-이해하기">4-2. Observability: 겉의 기록으로 속을 이해하기</h3>
<p>자동차 계기판과 점검 기록을 보면 차 안에서 어떤 문제가 생겼는지 추정할 수 있습니다. 프로그램도 마찬가지입니다.</p>
<p>Observability, 즉 관찰 가능성은 외부로 수집한 정보를 연결해 내부 동작을 이해하는 능력입니다.</p>
<table>
<thead>
<tr>
<th>정보</th>
<th>쉬운 뜻</th>
<th>도서관 AI 예시</th>
</tr>
</thead>
<tbody><tr>
<td>Metrics</td>
<td>숫자로 보는 변화</td>
<td>요청 수, 오류율, 응답 시간</td>
</tr>
<tr>
<td>Logs</td>
<td>특정 사건의 상세 기록</td>
<td>검색 도구가 시간 초과됨</td>
</tr>
<tr>
<td>Traces</td>
<td>요청 하나의 전체 이동 경로</td>
<td>접수 → 검색 → 상태 조회 → 답변</td>
</tr>
<tr>
<td>Span</td>
<td>Trace 안의 한 작업 구간</td>
<td>검색 도구 실행 구간</td>
</tr>
<tr>
<td>Metadata</td>
<td>기록을 분류하는 정보</td>
<td>실행 버전, 요청 ID, 시험 환경</td>
</tr>
</tbody></table>
<p>Monitoring은 미리 정한 지표와 기준으로 상태를 살피는 데 초점이 있고, Observability는 기록을 연결해 원인을 탐구하는 데 도움을 줍니다. 실제 도구는 이 기능들을 함께 제공하기도 합니다.</p>
<p>관찰 도구를 켰다고 모든 원인이 자동으로 밝혀지는 것은 아닙니다. 요청과 단계가 연결되도록 식별자를 붙이고, 필요한 구간을 기록해야 합니다.</p>
<h3 id="4-3-agent는-왜-전체-흐름을-봐야-할까">4-3. Agent는 왜 전체 흐름을 봐야 할까?</h3>
<p>최종 답변이 12초 뒤에 나왔다고 가정해 봅시다. 다음은 설명용 가상 측정입니다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th align="right">걸린 시간</th>
</tr>
</thead>
<tbody><tr>
<td>질문 해석</td>
<td align="right">1초</td>
</tr>
<tr>
<td>책 검색</td>
<td align="right">8초</td>
</tr>
<tr>
<td>답변 생성</td>
<td align="right">3초</td>
</tr>
</tbody></table>
<p>최종 시간만 보면 모델이 느리다고 생각할 수 있습니다. Trace를 보면 검색이 오래 걸렸음을 알 수 있습니다.</p>
<p>멀티에이전트에서는 호출이 더 많아져 다음을 확인할 필요가 있습니다.</p>
<ul>
<li>같은 도구를 이유 없이 반복했는가?</li>
<li>검색 결과가 비어 다시 검색했는가?</li>
<li>재시도가 전체 시간을 늘렸는가?</li>
<li>어떤 단계의 입력·출력 토큰이 늘었는가?</li>
<li>부분 실패를 마지막 답변에서 숨겼는가?</li>
</ul>
<p>로그나 추적 기록에도 사용자 입력과 문서 내용이 들어갈 수 있습니다. 운영 데이터를 공개용 예제에 복사하지 말고, 수집 항목·마스킹·보관 범위를 정합니다.</p>
<h3 id="4-4-eval-답이-좋은지-기준을-세워-확인하기">4-4. Eval: 답이 좋은지 기준을 세워 확인하기</h3>
<p>Observability가 ‘무슨 일이 일어났지?’라면 Eval은 ‘결과가 기준에 맞을까?’를 확인합니다.</p>
<p>선생님이 시험 답안을 채점하는 상황과 비슷합니다.</p>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td>Dataset</td>
<td>시험할 질문과 기대 결과·기준을 모은 자료</td>
</tr>
<tr>
<td>Target</td>
<td>평가할 Agent·함수·전체 시스템</td>
</tr>
<tr>
<td>Evaluator</td>
<td>결과를 판단하는 규칙이나 채점 함수</td>
</tr>
<tr>
<td>Score</td>
<td>평가 결과</td>
</tr>
<tr>
<td>Experiment</td>
<td>특정 설정으로 데이터셋을 실행해 얻은 결과 묶음</td>
</tr>
<tr>
<td>Baseline</td>
<td>개선 전 비교 기준</td>
</tr>
</tbody></table>
<p>예를 들어 ‘예약 전에 사용자의 확인을 받는가?’는 최종 문장만으로 판단하기 어렵습니다. 실제 도구 호출 기록과 상태도 함께 봐야 합니다.</p>
<h4 id="평가-방식-세-가지">평가 방식 세 가지</h4>
<table>
<thead>
<tr>
<th>방식</th>
<th>설명</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>Reference-based</td>
<td>정답이나 기준 결과와 비교</td>
<td>책 식별자가 기대한 값인가?</td>
</tr>
<tr>
<td>Heuristic</td>
<td>명시적인 규칙으로 검사</td>
<td>완료 상태가 존재하는가? 예약 Tool이 중복 호출되지 않았는가?</td>
</tr>
<tr>
<td>LLM-as-a-Judge</td>
<td>모델이 평가 기준에 따라 판단</td>
<td>추천 이유가 질문과 관련 있고 근거에 맞는가?</td>
</tr>
</tbody></table>
<p>LLM 심사위원도 틀릴 수 있습니다. 명확한 Rubric, 즉 채점 기준을 주고, 일부는 사람이 확인하며, 반복 평가의 변동도 살펴야 합니다. 평가 모델 호출 비용도 고려합니다.</p>
<h3 id="4-5-평가-데이터셋은-어떻게-만들까">4-5. 평가 데이터셋은 어떻게 만들까?</h3>
<p>다음은 학습용 설계 예시입니다. 실제 측정 결과가 아닙니다.</p>
<table>
<thead>
<tr>
<th>질문·상황</th>
<th>기대 동작</th>
<th>확인 방법</th>
</tr>
</thead>
<tbody><tr>
<td>공룡 책 추천</td>
<td>관련 자료 검색 후 추천</td>
<td>도구 호출과 추천 근거 확인</td>
</tr>
<tr>
<td>없는 책 검색</td>
<td>찾지 못했다고 안내</td>
<td>존재하지 않는 책을 지어내지 않는지 확인</td>
</tr>
<tr>
<td>그 책 빌릴 수 있어?</td>
<td>같은 대화의 책 확인 후 상태 조회</td>
<td>State와 조회 입력 확인</td>
</tr>
<tr>
<td>상태 조회 API 중단</td>
<td>추천과 조회 실패를 구분</td>
<td>부분 완료 상태와 오류 안내 확인</td>
</tr>
<tr>
<td>예약 응답이 시간 초과</td>
<td>처리 결과 확인 후 중복 방지</td>
<td>실제 예약 건수 확인</td>
</tr>
<tr>
<td>다른 사람의 대화 ID 요청</td>
<td>접근 거절</td>
<td>인증·소유권 검사 확인</td>
</tr>
</tbody></table>
<p>같은 데이터셋으로 변경 전후를 비교해야 개선 여부를 판단하기 좋습니다. 답변 품질뿐 아니라 작업 성공률·실패 안내·시간·비용도 함께 봅니다.</p>
<h3 id="4-6-관찰과-평가를-연결하는-개선-루프">4-6. 관찰과 평가를 연결하는 개선 루프</h3>
<pre><code class="language-text">기준 데이터셋 만들기
    ↓
현재 버전 평가로 Baseline 확보
    ↓
운영 중 Trace 관찰
    ↓
실패 사례를 비식별화해 시험 항목으로 추가
    ↓
원인 개선
    ↓
기존 항목 + 신규 항목으로 재평가
    ↓
좋아졌는지 확인 후 적용</code></pre>
<p>관찰만 하면 고친 버전이 더 좋아졌는지 판단하기 어렵습니다. 평가만 하면 실제 사용자의 새로운 실패를 놓칠 수 있습니다.</p>
<h3 id="4-7-실습-가이드-①-observability--eval">4-7. 실습 가이드 ① Observability &amp; Eval</h3>
<ol>
<li>요청 전체와 LLM·도구 단계가 연결된 Trace를 남깁니다.</li>
<li>작은 평가 데이터셋과 명확한 통과 기준을 만듭니다.</li>
<li>그래프를 직접 호출해 Agent 자체의 품질을 평가합니다.</li>
<li>실패 항목에서 어떤 도구·단계가 원인인지 확인합니다.</li>
<li>프롬프트나 흐름을 바꾼 뒤 같은 데이터셋으로 비교합니다.</li>
<li>API·브라우저 연결은 별도의 통합 시험으로 확인합니다.</li>
</ol>
<p><strong>성공 확인:</strong> 그래프 직접 호출 평가가 통과해도 로그인, HTTP, SSE, 화면까지 정상이라는 뜻은 아닙니다. 평가 대상의 범위를 분명히 기록합니다.</p>
<h3 id="4-8-error-handling-오류를-한-종류로-취급하지-않기">4-8. Error Handling: 오류를 한 종류로 취급하지 않기</h3>
<p>오류는 서로 다른 세 축으로 분류해 볼 수 있습니다.</p>
<table>
<thead>
<tr>
<th>분류 축</th>
<th>종류</th>
<th>예시와 대응</th>
</tr>
</thead>
<tbody><tr>
<td>다시 시도하면 나아질까?</td>
<td>일시적 / 지속적</td>
<td>잠깐의 네트워크 오류는 재시도 후보, 잘못된 입력은 수정 필요</td>
</tr>
<tr>
<td>대안으로 계속할 수 있을까?</td>
<td>복구 가능 / 복구 불가</td>
<td>상태 조회 실패 후 안내문만 제공 가능, 필수 기능 전체 중단이면 종료</td>
</tr>
<tr>
<td>원인이 어디에 있을까?</td>
<td>사용자 입력 / 시스템</td>
<td>검색어 누락은 재입력, DB 장애는 서버 점검</td>
</tr>
</tbody></table>
<p>이 축들은 서로 독립적입니다. 하나의 오류가 ‘일시적이고 시스템에서 발생했으며 대안으로 부분 복구 가능’할 수 있습니다.</p>
<p>HTTP 상태 코드만 보고 무조건 결정하지 않습니다. 같은 시간 초과라도 읽기 조회인지 실제 예약 생성인지에 따라 재시도 위험이 다릅니다.</p>
<h3 id="4-9-계층별-오류-책임-정하기">4-9. 계층별 오류 책임 정하기</h3>
<table>
<thead>
<tr>
<th>계층</th>
<th>맡을 책임</th>
</tr>
</thead>
<tbody><tr>
<td>데이터·외부 API 연동</td>
<td>원인을 보존하고 안전한 범위의 재시도 수행</td>
</tr>
<tr>
<td>MCP 연결·도구 계층</td>
<td>연결 끊김, 잘못된 입력, 도구 실패를 구분해 전달</td>
</tr>
<tr>
<td>Agent·오케스트레이션</td>
<td>대체 경로, 작업 중단, 부분 결과의 가치 판단</td>
</tr>
<tr>
<td>Backend</td>
<td>HTTP·스트림 오류 규약과 요청의 최종 상태 관리</td>
</tr>
<tr>
<td>Frontend</td>
<td>사용자에게 실패와 다음 행동을 명확하게 표시</td>
</tr>
</tbody></table>
<p>사용자에게는 ‘대출 상태를 확인하지 못했어요. 잠시 후 다시 확인해 주세요’라고 설명할 수 있습니다. 내부 진단에는 원인 코드와 실패 지점을 남깁니다. 내부 예외·접속 정보·원문 로그를 그대로 화면에 보여 주지는 않습니다.</p>
<h3 id="4-10-retry-다시-시도할-때도-규칙이-필요하다">4-10. Retry: 다시 시도할 때도 규칙이 필요하다</h3>
<p>재시도는 엘리베이터 버튼을 계속 누르는 행동과 달라야 합니다. 너무 자주 요청하면 상대 시스템의 회복을 방해할 수 있습니다.</p>
<p>정할 항목은 다음과 같습니다.</p>
<ul>
<li>재시도 대상 오류</li>
<li>최초 호출을 포함한 총 시도 횟수</li>
<li>전체 시간 제한과 각 시도의 제한 시간</li>
<li>점차 길어지는 대기 시간과 무작위 지연인 jitter</li>
<li>서버가 알려 준 재요청 시점의 반영 여부</li>
<li>실패 후 상위 계층에 전달할 오류 형식</li>
</ul>
<p>가령 ‘최초 1회 + 재시도 2회 = 최대 3회 시도’처럼 표현하면 횟수 혼동을 줄일 수 있습니다.</p>
<p>여러 계층이 각각 최대 3회씩 시도하면 외부 요청이 최대 9회로 늘 수 있습니다. 재시도 책임과 전체 예산을 한곳에서 조율해야 합니다.</p>
<h4 id="쓰기-작업은-중복에-주의하기">쓰기 작업은 중복에 주의하기</h4>
<p>예약 요청을 보냈는데 응답만 사라졌을 수 있습니다. 이때 무조건 다시 보내면 예약이 두 건 생길 수 있습니다.</p>
<p><strong>멱등성</strong>은 같은 작업을 반복 요청해도 업무 결과가 중복되지 않도록 하는 성질입니다. 예를 들어 같은 예약 요청에 같은 작업 키를 붙이고 서버가 이미 처리한 키인지 확인하는 방식이 있습니다. 키를 붙이기만 하면 끝나는 것이 아니라 실제 저장·중복 검사 처리가 함께 있어야 합니다.</p>
<h3 id="4-11-graceful-degradation-일부가-고장-나도-가능한-도움-제공하기">4-11. Graceful Degradation: 일부가 고장 나도 가능한 도움 제공하기</h3>
<p>엘리베이터가 고장 나도 계단이 열려 있다면 건물의 일부 기능을 계속 사용할 수 있습니다.</p>
<p>AI에서도 일부 기능 실패가 항상 전체 무응답을 뜻하지는 않습니다.</p>
<pre><code class="language-text">책 검색: 성공
대출 상태 조회: 실패

안내:
관련 책 두 권을 찾았어요.
현재 대출 가능한지는 조회하지 못했어요.
예약 완료로 처리하지 않았어요.</code></pre>
<p>핵심은 성공한 것과 실패한 것을 정확히 구분하는 것입니다. 누락된 부분을 모델이 상상으로 채우는 것은 대체 동작이 아닙니다.</p>
<h3 id="4-12-실습-가이드-②-error-handling">4-12. 실습 가이드 ② Error Handling</h3>
<p><strong>목표:</strong> 의도적으로 오류를 만들고, 실패한 위치와 사용자 안내가 일치하는지 확인합니다.</p>
<ol>
<li>잘못된 질문 양식을 보냅니다.</li>
<li>MCP 서버와의 연결을 끊어 봅니다.</li>
<li>도구에 시간 초과와 일시적 실패를 발생시킵니다.</li>
<li>재시도 횟수와 전체 시간이 한도를 넘지 않는지 봅니다.</li>
<li>한 작업만 실패했을 때 부분 응답을 확인합니다.</li>
<li>예약 응답이 끊긴 경우 실제 예약 건수와 중복 방지를 확인합니다.</li>
<li>스트림 중간 오류 후 로딩 표시가 끝나는지 봅니다.</li>
</ol>
<p><strong>성공 확인:</strong> 오류가 안 보이는 것이 성공은 아닙니다. 실패의 원인·처리·최종 상태가 Trace와 화면에 일관되게 나타나는지 확인합니다.</p>
<h3 id="4-13-cost--latency-돈·시간·품질을-함께-보기">4-13. Cost &amp; Latency: 돈·시간·품질을 함께 보기</h3>
<p>Cost는 비용, Latency는 응답 지연, Quality는 결과의 품질입니다.</p>
<p>가장 싼 방법이 필요한 정보를 빠뜨릴 수 있고, 가장 복잡한 방법이 답변을 늦출 수 있습니다. 실제로 측정해 균형을 정합니다.</p>
<p>비용에는 다음이 포함될 수 있습니다.</p>
<ul>
<li>모델 입력·출력 토큰 비용</li>
<li>임베딩 생성 비용</li>
<li>외부 도구·API 사용 비용</li>
<li>서버·DB·관찰 기록 저장 비용</li>
<li>평가와 재시도 비용</li>
</ul>
<p>Token은 모델이 문장을 처리하는 조각입니다. 한국어 글자 한 개가 항상 토큰 한 개가 되는 것은 아닙니다.</p>
<p>도구 호출을 결정하는 모델 실행과, 도구 결과를 읽고 답변하는 모델 실행이 이어지면 호출이 늘 수 있습니다. 그러나 도구 하나가 반드시 모델 호출 두 번을 뜻하지는 않습니다. 여러 도구를 한 번에 요청하거나 모델 없이 실행하는 흐름도 있습니다.</p>
<p><strong>에이전트 수만으로 호출 수와 비용을 단정하지 말고 실제 사용량을 측정합니다.</strong></p>
<h3 id="4-14-대화가-길어질수록-왜-비싸질까">4-14. 대화가 길어질수록 왜 비싸질까?</h3>
<p>매번 앞의 대화와 자료를 전부 다시 모델에 보내면 입력이 길어집니다. 짧은 질문을 해도 과거 기록 때문에 비용이 커질 수 있습니다.</p>
<p>대응 예시는 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>방법</th>
<th>기대 효과</th>
<th>확인할 위험</th>
</tr>
</thead>
<tbody><tr>
<td>필요한 문서만 전달</td>
<td>입력 길이 감소</td>
<td>핵심 근거 누락</td>
</tr>
<tr>
<td>이전 대화 요약</td>
<td>반복 입력 감소</td>
<td>약속·조건·식별자 손실</td>
</tr>
<tr>
<td>검색 결과 중복 제거</td>
<td>같은 내용 반복 감소</td>
<td>서로 다른 근거를 잘못 합침</td>
</tr>
<tr>
<td>캐시 사용</td>
<td>반복 계산·조회 감소 가능</td>
<td>오래된 결과와 사용자 권한 혼합</td>
</tr>
<tr>
<td>불필요한 도구 호출 줄이기</td>
<td>왕복 시간·비용 감소 가능</td>
<td>필수 정보 조회 누락</td>
</tr>
</tbody></table>
<p>예약 가능 여부처럼 빠르게 바뀌는 정보는 오래된 캐시를 그대로 쓰기 어렵습니다. 캐시가 항상 품질을 높이거나 요약이 항상 품질을 낮추는 것은 아닙니다. 데이터의 신선도와 평가 결과로 판단합니다.</p>
<h3 id="4-15-순차-실행과-병렬-실행">4-15. 순차 실행과 병렬 실행</h3>
<p>독립된 조회 A가 2초, B가 3초 걸리는 가상 상황을 생각해 봅시다.</p>
<pre><code class="language-text">순차: A 2초 → B 3초 = 약 5초
병렬: A와 B를 함께 시작 = 약 3초 + 합치는 시간</code></pre>
<p>병렬 실행은 기다리는 시간을 줄일 수 있지만 실행한 일이 같다면 총 작업량과 API 비용은 크게 줄지 않을 수 있습니다. 동시 호출 제한이나 실패 재시도로 비용이 늘 수도 있습니다.</p>
<p>초보자 프로젝트에서는 다음 네 조건을 만족하는 작업부터 병렬로 시도하면 이해하기 쉽습니다.</p>
<ol>
<li>서로의 실행 결과를 입력으로 필요로 하지 않습니다.</li>
<li>공유 상태를 충돌 없이 다룰 수 있습니다. 처음에는 각자 결과를 반환한 뒤 합칩니다.</li>
<li>실행 순서가 업무 결과에 영향을 주지 않습니다.</li>
<li>일부 실패를 어떻게 처리할지 정해져 있습니다.</li>
</ol>
<p>‘도서관 이용 시간 조회’와 ‘추천 안내문 조회’는 독립적일 수 있습니다. ‘책 검색’과 ‘검색된 책 예약’은 앞 결과가 뒤 입력이므로 순서가 필요합니다.</p>
<p>공유 상태 갱신이 있는 모든 병렬 실행이 불가능한 것은 아닙니다. 다만 reducer나 동시성 제어 등의 추가 설계가 필요하므로 처음에는 충돌 없는 구조부터 시작합니다.</p>
<h3 id="4-16-실습-가이드-③-cost--latency">4-16. 실습 가이드 ③ Cost &amp; Latency</h3>
<ol>
<li>같은 질문 묶음으로 현재 버전을 측정합니다.</li>
<li>모델 호출 수, 입력·출력 토큰, 도구별 시간, 총시간을 기록합니다.</li>
<li>독립적인 조회만 병렬로 바꿉니다.</li>
<li>같은 데이터셋으로 결과 품질과 실패 처리도 다시 봅니다.</li>
<li>평균뿐 아니라 느린 요청 구간을 나타내는 p95도 확인합니다.</li>
<li>그래프 직접 호출 측정과 사용자가 화면에서 느끼는 시간을 구분합니다.</li>
</ol>
<p>p95는 관측값의 약 95%가 그 값 이하인 지점입니다. 일부 요청만 유난히 느린 상황을 볼 때 유용합니다.</p>
<p>측정표에는 환경, 시험 횟수, 사용 모델·버전, 캐시 상태를 함께 남깁니다. 그래야 변경 효과를 비교하기 쉽습니다. 이 글의 예시 시간은 실제 성능 결과가 아닙니다.</p>
<h3 id="4-17-stateful과-stateless-기억이-사라진다는-뜻일까">4-17. Stateful과 Stateless: 기억이 사라진다는 뜻일까?</h3>
<p><strong>Stateful:</strong> 특정 인스턴스가 자기 메모리에 대화 상태를 가지고 있습니다.</p>
<p><strong>Stateless Backend:</strong> 다음 요청을 처리할 때 필요한 대화 상태를 외부 저장소 등에서 읽을 수 있어, 특정 인스턴스의 메모리에 의존하지 않습니다.</p>
<pre><code class="language-text">Stateful 비유:
직원 A만 손님의 상담 내용을 기억함
→ 다음에 직원 B를 만나면 모름

Stateless Backend 비유:
모든 직원이 권한에 따라 공동 상담 기록을 읽음
→ 어느 직원이 맡아도 이어서 상담 가능</code></pre>
<p>Stateless는 서비스 전체에 상태가 없다는 뜻이 아닙니다. 상태를 Backend 인스턴스와 분리한 것입니다. 진행 중인 연결이나 작업이 순간적인 메모리를 사용하는 것까지 금지한다는 뜻도 아닙니다.</p>
<h3 id="4-18-메모리-checkpointer의-한계와-외부-저장소">4-18. 메모리 Checkpointer의 한계와 외부 저장소</h3>
<p>메모리 저장 방식은 작은 실습에서 단순하고 유용합니다. 하지만 프로세스를 끄면 기록이 사라지고 다른 인스턴스가 같은 기록을 자동으로 읽지 못합니다.</p>
<p>PostgreSQL 같은 외부 저장소에 체크포인트를 저장하면 여러 Backend가 같은 대화 상태를 읽을 수 있습니다. 비동기 PostgreSQL 체크포인터는 비동기 실행 흐름에서 DB 입출력을 처리하는 구현체입니다.</p>
<p>다만 외부 DB를 붙이는 것으로 모든 문제가 해결되지는 않습니다.</p>
<ul>
<li>같은 대화에 동시에 들어오는 요청의 순서를 정해야 합니다.</li>
<li>대화 소유권을 검사해야 합니다.</li>
<li>저장 데이터의 스키마와 직렬화 호환성을 관리해야 합니다.</li>
<li>DB 장애와 연결 수 한도를 처리해야 합니다.</li>
<li>부분 진행을 재개할 때 외부 쓰기 작업이 중복되지 않아야 합니다.</li>
</ul>
<p>체크포인트를 과거로 돌려도 외부에 이미 생성한 예약까지 자동으로 되돌아가는 것은 아닙니다.</p>
<h3 id="4-19-lifespan-개점-준비와-폐점-정리">4-19. Lifespan: 개점 준비와 폐점 정리</h3>
<p>가게는 주문이 들어올 때마다 수도와 전기를 새로 연결하지 않습니다. 문을 열 때 준비하고 닫을 때 정리합니다.</p>
<p>FastAPI의 Lifespan은 애플리케이션 시작과 종료에 필요한 자원을 준비·정리하는 구간입니다. DB 연결이나 Agent가 사용할 자원을 이곳에서 준비할 수 있습니다. 관련 방식은 <a href="https://fastapi.tiangolo.com/advanced/events/">FastAPI Lifespan 공식 문서</a>에서 확인합니다.</p>
<pre><code class="language-text">애플리케이션 시작
    ↓
DB 연결 자원과 체크포인터 준비
    ↓
체크포인터를 전달해 Agent 생성
    ↓
여러 요청 처리
    ↓
애플리케이션 종료 시 연결 정리</code></pre>
<p>다중 worker나 여러 인스턴스에서는 <strong>프로세스마다</strong> 시작·종료가 일어납니다. 서비스 전체에서 하나만 생성되는 전역 Singleton이라고 생각하지 않습니다. 체크포인터마다 연결 관리 방식도 다를 수 있으므로 구현체 사용법을 확인합니다.</p>
<h3 id="4-20-의존성-주입-사용할-기록장을-밖에서-건네기">4-20. 의존성 주입: 사용할 기록장을 밖에서 건네기</h3>
<p>Agent를 만드는 함수 안에서 저장소까지 정하면, 저장소를 바꾸기 어렵습니다.</p>
<p>기록장을 내부에서 무조건 만드는 대신 밖에서 건네주면 됩니다.</p>
<pre><code class="language-python"># 구조를 설명하는 예시입니다.
# builder는 별도로 구성한 LangGraph 그래프 빌더라고 가정합니다.
def build_agent(builder, checkpointer):
    return builder.compile(checkpointer=checkpointer)</code></pre>
<p>이것을 의존성 주입이라고 합니다. 어려운 이름이지만 ‘필요한 도구를 밖에서 전달한다’는 뜻입니다.</p>
<ul>
<li>연습할 때는 메모리 저장소를 전달할 수 있습니다.</li>
<li>운영할 때는 외부 DB 저장소를 전달할 수 있습니다.</li>
<li>시험에서는 격리된 테스트 저장소를 전달할 수 있습니다.</li>
</ul>
<p>Agent의 업무 로직과 자원을 생성·종료하는 책임을 구분하면 변경과 시험이 쉬워집니다.</p>
<h3 id="4-21-실습-가이드-④-stateless-session-전환">4-21. 실습 가이드 ④ Stateless Session 전환</h3>
<p><strong>목표:</strong> Backend 프로세스가 바뀌어도 같은 대화를 이어갑니다.</p>
<ol>
<li>외부 체크포인트 저장소를 준비합니다.</li>
<li>초기 스키마 생성·설정은 사용하는 구현체의 지침에 맞춥니다.</li>
<li>Lifespan에서 저장소 연결과 체크포인터를 준비합니다.</li>
<li>Agent 생성 함수가 체크포인터를 외부에서 받도록 합니다.</li>
<li>같은 저장소를 사용하는 Backend A와 B를 준비합니다.</li>
<li>A에 첫 메시지를 보내고 B에 같은 대화 ID로 후속 질문을 보냅니다.</li>
<li>Backend를 재시작한 뒤에도 대화를 읽는지 확인합니다.</li>
<li>다른 ID와 다른 사용자의 기록이 섞이거나 노출되지 않는지 확인합니다.</li>
</ol>
<p>단순히 <code>thread_id</code>가 응답에 다시 나오는 것만으로 저장 성공이라고 볼 수 없습니다. 이전에만 존재했던 정보가 새 프로세스에서 복원되는지 실제로 확인해야 합니다.</p>
<p>대화 이어가기와 실행 중단 지점에서 재개하기도 구분합니다. 중단 재개 기능이 필요하다면 새 그래프·새 연결에서 미완료 작업이 이어지는지, 이미 끝난 도구가 다시 실행되지 않는지 별도로 시험합니다.</p>
<p><strong>파트 복습:</strong> 관찰로 원인을 찾고, 평가로 개선을 검증하며, 오류·비용·상태 저장을 실제 운영 조건에 맞게 관리합니다.</p>
<hr />
<h2 id="part-5-보충-agent-확장과-프로젝트-완성-순서">Part 5. 보충: Agent 확장과 프로젝트 완성 순서</h2>
<blockquote>
<p><strong>범위 안내:</strong> 제공된 PDF에는 목차의 ‘Agent 확장’에 대응하는 독립 본문이 없습니다. 다음은 본문에 있는 멀티에이전트 스트리밍·병렬 실행·운영 개념을 연결한 학습 보충입니다. 별도 5장 원문을 요약한 것으로 읽지 않도록 구분합니다.</p>
</blockquote>
<h3 id="5-1-언제-agent를-늘릴까">5-1. 언제 Agent를 늘릴까?</h3>
<p>조별 과제에서 한 사람이 충분히 할 일을 다섯 명에게 나누면 오히려 전달 시간이 길어질 수 있습니다. 에이전트도 비슷합니다.</p>
<p>역할을 나누기 전에 질문합니다.</p>
<ul>
<li>지금 실패하는 이유가 무엇인가?</li>
<li>도구 설명이나 검색 품질을 고치면 해결되는가?</li>
<li>별도 권한·자료·평가 기준이 필요한 역할인가?</li>
<li>역할을 나눴을 때 실제 품질이 좋아지는가?</li>
<li>호출·시간·비용 증가를 감당할 수 있는가?</li>
</ul>
<p>역할 분리가 유리한 경우는 조사와 검토의 기준이 다르거나, 독립적인 데이터 조회를 분리할 수 있는 경우 등입니다. 에이전트 수가 많다는 것 자체가 좋은 설계의 증거는 아닙니다.</p>
<h3 id="5-2-역할을-추가할-때-함께-정할-계약">5-2. 역할을 추가할 때 함께 정할 계약</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>맡을 일</td>
<td>관련 책 후보를 찾는다</td>
</tr>
<tr>
<td>입력</td>
<td>질문, 검색 범위, 최대 후보 수</td>
</tr>
<tr>
<td>출력</td>
<td>책 식별자, 근거, 조회 시각</td>
</tr>
<tr>
<td>사용 도구</td>
<td>책 검색·상세 조회만 허용</td>
</tr>
<tr>
<td>종료 조건</td>
<td>후보 확보 또는 검색 한도 도달</td>
</tr>
<tr>
<td>실패 계약</td>
<td>실패 원인과 확보한 부분 결과 반환</td>
</tr>
<tr>
<td>실행 한도</td>
<td>시간·도구 호출·반복 횟수 제한</td>
</tr>
</tbody></table>
<p>최종 결과를 합치는 담당자는 근거 충돌과 누락을 확인합니다. ‘검색 담당은 재고 있음, 상태 조회 담당은 재고 없음’처럼 충돌하면 출처와 확인 시각을 비교하고, 필요한 경우 다시 확인하거나 불확실함을 표시합니다.</p>
<h3 id="5-3-작은-성공을-차례로-쌓는-프로젝트-순서">5-3. 작은 성공을 차례로 쌓는 프로젝트 순서</h3>
<table>
<thead>
<tr>
<th>단계</th>
<th>만드는 것</th>
<th>통과 기준</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>문제와 사용자 요구</td>
<td>어떤 사용자의 어떤 행동을 돕는지 설명 가능</td>
</tr>
<tr>
<td>2</td>
<td>MCP 기능</td>
<td>Inspector에서 직접 정상·실패 호출 확인</td>
</tr>
<tr>
<td>3</td>
<td>단일 Agent</td>
<td>필요한 도구를 선택하고 결과에 근거해 답변</td>
</tr>
<tr>
<td>4</td>
<td>Backend 통합</td>
<td>입력 검사, 대화 분리, 실제 API 경로 확인</td>
</tr>
<tr>
<td>5</td>
<td>Streaming UI</td>
<td>진행·오류·완료가 화면에 정확히 표시</td>
</tr>
<tr>
<td>6</td>
<td>멀티에이전트</td>
<td>역할·작업 ID·종료 조건·부분 실패 처리</td>
</tr>
<tr>
<td>7</td>
<td>관찰과 평가</td>
<td>변경 전후의 품질과 실행 경로 비교</td>
</tr>
<tr>
<td>8</td>
<td>오류 처리</td>
<td>제한된 재시도, 중복 방지, 부분 응답 확인</td>
</tr>
<tr>
<td>9</td>
<td>성능 개선</td>
<td>같은 시험 조건에서 시간·비용·품질 비교</td>
</tr>
<tr>
<td>10</td>
<td>상태 외부화</td>
<td>재시작과 다른 인스턴스에서 대화 복원</td>
</tr>
</tbody></table>
<p>처음에는 ‘가상의 책 3개를 검색하고 설명하기’처럼 작은 범위로 시작하면 좋습니다. 한 단계의 성공 기준을 확인한 뒤 다음 연결을 추가하면, 고장이 났을 때 원인을 찾기 쉽습니다.</p>
<h3 id="5-4-학습용-기획서-양식">5-4. 학습용 기획서 양식</h3>
<pre><code class="language-text">프로젝트 이름:
도움을 받을 사용자:
사용자가 겪는 문제:
AI가 도울 업무:
사람의 확인이 필요한 업무:
사용할 공개·가상 데이터:

Resource 목록:
Tool 목록과 입력·출력:
Prompt 목록:

검색 담당:
MCP 실행 위치:
대화 ID 발급·소유권 관리 방식:
실제 대화 상태 저장 위치:

화면에 보낼 이벤트:
정상·부분 완료·실패의 기준:
재시도와 중복 방지 정책:

평가할 대표 질문:
실패 상황 시험:
측정할 시간·비용·품질:
최종 시연 순서:</code></pre>
<p>이 양식은 본문 개념을 정리하기 위해 새로 만든 것입니다. 처음부터 모든 기술을 정하려 하기보다 사용자 문제와 성공 기준을 먼저 적어 봅시다.</p>
<h3 id="5-5-프로젝트-완료를-확인하는-질문">5-5. 프로젝트 완료를 확인하는 질문</h3>
<ul>
<li><input disabled="" type="checkbox" /> 사용자의 문제와 서비스의 도움을 쉬운 말로 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Resource·Tool·Prompt를 나눈 이유를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Tool 이름·설명·입력·출력이 명확하다.</li>
<li><input disabled="" type="checkbox" /> Inspector에서 기능을 직접 실행해 보았다.</li>
<li><input disabled="" type="checkbox" /> Backend가 사용자와 대화 ID의 권한을 확인한다.</li>
<li><input disabled="" type="checkbox" /> 같은 대화는 이어지고 다른 대화는 구분된다.</li>
<li><input disabled="" type="checkbox" /> 도구 실패와 검색 결과 없음이 구분된다.</li>
<li><input disabled="" type="checkbox" /> 스트림의 진행·부분 실패·전체 종료가 구분된다.</li>
<li><input disabled="" type="checkbox" /> 긴 작업과 연결 끊김을 시험했다.</li>
<li><input disabled="" type="checkbox" /> 재시도의 횟수·시간 한도와 쓰기 작업 중복 방지가 있다.</li>
<li><input disabled="" type="checkbox" /> Trace로 병목을 찾고 같은 데이터셋으로 개선을 비교했다.</li>
<li><input disabled="" type="checkbox" /> 상태 저장소를 바꿨다면 새 프로세스에서도 복원을 시험했다.</li>
<li><input disabled="" type="checkbox" /> 공개 시연 자료에는 가상·공개 데이터만 사용했다.</li>
</ul>
<p>이 목록은 개발 후 사용할 확인 항목입니다. 이 글이 실제 서비스를 구축했거나 시험을 통과했다는 뜻은 아닙니다.</p>
<hr />
<h2 id="헷갈리기-쉬운-질문으로-복습하기">헷갈리기 쉬운 질문으로 복습하기</h2>
<h3 id="q1-ai가-말을-잘하면-agent-프로젝트가-완성된-걸까">Q1. AI가 말을 잘하면 Agent 프로젝트가 완성된 걸까?</h3>
<p>아닙니다. 실제 자료 조회, 도구 실행, 권한 확인, 오류 처리, 상태 저장, 화면 연결을 함께 확인해야 합니다.</p>
<h3 id="q2-조회는-전부-resource일까">Q2. 조회는 전부 Resource일까?</h3>
<p>아닙니다. 조건을 입력받아 검색하는 기능처럼 읽기 전용 Tool도 있습니다. 데이터 제공 방식과 실행·선택 주체를 함께 봅니다.</p>
<h3 id="q3-mcp-server를-실행하면-llm도-자동으로-생길까">Q3. MCP Server를 실행하면 LLM도 자동으로 생길까?</h3>
<p>아닙니다. MCP 서버는 기능 제공자입니다. 이를 사용하는 모델과 Host 애플리케이션은 별도로 연결합니다.</p>
<h3 id="q4-thread_id를-저장하면-대화-내용도-저장될까">Q4. thread_id를 저장하면 대화 내용도 저장될까?</h3>
<p>아닙니다. ID는 이름표입니다. 실제 State를 저장하는 장치가 필요합니다.</p>
<h3 id="q5-frontend가-id를-만들면-서버가-자동으로-stateless일까">Q5. Frontend가 ID를 만들면 서버가 자동으로 Stateless일까?</h3>
<p>아닙니다. 실제 상태가 어디에 있고 특정 인스턴스에 의존하는지를 봐야 합니다.</p>
<h3 id="q6-스트리밍이면-답변이-더-빨리-완성될까">Q6. 스트리밍이면 답변이 더 빨리 완성될까?</h3>
<p>항상 그렇지는 않습니다. 첫 정보가 일찍 보일 수는 있지만 전체 계산 시간은 그대로일 수 있습니다.</p>
<h3 id="q7-연결이-끊어지면-예약도-취소될까">Q7. 연결이 끊어지면 예약도 취소될까?</h3>
<p>자동으로 그렇지는 않습니다. 네트워크 연결, 서버 작업, 외부 예약의 상태는 따로 확인해야 합니다.</p>
<h3 id="q8-캐시·병렬-실행·멀티에이전트는-무조건-좋을까">Q8. 캐시·병렬 실행·멀티에이전트는 무조건 좋을까?</h3>
<p>아닙니다. 신선도, 작업 의존성, 호출 제한, 평가 결과를 보고 적용합니다.</p>
<h3 id="q9-postgresql을-연결하면-모든-운영-문제가-해결될까">Q9. PostgreSQL을 연결하면 모든 운영 문제가 해결될까?</h3>
<p>아닙니다. 상태 공유를 돕지만 인증, 동시 실행, 중복 방지, 장애 처리도 필요합니다.</p>
<h3 id="q10-프로젝트에서-가장-먼저-할-일은-무엇일까">Q10. 프로젝트에서 가장 먼저 할 일은 무엇일까?</h3>
<p>‘누구의 어떤 문제를 해결할까?’와 ‘무엇을 확인하면 성공이라고 할까?’를 정하는 일입니다. 그다음 작은 기능부터 연결합니다.</p>
<hr />
<h2 id="공개-문서로-더-확인하기">공개 문서로 더 확인하기</h2>
<p>이 글은 제공된 교재의 구성과 학습 주제를 바탕으로 독자적인 비유와 가상 사례로 다시 설명했습니다. API와 SDK의 실제 사용법은 설치 버전에 맞는 공식 문서를 함께 확인하세요.</p>
<ul>
<li><a href="https://modelcontextprotocol.io/specification/2025-06-18/server/resources">MCP Resource 명세</a>: 자원을 통한 데이터·문맥 제공</li>
<li><a href="https://modelcontextprotocol.io/specification/2025-06-18/server/tools">MCP Tool 명세</a>: 모델이 사용할 실행 기능</li>
<li><a href="https://docs.langchain.com/oss/python/langgraph/persistence">LangGraph Persistence</a>: Thread 상태 저장과 영속성</li>
<li><a href="https://fastapi.tiangolo.com/advanced/events/">FastAPI Lifespan</a>: 시작·종료 자원 관리</li>
<li><a href="https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol">AI SDK Stream Protocols</a>: Backend와 채팅 UI 사이의 스트림 규약</li>
</ul>