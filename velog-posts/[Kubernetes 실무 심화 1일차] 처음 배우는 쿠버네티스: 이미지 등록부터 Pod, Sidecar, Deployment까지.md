<h2 id="이-글의-목차">이 글의 목차</h2>
<ol>
<li>실습 환경 및 실습 이미지 등록하기</li>
<li>Kubernetes 기본 이해 및 Label Selector</li>
<li>쿠버네티스 구조 및 전체 흐름 이해</li>
<li>Init Container &amp; Sidecar 이해</li>
<li>Deployment &amp; ReplicaSet 이해, 볼륨과 PV 기초</li>
<li>조별 토론하기: 배운 내용을 문제에 적용하기</li>
</ol>
<p>처음에는 아래 문장 하나만 기억해도 좋습니다.</p>
<p><strong>쿠버네티스는 여러 컴퓨터에 앱을 실행하고, 우리가 정한 실행 상태를 유지하도록 조정하는 시스템입니다.</strong></p>
<p>모든 이름을 한 번에 외우려고 하지 마세요. 먼저 “이것이 어떤 문제를 해결하는가?”를 이해한 다음 이름을 연결하면 됩니다.</p>
<hr />
<h2 id="part-1-실습-환경-및-실습-이미지-등록하기">PART 1. 실습 환경 및 실습 이미지 등록하기</h2>
<h3 id="1-1-앱을-만든-다음에는-무엇을-해야-할까">1-1. 앱을 만든 다음에는 무엇을 해야 할까?</h3>
<p>친구에게 우리가 만든 웹사이트를 보여주고 싶다고 생각해 봅시다.</p>
<p>내 노트북에서는 잘 실행되는데 친구의 컴퓨터에는 Python이 없을 수 있습니다. Python이 있어도 버전이 다르거나 필요한 라이브러리가 없을 수 있습니다.</p>
<p>이 문제를 줄이기 위해 <strong>앱과 실행에 필요한 파일을 일정한 형태로 포장</strong>합니다. 이 포장된 결과물이 컨테이너 이미지입니다.</p>
<pre><code class="language-text">앱 소스와 필요한 파일
        ↓ 빌드
컨테이너 이미지
        ↓ 업로드
이미지 레지스트리
        ↓ 다운로드
쿠버네티스의 노드
        ↓ 실행
Pod 안의 컨테이너</code></pre>
<p>음식으로 비유하면 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>기술 용어</th>
<th>음식 비유</th>
<th>실제 의미</th>
</tr>
</thead>
<tbody><tr>
<td>소스 코드</td>
<td>요리법과 재료</td>
<td>개발자가 작성한 프로그램</td>
</tr>
<tr>
<td>Dockerfile</td>
<td>밀키트 제작 설명서</td>
<td>이미지에 무엇을 넣고 어떻게 준비할지 적은 파일</td>
</tr>
<tr>
<td>이미지</td>
<td>완성된 밀키트</td>
<td>실행할 앱과 의존 파일을 담은 패키지</td>
</tr>
<tr>
<td>컨테이너</td>
<td>밀키트로 요리하는 작업</td>
<td>이미지를 바탕으로 실행 중인 격리된 프로세스 환경</td>
</tr>
<tr>
<td>레지스트리</td>
<td>밀키트 보관·배송 창고</td>
<td>이미지를 저장하고 배포하는 서비스</td>
</tr>
<tr>
<td>Kubernetes</td>
<td>여러 주방의 운영 관리자</td>
<td>실행 위치와 개수 등을 조정하는 시스템</td>
</tr>
</tbody></table>
<p>이미지와 컨테이너는 다릅니다. 같은 이미지로 컨테이너를 여러 개 실행할 수 있습니다. 같은 밀키트 제품을 여러 주방에서 조리하는 것과 비슷합니다.</p>
<h3 id="1-2-docker와-kubernetes는-어떤-관계일까">1-2. Docker와 Kubernetes는 어떤 관계일까?</h3>
<p>Docker는 컨테이너 이미지를 만들고 로컬에서 실행하는 데 널리 쓰이는 도구입니다. Kubernetes는 여러 노드에 걸쳐 컨테이너를 운영하는 데 쓰입니다.</p>
<p>Kubernetes를 사용한다고 해서 모든 노드에서 Docker Engine을 사용해야 하는 것은 아닙니다. 실제 노드에서는 containerd나 CRI-O 같은 <strong>컨테이너 런타임</strong>이 실행을 담당할 수 있습니다.</p>
<p>런타임은 “이미지를 실제로 실행하는 담당자”라고 이해하면 됩니다. Docker로 만든 호환 이미지도 적절한 런타임에서 실행할 수 있습니다.</p>
<h3 id="1-3-실습에-필요한-도구를-역할로-구분하기">1-3. 실습에 필요한 도구를 역할로 구분하기</h3>
<table>
<thead>
<tr>
<th>준비 항목</th>
<th>역할</th>
<th>확인할 내용</th>
</tr>
</thead>
<tbody><tr>
<td>터미널</td>
<td>명령을 입력하는 창</td>
<td>현재 작업 위치와 명령 실행 가능 여부</td>
</tr>
<tr>
<td>Docker 또는 호환 빌드 도구</td>
<td>이미지 빌드</td>
<td>빌드 엔진이 실행 중인지</td>
</tr>
<tr>
<td>kubectl</td>
<td>Kubernetes API에 요청</td>
<td>설치 여부와 접속 설정</td>
</tr>
<tr>
<td>kubeconfig</td>
<td>클러스터 접속 정보</td>
<td>접속 대상, 사용자, 기본 네임스페이스</td>
</tr>
<tr>
<td>클러스터</td>
<td>앱을 실행할 서버 묶음</td>
<td>접근 가능한 대상이 있는지</td>
</tr>
<tr>
<td>네임스페이스</td>
<td>실습 자원을 나눌 공간</td>
<td>내가 사용할 공간과 권한</td>
</tr>
<tr>
<td>레지스트리</td>
<td>이미지 보관</td>
<td>업로드 권한과 노드의 다운로드 권한</td>
</tr>
</tbody></table>
<p><code>kubectl</code>은 쿠버네티스 자체가 아닙니다. 쿠버네티스에 요청하는 <strong>리모컨</strong>에 가깝습니다. 리모컨만 설치하고 TV를 연결하지 않으면 사용할 수 없듯, 접속할 클러스터와 인증 설정이 필요합니다.</p>
<p>다음 명령은 준비 상태를 살펴보는 예시입니다.</p>
<pre><code class="language-bash">docker version
kubectl version --client
kubectl config current-context
kubectl get pods -n demo</code></pre>
<p>마지막 명령은 <code>demo</code> 네임스페이스가 있고 조회 권한이 있을 때 사용합니다. 아래 예제 전체도 개인 학습 환경이나 배정받은 실습 공간에서 사용하세요. 공유 클러스터에서는 <code>demo</code> 대신 본인에게 배정된 네임스페이스를 넣습니다.</p>
<h3 id="1-4-kubeconfig와-context를-쉽게-이해하기">1-4. kubeconfig와 Context를 쉽게 이해하기</h3>
<p><code>kubeconfig</code>에는 어느 클러스터에 어떤 사용자로 접속할지에 관한 설정이 들어 있습니다. 환경에 따라 토큰·인증서·외부 인증 프로그램 설정이 포함될 수 있으므로 파일 전체를 공개하지 않습니다.</p>
<p><strong>Context는 클러스터, 사용자, 기본 네임스페이스를 묶은 접속 설정</strong>입니다.</p>
<pre><code class="language-text">Context: practice
  클러스터: 연습용 서버 묶음
  사용자: 연습용 계정
  네임스페이스: demo</code></pre>
<p>같은 <code>kubectl apply</code> 명령도 선택된 Context가 다르면 다른 클러스터에 적용될 수 있습니다. 따라서 실습 전에 접속 대상을 확인하는 습관이 중요합니다.</p>
<pre><code class="language-bash"># 현재 접속 대상 확인
kubectl config current-context

# 특정 네임스페이스에서 조회
kubectl get pods -n demo

# 필요한 권한이 있는지 확인
kubectl auth can-i create deployments -n demo</code></pre>
<p><code>-n demo</code>는 “demo 공간에서 작업하겠다”는 뜻입니다. 권한 오류가 나오면 명령을 반복하기보다 계정과 네임스페이스부터 확인합니다.</p>
<h3 id="1-5-dockerfile은-이미지를-만드는-설명서다">1-5. Dockerfile은 이미지를 만드는 설명서다</h3>
<p>작은 정적 웹사이트를 이미지로 만든다고 가정해 보겠습니다. 아래 두 파일을 같은 폴더에 둡니다.</p>
<p><code>index.html</code>:</p>
<pre><code class="language-html">&lt;!doctype html&gt;
&lt;html lang=&quot;ko&quot;&gt;
  &lt;meta charset=&quot;utf-8&quot;&gt;
  &lt;title&gt;쿠버네티스 연습&lt;/title&gt;
  &lt;h1&gt;안녕하세요! 첫 번째 컨테이너입니다.&lt;/h1&gt;
&lt;/html&gt;</code></pre>
<p><code>Dockerfile</code>:</p>
<pre><code class="language-dockerfile">FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80</code></pre>
<table>
<thead>
<tr>
<th>줄</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td><code>FROM</code></td>
<td>웹서버가 준비된 이미지를 출발점으로 사용</td>
</tr>
<tr>
<td><code>COPY</code></td>
<td>내 HTML 파일을 이미지 안으로 복사</td>
</tr>
<tr>
<td><code>EXPOSE</code></td>
<td>앱이 사용할 포트를 문서화</td>
</tr>
</tbody></table>
<p><code>EXPOSE 80</code>만으로 외부 접속이 열리지는 않습니다. 로컬 Docker에서는 <code>-p</code>, Kubernetes에서는 Service나 port-forward 등으로 연결합니다.</p>
<p>이 글의 이미지 버전은 예시를 일정하게 읽기 위한 값입니다. 운영에서는 지원 중인 버전과 보안 업데이트를 확인하고, 필요하면 이미지 digest로 실행 대상을 고정합니다.</p>
<h3 id="1-6-이미지-빌드와-로컬-확인">1-6. 이미지 빌드와 로컬 확인</h3>
<p>위 파일이 있는 폴더에서 실행합니다.</p>
<pre><code class="language-bash">docker build -t demo-web:1.0 .
docker run --rm --name demo-web-local -p 8080:80 demo-web:1.0</code></pre>
<p>첫 번째 명령을 나누어 읽어 봅시다.</p>
<ul>
<li><code>docker build</code>: 이미지를 만듭니다.</li>
<li><code>-t demo-web:1.0</code>: 이미지 이름과 태그를 붙입니다.</li>
<li><code>.</code>: 현재 폴더를 빌드에 사용할 파일 범위, 즉 빌드 컨텍스트로 지정합니다.</li>
</ul>
<p>두 번째 명령의 <code>8080:80</code>은 <strong>내 컴퓨터의 8080번 포트 → 컨테이너의 80번 포트</strong>를 뜻합니다.</p>
<p>브라우저에서 <code>http://localhost:8080</code>으로 확인합니다. 실행 중인 터미널에서 <code>Ctrl+C</code>를 누르면 컨테이너가 종료되고, 이 예제의 <code>--rm</code> 옵션에 의해 컨테이너가 제거됩니다. 이미지는 남아 있습니다.</p>
<h3 id="1-7-레지스트리에-등록하기-login-→-tag-→-push">1-7. 레지스트리에 등록하기: login → tag → push</h3>
<p>내 노트북에만 있는 이미지는 다른 서버에서 바로 사용할 수 없습니다. 노드가 다운로드할 수 있는 레지스트리에 올려야 합니다.</p>
<p>아래 <code>registry.example.com</code>은 <strong>설명용 도메인</strong>입니다. 그대로 실행되는 실제 업로드 주소가 아닙니다. 실제 작업에서는 승인된 레지스트리와 본인의 프로젝트 경로를 사용합니다.</p>
<pre><code class="language-bash">docker login registry.example.com

docker tag demo-web:1.0 registry.example.com/demo/demo-web:1.0

docker push registry.example.com/demo/demo-web:1.0</code></pre>
<p>비밀번호는 로그인 프롬프트 등 승인된 인증 방식으로 입력합니다. 비밀번호를 문서에 적거나 <code>echo</code>로 출력할 필요는 없습니다.</p>
<p>이미지 주소의 구조는 다음과 같습니다.</p>
<pre><code class="language-text">registry.example.com / demo / demo-web : 1.0
레지스트리              프로젝트  이미지명    태그</code></pre>
<p><code>tag</code>는 이미지에 추가 이름표를 붙이는 동작이고, <code>push</code>는 실제 업로드입니다. 업로드 후에는 레지스트리 화면이나 CLI에서 저장된 이미지와 태그를 확인합니다.</p>
<p>프라이빗 레지스트리라면 <strong>개발자의 업로드 인증</strong>과 <strong>Kubernetes 노드가 이미지를 내려받을 인증</strong>을 모두 고려해야 합니다. 내 컴퓨터에서 <code>docker login</code>이 성공했다고 노드에도 자동으로 같은 권한이 생기는 것은 아닙니다.</p>
<h3 id="1-8-cpu-종류가-다르면-실행이-안-될-수도-있다">1-8. CPU 종류가 다르면 실행이 안 될 수도 있다</h3>
<p>Apple Silicon 노트북은 보통 ARM 계열이고 서버는 AMD64 계열일 수 있습니다. CPU 구조에 맞지 않는 실행 파일이 들어 있으면 컨테이너가 시작하지 못할 수 있습니다.</p>
<p>다중 플랫폼 이미지를 준비하는 형식 예시는 다음과 같습니다.</p>
<pre><code class="language-bash">docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/demo/demo-web:1.0 \
  --push .</code></pre>
