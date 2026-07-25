<h2 id="0-이-노트를-사용하는-방법">0. 이 노트를 사용하는 방법</h2>
<ol>
<li>각 장의 <strong>한 문장 이해</strong>를 먼저 읽는다.</li>
<li>표의 키워드와 문법을 확인한다.</li>
<li>예제는 직접 입력하고 브라우저에서 실행한다.</li>
<li>장 끝의 <strong>핵심 정리</strong>만 보고 내용을 말로 설명해 본다.</li>
<li>마지막의 체크리스트와 퀴즈로 이해도를 확인한다.</li>
</ol>
<h3 id="전체-흐름">전체 흐름</h3>
<pre><code class="language-text">브라우저가 서버에 요청
        ↓
서버가 HTML·CSS 등의 파일을 응답
        ↓
HTML이 콘텐츠의 구조를 만듦
        ↓
CSS가 구조의 모양과 배치를 꾸밈
        ↓
반응형 CSS가 화면 크기에 맞춰 배치를 바꿈</code></pre>
<table>
<thead>
<tr>
<th>기술</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>HTML</td>
<td>제목, 문단, 이미지, 폼 등 콘텐츠의 구조와 의미</td>
<td>건물의 뼈대와 방</td>
</tr>
<tr>
<td>CSS</td>
<td>색상, 크기, 간격, 정렬, 레이아웃, 애니메이션</td>
<td>인테리어와 가구 배치</td>
</tr>
<tr>
<td>JavaScript</td>
<td>클릭, 데이터 처리 등 동작</td>
<td>전기·자동화 장치</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-web-개요">1. Web 개요</h1>
<h2 id="11-인터넷과-웹은-다르다">1.1 인터넷과 웹은 다르다</h2>
<p><strong>한 문장 이해:</strong> 인터넷은 전 세계 컴퓨터를 연결하는 기반 시설이고, 웹은 그 위에서 HTML 문서를 주고받는 서비스이다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>인터넷(Internet)</th>
<th>웹(WWW)</th>
</tr>
</thead>
<tbody><tr>
<td>뜻</td>
<td>전 세계 컴퓨터 네트워크를 연결한 통신망</td>
<td>인터넷 위에서 정보를 공유하는 서비스</td>
</tr>
<tr>
<td>핵심 요소</td>
<td>케이블, 공유기, 라우터, 스위치, 컴퓨터</td>
<td>URL, HTTP, HTML, 브라우저</td>
</tr>
<tr>
<td>다른 예</td>
<td>이메일, 파일 전송, 원격 접속도 인터넷 사용</td>
<td>웹은 인터넷 서비스 중 하나</td>
</tr>
</tbody></table>
<p>웹을 기억하는 세 가지 질문:</p>
<ul>
<li><strong>어디로?</strong> URL</li>
<li><strong>어떻게?</strong> HTTP/HTTPS</li>
<li><strong>무엇을?</strong> HTML 등의 웹 자원</li>
</ul>
<h2 id="12-클라이언트와-서버">1.2 클라이언트와 서버</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>역할</th>
<th>예</th>
</tr>
</thead>
<tbody><tr>
<td>Client</td>
<td>서비스를 요청하고 결과를 소비</td>
<td>Chrome, Safari, 모바일 앱</td>
</tr>
<tr>
<td>Server</td>
<td>요청을 처리하고 자원을 응답</td>
<td>웹 서버, API 서버, DB 서버</td>
</tr>
</tbody></table>
<pre><code class="language-text">Client: “index.html을 주세요.”  ── HTTP Request ──▶  Server
Client: 화면에 결과를 표시       ◀─ HTTP Response ─  Server</code></pre>
<p>대표적인 웹 서버에는 Apache, Nginx, Microsoft IIS가 있다.</p>
<h2 id="13-tcpip-4계층">1.3 TCP/IP 4계층</h2>
<p><strong>프로토콜(protocol)</strong>은 통신할 때 지켜야 하는 규칙이다. 인터넷은 기능을 계층별로 나누어 처리한다.</p>
<table>
<thead>
<tr>
<th>계층</th>
<th>주요 역할</th>
<th>대표 프로토콜·기술</th>
<th>데이터 단위</th>
</tr>
</thead>
<tbody><tr>
<td>Application</td>
<td>사용자가 쓰는 서비스와 직접 상호작용</td>
<td>HTTP/HTTPS, DNS, FTP, SSH, SMTP</td>
<td>Data/Message</td>
</tr>
<tr>
<td>Transport</td>
<td>두 프로그램 사이의 전송 방식 결정</td>
<td>TCP, UDP</td>
<td>Segment/Datagram</td>
</tr>
<tr>
<td>Internet</td>
<td>IP 주소로 다른 네트워크까지 경로 탐색</td>
<td>IP, ICMP</td>
<td>Packet</td>
</tr>
<tr>
<td>Network Access</td>
<td>케이블·Wi-Fi를 통해 실제 신호 전송</td>
<td>Ethernet, Wi-Fi</td>
<td>Frame → Bit</td>
</tr>
</tbody></table>
<p>초보자가 기억할 흐름:</p>
<pre><code class="language-text">HTTP 메시지
  → TCP가 잘게 나누고 전송을 관리
  → IP가 목적지까지 길을 찾음
  → Ethernet/Wi-Fi가 실제 신호로 보냄</code></pre>
<h2 id="14-ip-mac-공인-ip-사설-ip-nat">1.4 IP, MAC, 공인 IP, 사설 IP, NAT</h2>
<ul>
<li><strong>IP 주소</strong>: 네트워크에서 장치를 찾기 위한 논리적 주소</li>
<li><strong>MAC 주소</strong>: 네트워크 장치에 부여된 하드웨어 식별자</li>
<li><strong>IPv4</strong>: <code>192.168.0.10</code>처럼 4개의 숫자로 표현</li>
<li><strong>IPv6</strong>: IPv4 주소 부족을 해결하기 위해 더 큰 주소 공간 제공</li>
<li><strong>Public IP</strong>: 인터넷 전체에서 식별되는 공인 주소</li>
<li><strong>Private IP</strong>: 집·회사 같은 내부 네트워크에서 사용하는 주소</li>
<li><strong>NAT</strong>: 여러 사설 IP가 하나의 공인 IP를 공유하도록 주소를 변환하는 기술</li>
</ul>
<pre><code class="language-text">노트북 192.168.0.10 ┐
휴대폰 192.168.0.11 ├─ 공유기/NAT ─ 공인 IP ─ 인터넷
태블릿 192.168.0.12 ┘</code></pre>
<h3 id="네트워크-확인-명령어">네트워크 확인 명령어</h3>
<table>
<thead>
<tr>
<th>목적</th>
<th>Windows</th>
<th>macOS/Linux</th>
</tr>
</thead>
<tbody><tr>
<td>네트워크·IP 정보</td>
<td><code>ipconfig /all</code></td>
<td><code>ifconfig</code> 또는 <code>ip addr</code></td>
</tr>
<tr>
<td>DNS에 도메인 질의</td>
<td><code>nslookup example.com</code></td>
<td><code>nslookup example.com</code></td>
</tr>
<tr>
<td>연결·포트 확인</td>
<td><code>netstat -an</code></td>
<td><code>lsof -iTCP -P -n</code></td>
</tr>
</tbody></table>
<h2 id="15-tcp와-udp">1.5 TCP와 UDP</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>TCP</th>
<th>UDP</th>
</tr>
</thead>
<tbody><tr>
<td>연결</td>
<td>연결 후 전송</td>
<td>연결 과정 없이 전송</td>
</tr>
<tr>
<td>신뢰성</td>
<td>순서·도착을 확인하고 필요 시 재전송</td>
<td>도착·순서를 보장하지 않음</td>
</tr>
<tr>
<td>특징</td>
<td>안정적이지만 상대적으로 무거움</td>
<td>빠르고 가벼움</td>
</tr>
<tr>
<td>예</td>
<td>웹, 파일 전송, 이메일</td>
<td>실시간 게임, 스트리밍, 음성 통화</td>
</tr>
</tbody></table>
<p>TCP 연결의 대표 과정은 <strong>3-way handshake</strong>이다.</p>
<pre><code class="language-text">Client ── SYN ─────────▶ Server
Client ◀─ SYN + ACK ─── Server
Client ── ACK ─────────▶ Server</code></pre>
<h2 id="16-브라우저가-웹-페이지를-받는-과정">1.6 브라우저가 웹 페이지를 받는 과정</h2>
<ol>
<li>사용자가 브라우저에 URL을 입력한다.</li>
<li>DNS에 도메인의 IP 주소를 묻는다.</li>
<li>서버와 통신 연결을 만든다. HTTPS라면 TLS 보안 연결도 설정한다.</li>
<li>브라우저가 HTTP Request를 보낸다.</li>
<li>서버가 요청을 처리하고 HTTP Response를 보낸다.</li>
<li>브라우저가 HTML을 해석하고 필요한 CSS, 이미지 등을 추가로 요청한다.</li>
<li>브라우저가 결과를 화면에 렌더링한다.</li>
</ol>
<h2 id="17-url-구조">1.7 URL 구조</h2>
<p>예시:</p>
<pre><code class="language-text">https://www.example.com:443/products/item.html?id=10#review
└─scheme └────host────┘ port └────path────┘ └query┘ fragment</code></pre>
<table>
<thead>
<tr>
<th>부분</th>
<th>뜻</th>
<th>예</th>
</tr>
</thead>
<tbody><tr>
<td>scheme</td>
<td>사용할 통신 방식</td>
<td><code>https</code></td>
</tr>
<tr>
<td>host</td>
<td>서버의 도메인 또는 IP</td>
<td><code>www.example.com</code></td>
</tr>
<tr>
<td>port</td>
<td>서버 프로그램의 통신 입구</td>
<td><code>443</code></td>
</tr>
<tr>
<td>path</td>
<td>서버 자원의 경로</td>
<td><code>/products/item.html</code></td>
</tr>
<tr>
<td>query string</td>
<td>서버에 전달할 추가 값</td>
<td><code>?id=10</code></td>
</tr>
<tr>
<td>fragment</td>
<td>같은 문서 안의 특정 위치</td>
<td><code>#review</code></td>
</tr>
</tbody></table>
<h2 id="18-http-request와-response">1.8 HTTP Request와 Response</h2>
<h3 id="request">Request</h3>
<pre><code class="language-http">GET /products?id=10 HTTP/1.1
Host: example.com
User-Agent: Chrome</code></pre>
<table>
<thead>
<tr>
<th>구성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Request Line</td>
<td>메서드, URL, HTTP 버전</td>
</tr>
<tr>
<td>Headers</td>
<td>브라우저·콘텐츠 형식 등 부가 정보</td>
</tr>
<tr>
<td>Body</td>
<td>서버에 보낼 데이터. GET에는 보통 없고 POST 등에 사용</td>
</tr>
</tbody></table>
<p>주요 HTTP 메서드:</p>
<ul>
<li><code>GET</code>: 자원 조회</li>
<li><code>POST</code>: 데이터 전송 및 자원 생성</li>
<li><code>PUT</code>: 자원 전체 교체</li>
<li><code>PATCH</code>: 자원 일부 수정</li>
<li><code>DELETE</code>: 자원 삭제</li>
</ul>
<h3 id="response">Response</h3>
<pre><code class="language-http">HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