<p>다중 플랫폼 빌드를 지원하는 builder와 기반 이미지가 필요합니다. 핵심은 명령 암기보다 <strong>빌드 대상과 실행 대상의 CPU 구조를 확인하는 것</strong>입니다.</p>
<h3 id="1-9-spring-python-정적-웹-vue는-무엇이-다른가">1-9. Spring, Python, 정적 웹, Vue는 무엇이 다른가?</h3>
<p>이미지를 만드는 목적은 같지만 포장 과정이 달라집니다.</p>
<table>
<thead>
<tr>
<th>앱 종류</th>
<th>빌드 과정에서 준비하는 것</th>
<th>실행할 때 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>Spring Boot</td>
<td>소스를 빌드해 JAR 준비</td>
<td>Java로 JAR 실행</td>
</tr>
<tr>
<td>Python API</td>
<td>코드와 라이브러리 준비</td>
<td>Python/애플리케이션 서버 실행</td>
</tr>
<tr>
<td>정적 HTML</td>
<td>HTML·CSS·JS 복사</td>
<td>Nginx 등으로 파일 제공</td>
</tr>
<tr>
<td>Vue 프런트엔드</td>
<td>Node 도구로 정적 결과물 빌드</td>
<td>완성된 결과물을 웹서버로 제공</td>
</tr>
</tbody></table>
<p>Vue 운영 이미지는 개발 서버를 그대로 실행하기보다 빌드된 정적 파일을 제공하는 형태가 흔합니다. 이미지 빌드용 도구와 실제 실행용 도구를 분리하는 것을 <strong>멀티 스테이지 빌드</strong>라고 합니다.</p>
<p>학교 미술 시간에 비유하면 제작할 때는 가위·풀·색연필이 필요하지만, 전시할 때는 완성된 작품만 가져가는 것입니다.</p>
<p>교육용 스크립트가 이미지 이름, 버전, YAML을 자동으로 만들기도 합니다. 스크립트는 같은 명령을 반복하기 쉽게 묶어 둔 도구입니다. 실행 전에 어떤 레지스트리, 태그, 네임스페이스를 사용하도록 설정되어 있는지 확인합니다.</p>
<h3 id="1-10-이미지-등록-단계에서-자주-만나는-문제">1-10. 이미지 등록 단계에서 자주 만나는 문제</h3>
<table>
<thead>
<tr>
<th>증상</th>
<th>생각할 원인</th>
<th>먼저 확인할 것</th>
</tr>
</thead>
<tbody><tr>
<td>빌드 엔진에 연결되지 않음</td>
<td>Docker 실행 환경이 꺼져 있음</td>
<td>Docker 실행 상태</td>
</tr>
<tr>
<td><code>COPY</code> 대상이 없다는 오류</td>
<td>폴더나 빌드 컨텍스트가 다름</td>
<td>파일 위치와 <code>.dockerignore</code></td>
</tr>
<tr>
<td>업로드 권한 오류</td>
<td>프로젝트 경로나 권한 문제</td>
<td>로그인 대상과 push 권한</td>
</tr>
<tr>
<td><code>ImagePullBackOff</code></td>
<td>이미지 다운로드 실패 반복</td>
<td>주소, 태그, 인증, 네트워크, Events</td>
</tr>
<tr>
<td><code>exec format error</code></td>
<td>실행 파일·CPU 구조 문제 등</td>
<td>이미지 플랫폼과 노드 구조</td>
</tr>
<tr>
<td>새 HTML이 보이지 않음</td>
<td>캐시 또는 실제 배포 버전 차이</td>
<td>이미지 태그·digest와 브라우저 캐시</td>
</tr>
</tbody></table>
<p><strong>이 파트의 핵심:</strong> 앱을 이미지로 만들고, 실행할 서버가 가져갈 수 있는 위치에 저장합니다.</p>
<hr />
<h2 id="part-2-kubernetes-기본-이해-및-label-selector">PART 2. Kubernetes 기본 이해 및 Label Selector</h2>
<h3 id="2-1-왜-컨테이너-실행만으로는-부족할까">2-1. 왜 컨테이너 실행만으로는 부족할까?</h3>
<p>작은 웹서버 하나는 <code>docker run</code>으로 실행할 수 있습니다. 그런데 웹서버가 100개이고 여러 컴퓨터에 흩어져 있다면 어떨까요?</p>
<ul>
<li>앱이 종료되면 누가 다시 시작할까요?</li>
<li>컴퓨터가 고장 나면 어디에 대체 앱을 실행할까요?</li>
<li>실행 개수를 3개에서 5개로 늘리려면 어떻게 할까요?</li>
<li>새 버전으로 교체하는 동안 접속을 어떻게 유지할까요?</li>
</ul>
<p>이런 실행 단위를 배치하고 관리하는 일을 <strong>오케스트레이션</strong>이라고 합니다. Kubernetes는 이를 위한 도구입니다. 줄여서 <code>K8s</code>라고도 부릅니다. K와 s 사이에 글자가 8개 있다는 뜻입니다.</p>
<h3 id="2-2-명령을-시키기와-원하는-상태를-적기">2-2. “명령을 시키기”와 “원하는 상태를 적기”</h3>
<p>두 지시를 비교해 보겠습니다.</p>
<pre><code class="language-text">명령형: 앱 하나 실행해. 하나 더 실행해. 종료됐으니 다시 실행해.
선언형: 앱이 3개 실행되는 상태를 유지해 줘.</code></pre>
<p>Kubernetes는 주로 두 번째 방식을 사용합니다.</p>
<table>
<thead>
<tr>
<th>상태</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>Desired State</td>
<td>원하는 상태</td>
<td>Pod 3개</td>
</tr>
<tr>
<td>Actual State</td>
<td>실제 관찰된 상태</td>
<td>Pod 2개</td>
</tr>
<tr>
<td>Reconciliation</td>
<td>차이를 줄이는 조정</td>
<td>부족한 Pod 생성</td>
</tr>
</tbody></table>
<p>수조의 물 높이를 일정하게 유지하는 장치를 생각해 보세요. “물을 한 컵 넣어”보다 “이 선까지 물 높이를 유지해”라는 목표가 있습니다. 물이 줄면 장치가 다시 보충합니다.</p>
<p>쿠버네티스도 비슷하게 원하는 상태와 실제 상태를 반복해서 비교합니다. 다만 자원 부족이나 잘못된 이미지 때문에 원하는 상태에 도달하지 못할 수 있습니다. 선언은 목표이지 성공 보증서는 아닙니다.</p>
<h3 id="2-3-cluster-node-pod-container의-관계">2-3. Cluster, Node, Pod, Container의 관계</h3>
<pre><code class="language-text">Cluster: 함께 관리하는 서버 묶음
├─ Node A: 서버 한 대
│  ├─ Pod 1
│  │  └─ 웹 컨테이너
│  └─ Pod 2
│     └─ API 컨테이너
└─ Node B: 서버 한 대
   └─ Pod 3
      ├─ API 컨테이너
      └─ 보조 컨테이너</code></pre>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 뜻</th>
<th>학교 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Cluster</td>
<td>하나의 시스템으로 관리하는 노드 묶음</td>
<td>학교 전체</td>
</tr>
<tr>
<td>Node</td>
<td>Pod를 실행하는 물리 서버 또는 가상머신</td>
<td>교실</td>
</tr>
<tr>
<td>Pod</td>
<td>함께 배치하고 실행할 컨테이너 묶음</td>
<td>한 조의 활동 공간</td>
</tr>
<tr>
<td>Container</td>
<td>앱 프로세스의 실행 환경</td>
<td>조 안에서 특정 일을 하는 구성원</td>
</tr>
</tbody></table>
<p>여기서 Pod가 특히 중요합니다. Kubernetes가 컨테이너를 노드에 배치할 때 기본으로 다루는 단위는 <strong>Pod</strong>입니다.</p>
<h3 id="2-4-pod는-왜-필요한가">2-4. Pod는 왜 필요한가?</h3>
<p>Pod에는 컨테이너 하나가 들어갈 수도 있고, 서로 밀접하게 협력하는 컨테이너 여러 개가 들어갈 수도 있습니다.</p>
<p>같은 Pod의 컨테이너들은 네트워크 공간을 공유합니다. 따라서 한 컨테이너에서 <code>localhost</code>로 같은 Pod의 다른 컨테이너에 접근할 수 있습니다.</p>
<pre><code class="language-text">Pod
├─ API 컨테이너: 8080 포트
└─ 프록시 컨테이너: 80 포트

프록시 → http://localhost:8080 → API</code></pre>
<p>공유하는 것은 IP와 포트 공간입니다. 그래서 두 컨테이너가 같은 주소의 같은 포트를 동시에 점유하려 하면 충돌할 수 있습니다.</p>
<p>반면 <strong>파일은 자동으로 모두 공유되지 않습니다.</strong> 각 컨테이너는 자기 파일 시스템을 가지며, 같은 볼륨을 연결한 경로를 통해 파일을 공유합니다. 프로세스 공간도 기본적으로 모두 합쳐지는 것은 아닙니다. 필요한 경우 별도 설정을 사용합니다.</p>
<p>다른 Pod의 <code>localhost</code>는 그 다른 Pod 자신입니다. 다른 Pod에 접근하려면 Pod IP나 Service 주소 등을 사용합니다.</p>
<h3 id="2-5-pod는-영원한-서버가-아니다">2-5. Pod는 영원한 서버가 아니다</h3>
<p>Pod는 생성되고 사라질 수 있는 실행 단위입니다. 삭제된 Pod를 다른 서버로 그대로 이동시키는 방식이 아니라, 관리 리소스가 <strong>새 Pod를 생성</strong>해 대체하는 경우가 일반적입니다.</p>
<p>새 Pod는 UID와 IP 등이 달라질 수 있습니다. 그래서 앱 연결을 Pod IP 하나에만 의존하거나, 중요한 데이터를 컨테이너 내부에만 저장하면 문제가 생깁니다.</p>
<p>Pod 상태를 읽을 때 다음을 구분하세요.</p>
<table>
<thead>
<tr>
<th>표시·상태</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>Pending</code></td>
<td>배치·이미지 준비 등 실행 준비가 완료되지 않음</td>
</tr>
<tr>
<td><code>Running</code></td>
<td>Pod의 컨테이너가 생성되었고 하나 이상이 실행 또는 시작·재시작 중</td>
</tr>
<tr>
<td><code>Succeeded</code></td>
<td>컨테이너들이 성공 종료한 완료 상태</td>
</tr>
<tr>
<td><code>Failed</code></td>
<td>컨테이너들이 종료했고 하나 이상이 실패한 상태</td>
</tr>
<tr>
<td><code>Unknown</code></td>
<td>상태를 확인하기 어려운 상태</td>
</tr>
<tr>
<td><code>CrashLoopBackOff</code></td>
<td>실패와 재시작이 반복되어 재시도를 지연하는 컨테이너 상태 표시</td>
</tr>
<tr>
<td><code>Terminating</code></td>
<td>삭제 진행 중임을 보여 주는 CLI 표시</td>
</tr>
</tbody></table>
<p><code>CrashLoopBackOff</code>와 <code>Terminating</code>을 Pod의 공식 phase 목록과 동일하게 취급하지 않습니다.</p>
<p>또한 <code>Running</code>이라고 정상 응답을 보장하지 않습니다. <strong>실행 여부와 요청을 처리할 준비가 되었는지는 다른 질문</strong>입니다. 뒤에서 readiness와 연결해 설명합니다.</p>
<h3 id="2-6-yaml은-원하는-상태를-적는-문서다">2-6. YAML은 원하는 상태를 적는 문서다</h3>
<p>YAML은 들여쓰기로 구조를 나타내는 텍스트 형식입니다. 아래는 Pod를 선언하는 예제입니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  namespace: demo
  labels:
    app: hello
    env: practice
spec:
  containers:
    - name: web
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80</code></pre>
<table>
<thead>
<tr>
<th>항목</th>
<th>묻는 질문</th>
<th>예제의 답</th>
</tr>
</thead>
<tbody><tr>
<td><code>apiVersion</code></td>
<td>어느 API 규칙을 쓰나?</td>
<td><code>v1</code></td>
</tr>
<tr>
<td><code>kind</code></td>
<td>무엇을 만들까?</td>
<td>Pod</td>
</tr>
<tr>
<td><code>metadata</code></td>
<td>이름과 분류는 무엇인가?</td>
<td>hello-pod, demo, labels</td>
</tr>
<tr>
<td><code>spec</code></td>
<td>원하는 실행 내용은 무엇인가?</td>
<td>Nginx 컨테이너 실행</td>
</tr>
</tbody></table>
<p>이 파일을 <code>hello-pod.yaml</code>로 저장했다면 다음처럼 적용합니다.</p>
<pre><code class="language-bash">kubectl apply -f hello-pod.yaml
kubectl get pods -n demo
kubectl describe pod hello-pod -n demo
kubectl logs hello-pod -n demo</code></pre>
<p><code>get</code>은 목록, <code>describe</code>는 상세 정보와 사건 기록, <code>logs</code>는 앱의 표준 출력 로그를 확인하는 데 사용합니다.</p>
<p><code>containerPort</code>도 외부 노출을 자동으로 만들지는 않습니다. 이 예제는 Pod 구조를 읽는 연습이며, 실제 앱 운영에서는 보통 Deployment로 Pod를 관리합니다.</p>
<h3 id="2-7-namespace는-같은-클러스터-안의-구분-공간이다">2-7. Namespace는 같은 클러스터 안의 구분 공간이다</h3>
<p>같은 학교에 1반과 2반이 있듯, 하나의 클러스터에서 리소스를 논리적으로 나눌 수 있습니다. 이것이 Namespace입니다.</p>
<pre><code class="language-text">Cluster
├─ dev: 개발 환경
│  └─ web이라는 Deployment
└─ prod: 운영 환경
   └─ web이라는 Deployment</code></pre>
<p>네임스페이스가 다르면 같은 종류의 리소스가 같은 이름을 사용해도 구분할 수 있습니다.</p>
<p>Pod, Service, Deployment는 네임스페이스에 속합니다. Node, Namespace 자체, PV 같은 리소스는 클러스터 범위입니다.</p>
<pre><code class="language-bash">kubectl get namespaces
kubectl get pods -n demo
kubectl get pods -A</code></pre>
<p><code>-A</code>는 모든 네임스페이스를 대상으로 조회합니다. 계정에 권한이 있어야 사용할 수 있습니다.</p>
<p><strong>Namespace를 만들었다고 접근 권한이나 네트워크가 자동으로 완전히 격리되지는 않습니다.</strong> 권한은 RBAC, 네트워크 제한은 NetworkPolicy 같은 별도 정책과 구현을 함께 봐야 합니다.</p>
<h3 id="2-8-service-pod가-바뀌어도-사용할-연결-이름">2-8. Service: Pod가 바뀌어도 사용할 연결 이름</h3>
<p>친구가 사무실 자리만 계속 바꾸고 전화번호는 유지한다고 생각해 봅시다. 우리는 자리 번호를 외우기보다 전화번호로 연락합니다.</p>
<p>Service도 비슷합니다. 뒤에 연결된 Pod가 바뀌어도 앱이 사용할 안정적인 연결 지점을 제공합니다.</p>
<pre><code class="language-text">호출하는 앱
     ↓ hello-service
Service
     ├─ Pod A
     ├─ Pod B
     └─ Pod C</code></pre>
<p>일반적인 ClusterIP Service는 내부용 가상 IP와 DNS 이름을 제공합니다. 외부 인터넷 공개는 별도 설정이 필요합니다.</p>
<p>Service가 어떤 Pod에 연결할지는 <strong>Label과 Selector</strong>로 정합니다.</p>
<h3 id="2-9-label은-이름표-selector는-선택-조건">2-9. Label은 이름표, Selector는 선택 조건</h3>
<p>과일 바구니마다 이름표가 붙어 있다고 생각해 보세요.</p>
<pre><code class="language-text">바구니 A: fruit=apple, color=red
바구니 B: fruit=apple, color=green
바구니 C: fruit=pear,  color=green</code></pre>
<p>“사과를 고르세요”는 <code>fruit=apple</code>이고, “초록색 사과를 고르세요”는 두 조건을 동시에 만족하는 대상을 고르는 것입니다.</p>
<p>Kubernetes에서는 다음처럼 사용합니다.</p>
<pre><code class="language-yaml">metadata:
  labels:
    app: shop
    tier: frontend
    env: practice</code></pre>
<p>Label은 <code>키: 값</code> 형태의 메타데이터입니다. 여러 리소스가 같은 Label을 가질 수 있으며, 이름처럼 유일할 필요가 없습니다.</p>
<p>Selector는 Label을 읽고 대상을 고르는 조건입니다.</p>
<pre><code class="language-yaml">selector:
  app: shop
  tier: frontend</code></pre>
<p>이 조건은 <code>app=shop</code> <strong>그리고</strong> <code>tier=frontend</code>인 Pod를 선택합니다. 두 조건 중 하나만 만족하면 선택되지 않습니다.</p>
<h3 id="2-10-selector가-잘못되면-어떤-일이-생길까">2-10. Selector가 잘못되면 어떤 일이 생길까?</h3>
<pre><code class="language-text">Pod의 Label:       app=shop
Service의 조건:   app=store</code></pre>
<p>앱이 정상 실행되어도 Service가 그 Pod를 선택하지 못합니다. “웹서버는 켜졌는데 서비스로 연결이 안 된다”는 문제가 생길 수 있습니다.</p>
<p>반대로 지나치게 넓은 조건을 쓰면 의도하지 않은 Pod까지 선택할 수 있습니다. Service는 Pod가 어떤 Deployment에서 만들어졌는지보다 설정한 Label 조건을 기준으로 연결 대상을 찾습니다.</p>
<h3 id="2-11-label-조회와-변경">2-11. Label 조회와 변경</h3>
<pre><code class="language-bash"># 이름표 전체 확인
kubectl get pods -n demo --show-labels

# 특정 이름표를 표의 열로 표시
kubectl get pods -n demo -L app,env

# app=hello인 Pod 조회
kubectl get pods -n demo -l app=hello

# 두 조건을 동시에 만족하는 Pod 조회
kubectl get pods -n demo -l 'app=hello,env=practice'

# 예제 Pod에 Label 추가
kubectl label pod hello-pod -n demo team=study

# 기존 Label 변경
kubectl label pod hello-pod -n demo team=practice --overwrite

# Label 제거: 키 뒤의 -가 삭제를 뜻함
kubectl label pod hello-pod -n demo team-</code></pre>
<p>Deployment가 만든 Pod에만 수동 Label을 붙이면, 그 Pod가 교체될 때 변경 내용이 새 Pod에 반영되지 않을 수 있습니다. 계속 유지할 Label은 Deployment의 <strong>Pod 템플릿</strong>에 적습니다.</p>
<h3 id="2-12-동등-조건과-집합-조건">2-12. 동등 조건과 집합 조건</h3>
<p>CLI에서 다음처럼 여러 값 중 하나를 고를 수 있습니다.</p>
<pre><code class="language-bash">kubectl get pods -n demo -l 'env in (dev,practice)'
kubectl get pods -n demo -l 'tier notin (database)'
kubectl get pods -n demo -l 'team'</code></pre>
<p>첫 번째는 환경이 dev 또는 practice인 Pod, 세 번째는 <code>team</code> 키가 있는 Pod를 조회합니다. 여러 조건을 쉼표로 연결하면 AND입니다.</p>
<p>Deployment·ReplicaSet의 Selector에는 <code>matchLabels</code>와 <code>matchExpressions</code>를 사용할 수 있지만, <strong>Service의 <code>spec.selector</code>에는 같은 구조의 집합 표현식을 그대로 넣을 수 없습니다.</strong> 리소스마다 지원하는 Selector 형식을 확인합니다. <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/">공식 문서: Labels and Selectors</a></p>
<p><strong>이 파트의 핵심:</strong> Pod로 실행하고, Namespace로 구분하고, Label과 Selector로 필요한 대상을 연결합니다.</p>
<hr />
<h2 id="part-3-쿠버네티스-구조-및-전체-흐름-이해">PART 3. 쿠버네티스 구조 및 전체 흐름 이해</h2>
<h3 id="3-1-식당으로-이해하는-control-plane과-worker-node">3-1. 식당으로 이해하는 Control Plane과 Worker Node</h3>
<p>큰 식당에는 주문을 접수하고 일을 배정하는 담당자와 실제 요리를 하는 주방이 있습니다.</p>
<p>Kubernetes도 역할을 크게 나눌 수 있습니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>역할</th>
<th>식당 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Control Plane</td>
<td>상태 관리와 배치 결정</td>
<td>주문 접수·운영 관리실</td>
</tr>
<tr>
<td>Worker Node</td>
<td>Pod의 실제 실행</td>
<td>요리하는 주방</td>
</tr>
</tbody></table>
<pre><code class="language-text">사용자: kubectl로 요청
          ↓
Control Plane
├─ API Server: 요청 창구
├─ etcd: 클러스터 상태 저장
├─ Controller Manager: 목표 상태로 조정
└─ Scheduler: 실행할 노드 선택
          ↓ API를 통한 관찰과 갱신
Worker Node
├─ kubelet: 이 노드의 Pod 관리
├─ Container Runtime: 컨테이너 실행
└─ 네트워크 구성 요소: Pod·Service 통신 지원</code></pre>
<p>이 그림은 역할을 이해하기 위한 것입니다. 실제 시스템에서는 여러 구성 요소가 API Server를 관찰하며 독립적으로 동작합니다. 하나의 담당자가 모든 단계를 동기적으로 지시하는 구조라고 생각하지 않아도 됩니다.</p>
<h3 id="3-2-api-server-모든-요청이-만나는-창구">3-2. API Server: 모든 요청이 만나는 창구</h3>
<p><code>kubectl apply</code>를 실행하면 YAML을 바탕으로 API Server에 요청합니다.</p>
<p>API Server는 대략 다음 질문을 확인합니다.</p>
<ol>
<li>요청한 사람이 누구인가? → 인증</li>
<li>이 작업을 해도 되는가? → 인가</li>
<li>객체 형식과 정책에 맞는가? → 검증과 Admission</li>
<li>허용된 내용을 저장할 수 있는가? → 상태 저장</li>
</ol>
<p>인증과 인가는 다릅니다.</p>
<pre><code class="language-text">인증: 학생증을 확인해서 누구인지 알아보기
인가: 그 학생이 이 실험실에 들어가도 되는지 판단하기</code></pre>
<p>로그인에 성공해도 모든 네임스페이스를 수정할 수 있는 것은 아닙니다.</p>
<p>Admission은 “이 요청을 실제로 받아들일 것인가?”를 검사하는 단계입니다. Mutating Admission은 기본값이나 보조 설정을 추가할 수 있고, Validating Admission은 정책에 맞는지 허용·거부를 판단합니다. 세부 처리에는 객체 종류와 설정에 따른 차이가 있지만, 초보자는 <strong>인증·권한·형식·정책을 통과해야 저장된다</strong>는 흐름을 먼저 이해하면 됩니다.</p>
<h3 id="3-3-etcd-클러스터-상태를-기록하는-장부">3-3. etcd: 클러스터 상태를 기록하는 장부</h3>
<p>etcd에는 Kubernetes 객체와 클러스터 상태 정보가 저장됩니다.</p>
<pre><code class="language-text">이 Deployment는 어떤 이미지를 사용할까?
원하는 복제본은 몇 개일까?
이 Pod는 어느 노드에 배정되었을까?
어떤 Service가 정의되어 있을까?</code></pre>
<p>etcd는 앱의 상품 주문 DB나 파일 저장소를 대신하는 것이 아닙니다. <strong>Kubernetes를 운영하기 위한 상태 저장소</strong>입니다.</p>
<p>다른 구성 요소가 보통 etcd에 직접 연결해 객체를 수정하는 것도 아닙니다. Controller, Scheduler, kubelet 등은 API Server를 통해 필요한 상태를 관찰하고 갱신합니다.</p>
<h3 id="3-4-controller-manager-계속-확인하는-관리자">3-4. Controller Manager: 계속 확인하는 관리자</h3>
<p>Controller는 원하는 상태와 실제 상태를 비교하여 차이를 줄이는 조정 프로그램입니다. Controller Manager는 여러 Controller를 실행합니다.</p>
<p>예를 들어 ReplicaSet Controller는 “Pod 3개가 필요하다”는 목표를 확인하고 부족하면 Pod 객체를 만듭니다.</p>
<pre><code class="language-text">확인 → 비교 → 필요한 변경 → 다시 확인</code></pre>
<p>이 반복을 <strong>조정 루프</strong>라고 부릅니다. 로봇 청소기가 한 번만 바닥을 보는 대신 계속 주변을 확인하는 것처럼, 상태를 반복해서 관찰합니다.</p>
<p>Deployment Controller, ReplicaSet Controller, Node Controller 등은 각자 담당할 대상과 규칙이 다릅니다.</p>
<h3 id="3-5-scheduler-어느-노드에서-실행할까">3-5. Scheduler: 어느 노드에서 실행할까?</h3>
<p>새 Pod가 만들어져도 실행 노드가 아직 정해지지 않았을 수 있습니다. Scheduler는 이런 Pod를 보고 적절한 노드를 선택합니다.</p>
<p>대표적으로 다음을 고려합니다.</p>
<ul>
<li>Pod가 요청한 CPU·메모리를 배치할 여유가 있는가?</li>
<li>필요한 노드 조건을 만족하는가?</li>
<li>특정 노드를 피하거나 선호해야 하는가?</li>
<li>Pod들이 특정 노드에 너무 몰리지 않도록 해야 하는가?</li>
</ul>
<p>학생 30명을 교실에 배정할 때 빈자리 수와 수업 조건을 함께 보는 것과 비슷합니다.</p>
<table>
<thead>
<tr>
<th>설정·용어</th>
<th>쉬운 의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>requests</code></td>
<td>배치 판단 등에 사용할 자원 요청량</td>
</tr>
<tr>
<td>nodeSelector</td>
<td>특정 Label이 있는 노드에 배치</td>
</tr>
<tr>
<td>affinity</td>
<td>같이 또는 특정 위치에 배치할 조건·선호</td>
</tr>
<tr>
<td>taint / toleration</td>
<td>노드의 배치 제한과 그 제한을 허용하는 Pod 조건</td>
</tr>
<tr>
<td>topology spread</td>
<td>노드·영역에 고르게 나누어 배치하도록 설정</td>
</tr>
</tbody></table>
<p>Affinity에는 필수 조건도 선호 조건도 있습니다. 모두 “가능하면” 수준의 약한 조건인 것은 아닙니다.</p>
<p>Scheduler는 노드를 결정합니다. <strong>이미지를 직접 다운로드하거나 컨테이너를 실행하지는 않습니다.</strong> 또한 일반적인 스케줄링은 단순히 현재 CPU 사용률만 보고 빈 노드를 고르는 방식이 아닙니다.</p>
<h3 id="3-6-kubelet-각-노드의-현장-관리자">3-6. kubelet: 각 노드의 현장 관리자</h3>
<p>kubelet은 노드에서 실행되며, 그 노드에 배정된 Pod가 명세대로 실행되도록 관리합니다.</p>
<pre><code class="language-text">API에서 내 노드에 배정된 Pod를 확인
         ↓
네트워크·볼륨·컨테이너 실행 준비를 조율
         ↓
런타임에 실행 요청
         ↓
상태를 확인하고 API에 보고</code></pre>
<p>컨테이너 재시작과 건강 상태 확인에도 관여합니다. 전체 클러스터의 목표 복제본 수를 혼자 관리하는 역할은 아닙니다.</p>
<h3 id="3-7-containerd-cri-runc를-이름보다-역할로-이해하기">3-7. containerd, CRI, runc를 이름보다 역할로 이해하기</h3>
<p>낯선 이름을 다음처럼 나누면 쉽습니다.</p>
<table>
<thead>
<tr>
<th>이름</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>CRI</td>
<td>kubelet과 런타임 사이의 표준 요청 규칙</td>
</tr>
<tr>
<td>containerd / CRI-O</td>
<td>이미지와 컨테이너 실행을 관리하는 런타임</td>
</tr>
<tr>
<td>runc 등</td>
<td>Linux에서 컨테이너 프로세스의 실제 생성·실행을 맡는 저수준 도구</td>
</tr>
<tr>
<td>shim</td>
<td>런타임과 실행 중인 프로세스 사이의 관리 연결을 돕는 구성 요소</td>
</tr>
</tbody></table>
<pre><code class="language-text">kubelet
   ↓ CRI 규칙으로 요청
containerd 또는 CRI-O
   ↓ 저수준 실행 도구 사용