&lt;!DOCTYPE html&gt;...</code></pre>
<table>
<thead>
<tr>
<th>구성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Status Line</td>
<td>HTTP 버전, 상태 코드, 상태 메시지</td>
</tr>
<tr>
<td>Headers</td>
<td>응답 형식, 서버 정보 등</td>
</tr>
<tr>
<td>Body</td>
<td>HTML, JSON, 이미지 등의 실제 내용</td>
</tr>
</tbody></table>
<p>상태 코드 묶음:</p>
<table>
<thead>
<tr>
<th>범위</th>
<th>의미</th>
<th>자주 보는 예</th>
</tr>
</thead>
<tbody><tr>
<td>1xx</td>
<td>처리 중인 정보</td>
<td><code>101 Switching Protocols</code></td>
</tr>
<tr>
<td>2xx</td>
<td>성공</td>
<td><code>200 OK</code>, <code>201 Created</code></td>
</tr>
<tr>
<td>3xx</td>
<td>다른 위치·캐시로 처리</td>
<td><code>301 Moved Permanently</code>, <code>304 Not Modified</code></td>
</tr>
<tr>
<td>4xx</td>
<td>클라이언트 요청 문제</td>
<td><code>400 Bad Request</code>, <code>404 Not Found</code></td>
</tr>
<tr>
<td>5xx</td>
<td>서버 처리 문제</td>
<td><code>500 Internal Server Error</code></td>
</tr>
</tbody></table>
<h3 id="https">HTTPS</h3>
<p>HTTPS는 HTTP 통신을 TLS로 암호화한다. 요청·응답의 헤더와 본문을 보호하지만 통신에 필요한 목적지 IP 등 모든 메타정보가 완전히 숨겨지는 것은 아니다.</p>
<blockquote>
<p>폼에서 <code>POST</code>를 쓴다고 자동으로 안전해지는 것은 아니다. 민감한 정보를 전송하려면 반드시 HTTPS가 필요하다.</p>
</blockquote>
<h2 id="19-개발-환경과-브라우저-도구">1.9 개발 환경과 브라우저 도구</h2>
<h3 id="vs-code--live-server">VS Code + Live Server</h3>
<ul>
<li>HTML 파일을 저장하면 브라우저가 자동 새로고침된다.</li>
<li>VS Code 오른쪽 아래의 <strong>Go Live</strong> 또는 명령 팔레트의 <code>Live Server: Open with Live Server</code>를 사용한다.</li>
</ul>
<h3 id="emmet">Emmet</h3>
<table>
<thead>
<tr>
<th>입력</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>!</code></td>
<td>HTML 기본 문서 생성</td>
</tr>
<tr>
<td><code>ul&gt;li</code></td>
<td>자식 요소 생성</td>
</tr>
<tr>
<td><code>h1+p</code></td>
<td>형제 요소 생성</td>
</tr>
<tr>
<td><code>li*3</code></td>
<td>요소 3번 반복</td>
</tr>
<tr>
<td><code>.card</code></td>
<td><code>&lt;div class=&quot;card&quot;&gt;&lt;/div&gt;</code></td>
</tr>
<tr>
<td><code>p#intro</code></td>
<td><code>&lt;p id=&quot;intro&quot;&gt;&lt;/p&gt;</code></td>
</tr>
</tbody></table>
<h3 id="chrome-개발자-도구-network-탭">Chrome 개발자 도구 Network 탭</h3>
<ol>
<li><code>F12</code> 또는 <code>Ctrl/Cmd + Shift + I</code>로 연다.</li>
<li>Network 탭에서 새로고침한다.</li>
<li>요청 하나를 선택한다.</li>
<li>Headers에서 URL, Method, Status를 확인한다.</li>
<li>Preview/Response에서 응답 내용을 확인한다.</li>
<li>Timing에서 요청에 걸린 시간을 확인한다.</li>
</ol>
<h3 id="1장-핵심-정리">1장 핵심 정리</h3>
<ul>
<li>인터넷은 기반 시설, 웹은 인터넷 위의 서비스이다.</li>
<li>웹은 URL로 위치를 지정하고 HTTP로 HTML 등의 자원을 주고받는다.</li>
<li>브라우저는 클라이언트, 웹 서버는 요청을 처리하는 서버이다.</li>
<li>HTTP는 Request/Response 구조이며 상태 코드로 결과를 알린다.</li>
<li>HTTPS는 TLS 암호화를 사용하는 현대 웹의 기본이다.</li>
</ul>
<hr />
<h1 id="2-html-기초">2. HTML 기초</h1>
<h2 id="21-html-문서의-기본-구조">2.1 HTML 문서의 기본 구조</h2>
<p><strong>한 문장 이해:</strong> HTML은 태그로 콘텐츠의 구조와 의미를 표현하는 마크업 언어이다.</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;ko&quot;&gt;
  &lt;head&gt;
    &lt;meta charset=&quot;UTF-8&quot;&gt;
    &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
    &lt;title&gt;나의 첫 페이지&lt;/title&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;h1&gt;안녕하세요&lt;/h1&gt;
    &lt;p&gt;첫 HTML 문서입니다.&lt;/p&gt;
  &lt;/body&gt;
&lt;/html&gt;</code></pre>
<table>
<thead>
<tr>
<th>코드</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;!DOCTYPE html&gt;</code></td>
<td>HTML5 문서임을 선언</td>
</tr>
<tr>
<td><code>&lt;html&gt;</code></td>
<td>전체 HTML 문서의 루트</td>
</tr>
<tr>
<td><code>&lt;head&gt;</code></td>
<td>제목·인코딩·CSS 연결 등 문서 정보</td>
</tr>
<tr>
<td><code>&lt;body&gt;</code></td>
<td>브라우저 화면에 표시할 내용</td>
</tr>
<tr>
<td><code>lang=&quot;ko&quot;</code></td>
<td>문서의 주 언어가 한국어임을 표시</td>
</tr>
<tr>
<td><code>charset=&quot;UTF-8&quot;</code></td>
<td>한글·이모지 등을 올바르게 표현</td>
</tr>
</tbody></table>
<h2 id="22-요소-태그-속성">2.2 요소, 태그, 속성</h2>
<pre><code class="language-html">&lt;a href=&quot;https://example.com&quot; target=&quot;_blank&quot;&gt;사이트 열기&lt;/a&gt;</code></pre>
<ul>
<li>시작 태그: <code>&lt;a ...&gt;</code></li>
<li>내용: <code>사이트 열기</code></li>
<li>종료 태그: <code>&lt;/a&gt;</code></li>
<li>요소(element): 시작 태그부터 종료 태그까지 전체</li>
<li>속성(attribute): <code>href</code>, <code>target</code></li>
<li>속성값: <code>&quot;https://example.com&quot;</code>, <code>&quot;_blank&quot;</code></li>
</ul>
<p>내용과 종료 태그가 없는 <strong>빈 요소</strong>도 있다.</p>
<pre><code class="language-html">&lt;img src=&quot;cat.jpg&quot; alt=&quot;창가에 앉은 고양이&quot;&gt;
&lt;br&gt;
&lt;hr&gt;
&lt;meta charset=&quot;UTF-8&quot;&gt;</code></pre>
<p>HTML은 대소문자를 구분하지 않지만, 일관성과 가독성을 위해 태그와 속성은 소문자로 작성한다.</p>
<h2 id="23-자주-쓰는-전역-속성">2.3 자주 쓰는 전역 속성</h2>
<table>
<thead>
<tr>
<th>속성</th>
<th>의미</th>
<th>예</th>
</tr>
</thead>
<tbody><tr>
<td><code>id</code></td>
<td>문서 안에서 하나만 쓰는 고유 식별자</td>
<td><code>id=&quot;header&quot;</code></td>
</tr>
<tr>
<td><code>class</code></td>
<td>여러 요소가 공유할 수 있는 분류명</td>
<td><code>class=&quot;card featured&quot;</code></td>
</tr>
<tr>
<td><code>style</code></td>
<td>요소에 직접 쓰는 CSS</td>
<td><code>style=&quot;color: red&quot;</code></td>
</tr>
<tr>
<td><code>title</code></td>
<td>추가 설명·툴팁</td>
<td><code>title=&quot;도움말&quot;</code></td>
</tr>
<tr>
<td><code>lang</code></td>
<td>콘텐츠의 언어</td>
<td><code>lang=&quot;ko&quot;</code></td>
</tr>
<tr>
<td><code>hidden</code></td>
<td>요소를 렌더링하지 않음</td>
<td><code>&lt;div hidden&gt;</code></td>
</tr>
<tr>
<td><code>tabindex</code></td>
<td>키보드 초점 순서·가능 여부</td>
<td><code>tabindex=&quot;0&quot;</code></td>
</tr>
<tr>
<td><code>draggable</code></td>
<td>드래그 가능 여부</td>
<td><code>draggable=&quot;true&quot;</code></td>
</tr>
<tr>
<td><code>contenteditable</code></td>
<td>사용자가 내용을 편집 가능</td>
<td><code>contenteditable=&quot;true&quot;</code></td>
</tr>
<tr>
<td><code>data-*</code></td>
<td>개발자가 정의하는 데이터</td>
<td><code>data-user-id=&quot;42&quot;</code></td>
</tr>
</tbody></table>
<h3 id="id와-class-비교"><code>id</code>와 <code>class</code> 비교</h3>
<pre><code class="language-html">&lt;h1 id=&quot;page-title&quot;&gt;여행 기록&lt;/h1&gt;

&lt;article class=&quot;card featured&quot;&gt;서울&lt;/article&gt;
&lt;article class=&quot;card&quot;&gt;부산&lt;/article&gt;</code></pre>
<ul>
<li><code>id</code>: 페이지 내 고유한 한 요소. CSS에서는 <code>#page-title</code>.</li>
<li><code>class</code>: 여러 요소에 반복 사용 가능. CSS에서는 <code>.card</code>.</li>
<li>한 요소는 여러 클래스를 공백으로 구분해 가질 수 있다.</li>
</ul>
<h2 id="24-제목-문단-줄바꿈">2.4 제목, 문단, 줄바꿈</h2>
<pre><code class="language-html">&lt;h1&gt;페이지의 대표 제목&lt;/h1&gt;
&lt;h2&gt;첫 번째 주제&lt;/h2&gt;
&lt;p&gt;하나의 문단입니다.&lt;/p&gt;
&lt;hr&gt;
&lt;p&gt;주소: 서울시&lt;br&gt;연락처: 02-000-0000&lt;/p&gt;</code></pre>
<ul>
<li><code>&lt;h1&gt;</code>~<code>&lt;h6&gt;</code>: 제목의 <strong>계층</strong>을 표현한다. 단순히 글자 크기를 고르는 태그가 아니다.</li>
<li><code>&lt;p&gt;</code>: 하나의 문단. 소스의 여러 공백·줄바꿈은 보통 하나의 공백으로 합쳐진다.</li>
<li><code>&lt;br&gt;</code>: 문장 안의 의미 있는 줄바꿈.</li>
<li><code>&lt;hr&gt;</code>: 주제 전환을 나타내는 구분선.</li>
</ul>
<h2 id="25-텍스트-의미와-표현">2.5 텍스트 의미와 표현</h2>
<table>
<thead>
<tr>
<th>태그</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;strong&gt;</code></td>
<td>중요함을 표시</td>
</tr>
<tr>
<td><code>&lt;em&gt;</code></td>
<td>말의 강조를 표시</td>
</tr>
<tr>
<td><code>&lt;b&gt;</code></td>
<td>의미 추가 없이 굵은 모양</td>
</tr>
<tr>
<td><code>&lt;i&gt;</code></td>
<td>의미 추가 없이 기울임 모양</td>
</tr>
<tr>
<td><code>&lt;mark&gt;</code></td>
<td>표시·하이라이트</td>
</tr>
<tr>
<td><code>&lt;small&gt;</code></td>
<td>부가 설명·작은 글씨</td>
</tr>
<tr>
<td><code>&lt;del&gt;</code> / <code>&lt;ins&gt;</code></td>
<td>삭제된 내용 / 추가된 내용</td>
</tr>
<tr>
<td><code>&lt;sub&gt;</code> / <code>&lt;sup&gt;</code></td>
<td>아래 첨자 / 위 첨자</td>
</tr>
<tr>
<td><code>&lt;code&gt;</code></td>
<td>짧은 코드 조각</td>
</tr>
<tr>
<td><code>&lt;pre&gt;</code></td>
<td>공백과 줄바꿈을 유지</td>
</tr>
</tbody></table>
<blockquote>
<p>원본 일부의 <code>&lt;string&gt;</code> 표기는 오타이다. 중요성을 나타내는 올바른 태그는 <code>&lt;strong&gt;</code>이다.</p>
</blockquote>
<h2 id="26-block과-inline">2.6 Block과 Inline</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Block</th>
<th>Inline</th>
</tr>
</thead>
<tbody><tr>
<td>배치</td>
<td>새 줄에서 시작</td>
<td>같은 줄 안에 배치</td>
</tr>
<tr>
<td>기본 너비</td>
<td>가능한 가로 공간을 차지</td>
<td>콘텐츠 크기만큼 차지</td>
</tr>
<tr>
<td>대표 태그</td>
<td><code>div</code>, <code>p</code>, <code>h1</code>, <code>section</code>, <code>ul</code></td>
<td><code>span</code>, <code>a</code>, <code>strong</code>, <code>em</code>, <code>img</code></td>
</tr>
<tr>
<td>주 용도</td>
<td>큰 구조·구역</td>
<td>문장 일부·작은 콘텐츠</td>
</tr>
</tbody></table>
<pre><code class="language-html">&lt;div class=&quot;card&quot;&gt;
  &lt;p&gt;가격은 &lt;span class=&quot;price&quot;&gt;10,000원&lt;/span&gt;입니다.&lt;/p&gt;
&lt;/div&gt;</code></pre>
<ul>
<li><code>&lt;div&gt;</code>: 의미 없는 범용 block 컨테이너</li>
<li><code>&lt;span&gt;</code>: 의미 없는 범용 inline 컨테이너</li>
<li>의미에 맞는 <code>&lt;main&gt;</code>, <code>&lt;section&gt;</code>, <code>&lt;article&gt;</code> 등이 있다면 무조건 <code>&lt;div&gt;</code>만 쓰지 않는다.</li>
</ul>
<h2 id="27-링크">2.7 링크</h2>
<pre><code class="language-html">&lt;a href=&quot;profile.html&quot;&gt;프로필&lt;/a&gt;
&lt;a href=&quot;https://example.com&quot; target=&quot;_blank&quot; rel=&quot;noopener&quot;&gt;새 탭으로 열기&lt;/a&gt;
&lt;a href=&quot;#contact&quot;&gt;연락처로 이동&lt;/a&gt;