컨테이너의 앱 프로세스</code></pre>
<p>CRI는 제품 이름이 아니라 <strong>대화 규칙</strong>입니다. USB 규격을 여러 회사 제품이 지원하듯, 런타임도 같은 인터페이스를 구현해 Kubernetes와 연결됩니다.</p>
<h3 id="3-8-cni와-pod-sandbox">3-8. CNI와 Pod Sandbox</h3>
<p>CNI는 컨테이너 네트워크를 구성할 때 사용하는 인터페이스입니다. CNI 플러그인은 구현에 따라 Pod IP 할당, 인터페이스 연결, 라우팅 등의 작업을 수행합니다.</p>
<p>Pod Sandbox는 컨테이너들이 함께 사용할 실행 환경을 준비하는 개념입니다. Linux의 흔한 구현에서는 작은 pause 컨테이너가 네트워크 공간 등을 유지하는 데 사용됩니다.</p>
<p>pause 컨테이너를 “웹 요청을 처리하는 앱”이라고 생각할 필요는 없습니다. 같은 Pod의 컨테이너들이 공유할 공간을 유지하는 기반 역할에 가깝습니다.</p>
<p>Pod들이 서로 통신할 수 있다는 말은 모두 같은 물리 서브넷에 있다는 뜻도, NetworkPolicy가 필요 없다는 뜻도 아닙니다. 실제 연결 방식은 CNI와 클러스터 네트워크 구성에 따라 달라집니다.</p>
<h3 id="3-9-kube-proxy-coredns-cloud-controller-manager">3-9. kube-proxy, CoreDNS, cloud-controller-manager</h3>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>CoreDNS</td>
<td>Service 이름 등의 DNS 조회</td>
<td>이름으로 전화번호 찾는 안내 서비스</td>
</tr>
<tr>
<td>kube-proxy</td>
<td>Service 트래픽 전달을 위한 노드의 규칙 구성</td>
<td>안내표와 길 설정 담당</td>
</tr>
<tr>
<td>cloud-controller-manager</td>
<td>Kubernetes와 클라우드 자원 사이의 연동</td>
<td>외부 시설 업체와 연락하는 담당자</td>
</tr>
</tbody></table>
<p>kube-proxy는 구성에 따라 커널의 네트워크 규칙을 설정합니다. 매 요청이 kube-proxy 프로세스를 직접 통과해야 한다고 이해할 필요는 없습니다. 일부 네트워크 구현은 Service 전달 기능을 다른 방식으로 제공하기도 합니다.</p>
<p>클라우드 연동도 환경에 따라 다릅니다. 예를 들어 클라우드의 노드 정보나 로드밸런서 자원을 연결할 수 있지만, 모든 클러스터가 동일한 클라우드 구성을 쓰는 것은 아닙니다.</p>
<p>구성 요소의 큰 역할은 <a href="https://kubernetes.io/docs/concepts/overview/components/">Kubernetes Components</a>에서 확인할 수 있습니다.</p>
<h3 id="3-10-kubectl-apply-한-줄이-앱-실행으로-이어지는-과정">3-10. <code>kubectl apply</code> 한 줄이 앱 실행으로 이어지는 과정</h3>
<p>이제 “웹 앱을 3개 실행해 주세요”라는 Deployment를 제출한다고 생각해 봅시다.</p>
<pre><code class="language-text">① kubectl이 Deployment 요청을 API Server에 전달
② 요청 검증 후 Deployment 객체 저장
③ Deployment Controller가 필요한 ReplicaSet 객체 생성
④ ReplicaSet Controller가 부족한 Pod 객체 생성
⑤ Scheduler가 각 Pod를 실행할 노드에 배정
⑥ 노드의 kubelet이 런타임과 실행 준비를 조율
⑦ 런타임이 이미지를 확보하고 컨테이너 실행
⑧ 상태를 API에 보고하고 Controller들이 계속 관찰</code></pre>
<p>이 단계는 이해를 위한 논리 순서입니다. 실제로는 비동기 관찰과 재시도가 포함됩니다. <code>apply</code>가 성공했다는 것은 요청한 객체가 반영되었다는 뜻이지, 이미 앱 3개가 정상 응답한다는 뜻은 아닙니다.</p>
<h3 id="3-11-장애-두-가지를-구분해야-한다">3-11. 장애 두 가지를 구분해야 한다</h3>
<p><strong>상황 A: 컨테이너의 앱 프로세스가 종료되었습니다.</strong></p>
<p>Deployment의 Pod에서는 일반적으로 kubelet이 재시작 정책에 따라 같은 Pod 안의 컨테이너를 다시 시작합니다. <code>RESTARTS</code>가 증가할 수 있습니다.</p>
<p><strong>상황 B: Pod 자체가 삭제되었거나 사용할 수 없게 되었습니다.</strong></p>
<p>관리 Controller가 부족한 개수를 채우려고 새 Pod를 생성합니다. 새 Pod는 다른 이름이나 IP를 가질 수 있습니다.</p>
<pre><code class="language-text">프로세스 종료 → 같은 Pod 안의 컨테이너 재시작 가능
Pod 삭제     → 새 Pod 생성으로 복제본 수 회복</code></pre>
<p>노드 장애는 감지와 판정, 스케줄링, 이미지·볼륨 준비 시간이 걸립니다. 자동 복구를 “즉시 복구”라고 이해하지 않습니다.</p>
<h3 id="3-12-service-요청은-어떻게-앱까지-갈까">3-12. Service 요청은 어떻게 앱까지 갈까?</h3>
<p>Service 이름을 사용하는 앱의 논리적인 통신 과정은 다음과 같습니다.</p>
<pre><code class="language-text">호출 앱이 http://hello-service:80 요청
       ↓
DNS가 Service의 주소를 알려 줌
       ↓
Service 전달 규칙이 대상 엔드포인트로 연결
       ↓
선택된 Pod의 앱 포트로 요청 도착</code></pre>
<p>여기서 <code>port</code>는 Service에 접근하는 포트이고, <code>targetPort</code>는 뒤의 Pod 앱에 연결할 포트입니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: demo
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 80</code></pre>
<p>이 예제는 앞의 <code>hello-pod</code>와 Label을 맞췄습니다. Pod가 실제로 Ready인지와 정책·네트워크 조건도 확인해야 합니다.</p>
<p>위 Service YAML을 <code>hello-service.yaml</code>로 저장하고 <code>kubectl apply -f hello-service.yaml</code>로 적용한 뒤 아래 명령으로 확인합니다.</p>
<p>일반적인 Selector 기반 Service에서는 Controller가 연결 대상 정보를 EndpointSlice에 반영합니다. EndpointSlice는 “이 Service의 실제 목적지는 어디인가?”를 담는 목록이라고 이해하면 됩니다. 교재나 오래된 자료에서 보이는 Endpoints와 역할이 연결되지만, 현재 상태 확인에서는 EndpointSlice도 함께 봅니다.</p>
<pre><code class="language-bash">kubectl get service hello-service -n demo
kubectl get pods -n demo -l app=hello --show-labels
kubectl get endpointslice -n demo \
  -l kubernetes.io/service-name=hello-service</code></pre>
<p>Service 주소와 Pod 주소를 연결하는 방식은 구현에 따라 다릅니다. 위 그림은 항상 물리적인 중간 프록시 서버 한 대를 거친다는 뜻이 아닙니다.</p>
<h3 id="3-13-상태-확인은-작은-것부터-시작한다">3-13. 상태 확인은 작은 것부터 시작한다</h3>
<pre><code class="language-bash">kubectl get deployment,replicaset,pod,service -n demo
kubectl get pods -n demo -o wide
kubectl describe pod hello-pod -n demo
kubectl logs hello-pod -n demo -c web
kubectl get events -n demo --sort-by=.metadata.creationTimestamp</code></pre>
<p>운영 현장에서는 노드의 kubelet·containerd 프로세스를 조사하기도 합니다. 하지만 처음에는 Pod의 상태, 로그, Events만으로도 많은 문제를 찾을 수 있습니다.</p>
<p>호스트 파일 시스템이나 프로세스 공간을 노출하는 진단 Pod는 권한과 접근 범위가 크므로, 여기서는 호스트 접근 설정을 복사하지 않습니다. <strong>문제가 발생한 계층을 찾아 그 계층의 상태를 확인한다</strong>는 원리를 익힙니다.</p>
<p><strong>이 파트의 핵심:</strong> API가 요청을 받고, Controller가 상태를 조정하고, Scheduler가 위치를 정하고, kubelet과 런타임이 실행을 맡습니다.</p>
<hr />
<h2 id="part-4-init-container--sidecar-이해">PART 4. Init Container &amp; Sidecar 이해</h2>
<h3 id="4-1-먼저-컨테이너의-실행-명령을-이해하자">4-1. 먼저 컨테이너의 실행 명령을 이해하자</h3>
<p>컨테이너는 “이미지 파일을 켜는 것”이 아니라 이미지에 준비된 <strong>프로그램을 실행하는 것</strong>입니다.</p>
<p>Dockerfile에서 정한 기본 실행 명령을 그대로 사용할 수도 있고, Kubernetes에서 바꿀 수도 있습니다.</p>
<table>
<thead>
<tr>
<th>Dockerfile의 설정</th>
<th>Kubernetes에서 대응하는 설정</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>ENTRYPOINT</code></td>
<td><code>command</code></td>
<td>어떤 프로그램을 실행할지</td>
</tr>
<tr>
<td><code>CMD</code></td>
<td><code>args</code></td>
<td>기본 실행 인자</td>
</tr>
</tbody></table>
<p>가령 <code>python3 app.py --port 8080</code>은 Python이라는 프로그램에 파일과 옵션을 전달합니다.</p>
<p>아래는 <strong>컨테이너 설정의 일부</strong>입니다. 전체 Pod YAML은 아닙니다.</p>
<pre><code class="language-yaml">containers:
  - name: api
    image: registry.example.com/demo/api:1.0
    command: [&quot;python3&quot;]
    args: [&quot;app.py&quot;, &quot;--port&quot;, &quot;8080&quot;]</code></pre>
<p>이미지 안에 Python과 <code>app.py</code>가 있어야 합니다. 이 주소는 설명용이므로 실제 이미지로 바꿔야 실행할 수 있습니다.</p>
<p><code>command</code>를 생략하고 <code>args</code>만 지정하면 이미지의 기본 ENTRYPOINT에 실행 인자를 전달하는 형태로 사용할 수 있습니다. 둘 다 생략하면 이미지의 기본값을 사용합니다.</p>
<h3 id="4-2-환경-변수는-실행할-때-건네는-설정이다">4-2. 환경 변수는 실행할 때 건네는 설정이다</h3>
<p>학교 행사 진행자에게 “오늘 행사는 운동회야”라는 메모를 전달하는 것처럼, 프로그램에 실행 설정을 전달할 수 있습니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: message-demo
  namespace: demo
spec:
  restartPolicy: Never
  containers:
    - name: printer
      image: busybox:1.37
      command: [&quot;/bin/echo&quot;]
      args: [&quot;$(GREETING)&quot;]
      env:
        - name: GREETING
          value: &quot;쿠버네티스 공부를 시작합니다!&quot;</code></pre>
<p>이 예제는 메시지를 출력한 뒤 종료합니다. <code>restartPolicy: Never</code>이므로 성공 종료 후 계속 재시작하지 않습니다.</p>
<p>Kubernetes의 <code>command</code>·<code>args</code>에서는 <code>$(GREETING)</code> 형태로 환경 변수 값을 참조할 수 있습니다. 셸을 명시적으로 실행하는 경우에는 셸의 <code>$GREETING</code> 문법도 사용할 수 있습니다.</p>
<pre><code class="language-yaml">command: [&quot;sh&quot;, &quot;-c&quot;]
args:
  - 'echo &quot;$GREETING&quot;'</code></pre>
<p>즉, 환경 변수 사용을 위해 항상 셸을 실행해야 하는 것은 아닙니다. 다만 <code>&amp;&amp;</code>, 파이프, 파일로 출력하는 <code>&gt;</code> 같은 셸 문법을 사용하려면 셸 실행을 명시해야 합니다. <a href="https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/">공식 문서: 컨테이너 명령과 인자</a></p>
<h3 id="4-3-pid-1과-exec-종료-신호가-앱에-도착해야-한다">4-3. PID 1과 <code>exec</code>: 종료 신호가 앱에 도착해야 한다</h3>
<p>PID는 프로세스 번호입니다. 일반적인 컨테이너의 프로세스 공간에서 첫 번째 프로세스는 PID 1입니다.</p>
<p>컨테이너를 종료할 때 런타임은 주 프로세스에 종료 신호를 보냅니다. 그런데 아래처럼 셸을 앞에 두면 셸이 먼저 신호를 받고 실제 앱은 그 자식 프로세스가 됩니다.</p>
<pre><code class="language-text">PID 1: sh
└─ 자식 프로세스: 앱</code></pre>
<p>셸이 신호를 전달하지 않으면 앱이 정상 종료를 준비할 기회를 놓칠 수 있습니다. 셸 스크립트의 마지막에서 <code>exec</code>로 앱을 실행하면 셸을 앱 프로세스로 교체할 수 있습니다.</p>
<pre><code class="language-yaml">command: [&quot;sh&quot;, &quot;-c&quot;]
args:
  - |
    echo &quot;앱을 시작합니다.&quot;
    exec python3 app.py</code></pre>
<p><code>exec</code>는 모든 종료 문제를 자동 해결하는 마법이 아닙니다. 앱 자체도 종료 신호 처리와 작업 마무리를 지원해야 합니다. 이미지의 진입 스크립트나 init 프로세스 구성에 따라서도 동작이 달라질 수 있습니다.</p>
<h3 id="4-4-init-container-문-열기-전에-준비하는-담당자">4-4. Init Container: 문 열기 전에 준비하는 담당자</h3>
<p>카페가 문을 열기 전에는 해야 할 일이 있습니다.</p>
<pre><code class="language-text">바닥 청소 → 재료 준비 → 커피 머신 점검 → 영업 시작</code></pre>
<p>Init Container는 이런 <strong>사전 준비 작업</strong>을 담당합니다. 여기서 설명하는 것은 성공 종료 후 다음 단계로 넘어가는 일반적인 Init Container입니다.</p>
<pre><code class="language-text">Init Container 1 성공 종료
           ↓
Init Container 2 성공 종료
           ↓
일반 앱 컨테이너 시작</code></pre>
<p>순서가 있는 여러 Init Container는 앞의 작업이 성공해야 다음 작업이 시작됩니다. 실패하면 Pod의 재시작 정책에 따라 재시도하거나 실패 상태에 머무를 수 있습니다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/init-containers/">공식 문서: Init Containers</a></p>
<h3 id="4-5-언제-init-container를-사용할까">4-5. 언제 Init Container를 사용할까?</h3>
<table>
<thead>
<tr>
<th>상황</th>
<th>사전 준비 예시</th>
</tr>
</thead>
<tbody><tr>
<td>실행할 파일이 필요함</td>
<td>파일 다운로드·압축 해제</td>
</tr>
<tr>
<td>공유 디렉터리 준비</td>
<td>기본 파일 생성, 권한 점검</td>
</tr>
<tr>
<td>외부 서비스에 의존함</td>
<td>필요한 주소의 응답 여부 확인</td>
</tr>
<tr>
<td>설정 조합이 필요함</td>
<td>공개된 설정 값을 합쳐 실행 파일 작성</td>
</tr>
</tbody></table>
<p>예를 들어 메인 웹서버에는 파일을 제공하는 기능만 두고, 준비용 컨테이너에는 파일 생성 도구를 둘 수 있습니다. 역할을 나누면 메인 이미지에 모든 준비 도구를 넣을 필요가 줄어듭니다.</p>
<p>DB 마이그레이션처럼 중요한 변경을 모든 Pod의 Init Container에서 무조건 실행하는 것은 주의해야 합니다. 여러 Pod가 동시에 같은 변경을 시도할 수 있으므로 별도 실행 단계, 잠금, 중복 실행 안전성 등을 설계합니다.</p>
<h3 id="4-6-예제-웹서버를-시작하기-전에-첫-페이지-만들기">4-6. 예제: 웹서버를 시작하기 전에 첫 페이지 만들기</h3>
<p>다음 파일을 <code>init-demo.yaml</code>로 저장합니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: init-demo
  namespace: demo