&lt;h2 id=&quot;contact&quot;&gt;연락처&lt;/h2&gt;</code></pre>
<ul>
<li><code>href</code>: 이동할 주소</li>
<li><code>target=&quot;_self&quot;</code>: 현재 탭에서 열기(기본값)</li>
<li><code>target=&quot;_blank&quot;</code>: 새 탭에서 열기</li>
<li><code>#id</code>: 같은 문서의 해당 <code>id</code>로 이동</li>
<li>새 탭으로 외부 사이트를 열 때 <code>rel=&quot;noopener&quot;</code>를 함께 쓰는 습관이 좋다.</li>
</ul>
<h2 id="28-목록">2.8 목록</h2>
<h3 id="순서-없는-목록">순서 없는 목록</h3>
<pre><code class="language-html">&lt;ul&gt;
  &lt;li&gt;HTML&lt;/li&gt;
  &lt;li&gt;CSS&lt;/li&gt;
&lt;/ul&gt;</code></pre>
<h3 id="순서-있는-목록">순서 있는 목록</h3>
<pre><code class="language-html">&lt;ol&gt;
  &lt;li&gt;파일을 연다.&lt;/li&gt;
  &lt;li&gt;코드를 작성한다.&lt;/li&gt;
  &lt;li&gt;브라우저에서 확인한다.&lt;/li&gt;
&lt;/ol&gt;</code></pre>
<h3 id="중첩-목록">중첩 목록</h3>
<pre><code class="language-html">&lt;ul&gt;
  &lt;li&gt;Frontend
    &lt;ol&gt;
      &lt;li&gt;HTML&lt;/li&gt;
      &lt;li&gt;CSS&lt;/li&gt;
    &lt;/ol&gt;
  &lt;/li&gt;
&lt;/ul&gt;</code></pre>
<h3 id="설명-목록">설명 목록</h3>
<pre><code class="language-html">&lt;dl&gt;
  &lt;dt&gt;HTML&lt;/dt&gt;
  &lt;dd&gt;웹 문서의 구조와 의미를 표현하는 언어&lt;/dd&gt;
  &lt;dt&gt;CSS&lt;/dt&gt;
  &lt;dd&gt;웹 문서의 모양과 배치를 표현하는 언어&lt;/dd&gt;
&lt;/dl&gt;</code></pre>
<h2 id="29-표">2.9 표</h2>
<pre><code class="language-html">&lt;table&gt;
  &lt;caption&gt;주간 수업 시간표&lt;/caption&gt;
  &lt;thead&gt;
    &lt;tr&gt;
      &lt;th scope=&quot;col&quot;&gt;시간&lt;/th&gt;
      &lt;th scope=&quot;col&quot;&gt;월요일&lt;/th&gt;
      &lt;th scope=&quot;col&quot;&gt;화요일&lt;/th&gt;
    &lt;/tr&gt;
  &lt;/thead&gt;
  &lt;tbody&gt;
    &lt;tr&gt;
      &lt;th scope=&quot;row&quot;&gt;09:00&lt;/th&gt;
      &lt;td&gt;HTML&lt;/td&gt;
      &lt;td rowspan=&quot;2&quot;&gt;CSS 실습&lt;/td&gt;
    &lt;/tr&gt;
    &lt;tr&gt;
      &lt;th scope=&quot;row&quot;&gt;10:00&lt;/th&gt;
      &lt;td&gt;복습&lt;/td&gt;
    &lt;/tr&gt;
  &lt;/tbody&gt;
  &lt;tfoot&gt;
    &lt;tr&gt;
      &lt;th scope=&quot;row&quot;&gt;합계&lt;/th&gt;
      &lt;td colspan=&quot;2&quot;&gt;4시간&lt;/td&gt;
    &lt;/tr&gt;
  &lt;/tfoot&gt;
&lt;/table&gt;</code></pre>
<table>
<thead>
<tr>
<th>태그·속성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;table&gt;</code></td>
<td>표 전체</td>
</tr>
<tr>
<td><code>&lt;caption&gt;</code></td>
<td>표의 제목</td>
</tr>
<tr>
<td><code>&lt;thead&gt;</code> / <code>&lt;tbody&gt;</code> / <code>&lt;tfoot&gt;</code></td>
<td>머리글·본문·요약 영역</td>
</tr>
<tr>
<td><code>&lt;tr&gt;</code></td>
<td>한 행(row)</td>
</tr>
<tr>
<td><code>&lt;th&gt;</code></td>
<td>행·열의 제목 셀</td>
</tr>
<tr>
<td><code>&lt;td&gt;</code></td>
<td>일반 데이터 셀</td>
</tr>
<tr>
<td><code>colspan=&quot;2&quot;</code></td>
<td>가로 셀 2개 병합</td>
</tr>
<tr>
<td><code>rowspan=&quot;2&quot;</code></td>
<td>세로 셀 2개 병합</td>
</tr>
</tbody></table>
<p>표의 모양은 <code>border</code>, <code>padding</code>, <code>background-color</code> 등의 CSS로 꾸미는 것이 권장된다. 오래된 <code>border</code>, <code>cellpadding</code>, <code>bgcolor</code> HTML 속성은 피한다.</p>
<h2 id="210-이모지와-문자-인코딩">2.10 이모지와 문자 인코딩</h2>
<p>이모지는 이미지가 아니라 Unicode 문자이다. 문서에 <code>&lt;meta charset=&quot;UTF-8&quot;&gt;</code>을 지정하고 일반 문자처럼 사용할 수 있다.</p>
<pre><code class="language-html">&lt;p aria-label=&quot;완료&quot;&gt;✅ 과제를 제출했습니다.&lt;/p&gt;</code></pre>
<h3 id="2장-핵심-정리">2장 핵심 정리</h3>
<ul>
<li>HTML 태그는 모양보다 콘텐츠의 구조와 의미를 표현한다.</li>
<li>속성은 시작 태그에 <code>이름=&quot;값&quot;</code> 형태로 쓴다.</li>
<li><code>id</code>는 고유 식별자, <code>class</code>는 반복 가능한 분류이다.</li>
<li>block은 큰 구조, inline은 문장 내부의 작은 영역에 주로 사용한다.</li>
<li>목록과 표는 콘텐츠의 관계를 구조적으로 표현한다.</li>
</ul>
<hr />
<h1 id="3-html-form">3. HTML Form</h1>
<h2 id="31-form의-역할">3.1 Form의 역할</h2>
<p><strong>한 문장 이해:</strong> <code>&lt;form&gt;</code>은 사용자가 입력한 값을 이름표와 함께 서버로 보내는 영역이다.</p>
<pre><code class="language-html">&lt;form action=&quot;signup-result.html&quot; method=&quot;get&quot;&gt;
  &lt;!-- 입력 요소 --&gt;
&lt;/form&gt;</code></pre>
<table>
<thead>
<tr>
<th><code>&lt;form&gt;</code> 속성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>action</code></td>
<td>제출할 목적지 URL</td>
</tr>
<tr>
<td><code>method</code></td>
<td>전송 방식: <code>get</code> 또는 <code>post</code></td>
</tr>
<tr>
<td><code>autocomplete</code></td>
<td>자동 완성 사용 여부</td>
</tr>
<tr>
<td><code>enctype</code></td>
<td>POST 데이터 인코딩 방식. 파일 업로드 시 중요</td>
</tr>
<tr>
<td><code>novalidate</code></td>
<td>브라우저 기본 검증을 생략</td>
</tr>
<tr>
<td><code>target</code></td>
<td>제출 결과를 열 위치</td>
</tr>
</tbody></table>
<h3 id="get과-post">GET과 POST</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>GET</th>
<th>POST</th>
</tr>
</thead>
<tbody><tr>
<td>데이터 위치</td>
<td>URL의 query string</td>
<td>요청 body</td>
</tr>
<tr>
<td>특징</td>
<td>URL에 값이 보여 공유·북마크 가능</td>
<td>URL에 값이 직접 드러나지 않음</td>
</tr>
<tr>
<td>용도</td>
<td>검색·조회 조건</td>
<td>가입·등록·파일 업로드</td>
</tr>
<tr>
<td>주의</td>
<td>비밀번호 전송에 사용하면 안 됨</td>
<td>HTTPS 없이는 안전하다고 볼 수 없음</td>
</tr>
</tbody></table>
<h2 id="32-label-id-name-value의-관계">3.2 <code>label</code>, <code>id</code>, <code>name</code>, <code>value</code>의 관계</h2>
<pre><code class="language-html">&lt;label for=&quot;user-name&quot;&gt;이름&lt;/label&gt;
&lt;input id=&quot;user-name&quot; name=&quot;userName&quot; value=&quot;홍길동&quot;&gt;</code></pre>
<table>
<thead>
<tr>
<th>항목</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>id</code></td>
<td>문서 안에서 입력창을 식별. <code>label for</code>와 연결</td>
</tr>
<tr>
<td><code>for</code></td>
<td>같은 값을 가진 <code>id</code>의 입력창을 가리킴</td>
</tr>
<tr>
<td><code>name</code></td>
<td>서버에 보낼 데이터의 이름</td>
</tr>
<tr>
<td><code>value</code></td>
<td>서버에 보낼 값 또는 초기값</td>
</tr>
</tbody></table>
<p>제출 결과는 개념적으로 다음처럼 된다.</p>
<pre><code class="language-text">userName=홍길동
└─ name  └─ value</code></pre>
<blockquote>
<p>입력창에 값이 보여도 <code>name</code>이 없으면 일반적인 폼 제출 데이터에 포함되지 않는다.</p>
</blockquote>
<h2 id="33-form-요소">3.3 Form 요소</h2>
<table>
<thead>
<tr>
<th>요소</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;input&gt;</code></td>
<td>한 줄 입력, 선택, 파일 등 다양한 입력</td>
</tr>
<tr>
<td><code>&lt;label&gt;</code></td>
<td>입력 필드의 이름·설명</td>
</tr>
<tr>
<td><code>&lt;textarea&gt;</code></td>
<td>여러 줄 텍스트 입력</td>
</tr>
<tr>
<td><code>&lt;select&gt;</code> / <code>&lt;option&gt;</code></td>
<td>드롭다운 목록과 선택 항목</td>
</tr>
<tr>
<td><code>&lt;button&gt;</code></td>
<td>제출·초기화·일반 버튼</td>
</tr>
<tr>
<td><code>&lt;fieldset&gt;</code> / <code>&lt;legend&gt;</code></td>
<td>관련 입력을 하나의 그룹으로 묶고 제목 제공</td>
</tr>
<tr>
<td><code>&lt;datalist&gt;</code></td>
<td>input에 추천 선택지 제공</td>
</tr>
<tr>
<td><code>&lt;output&gt;</code></td>
<td>계산 결과 출력</td>
</tr>
</tbody></table>
<h2 id="34-자주-쓰는-input-type">3.4 자주 쓰는 input type</h2>
<table>
<thead>
<tr>
<th>type</th>
<th>용도</th>
<th>핵심 속성·주의</th>
</tr>
</thead>
<tbody><tr>
<td><code>text</code></td>
<td>일반 한 줄 텍스트</td>
<td><code>minlength</code>, <code>maxlength</code></td>
</tr>
<tr>
<td><code>password</code></td>
<td>가려진 입력</td>
<td>보안 전송은 HTTPS가 담당</td>
</tr>
<tr>
<td><code>email</code></td>
<td>이메일</td>
<td>브라우저의 기본 형식 검증</td>
</tr>
<tr>
<td><code>number</code></td>
<td>숫자</td>
<td><code>min</code>, <code>max</code>, <code>step</code></td>
</tr>
<tr>
<td><code>tel</code></td>
<td>전화번호</td>
<td>형식 검증은 자동 보장되지 않음</td>
</tr>
<tr>
<td><code>url</code></td>
<td>URL</td>
<td>브라우저의 기본 형식 검증</td>
</tr>
<tr>
<td><code>date</code></td>
<td>날짜</td>
<td>브라우저별 UI가 다를 수 있음</td>
</tr>
<tr>
<td><code>color</code></td>
<td>색상 선택</td>
<td>색상 선택기 제공</td>
</tr>
<tr>
<td><code>range</code></td>
<td>범위 슬라이더</td>
<td><code>min</code>, <code>max</code>, <code>step</code></td>
</tr>
<tr>
<td><code>checkbox</code></td>
<td>0개 이상 다중 선택</td>
<td>같은 <code>name</code> 사용 가능</td>
</tr>
<tr>
<td><code>radio</code></td>
<td>그룹에서 하나만 선택</td>
<td>같은 그룹은 같은 <code>name</code> 필수</td>
</tr>
<tr>
<td><code>file</code></td>
<td>파일 선택</td>
<td>form에 <code>method=&quot;post&quot;</code>, <code>enctype=&quot;multipart/form-data&quot;</code></td>
</tr>
<tr>
<td><code>hidden</code></td>
<td>화면에 보이지 않는 값</td>
<td>비밀 값을 숨기는 보안 기능은 아님</td>
</tr>
</tbody></table>
<h2 id="35-주요-input-속성">3.5 주요 input 속성</h2>
<table>
<thead>
<tr>
<th>속성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>placeholder</code></td>
<td>입력 전 표시되는 짧은 힌트. label을 대신할 수 없음</td>
</tr>
<tr>
<td><code>required</code></td>
<td>제출 전 반드시 입력</td>
</tr>
<tr>
<td><code>readonly</code></td>
<td>수정 불가. 값은 보통 제출됨</td>
</tr>
<tr>
<td><code>disabled</code></td>
<td>조작 불가. 값도 일반적으로 제출되지 않음</td>
</tr>
<tr>
<td><code>minlength</code> / <code>maxlength</code></td>
<td>문자열 최소·최대 길이</td>
</tr>
<tr>
<td><code>min</code> / <code>max</code> / <code>step</code></td>
<td>숫자·날짜 범위와 간격</td>
</tr>
<tr>
<td><code>pattern</code></td>
<td>정규 표현식 형식 검사</td>
</tr>
<tr>
<td><code>checked</code></td>
<td>checkbox/radio 초기 선택</td>
</tr>
<tr>
<td><code>selected</code></td>
<td>option 초기 선택</td>
</tr>
<tr>
<td><code>multiple</code></td>
<td>여러 값 선택 허용</td>
</tr>
<tr>
<td><code>autofocus</code></td>
<td>페이지 로드 시 초점. 남용 금지</td>
</tr>
<tr>
<td><code>autocomplete</code></td>
<td>자동 완성 힌트·사용 여부</td>
</tr>
</tbody></table>
<h2 id="36-버튼-타입">3.6 버튼 타입</h2>
<pre><code class="language-html">&lt;button type=&quot;submit&quot;&gt;가입하기&lt;/button&gt;
&lt;button type=&quot;reset&quot;&gt;다시 작성&lt;/button&gt;
&lt;button type=&quot;button&quot;&gt;주소 찾기&lt;/button&gt;</code></pre>
<ul>
<li><code>submit</code>: form 제출</li>
<li><code>reset</code>: 입력값 초기화</li>
<li><code>button</code>: 기본 동작이 없는 일반 버튼</li>
</ul>
<blockquote>
<p><code>&lt;form&gt;</code> 안의 <code>&lt;button&gt;</code>은 브라우저에서 기본적으로 <code>submit</code>이 될 수 있다. 의도를 분명하게 하려면 항상 <code>type</code>을 쓴다.</p>
</blockquote>
<h2 id="37-종합-예제-회원가입-폼">3.7 종합 예제: 회원가입 폼</h2>
<pre><code class="language-html">&lt;form action=&quot;signup-result.html&quot; method=&quot;get&quot;&gt;
  &lt;fieldset&gt;
    &lt;legend&gt;계정 정보&lt;/legend&gt;

    &lt;p&gt;
      &lt;label for=&quot;user-id&quot;&gt;아이디&lt;/label&gt;
      &lt;input
        type=&quot;text&quot;
        id=&quot;user-id&quot;
        name=&quot;userId&quot;
        minlength=&quot;4&quot;
        maxlength=&quot;15&quot;
        required
        autocomplete=&quot;username&quot;
      &gt;
    &lt;/p&gt;

    &lt;p&gt;
      &lt;label for=&quot;user-password&quot;&gt;비밀번호&lt;/label&gt;
      &lt;input
        type=&quot;password&quot;
        id=&quot;user-password&quot;
        name=&quot;userPassword&quot;
        minlength=&quot;8&quot;
        required
        autocomplete=&quot;new-password&quot;
      &gt;
    &lt;/p&gt;

    &lt;p&gt;
      &lt;label for=&quot;email&quot;&gt;이메일&lt;/label&gt;
      &lt;input type=&quot;email&quot; id=&quot;email&quot; name=&quot;email&quot; required&gt;
    &lt;/p&gt;
  &lt;/fieldset&gt;

  &lt;fieldset&gt;
    &lt;legend&gt;프로필&lt;/legend&gt;

    &lt;p&gt;관심 분야&lt;/p&gt;
    &lt;label&gt;&lt;input type=&quot;checkbox&quot; name=&quot;interest&quot; value=&quot;html&quot;&gt; HTML&lt;/label&gt;
    &lt;label&gt;&lt;input type=&quot;checkbox&quot; name=&quot;interest&quot; value=&quot;css&quot;&gt; CSS&lt;/label&gt;

    &lt;p&gt;학습 수준&lt;/p&gt;
    &lt;label&gt;&lt;input type=&quot;radio&quot; name=&quot;level&quot; value=&quot;beginner&quot; checked&gt; 초급&lt;/label&gt;
    &lt;label&gt;&lt;input type=&quot;radio&quot; name=&quot;level&quot; value=&quot;intermediate&quot;&gt; 중급&lt;/label&gt;

    &lt;p&gt;
      &lt;label for=&quot;intro&quot;&gt;자기소개&lt;/label&gt;&lt;br&gt;
      &lt;textarea id=&quot;intro&quot; name=&quot;intro&quot; rows=&quot;5&quot; maxlength=&quot;200&quot;&gt;&lt;/textarea&gt;
    &lt;/p&gt;
  &lt;/fieldset&gt;

  &lt;button type=&quot;submit&quot;&gt;회원가입&lt;/button&gt;
  &lt;button type=&quot;reset&quot;&gt;다시 작성&lt;/button&gt;
&lt;/form&gt;</code></pre>
<h3 id="3장-핵심-정리">3장 핵심 정리</h3>
<ul>
<li>폼 데이터는 <code>name=value</code> 쌍으로 전송된다.</li>
<li><code>label for</code>와 input의 <code>id</code>를 연결하면 클릭 영역과 접근성이 좋아진다.</li>
<li>radio는 같은 <code>name</code>을 사용해야 하나의 그룹에서 하나만 선택된다.</li>
<li><code>required</code>, <code>min</code>, <code>pattern</code> 등은 사용자 실수를 줄이지만 서버 검증을 대신하지 못한다.</li>
<li>버튼에는 <code>type</code>을 명시한다.</li>
</ul>
<hr />
<h1 id="4-html-심화">4. HTML 심화</h1>
<h2 id="41-이미지와-반응형-이미지">4.1 이미지와 반응형 이미지</h2>
<pre><code class="language-html">&lt;img
  src=&quot;media/jeju.jpg&quot;
  alt=&quot;성산일출봉에서 바라본 일출&quot;
  width=&quot;800&quot;
  height=&quot;450&quot;
&gt;</code></pre>
<ul>
<li><code>src</code>: 이미지 경로</li>
<li><code>alt</code>: 이미지가 전달하는 의미를 텍스트로 설명</li>
<li><code>width</code>, <code>height</code>: 원본 비율을 알려주어 레이아웃 흔들림을 줄일 수 있음</li>
<li>화면에 맞춘 실제 크기는 CSS로 조절하는 편이 좋다.</li>
</ul>
<pre><code class="language-css">img {
  max-width: 100%;
  height: auto;
}</code></pre>
<p>서로 다른 화면 조건에 다른 이미지를 제공할 때 <code>&lt;picture&gt;</code>를 사용한다.</p>
<pre><code class="language-html">&lt;picture&gt;
  &lt;source media=&quot;(min-width: 900px)&quot; srcset=&quot;hero-large.jpg&quot;&gt;
  &lt;source media=&quot;(min-width: 600px)&quot; srcset=&quot;hero-medium.jpg&quot;&gt;
  &lt;img src=&quot;hero-small.jpg&quot; alt=&quot;바닷가 전경&quot;&gt;
&lt;/picture&gt;</code></pre>
<p><code>&lt;img&gt;</code>는 지원되지 않는 조건에 대비한 마지막 대체 이미지이다.</p>
<h2 id="42-오디오와-비디오">4.2 오디오와 비디오</h2>
<pre><code class="language-html">&lt;audio controls&gt;
  &lt;source src=&quot;music.ogg&quot; type=&quot;audio/ogg&quot;&gt;
  &lt;source src=&quot;music.mp3&quot; type=&quot;audio/mpeg&quot;&gt;
  브라우저가 오디오를 지원하지 않습니다.
&lt;/audio&gt;

&lt;video controls width=&quot;640&quot; poster=&quot;thumbnail.jpg&quot;&gt;
  &lt;source src=&quot;trip.mp4&quot; type=&quot;video/mp4&quot;&gt;
  &lt;track kind=&quot;captions&quot; src=&quot;captions-ko.vtt&quot; srclang=&quot;ko&quot; label=&quot;한국어&quot;&gt;
  브라우저가 비디오를 지원하지 않습니다.
&lt;/video&gt;</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>controls</code></td>
<td>재생·일시정지·음량 UI 표시</td>
</tr>
<tr>
<td><code>autoplay</code></td>
<td>자동 재생. 브라우저가 제한할 수 있음</td>
</tr>
<tr>
<td><code>muted</code></td>
<td>음소거. 자동 재생 정책과 함께 자주 사용</td>
</tr>
<tr>
<td><code>loop</code></td>
<td>반복 재생</td>
</tr>
<tr>
<td><code>poster</code></td>
<td>영상 재생 전 표시할 이미지</td>
</tr>
</tbody></table>
<h2 id="43-시맨틱-html">4.3 시맨틱 HTML</h2>
<p><strong>시맨틱(semantic)</strong>은 태그 이름만으로도 콘텐츠의 역할을 알 수 있다는 뜻이다.</p>
<table>
<thead>
<tr>
<th>태그</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;header&gt;</code></td>
<td>페이지·섹션의 머리말</td>
</tr>
<tr>
<td><code>&lt;nav&gt;</code></td>
<td>주요 탐색 링크</td>
</tr>
<tr>
<td><code>&lt;main&gt;</code></td>
<td>페이지의 핵심 콘텐츠. 보통 한 번 사용</td>
</tr>
<tr>
<td><code>&lt;section&gt;</code></td>
<td>하나의 주제로 묶인 구역. 보통 제목 포함</td>
</tr>
<tr>
<td><code>&lt;article&gt;</code></td>
<td>독립적으로 배포·재사용 가능한 글·카드</td>
</tr>
<tr>
<td><code>&lt;aside&gt;</code></td>
<td>본문과 간접적으로 관련된 부가 정보</td>
</tr>
<tr>
<td><code>&lt;footer&gt;</code></td>
<td>페이지·섹션의 바닥글</td>
</tr>
<tr>
<td><code>&lt;figure&gt;</code> / <code>&lt;figcaption&gt;</code></td>
<td>이미지·도표와 설명을 묶음</td>
</tr>
</tbody></table>
<pre><code class="language-html">&lt;body&gt;
  &lt;header&gt;
    &lt;h1&gt;나의 여행 포털&lt;/h1&gt;
    &lt;nav aria-label=&quot;주요 메뉴&quot;&gt;
      &lt;a href=&quot;index.html&quot;&gt;홈&lt;/a&gt;
      &lt;a href=&quot;my-trip.html&quot;&gt;여행&lt;/a&gt;
    &lt;/nav&gt;
  &lt;/header&gt;

  &lt;main&gt;
    &lt;section aria-labelledby=&quot;recent-title&quot;&gt;
      &lt;h2 id=&quot;recent-title&quot;&gt;최근 여행&lt;/h2&gt;
      &lt;article&gt;
        &lt;h3&gt;제주도&lt;/h3&gt;
        &lt;p&gt;성산일출봉을 다녀왔습니다.&lt;/p&gt;
      &lt;/article&gt;
    &lt;/section&gt;

    &lt;aside&gt;
      &lt;h2&gt;여행 팁&lt;/h2&gt;
      &lt;p&gt;날씨를 미리 확인하세요.&lt;/p&gt;
    &lt;/aside&gt;
  &lt;/main&gt;

  &lt;footer&gt;&lt;small&gt;© 2026 My Trip&lt;/small&gt;&lt;/footer&gt;
&lt;/body&gt;</code></pre>
<p>시맨틱 태그의 장점:</p>
<ul>
<li>코드 구조를 사람이 이해하기 쉽다.</li>
<li>스크린 리더가 페이지 영역을 파악하기 쉽다.</li>
<li>검색 엔진이 콘텐츠 관계를 이해하는 데 도움이 된다.</li>
</ul>
<h2 id="44-인용과-약어">4.4 인용과 약어</h2>
<pre><code class="language-html">&lt;blockquote cite=&quot;https://example.com/source&quot;&gt;
  긴 인용문을 작성합니다.
&lt;/blockquote&gt;