spec:
  initContainers:
    - name: prepare-page
      image: busybox:1.37
      command: [&quot;sh&quot;, &quot;-c&quot;]
      args:
        - |
          echo '&lt;h1&gt;준비가 끝났습니다!&lt;/h1&gt;' &gt; /work/index.html
      volumeMounts:
        - name: page-data
          mountPath: /work
  containers:
    - name: web
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - name: page-data
          mountPath: /usr/share/nginx/html
  volumes:
    - name: page-data
      emptyDir: {}</code></pre>
<p>이 예제의 핵심은 세 가지입니다.</p>
<ol>
<li><code>prepare-page</code>가 먼저 파일을 만듭니다.</li>
<li>성공 종료 후 Nginx가 시작됩니다.</li>
<li>두 컨테이너가 같은 <code>page-data</code> 볼륨을 서로 다른 경로에 연결합니다.</li>
</ol>
<pre><code class="language-text">prepare-page의 /work/index.html
                    ↕ 같은 볼륨
web의 /usr/share/nginx/html/index.html</code></pre>
<p>경로 이름이 같아야 공유되는 것이 아닙니다. <strong>같은 볼륨을 연결했기 때문에</strong> 같은 파일을 봅니다.</p>
<pre><code class="language-bash">kubectl apply -f init-demo.yaml
kubectl get pod init-demo -n demo -w</code></pre>
<p><code>-w</code>는 변화를 계속 관찰합니다. 관찰을 끝낼 때 <code>Ctrl+C</code>를 누릅니다. 작업이 빠르면 Init 진행 표시를 놓칠 수 있습니다. 다음 명령으로 완료 후에도 Init 로그를 확인할 수 있습니다.</p>
<pre><code class="language-bash">kubectl logs init-demo -n demo -c prepare-page
kubectl describe pod init-demo -n demo
kubectl port-forward pod/init-demo -n demo 8082:80</code></pre>
<p>마지막 명령이 실행 중일 때 브라우저에서 <code>http://localhost:8082</code>로 확인합니다. 이 Init 예제는 <code>echo</code> 결과를 파일로 저장하므로 로그 출력이 비어 있어도 정상일 수 있습니다.</p>
<h3 id="4-7-파일이-생길-때까지-기다리기는-왜-쓰는가">4-7. “파일이 생길 때까지 기다리기”는 왜 쓰는가?</h3>
<p>준비가 끝났다는 신호로 특정 파일을 사용할 수 있습니다.</p>
<pre><code class="language-sh">while [ ! -f /work/ready.flag ]; do
  echo &quot;준비 완료 신호를 기다립니다.&quot;
  sleep 2
done</code></pre>
<p>이 코드는 파일이 없으면 기다리고, 파일이 생기면 반복을 끝냅니다. 교실 앞에 “입장 가능” 표지판이 생길 때까지 기다리는 것과 같습니다.</p>
<p>하지만 누가 그 파일을 만들지 반드시 정해야 합니다. <strong>아직 시작하지 않은 메인 컨테이너가 파일을 만들도록 하면 서로 기다리는 상태가 됩니다.</strong> Init이 끝나야 메인이 시작되기 때문입니다.</p>
<p>또한 Init Container에 일정 시간이 지나면 무조건 종료되는 기본 시간 제한이 있다고 생각하면 안 됩니다. 필요하면 작업 스크립트에 timeout 등을 설계합니다. Pod 전체 실행 기한을 설정하면 초기화 이후의 앱 실행에도 영향을 줄 수 있습니다.</p>
<h3 id="4-8-init-완료와-readiness는-서로-다른-문이다">4-8. Init 완료와 Readiness는 서로 다른 문이다</h3>
<pre><code class="language-text">Init 완료: 시작 전에 해야 할 준비가 끝났는가?
Readiness: 실행 중인 앱이 지금 요청을 처리할 준비가 되었는가?
Liveness: 실행 중인 앱을 재시작해야 할 정도로 문제가 있는가?</code></pre>
<p>식당으로 비유하면 다음과 같습니다.</p>
<ul>
<li>Init: 주방 준비가 끝났습니다.</li>
<li>Readiness: 지금 주문을 받을 수 있습니다.</li>
<li>Liveness: 주방 시스템이 멈춰 재시작이 필요합니다.</li>
</ul>
<p>Readiness가 실패한다고 곧바로 컨테이너가 재시작되는 것은 아닙니다. 보통 Service에서 그 Pod로 요청을 보내는 대상 선정에 영향을 줍니다. 일반적인 Init Container는 readiness·liveness·startup probe로 운영하는 장기 실행 앱과는 다릅니다.</p>
<h3 id="4-9-sidecar-앱-옆에서-함께-일하는-보조-컨테이너">4-9. Sidecar: 앱 옆에서 함께 일하는 보조 컨테이너</h3>
<p>오토바이 옆에 붙는 작은 좌석을 사이드카라고 부릅니다. Kubernetes에서도 주 앱 옆에서 보조 기능을 제공하는 컨테이너를 Sidecar라고 부릅니다.</p>
<pre><code class="language-text">Pod
├─ Main: 주문을 처리하는 API
└─ Sidecar: 로그 수집 또는 프록시 기능</code></pre>
<table>
<thead>
<tr>
<th>주 앱</th>
<th>Sidecar의 역할</th>
</tr>
</thead>
<tbody><tr>
<td>웹 API</td>
<td>앞에서 요청을 받는 프록시</td>
</tr>
<tr>
<td>업무 앱</td>
<td>로그 전달</td>
</tr>
<tr>
<td>서비스</td>
<td>모니터링 데이터 수집</td>
</tr>
<tr>
<td>서비스</td>
<td>서비스 간 통신 보안·제어</td>
</tr>
</tbody></table>
<p>Init은 준비를 마치고 종료하지만, Sidecar는 앱 옆에서 계속 일하는 것이 일반적입니다.</p>
<h3 id="4-10-왜-모든-앱을-하나의-pod에-넣지-않을까">4-10. 왜 모든 앱을 하나의 Pod에 넣지 않을까?</h3>
<p>주문 API와 결제 API를 같은 Pod에 넣었다고 해 봅시다. 주문만 5개, 결제는 2개 실행하고 싶어도 Pod 단위 복제에서는 둘이 함께 늘어납니다.</p>
<pre><code class="language-text">Pod를 3개 복제
  → 주 앱도 3개
  → 보조 컨테이너도 3개</code></pre>
<p>독립적인 배포·확장이 필요한 앱은 보통 별도 Pod와 Deployment로 나눕니다. 같은 Pod에는 <strong>같은 위치에서 같은 수명으로 밀접하게 협력할 이유가 있는 기능</strong>을 묶습니다.</p>
<p>같은 Pod에 있다는 이유로 모든 컨테이너가 항상 동시에 시작하고 동시에 재시작되는 것은 아닙니다. 개별 컨테이너는 독립적으로 재시작될 수 있고, Pod 삭제는 그 Pod의 컨테이너 전체에 영향을 줍니다.</p>
<h3 id="4-11-실무-예시-nginx가-앞에서-요청을-받기">4-11. 실무 예시: Nginx가 앞에서 요청을 받기</h3>
<pre><code class="language-text">클라이언트
    ↓ Pod의 80번 포트
Nginx Sidecar
    ↓ localhost:8080
주 앱</code></pre>
<p>이때 Nginx는 <strong>리버스 프록시</strong>입니다. 클라이언트의 요청을 받아 뒤의 앱에 전달하고 응답을 돌려줍니다. 안내 직원이 손님의 문의를 담당 부서에 연결하는 것과 같습니다.</p>
<p>실무에서는 Spring Boot 같은 앱 앞에 Nginx를 둘 수 있습니다. 다음은 구조를 작은 예제로 확인하도록 메인 앱을 BusyBox의 간단한 웹서버로 바꾼 예제입니다.</p>
<p>아래 내용을 <code>sidecar-demo.yaml</code>로 저장합니다. <code>---</code>는 한 파일 안에서 Kubernetes 객체를 나누는 표시입니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: sidecar-proxy-config
  namespace: demo
data:
  default.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://127.0.0.1:8080;
        add_header X-Learning-Proxy &quot;sidecar-demo&quot; always;
      }
    }
---
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
  namespace: demo
  labels:
    app: sidecar-demo
spec:
  containers:
    - name: main
      image: busybox:1.37
      command: [&quot;sh&quot;, &quot;-c&quot;]
      args:
        - |
          mkdir -p /www
          echo 'Hello from the main app' &gt; /www/index.html
          exec httpd -f -p 8080 -h /www
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /
          port: 8080
    - name: proxy
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
      readinessProbe:
        httpGet:
          path: /
          port: 80
      volumeMounts:
        - name: proxy-config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
  volumes:
    - name: proxy-config
      configMap:
        name: sidecar-proxy-config</code></pre>
<p>여기서 ConfigMap은 Nginx 설정 텍스트를 담는 상자입니다. ConfigMap 전체 기능은 2일차 주제지만, 이 예제에서는 “설정 파일을 컨테이너에 연결한다”는 역할만 이해하면 됩니다.</p>
<p>메인 앱은 8080, 프록시는 80을 사용합니다. 같은 Pod에서 포트 공간을 공유하므로 포트를 나눈 것입니다. Nginx는 <code>127.0.0.1:8080</code>으로 메인 앱에 요청합니다.</p>
<p><code>subPath</code>는 ConfigMap의 특정 파일 하나를 연결합니다. 이 방식의 마운트는 ConfigMap 변경이 자동으로 파일에 반영되지 않으므로, 수정한 설정을 사용하려면 Pod 재생성 등 반영 절차를 고려합니다.</p>
<h3 id="4-12-sidecar를-실제로-통과했는지-확인하기">4-12. Sidecar를 실제로 통과했는지 확인하기</h3>
<pre><code class="language-bash">kubectl apply -f sidecar-demo.yaml
kubectl get pod sidecar-demo -n demo
kubectl logs sidecar-demo -n demo -c main
kubectl logs sidecar-demo -n demo -c proxy
kubectl port-forward pod/sidecar-demo -n demo 8083:80</code></pre>
<p>다른 터미널에서 요청합니다.</p>
<pre><code class="language-bash">curl -i http://localhost:8083/</code></pre>
<p>정상 동작하면 본문에는 메인 앱의 메시지가, 헤더에는 <code>X-Learning-Proxy: sidecar-demo</code>가 나타납니다. 이것으로 <strong>본문은 메인 앱이 만들고 응답은 프록시를 통과했다</strong>는 관계를 확인합니다.</p>
<p>두 일반 컨테이너의 시작 순서는 보장하지 않았습니다. 처음에는 프록시가 켜졌지만 메인이 준비되지 않아 연결 오류가 생길 수 있습니다. 그래서 실행 상태만 보지 않고 두 컨테이너의 readiness도 확인합니다.</p>
<p>여러 컨테이너가 있는 Pod의 로그나 내부 명령은 <code>-c</code>로 대상을 지정합니다.</p>
<pre><code class="language-bash">kubectl exec sidecar-demo -n demo -c main -- \
  wget -qO- http://127.0.0.1:8080/</code></pre>
<h3 id="4-13-service-mesh와-sidecar의-관계">4-13. Service Mesh와 Sidecar의 관계</h3>
<p>앱이 여러 개로 나뉘면 서비스 사이의 통신에도 공통 기능이 필요합니다.</p>
<pre><code class="language-text">주문 API → 결제 API → 재고 API</code></pre>
<p>통신 암호화, 요청 추적, 재시도, 트래픽 분배 등을 각 앱 코드에 반복해서 작성하는 대신, 프록시와 관리 계층으로 처리하는 구성이 있습니다. 이를 Service Mesh라고 부릅니다.</p>
<p>Sidecar 프록시를 붙이는 방식이 대표적인 구현 중 하나입니다. 다만 모든 Service Mesh가 Sidecar 방식을 필수로 사용하지는 않습니다. 자동 주입 기능도 별도 설정을 해야 작동합니다.</p>
<p>또한 Sidecar를 달았다고 주 앱의 포트가 자동으로 외부 접근 불가능해지는 것은 아닙니다. 접속 경로, Service 포트, 네트워크 정책 등을 함께 설계합니다.</p>
<h3 id="4-14-일반-sidecar-패턴과-네이티브-sidecar">4-14. 일반 Sidecar 패턴과 네이티브 Sidecar</h3>
<p>앞의 예제는 <code>containers</code> 아래에 두 컨테이너를 적는 <strong>일반적인 다중 컨테이너 패턴</strong>입니다.</p>
<p>Kubernetes에는 <code>initContainers</code>의 컨테이너에 <code>restartPolicy: Always</code>를 지정해 장기 실행 보조 컨테이너를 다루는 네이티브 Sidecar 기능도 있습니다.</p>
<p>다음은 구조를 설명하기 위한 일부 설정입니다.</p>
<pre><code class="language-yaml">initContainers:
  - name: helper
    image: busybox:1.37
    restartPolicy: Always
    command: [&quot;sh&quot;, &quot;-c&quot;]
    args: [&quot;while true; do echo helper-running; sleep 10; done&quot;]</code></pre>