&lt;p&gt;&lt;abbr title=&quot;HyperText Markup Language&quot;&gt;HTML&lt;/abbr&gt;을 학습합니다.&lt;/p&gt;</code></pre>
<ul>
<li><code>&lt;blockquote&gt;</code>: 다른 출처의 긴 인용 영역</li>
<li><code>&lt;abbr&gt;</code>: 약어의 전체 의미를 <code>title</code>로 제공</li>
</ul>
<h2 id="45-head와-메타데이터">4.5 <code>&lt;head&gt;</code>와 메타데이터</h2>
<pre><code class="language-html">&lt;head&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
  &lt;meta name=&quot;description&quot; content=&quot;HTML과 CSS 학습 기록&quot;&gt;
  &lt;title&gt;HTML·CSS 학습 노트&lt;/title&gt;
  &lt;link rel=&quot;stylesheet&quot; href=&quot;css/style.css&quot;&gt;
  &lt;script src=&quot;js/app.js&quot; defer&gt;&lt;/script&gt;
&lt;/head&gt;</code></pre>
<table>
<thead>
<tr>
<th>요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;title&gt;</code></td>
<td>브라우저 탭과 검색 결과의 제목</td>
</tr>
<tr>
<td><code>&lt;meta&gt;</code></td>
<td>문자 인코딩, 설명, viewport 등 문서 정보</td>
</tr>
<tr>
<td><code>&lt;link&gt;</code></td>
<td>외부 CSS·아이콘 등 연결</td>
</tr>
<tr>
<td><code>&lt;style&gt;</code></td>
<td>현재 문서 안에 CSS 작성</td>
</tr>
<tr>
<td><code>&lt;script&gt;</code></td>
<td>JavaScript 연결·작성</td>
</tr>
<tr>
<td><code>&lt;base&gt;</code></td>
<td>상대 URL의 기준 주소 설정. 영향 범위가 커서 신중히 사용</td>
</tr>
</tbody></table>
<h2 id="46-주석">4.6 주석</h2>
<pre><code class="language-html">&lt;!-- 브라우저 화면에는 표시되지 않는 설명 --&gt;</code></pre>
<p>주석은 코드의 이유나 큰 구역을 설명하는 데 사용한다. 비밀번호, API 키, 개인정보처럼 공개되면 안 되는 값은 주석에도 넣지 않는다. 사용자는 페이지 소스에서 주석을 볼 수 있다.</p>
<h2 id="47-웹-접근성-기본-규칙">4.7 웹 접근성 기본 규칙</h2>
<p>접근성은 장애 여부나 입력 방식과 관계없이 누구나 웹을 사용할 수 있게 만드는 원칙이다.</p>
<ul>
<li>문서의 주 언어를 <code>&lt;html lang=&quot;ko&quot;&gt;</code>로 지정한다.</li>
<li>제목은 <code>&lt;h1&gt;</code> → <code>&lt;h2&gt;</code>처럼 논리적인 계층으로 사용한다.</li>
<li>페이지 영역은 <code>header</code>, <code>nav</code>, <code>main</code>, <code>footer</code> 등으로 표현한다.</li>
<li>의미 있는 이미지에는 내용을 설명하는 <code>alt</code>를 쓴다.</li>
<li>장식용 이미지는 <code>alt=&quot;&quot;</code>로 스크린 리더가 건너뛰게 한다.</li>
<li>모든 입력창에는 연결된 <code>&lt;label&gt;</code>을 제공한다.</li>
<li>마우스뿐 아니라 키보드 Tab과 Enter로도 조작 가능해야 한다.</li>
<li>색상만으로 오류나 상태를 전달하지 않는다.</li>
<li>자동 재생 미디어는 피하고 영상에는 자막을 제공한다.</li>
</ul>
<h3 id="4장-핵심-정리">4장 핵심 정리</h3>
<ul>
<li>이미지에는 올바른 <code>src</code>와 의미 있는 <code>alt</code>가 필요하다.</li>
<li>여러 미디어 형식을 <code>&lt;source&gt;</code>로 제공하면 브라우저 호환성이 좋아진다.</li>
<li>시맨틱 태그는 가독성, 접근성, 검색 엔진 이해에 도움이 된다.</li>
<li><code>&lt;head&gt;</code>에는 화면 내용이 아니라 문서를 설명·연결하는 정보가 들어간다.</li>
<li>접근성은 나중에 추가하는 장식이 아니라 처음부터 지켜야 하는 HTML 작성 방식이다.</li>
</ul>
<hr />
<h1 id="5-css-기초">5. CSS 기초</h1>
<h2 id="51-css-문법">5.1 CSS 문법</h2>
<p><strong>한 문장 이해:</strong> CSS는 선택자로 HTML 요소를 고르고, 속성과 값으로 모양을 지정한다.</p>
<pre><code class="language-css">p {
  color: red;
  text-align: center;
}</code></pre>
<table>
<thead>
<tr>
<th>부분</th>
<th>예</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>selector</td>
<td><code>p</code></td>
<td>꾸밀 HTML 요소를 선택</td>
</tr>
<tr>
<td>property</td>
<td><code>color</code></td>
<td>바꿀 스타일 항목</td>
</tr>
<tr>
<td>value</td>
<td><code>red</code></td>
<td>적용할 값</td>
</tr>
<tr>
<td>declaration</td>
<td><code>color: red;</code></td>
<td>속성과 값 한 쌍</td>
</tr>
<tr>
<td>declaration block</td>
<td><code>{ ... }</code></td>
<td>여러 선언을 묶는 영역</td>
</tr>
</tbody></table>
<h2 id="52-css를-적용하는-세-방법">5.2 CSS를 적용하는 세 방법</h2>
<h3 id="inline">Inline</h3>
<pre><code class="language-html">&lt;h1 style=&quot;color: blue;&quot;&gt;제목&lt;/h1&gt;</code></pre>
<p>빠른 확인에는 편하지만 반복·유지보수에 불리하다.</p>
<h3 id="internal">Internal</h3>
<pre><code class="language-html">&lt;head&gt;
  &lt;style&gt;
    h1 { color: blue; }
  &lt;/style&gt;
&lt;/head&gt;</code></pre>
<p>한 문서에만 필요한 간단한 스타일에 사용할 수 있다.</p>
<h3 id="external---권장">External - 권장</h3>
<pre><code class="language-html">&lt;link rel=&quot;stylesheet&quot; href=&quot;css/style.css&quot;&gt;</code></pre>
<pre><code class="language-css">/* css/style.css */
h1 {
  color: blue;
}</code></pre>
<p>여러 HTML 문서가 같은 스타일을 공유할 수 있고 관리하기 쉽다.</p>
<h2 id="53-선택자">5.3 선택자</h2>
<h3 id="기본-선택자">기본 선택자</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>문법</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td>전체</td>
<td><code>*</code></td>
<td>모든 요소</td>
</tr>
<tr>
<td>요소</td>
<td><code>p</code></td>
<td>모든 <code>&lt;p&gt;</code></td>
</tr>
<tr>
<td>클래스</td>
<td><code>.card</code></td>
<td>class에 <code>card</code>가 있는 요소</td>
</tr>
<tr>
<td>ID</td>
<td><code>#header</code></td>
<td>id가 <code>header</code>인 요소</td>
</tr>
<tr>
<td>그룹</td>
<td><code>h1, h2, p</code></td>
<td>여러 선택자를 함께 선택</td>
</tr>
</tbody></table>
<h3 id="결합자">결합자</h3>
<p>HTML:</p>
<pre><code class="language-html">&lt;section class=&quot;news&quot;&gt;
  &lt;article&gt;
    &lt;h2&gt;제목&lt;/h2&gt;
    &lt;p&gt;첫 문단&lt;/p&gt;
    &lt;p&gt;둘째 문단&lt;/p&gt;
  &lt;/article&gt;
&lt;/section&gt;</code></pre>
<table>
<thead>
<tr>
<th>선택자</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td><code>.news p</code></td>
<td><code>.news</code> 안의 모든 후손 <code>p</code></td>
</tr>
<tr>
<td><code>.news &gt; article</code></td>
<td><code>.news</code>의 직계 자식 <code>article</code></td>
</tr>
<tr>
<td><code>h2 + p</code></td>
<td><code>h2</code> 바로 다음 형제 <code>p</code> 하나</td>
</tr>
<tr>
<td><code>h2 ~ p</code></td>
<td><code>h2</code> 뒤에 있는 같은 부모의 모든 <code>p</code></td>
</tr>
</tbody></table>
<h3 id="속성-선택자">속성 선택자</h3>
<table>
<thead>
<tr>
<th>선택자</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td><code>[required]</code></td>
<td><code>required</code> 속성이 있는 요소</td>
</tr>
<tr>
<td><code>input[type=&quot;email&quot;]</code></td>
<td>type 값이 정확히 email인 input</td>
</tr>
<tr>
<td><code>a[href^=&quot;https&quot;]</code></td>
<td>href가 <code>https</code>로 시작</td>
</tr>
<tr>
<td><code>a[href$=&quot;.pdf&quot;]</code></td>
<td>href가 <code>.pdf</code>로 끝남</td>
</tr>
<tr>
<td><code>[class~=&quot;card&quot;]</code></td>
<td>공백으로 나뉜 단어 중 <code>card</code>가 있음</td>
</tr>
<tr>
<td><code>[data-name*=&quot;trip&quot;]</code></td>
<td>값에 <code>trip</code> 문자열 포함</td>
</tr>
</tbody></table>
<h3 id="의사-클래스pseudo-class">의사 클래스(pseudo-class)</h3>
<p>요소의 상태나 구조적 위치를 선택하며 콜론 하나 <code>:</code>를 쓴다.</p>
<pre><code class="language-css">a:hover { color: tomato; }
input:focus { outline: 3px solid royalblue; }
li:first-child { font-weight: bold; }
li:nth-child(2n) { background: #f3f4f6; }</code></pre>
<table>
<thead>
<tr>
<th>종류</th>
<th>예</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>상호작용</td>
<td><code>:hover</code>, <code>:active</code>, <code>:focus</code></td>
<td>마우스·키보드 상태</td>
</tr>
<tr>
<td>링크</td>
<td><code>:link</code>, <code>:visited</code></td>
<td>방문 여부</td>
</tr>
<tr>
<td>구조</td>
<td><code>:first-child</code>, <code>:last-child</code>, <code>:nth-child(n)</code></td>
<td>부모 안에서의 위치</td>
</tr>
<tr>
<td>타입 구조</td>
<td><code>:first-of-type</code>, <code>:last-of-type</code></td>
<td>같은 태그 타입 중 위치</td>
</tr>
</tbody></table>
<h3 id="의사-요소pseudo-element">의사 요소(pseudo-element)</h3>
<p>요소의 특정 부분이나 가상 콘텐츠를 선택하며 콜론 두 개 <code>::</code>를 쓴다.</p>
<pre><code class="language-css">p::first-letter { font-size: 2rem; }
::selection { background: gold; }
.required-label::after {
  content: &quot; *&quot;;
  color: red;
}</code></pre>
<p><code>::before</code>와 <code>::after</code>로 만든 콘텐츠는 핵심 정보 전달용으로 쓰지 않는다. 스크린 리더 처리 방식이 일관되지 않을 수 있다.</p>
<h2 id="54-cascading과-specificity">5.4 Cascading과 Specificity</h2>
<p>같은 요소의 같은 속성에 여러 규칙이 충돌하면 브라우저가 우선순위를 계산한다.</p>
<table>
<thead>
<tr>
<th align="right">우선순위</th>
<th>선택자</th>
<th>표기 예</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1</td>
<td>inline style</td>
<td><code>1-0-0-0</code></td>
</tr>
<tr>
<td align="right">2</td>
<td>ID</td>
<td><code>0-1-0-0</code></td>
</tr>
<tr>
<td align="right">3</td>
<td>class, 속성, 의사 클래스</td>
<td><code>0-0-1-0</code></td>
</tr>
<tr>
<td align="right">4</td>
<td>요소, 의사 요소</td>
<td><code>0-0-0-1</code></td>
</tr>
<tr>
<td align="right">0</td>
<td>전체 선택자, <code>:where()</code></td>
<td><code>0-0-0-0</code></td>
</tr>
</tbody></table>
<pre><code class="language-css">p { color: red; }              /* 0-0-0-1 */
.notice { color: green; }      /* 0-0-1-0: 승리 */
#message { color: blue; }      /* 0-1-0-0: 더 강함 */</code></pre>
<p>판단 순서의 핵심:</p>
<ol>
<li><code>!important</code> 여부</li>
<li>작성 위치·출처와 cascade 관계</li>
<li>specificity 비교</li>
<li>조건이 같으면 뒤에 작성된 선언</li>
</ol>
<p><code>!important</code>는 추적과 재정의를 어렵게 만들므로 일상적인 우선순위 해결책으로 남용하지 않는다.</p>
<h2 id="55-색상과-단위">5.5 색상과 단위</h2>
<h3 id="색상">색상</h3>
<pre><code class="language-css">.sample {
  color: tomato;
  border-color: #2563eb;
  background-color: rgb(240 249 255);
  box-shadow: 0 4px 12px rgb(0 0 0 / 15%);
  outline-color: hsl(221 83% 53%);
}</code></pre>
<h3 id="길이-단위">길이 단위</h3>
<table>
<thead>
<tr>
<th>단위</th>
<th>기준</th>
<th>주 사용 예</th>
</tr>
</thead>
<tbody><tr>
<td><code>px</code></td>
<td>화면 픽셀 기준</td>
<td>테두리, 작은 고정 간격</td>
</tr>
<tr>
<td><code>em</code></td>
<td><code>font-size</code>에서는 부모 글자 크기, 그 밖의 속성에서는 해당 요소의 계산된 글자 크기</td>
<td>컴포넌트 내부 상대 크기</td>
</tr>
<tr>
<td><code>rem</code></td>
<td>루트 <code>&lt;html&gt;</code>의 font-size</td>
<td>전체 글꼴·간격 체계</td>
</tr>
<tr>
<td><code>%</code></td>
<td>부모의 관련 크기</td>
<td>유동적인 너비</td>
</tr>
<tr>
<td><code>vw</code></td>
<td>viewport 너비의 1%</td>
<td>화면 기준 크기</td>
</tr>
<tr>
<td><code>vh</code></td>
<td>viewport 높이의 1%</td>
<td>화면 기준 최소 높이</td>
</tr>
<tr>
<td><code>fr</code></td>
<td>Grid의 남은 공간 비율</td>
<td>Grid 열·행 크기</td>
</tr>
</tbody></table>
<p><code>em</code>은 중첩될 때 크기가 누적될 수 있다. 페이지 전체의 일관된 글꼴·간격에는 <code>rem</code>이 이해하기 쉽다.</p>
<h2 id="56-글꼴과-텍스트">5.6 글꼴과 텍스트</h2>
<pre><code class="language-css">body {
  font-family: &quot;Noto Sans KR&quot;, Arial, sans-serif;
  font-size: 1rem;
  line-height: 1.6;
  color: #1f2937;
}

h1 {
  font-size: 2rem;
  font-weight: 700;
  text-align: center;
  letter-spacing: -0.02em;
}

a {
  color: #2563eb;
  text-decoration: underline;
  text-underline-offset: 0.2em;
}</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>font-family</code></td>
<td>글꼴과 대체 글꼴 목록</td>
</tr>
<tr>
<td><code>font-size</code></td>
<td>글자 크기</td>
</tr>
<tr>
<td><code>font-weight</code></td>
<td>굵기</td>
</tr>
<tr>
<td><code>font-style</code></td>
<td>기울임 등</td>
</tr>
<tr>
<td><code>color</code></td>
<td>글자색</td>
</tr>
<tr>
<td><code>text-align</code></td>
<td>정렬</td>
</tr>
<tr>
<td><code>text-transform</code></td>
<td>영문 대·소문자 변환</td>
</tr>
<tr>
<td><code>text-decoration</code></td>
<td>밑줄·취소선 등</td>
</tr>
<tr>
<td><code>line-height</code></td>
<td>줄 높이</td>
</tr>
<tr>
<td><code>letter-spacing</code> / <code>word-spacing</code></td>
<td>글자·단어 간격</td>
</tr>
<tr>
<td><code>text-shadow</code></td>
<td>글자 그림자</td>
</tr>
</tbody></table>
<p>웹 폰트는 <code>&lt;link&gt;</code> 또는 CSS <code>@import</code>로 불러올 수 있다. 네트워크 실패에 대비해 마지막에 <code>sans-serif</code> 같은 일반 글꼴군을 쓴다.</p>
<h2 id="57-배경">5.7 배경</h2>
<pre><code class="language-css">.hero {
  background-color: #dbeafe;
  background-image: url(&quot;../media/hero.jpg&quot;);
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  background-attachment: scroll;
}</code></pre>
<blockquote>
<p>정확한 속성명은 <code>background-attachment</code>이다. <code>opacity</code>를 부모에 주면 글자와 자식까지 모두 투명해진다. 배경만 반투명하게 하려면 <code>rgb(255 255 255 / 80%)</code> 같은 알파 색상을 사용한다.</p>
</blockquote>
<h2 id="58-box-model">5.8 Box Model</h2>
<p>모든 HTML 요소는 다음 네 겹의 상자로 생각할 수 있다.</p>
<pre><code class="language-text">┌──────────────────── margin: 다른 요소와의 바깥 간격
│ ┌────────────────── border: 테두리
│ │ ┌──────────────── padding: 테두리와 내용의 안쪽 간격
│ │ │  content: 실제 내용
│ │ └────────────────
│ └──────────────────
└────────────────────</code></pre>
<p>기본 <code>content-box</code>에서 실제 가로 크기:</p>
<pre><code class="language-text">총 너비 = width + 왼쪽 padding + 오른쪽 padding
        + 왼쪽 border + 오른쪽 border</code></pre>
<pre><code class="language-css">* {
  box-sizing: border-box;
}</code></pre>
<p><code>border-box</code>를 쓰면 선언한 <code>width</code> 안에 padding과 border가 포함되어 크기 계산이 쉬워진다.</p>
<h3 id="width와-height">Width와 Height</h3>
<pre><code class="language-css">.container {
  width: min(calc(100% - 2rem), 70rem);
  margin-inline: auto;
}

img {
  max-width: 100%;
  height: auto;
}</code></pre>
<ul>
<li><code>width</code>는 고정 크기를 강제할 수 있다.</li>
<li><code>max-width</code>는 화면이 좁을 때 요소가 함께 줄어들 수 있게 한다.</li>
<li>콘텐츠 높이는 가능하면 고정하지 않고 내용에 맡긴다.</li>
</ul>
<h3 id="margin과-padding-축약형">Margin과 Padding 축약형</h3>
<pre><code class="language-css">.box { margin: 10px 20px 30px 40px; }
/*              위   오른쪽 아래 왼쪽 */

.box { padding: 10px 20px; }
/*               위아래 좌우 */</code></pre>
<p>시계 방향 <code>top → right → bottom → left</code>로 기억한다.</p>
<p>세로 방향 block margin은 서로 만날 때 큰 값 하나로 합쳐지는 <strong>margin collapse</strong>가 발생할 수 있다. padding은 병합되지 않는다.</p>
<h3 id="border">Border</h3>
<pre><code class="language-css">.card {
  border: 1px solid #d1d5db;
  border-left: 4px solid #2563eb;
  border-radius: 0.75rem;
}</code></pre>
<h2 id="59-position과-z-index">5.9 Position과 z-index</h2>
<table>
<thead>
<tr>
<th>값</th>
<th>기준과 특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>static</code></td>
<td>기본 문서 흐름. <code>top</code>, <code>left</code> 등이 적용되지 않음</td>
</tr>
<tr>
<td><code>relative</code></td>
<td>원래 위치를 기준으로 이동. 원래 공간은 유지</td>
</tr>
<tr>
<td><code>absolute</code></td>
<td>가장 가까운 positioned 조상을 기준으로 배치. 문서 흐름에서 빠짐</td>
</tr>
<tr>
<td><code>fixed</code></td>
<td>viewport 기준으로 고정. 스크롤해도 같은 위치</td>
</tr>
<tr>
<td><code>sticky</code></td>
<td>임계점 전에는 일반 흐름, 이후 스크롤 영역에 고정</td>
</tr>
</tbody></table>
<pre><code class="language-css">.card {
  position: relative;
}

.badge {
  position: absolute;
  top: 0.5rem;
  right: 0.5rem;
}

.site-nav {
  position: sticky;
  top: 0;
  z-index: 10;
}</code></pre>
<p><code>z-index</code>는 겹친 요소의 앞뒤 순서를 정한다. positioned 요소와 stacking context의 영향을 받으므로 값만 무작정 크게 올리기보다 부모의 <code>position</code>, <code>transform</code>, <code>opacity</code> 등도 확인한다.</p>
<h2 id="510-상속">5.10 상속</h2>
<p>부모 스타일 중 일부는 자식에게 자동으로 전달된다.</p>
<table>
<thead>
<tr>
<th>주로 상속됨</th>
<th>주로 상속되지 않음</th>
</tr>
</thead>
<tbody><tr>
<td><code>color</code>, <code>font-family</code>, <code>font-size</code>, <code>line-height</code>, <code>text-align</code></td>
<td><code>margin</code>, <code>padding</code>, <code>border</code>, <code>width</code>, <code>height</code>, <code>display</code>, <code>position</code>, <code>background</code></td>
</tr>
</tbody></table>
<pre><code class="language-css">body {
  color: #1f2937;
  font-family: sans-serif;
}

button {
  font: inherit; /* 폼 요소가 부모 글꼴을 따르게 함 */
}</code></pre>
<p><code>inherit</code>는 원래 상속되지 않는 속성도 부모의 계산된 값을 따르게 한다.</p>
<h2 id="511-css-기초-종합-예제">5.11 CSS 기초 종합 예제</h2>
<pre><code class="language-css">* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: &quot;Noto Sans KR&quot;, sans-serif;
  line-height: 1.6;
  color: #1f2937;
  background: #f8fafc;
}

.container {
  width: min(calc(100% - 2rem), 70rem);
  margin-inline: auto;
}

.card {
  margin-block: 1rem;
  padding: 1.25rem;
  border: 1px solid #dbe2ea;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 0.25rem 1rem rgb(15 23 42 / 8%);
}

.card h2 {
  margin-top: 0;
}

.card a:hover,
.card a:focus-visible {
  color: #1d4ed8;
}</code></pre>
<h3 id="5장-핵심-정리">5장 핵심 정리</h3>
<ul>
<li>CSS 규칙은 <code>선택자 { 속성: 값; }</code> 구조이다.</li>
<li>반복되는 스타일은 외부 CSS 파일로 분리한다.</li>
<li>선택자는 필요한 만큼만 구체적으로 작성하고 <code>!important</code>는 최소화한다.</li>
<li>모든 요소는 content, padding, border, margin으로 구성된 box이다.</li>
<li><code>box-sizing: border-box</code>를 쓰면 크기 계산이 쉬워진다.</li>
<li><code>position: absolute</code>는 기준이 될 부모에 <code>position: relative</code>를 주는 경우가 많다.</li>
</ul>
<hr />
<h1 id="6-css-심화">6. CSS 심화</h1>
<h2 id="61-레이아웃-방법-비교">6.1 레이아웃 방법 비교</h2>
<table>
<thead>
<tr>
<th>방식</th>
<th>차원·특징</th>
<th>지금의 주요 용도</th>
</tr>
</thead>
<tbody><tr>
<td>Float</td>
<td>원래 텍스트가 이미지를 감싸게 하는 기능</td>
<td>본문 속 이미지 배치, 구형 코드 유지</td>
</tr>
<tr>
<td>Inline-block</td>
<td>inline처럼 나란히 놓고 block처럼 크기 지정</td>
<td>작은 메뉴·레거시 코드</td>
</tr>
<tr>
<td>Flexbox</td>
<td>한 축을 중심으로 배치하는 1차원 레이아웃</td>
<td>메뉴, 정렬, 카드 한 줄, 컴포넌트</td>
</tr>
<tr>
<td>Grid</td>
<td>행과 열을 함께 다루는 2차원 레이아웃</td>
<td>페이지 구조, 갤러리, 대시보드</td>
</tr>
</tbody></table>
<p>현대적인 기본 판단:</p>
<ul>
<li>한 방향 정렬·간격 배분 → <strong>Flexbox</strong></li>
<li>행과 열을 함께 설계 → <strong>Grid</strong></li>
<li>본문이 이미지를 감싸야 함 → <strong>Float</strong></li>
</ul>
<h2 id="62-float와-inline-block">6.2 Float와 Inline-block</h2>
<pre><code class="language-css">.photo {
  float: left;
  margin: 0 1rem 0.5rem 0;
}</code></pre>
<p>float 이후 요소가 의도치 않게 감싸면 <code>clear</code> 또는 새로운 block formatting context를 고려한다.</p>
<pre><code class="language-css">.old-nav li {
  display: inline-block;
}</code></pre>
<p>inline-block은 HTML 소스의 공백이 요소 사이 간격으로 보일 수 있다. 새 레이아웃에는 Flexbox가 더 편리한 경우가 많다.</p>
<h2 id="63-flexbox">6.3 Flexbox</h2>
<h3 id="핵심-구조와-축">핵심 구조와 축</h3>
<pre><code class="language-html">&lt;div class=&quot;toolbar&quot;&gt;
  &lt;button&gt;이전&lt;/button&gt;
  &lt;button&gt;저장&lt;/button&gt;
  &lt;button&gt;다음&lt;/button&gt;