<p>이 경우는 “Init이면 반드시 종료해야 한다”는 일반 Init 규칙의 예외입니다. 시작·종료 순서 등을 다루는 기능이 있으며, 적용할 클러스터 버전과 기능 지원을 확인해야 합니다. 초보자는 먼저 <strong>준비 후 종료하는 Init</strong>과 <strong>함께 실행하는 Sidecar</strong>의 목적을 구분하세요. <a href="https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/">공식 문서: Sidecar Containers</a></p>
<h3 id="4-15-init과-sidecar-비교">4-15. Init과 Sidecar 비교</h3>
<table>
<thead>
<tr>
<th>질문</th>
<th>일반 Init Container</th>
<th>Sidecar</th>
</tr>
</thead>
<tbody><tr>
<td>주 목적은?</td>
<td>실행 전 준비</td>
<td>앱 실행 중 보조</td>
</tr>
<tr>
<td>종료를 기다리는가?</td>
<td>성공 종료 후 다음 단계로 진행</td>
<td>보통 계속 실행</td>
</tr>
<tr>
<td>예시는?</td>
<td>파일 준비, 의존 조건 확인</td>
<td>프록시, 로그·지표 수집</td>
</tr>
<tr>
<td>메인 앱과 통신하는가?</td>
<td>메인 앱 시작 전이라 직접 대화할 수 없음</td>
<td>함께 실행하며 localhost 통신 가능</td>
</tr>
<tr>
<td>파일 전달은?</td>
<td>같은 볼륨에 준비 결과 저장</td>
<td>필요한 볼륨을 연결해 공유</td>
</tr>
</tbody></table>
<p><strong>이 파트의 핵심:</strong> Init은 시작 전 준비, Sidecar는 실행 중 지원입니다. 실행 명령과 신호 처리도 함께 이해해야 합니다.</p>
<hr />
<h2 id="part-5-deployment--replicaset-이해-볼륨과-pv-기초">PART 5. Deployment &amp; ReplicaSet 이해, 볼륨과 PV 기초</h2>
<h3 id="5-1-pod-하나를-직접-만들면-어떤-한계가-있을까">5-1. Pod 하나를 직접 만들면 어떤 한계가 있을까?</h3>
<p>앞에서 만든 독립 Pod는 구조를 배우기에 좋습니다. 하지만 그 Pod를 삭제하면 원하는 복제본 수를 채워 줄 Deployment·ReplicaSet이 없습니다.</p>
<p>실제 서비스를 운영하려면 다음 목표가 필요합니다.</p>
<pre><code class="language-text">앱은 3개 유지한다.
버전 변경은 차례로 진행한다.
변경에 문제가 생기면 이전 실행 설정으로 되돌린다.</code></pre>
<p>이 목표를 관리하는 리소스가 Deployment입니다.</p>
<h3 id="5-2-deployment-replicaset-pod의-역할을-나누기">5-2. Deployment, ReplicaSet, Pod의 역할을 나누기</h3>
<pre><code class="language-text">Deployment: 버전과 배포 상태 관리
      ↓
ReplicaSet: 특정 Pod 템플릿의 복제본 개수 관리
      ↓
Pod: 실제 앱을 실행할 컨테이너 묶음
      ↓
Container: 앱 프로세스 실행</code></pre>
<p>피자 가게로 비유하면 다음과 같습니다.</p>
<table>
<thead>
<tr>
<th>리소스</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>Deployment</td>
<td>새 조리법 도입과 이전 조리법 복귀를 관리하는 점장</td>
</tr>
<tr>
<td>ReplicaSet</td>
<td>같은 조리법의 작업 공간 개수를 맞추는 관리자</td>
</tr>
<tr>
<td>Pod</td>
<td>한 조리 작업 공간</td>
</tr>
<tr>
<td>Container</td>
<td>그 공간에서 실제 일을 하는 프로그램</td>
</tr>
</tbody></table>
<p>Deployment Controller가 ReplicaSet을 만들고 조정합니다. ReplicaSet Controller는 Pod 수를 조정합니다. ReplicaSet이 직접 앱 프로세스를 실행하는 것은 아닙니다.</p>
<h3 id="5-3-replicaset은-몇-개를-유지한다">5-3. ReplicaSet은 “몇 개”를 유지한다</h3>
<pre><code class="language-text">목표: 3개
현재: 2개 → 부족한 Pod를 생성
현재: 3개 → 수량 목표 충족
현재: 4개 → 조건에 따라 초과 Pod 정리</code></pre>
<p>ReplicaSet의 목표는 복제본 수 유지입니다. 개수가 맞는다고 모든 앱이 정상 응답하는 것은 아닙니다. 준비 상태와 장애 원인을 함께 봐야 합니다.</p>
<p>실무에서는 보통 ReplicaSet을 직접 작성하기보다 Deployment를 통해 관리합니다. Deployment가 관리하는 ReplicaSet을 직접 수정하면 상위 Controller가 다시 상태를 조정할 수 있습니다. <a href="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/">공식 문서: ReplicaSet</a></p>
<h3 id="5-4-deployment-yaml-읽기">5-4. Deployment YAML 읽기</h3>
<p>다음 예제를 <code>web-deployment.yaml</code>로 저장합니다.</p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-demo
  namespace: demo
  labels:
    app: web-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-demo
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web-demo
    spec:
      containers:
        - name: web
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 2
            periodSeconds: 3</code></pre>
<p>이 문서에는 겉의 Deployment와 안의 Pod 설계도가 함께 들어 있습니다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>겉의 <code>metadata.labels</code></td>
<td>Deployment 자체의 이름표</td>
</tr>
<tr>
<td><code>spec.replicas</code></td>
<td>목표 Pod 수</td>
</tr>
<tr>
<td><code>spec.selector</code></td>
<td>관리 대상 Pod를 고르는 조건</td>
</tr>
<tr>
<td><code>spec.template.metadata.labels</code></td>
<td>생성될 Pod의 이름표</td>
</tr>
<tr>
<td><code>spec.template.spec</code></td>
<td>생성될 Pod의 실행 내용</td>
</tr>
<tr>
<td><code>spec.strategy</code></td>
<td>버전 교체 방식</td>
</tr>
</tbody></table>
<p>특히 <strong>Deployment의 Label과 Pod의 Label은 다른 위치에 있습니다.</strong> 겉에만 Label을 적었다고 Pod에 자동 복사되는 것이 아닙니다.</p>
<p><code>selector.matchLabels</code>의 조건은 Pod 템플릿의 Label과 맞아야 합니다. <code>apps/v1</code> Deployment의 Selector는 생성 후 일반적으로 바꿀 수 없으므로, 처음부터 다른 앱과 겹치지 않도록 설계합니다.</p>
<h3 id="5-5-생성된-관계-확인하기">5-5. 생성된 관계 확인하기</h3>
<pre><code class="language-bash">kubectl apply -f web-deployment.yaml
kubectl rollout status deployment/web-demo -n demo
kubectl get deployment,replicaset,pod -n demo -l app=web-demo</code></pre>
<p>Pod 이름에는 Deployment 관련 이름, 템플릿 해시, 고유 접미사가 보일 수 있습니다. 정확한 소속은 이름 모양만 추측하기보다 <code>ownerReferences</code>로 확인할 수 있습니다.</p>
<pre><code class="language-bash">kubectl get pods -n demo -l app=web-demo -o yaml</code></pre>
<p>관계를 읽을 때는 “이 Pod는 어느 ReplicaSet이 관리하고, 그 ReplicaSet은 어느 Deployment가 관리하는가?”를 생각합니다.</p>
<p><code>DESIRED</code>, <code>CURRENT</code>, <code>READY</code>, <code>AVAILABLE</code> 등의 열은 각각 목표 개수, 현재 개수, 준비된 개수, 가용 조건을 만족하는 개수 등을 나타냅니다. 조회하는 리소스에 따라 출력 열이 다릅니다.</p>
<h3 id="5-6-scale-실행-개수-바꾸기">5-6. Scale: 실행 개수 바꾸기</h3>
<pre><code class="language-bash">kubectl scale deployment/web-demo -n demo --replicas=5
kubectl get pods -n demo -l app=web-demo</code></pre>
<p>이 명령은 목표 개수를 5개로 변경합니다. 나중에 <code>replicas: 3</code>인 YAML을 다시 적용하면 설정에 따라 다시 3개가 될 수 있습니다. 명령으로 바꾼 값과 관리하는 YAML의 목표가 어긋나지 않도록 합니다.</p>
<p>운영에서는 HPA 같은 별도 Controller가 복제본 수를 조정할 수도 있습니다. HPA를 설치하지 않고 <code>replicas</code>만 적었다고 사용량에 따라 자동으로 개수가 늘어나는 것은 아닙니다.</p>
<h3 id="5-7-새-버전을-배포하면-왜-replicaset이-늘어날까">5-7. 새 버전을 배포하면 왜 ReplicaSet이 늘어날까?</h3>
<p>Deployment의 Pod 템플릿이 바뀌면 다른 템플릿의 Pod를 관리할 새 ReplicaSet이 생성될 수 있습니다.</p>
<pre><code class="language-text">변경 전
Deployment
└─ ReplicaSet A: 이전 템플릿, Pod 3개

변경 중
Deployment
├─ ReplicaSet A: 이전 템플릿의 Pod를 줄임
└─ ReplicaSet B: 새 템플릿의 Pod를 늘림

변경 완료
Deployment
├─ ReplicaSet A: Pod 0개, 이력으로 남을 수 있음
└─ ReplicaSet B: 새 Pod 3개</code></pre>
<p>이미지, 환경 변수 등 Pod 템플릿 변경은 rollout을 일으킬 수 있습니다. 반면 복제본 수만 바꾸는 것은 새 버전의 Pod 템플릿을 만드는 변경과 다릅니다.</p>
<h3 id="5-8-rolling-update-조금씩-교체하기">5-8. Rolling Update: 조금씩 교체하기</h3>
<p>학교 컴퓨터실의 컴퓨터를 교체한다고 생각해 보세요. 모두 동시에 치우면 수업을 할 수 없습니다. 새 컴퓨터를 준비하고 기존 것을 차례로 치우면 사용 가능한 컴퓨터를 유지하기 쉽습니다.</p>
<p>Rolling Update는 비슷하게 구버전 Pod와 신버전 Pod를 점진적으로 교체합니다.</p>
<p>여기서 두 숫자가 중요합니다.</p>
<table>
<thead>
<tr>
<th>설정</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>maxSurge</code></td>
<td>목표 수보다 추가로 만들 수 있는 Pod 수 또는 비율</td>
</tr>
<tr>
<td><code>maxUnavailable</code></td>
<td>업데이트 중 허용할 가용 복제본의 부족 수 또는 비율</td>
</tr>
</tbody></table>
<p><code>replicas: 3</code>, <code>maxSurge: 1</code>, <code>maxUnavailable: 0</code>이면 업데이트 전략상 목표 3개를 가용하게 유지하면서 새 Pod를 추가하는 방향으로 조정합니다.</p>
<h3 id="5-9-예시-a-가용-개수-3개를-유지하며-교체">5-9. 예시 A: 가용 개수 3개를 유지하며 교체</h3>
<p>아래 표는 외부 장애가 없고 새 Pod가 정상 준비된다는 가정의 <strong>설명용 진행 예</strong>입니다. 실제 Controller의 세부 작업 순서를 고정한 표는 아닙니다.</p>
<table>
<thead>
<tr>
<th>단계</th>
<th>구버전</th>
<th>신버전</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>시작</td>
<td>가용 3</td>
<td>0</td>
<td>기존 서비스</td>
</tr>
<tr>
<td>새 Pod 추가</td>
<td>가용 3</td>
<td>준비 중 1</td>
<td>총 4개로 확장</td>
</tr>
<tr>
<td>새 Pod 준비 완료</td>
<td>가용 3</td>
<td>가용 1</td>
<td>구버전을 줄일 수 있음</td>
</tr>
<tr>
<td>구버전 하나 정리</td>
<td>가용 2</td>
<td>가용 1</td>
<td>가용 3개 유지</td>
</tr>
<tr>
<td>반복</td>
<td>점점 감소</td>
<td>점점 증가</td>
<td>버전 교체</td>
</tr>
<tr>
<td>완료</td>
<td>0</td>
<td>가용 3</td>
<td>새 버전으로 운영</td>
</tr>
</tbody></table>
<p>새 Pod가 계속 실패하면 교체가 멈출 수 있습니다. 여유 자원이 없어 추가 Pod를 배치하지 못할 수도 있습니다.</p>
<p>종료 중인 Pod는 종료 유예 기간 동안 남아 있을 수 있어, 순간적인 실제 Pod 수·자원 사용을 항상 정확히 4개 이하라고 단정하지 않습니다.</p>
<h3 id="5-10-예시-b-한-개의-가용-부족을-허용하기">5-10. 예시 B: 한 개의 가용 부족을 허용하기</h3>
<pre><code class="language-yaml">replicas: 3
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1</code></pre>
<p>업데이트 조정 시 가용 복제본이 목표보다 1개 적은 상태를 허용하므로, 정상적인 교체 과정에서 2개까지 가용 개수가 줄 수 있습니다.</p>
<p>장점은 자원 여유가 적을 때 교체 선택지가 늘어난다는 것입니다. 단점은 교체 중 서비스 처리 여력이 줄 수 있다는 것입니다. “허용했으니 반드시 먼저 하나를 삭제한다”는 고정 순서는 아닙니다.</p>
<h3 id="5-11-기본-비율과-반올림">5-11. 기본 비율과 반올림</h3>
<p>기본 RollingUpdate 설정은 <code>maxSurge: 25%</code>, <code>maxUnavailable: 25%</code>입니다. 비율을 Pod 개수로 계산할 때 Surge는 올림, Unavailable은 내림합니다.</p>
<pre><code class="language-text">replicas = 3
25% = 0.75

maxSurge       → 올림 → 1
maxUnavailable → 내림 → 0</code></pre>
<p>같은 25%라도 결과는 다를 수 있습니다. <code>maxSurge</code>와 <code>maxUnavailable</code>을 동시에 0으로 설정할 수는 없습니다. <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/">공식 문서: Deployment 업데이트 전략</a></p>
<h3 id="5-12-rollingupdate와-recreate-비교">5-12. RollingUpdate와 Recreate 비교</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>동작</th>
<th>판단할 점</th>
</tr>
</thead>
<tbody><tr>
<td>RollingUpdate</td>
<td>이전·새 Pod를 겹쳐 실행하며 점진적으로 교체</td>
<td>버전 공존, 여유 자원, 준비 상태</td>
</tr>
<tr>
<td>Recreate</td>
<td>업데이트 시 기존 버전을 종료하고 새 버전 생성</td>
<td>서비스 중단 가능성, 버전 공존 제약</td>
</tr>
</tbody></table>
<p>RollingUpdate를 쓴다고 무중단이 자동 보장되지는 않습니다. 앱 준비 상태, 종료 처리, 연결 유지, 새·구 버전의 데이터 호환성, 여유 용량 등이 함께 맞아야 합니다.</p>
<p>가령 새 API가 바꾼 DB 형식을 이전 API가 읽을 수 없다면, 두 버전이 공존하는 중 문제가 생길 수 있습니다. 업데이트 정책은 앱과 데이터 설계를 대신하지 않습니다.</p>
<h3 id="5-13-업데이트-상태와-이전-버전-복귀-확인하기">5-13. 업데이트 상태와 이전 버전 복귀 확인하기</h3>
<p>새 이미지의 태그가 실제 존재한다고 가정할 때 명령 형식은 다음과 같습니다.</p>
<pre><code class="language-bash">kubectl set image deployment/web-demo -n demo \
  web=registry.example.com/demo/demo-web:2.0