&lt;/div&gt;</code></pre>
<pre><code class="language-css">.toolbar {
  display: flex;
}</code></pre>
<ul>
<li>Flex container: <code>display: flex</code>가 적용된 부모</li>
<li>Flex item: 그 부모의 <strong>직계 자식</strong></li>
<li>main axis: item이 흐르는 주축. 기본은 가로(row)</li>
<li>cross axis: 주축과 직각인 교차축</li>
</ul>
<h3 id="container-속성">Container 속성</h3>
<table>
<thead>
<tr>
<th>속성</th>
<th>역할</th>
<th>대표 값</th>
</tr>
</thead>
<tbody><tr>
<td><code>flex-direction</code></td>
<td>주축 방향</td>
<td><code>row</code>, <code>column</code>, <code>row-reverse</code></td>
</tr>
<tr>
<td><code>flex-wrap</code></td>
<td>넘칠 때 줄바꿈</td>
<td><code>nowrap</code>, <code>wrap</code></td>
</tr>
<tr>
<td><code>justify-content</code></td>
<td>주축 정렬·공간 배분</td>
<td><code>flex-start</code>, <code>center</code>, <code>space-between</code>, <code>space-evenly</code></td>
</tr>
<tr>
<td><code>align-items</code></td>
<td>한 줄의 교차축 정렬</td>
<td><code>stretch</code>, <code>center</code>, <code>flex-start</code>, <code>baseline</code></td>
</tr>
<tr>
<td><code>align-content</code></td>
<td>여러 줄 전체의 교차축 배치</td>
<td><code>stretch</code>, <code>center</code>, <code>space-between</code></td>
</tr>
<tr>
<td><code>gap</code></td>
<td>item 사이 간격</td>
<td><code>1rem</code></td>
</tr>
</tbody></table>
<pre><code class="language-css">.toolbar {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 0.75rem;
}</code></pre>
<h3 id="item-속성">Item 속성</h3>
<table>
<thead>
<tr>
<th>속성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>order</code></td>
<td>시각적 순서 변경</td>
</tr>
<tr>
<td><code>flex-grow</code></td>
<td>남은 공간을 늘려 가지는 비율</td>
</tr>
<tr>
<td><code>flex-shrink</code></td>
<td>공간이 부족할 때 줄어드는 비율</td>
</tr>
<tr>
<td><code>flex-basis</code></td>
<td>늘고 줄기 전의 기본 크기</td>
</tr>
<tr>
<td><code>flex</code></td>
<td>grow, shrink, basis 축약</td>
</tr>
<tr>
<td><code>align-self</code></td>
<td>특정 item만 교차축 정렬 변경</td>
</tr>
</tbody></table>
<pre><code class="language-css">.main-content { flex: 1 1 40rem; }
.sidebar { flex: 0 1 18rem; }</code></pre>
<blockquote>
<p><code>order</code>는 시각적 순서만 바꾸고 키보드·스크린 리더 순서와 달라질 수 있다. 콘텐츠 의미 순서는 HTML에서 올바르게 작성한다.</p>
</blockquote>
<h2 id="64-grid">6.4 Grid</h2>
<h3 id="핵심-용어">핵심 용어</h3>
<ul>
<li>Grid container: <code>display: grid</code>인 부모</li>
<li>Grid item: container의 직계 자식</li>
<li>Grid line: 칸을 나누는 선</li>
<li>Grid track: 하나의 행 또는 열</li>
<li>Grid cell: 한 칸</li>
<li>Grid area: 여러 칸을 합친 영역</li>
</ul>
<h3 id="container-속성-1">Container 속성</h3>
<pre><code class="language-css">.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>grid-template-columns</code></td>
<td>열의 수와 너비</td>
</tr>
<tr>
<td><code>grid-template-rows</code></td>
<td>행의 수와 높이</td>
</tr>
<tr>
<td><code>gap</code>, <code>row-gap</code>, <code>column-gap</code></td>
<td>track 사이 간격</td>
</tr>
<tr>
<td><code>justify-items</code></td>
<td>각 셀 안 item의 가로 정렬</td>
</tr>
<tr>
<td><code>align-items</code></td>
<td>각 셀 안 item의 세로 정렬</td>
</tr>
</tbody></table>
<p><code>fr</code>은 Grid에서 남은 공간을 나누어 가지는 비율이다.</p>
<pre><code class="language-css">.layout {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
}</code></pre>
<h3 id="item-위치-지정">Item 위치 지정</h3>
<pre><code class="language-css">.featured {
  grid-column: 1 / 3;
  grid-row: 1 / 2;
  justify-self: stretch;
  align-self: start;
}</code></pre>
<p><code>grid-column: 1 / 3</code>은 1번 <strong>선</strong>에서 3번 선까지 차지하므로 두 칸을 사용한다.</p>
<h3 id="반응형-카드-grid">반응형 카드 Grid</h3>
<pre><code class="language-css">.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 16rem), 1fr));
  gap: 1rem;
}</code></pre>
<p>화면에 들어갈 수 있는 만큼 열을 만들고, 좁아지면 자동으로 열 수를 줄인다.</p>
<h2 id="65-transform">6.5 Transform</h2>
<p>Transform은 요소의 문서 흐름상 공간을 다시 계산하지 않고 보이는 모양을 변환한다.</p>
<pre><code class="language-css">.card:hover {
  transform: translateY(-4px) scale(1.02);
}

.diamond {
  transform: rotate(45deg);
}</code></pre>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>translate(x, y)</code></td>
<td>이동</td>
</tr>
<tr>
<td><code>scale(x, y)</code></td>
<td>확대·축소</td>
</tr>
<tr>
<td><code>rotate(45deg)</code></td>
<td>2D 회전</td>
</tr>
<tr>
<td><code>skew(10deg)</code></td>
<td>기울이기</td>
</tr>
<tr>
<td><code>rotateX()</code>, <code>rotateY()</code>, <code>rotateZ()</code></td>
<td>3D 축 회전</td>
</tr>
</tbody></table>
<p>3D 효과 관련 속성에는 <code>perspective</code>, <code>transform-style</code>, <code>backface-visibility</code>, <code>transform-origin</code> 등이 있다.</p>
<h2 id="66-transition">6.6 Transition</h2>
<p>Transition은 속성값이 A 상태에서 B 상태로 변할 때 그 사이를 부드럽게 연결한다.</p>
<pre><code class="language-css">.button {
  background: #2563eb;
  transform: translateY(0);
  transition: background-color 200ms ease, transform 200ms ease;
}