kubectl rollout status deployment/web-demo -n demo
kubectl rollout history deployment/web-demo -n demo
kubectl get replicaset -n demo -l app=web-demo</code></pre>
<p>위 레지스트리 주소는 설명용입니다. 실제로 실행하려면 접근 가능한 이미지로 바꿉니다. 새 버전의 문제로 이전 Pod 템플릿으로 되돌리는 명령은 다음과 같습니다.</p>
<pre><code class="language-bash"># 직전 이력으로 되돌리기
kubectl rollout undo deployment/web-demo -n demo

# 존재하는 이력 번호를 확인한 뒤 특정 이력으로 되돌리기
kubectl rollout undo deployment/web-demo -n demo --to-revision=1

kubectl rollout status deployment/web-demo -n demo</code></pre>
<p>Revision은 배포 이력 번호입니다. 제품 버전 <code>2.0</code>과 항상 같지 않습니다. 유지된 이력과 설정에 따라 복귀 가능한 범위가 달라집니다.</p>
<p>Rollback은 보통 <strong>Deployment의 Pod 템플릿을 되돌리는 것</strong>입니다. DB 내용, 외부 파일, ConfigMap·Secret 변경까지 자동으로 되돌리는 것은 아닙니다.</p>
<h3 id="5-14-pod가-없어지면-데이터는-어디에-남을까">5-14. Pod가 없어지면 데이터는 어디에 남을까?</h3>
<p>앱에 사진을 업로드했는데 Pod 교체 후 사진이 사라진다고 생각해 봅시다. 앱의 실행과 데이터 보관을 같은 수명에 묶었기 때문일 수 있습니다.</p>
<p>컨테이너의 쓰기 가능 계층을 영구 저장소로 믿으면 안 됩니다. 새 컨테이너나 새 Pod에는 이전 임의 변경 내용이 이어진다고 보장할 수 없습니다.</p>
<p><strong>볼륨은 데이터를 저장하거나 공유할 공간을 컨테이너에 연결하는 방법</strong>입니다.</p>
<pre><code class="language-text">컨테이너 안의 /data
          ↕ 마운트
컨테이너 실행과 분리해 연결한 저장 공간</code></pre>
<p>마운트는 다른 저장 공간을 특정 폴더에서 보이게 연결하는 것입니다. 실제로는 바깥 창고에 물건이 있지만 방 안의 문으로 들어가 사용하는 것과 비슷합니다.</p>
<h3 id="5-15-volumes와-volumemounts의-차이">5-15. <code>volumes</code>와 <code>volumeMounts</code>의 차이</h3>
<pre><code class="language-yaml">volumes:
  - name: shared-data
    emptyDir: {}</code></pre>
<p>이 설정은 Pod에서 사용할 저장 공간을 정의합니다.</p>
<pre><code class="language-yaml">volumeMounts:
  - name: shared-data
    mountPath: /data</code></pre>
<p>이 설정은 그 볼륨을 특정 컨테이너의 어느 경로에서 사용할지 정의합니다.</p>
<pre><code class="language-text">volumes      = 무엇을 연결할까?
volumeMounts = 이 컨테이너의 어디에서 사용할까?</code></pre>
<p>Pod에 볼륨을 정의했다고 모든 컨테이너가 자동으로 같은 폴더를 사용하는 것은 아닙니다. 사용할 컨테이너마다 마운트 설정이 필요합니다.</p>
<h3 id="5-16-emptydir-같은-pod의-임시-공동-작업장">5-16. emptyDir: 같은 Pod의 임시 공동 작업장</h3>
<p>emptyDir는 Pod에 붙는 임시 공간입니다. 앞의 Init 예제에서 파일을 전달할 때 사용했습니다.</p>
<pre><code class="language-text">Pod
├─ 컨테이너 A: /data-a
├─ 컨테이너 B: /data-b
└─ 둘이 연결한 같은 emptyDir 볼륨</code></pre>
<p><strong>동일 Pod 안의 컨테이너가 공유하는 공간</strong>입니다. 다른 Pod가 같은 노드에 있다고 그 emptyDir를 자동으로 공유하지는 않습니다.</p>
<table>
<thead>
<tr>
<th>상황</th>
<th>emptyDir 데이터</th>
</tr>
</thead>
<tbody><tr>
<td>같은 Pod 안에서 컨테이너만 재시작</td>
<td>유지될 수 있음</td>
</tr>
<tr>
<td>Pod가 삭제·제거되고 새 Pod 생성</td>
<td>이전 공간을 이어받지 않음</td>
</tr>
<tr>
<td>새 Pod가 같은 노드에 생성</td>
<td>새 emptyDir 사용</td>
</tr>
</tbody></table>
<p>일반적으로 노드의 저장 공간을 사용하며, <code>medium: Memory</code>로 메모리 기반 공간을 사용할 수도 있습니다. 메모리 기반 공간에 쓴 데이터는 메모리 사용량에 영향을 줍니다.</p>
<p>적합한 용도는 임시 파일, 캐시, 같은 Pod의 파일 전달입니다. 장기 보관해야 할 업로드 파일이나 DB 데이터의 기본 선택으로 쓰지 않습니다. <a href="https://kubernetes.io/docs/concepts/storage/volumes/">공식 문서: Volumes</a></p>
<h3 id="5-17-emptydir-공유를-확인하는-보충-예제">5-17. emptyDir 공유를 확인하는 보충 예제</h3>
<p>다음 내용을 <code>volume-demo.yaml</code>로 저장합니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: volume-demo
  namespace: demo
spec:
  containers:
    - name: writer
      image: busybox:1.37
      command: [&quot;sh&quot;, &quot;-c&quot;]
      args:
        - |
          echo '같은 Pod에서 공유합니다.' &gt; /write/message.txt
          exec sleep 3600
      volumeMounts:
        - name: shared-data
          mountPath: /write
    - name: reader
      image: busybox:1.37
      command: [&quot;sleep&quot;]
      args: [&quot;3600&quot;]
      volumeMounts:
        - name: shared-data
          mountPath: /read
  volumes:
    - name: shared-data
      emptyDir: {}</code></pre>
<pre><code class="language-bash">kubectl apply -f volume-demo.yaml
kubectl wait -n demo --for=condition=Ready pod/volume-demo --timeout=120s
kubectl exec volume-demo -n demo -c reader -- cat /read/message.txt</code></pre>
<p>writer가 <code>/write/message.txt</code>에 쓴 파일을 reader는 <code>/read/message.txt</code>에서 읽습니다. 같은 볼륨이 서로 다른 경로에 연결되었기 때문입니다. 아주 빠르게 실행해 파일이 아직 없으면 writer 로그·상태를 확인하고 다시 읽습니다.</p>
<h3 id="5-18-hostpath-노드의-폴더를-직접-연결하기">5-18. hostPath: 노드의 폴더를 직접 연결하기</h3>
<p>hostPath는 노드의 특정 경로를 Pod에 연결합니다.</p>
<p>아래는 개념을 설명하는 일부 설정입니다.</p>
<pre><code class="language-yaml">volumes:
  - name: node-data
    hostPath:
      path: /var/local/learning-data
      type: DirectoryOrCreate</code></pre>
<p>이 경로는 <strong>kubectl을 입력한 노트북의 폴더가 아니라 Pod가 배치된 노드의 폴더</strong>입니다.</p>
<pre><code class="language-text">Node A의 /var/local/learning-data
        ≠
Node B의 /var/local/learning-data</code></pre>
<p>같은 노드에 다시 배치되면 기존 데이터를 볼 수 있지만, 다른 노드에서는 같은 이름의 경로여도 다른 데이터일 수 있습니다. 노드가 사라지면 데이터도 위험해집니다.</p>
<p>노드 파일에 접근하는 설정이라 권한과 보안 영향을 고려해야 합니다. 앱이 어느 노드에서도 실행되길 바라면서 hostPath를 영구 공유 저장소처럼 사용하면 곤란합니다.</p>
<h3 id="5-19-pv와-pvc-저장-공간과-사용-신청">5-19. PV와 PVC: 저장 공간과 사용 신청</h3>
<p>이제 Pod가 교체되어도 사용할 저장 공간을 연결하고 싶습니다.</p>
<table>
<thead>
<tr>
<th>용어</th>
<th>뜻</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>PV, PersistentVolume</td>
<td>저장소를 나타내는 클러스터 리소스</td>
<td>사용 가능한 창고 공간과 연결 정보</td>
</tr>
<tr>
<td>PVC, PersistentVolumeClaim</td>
<td>앱이 필요한 저장소를 요청하는 리소스</td>
<td>“이 조건의 창고를 사용하겠습니다”라는 신청서</td>
</tr>
<tr>
<td>StorageClass</td>
<td>저장소 생성·선택 정책</td>
<td>창고 상품의 종류</td>
</tr>
<tr>
<td>CSI Driver</td>
<td>저장소와 Kubernetes를 연결하는 구현</td>
<td>창고 연결·관리 담당자</td>
</tr>
</tbody></table>
<pre><code class="language-text">Pod
 ↓ PVC를 참조
PVC
 ↓ 조건에 맞는 PV와 연결
PV
 ↓ 저장소 정보·드라이버를 통해 연결
실제 디스크 또는 파일 시스템</code></pre>
<p>PV는 그 자체로 파일을 품은 거대한 데이터 상자라기보다 실제 저장소를 표현하는 리소스입니다. PVC는 저장 공간의 조건을 요청합니다. <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/">공식 문서: Persistent Volumes</a></p>
<h3 id="5-20-pod에서는-pvc를-참조한다">5-20. Pod에서는 PVC를 참조한다</h3>
<p>아래는 Pod 설정의 일부입니다. 이미 사용할 수 있는 <code>demo-data</code> PVC가 있다는 가정입니다.</p>
<pre><code class="language-yaml">volumes:
  - name: app-data
    persistentVolumeClaim:
      claimName: demo-data</code></pre>
<p>컨테이너에서는 연결 경로를 적습니다.</p>
<pre><code class="language-yaml">volumeMounts:
  - name: app-data
    mountPath: /data</code></pre>