.button:hover {
  background: #1d4ed8;
  transform: translateY(-2px);
}</code></pre>
<pre><code class="language-css">/* property duration timing-function delay */
transition: transform 200ms ease 0ms;</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>transition-property</code></td>
<td>변화시킬 CSS 속성</td>
</tr>
<tr>
<td><code>transition-duration</code></td>
<td>진행 시간</td>
</tr>
<tr>
<td><code>transition-timing-function</code></td>
<td>속도 곡선: <code>ease</code>, <code>linear</code> 등</td>
</tr>
<tr>
<td><code>transition-delay</code></td>
<td>시작 전 대기 시간</td>
</tr>
</tbody></table>
<p><code>transition: all</code>은 예상하지 않은 속성까지 애니메이션할 수 있으므로 바뀔 속성을 명시하는 편이 좋다.</p>
<h2 id="67-animation">6.7 Animation</h2>
<p>Transition이 두 상태 사이의 변화라면 Animation은 여러 중간 단계를 정의하고 자동 반복할 수 있다.</p>
<pre><code class="language-css">@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(-1rem);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.site-title {
  animation: fade-in 600ms ease-out both;
}</code></pre>
<table>
<thead>
<tr>
<th>속성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>animation-name</code></td>
<td>사용할 <code>@keyframes</code> 이름</td>
</tr>
<tr>
<td><code>animation-duration</code></td>
<td>한 번의 지속 시간</td>
</tr>
<tr>
<td><code>animation-delay</code></td>
<td>시작 전 대기</td>
</tr>
<tr>
<td><code>animation-timing-function</code></td>
<td>속도 변화</td>
</tr>
<tr>
<td><code>animation-iteration-count</code></td>
<td>반복 횟수 또는 <code>infinite</code></td>
</tr>
<tr>
<td><code>animation-direction</code></td>
<td>반복 방향</td>
</tr>
<tr>
<td><code>animation-fill-mode</code></td>
<td>시작 전·종료 후 스타일 유지 방식</td>
</tr>
<tr>
<td><code>animation-play-state</code></td>
<td>실행·일시정지</td>
</tr>
</tbody></table>
<p>사용자의 동작 감소 설정을 존중한다.</p>
<pre><code class="language-css">@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}</code></pre>
<h2 id="68-반응형-웹-디자인">6.8 반응형 웹 디자인</h2>
<p>하나의 HTML이 스마트폰, 태블릿, PC 등 다양한 화면에서 읽기 좋게 배치되도록 만드는 방식이다.</p>
<p>핵심 세 가지:</p>
<ol>
<li>viewport meta tag</li>
<li>유연한 Flex/Grid와 상대 단위</li>
<li>media query</li>
</ol>
<h3 id="viewport">Viewport</h3>
<pre><code class="language-html">&lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;</code></pre>
<ul>
<li><code>width=device-width</code>: 레이아웃 viewport를 실제 기기 폭에 맞춤</li>
<li><code>initial-scale=1.0</code>: 초기 배율을 100%로 설정</li>
</ul>
<blockquote>
<p>원본의 공백이 빠진 <code>&lt;metaname=...&gt;</code> 표기는 실행 가능한 문법이 아니다. 위 예제처럼 <code>&lt;meta name=&quot;...&quot; content=&quot;...&quot;&gt;</code>로 작성한다.</p>
</blockquote>
<h3 id="media-query">Media Query</h3>
<pre><code class="language-css">/* 기본: 모바일 1열 */
.layout {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

/* 48rem 이상: 2열 */
@media (min-width: 48rem) {
  .layout {
    grid-template-columns: 2fr 1fr;
  }
}</code></pre>
<p>레이아웃이 실제 콘텐츠 때문에 깨지는 지점을 breakpoint로 정한다. 특정 기기 이름에 지나치게 맞추기보다 콘텐츠를 기준으로 선택한다.</p>
<h3 id="mobile-first와-desktop-first">Mobile First와 Desktop First</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>기본 CSS</th>
<th>Media query</th>
</tr>
</thead>
<tbody><tr>
<td>Mobile First</td>
<td>작은 화면부터 작성</td>
<td>화면이 커질 때 <code>min-width</code></td>
</tr>
<tr>
<td>Desktop First</td>
<td>큰 화면부터 작성</td>
<td>화면이 작아질 때 <code>max-width</code></td>
</tr>
</tbody></table>
<p>Mobile First는 작은 화면에서 필요한 핵심 콘텐츠를 먼저 설계하고 점진적으로 기능과 배치를 확장하기 쉽다.</p>
<h2 id="69-css-변수custom-properties">6.9 CSS 변수(Custom Properties)</h2>
<pre><code class="language-css">:root {
  --color-primary: #2563eb;
  --color-danger: #dc2626;
  --space-md: 1rem;
  --radius-md: 0.75rem;
}

.button {
  padding: var(--space-md);
  border-radius: var(--radius-md);
  background: var(--color-primary);
}

.alert {
  border-color: var(--color-danger, red); /* 두 번째 값은 fallback */
}</code></pre>
<p>장점:</p>
<ul>
<li>반복되는 색상·간격을 한곳에서 관리한다.</li>
<li>변수 이름으로 값의 의도를 표현한다.</li>
<li>특정 영역에서 변수를 덮어 테마를 만들 수 있다.</li>
<li>JavaScript로 런타임에 값을 변경할 수 있다.</li>
</ul>
<h2 id="610-css-전처리기-scsssass">6.10 CSS 전처리기: SCSS/Sass</h2>
<p>브라우저는 SCSS/Sass를 직접 읽지 못하므로 빌드 과정에서 일반 CSS로 변환해야 한다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>CSS</th>
<th>SCSS</th>
<th>들여쓰기 문법 Sass</th>
</tr>
</thead>
<tbody><tr>
<td>브라우저 직접 실행</td>
<td>가능</td>
<td>불가</td>
<td>불가</td>
</tr>
<tr>
<td>중괄호·세미콜론</td>
<td>사용</td>
<td>사용</td>
<td>생략</td>
</tr>
<tr>
<td>추가 기능</td>
<td>표준 기능</td>
<td>변수, 중첩, mixin, 모듈</td>
<td>SCSS와 유사</td>
</tr>
</tbody></table>
<pre><code class="language-scss">$primary: #2563eb;

@mixin card {
  padding: 1rem;
  border-radius: 0.75rem;
}

.card {
  @include card;

  a {
    color: $primary;
  }
}</code></pre>
<p>CSS 자체도 변수와 nesting 등 많은 기능을 지원하므로 프로젝트 규모와 도구 환경을 보고 전처리기 도입을 결정한다.</p>
<h2 id="611-강의-미션을-합친-최종-스타일-예시">6.11 강의 미션을 합친 최종 스타일 예시</h2>
<pre><code class="language-css">:root {
  --primary: #2563eb;
  --surface: #ffffff;
  --background: #f8fafc;
  --text: #1f2937;
  --radius: 0.75rem;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: &quot;Noto Sans KR&quot;, sans-serif;
  line-height: 1.6;
  color: var(--text);
  background: var(--background);
}

.container {
  width: min(calc(100% - 2rem), 70rem);
  margin-inline: auto;
}

.page-layout {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.5rem;
}

.quick-links {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.trip-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.trip-card {
  padding: 1rem;
  border: 1px solid #dbe2ea;
  border-radius: var(--radius);
  background: var(--surface);
  box-shadow: 0 0.25rem 1rem rgb(15 23 42 / 8%);
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.trip-card:hover,
.trip-card:focus-within {
  transform: translateY(-0.25rem);
  box-shadow: 0 0.75rem 1.5rem rgb(15 23 42 / 16%);
}

.site-title {
  animation: fade-in 600ms ease-out both;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(-0.75rem); }
  to { opacity: 1; transform: translateY(0); }
}

@media (min-width: 48rem) {
  .page-layout {
    grid-template-columns: 2fr 1fr;
  }

  .quick-links {
    flex-direction: row;
    flex-wrap: wrap;
  }

  .trip-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}</code></pre>
<h3 id="6장-핵심-정리">6장 핵심 정리</h3>
<ul>
<li>한 축의 정렬은 Flexbox, 행·열의 배치는 Grid가 잘 맞는다.</li>
<li><code>justify-content</code>는 주축, <code>align-items</code>는 교차축을 기준으로 생각한다.</li>
<li>transform은 이동·회전·크기 변환, transition은 두 상태 사이의 변화이다.</li>
<li>animation은 <code>@keyframes</code>로 여러 단계를 정의한다.</li>
<li>반응형 웹의 기본은 viewport, 유연한 레이아웃, media query이다.</li>
<li>CSS 변수는 반복 값을 의미 있는 이름으로 중앙 관리한다.</li>
</ul>
<hr />
<h1 id="7-자주-틀리는-문법과-교정">7. 자주 틀리는 문법과 교정</h1>
<table>
<thead>
<tr>
<th>틀리기 쉬운 형태</th>
<th>올바른 형태</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;string&gt;</code></td>
<td><code>&lt;strong&gt;</code></td>
<td>중요성을 표현하는 HTML 태그</td>
</tr>
<tr>
<td><code>&lt;metaname=&quot;viewport&quot;...&gt;</code></td>
<td><code>&lt;meta name=&quot;viewport&quot; content=&quot;...&quot;&gt;</code></td>
<td>태그명과 속성 사이 공백 필요</td>
</tr>
<tr>
<td><code>background-attatchement</code></td>
<td><code>background-attachment</code></td>
<td>CSS 속성명 철자</td>
</tr>
<tr>
<td><code>algin-self</code></td>
<td><code>align-self</code></td>
<td>Flex/Grid 개별 정렬 속성</td>
</tr>
<tr>
<td><code>@keyframe</code></td>
<td><code>@keyframes</code></td>
<td>애니메이션 단계 선언 문법</td>
</tr>
<tr>
<td><code>/* 주석 %/</code></td>
<td><code>/* 주석 */</code></td>
<td>CSS 주석 종료 문법</td>
</tr>
<tr>
<td><code>class=&quot;trip-card“</code></td>
<td><code>class=&quot;trip-card&quot;</code></td>
<td>코드에는 일반 직선 따옴표 사용</td>
</tr>
<tr>
<td><code>&lt;submit&gt;</code> / <code>&lt;reset&gt;</code></td>
<td><code>&lt;button type=&quot;submit&quot;&gt;</code> / <code>&lt;button type=&quot;reset&quot;&gt;</code></td>
<td>해당 이름의 표준 HTML 태그는 없음</td>
</tr>
</tbody></table>
<hr />
<h1 id="8-한-장으로-보는-html·css-치트시트">8. 한 장으로 보는 HTML·CSS 치트시트</h1>
<h2 id="html">HTML</h2>
<pre><code class="language-html">&lt;!-- 구조 --&gt;
&lt;header&gt;&lt;nav&gt;...&lt;/nav&gt;&lt;/header&gt;
&lt;main&gt;&lt;section&gt;&lt;article&gt;...&lt;/article&gt;&lt;/section&gt;&lt;/main&gt;
&lt;aside&gt;...&lt;/aside&gt;
&lt;footer&gt;...&lt;/footer&gt;

&lt;!-- 텍스트 --&gt;
&lt;h1&gt;제목&lt;/h1&gt;
&lt;p&gt;문단 &lt;strong&gt;중요&lt;/strong&gt; &lt;em&gt;강조&lt;/em&gt;&lt;/p&gt;
&lt;a href=&quot;page.html&quot;&gt;링크&lt;/a&gt;

&lt;!-- 목록·표 --&gt;
&lt;ul&gt;&lt;li&gt;항목&lt;/li&gt;&lt;/ul&gt;
&lt;table&gt;&lt;tr&gt;&lt;th&gt;제목&lt;/th&gt;&lt;td&gt;값&lt;/td&gt;&lt;/tr&gt;&lt;/table&gt;

&lt;!-- 미디어 --&gt;
&lt;img src=&quot;image.jpg&quot; alt=&quot;이미지 설명&quot;&gt;
&lt;video controls&gt;&lt;source src=&quot;movie.mp4&quot; type=&quot;video/mp4&quot;&gt;&lt;/video&gt;

&lt;!-- 폼 --&gt;
&lt;label for=&quot;email&quot;&gt;이메일&lt;/label&gt;
&lt;input type=&quot;email&quot; id=&quot;email&quot; name=&quot;email&quot; required&gt;
&lt;button type=&quot;submit&quot;&gt;제출&lt;/button&gt;</code></pre>
<h2 id="css">CSS</h2>
<pre><code class="language-css">/* 선택자 */
p {}
.class-name {}
#unique-id {}
[required] {}
button:hover {}
.label::after {}

/* 박스 */
.box {
  width: 100%;
  max-width: 40rem;
  margin: 1rem auto;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 0.5rem;
}

/* Flex */
.flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

/* Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

/* 반응형 */
@media (min-width: 48rem) {
  .layout { grid-template-columns: 2fr 1fr; }
}</code></pre>
<hr />
<h1 id="9-실습-점검-체크리스트">9. 실습 점검 체크리스트</h1>
<h2 id="html-1">HTML</h2>
<ul>
<li><input disabled="" type="checkbox" /> <code>&lt;!DOCTYPE html&gt;</code>, <code>lang=&quot;ko&quot;</code>, UTF-8, viewport를 작성했다.</li>
<li><input disabled="" type="checkbox" /> 페이지마다 내용을 설명하는 <code>&lt;title&gt;</code>이 있다.</li>
<li><input disabled="" type="checkbox" /> 제목은 논리적인 <code>h1</code>~<code>h6</code> 계층을 따른다.</li>
<li><input disabled="" type="checkbox" /> 단순 배치가 아니라 의미에 맞는 시맨틱 태그를 사용했다.</li>
<li><input disabled="" type="checkbox" /> 이미지에 적절한 <code>alt</code>가 있다.</li>
<li><input disabled="" type="checkbox" /> 모든 폼 입력에 label이 연결되어 있다.</li>
<li><input disabled="" type="checkbox" /> radio 그룹의 <code>name</code>이 같다.</li>
<li><input disabled="" type="checkbox" /> 모든 버튼의 <code>type</code>이 명시되어 있다.</li>
<li><input disabled="" type="checkbox" /> 키보드 Tab으로 링크와 입력 요소를 사용할 수 있다.</li>
</ul>
<h2 id="css-1">CSS</h2>
<ul>
<li><input disabled="" type="checkbox" /> 공통 스타일을 외부 CSS 파일에 작성했다.</li>
<li><input disabled="" type="checkbox" /> <code>* { box-sizing: border-box; }</code>를 적용했다.</li>
<li><input disabled="" type="checkbox" /> class 이름이 역할을 이해할 수 있게 작성되었다.</li>
<li><input disabled="" type="checkbox" /> <code>!important</code>에 의존하지 않는다.</li>
<li><input disabled="" type="checkbox" /> 레이아웃 목적에 따라 Flex와 Grid를 구분했다.</li>
<li><input disabled="" type="checkbox" /> 고정 <code>width</code> 때문에 작은 화면에서 가로 스크롤이 생기지 않는다.</li>
<li><input disabled="" type="checkbox" /> hover뿐 아니라 <code>:focus-visible</code> 상태도 확인했다.</li>
<li><input disabled="" type="checkbox" /> 애니메이션이 너무 길거나 과하지 않다.</li>
<li><input disabled="" type="checkbox" /> 모바일과 데스크톱 크기에서 모두 확인했다.</li>
</ul>
<hr />
<h1 id="10-셀프-테스트">10. 셀프 테스트</h1>
<h2 id="문제">문제</h2>
<ol>
<li>인터넷과 웹의 차이를 한 문장으로 설명해 보자.</li>
<li>URL의 scheme, host, path, query string은 각각 무엇인가?</li>
<li>HTTP 2xx, 4xx, 5xx의 의미는 무엇인가?</li>
<li><code>id</code>와 <code>class</code>는 언제 각각 사용하는가?</li>
<li>block 요소와 inline 요소의 기본 배치 차이는 무엇인가?</li>
<li><code>&lt;label for=&quot;email&quot;&gt;</code>의 <code>for</code>는 무엇과 연결되는가?</li>
<li>form input에 <code>name</code>이 없으면 어떤 문제가 생기는가?</li>
<li><code>readonly</code>와 <code>disabled</code>의 폼 제출 차이는 무엇인가?</li>
<li>시맨틱 태그가 접근성에 도움이 되는 이유는 무엇인가?</li>
<li><code>.menu &gt; a</code>와 <code>.menu a</code>의 차이는 무엇인가?</li>
<li>specificity가 같으면 어느 CSS 규칙이 적용되는가?</li>
<li>margin과 padding의 차이는 무엇인가?</li>
<li><code>position: absolute</code>의 기준은 무엇인가?</li>
<li>Flexbox에서 주축 정렬과 교차축 정렬에 쓰는 대표 속성은 무엇인가?</li>
<li>Grid의 <code>1fr 2fr</code>은 무엇을 의미하는가?</li>
<li>transition과 animation의 차이는 무엇인가?</li>
<li>viewport meta tag가 필요한 이유는 무엇인가?</li>
<li>Mobile First에서는 주로 <code>min-width</code>와 <code>max-width</code> 중 무엇을 사용하는가?</li>
</ol>
<h2 id="짧은-정답">짧은 정답</h2>
<ol>
<li>인터넷은 연결 기반 시설이고 웹은 그 위에서 HTTP와 HTML로 정보를 공유하는 서비스이다.</li>
<li>통신 방식, 서버 주소, 자원 경로, 추가 전달 값이다.</li>
<li>성공, 클라이언트 오류, 서버 오류이다.</li>
<li><code>id</code>는 고유 요소, <code>class</code>는 반복되는 분류에 사용한다.</li>
<li>block은 새 줄과 넓은 영역, inline은 같은 줄과 콘텐츠 크기를 기본으로 한다.</li>
<li>같은 값을 가진 input의 <code>id</code>와 연결된다.</li>
<li>해당 값이 일반적인 폼 제출 데이터에 포함되지 않는다.</li>
<li>readonly 값은 보통 제출되지만 disabled 값은 제출되지 않는다.</li>
<li>보조 기술이 페이지 영역과 콘텐츠 관계를 더 정확히 파악할 수 있다.</li>
<li><code>&gt;</code>는 직계 자식만, 공백은 모든 깊이의 후손을 선택한다.</li>
<li>같은 조건이라면 뒤에 선언된 규칙이 적용된다.</li>
<li>margin은 요소 바깥 간격, padding은 콘텐츠와 border 사이의 안쪽 간격이다.</li>
<li>가장 가까운 positioned 조상이며 없으면 초기 containing block을 기준으로 한다.</li>
<li><code>justify-content</code>, <code>align-items</code>이다.</li>
<li>남은 공간을 1:2 비율로 나눈 두 열이다.</li>
<li>transition은 상태 변화 사이, animation은 <code>@keyframes</code>로 여러 단계를 정의한다.</li>
<li>모바일 브라우저가 레이아웃 폭을 실제 기기 폭에 맞추게 한다.</li>
<li><code>min-width</code>를 주로 사용한다.</li>
</ol>
<hr />
<h2 id="최종-암기-문장">최종 암기 문장</h2>
<blockquote>
<p>HTML은 <strong>의미와 구조</strong>, CSS는 <strong>표현과 배치</strong>를 담당한다. 좋은 웹 페이지는 올바른 태그로 구조를 만들고, 유지보수 가능한 선택자와 Box Model을 바탕으로 Flex/Grid 및 반응형 스타일을 적용하며, 접근성과 다양한 화면을 처음부터 함께 고려한다.</p>
</blockquote>