<p>단순히 PVC 이름만 적으면 모든 환경에서 자동으로 저장소가 생기는 것은 아닙니다. PVC 상태, StorageClass·PV의 구성, 드라이버, 권한, 저장소 접근 조건 등이 맞아야 합니다.</p>
<p>PVC는 네임스페이스에 속하며 일반적인 Pod의 참조는 같은 네임스페이스의 PVC를 사용합니다. PV는 클러스터 범위입니다. 다른 사람의 PVC를 이름만 바꿔 임의로 사용하면 된다고 생각하지 않습니다.</p>
<h3 id="5-21-외부-저장소라고-모두-같은-방식으로-공유되지는-않는다">5-21. 외부 저장소라고 모두 같은 방식으로 공유되지는 않는다</h3>
<p>블록 스토리지는 디스크처럼 사용하는 저장소이고, 네트워크 파일 스토리지는 여러 컴퓨터에서 파일 시스템으로 연결하는 방식입니다. 실제 공유 가능 여부는 제품, 드라이버, 접근 모드에 따라 달라집니다.</p>
<p><code>ReadWriteOnce</code>는 “오직 Pod 하나만 쓸 수 있다”는 뜻으로 단순화하지 않습니다. 보통 한 노드에서 읽기·쓰기 마운트를 허용하는 접근 모드입니다. 단일 Pod 접근을 의미하는 모드와 구분해야 합니다.</p>
<p><code>ReadWriteMany</code>도 앱의 동시 쓰기 충돌을 자동으로 해결하지 않습니다. 여러 사람이 같은 문서를 동시에 고치면 규칙이 필요하듯, 앱의 파일·데이터 동시 접근 설계가 필요합니다.</p>
<p>PV/PVC의 바인딩, 정적·동적 프로비저닝, 접근 모드와 회수 정책의 상세 실습은 2일차의 PV 파트에서 확장합니다. 1일차에는 <strong>앱 실행 단위와 데이터 저장 수명을 분리한다</strong>는 원리를 먼저 잡습니다.</p>
<h3 id="5-22-저장-공간-비교">5-22. 저장 공간 비교</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>연결 대상·수명</th>
<th>주로 생각할 용도</th>
<th>주의점</th>
</tr>
</thead>
<tbody><tr>
<td>컨테이너의 쓰기 가능 계층</td>
<td>컨테이너 실행 환경</td>
<td>실행 중 변경·임시 파일</td>
<td>교체 후 유지 보장 없음</td>
</tr>
<tr>
<td>emptyDir</td>
<td>해당 Pod의 임시 공간</td>
<td>Pod 안의 파일 공유·캐시</td>
<td>Pod 제거 시 데이터 사라짐</td>
</tr>
<tr>
<td>hostPath</td>
<td>특정 노드의 경로</td>
<td>제한된 노드 연동·진단</td>
<td>노드 종속과 보안 영향</td>
</tr>
<tr>
<td>PVC로 연결한 저장소</td>
<td>저장소·정책에 따른 수명</td>
<td>지속 데이터 저장</td>
<td>회수 정책과 백업 확인</td>
</tr>
</tbody></table>
<p>“PV를 사용했으니 데이터가 절대 안 사라진다”는 결론도 틀립니다. PVC 삭제 시 실제 저장소를 삭제하는 정책이 있을 수 있고, 저장소 장애나 앱의 잘못된 삭제도 가능합니다. 영속 저장과 백업은 별도 과제입니다.</p>
<h3 id="5-23-보충-실습-자원-정리하기">5-23. 보충 실습 자원 정리하기</h3>
<p>이 글에서 직접 만든 예제만 제거하려면 해당 파일을 지정합니다. 아래 명령은 각 파일을 생성·적용한 경우에 사용합니다.</p>
<pre><code class="language-bash">kubectl delete -f hello-pod.yaml
kubectl delete -f hello-service.yaml
kubectl delete -f init-demo.yaml
kubectl delete -f sidecar-demo.yaml
kubectl delete -f web-deployment.yaml
kubectl delete -f volume-demo.yaml</code></pre>
<p><code>message-demo</code> 예제도 파일로 저장해 적용했다면 그 파일을 지정해 삭제합니다. 삭제 명령은 예제에 사용한 원본 YAML을 대상으로 실행합니다.</p>
<p>이렇게 파일에 담긴 리소스만 지정하면 공유 환경의 다른 실습 자원을 지울 위험을 줄일 수 있습니다. Deployment를 지우면 일반적으로 그 소유의 ReplicaSet과 Pod도 정리됩니다.</p>
<p><strong>이 파트의 핵심:</strong> Deployment는 배포 변화를, ReplicaSet은 복제본 수를 관리합니다. 데이터 보관은 앱 실행 수명과 별도로 설계합니다.</p>
<hr />
<h2 id="part-6-조별-토론하기-배운-내용을-문제에-적용하기">PART 6. 조별 토론하기: 배운 내용을 문제에 적용하기</h2>
<blockquote>
<p>아래는 1일차 개념을 연결하기 위해 새로 만든 보충 토론입니다. 실제 수업의 내부 토론 문제, 평가 항목, 참여자 정보는 옮기지 않았습니다.</p>
</blockquote>
<h3 id="6-1-토론하는-방법">6-1. 토론하는 방법</h3>
<p>각 문제에 다음 순서로 답해 보세요.</p>
<ol>
<li>현상을 쉬운 말로 설명합니다.</li>
<li>어떤 Kubernetes 개념과 연결되는지 말합니다.</li>
<li>추측을 확인할 조회 명령이나 관찰 항목을 고릅니다.</li>
<li>해결할 설정과 그 이유를 설명합니다.</li>
</ol>
<p>정답 용어를 빨리 말하는 것보다 “왜 그렇게 생각했는가?”를 설명하는 연습이 중요합니다.</p>
<h3 id="6-2-토론-1-내-컴퓨터에서는-이미지가-잘-실행된다">6-2. 토론 1: 내 컴퓨터에서는 이미지가 잘 실행된다</h3>
<p><strong>문제:</strong> 로컬에서는 앱이 실행되지만 클러스터의 Pod는 <code>ImagePullBackOff</code>입니다. 무엇부터 볼까요?</p>
<p><strong>생각해 볼 질문</strong></p>
<ul>
<li>이미지를 레지스트리에 올렸나요?</li>
<li>YAML의 주소와 태그가 정확한가요?</li>
<li>노드가 해당 레지스트리에 접근할 수 있나요?</li>
<li>노드가 이미지를 내려받을 인증 설정이 있나요?</li>
</ul>
<p><strong>풀이</strong></p>
<p>로컬 이미지 보유 여부와 노드의 이미지 확보는 다릅니다. <code>kubectl describe pod</code>의 Events를 확인해 이미지 없음, 인증 실패, 네트워크 실패 등을 구분합니다. 다운로드가 성공한 뒤 실행 실패라면 앱 설정이나 CPU 구조 등 다음 계층을 확인합니다.</p>
<h3 id="6-3-토론-2-pod는-running인데-service-연결이-안-된다">6-3. 토론 2: Pod는 Running인데 Service 연결이 안 된다</h3>
<p><strong>문제:</strong> Pod가 실행 중이지만 Service로 요청하면 응답하지 않습니다.</p>
<p><strong>생각해 볼 질문</strong></p>
<ul>
<li>Pod의 Label과 Service의 Selector가 맞나요?</li>
<li>EndpointSlice에 기대한 대상이 있나요?</li>
<li>Pod가 Ready인가요?</li>
<li>앱이 실제로 어느 포트·주소에서 듣고 있나요?</li>
<li>Service의 <code>targetPort</code>가 앱 포트와 맞나요?</li>
<li>네트워크 정책이나 DNS 문제는 없나요?</li>
</ul>
<p><strong>풀이</strong></p>
<p>Running은 실행 상태이고 Service 연결 성공은 별도 조건입니다. Label과 Selector를 비교하고, readiness와 연결 대상 정보를 확인한 다음 포트·네트워크 문제를 좁힙니다.</p>
<h3 id="6-4-토론-3-init이-계속-기다린다">6-4. 토론 3: Init이 계속 기다린다</h3>
<p><strong>문제:</strong> Init Container는 <code>ready.flag</code>가 생길 때까지 기다립니다. 파일은 메인 앱이 시작하면서 만들도록 작성했습니다.</p>
<p><strong>풀이</strong></p>
<p>메인은 Init 성공 종료 후 시작합니다. 따라서 Init은 메인을 기다리고 메인은 Init을 기다립니다. 앞에서 실행 가능한 준비 작업이 파일을 만들도록 바꾸거나, 다른 실행 단계로 역할을 옮겨야 합니다.</p>
<p>비유하면 “입장해야 발급되는 입장권을 입장 전에 제출하라”는 규칙을 만든 셈입니다.</p>
<h3 id="6-5-토론-4-sidecar와-메인-앱을-서로-다른-수로-늘리고-싶다">6-5. 토론 4: Sidecar와 메인 앱을 서로 다른 수로 늘리고 싶다</h3>
<p><strong>문제:</strong> 메인 앱은 5개, Sidecar는 전체 서비스에서 1개만 실행하고 싶습니다. 같은 Pod에 넣는 설계가 적절할까요?</p>
<p><strong>풀이</strong></p>
<p>같은 Pod의 템플릿을 복제하면 두 컨테이너가 함께 늘어납니다. 메인 앱마다 보조 기능이 필요한지, 하나의 중앙 서비스로 충분한지부터 판단합니다. 독립적인 확장이 필요한 기능은 별도 Deployment로 분리하는 방식을 검토합니다.</p>
<h3 id="6-6-토론-5-pod를-지웠는데-다시-나타난다">6-6. 토론 5: Pod를 지웠는데 다시 나타난다</h3>
<p><strong>문제:</strong> Deployment가 만든 Pod 하나를 지웠더니 새로운 이름의 Pod가 생겼습니다. 삭제가 실패했을까요?</p>
<p><strong>풀이</strong></p>
<p>삭제는 성공했을 수 있습니다. 원하는 복제본 수가 유지되어 있으므로 ReplicaSet Controller가 부족한 Pod를 새로 만든 것입니다. 서비스를 중단하려면 관리 리소스의 목표를 바꾸거나 관리 리소스를 정리해야 합니다.</p>
<p>반대로 독립 Pod라면 이를 대체할 Controller가 없어서 자동으로 같은 수를 복구하지 못합니다.</p>
<h3 id="6-7-토론-6-새-pod가-준비되지-않아-배포가-멈춘다">6-7. 토론 6: 새 Pod가 준비되지 않아 배포가 멈춘다</h3>
<p><strong>문제:</strong> <code>maxUnavailable: 0</code>, <code>maxSurge: 1</code>인데 새 버전이 Ready가 되지 않습니다.</p>
<p><strong>풀이</strong></p>
<p>기존 가용 복제본을 유지하면서 새 버전이 준비되기를 기다릴 수 있습니다. 새 Pod의 로그, readiness 경로, 이미지, 환경 설정, Events를 확인합니다. 자원 부족으로 Pending이라면 requests·배치 조건·노드 여유를 확인합니다.</p>
<p>진행 제한 시간을 넘겨 Deployment에 실패 조건이 기록되더라도, 그것만으로 자동으로 이전 버전으로 복귀하는 것은 아닙니다. 별도 배포 자동화가 있는지 확인하고 필요하면 수정 또는 rollback을 수행합니다.</p>
<h3 id="6-8-토론-7-pod-교체-후-파일이-사라진다">6-8. 토론 7: Pod 교체 후 파일이 사라진다</h3>
<p><strong>문제:</strong> 업로드한 사진을 emptyDir에 저장했습니다. Pod 교체 후 사진이 없어졌습니다.</p>
<p><strong>풀이</strong></p>
<p>emptyDir는 해당 Pod의 임시 공간입니다. 새로운 Pod에 같은 데이터가 자동으로 연결되지 않습니다. 지속 보관이 필요하면 PVC 기반 저장소나 앱에 맞는 외부 저장 서비스를 선택합니다. 그 뒤 백업과 삭제 정책도 정합니다.</p>
<h3 id="6-9-토론-8-rollback을-했는데-데이터-오류가-계속된다">6-9. 토론 8: rollback을 했는데 데이터 오류가 계속된다</h3>
<p><strong>문제:</strong> 새 버전에서 DB 형식을 바꿨습니다. Deployment를 이전 revision으로 되돌렸는데 이전 앱도 오류를 냅니다.</p>
<p><strong>풀이</strong></p>
<p>Pod 템플릿 복귀가 DB 변경을 되돌리는 것은 아닙니다. 이전 앱과 새로운 데이터 형식이 호환되는지 확인해야 합니다. 버전 공존을 고려한 데이터 변경, 별도 복구 계획, 백업이 필요합니다.</p>
<h3 id="6-10-직접-설명해-보는-확인-질문">6-10. 직접 설명해 보는 확인 질문</h3>
<table>
<thead>
<tr>
<th>질문</th>
<th>핵심 답</th>
</tr>
</thead>
<tbody><tr>
<td>이미지와 컨테이너의 차이는?</td>
<td>패키지와 실행 환경의 차이</td>
</tr>
<tr>
<td>kubectl은 무엇인가?</td>
<td>Kubernetes API에 요청하는 CLI</td>
</tr>
<tr>
<td>Kubernetes가 노드에 배치하는 기본 단위는?</td>
<td>Pod</td>
</tr>
<tr>
<td>같은 Pod에서 localhost 통신이 가능한 이유는?</td>
<td>네트워크 공간 공유</td>
</tr>
<tr>
<td>같은 Pod면 모든 파일도 자동 공유하는가?</td>
<td>아니요. 볼륨 연결 필요</td>
</tr>
<tr>
<td>Label과 Selector의 차이는?</td>
<td>이름표와 선택 조건</td>
</tr>
<tr>
<td>Namespace는 완전한 보안 경계인가?</td>
<td>아니요. 권한·네트워크 정책 필요</td>
</tr>
<tr>
<td>Scheduler와 kubelet의 차이는?</td>
<td>실행 위치 결정과 노드에서의 실행 관리</td>
</tr>
<tr>
<td>일반 Init과 Sidecar의 차이는?</td>
<td>준비 후 종료와 실행 중 지원</td>
</tr>
<tr>
<td>Deployment와 ReplicaSet의 차이는?</td>
<td>배포 상태·버전 관리와 복제본 수 관리</td>
</tr>
<tr>
<td>emptyDir와 PVC 기반 저장소의 차이는?</td>
<td>Pod 임시 공간과 별도 저장소 연결</td>
</tr>
<tr>
<td>RollingUpdate면 항상 무중단인가?</td>
<td>아니요. 앱·준비 상태·종료·용량 조건도 필요</td>
</tr>
</tbody></table>
<h3 id="6-11-하나의-사례로-전체-흐름-연결하기">6-11. 하나의 사례로 전체 흐름 연결하기</h3>
<p>작은 온라인 서점 API를 운영한다고 가정해 보겠습니다.</p>
<pre><code class="language-text">① 책 목록 API 작성
② Dockerfile로 이미지 생성
③ 레지스트리에 API 이미지 등록
④ Deployment에 이미지와 replicas=3 선언
⑤ API Server가 요청을 받아 저장
⑥ Controller들이 ReplicaSet과 Pod 구성
⑦ Scheduler가 노드 배정
⑧ kubelet과 런타임이 Pod 실행
⑨ Init이 실행 전 필요한 파일 준비
⑩ 앱과 Sidecar가 함께 동작
⑪ Label에 맞는 Service가 앱 연결 지점 제공
⑫ readiness가 요청 처리 준비 상태를 나타냄
⑬ 파일 보관은 적절한 저장소로 분리
⑭ 새 버전은 Deployment의 정책에 따라 교체</code></pre>
<p>요청 경로와 데이터 경로를 따로 생각하면 이해하기 쉽습니다.</p>
<pre><code class="language-text">요청 경로: 사용자 → 연결 지점 → 프록시 → 앱
데이터 경로: 앱 → 마운트된 경로 또는 외부 저장 서비스
관리 경로: 사용자 선언 → API → Controller → 노드의 실행 관리</code></pre>
<p>세 경로가 연결되지만 역할은 다릅니다. Service는 데이터 영구 보관을 맡지 않고, PV는 앱의 버전 교체를 맡지 않습니다.</p>
<hr />
<h2 id="처음-배우는-사람을-위한-복습-순서">처음 배우는 사람을 위한 복습 순서</h2>
<p>먼저 이미지와 컨테이너의 차이를 설명해 보세요. 그다음 “여러 서버에서 앱을 계속 운영해야 한다면 어떤 관리가 필요할까?”를 생각합니다.</p>
<p>Pod와 네임스페이스를 이해했다면 Label·Selector를 붙여 대상 선택을 연습합니다. 이후 <code>apply</code> 요청이 API, Controller, Scheduler, kubelet을 거쳐 실행으로 이어지는 흐름을 말로 설명해 봅니다.</p>
<p>마지막에는 두 쌍을 구분해 보세요. <strong>Init과 Sidecar</strong>, <strong>Deployment와 ReplicaSet</strong>입니다. 그리고 데이터가 어느 실행 단위의 수명에 연결되어 있는지 확인합니다.</p>
<p>처음부터 모든 옵션을 외우는 것보다 다음 질문에 답할 수 있으면 충분한 출발입니다.</p>
<blockquote>
<p>무엇을 실행할까? 어디에서 실행할까? 몇 개를 유지할까? 무엇과 연결할까? 준비되었는지 어떻게 알까? 데이터는 어디에 남길까?</p>
</blockquote>
<h2 id="참고-자료와-예제-범위">참고 자료와 예제 범위</h2>
<p>첨부된 Kubernetes 심화 교재의 1일차 학습 주제와 마지막으로 제시된 시간표를 기준으로 새롭게 설명했습니다. 교재 원문, 실습 화면, 내부 인프라 정보는 재배포하지 않습니다.</p>
<p>세부 동작을 확인할 수 있는 공식 문서는 관련 설명 가까이에 연결했습니다. 예제는 학습용으로 단순화했으며, 이 글 작성 과정에서 실제 클러스터에 배포·실행한 결과를 주장하지 않습니다. 실제 적용에는 클러스터 접근 권한, 네임스페이스, 이미지 접근, 정책과 자원 조건을 확인해야 합니다.</p>
<p>2일차에는 PV의 상세 구성, StatefulSet·Headless Service, ConfigMap, Secret, HTTP Ingress, 무중단 배포를 이어서 학습합니다.</p>