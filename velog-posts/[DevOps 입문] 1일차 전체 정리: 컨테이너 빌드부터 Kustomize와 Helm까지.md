<blockquote>
<p>프로그램을 만드는 것과 사람들이 안정적으로 사용하게 만드는 것은 연결된 일이다. DevOps는 이 연결을 잘 만드는 방법이다.</p>
</blockquote>
<p>처음 DevOps 수업을 들으면 Git, Docker, Kubernetes, CI/CD, Kustomize, Helm이라는 단어가 한꺼번에 등장한다. 이름을 모두 외우려고 하면 어렵다. 먼저 <strong>각 도구가 어떤 문제를 해결하는지</strong> 이해해 보자.</p>
<p>이 글은 「DevOps 이해 및 활용」 교재의 <strong>1일차 학습 주제 전체</strong>를 수업 순서에 맞춰 풀어 쓴 공개용 학습 글이다. 실습 환경 구성부터 조별 토론까지 모두 다룬다. 설명과 예제는 학습을 위해 새로 작성했으며, 원본 슬라이드·실습 스크립트의 복사본은 싣지 않는다. 개인 식별 정보, 교육기관 내부 주소, 공용 계정, 비밀번호, 토큰, 인증 파일도 포함하지 않는다. 교재의 비학습 안내와 쉬어가는 사진은 공개 글의 개념 설명으로 옮기지 않았다.</p>
<p>예제는 작은 <strong>학교 도서관 웹서비스</strong>를 운영한다고 생각하며 읽자. 책 목록을 보여 주는 화면과 대출 신청을 처리하는 서버가 있는 서비스다. 코드 예제에서는 복잡한 앱 대신 공개 웹서버 이미지로 배포 흐름을 연습한다.</p>
<p>명령어는 모두 이해를 위한 예시다. <code>registry.example.com</code>과 <code>charts.example.com</code>은 가상의 주소다. 외부 주소를 사용하는 예제는 실제로 사용할 수 있는 저장소와 권한을 준비해야 한다. Helm의 배포 옵션은 <strong>교재 흐름에 맞춰 Helm 3 기준</strong>으로 설명한다. 다른 주요 버전을 사용한다면 <code>helm version</code>과 해당 버전의 도움말을 확인하자.</p>
<h2 id="목차">목차</h2>
<ol>
<li>실습 환경 구성하기</li>
<li>DevOps 개요</li>
<li>CI 전용 컨테이너 빌드 도구</li>
<li>Kustomize 이해</li>
<li>패키지 매니저 Helm Chart 이해 (1): 개념과 구조</li>
<li>패키지 매니저 Helm Chart 이해 (2): 배포·관리·공유</li>
<li>Helm Chart 기본 문법</li>
<li>조별 토론하기</li>
</ol>
<hr />
<h2 id="1-실습-환경-구성하기">1. 실습 환경 구성하기</h2>
<h3 id="1-1-실습-환경은-무엇일까">1-1. 실습 환경은 무엇일까?</h3>
<p>요리를 배우려면 재료만 있어서는 안 된다. 조리대, 냄비, 가스레인지와 손 씻을 곳이 필요하다. 개발 실습도 비슷하다. 코드를 가지고 있어도 실행 도구와 접속 권한이 없으면 실습할 수 없다.</p>
<p>DevOps 실습에서는 다음 네 가지가 연결되어야 한다.</p>
<pre><code class="language-text">내 컴퓨터
  ├─ Git으로 코드를 가져온다.
  ├─ 빌드 도구로 실행할 이미지를 만든다.
  ├─ 이미지 레지스트리에 이미지를 저장한다.
  └─ kubectl 또는 Helm으로 Kubernetes에 배포한다.</code></pre>
<p><strong>실습 환경 구성은 이 연결이 가능한 상태를 만드는 작업</strong>이다.</p>
<h3 id="1-2-자주-나오는-도구의-역할">1-2. 자주 나오는 도구의 역할</h3>
<table>
<thead>
<tr>
<th>이름</th>
<th>쉬운 설명</th>
<th>실습에서 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td>터미널</td>
<td>글자로 컴퓨터에 부탁하는 창</td>
<td>명령어 실행</td>
</tr>
<tr>
<td>Git</td>
<td>파일 변경 이력을 기록하는 도구</td>
<td>코드 저장·비교·협업</td>
</tr>
<tr>
<td>GitHub 등 원격 저장소</td>
<td>팀이 공유하는 코드 보관함</td>
<td>코드 올리기·가져오기</td>
</tr>
<tr>
<td>Docker 또는 Podman</td>
<td>컨테이너를 다루는 도구 모음</td>
<td>로컬 컨테이너 실행 등</td>
</tr>
<tr>
<td>이미지 빌더</td>
<td>실행에 필요한 파일을 포장하는 도구</td>
<td>컨테이너 이미지 생성</td>
</tr>
<tr>
<td>이미지 레지스트리</td>
<td>만든 이미지의 창고</td>
<td>이미지 업로드·다운로드</td>
</tr>
<tr>
<td>Kubernetes</td>
<td>여러 컴퓨터에서 컨테이너를 관리하는 시스템</td>
<td>앱 배치·복구·복제 관리</td>
</tr>
<tr>
<td>kubectl</td>
<td>Kubernetes에 요청을 보내는 도구</td>
<td>조회·배포·로그 확인</td>
</tr>
<tr>
<td>Helm</td>
<td>Kubernetes 앱 설치 패키지 관리자</td>
<td>차트 설치·업그레이드·롤백</td>
</tr>
<tr>
<td>tree</td>
<td>폴더 구조를 나무 모양으로 보여 주는 도구</td>
<td>실습 파일 구조 확인</td>
</tr>
</tbody></table>
<p><code>kubectl</code>을 설치했다고 Kubernetes 서버가 생기는 것은 아니다. <strong>리모컨을 샀다고 TV가 함께 생기지 않는 것</strong>과 같다. 명령을 받을 클러스터와 접속 설정이 따로 필요하다.</p>
<h3 id="1-3-kubernetes-기본-단어부터-익히자">1-3. Kubernetes 기본 단어부터 익히자</h3>
<table>
<thead>
<tr>
<th>용어</th>
<th>비유</th>
<th>실제 의미</th>
</tr>
</thead>
<tbody><tr>
<td>클러스터</td>
<td>하나의 학교</td>
<td>함께 작동하는 Kubernetes 시스템</td>
</tr>
<tr>
<td>노드</td>
<td>학교 안의 교실</td>
<td>프로그램을 실행하는 서버</td>
</tr>
<tr>
<td>네임스페이스</td>
<td>학년별 사물함 구역</td>
<td>리소스 이름과 접근 범위를 구분하는 논리적 공간</td>
</tr>
<tr>
<td>Pod</td>
<td>함께 사용하는 작은 작업 공간</td>
<td>Kubernetes의 기본 배포 단위. 하나 이상의 컨테이너 포함</td>
</tr>
<tr>
<td>Deployment</td>
<td>필요한 인원을 유지하는 담당자</td>
<td>지정한 Pod 복제 수와 업데이트 관리</td>
</tr>
<tr>
<td>Service</td>
<td>바뀌지 않는 대표 연락처</td>
<td>Pod가 바뀌어도 접근할 수 있는 네트워크 연결 제공</td>
</tr>
<tr>
<td>Ingress</td>
<td>건물의 안내 데스크</td>
<td>HTTP/HTTPS 요청을 규칙에 따라 Service로 연결</td>
</tr>
<tr>
<td>ConfigMap</td>
<td>일반 설정 메모</td>
<td>비밀이 아닌 설정 데이터</td>
</tr>
<tr>
<td>Secret</td>
<td>접근을 통제해야 하는 보관함</td>
<td>비밀번호·토큰 등 민감 데이터 관리용 리소스</td>
</tr>
<tr>
<td>PVC</td>
<td>사용할 저장 공간 신청서</td>
<td>영구 저장 공간을 요청하는 리소스</td>
</tr>
</tbody></table>
<p>Ingress 규칙만 작성한다고 외부 접속이 완성되지는 않는다. 규칙을 처리할 <strong>Ingress Controller</strong>와 환경에 맞는 네트워크·도메인 설정이 필요하다.</p>
<p>네임스페이스도 그 자체로 완전한 보안벽은 아니다. 사용자 권한과 네트워크 정책 등을 함께 구성해야 한다.</p>
<h3 id="1-4-도구-설치와-확인">1-4. 도구 설치와 확인</h3>
<p>macOS에서 Homebrew가 준비되어 있다면 다음과 같은 방식으로 도구를 설치할 수 있다.</p>
<pre><code class="language-bash">brew install kubectl helm tree</code></pre>
<p>설치했는지 확인한다.</p>
<pre><code class="language-bash">git --version
kubectl version --client
helm version
tree --version</code></pre>
<p>Windows와 Linux에서는 각 도구의 공식 설치 안내에 맞게 준비한다. 중요한 것은 명령어를 똑같이 따라 치는 것보다 <strong>내 운영체제에 맞는 도구가 실행되는지 확인하는 것</strong>이다.</p>
<h3 id="1-5-kubeconfig는-출입증과-목적지-정보다">1-5. kubeconfig는 출입증과 목적지 정보다</h3>
<p><code>kubeconfig</code>는 <code>kubectl</code>이 어떤 클러스터에 어떤 사용자로 접근할지 알려 주는 설정 파일이다. 흔히 <code>~/.kube/config</code>에 놓인다.</p>
<p>주요 구성은 다음과 같다.</p>
<ul>
<li><strong>cluster</strong>: 어느 Kubernetes API 서버에 접속하는가?</li>
<li><strong>user</strong>: 어떤 인증 정보로 접속하는가?</li>
<li><strong>context</strong>: 클러스터·사용자·기본 네임스페이스를 어떤 조합으로 사용하는가?</li>
</ul>
<p>비유하면 학교 주소, 내 출입증, 오늘 방문할 교실을 한 묶음으로 기억한 것이다.</p>
<p>교육용 접속 파일이나 설정 스크립트를 받은 경우에는 제공된 절차로 설정한다. 지역·반별 설정 절차가 달라도 목적은 같다. <strong>내가 사용할 클러스터와 작업 공간을 정확히 선택하는 것</strong>이다.</p>
<pre><code class="language-bash"># 현재 선택된 접속 대상
kubectl config current-context

# 선택한 접속 대상과 기본 네임스페이스 확인
kubectl config view --minify

# 작업 공간에서 Pod 조회
kubectl get pods -n devops-demo</code></pre>
<p><code>devops-demo</code>는 예시 이름이다. 수업에서는 자신에게 허용된 네임스페이스로 바꾸면 된다. 접속 설정에는 인증 정보가 포함될 수 있으므로 파일 전체나 출력 화면을 공개 글에 붙이지 않는다.</p>
<p><code>No resources found</code>는 조회가 성공했지만 해당 공간에 Pod가 없다는 뜻일 수 있다. <code>Unauthorized</code>는 인증 문제, <code>Forbidden</code>은 권한 문제일 수 있다. 서로 다른 문제다.</p>
<h3 id="1-6-git-저장소를-준비하는-이유">1-6. Git 저장소를 준비하는 이유</h3>
<p>자동화 도구는 보통 Git 저장소에서 코드를 가져온다. 사람이 자신의 컴퓨터에서만 관리하는 코드는 다른 실행 환경에서 재현하기 어렵다.</p>
<p>Git 작업 흐름은 다음처럼 이해하면 된다.</p>
<pre><code class="language-text">파일 수정 → 변경 파일 선택(add) → 기록 만들기(commit) → 원격 공유(push)</code></pre>
<pre><code class="language-bash">git init
git add README.md
git commit -m &quot;Add project introduction&quot;
git branch -M main
git remote add origin https://github.com/YOUR_ACCOUNT/library-demo.git
git push -u origin main</code></pre>
<p>이 예제는 <code>README.md</code>와 원격 저장소를 먼저 준비했다고 가정한다. <code>YOUR_ACCOUNT</code>는 자신의 계정으로 바꾸는 자리다.</p>
<p><strong>commit은 내 컴퓨터의 기록이고 push는 원격 저장소로 보내는 작업</strong>이다. commit만 하면 팀원이 바로 볼 수 있는 것은 아니다.</p>
<h3 id="1-7-pat는-저장소용-출입권이다">1-7. PAT는 저장소용 출입권이다</h3>
<p>PAT는 Personal Access Token의 줄임말이다. 외부 도구나 HTTPS Git 작업에서 사용할 수 있는 토큰 방식의 인증 수단이다.</p>
<p>“우리 학교의 모든 문을 열 수 있는 열쇠”보다 “도서관에만 들어갈 수 있는 출입증”이 더 적절하다. 필요한 저장소와 작업에 맞춰 권한을 제한하고 유효기간을 설정한다.</p>
<p>GitHub의 세부 화면은 바뀔 수 있으므로 여기서는 메뉴 클릭 순서보다 개념을 기억하자.</p>
<ul>
<li>어떤 저장소에 접근하는가?</li>
<li>읽기만 필요한가, 쓰기도 필요한가?</li>
<li>언제 만료되는가?</li>
<li>자동화 도구에서 어떻게 안전하게 전달할 것인가?</li>
</ul>
<p>토큰은 공개 저장소, 블로그, 스크린샷에 넣지 않는다. 실습 설정 파일에 인증값을 입력해야 한다면 그 파일은 버전 관리에서 제외하고, 실제 운영에서는 CI의 인증정보 저장 기능이나 Secret 관리 기능을 활용한다.</p>
<h3 id="1-8-환경-설정과-yaml-생성">1-8. 환경 설정과 YAML 생성</h3>
<p>교재의 실습 흐름에는 서비스 이름, 이미지 이름, 이미지 버전, 네임스페이스 등을 설정하고 YAML을 생성하는 단계가 있다.</p>
<p>같은 신청서에 이름과 반을 채우는 과정이라고 생각하면 쉽다.</p>
<pre><code class="language-properties"># 비밀정보가 없는 학습용 설정 예시
SERVICE_NAME=library-api
IMAGE_NAME=library-api
VERSION=1.0.0
NAMESPACE=devops-demo
REPLICAS=1</code></pre>
<p>생성 스크립트는 이 값을 읽어 Kubernetes 설정 파일을 만들 수 있다. 생성 후에는 반드시 결과를 확인한다. 입력한 값을 잘못 읽거나 기본값이 남아 있을 수 있기 때문이다.</p>
<p>여러 사람이 같은 환경을 사용한다면 이름 규칙도 중요하다. 각자의 서비스 이름이 겹치면 서로 다른 사람이 같은 리소스를 수정하게 될 수 있다. 공개 예제에서는 학습용 이름을 사용하지만 실제 실습에서는 제공된 명명 규칙을 지킨다.</p>
<h3 id="1-9-이-파트-확인-질문">1-9. 이 파트 확인 질문</h3>
<p><strong>Q. kubectl은 Kubernetes 서버일까?</strong></p>
<p>A. 서버에 요청을 보내는 클라이언트 도구다.</p>
<p><strong>Q. 같은 클러스터인데 다른 친구의 Pod가 안 보일 수 있을까?</strong></p>
<p>A. 조회한 네임스페이스가 다르거나 접근 권한이 다를 수 있다.</p>
<p><strong>Q. Git에 올릴 것과 올리지 않을 것은?</strong></p>
<p>A. 코드와 비밀정보 없는 설정 예제는 올릴 수 있다. 실제 토큰·비밀번호·인증 파일은 공개하지 않는다.</p>
<hr />
<h2 id="2-devops-개요">2. DevOps 개요</h2>
<h3 id="2-1-개발과-운영은-무엇을-할까">2-1. 개발과 운영은 무엇을 할까?</h3>
<p><strong>개발(Development)</strong>은 기능을 만드는 일이다. 도서관 서비스에 책 검색 기능을 추가하는 것이 개발이다.</p>
<p><strong>운영(Operations)</strong>은 사용자가 서비스를 계속 이용할 수 있게 관리하는 일이다. 서버가 잘 작동하는지 살펴보고 문제가 생기면 복구하는 것이 운영이다.</p>
<p>두 업무가 서로 멀어지면 다음과 같은 대화가 생기기 쉽다.</p>
<blockquote>
<p>개발자: “내 컴퓨터에서는 잘 실행되는데요.”</p>
<p>운영자: “실제 서버에서는 실행되지 않는데요.”</p>
</blockquote>
<p>개발자는 새 기능을 빨리 제공하고 싶고, 운영자는 서비스가 멈추지 않기를 원한다. 협업 구조가 나쁘면 속도와 안정성이 서로 싸우는 목표처럼 보인다.</p>
<h3 id="2-2-devops는-함께-책임지고-개선하는-방식이다">2-2. DevOps는 함께 책임지고 개선하는 방식이다</h3>
<p>DevOps는 Development와 Operations를 연결한 말이다. <strong>개발부터 운영까지 같은 목표로 협력하고, 반복 작업을 자동화하며, 결과를 바탕으로 개선하는 문화와 업무 방식</strong>으로 이해하자.</p>
<p>급식실에 비유해 보자.</p>
<ul>
<li>요리 담당은 맛있는 음식을 만든다.</li>
<li>배식 담당은 학생들에게 안전하게 전달한다.</li>
<li>학생 반응을 함께 확인하고 다음 메뉴를 개선한다.</li>
</ul>
<p>음식이 식었다고 서로 탓하기만 하면 문제가 해결되지 않는다. 조리·보관·배식 과정을 함께 살펴봐야 한다.</p>
<p>DevOps에서도 “기능을 완성했으니 내 일은 끝”보다 “사용자가 잘 사용하는가?”를 함께 확인한다. 그렇다고 한 사람이 모든 전문 업무를 혼자 해야 한다는 뜻은 아니다. 역할은 나눌 수 있지만 결과와 피드백을 연결한다.</p>
<h3 id="2-3-핵심-요소-문화-프로세스-자동화-피드백">2-3. 핵심 요소: 문화, 프로세스, 자동화, 피드백</h3>
<table>
<thead>
<tr>
<th>요소</th>
<th>의미</th>
<th>도서관 서비스 예시</th>
</tr>
</thead>
<tbody><tr>
<td>문화</td>
<td>서로 정보를 공유하고 함께 문제 해결</td>
<td>장애 원인을 함께 조사</td>
</tr>
<tr>
<td>프로세스</td>
<td>작업 순서와 확인 기준을 정함</td>
<td>테스트를 통과한 변경을 배포</td>
</tr>
<tr>
<td>자동화</td>
<td>반복 작업을 도구가 수행</td>
<td>코드 변경 시 자동 빌드</td>
</tr>
<tr>
<td>피드백</td>
<td>실제 결과를 보고 다음 작업 개선</td>
<td>느린 검색을 측정해 개선</td>
</tr>
</tbody></table>
<p>자동화만 도입하고 협업이 그대로라면 DevOps의 목적을 충분히 달성하기 어렵다. 잘못된 절차를 자동화하면 잘못된 일이 더 빨리 반복될 수도 있다.</p>
<h3 id="2-4-devops는-계속-도는-순환이다">2-4. DevOps는 계속 도는 순환이다</h3>
<pre><code class="language-text">계획 → 코드 작성 → 빌드 → 테스트 → 릴리스 준비
  ↑                                  ↓
개선 ← 관찰·모니터링 ← 운영 ← 배포</code></pre>
<p>각 단계는 다음 의미다.</p>
<ol>
<li><strong>계획(Plan)</strong>: 어떤 문제를 해결할지 정한다.</li>
<li><strong>코드(Code)</strong>: 기능을 구현하고 변경을 기록한다.</li>
<li><strong>빌드(Build)</strong>: 실행 가능한 프로그램이나 이미지를 만든다.</li>
<li><strong>테스트(Test)</strong>: 기대한 대로 작동하는지 확인한다.</li>
<li><strong>릴리스(Release)</strong>: 제공할 버전과 배포 준비 상태를 정리한다.</li>
<li><strong>배포(Deploy)</strong>: 실행 환경에 변경을 적용한다.</li>
<li><strong>운영(Operate)</strong>: 사용자에게 서비스를 제공한다.</li>
<li><strong>관찰(Monitor)</strong>: 오류·속도·사용량 등을 확인한다.</li>
</ol>
<p>검색 기능을 배포한 뒤 검색이 느리다는 사실을 발견했다면 다시 계획 단계로 돌아간다. <strong>배포는 학습이 끝나는 지점이 아니라 실제 사용 결과를 배우기 시작하는 지점</strong>이다.</p>
<h3 id="2-5-ci-자주-합치고-자동으로-확인하기">2-5. CI: 자주 합치고 자동으로 확인하기</h3>
<p>CI는 Continuous Integration, 지속적 통합이다.</p>
<p>여러 사람이 코드를 오랫동안 따로 작업한 뒤 마지막에 합치면 충돌이 크게 생길 수 있다. 작은 변경을 자주 합치고 빌드·테스트로 문제를 일찍 찾는 것이 CI의 핵심이다.</p>
<p>학교 공동 발표를 생각해 보자. 발표 당일 처음 자료를 합치면 제목 형식이나 내용이 맞지 않을 수 있다. 매일 자료를 함께 확인하면 문제를 조금씩 해결할 수 있다.</p>
<pre><code class="language-text">코드 변경 → Git에 공유 → 자동 빌드 → 자동 테스트 → 결과 확인</code></pre>
<p>Git에 저장하는 것만으로 충분한 CI가 완성되는 것은 아니다. 팀의 통합 방식과 자동 검증이 함께 필요하다.</p>
<h3 id="2-6-cd에는-두-가지-뜻이-있다">2-6. CD에는 두 가지 뜻이 있다</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>의미</th>
<th>운영 배포 직전</th>
</tr>
</thead>
<tbody><tr>
<td>Continuous Delivery</td>
<td>지속적 전달</td>
<td>배포 가능한 상태를 유지하고 사람·정책에 따라 배포 결정</td>
</tr>
<tr>
<td>Continuous Deployment</td>
<td>지속적 배포</td>
<td>정해진 검증을 통과한 변경을 자동으로 운영 배포</td>
</tr>
</tbody></table>
<p>지속적 전달은 도시락을 완성해 배송 가능한 상태로 준비하는 것과 같다. 배송 시작은 사람이 결정할 수 있다.</p>
<p>지속적 배포는 준비와 검사를 통과한 도시락이 자동 배송되는 것과 같다.</p>
<p>“모든 변경을 자동 배포한다”는 말은 실패한 테스트까지 무시한다는 뜻이 아니다. <strong>배포 조건을 통과한 변경</strong>을 자동으로 배포한다는 뜻이다.</p>
<h3 id="2-7-파이프라인은-작업을-연결한-길이다">2-7. 파이프라인은 작업을 연결한 길이다</h3>
<p>파이프라인은 작업 순서를 연결한 흐름이다.</p>
<pre><code class="language-text">Git의 앱 코드
    ↓
CI 도구가 코드 가져오기·빌드·테스트
    ↓
이미지 빌더가 컨테이너 이미지 생성
    ↓
레지스트리에 이미지 저장
    ↓
새 이미지 버전을 배포 설정에 기록
    ↓
CD 도구가 Kubernetes에 반영</code></pre>
<p>교재에서는 Jenkins 또는 Tekton, 이미지 빌더, 이미지 레지스트리, Kustomize 또는 Helm, Argo CD가 연결되는 구성을 소개한다. 1일차에는 역할 관계를 이해하고 빌드·배포 설정을 배운다. Jenkins와 GitOps의 상세 구성은 2일차 범위다.</p>
<p>EKS는 AWS가 제공하는 관리형 Kubernetes 서비스다. “Kubernetes를 AWS 환경에서 사용하는 방법 중 하나”라고 이해하면 된다. 오늘 배우는 개념이 EKS에만 한정되는 것은 아니다.</p>
<h3 id="2-8-무엇이-좋아지고-무엇을-확인해야-할까">2-8. 무엇이 좋아지고, 무엇을 확인해야 할까?</h3>
<p>작은 변경을 자주 검증하면 문제를 일찍 발견할 수 있다. 반복 작업을 자동화하면 수작업 누락을 줄이고 배포 흐름을 일정하게 만들 수 있다.</p>
<p>그러나 자동화했다고 품질이나 속도가 무조건 좋아지는 것은 아니다. 다음과 같은 질문으로 실제 개선을 확인하자.</p>
<ul>
<li>변경한 코드가 사용자에게 전달되기까지 얼마나 걸리는가?</li>
<li>배포 후 장애나 긴급 수정이 얼마나 자주 필요한가?</li>
<li>문제가 생겼을 때 얼마나 빨리 복구하는가?</li>
<li>반복 배포에서 수작업 단계가 얼마나 남아 있는가?</li>
</ul>
<h3 id="2-9-이-파트-확인-질문">2-9. 이 파트 확인 질문</h3>
<p><strong>Q. Jenkins를 설치하면 DevOps가 완성될까?</strong></p>
<p>A. 자동화 도구를 준비한 것이다. 협업·작업 기준·관찰·개선까지 연결해야 한다.</p>
<p><strong>Q. CI와 CD의 차이는?</strong></p>
<p>A. CI는 변경을 통합하고 검증하는 데 초점이 있고, CD는 검증된 변경을 전달하거나 배포하는 데 초점이 있다.</p>
<hr />
<h2 id="3-ci-전용-컨테이너-빌드-도구">3. CI 전용 컨테이너 빌드 도구</h2>
<h3 id="3-1-소스-코드-빌드-결과물-이미지-컨테이너">3-1. 소스 코드, 빌드 결과물, 이미지, 컨테이너</h3>
<p>네 가지를 구분하면 이 파트가 쉬워진다.</p>
<table>
<thead>
<tr>
<th>개념</th>
<th>요리 비유</th>
<th>프로그램 예시</th>
</tr>
</thead>
<tbody><tr>
<td>소스 코드</td>
<td>조리 방법</td>
<td>Java·Python 코드</td>
</tr>
<tr>
<td>앱 빌드 결과물</td>
<td>준비한 음식</td>
<td>실행 가능한 JAR 파일 등</td>
</tr>
<tr>
<td>컨테이너 이미지</td>
<td>음식과 필요한 것을 포장한 밀키트</td>
<td>앱 파일·실행 환경이 담긴 배포 단위</td>
</tr>
<tr>
<td>컨테이너</td>
<td>밀키트를 실제로 조리하는 상황</td>
<td>이미지를 바탕으로 만들어 실행하는 프로세스</td>
</tr>
</tbody></table>
<p>이미지는 실행 준비물이고 컨테이너는 그 준비물을 바탕으로 실행되는 것이다. 하나의 이미지로 여러 컨테이너를 만들 수 있다.</p>
<p>Python 앱은 Java처럼 반드시 JAR를 만드는 단계가 필요한 것은 아니다. 언어와 프로젝트에 따라 앱 빌드 과정이 다르다.</p>
<h3 id="3-2-dockerfile과-빌드-컨텍스트">3-2. Dockerfile과 빌드 컨텍스트</h3>
<p>Dockerfile은 “어떤 이미지에서 시작해 무엇을 복사하고 어떤 명령으로 실행할지” 적은 파일이다.</p>
<p>아래는 직접 만든 HTML을 웹서버 이미지에 넣는 학습 예제다.</p>
<pre><code class="language-dockerfile"># Dockerfile
FROM nginx:1.28-alpine
COPY index.html /usr/share/nginx/html/index.html</code></pre>
<pre><code class="language-html">&lt;!-- index.html --&gt;
&lt;!doctype html&gt;
&lt;html lang=&quot;ko&quot;&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;title&gt;학교 도서관&lt;/title&gt;
  &lt;h1&gt;도서관 서비스에 오신 것을 환영합니다.&lt;/h1&gt;
&lt;/html&gt;</code></pre>
<p><strong>빌드 컨텍스트</strong>는 빌더가 읽을 수 있는 작업 폴더다. 위 예제에서는 Dockerfile과 <code>index.html</code>이 들어 있는 폴더다.</p>
<pre><code class="language-text">library-web/
├── Dockerfile
└── index.html</code></pre>
<p><code>COPY</code>할 파일이 컨텍스트에 없으면 빌드가 실패한다. 컨텍스트에 비밀번호 파일까지 넣으면 실수로 이미지에 포함될 수 있으므로 <code>.dockerignore</code>로 불필요한 파일을 제외한다.</p>
<pre><code class="language-text"># .dockerignore 예시
.git
.env
auth/</code></pre>
<h3 id="3-3-빌더와-런타임은-하는-일이-다르다">3-3. 빌더와 런타임은 하는 일이 다르다</h3>
<ul>
<li><strong>빌더</strong>: 이미지를 만든다. BuildKit, Buildah, Kaniko 등이 여기에 해당한다.</li>
<li><strong>런타임</strong>: 이미지를 바탕으로 컨테이너를 실행·관리한다. containerd, CRI-O 등이 Kubernetes 환경에서 사용된다.</li>
<li><strong>플랫폼·도구 모음</strong>: Docker, Podman처럼 여러 기능을 묶어 제공할 수 있다.</li>
</ul>
<p>레고 설명서를 만들어 상자에 넣는 일과 실제로 조립해 사용하는 일이 다르다고 생각하면 된다.</p>
<p>OCI는 컨테이너 이미지·실행·유통 관련 규격을 정의하는 표준화 프로젝트다. CRI는 Kubernetes가 컨테이너 런타임과 통신하는 인터페이스다. 둘은 같은 개념이 아니다.</p>
<h3 id="3-4-왜-ci용-빌드-도구를-따로-배울까">3-4. 왜 CI용 빌드 도구를 따로 배울까?</h3>
<p>내 노트북에서는 Docker를 실행할 수 있어도 CI 서버나 Kubernetes Pod에서는 같은 구성이 쉽지 않을 수 있다. 호스트 Docker 소켓에 접근하거나 높은 권한을 요구하는 방식은 환경 정책과 충돌할 수 있다.</p>
<p>그래서 다음을 고민한다.</p>
<ul>
<li>Docker 데몬 없이 빌드할 수 있는가?</li>
<li>컨테이너 안에서 실행 가능한가?</li>
<li>캐시로 반복 빌드 시간을 줄일 수 있는가?</li>
<li>여러 CPU 종류에 맞는 이미지를 만들 수 있는가?</li>
<li>현재 환경의 권한·저장소 정책과 맞는가?</li>
</ul>
<p>데몬은 백그라운드에서 계속 요청을 받는 프로그램이다. “항상 대기하는 작업반장”이라고 생각하자. <strong>Docker 데몬이 필요 없다는 사실과 어떤 환경에서도 권한 설정 없이 실행된다는 말은 다르다.</strong></p>
<h3 id="3-5-kaniko-컨테이너-안에서-이미지를-만드는-방식">3-5. Kaniko: 컨테이너 안에서 이미지를 만드는 방식</h3>
<p>Kaniko는 Dockerfile을 읽고 컨테이너 내부에서 이미지 파일시스템을 구성한 뒤 레지스트리에 보낼 수 있는 빌더다. Docker 데몬을 사용하지 않는 방식이 핵심이다.</p>
<p>비유하면 공장 전체를 세우지 않고 이동식 작업대에서 제품을 포장하는 것이다.</p>
<p>세 입력값을 기억하자.</p>
<table>
<thead>
<tr>
<th>입력</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>--dockerfile</code></td>
<td>조립 방법이 적힌 파일 위치</td>
</tr>
<tr>
<td><code>--context</code></td>
<td>조립에 사용할 파일의 위치</td>
</tr>
<tr>
<td><code>--destination</code></td>
<td>완성된 이미지를 보낼 주소와 태그</td>
</tr>
</tbody></table>
<pre><code class="language-text">--dockerfile=/workspace/Dockerfile
--context=dir:///workspace
--destination=registry.example.com/demo/library-web:1.0.0</code></pre>
<p>이것은 옵션의 의미를 보여 주는 예시다. 실제 실행에는 빌더 컨테이너, 작업 파일 마운트, 레지스트리 인증 설정이 필요하다.</p>
<p>로컬에서 빌더 컨테이너를 실행한다면 내 폴더를 컨테이너의 <code>/workspace</code>에 연결한다. Pod 안에서 실행한다면 빌드 파일과 인증 설정을 볼 수 있는 볼륨 등을 준비한다.</p>
<blockquote>
<p>현재 상태도 구분하자. 교재에 등장하는 원본 <code>GoogleContainerTools/kaniko</code> 저장소는 2025년 6월 3일 보관 처리되었고, 원본 프로젝트는 더 이상 개발·유지보수되지 않는다고 안내한다. 교재의 빌드 원리를 학습하는 것과 새로운 시스템의 도구를 선택하는 것은 별도 판단이다. <a href="https://github.com/GoogleContainerTools/kaniko">원본 저장소 안내</a></p>
</blockquote>
<p>“Docker 명령으로 Kaniko 컨테이너를 실행한다”와 “Kaniko가 Docker 데몬을 사용해 이미지를 빌드한다”도 구분해야 한다. <strong>실행에 사용한 도구와 이미지 생성 방식은 다른 층위</strong>다.</p>
<h3 id="3-6-pod에서-python-이미지를-빌드하는-흐름">3-6. Pod에서 Python 이미지를 빌드하는 흐름</h3>
<p>교재의 실습을 개념 순서로 바꾸면 다음과 같다.</p>
<pre><code class="language-text">작은 Python 코드 + Dockerfile 준비
    ↓
Pod가 읽을 수 있게 작업 파일 전달
    ↓
레지스트리 인증을 Secret 등으로 전달
    ↓
빌더 Pod 실행
    ↓
로그 확인 → 레지스트리에서 이미지 확인
    ↓
완료된 학습용 빌드 Pod 정리</code></pre>
<p>작은 예제 파일은 ConfigMap으로 전달할 수 있지만, ConfigMap은 대형 소스코드 저장소가 아니다. 실제 프로젝트는 Git checkout이나 별도의 작업 볼륨 등을 사용하는 편이 맞을 수 있다.</p>
<p>Secret도 “내용이 자동으로 완벽히 암호화된 파일”이라고 생각하면 안 된다. Base64 표현은 암호화가 아니며, 적절한 접근 제어와 저장 시 암호화 설정 등을 고려해야 한다. <a href="https://kubernetes.io/docs/concepts/configuration/secret/">Kubernetes Secret 설명</a></p>
<p>학습용 조회 명령의 예시는 다음과 같다. 실제 Pod 이름은 만든 리소스 이름에 맞춘다.</p>
<pre><code class="language-bash">kubectl get pod library-build -n devops-demo
kubectl logs library-build -n devops-demo
kubectl describe pod library-build -n devops-demo</code></pre>
<p><code>Completed</code>는 일회성 빌드 작업이 성공적으로 종료된 상태일 수 있다. 웹서버처럼 계속 실행해야 하는 프로그램과 판단 기준이 다르다. 이미지가 레지스트리에 실제로 저장됐는지도 확인한다.</p>
<h3 id="3-7-spring-boot-앱-빌드와-이미지-빌드를-나누기">3-7. Spring Boot: 앱 빌드와 이미지 빌드를 나누기</h3>
<p>Spring Boot 같은 Java 프로젝트에서는 보통 소스 코드를 실행 가능한 JAR로 만드는 과정이 먼저 있다.</p>
<pre><code class="language-text">Git에서 Java 코드 가져오기
    ↓
Maven으로 앱 빌드·테스트
    ↓
JAR 생성
    ↓
Dockerfile로 컨테이너 이미지 생성
    ↓
레지스트리에 push</code></pre>
<p>Pod에서는 <strong>init container</strong>가 먼저 코드를 가져오고 앱을 빌드하도록 구성할 수 있다. 그 작업이 성공하면 일반 컨테이너에서 이미지 빌더가 실행된다.</p>
<p>공유 작업 공간이 중요하다. 앞 작업자가 만든 JAR를 뒤 작업자가 읽어야 하기 때문이다. <code>emptyDir</code> 같은 Pod 내 공유 볼륨을 사용할 수 있다. 이 볼륨의 데이터는 Pod 수명과 연결되므로 영구 보관 장소로 생각하면 안 된다.</p>
<pre><code class="language-text">init container: 코드 가져오기 + Maven 빌드
          │
          └── 공유 작업 공간 ──→ 이미지 빌더 컨테이너</code></pre>
<p>실습에서 테스트 생략 옵션을 보더라도 실제 CI의 일반 원칙으로 외우지 말자. 정상적인 CI 흐름에서는 필요한 테스트를 실행하고 실패한 변경을 걸러내는 것이 중요하다.</p>
<p>컨테이너가 여러 개라면 로그도 구분한다.</p>
<pre><code class="language-bash">kubectl logs library-build -n devops-demo -c app-build
kubectl logs library-build -n devops-demo -c image-build</code></pre>
<h3 id="3-8-buildah-이미지를-조립하는-도구">3-8. Buildah: 이미지를 조립하는 도구</h3>
<p>Buildah는 OCI 이미지를 생성하는 도구다. Dockerfile 기반 빌드뿐 아니라 명령을 조합해 이미지를 구성하는 방식도 지원한다.</p>
<p>레고를 설명서대로 조립할 수도 있고, 부품을 직접 골라 조립할 수도 있는 방식이라고 생각하자. <a href="https://github.com/podman-container-tools/buildah">Buildah 프로젝트 설명</a></p>
<pre><code class="language-bash"># 해당 빌드 환경에 Buildah와 레지스트리 인증이 준비된 경우
buildah bud -t registry.example.com/demo/library-web:1.0.0 .
buildah push registry.example.com/demo/library-web:1.0.0</code></pre>
<p><code>bud</code>는 Dockerfile을 사용해 빌드하는 명령이다. 끝의 <code>.</code>은 현재 폴더를 컨텍스트로 사용한다는 뜻이다.</p>
<p>Buildah도 로컬 또는 빌드용 Pod에서 실행할 수 있다. 컨테이너 내부 실행에서는 사용자 권한, 스토리지 드라이버, 보안 정책에 맞는 설정이 필요할 수 있다. “데몬이 없으니 무조건 무설정”이라고 외우지 않는다.</p>
<p>교재의 실습은 로컬 실행, 작은 Python 이미지 빌드, Spring Boot 앱 빌드와 이미지 생성 흐름을 확인하는 것으로 이해하면 된다.</p>
<h3 id="3-9-buildkit과-buildx-효율적인-이미지-빌드">3-9. BuildKit과 Buildx: 효율적인 이미지 빌드</h3>
<p>BuildKit은 이미지 빌드 엔진이고 Buildx는 Docker CLI에서 BuildKit 기능을 사용하는 확장 도구다.</p>
<p>BuildKit은 서로 독립적인 작업을 함께 처리하고 이전 작업 결과를 재사용하는 등 빌드를 효율적으로 수행하는 기능을 제공한다. 모든 Dockerfile 명령이 무조건 동시에 실행되는 것은 아니다. 앞 결과가 필요한 작업은 기다려야 한다. <a href="https://github.com/moby/buildkit">BuildKit 프로젝트 설명</a></p>
<pre><code class="language-bash"># Docker와 적절한 Buildx 빌더가 준비된 경우
docker buildx build \
  -t registry.example.com/demo/library-web:1.0.0 \
  --push .</code></pre>
<p><code>--push</code>는 빌드한 이미지를 레지스트리에 보낸다는 뜻이다. 이 명령을 썼다고 서비스가 Kubernetes에 자동 배포되지는 않는다.</p>
<p><strong>캐시</strong>는 이미 계산한 답을 다시 쓰는 것이다. 앱 코드만 조금 바뀌었는데 실행 환경 설치부터 매번 반복하면 낭비다. 변경하지 않은 단계의 결과를 재사용하면 반복 빌드 비용을 줄일 수 있다.</p>
<p>BuildKit의 rootless 실행은 관리자 권한 사용을 줄이는 방식이다. 다만 호스트와 클러스터가 필요한 조건을 지원해야 한다. rootless라는 단어만으로 모든 실행 권한 문제를 해결했다고 판단하지 않는다.</p>
<h3 id="3-10-멀티-플랫폼-빌드-같은-이름-다른-cpu용-이미지">3-10. 멀티 플랫폼 빌드: 같은 이름, 다른 CPU용 이미지</h3>
<p>CPU에는 서로 다른 종류가 있다. 흔히 <code>amd64</code>와 <code>arm64</code>를 만난다. 같은 앱이라도 포함된 실행 파일이 CPU 종류와 맞지 않으면 실행되지 않을 수 있다.</p>
<p>같은 책을 한국어판과 영어판으로 만들어 두고 독자에게 맞는 판을 주는 것에 비유할 수 있다. 실제 차이는 언어가 아니라 실행할 CPU·운영체제의 조합이다.</p>
<pre><code class="language-text">library-web:1.0.0
  ├─ linux/amd64용 이미지
  └─ linux/arm64용 이미지</code></pre>
<pre><code class="language-bash">docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/demo/library-web:1.0.0 \
  --push .</code></pre>
<p>여러 플랫폼의 빌드에는 네이티브 빌드 노드, 에뮬레이션 또는 크로스 컴파일 등의 준비가 필요하다. 옵션만 추가하면 모든 소스가 모든 플랫폼에서 실행된다고 보장되는 것은 아니다. <a href="https://docs.docker.com/build/building/multi-platform/">Docker 멀티 플랫폼 빌드 안내</a></p>
<h3 id="3-11-manifest-list와-oci-image-index">3-11. Manifest List와 OCI Image Index</h3>
<p>멀티 플랫폼 이미지에는 <strong>각 플랫폼의 이미지를 가리키는 목차</strong>가 필요하다.</p>
<ul>
<li><strong>Manifest List</strong>: Docker 이미지 규격의 목차 형식.</li>
<li><strong>OCI Image Index</strong>: OCI 규격의 목차 형식.</li>
<li><strong>개별 Image Manifest</strong>: 한 이미지의 설정과 파일 레이어들을 가리키는 정보.</li>
<li><strong>레이어</strong>: 이미지 파일시스템을 구성하는 내용의 묶음.</li>
<li><strong>digest</strong>: 내용에 기반한 식별값. <code>sha256:...</code> 같은 형태로 표시된다.</li>
</ul>
<pre><code class="language-text">태그: library-web:1.0.0
        ↓
플랫폼별 목차
  ├─ amd64 → 해당 이미지 manifest → 설정·레이어
  └─ arm64 → 해당 이미지 manifest → 설정·레이어</code></pre>
<p>일반적인 멀티 플랫폼 pull에서는 실행 환경에 맞는 항목을 선택한다. 태그는 이름표이고 digest는 내용의 지문에 가깝다. 같은 태그를 다른 내용에 붙일 수 있으므로 중요한 배포에서는 이미지 내용을 명확히 추적할 수 있어야 한다.</p>
<p>Buildah로는 플랫폼별 이미지를 만든 뒤 <code>manifest create</code>, <code>manifest add</code>, <code>manifest push --all</code>로 목차를 구성할 수 있다. 지원 버전에서는 여러 플랫폼과 <code>--manifest</code>를 함께 지정해 구성하는 방식도 있다. 핵심은 <strong>플랫폼별 이미지와 이들을 묶는 목차를 함께 관리하는 것</strong>이다.</p>
<p>Kaniko 자체는 여러 플랫폼 이미지를 하나의 index로 묶는 전체 과정을 제공하는 도구로 이해하면 안 된다. 각 플랫폼의 실행 환경에서 빌드하고 별도 도구로 목차를 결합하는 구성이 필요하다.</p>
<h3 id="3-12-도구를-비교할-때-기억할-것">3-12. 도구를 비교할 때 기억할 것</h3>
<table>
<thead>
<tr>
<th>도구</th>
<th>학습 핵심</th>
<th>확인할 조건</th>
</tr>
</thead>
<tbody><tr>
<td>Kaniko</td>
<td>Docker 데몬 없이 빌더 컨테이너로 이미지 생성</td>
<td>원본 보관 상태, 작업 파일·인증 전달</td>
</tr>
<tr>
<td>Buildah</td>
<td>Dockerfile 또는 명령 기반 이미지 구성</td>
<td>사용자 권한·스토리지·실행 정책</td>
</tr>
<tr>
<td>BuildKit / Buildx</td>
<td>캐시·의존성 분석·멀티 플랫폼 빌드</td>
<td>빌더 구성·캐시·지원 플랫폼</td>
</tr>
</tbody></table>
<p>교재의 점유율 추정이나 속도 순위를 모든 환경의 사실로 외우지 말자. 실제 성능은 앱 구조, 캐시, 네트워크, CPU, 설정에 따라 달라진다.</p>
<h3 id="3-13-이-파트-확인-질문">3-13. 이 파트 확인 질문</h3>
<p><strong>Q. 이미지를 push했으면 앱 배포가 끝난 것일까?</strong></p>
<p>A. 이미지 창고에 저장한 것이다. Kubernetes가 그 이미지를 사용하도록 배포 설정을 적용해야 한다.</p>
<p><strong>Q. Maven과 이미지 빌더는 같은 일을 할까?</strong></p>
<p>A. Maven은 Java 앱을 빌드하고 테스트하는 역할이고, 이미지 빌더는 앱과 실행 환경을 이미지로 만드는 역할이다.</p>
<p><strong>Q. 멀티 플랫폼 이미지는 하나의 실행 파일이 모든 CPU에서 작동한다는 뜻일까?</strong></p>
<p>A. 플랫폼에 맞는 개별 이미지를 만들고 목차로 묶는 방식이다.</p>
<hr />
<h2 id="4-kustomize-이해">4. Kustomize 이해</h2>
<h3 id="4-1-yaml은-kubernetes에-전달하는-주문서다">4-1. YAML은 Kubernetes에 전달하는 주문서다</h3>
<p>YAML은 사람이 읽기 쉬운 데이터 표현 형식이다. Kubernetes에는 YAML로 원하는 리소스의 상태를 적어 전달할 수 있다.</p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: library-web
spec:
  replicas: 1</code></pre>
<p>이 코드는 <strong>Deployment의 일부만 보여 주는 조각</strong>이다. “이름이 library-web인 Deployment를 만들고 Pod 복제 수를 1개로 유지해 달라”는 의미다. 실제 Deployment에는 selector와 Pod template 등도 필요하다.</p>
<p><code>kubectl apply</code>는 원하는 상태를 전달한다. 적용 요청이 성공했다고 앱 준비까지 끝난 것은 아니다. 이후 상태와 로그를 확인해야 한다.</p>
<h3 id="4-2-환경마다-yaml을-복사하면-무엇이-문제일까">4-2. 환경마다 YAML을 복사하면 무엇이 문제일까?</h3>
<p>보통 환경을 나누어 사용한다.</p>
<ul>
<li><strong>dev</strong>: 개발자가 기능을 만들고 확인하는 환경.</li>
<li><strong>staging</strong>: 운영과 비슷한 조건으로 최종 점검하는 환경.</li>
<li><strong>prod</strong>: 실제 사용자가 이용하는 운영 환경.</li>
</ul>
<p>개발 환경은 Pod 1개, 운영 환경은 Pod 3개가 필요하다고 하자. YAML 전체를 복사해 관리하면 공통 수정도 여러 파일에 반복해야 한다. 한쪽만 고치면 환경이 서로 달라진다.</p>
<p>기본 교복은 같고 학년별 명찰만 다른데, 교복 제작법을 학년마다 통째로 따로 관리하는 것과 비슷하다.</p>
<h3 id="4-3-base와-overlay">4-3. Base와 Overlay</h3>
<p>Kustomize는 <strong>공통 Kubernetes YAML을 그대로 두고 필요한 차이를 추가·변경해 최종 YAML을 만드는 도구</strong>다.</p>
<ul>
<li><strong>Base</strong>: 공통 기본 설정.</li>
<li><strong>Overlay</strong>: 특정 환경의 변경·추가 설정.</li>
<li><strong>Patch</strong>: 원본에서 일부 항목을 바꾸는 수정 내용.</li>
<li><strong>kustomization.yaml</strong>: 어떤 리소스를 모으고 어떤 변형을 적용할지 적은 파일.</li>
</ul>
<pre><code class="language-text">공통 Base + dev의 차이 → dev용 YAML
공통 Base + prod의 차이 → prod용 YAML</code></pre>
<p>Helm과 달리 별도의 <code>{{ ... }}</code> 템플릿 문법을 기본 방식으로 사용하지 않는다. 다만 설정 병합과 패치 규칙은 학습해야 한다.</p>
<h3 id="4-4-직접-읽어-보는-완전한-최소-예제">4-4. 직접 읽어 보는 완전한 최소 예제</h3>
<p>아래 파일을 만들면 로컬에서 Kustomize 결과를 생성해 볼 수 있다. 아직 클러스터에 배포할 필요는 없다.</p>
<pre><code class="language-text">kustomize-demo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── configmap.yaml
    │   ├── patch-deployment.yaml
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml</code></pre>
<p><strong>① <code>base/deployment.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: library-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: library-web
  template:
    metadata:
      labels:
        app: library-web
    spec:
      containers:
        - name: web
          image: nginx:1.28-alpine
          ports:
            - containerPort: 80</code></pre>
<p><code>selector.matchLabels</code>는 관리할 Pod를 고르는 조건이고 <code>template.metadata.labels</code>는 만들어질 Pod의 이름표다. 둘이 맞아야 한다.</p>
<p><strong>② <code>base/service.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v1
kind: Service
metadata:
  name: library-web
spec:
  selector:
    app: library-web
  ports:
    - port: 80
      targetPort: 80</code></pre>
<p>Service는 <code>app: library-web</code> 이름표가 있는 Pod로 요청을 보낸다. <code>port</code>는 Service 쪽 포트이고 <code>targetPort</code>는 Pod 쪽으로 보낼 포트다.</p>
<p><strong>③ <code>base/kustomization.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml</code></pre>
<p><code>resources</code>는 “이 파일들을 함께 사용하라”는 뜻이다.</p>
<p><strong>④ <code>overlays/dev/configmap.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: library-settings
data:
  APP_ENV: &quot;dev&quot;</code></pre>
<p><strong>⑤ <code>overlays/dev/patch-deployment.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: library-web
spec:
  template:
    spec:
      containers:
        - name: web
          envFrom:
            - configMapRef:
                name: library-settings</code></pre>
<p>이 패치는 기존 <code>web</code> 컨테이너에 환경변수 입력을 추가한다. Kubernetes가 환경변수를 전달해 주지만, 애플리케이션이 그 값을 사용해야 실제 행동이 바뀐다. 여기서 사용하는 nginx 이미지가 <code>APP_ENV</code>만 보고 개발 모드로 바뀌는 것은 아니다.</p>
<p><strong>⑥ <code>overlays/dev/kustomization.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: devops-demo
namePrefix: dev-
resources:
  - ../../base
  - configmap.yaml
patches:
  - path: patch-deployment.yaml
images:
  - name: nginx
    newTag: &quot;1.28-alpine&quot;</code></pre>
<p>기본 리소스에 개발용 ConfigMap을 더하고 패치를 적용한다. 리소스 이름에는 <code>dev-</code>를 붙인다. Kustomize는 알려진 리소스 참조도 함께 변환하므로 이 예제의 ConfigMap 참조 역시 생성된 이름에 맞춰 바뀐다.</p>
<p><strong>⑦ <code>overlays/prod/kustomization.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: devops-prod
resources:
  - ../../base
replicas:
  - name: library-web
    count: 3
images:
  - name: nginx
    newTag: &quot;1.28-alpine&quot;</code></pre>
<p>운영용에서는 같은 기본 설정을 가져오면서 Pod 수를 3개로 바꾼다. 운영용 고가용성이 이 숫자 하나만으로 완성되는 것은 아니다. 노드 분산, 자원, 네트워크 등도 함께 설계해야 한다.</p>
<h3 id="4-5-결과-생성과-실제-적용을-구분하자">4-5. 결과 생성과 실제 적용을 구분하자</h3>
<p><code>kustomize-demo</code> 폴더에서 실행한다.</p>
<pre><code class="language-bash"># YAML만 생성: 클러스터 변경 없음
kubectl kustomize overlays/dev
kubectl kustomize overlays/prod

# 생성 결과를 파일로 저장
kubectl kustomize overlays/dev &gt; rendered-dev.yaml</code></pre>
<p>출력에서 확인할 항목은 이름, 네임스페이스, Pod 수, 이미지, ConfigMap 참조다.</p>
<p>클러스터 접근 권한과 해당 네임스페이스가 준비되어 있다면 다음 단계를 수행할 수 있다.</p>
<pre><code class="language-bash"># 서버의 유효성 검사. 리소스를 실제 저장하지 않음
kubectl apply -k overlays/dev --dry-run=server

# 실제 적용
kubectl apply -k overlays/dev

# 상태 확인
kubectl get deployment,service,configmap,pods -n devops-demo
kubectl rollout status deployment/dev-library-web -n devops-demo</code></pre>
<p><code>-f</code>는 일반 YAML 파일을 적용할 때, <code>-k</code>는 Kustomize 구성을 적용할 때 사용하는 옵션이다.</p>
<p>실습을 마친 뒤 <strong>이 예제의 리소스를 실제로 제거할 때만</strong> 아래 명령을 사용한다.</p>
<pre><code class="language-bash">kubectl delete -k overlays/dev</code></pre>
<p>Kustomize 생성·적용·삭제 흐름은 Kubernetes 공식 문서에서도 확인할 수 있다. <a href="https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/">공식 Kustomize 안내</a></p>
<h3 id="4-6-이미지-태그와-저장소-이름-변경">4-6. 이미지 태그와 저장소 이름 변경</h3>
<pre><code class="language-yaml">images:
  - name: nginx
    newName: registry.example.com/demo/library-web
    newTag: &quot;2.0.0&quot;</code></pre>
<ul>
<li><code>name</code>: 원본에서 바꿀 이미지 이름.</li>
<li><code>newName</code>: 바꿀 이미지 저장소 이름.</li>
<li><code>newTag</code>: 바꿀 태그.</li>
</ul>
<p>이 설정은 <strong>이미지 사용처를 바꾸는 설정</strong>이다. 이미지를 새로 빌드하거나 업로드하는 기능은 아니다.</p>
<p>여기서 교재 문구를 정확히 구분하자. <code>kubectl kustomize</code>와 <code>kubectl apply -k</code>도 <code>images.newTag</code> 설정을 읽어 반영할 수 있다. 반면 <strong>파일을 명령으로 편집하는 <code>kustomize edit set image</code>는 별도 Kustomize CLI의 명령</strong>이다.</p>
<pre><code class="language-bash"># 별도 Kustomize CLI가 설치된 경우, overlays/dev 폴더에서 실행
kustomize edit set image nginx=nginx:1.28-alpine</code></pre>
<p>즉 “kubectl은 이미지 태그 변경 설정을 처리하지 못한다”가 아니라 “kubectl에 동일한 edit 명령이 있는 것은 아니다”라고 이해해야 한다.</p>
<h3 id="4-7-labels-annotations-selector-주의점">4-7. labels, annotations, selector 주의점</h3>
<p><strong>label</strong>은 검색·분류·선택에 사용하는 이름표다. <strong>annotation</strong>은 부가 설명이나 도구가 읽는 메타데이터다.</p>
<p>Kustomize의 <code>namespace</code>, <code>namePrefix</code>, <code>nameSuffix</code>, <code>labels</code>, <code>commonAnnotations</code> 등으로 공통 값을 적용할 수 있다. label을 추가할 때 selector까지 바꿀지는 별도 판단이 필요하다.</p>
<p>Deployment의 selector는 생성 뒤 임의로 변경할 수 없는 필드다. 이미 존재하는 Deployment에 selector 변경이 들어가면 <code>field is immutable</code> 오류가 날 수 있다. 단순 철자 문제가 아니라 <strong>기존 리소스의 변경 제한</strong>이다. <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/">Deployment 변경 제한</a></p>
<p><code>commonLabels</code>나 <code>labels</code>의 <code>includeSelectors</code> 설정이 어떤 결과를 만드는지 확인하자. 특히 기존 selector를 바꾸는지 생성된 YAML과 실제 리소스를 비교한다. 오류를 만났다고 운영 리소스를 무작정 삭제하기보다 label 정책과 변경 목적을 먼저 살핀다.</p>
<h3 id="4-8-이-파트-확인-질문">4-8. 이 파트 확인 질문</h3>
<p><strong>Q. 개발과 운영에서 Pod 수만 다르다면 YAML을 통째로 복사해야 할까?</strong></p>
<p>A. 공통 Base를 두고 환경별 Overlay에서 차이만 관리할 수 있다.</p>
<p><strong>Q. <code>kubectl kustomize</code>를 실행하면 배포될까?</strong></p>
<p>A. 최종 YAML을 생성한다. 실제 적용은 <code>kubectl apply -k</code>로 수행한다.</p>
<p><strong>Q. 이미지 태그를 바꾸면 새 이미지가 만들어질까?</strong></p>
<p>A. 기존에 만들어 저장한 이미지 중 사용할 버전을 지정하는 것이다.</p>
<hr />
<h2 id="5-패키지-매니저-helm-chart-이해-1-개념과-구조">5. 패키지 매니저 Helm Chart 이해 (1): 개념과 구조</h2>
<h3 id="5-1-왜-패키지-매니저가-필요할까">5-1. 왜 패키지 매니저가 필요할까?</h3>
<p>도서관 서비스를 배포하려면 웹서버만 필요한 것이 아니다. 앱 실행 설정, 네트워크 연결, 환경 설정, 저장 공간 등이 함께 필요할 수 있다.</p>
<pre><code class="language-text">앱 하나
  ├─ Deployment: 실행과 복제 관리
  ├─ Service: 접근 경로
  ├─ ConfigMap: 일반 설정
  ├─ Secret: 민감 설정
  ├─ Ingress: 외부 HTTP/HTTPS 연결 규칙
  └─ PVC: 영구 저장 공간 요청</code></pre>
<p>이 YAML들을 매번 하나씩 배포하고 버전을 맞추면 관리가 어려워진다. Helm은 <strong>관련 Kubernetes 리소스의 템플릿과 설정을 Chart라는 패키지로 묶어 설치·관리하는 도구</strong>다.</p>
<p>레고 부품을 각각 골라 사는 대신 필요한 부품과 조립 설명서가 들어 있는 세트를 사용하는 것과 비슷하다.</p>
<p>여러 YAML을 Helm으로 묶어도 그 안의 Kubernetes 리소스가 사라지는 것은 아니다. YAML의 생성과 배포 관리를 한 단위로 다루는 것이다.</p>
<h3 id="5-2-chart-values-template-release">5-2. Chart, Values, Template, Release</h3>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 의미</th>
<th>도서관 서비스 예시</th>
</tr>
</thead>
<tbody><tr>
<td>Chart</td>
<td>설치할 패키지와 생성 규칙</td>
<td>도서관 서비스 배포 세트</td>
</tr>
<tr>
<td>Values</td>
<td>템플릿에 넣을 설정값</td>
<td>이미지 태그·Pod 수·포트</td>
</tr>
<tr>
<td>Template</td>
<td>값이 들어갈 자리가 있는 YAML 틀</td>
<td>Deployment 생성용 틀</td>
</tr>
<tr>
<td>Release</td>
<td>차트를 설치한 하나의 인스턴스</td>
<td>개발용으로 설치한 <code>library-dev</code></td>
</tr>
<tr>
<td>Revision</td>
<td>릴리스 변경 이력 번호</td>
<td>설치 1번, 업그레이드 2번 등</td>
</tr>
</tbody></table>
<p><strong>Chart와 Release는 다르다.</strong> 같은 레고 세트를 두 개 조립할 수 있듯 같은 차트를 서로 다른 이름·설정으로 여러 번 설치할 수 있다. 다만 이름이 겹치는 리소스나 클러스터 공용 리소스를 만드는 차트는 별도 조정이 필요하다.</p>
<pre><code class="language-text">같은 Chart
  ├─ dev 설정 → library-dev Release
  └─ prod 설정 → library-prod Release</code></pre>
<p>Helm의 차트·릴리스·저장소 구분은 공식 사용 안내에서도 소개한다. <a href="https://helm.sh/docs/v3/intro/using_helm/">Helm 사용 안내</a></p>
<h3 id="5-3-템플릿은-빈칸이-있는-문서다">5-3. 템플릿은 빈칸이 있는 문서다</h3>
<p>다음은 개념을 설명하는 <strong>템플릿 조각</strong>이다.</p>
<pre><code class="language-yaml">spec:
  replicas: {{ .Values.replicaCount }}</code></pre>
<p>값이 아래와 같으면,</p>
<pre><code class="language-yaml">replicaCount: 3</code></pre>
<p>결과는 다음과 같다.</p>
<pre><code class="language-yaml">spec:
  replicas: 3</code></pre>
<p>빈칸을 채워 최종 문서를 만드는 과정을 <strong>렌더링</strong>이라고 한다.</p>
<pre><code class="language-text">차트의 템플릿 + 설정값 + 릴리스 정보
                 ↓
             Helm 렌더링
                 ↓
       일반 Kubernetes YAML 생성</code></pre>
<p>Kubernetes에 <code>{{ .Values.replicaCount }}</code>가 들어 있는 템플릿을 그대로 보내는 것이 아니다. Helm이 먼저 일반 YAML로 만든다.</p>
<h3 id="5-4-차트-디렉터리의-역할">5-4. 차트 디렉터리의 역할</h3>
<pre><code class="language-text">library-chart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl
│   └── NOTES.txt
└── .helmignore</code></pre>
<table>
<thead>
<tr>
<th>파일·폴더</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>Chart.yaml</code></td>
<td>차트 이름·버전·설명·의존성 등의 메타데이터</td>
</tr>
<tr>
<td><code>values.yaml</code></td>
<td>템플릿이 사용할 기본 설정값</td>
</tr>
<tr>
<td><code>templates/</code></td>
<td>Kubernetes YAML을 생성하는 템플릿</td>
</tr>
<tr>
<td><code>_helpers.tpl</code></td>
<td>여러 파일에서 재사용할 템플릿 정의</td>
</tr>
<tr>
<td><code>NOTES.txt</code></td>
<td>설치 뒤 보여 줄 사용 안내</td>
</tr>
<tr>
<td><code>charts/</code></td>
<td>하위 차트와 의존 차트</td>
</tr>
<tr>
<td><code>.helmignore</code></td>
<td>패키지에 넣지 않을 파일의 패턴</td>
</tr>
</tbody></table>
<p><code>helm create</code>로 기본 차트 틀을 만들 수 있다.</p>
<pre><code class="language-bash">helm create starter-chart
tree starter-chart</code></pre>
<p>생성되는 기본 파일은 Helm 버전에 따라 달라질 수 있다. 기본 차트에는 Ingress, 자동 확장, 테스트 등 여러 기능이 들어 있을 수 있으므로 파일을 이해하지 않고 이미지 이름만 바꾸면 설정이 맞지 않을 수 있다.</p>
<h3 id="5-5-chartyaml의-두-버전은-다르다">5-5. Chart.yaml의 두 버전은 다르다</h3>
<pre><code class="language-yaml">apiVersion: v2
name: library-chart
description: A small web service for learning Helm
type: application
version: 0.1.0
appVersion: &quot;1.28-alpine&quot;</code></pre>
<ul>
<li><code>apiVersion</code>: 차트 형식의 버전. Kubernetes Deployment의 <code>apiVersion</code>과 다르다.</li>
<li><code>name</code>: 차트 이름.</li>
<li><code>type</code>: <code>application</code>은 설치할 앱 차트, <code>library</code>는 재사용할 템플릿 등을 제공하는 차트.</li>
<li><code>version</code>: <strong>차트 패키지</strong>의 버전.</li>
<li><code>appVersion</code>: <strong>앱 버전</strong>을 설명하는 정보.</li>
</ul>
<p>앱 코드를 바꾸지 않아도 배포 구조를 고쳤다면 차트 버전을 올릴 수 있다. 반대로 앱의 이미지가 바뀌는 경우에는 앱 버전과 이미지 태그도 관리해야 한다.</p>
<p><code>appVersion</code>을 바꾼다고 이미지 태그가 자동으로 바뀌는 것은 아니다. 템플릿이 <code>.Chart.AppVersion</code>을 사용하는지, <code>.Values.image.tag</code>를 사용하는지에 따라 달라진다. 차트 버전도 이미지 태그와 별도다. <a href="https://helm.sh/docs/v3/topics/charts/">차트 메타데이터 설명</a></p>
<h3 id="5-6-values는-사용되는-설정만-효과가-있다">5-6. Values는 사용되는 설정만 효과가 있다</h3>
<pre><code class="language-yaml">replicaCount: 1
image:
  repository: nginx
  tag: &quot;1.28-alpine&quot;
  pullPolicy: IfNotPresent
service:
  port: 80</code></pre>
<p><code>.Values.image.tag</code>는 <code>image</code> 안에 있는 <code>tag</code>를 읽는다는 뜻이다. 마치 서랍장 → 두 번째 서랍 → 물건 순으로 찾아가는 것이다.</p>
<p><code>values.yaml</code>에 아무 키나 넣었다고 Kubernetes 설정이 생기지는 않는다. 템플릿이 그 키를 읽어 결과에 넣어야 한다.</p>
<p>예를 들어 <code>ingress.enabled: true</code>를 추가해도 Ingress 템플릿이 없다면 Ingress가 생성되지 않는다. <strong>설정값과 그 값을 사용하는 코드가 연결되어야 한다.</strong></p>
<h3 id="5-7-내장-정보-chart와-release">5-7. 내장 정보: .Chart와 .Release</h3>
<table>
<thead>
<tr>
<th>표현</th>
<th>출처</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>.Values.replicaCount</code></td>
<td>values</td>
<td>원하는 Pod 복제 수</td>
</tr>
<tr>
<td><code>.Chart.Name</code></td>
<td>Chart.yaml</td>
<td>차트 이름</td>
</tr>
<tr>
<td><code>.Chart.Version</code></td>
<td>Chart.yaml</td>
<td>차트 버전</td>
</tr>
<tr>
<td><code>.Chart.AppVersion</code></td>
<td>Chart.yaml</td>
<td>앱 버전 정보</td>
</tr>
<tr>
<td><code>.Release.Name</code></td>
<td>실행 명령</td>
<td>설치 인스턴스 이름</td>
</tr>
<tr>
<td><code>.Release.Namespace</code></td>
<td>실행 명령·설정</td>
<td>배포 네임스페이스</td>
</tr>
</tbody></table>
<pre><code class="language-bash">helm template library-dev ./library-chart -n devops-demo</code></pre>
<p>이 경우 <code>.Release.Name</code>은 <code>library-dev</code>, <code>.Release.Namespace</code>는 <code>devops-demo</code>다.</p>
<p><code>helm template</code>은 로컬에서 YAML을 생성한다. 릴리스를 클러스터에 설치하지 않는다.</p>
<h3 id="5-8-네-가지-주요-효과">5-8. 네 가지 주요 효과</h3>
<p><strong>복잡도 관리</strong>: 관련 리소스를 한 차트로 묶어 구조를 관리한다.</p>
<p><strong>업데이트 관리</strong>: 설정과 차트 버전을 지정해 릴리스를 업그레이드한다.</p>
<p><strong>공유</strong>: 차트를 패키지로 만들고 저장소에 올려 다른 사람이 설치할 수 있게 한다.</p>
<p><strong>롤백</strong>: 릴리스 이력을 보고 이전 리비전의 배포 정의로 되돌릴 수 있다. 데이터까지 완전히 되돌리는 기능은 아니므로 범위를 구분해야 한다.</p>
<p>교재에서 소개하는 Helm의 이름은 배의 조타 장치와 관련되어 있다. Kubernetes를 항해 관리에 비유한다면 Helm은 앱 설치와 변경을 다루는 조종 도구로 기억할 수 있다.</p>
<h3 id="5-9-kustomize와-helm은-어떻게-다를까">5-9. Kustomize와 Helm은 어떻게 다를까?</h3>
<table>
<thead>
<tr>
<th>비교</th>
<th>Kustomize</th>
<th>Helm</th>
</tr>
</thead>
<tbody><tr>
<td>중심 방식</td>
<td>원본 YAML + 환경별 변형</td>
<td>템플릿 + values</td>
</tr>
<tr>
<td>주요 단위</td>
<td>Base·Overlay</td>
<td>Chart·Release</td>
</tr>
<tr>
<td>문법</td>
<td>Kubernetes YAML과 변형 설정</td>
<td>YAML과 Go 템플릿 문법</td>
</tr>
<tr>
<td>환경별 차이</td>
<td>Overlay로 관리</td>
<td>환경별 values로 관리</td>
</tr>
<tr>
<td>패키지 공유</td>
<td>YAML 구성 공유</td>
<td>버전 있는 차트 패키지 공유</td>
</tr>
<tr>
<td>릴리스 이력·롤백</td>
<td>자체 Helm 릴리스 관리 기능은 없음</td>
<td>Helm 릴리스 이력 관리</td>
</tr>
</tbody></table>
<p>어느 도구가 무조건 더 좋다고 결정할 필요는 없다. 이미 있는 YAML의 환경별 차이를 관리할지, 여러 사용자가 설치할 패키지를 만들지에 따라 선택 기준이 달라진다. 함께 사용하는 구성도 가능하지만 복잡도가 늘어나는지도 살핀다.</p>
<h3 id="5-10-이-파트-확인-질문">5-10. 이 파트 확인 질문</h3>
<p><strong>Q. Chart는 앱을 실행하는 컨테이너 이미지일까?</strong></p>
<p>A. Kubernetes 리소스를 생성·설치하는 패키지다. 그 안에서 사용할 이미지 주소를 지정할 수 있다.</p>
<p><strong>Q. <code>version</code>, <code>appVersion</code>, <code>image.tag</code>는 항상 같은 값일까?</strong></p>
<p>A. 각각 차트 버전, 앱 버전 정보, 실행 이미지 태그로 역할이 다르다.</p>
<hr />
<h2 id="6-패키지-매니저-helm-chart-이해-2-배포·관리·공유">6. 패키지 매니저 Helm Chart 이해 (2): 배포·관리·공유</h2>
<h3 id="6-1-최소-차트로-흐름을-직접-확인하자">6-1. 최소 차트로 흐름을 직접 확인하자</h3>
<p>앞 파트의 구조를 가장 작은 예제로 만들어 보자. 아래 다섯 파일만 포함하는 새 <code>library-chart</code> 폴더를 사용한다. <code>helm create</code>가 만든 많은 파일 위에 일부만 덮어 쓰는 방식과 섞지 않는다.</p>
<pre><code class="language-text">library-chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment.yaml
    └── service.yaml</code></pre>
<p><strong>① <code>library-chart/Chart.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v2
name: library-chart
description: A small web service for learning Helm
type: application
version: 0.1.0
appVersion: &quot;1.28-alpine&quot;</code></pre>
<p><strong>② <code>library-chart/values.yaml</code></strong></p>
<pre><code class="language-yaml">replicaCount: 1
image:
  repository: nginx
  tag: &quot;1.28-alpine&quot;
  pullPolicy: IfNotPresent
service:
  port: 80</code></pre>
<p><strong>③ <code>library-chart/templates/_helpers.tpl</code></strong></p>
<pre><code class="language-gotemplate">{{- define &quot;library-chart.fullname&quot; -}}
{{- printf &quot;%s-%s&quot; .Release.Name .Chart.Name | trunc 63 | trimSuffix &quot;-&quot; -}}
{{- end -}}

{{- define &quot;library-chart.selectorLabels&quot; -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}</code></pre>
<p>이름 생성 규칙과 Pod 선택용 label을 여러 파일에서 공유한다. 자세한 문법은 다음 파트에서 배운다.</p>
<p><strong>④ <code>library-chart/templates/deployment.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include &quot;library-chart.fullname&quot; . }}
  labels:
    {{- include &quot;library-chart.selectorLabels&quot; . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include &quot;library-chart.selectorLabels&quot; . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include &quot;library-chart.selectorLabels&quot; . | nindent 8 }}
    spec:
      containers:
        - name: web
          image: &quot;{{ .Values.image.repository }}:{{ .Values.image.tag }}&quot;
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80</code></pre>
<p><strong>⑤ <code>library-chart/templates/service.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v1
kind: Service
metadata:
  name: {{ include &quot;library-chart.fullname&quot; . }}
spec:
  type: ClusterIP
  selector:
    {{- include &quot;library-chart.selectorLabels&quot; . | nindent 4 }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http</code></pre>
<p>여기서 Service가 선택하는 label과 Deployment가 만드는 Pod의 label이 같다. 서로 다른 Release가 다른 Pod를 선택하도록 릴리스 이름도 이름표에 넣었다.</p>
<p>예제는 흐름 학습에 집중한 최소 구성이다. 실무에서는 자원 요청·제한, 상태 점검, 보안 설정 등을 환경에 맞게 추가한다.</p>
<h3 id="6-2-환경별-values와-우선순위">6-2. 환경별 Values와 우선순위</h3>
<p>차트 밖에 <code>values-dev.yaml</code>, <code>values-prod.yaml</code>을 만들 수 있다.</p>
<pre><code class="language-yaml"># values-dev.yaml
replicaCount: 1</code></pre>
<pre><code class="language-yaml"># values-prod.yaml
replicaCount: 3</code></pre>
<pre><code class="language-bash">helm template library-prod ./library-chart \
  -f values-prod.yaml \
  -n devops-prod</code></pre>
<p>일반적인 설정 입력의 우선순위는 다음처럼 이해한다.</p>
<pre><code class="language-text">차트의 기본 values.yaml
       ↓ 덮어쓰기
먼저 지정한 -f 파일
       ↓ 덮어쓰기
나중에 지정한 -f 파일
       ↓ 덮어쓰기
명령줄 --set / --set-string 등의 입력</code></pre>
<pre><code class="language-bash">helm template library-prod ./library-chart \
  -f values-dev.yaml \
  -f values-prod.yaml \
  --set replicaCount=4</code></pre>
<p>이 예제의 최종 Pod 수는 4다. 이미지 태그처럼 문자열이어야 하는 값은 따옴표를 사용하고 필요하면 <code>--set-string</code>을 사용한다. 값의 계층과 타입도 맞아야 한다. <a href="https://helm.sh/docs/v3/chart_template_guide/values_files/">Values 우선순위 설명</a></p>
<h3 id="6-3-설치-전에-확인하는-세-단계">6-3. 설치 전에 확인하는 세 단계</h3>
<p><strong>① <code>helm lint</code>: 차트의 기본적인 문제 확인</strong></p>
<pre><code class="language-bash">helm lint ./library-chart</code></pre>
<p><strong>② <code>helm template</code>: 만들어질 YAML 확인</strong></p>
<pre><code class="language-bash">helm template library-dev ./library-chart \
  -f values-dev.yaml \
  -n devops-demo</code></pre>
<p><strong>③ dry-run: 설치 과정을 실제 저장 없이 미리 확인</strong></p>
<pre><code class="language-bash"># Helm 3의 클라이언트 dry-run 예시
helm install library-dev ./library-chart \
  -f values-dev.yaml \
  -n devops-demo \
  --dry-run --debug</code></pre>
<p>dry-run을 “앱이 실제 환경에서 정상 작동함을 보장하는 테스트”로 이해하면 안 된다. 이미지가 실행되는지, 트래픽을 처리하는지, DB가 연결되는지까지 확인해 주지는 않는다.</p>
<p>클라이언트 dry-run과 서버에 연결하는 dry-run도 다르다. 지원 버전에서 <code>--dry-run=server</code>는 서버 연결을 허용하는 동작이다. 서버와 관련된 검증 범위는 Helm 버전과 실행 경로에 따라 구분한다. <strong><code>--dry-run</code>이라는 이름만 보고 모든 서버 검증을 수행했다고 판단하지 않는다.</strong> 출력에는 Secret 내용이 포함될 수 있으므로 공개하지 않는다. <a href="https://helm.sh/docs/v3/helm/helm_install/">Helm 3 install 옵션</a></p>
<h3 id="6-4-실제-설치와-상태-확인">6-4. 실제 설치와 상태 확인</h3>
<p>권한 있는 학습 클러스터와 <code>devops-demo</code> 네임스페이스가 준비된 경우의 예시다. 여기부터는 실제 클러스터 상태가 바뀐다.</p>
<pre><code class="language-bash">helm install library-dev ./library-chart \
  -f values-dev.yaml \
  -n devops-demo</code></pre>
<p>결과를 단계별로 확인한다.</p>
<pre><code class="language-bash"># 릴리스 목록
helm list -n devops-demo

# 릴리스 상태 요약
helm status library-dev -n devops-demo

# 리소스 실행 상태
kubectl get deployment,service,pods -n devops-demo

# Deployment 준비 여부
kubectl rollout status deployment/library-dev-library-chart -n devops-demo</code></pre>
<p>이 예제의 Service는 ClusterIP이므로 곧바로 인터넷에 공개되는 것이 아니다. 로컬에서 확인할 때는 다음처럼 포트를 임시로 연결할 수 있다.</p>
<pre><code class="language-bash">kubectl port-forward service/library-dev-library-chart 8080:80 \
  -n devops-demo</code></pre>
<p>명령을 실행한 상태에서 브라우저로 <code>http://localhost:8080</code>에 접속한다. 확인이 끝나면 <code>Ctrl+C</code>로 연결을 종료한다. 이 최소 차트는 공개 nginx 이미지의 기본 화면을 보여 준다. 앞 파트에서 만든 도서관 HTML을 보려면 그 파일을 포함해 빌드한 이미지를 사용해야 한다.</p>
<h3 id="6-5-설치되지-않았으면-설치하고-있으면-업그레이드">6-5. 설치되지 않았으면 설치하고, 있으면 업그레이드</h3>
<pre><code class="language-bash">helm upgrade --install library-dev ./library-chart \
  -f values-dev.yaml \
  -n devops-demo</code></pre>
<p><code>upgrade --install</code>은 이름의 릴리스가 있으면 업그레이드하고 없으면 설치한다. 자동화 스크립트에서 자주 사용하는 패턴이다.</p>
<p>Helm 3에서 상태를 기다리고 실패 처리를 함께 구성한 예시는 다음과 같다.</p>
<pre><code class="language-bash">helm upgrade --install library-dev ./library-chart \
  -f values-dev.yaml \
  --set replicaCount=2 \
  -n devops-demo \
  --wait --timeout 5m --atomic</code></pre>
<table>
<thead>
<tr>
<th>옵션</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>--wait</code></td>
<td>지정한 종류의 리소스가 준비 조건을 충족할 때까지 기다림</td>
</tr>
<tr>
<td><code>--timeout 5m</code></td>
<td>해당 작업의 제한 시간 지정</td>
</tr>
<tr>
<td><code>--atomic</code></td>
<td>실패 시 설치 자원 정리 또는 업그레이드 롤백 수행. Helm 3에서 wait도 설정됨</td>
</tr>
<tr>
<td><code>--reuse-values</code></td>
<td>업그레이드에서 이전 릴리스 values를 재사용하고 새 입력과 병합</td>
</tr>
</tbody></table>
<p><code>5m</code>은 명령 실행의 대기 제한값이다.</p>
<p><code>--reuse-values</code>를 사용할 때는 오래된 설정이 계속 남는지 확인한다. 새 차트의 기본값을 어떻게 받아야 하는지도 검토해야 한다. 설정을 명확히 관리하려면 배포에 필요한 values 파일을 일관되게 관리하는 것이 도움이 된다. <a href="https://helm.sh/docs/v3/helm/helm_upgrade/">Helm 3 upgrade 옵션</a></p>
<h3 id="6-6-설치-후-실제-설정과-생성-결과-확인">6-6. 설치 후 실제 설정과 생성 결과 확인</h3>
<pre><code class="language-bash"># 사용자가 지정한 values
helm get values library-dev -n devops-demo

# 계산된 전체 values
helm get values library-dev -n devops-demo --all

# Helm 릴리스에 기록된 매니페스트
helm get manifest library-dev -n devops-demo

# 이력
helm history library-dev -n devops-demo</code></pre>
<p><code>helm get manifest</code>는 Helm이 기록한 배포 정의다. 다른 도구나 운영자가 이후 수정했다면 실제 리소스와 차이가 있을 수 있으므로 <code>kubectl get ... -o yaml</code> 등으로 실제 상태도 확인한다.</p>
<p>화면이 예전 내용처럼 보인다면 배포 상태, 실제 이미지, 브라우저 캐시를 각각 구분해 확인한다. 새로고침 한 번으로 서버 배포 문제까지 해결되는 것은 아니다.</p>
<h3 id="6-7-롤백-차트-버전과-리비전을-구분하자">6-7. 롤백: 차트 버전과 리비전을 구분하자</h3>
<pre><code class="language-bash">helm history library-dev -n devops-demo
helm rollback library-dev 1 -n devops-demo --wait --timeout 5m</code></pre>
<p>여기서 <code>1</code>은 이미지 태그나 차트 버전이 아니라 <strong>해당 릴리스의 리비전 번호</strong>다. 보통 처음 설치가 리비전 1이고, 이후 업그레이드가 새 리비전으로 기록된다. 롤백 역시 새 이력을 만든다.</p>
<p>비유하면 문서의 이전 저장본을 골라 새 현재 상태로 만드는 것이다.</p>
<p>단, Helm 롤백은 모든 현실의 일을 되돌리는 마법이 아니다.</p>
<ul>
<li>이미 변경된 데이터베이스 내용은 자동으로 복원되지 않는다.</li>
<li>외부 시스템에 보낸 메시지를 자동 취소하지 않는다.</li>
<li>이전 이미지가 삭제되었거나 이전 앱이 새 DB 구조와 맞지 않으면 복구가 어려울 수 있다.</li>
</ul>
<p><strong>이전 배포 정의로 되돌리는 것과 서비스 전체를 안전하게 복구하는 것은 구분</strong>해야 한다.</p>
<h3 id="6-8-릴리스-삭제">6-8. 릴리스 삭제</h3>
<pre><code class="language-bash"># 이 학습용 릴리스를 제거할 때 사용
helm uninstall library-dev -n devops-demo</code></pre>
<p>릴리스가 관리하는 리소스를 제거하는 작업이다. 영구 저장소, 리소스 보존 설정, 외부 시스템 등의 생명주기는 별도로 확인한다.</p>
<p>Helm으로 관리하던 리소스를 다른 도구로 임의 수정하면 다음 업그레이드에서 예상과 다른 결과가 생길 수 있다. 어떤 도구가 설정의 기준인지 팀에서 정해 두는 것이 좋다.</p>
<h3 id="6-9-차트를-패키지로-만들기">6-9. 차트를 패키지로 만들기</h3>
<pre><code class="language-bash">helm package ./library-chart --destination ./chart-packages</code></pre>
<p>차트 버전이 <code>0.1.0</code>이면 <code>library-chart-0.1.0.tgz</code> 같은 파일이 생성된다. <code>.tgz</code>는 tar 아카이브를 gzip으로 압축한 형식이다.</p>
<p>차트 패키지에는 템플릿과 설정이 들어간다. <strong>컨테이너 이미지 자체가 들어가는 것은 아니다.</strong> 차트가 지정한 이미지는 이미지 레지스트리에서 가져온다.</p>
<p>패키징 전에 차트 안에 토큰, kubeconfig, 내부 접속 정보가 없는지 확인한다. 개인 환경의 실제 values를 공유용 기본값에 그대로 복사하면 민감정보가 패키지에 들어갈 수 있다.</p>
<h3 id="6-10-chart-repository와-indexyaml">6-10. Chart Repository와 index.yaml</h3>
<p>전통적인 HTTP 기반 Helm 차트 저장소는 차트 패키지와 <code>index.yaml</code>을 제공한다.</p>
<pre><code class="language-text">차트 저장소
├── index.yaml
├── library-chart-0.1.0.tgz
└── library-chart-0.2.0.tgz</code></pre>
<p><code>index.yaml</code>은 <strong>차트 이름·버전·패키지 위치 등의 목록</strong>이다. 도서관에서 책의 제목과 위치를 찾는 목록표와 비슷하다.</p>
<pre><code class="language-text">저장소 등록·목록 갱신
       ↓
index.yaml에서 차트와 버전 찾기
       ↓
.tgz 패키지 가져오기
       ↓
템플릿 생성·설치</code></pre>
<p>패키지 목록을 생성하는 예시다.</p>
<pre><code class="language-bash">helm repo index ./chart-packages \
  --url https://charts.example.com/library</code></pre>
<p>이 명령은 로컬 목록 파일을 만든다. 웹서버에 자동 업로드하지는 않는다. <code>--url</code>은 사용자가 패키지를 다운로드할 수 있는 실제 주소여야 한다.</p>
<p>저장소 사용자는 다음처럼 등록하고 설치할 수 있다.</p>
<pre><code class="language-bash">helm repo add library-repo https://charts.example.com/library
helm repo update
helm search repo library-repo/library-chart --versions

helm install library-test library-repo/library-chart \
  --version 0.1.0 \
  -n devops-demo</code></pre>
<p><code>--version</code>은 여기서 <strong>차트 버전</strong>을 지정한다. 앱 이미지 태그와 구분한다. <a href="https://helm.sh/docs/v3/topics/chart_repository/">차트 저장소 가이드</a></p>
<h3 id="6-11-github-pages와-nginx-파일-서버">6-11. GitHub Pages와 nginx 파일 서버</h3>
<p>교재에서는 차트 저장소를 제공하는 두 가지 방식을 다룬다. 둘 다 기본 원리는 <strong>파일을 HTTP/HTTPS로 내려받게 하는 것</strong>이다.</p>
<p><strong>GitHub Pages 방식</strong></p>
<ol>
<li>공개해도 되는 차트만 담을 저장소를 준비한다.</li>
<li>차트를 패키징한다.</li>
<li>공개 페이지 주소에 맞춰 <code>index.yaml</code>을 생성한다.</li>
<li>목록과 패키지를 Git에 기록하고 올린다.</li>
<li>Pages가 파일을 제공하도록 설정한다.</li>
<li>사용자 쪽에서 저장소 등록·검색·설치를 확인한다.</li>
</ol>
<p>Pages에 올린 공개 패키지는 다른 사람이 다운로드할 수 있다. 교육용 내부 설정을 그대로 올리지 않고 공개 가능한 기본값을 사용한다. 구체적인 웹 설정 메뉴는 서비스 변경에 따라 달라질 수 있다.</p>
<p><strong>nginx 파일 서버 방식</strong></p>
<ol>
<li>웹서버가 제공할 폴더에 <code>.tgz</code>와 <code>index.yaml</code>을 넣는다.</li>
<li>nginx가 해당 파일을 HTTP/HTTPS로 제공하도록 설정한다.</li>
<li>사용자가 파일 URL을 실제로 읽을 수 있는지 확인한다.</li>
<li>그 주소를 <code>helm repo add</code>로 등록한다.</li>
</ol>
<pre><code class="language-text">nginx가 제공하는 폴더
└── library/
    ├── index.yaml
    └── library-chart-0.1.0.tgz</code></pre>
<p>파일 탐색 화면이 예쁘게 보이는지는 핵심이 아니다. Helm이 <strong>정확한 URL에서 목록과 패키지를 읽을 수 있어야 한다.</strong> 교재의 내부 서버 주소나 로그인 정보 대신 이 구조만 기억하자.</p>
<h3 id="6-12-이미지-레지스트리-차트-저장소-artifact-hub">6-12. 이미지 레지스트리, 차트 저장소, Artifact Hub</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>주로 다루는 것</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>이미지 레지스트리</td>
<td>컨테이너 이미지</td>
<td>실제 실행 준비물 창고</td>
</tr>
<tr>
<td>HTTP 차트 저장소</td>
<td><code>.tgz</code> 차트와 <code>index.yaml</code></td>
<td>설치 패키지 창고</td>
</tr>
<tr>
<td>OCI 레지스트리</td>
<td>OCI 방식의 이미지·차트 등 아티팩트</td>
<td>표준 형식의 물품 창고</td>
</tr>
<tr>
<td>Artifact Hub</td>
<td>패키지 검색·소개·발견 정보</td>
<td>여러 창고의 물품을 찾는 안내 목록</td>
</tr>
</tbody></table>
<p>Artifact Hub에서 차트를 찾을 수 있지만 모든 차트 패키지를 Artifact Hub 자체가 저장하는 것은 아니다. 실제 설치 출처와 유지관리 상태를 확인해야 한다. <a href="https://artifacthub.io/docs/topics/repositories/helm-charts/">Artifact Hub의 차트 저장소 설명</a></p>
<p>OCI 기반 Helm 차트 저장·배포는 전통적인 <code>index.yaml</code> 저장소와 방식이 다르다. 같은 “저장소”라는 말이 나와도 어떤 형식인지 구분하자. <a href="https://helm.sh/docs/v3/topics/registries/">Helm의 OCI 저장소 안내</a></p>
<h3 id="6-13-umbrella-chart-여러-차트를-한-묶음으로">6-13. Umbrella Chart: 여러 차트를 한 묶음으로</h3>
<p>도서관 서비스 전체에는 웹 화면, API, 데이터베이스가 있을 수 있다.</p>
<pre><code class="language-text">library-suite/            ← 부모 차트
├── Chart.yaml
├── values.yaml
└── charts/
    ├── frontend/         ← 하위 차트
    ├── backend/          ← 하위 차트
    └── database/         ← 하위 차트</code></pre>
<p>여러 하위 차트를 묶는 부모 차트를 흔히 <strong>Umbrella Chart</strong>라고 부른다. 단품 도시락 여러 개를 가족 세트로 묶는 것에 비유할 수 있다.</p>
<p><code>Chart.yaml</code>의 <code>dependencies</code>로 의존 차트를 선언하는 방법도 있다. 아래는 구조를 설명하는 예시다.</p>
<pre><code class="language-yaml">dependencies:
  - name: backend
    version: &quot;0.1.0&quot;
    repository: &quot;file://../backend&quot;</code></pre>
<p>원하는 이름·버전의 차트가 해당 경로에 있어야 한다. 구성 후 <code>helm dependency update ./library-suite</code> 같은 명령으로 의존 차트를 준비한다.</p>
<p>부모 values에서 <code>backend:</code> 아래에 설정을 넣으면 backend 하위 차트의 설정으로 전달할 수 있다. 하위 차트에서는 자신의 <code>.Values.image</code>처럼 읽는다. 부모의 모든 값을 마음대로 보는 구조는 아니다.</p>
<h3 id="6-14-global은-공통-설정-공간이다">6-14. global은 공통 설정 공간이다</h3>
<pre><code class="language-yaml"># 부모 차트의 values.yaml 예시
global:
  registry: registry.example.com/demo

backend:
  image:
    repository: library-api
    tag: &quot;1.0.0&quot;</code></pre>
<p>backend 하위 차트의 컨테이너 템플릿 조각은 다음처럼 값을 명시적으로 조합한다.</p>
<pre><code class="language-yaml">image: &quot;{{ .Values.global.registry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}&quot;</code></pre>
<p>결과는 <code>registry.example.com/demo/library-api:1.0.0</code>이다.</p>
<p><code>global</code>은 하위 차트에서 함께 읽을 수 있는 특별한 값 영역이다. <strong>global 값을 썼다고 Helm이 이미지 주소 앞에 자동으로 붙여 주지 않는다.</strong> 템플릿이 그 값을 사용해야 한다. 네임스페이스를 global에 적는 것만으로 모든 리소스 배포 위치가 자동 변경되는 것도 아니다. <a href="https://helm.sh/docs/v3/chart_template_guide/subcharts_and_globals/">하위 차트와 global 설명</a></p>
<h3 id="6-15-이-파트-확인-질문">6-15. 이 파트 확인 질문</h3>
<p><strong>Q. 차트를 패키징하면 컨테이너 이미지도 같이 압축될까?</strong></p>
<p>A. 템플릿과 설정이 패키징된다. 실행 이미지는 별도로 관리한다.</p>
<p><strong>Q. <code>helm rollback library-dev 1</code>의 1은 이미지 버전일까?</strong></p>
<p>A. 해당 릴리스의 리비전 번호다.</p>
<p><strong>Q. 같은 차트를 dev와 prod에 설치하려면?</strong></p>
<p>A. 환경에 맞는 values와 네임스페이스를 정하고 릴리스 이름·리소스 충돌을 고려해 각각 설치한다.</p>
<hr />
<h2 id="7-helm-chart-기본-문법">7. Helm Chart 기본 문법</h2>
<h3 id="7-1-오늘-배울-문법의-역할">7-1. 오늘 배울 문법의 역할</h3>
<p>템플릿은 빈칸을 채우는 기능뿐 아니라 조건에 따라 문장을 넣고, 목록을 반복하고, 공통 부분을 재사용할 수 있다.</p>
<table>
<thead>
<tr>
<th>문법</th>
<th>하는 일</th>
<th>일상 비유</th>
</tr>
</thead>
<tbody><tr>
<td><code>{{ ... }}</code></td>
<td>값을 계산해 출력</td>
<td>신청서 빈칸 채우기</td>
</tr>
<tr>
<td><code>.</code>과 <code>$</code></td>
<td>데이터를 찾는 기준점</td>
<td>현재 서랍과 전체 서랍장</td>
</tr>
<tr>
<td><code>if / else</code></td>
<td>조건에 따라 선택</td>
<td>비가 오면 우산 챙기기</td>
</tr>
<tr>
<td><code>default</code></td>
<td>빈 값에 기본값 사용</td>
<td>메뉴 미선택 시 기본 메뉴</td>
</tr>
<tr>
<td><code>required</code></td>
<td>꼭 필요한 값 검사</td>
<td>필수 항목 없으면 신청 중단</td>
</tr>
<tr>
<td><code>|</code></td>
<td>결과를 다음 함수로 전달</td>
<td>컨베이어벨트에서 다음 작업</td>
</tr>
<tr>
<td><code>toYaml</code></td>
<td>구조화된 값을 YAML 글자로 변환</td>
<td>표의 내용을 문서로 옮기기</td>
</tr>
<tr>
<td><code>indent / nindent</code></td>
<td>들여쓰기와 줄바꿈</td>
<td>문서 정렬</td>
</tr>
<tr>
<td><code>with</code></td>
<td>데이터 기준점을 잠시 변경</td>
<td>특정 서랍을 열고 그 안에서 찾기</td>
</tr>
<tr>
<td><code>range</code></td>
<td>목록을 반복</td>
<td>출석부 이름을 하나씩 읽기</td>
</tr>
<tr>
<td><code>define / include</code></td>
<td>공통 템플릿 정의·사용</td>
<td>공통 문구를 만들어 재사용</td>
</tr>
</tbody></table>
<p>Helm은 Go 템플릿 문법과 Helm이 제공하는 함수 등을 사용한다. 모든 함수가 Go 자체의 기본 함수인 것은 아니다.</p>
<p>아래의 짧은 예제는 문법을 설명하는 <strong>조각</strong>이며, 개별 조각을 그대로 Kubernetes 리소스로 적용하지 않는다. 파트 마지막에는 로컬에서 실행할 수 있는 완전한 문법 연습 차트를 제공한다.</p>
<h3 id="7-2-점-은-현재-데이터-기준점이다">7-2. 점 <code>.</code>은 현재 데이터 기준점이다</h3>
<pre><code class="language-gotemplate">{{ .Values.image.tag }}</code></pre>
<p>보통 템플릿 시작 시 <code>.</code>에는 Values, Chart, Release 같은 정보가 들어 있다. 위 표현은 Values → image → tag를 차례로 찾는다.</p>
<p>하지만 <code>with</code>나 <code>range</code> 안에서는 <code>.</code>의 뜻이 바뀔 수 있다. “지금 누구의 정보를 보고 있는가?”를 먼저 확인하자.</p>
<h3 id="7-3-달러-는-시작-기준점이다">7-3. 달러 <code>$</code>는 시작 기준점이다</h3>
<pre><code class="language-yaml"># values 예시
books:
  - title: &quot;별자리 이야기&quot;
  - title: &quot;바다 탐험&quot;
schoolName: &quot;예시학교&quot;</code></pre>
<pre><code class="language-gotemplate">{{- range .Values.books }}
책: {{ .title }}
학교: {{ $.Values.schoolName }}
{{- end }}</code></pre>
<p>반복 안에서 <code>.</code>은 현재 책이다. 그래서 <code>.title</code>을 읽을 수 있다. 반면 학교 이름은 시작 기준점의 Values에 있으므로 <code>$.Values.schoolName</code>으로 읽는다.</p>
<p>“책 목록 중 한 권을 들여다보고 있어도 전체 학교 정보를 다시 찾아가야 할 수 있다”는 뜻이다.</p>
<p><code>$</code>는 현재 템플릿 호출에서 시작한 기준점이다. <code>include</code>에 다른 객체를 전달하면 그 호출의 기준점도 달라질 수 있다. 기본 연습에서는 전체 컨텍스트 <code>.</code>을 전달하는 패턴을 먼저 익히자.</p>
<h3 id="7-4-공백과-줄바꿈-제어">7-4. 공백과 줄바꿈 제어</h3>
<table>
<thead>
<tr>
<th>표기</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>{{ ... }}</code></td>
<td>주변 공백을 그대로 둠</td>
</tr>
<tr>
<td><code>{{- ... }}</code></td>
<td>왼쪽 공백·줄바꿈 제거</td>
</tr>
<tr>
<td><code>{{ ... -}}</code></td>
<td>오른쪽 공백·줄바꿈 제거</td>
</tr>
<tr>
<td><code>{{- ... -}}</code></td>
<td>양쪽 공백·줄바꿈 제거</td>
</tr>
</tbody></table>
<p><code>{{-</code> 뒤에는 문법에 맞는 공백을 두고 표현을 작성한다. 공백 제거 기호와 음수 표현을 혼동하지 않는다.</p>
<p>YAML에서는 공백이 구조를 나타낸다. 보기 좋게 정리하려고 줄바꿈을 너무 많이 지우면 두 항목이 붙을 수 있다.</p>
<pre><code class="language-yaml"># 잘못된 공백 제거를 설명하는 예시
name: {{- .Values.name -}}
namespace: demo</code></pre>
<p>예를 들어 <code>name</code> 값이 <code>library</code>라면 결과가 <code>name:librarynamespace: demo</code>처럼 붙을 수 있다. 이것은 의도한 두 개의 YAML 항목이 아니다.</p>
<p>우선 읽기 쉬운 줄 구조를 만들고 필요한 곳에서 <code>{{-</code>와 <code>nindent</code>를 사용하자. <strong>공백을 최대한 지우는 것이 목표가 아니라 올바른 YAML을 만드는 것이 목표</strong>다. <a href="https://helm.sh/docs/v3/chart_template_guide/control_structures/">Helm 흐름 제어와 공백 설명</a></p>
<h3 id="7-5-if-else-if-else-end">7-5. if, else if, else, end</h3>
<pre><code class="language-yaml"># values 예시
service:
  enabled: true</code></pre>
<pre><code class="language-gotemplate">{{- if .Values.service.enabled }}
# 여기에 Service 리소스 템플릿을 작성
{{- else }}
# Service를 생성하지 않는 경우
{{- end }}</code></pre>
<p>설정이 true라면 해당 부분을 만들고, false라면 다른 부분을 사용한다. <code>else if</code>로 추가 조건을 검사할 수도 있다.</p>
<p>주의할 점은 문자열 <code>&quot;false&quot;</code>와 불리언 <code>false</code>가 다르다는 것이다. <code>&quot;false&quot;</code>는 비어 있지 않은 문자열이므로 조건에서 참처럼 취급될 수 있다.</p>
<p>조건문은 false, 숫자 0, 빈 문자열, null, 빈 목록·맵 등을 비어 있거나 거짓인 값으로 취급한다. 값의 타입을 정확히 입력하자.</p>
<h3 id="7-6-default-빈-값에-기본값-사용">7-6. default: 빈 값에 기본값 사용</h3>
<pre><code class="language-gotemplate">{{ .Values.message | default &quot;안녕하세요&quot; | quote }}</code></pre>
<p><code>message</code>가 비어 있으면 <code>안녕하세요</code>를 사용하고 문자열을 따옴표로 감싼다.</p>
<p><code>default</code>는 “키가 존재하지 않을 때만” 작동하는 함수가 아니다. 숫자 0, 불리언 false, 빈 문자열 등도 빈 값으로 판단한다.</p>
<pre><code class="language-gotemplate">{{ .Values.replicaCount | default 1 }}</code></pre>
<p>이 패턴은 <code>replicaCount: 0</code>으로 의도적으로 중지하려던 설정도 1로 바꿀 수 있다. 마찬가지로 false를 기본 true로 바꿔 버리는 패턴에 주의하자. 필요한 경우 키의 존재 여부를 별도로 확인한다.</p>
<h3 id="7-7-required-꼭-필요한-값이-없으면-중단">7-7. required: 꼭 필요한 값이 없으면 중단</h3>
<pre><code class="language-gotemplate">{{ required &quot;image.repository 값을 입력하세요&quot; .Values.image.repository }}</code></pre>
<p>필수 값이 비어 있으면 메시지를 보여 주고 템플릿 처리를 실패시킨다. 이미지 주소를 입력하지 않은 채 배포가 진행되는 일을 막을 수 있다.</p>
<p><code>required</code>도 값이 비어 있는지 검사한다. 숫자 0이나 false가 유효한 필드에 무조건 사용하면 의도한 값을 거부할 수 있다.</p>
<p>이 함수는 임의의 URL이 안전한지, 이미지가 실제로 존재하는지까지 검사하지 않는다. <strong>존재·빈 값 검증과 업무 규칙 검증은 별개</strong>다. <a href="https://helm.sh/docs/v3/chart_template_guide/functions_and_pipelines/">함수와 파이프라인 설명</a></p>
<h3 id="7-8-pipeline-앞-결과를-다음-함수에-전달">7-8. Pipeline: 앞 결과를 다음 함수에 전달</h3>
<pre><code class="language-gotemplate">{{ .Values.name | upper | quote }}</code></pre>
<p>값이 <code>library</code>라면 다음과 같이 처리한다.</p>
<pre><code class="language-text">library → upper → LIBRARY → quote → &quot;LIBRARY&quot;</code></pre>
<p><code>|</code>는 “그 결과를 다음 함수로 넘겨라”라는 의미다. 파이프라인의 결과는 다음 함수의 마지막 인자로 전달된다.</p>
<pre><code class="language-gotemplate">{{ .Values.name | default &quot;library&quot; }}</code></pre>
<p>이는 <code>default &quot;library&quot; .Values.name</code>과 같은 형태로 이해할 수 있다.</p>
<p>대문자로 바꾸는 예시는 문법 연습이다. Kubernetes 리소스 이름에는 소문자 등 이름 규칙이 있으므로 모든 이름에 <code>upper</code>를 사용하면 안 된다.</p>
<h3 id="7-9-자주-쓰는-함수-전체-훑어보기">7-9. 자주 쓰는 함수 전체 훑어보기</h3>
<table>
<thead>
<tr>
<th>함수</th>
<th>의미</th>
<th>예시 결과</th>
</tr>
</thead>
<tbody><tr>
<td><code>trunc 5</code></td>
<td>지정 길이로 자르기</td>
<td><code>&quot;library&quot;</code> → <code>&quot;libra&quot;</code></td>
</tr>
<tr>
<td><code>trimSuffix &quot;-&quot;</code></td>
<td>끝의 지정 문자열 제거</td>
<td><code>&quot;library-&quot;</code> → <code>&quot;library&quot;</code></td>
</tr>
<tr>
<td><code>trimPrefix &quot;dev-&quot;</code></td>
<td>앞의 지정 문자열 제거</td>
<td><code>&quot;dev-library&quot;</code> → <code>&quot;library&quot;</code></td>
</tr>
<tr>
<td><code>upper</code></td>
<td>대문자로 변환</td>
<td><code>&quot;book&quot;</code> → <code>&quot;BOOK&quot;</code></td>
</tr>
<tr>
<td><code>lower</code></td>
<td>소문자로 변환</td>
<td><code>&quot;BOOK&quot;</code> → <code>&quot;book&quot;</code></td>
</tr>
<tr>
<td><code>title</code></td>
<td>단어 첫 글자를 대문자로 변환</td>
<td><code>&quot;hello world&quot;</code> → <code>&quot;Hello World&quot;</code></td>
</tr>
<tr>
<td><code>quote</code></td>
<td>따옴표로 감싸기</td>
<td><code>book</code> → <code>&quot;book&quot;</code></td>
</tr>
<tr>
<td><code>replace &quot;-&quot; &quot;_&quot;</code></td>
<td>문자열 바꾸기</td>
<td><code>&quot;my-book&quot;</code> → <code>&quot;my_book&quot;</code></td>
</tr>
<tr>
<td><code>indent 4</code></td>
<td>각 줄 앞에 공백 4개</td>
<td>여러 줄의 정렬</td>
</tr>
<tr>
<td><code>nindent 4</code></td>
<td>앞에 줄바꿈 후 공백 4개</td>
<td>상위 YAML 키 아래로 배치</td>
</tr>
<tr>
<td><code>default &quot;book&quot;</code></td>
<td>빈 값이면 기본값 사용</td>
<td>빈 문자열 → <code>&quot;book&quot;</code></td>
</tr>
<tr>
<td><code>required &quot;값 필요&quot;</code></td>
<td>필수값 검사</td>
<td>빈 값이면 오류</td>
</tr>
<tr>
<td><code>printf &quot;%s-%s&quot;</code></td>
<td>형식에 맞춰 문자열 조합</td>
<td><code>dev</code>, <code>book</code> → <code>dev-book</code></td>
</tr>
<tr>
<td><code>list</code></td>
<td>목록 생성</td>
<td><code>list &quot;a&quot; &quot;b&quot;</code> → 목록</td>
</tr>
<tr>
<td><code>join &quot;,&quot;</code></td>
<td>목록을 구분자로 연결</td>
<td><code>a</code>, <code>b</code> → <code>a,b</code></td>
</tr>
<tr>
<td><code>toYaml</code></td>
<td>맵·목록 등을 YAML 문자열로 변환</td>
<td>구조화된 설정을 여러 줄로 출력</td>
</tr>
</tbody></table>
<p>이름을 만들어 쓰는 조각의 예시다.</p>
<pre><code class="language-gotemplate">{{ .Release.Name | lower | trunc 63 | trimSuffix &quot;-&quot; }}</code></pre>
<p>소문자로 바꾸고 길이를 제한한 뒤 마지막의 <code>-</code>를 제거한다. 다만 길이 제한만 했다고 모든 Kubernetes 이름 규칙을 만족하거나 이름 충돌이 없어지는 것은 아니다.</p>
<h3 id="7-10-map과-list-toyaml">7-10. Map과 List, toYaml</h3>
<p>맵(Map)은 이름과 값을 연결한 구조다.</p>
<pre><code class="language-yaml">config:
  greeting: &quot;안녕하세요&quot;
  mode: &quot;study&quot;</code></pre>
<p>목록(List)은 여러 항목을 순서대로 모은 구조다.</p>
<pre><code class="language-yaml">books:
  - &quot;별자리 이야기&quot;
  - &quot;바다 탐험&quot;</code></pre>
<p><code>toYaml</code>은 이런 구조를 YAML 형식의 문자열로 변환한다. 값을 한 항목씩 직접 적지 않고 여러 설정을 출력할 때 유용하다.</p>
<h3 id="7-11-indent와-nindent">7-11. indent와 nindent</h3>
<pre><code class="language-yaml">data:
  config.yaml: |
    {{- toYaml .Values.config | nindent 4 }}</code></pre>
<p>위 조각은 ConfigMap의 <code>data</code> 안에 설정 문서 전체를 문자열로 넣는 패턴이다. 결과는 다음과 같은 구조다.</p>
<pre><code class="language-yaml">data:
  config.yaml: |
    greeting: 안녕하세요
    mode: study</code></pre>
<p>값의 순서나 따옴표 표현은 렌더링에 따라 달라질 수 있지만 데이터 의미는 같을 수 있다.</p>
<p>여기서 YAML의 <code>|</code>와 템플릿의 <code>|</code>를 구분하자.</p>
<ul>
<li><code>config.yaml: |</code>의 기호는 <strong>여러 줄을 하나의 문자열로 보관</strong>한다.</li>
<li><code>toYaml ... | nindent 4</code>의 기호는 <strong>함수 결과를 다음 함수에 전달</strong>한다.</li>
</ul>
<p><code>indent</code>는 각 줄을 밀고 <code>nindent</code>는 먼저 새 줄을 만든 뒤 각 줄을 민다. 상위 키 뒤에 값을 붙이지 않으려면 <code>nindent</code>가 편리하다.</p>
<h3 id="7-12-with-지금-보는-데이터-위치를-바꾸기">7-12. with: 지금 보는 데이터 위치를 바꾸기</h3>
<pre><code class="language-yaml"># values 예시
resources:
  limits:
    cpu: &quot;100m&quot;
    memory: &quot;128Mi&quot;</code></pre>
<pre><code class="language-yaml"># 컨테이너 템플릿의 조각
resources:
  {{- with .Values.resources }}
  {{- toYaml . | nindent 2 }}
  {{- end }}</code></pre>
<p><code>with</code> 안에서 <code>.</code>은 <code>.Values.resources</code>가 된다. 따라서 <code>.limits.cpu</code>처럼 접근하거나 <code>toYaml .</code>로 해당 객체를 출력할 수 있다. <code>with</code> 블록은 값이 비어 있지 않을 때 실행된다.</p>
<p>비유하면 전체 서랍장을 매번 말하지 않고 특정 서랍을 연 뒤 “이 안의 물건”을 보는 것이다.</p>
<p>예제의 <code>100m</code>은 CPU 0.1개에 해당하는 양이고 <code>128Mi</code>는 메모리 양이다. 여기서 limits는 사용 상한 설정이다. 자원 배치를 위한 requests와는 역할이 다르다.</p>
<h3 id="7-13-with와-else-값이-없을-때-대안">7-13. with와 else: 값이 없을 때 대안</h3>
<pre><code class="language-yaml">resources:
  {{- with .Values.resources }}
  {{- toYaml . | nindent 2 }}
  {{- else }}
  limits:
    cpu: &quot;50m&quot;
    memory: &quot;64Mi&quot;
  {{- end }}</code></pre>
<p><code>resources</code>가 비어 있지 않으면 제공한 값을 쓰고, 비어 있으면 예시 기본값을 출력한다. 실제 운영의 적절한 자원 크기는 앱을 측정해 정한다.</p>
<p><code>with</code> 안에서 릴리스 정보가 필요하면 <code>$.Release.Name</code>처럼 시작 기준점으로 돌아가야 한다. <code>with</code> 안에서 무심코 <code>.Values</code>를 읽으려 하면 현재 객체에 Values가 없어 오류가 날 수 있다.</p>
<h3 id="7-14-range-여러-값을-하나씩-반복하기">7-14. range: 여러 값을 하나씩 반복하기</h3>
<pre><code class="language-yaml"># values 예시
envs:
  - name: APP_MODE
    value: &quot;study&quot;
  - name: LOG_LEVEL
    value: &quot;info&quot;</code></pre>
<pre><code class="language-yaml"># 컨테이너 설정의 env 부분
env:
  {{- range .Values.envs }}
  - name: {{ .name }}
    value: {{ .value | quote }}
  {{- end }}</code></pre>
<p>결과는 다음과 같다.</p>
<pre><code class="language-yaml">env:
  - name: APP_MODE
    value: &quot;study&quot;
  - name: LOG_LEVEL
    value: &quot;info&quot;</code></pre>
<p>반복 중 <code>.</code>은 현재 항목이다. <code>range</code>는 목록뿐 아니라 맵도 순회할 수 있다. 인덱스와 값을 변수로 받을 수도 있다.</p>
<pre><code class="language-gotemplate">{{- range $index, $book := .Values.books }}
{{ $index }}: {{ $book }}
{{- end }}</code></pre>
<p>Kubernetes 컨테이너 환경변수에는 <code>name</code>, <code>value</code> 또는 <code>valueFrom</code> 등 실제 스키마에 맞는 필드를 사용한다. 문법 설명용 임의 키와 실제 Kubernetes 필드를 혼동하지 않는다. 비밀값은 일반 환경변수 예제에 적지 않고 필요한 Secret 참조를 구성한다.</p>
<h3 id="7-15-define과-include-반복-코드를-재사용하기">7-15. define과 include: 반복 코드를 재사용하기</h3>
<p>Deployment와 Service에서 앱 이름 생성 규칙을 따로 적으면 한쪽만 바뀔 수 있다. 공통 규칙을 <code>_helpers.tpl</code>에 정의한다.</p>
<pre><code class="language-gotemplate">{{- define &quot;library-chart.fullname&quot; -}}
{{- printf &quot;%s-%s&quot; .Release.Name .Chart.Name | trunc 63 | trimSuffix &quot;-&quot; -}}
{{- end -}}</code></pre>
<p>필요한 곳에서 호출한다.</p>
<pre><code class="language-yaml">metadata:
  name: {{ include &quot;library-chart.fullname&quot; . }}</code></pre>
<ul>
<li><code>define</code>: 이름을 붙여 템플릿 정의.</li>
<li><code>include</code>: 이름으로 호출하고 결과를 문자열로 사용.</li>
<li>마지막 <code>.</code>: 호출할 템플릿에 전달할 데이터 기준점.</li>
</ul>
<p>여러 줄의 label을 출력하는 템플릿은 다음처럼 사용한다.</p>
<pre><code class="language-yaml">metadata:
  labels:
    {{- include &quot;library-chart.selectorLabels&quot; . | nindent 4 }}</code></pre>
<p><code>include</code>의 결과를 <code>nindent</code>에 전달해 알맞은 깊이로 배치한다. 공통 템플릿 이름은 다른 차트와 충돌하지 않도록 차트 이름 등을 포함한다.</p>
<h3 id="7-16-완전한-문법-연습-차트">7-16. 완전한 문법 연습 차트</h3>
<p>조각만 보고 끝내지 말고 렌더링 결과를 확인해 보자. 이 연습은 클러스터 접속 없이 실행할 수 있다.</p>
<pre><code class="language-text">syntax-lab/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    └── configmap.yaml</code></pre>
<p><strong>① <code>syntax-lab/Chart.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v2
name: syntax-lab
version: 0.1.0
type: application</code></pre>
<p><strong>② <code>syntax-lab/values.yaml</code></strong></p>
<pre><code class="language-yaml">enabled: true
schoolName: &quot;예시학교&quot;
message: &quot;hello library&quot;
config:
  mode: &quot;study&quot;
books:
  - title: &quot;별자리 이야기&quot;
  - title: &quot;바다 탐험&quot;
resources: {}</code></pre>
<p><strong>③ <code>syntax-lab/templates/_helpers.tpl</code></strong></p>
<pre><code class="language-gotemplate">{{- define &quot;syntax-lab.name&quot; -}}
{{- printf &quot;%s-%s&quot; .Release.Name .Chart.Name | trunc 63 | trimSuffix &quot;-&quot; -}}
{{- end -}}</code></pre>
<p><strong>④ <code>syntax-lab/templates/configmap.yaml</code></strong></p>
<pre><code class="language-yaml">{{- if .Values.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include &quot;syntax-lab.name&quot; . }}
data:
  school: {{ required &quot;schoolName 값을 입력하세요&quot; .Values.schoolName | quote }}
  message: {{ .Values.message | default &quot;hello&quot; | upper | quote }}
  config.yaml: |
    {{- toYaml .Values.config | nindent 4 }}
  books.txt: |
    {{- range .Values.books }}
    {{ .title }} / {{ $.Values.schoolName }}
    {{- end }}
  resources.yaml: |
    {{- with .Values.resources }}
    {{- toYaml . | nindent 4 }}
    {{- else }}
    limits:
      cpu: &quot;50m&quot;
      memory: &quot;64Mi&quot;
    {{- end }}
{{- end }}</code></pre>
<p>이 ConfigMap에서 <code>config.yaml</code>, <code>books.txt</code>, <code>resources.yaml</code>은 문자열 데이터의 이름이다. 컨테이너의 실제 resources 설정을 적용하는 예제가 아니다. <strong>문법 결과를 한 문서에서 읽기 위한 연습용 구성</strong>이다.</p>
<p>먼저 실행하기 전에 예상해 보자.</p>
<ul>
<li>message는 대문자가 될까?</li>
<li>책 두 권이 각각 출력될까?</li>
<li>resources가 빈 맵이면 어떤 값이 출력될까?</li>
</ul>
<pre><code class="language-bash">helm lint ./syntax-lab
helm template practice ./syntax-lab --debug

# 특정 파일만 출력
helm template practice ./syntax-lab \
  --show-only templates/configmap.yaml</code></pre>
<p>주요 결과는 다음과 같다. 따옴표나 키 정렬 방식은 도구 버전에 따라 달라질 수 있다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: practice-syntax-lab
data:
  school: &quot;예시학교&quot;
  message: &quot;HELLO LIBRARY&quot;
  config.yaml: |
    mode: study
  books.txt: |
    별자리 이야기 / 예시학교
    바다 탐험 / 예시학교
  resources.yaml: |
    limits:
      cpu: &quot;50m&quot;
      memory: &quot;64Mi&quot;</code></pre>
<p>다음처럼 값을 바꿔 각각의 기능을 확인한다.</p>
<pre><code class="language-bash"># false이면 ConfigMap을 생성하지 않음
helm template practice ./syntax-lab --set enabled=false

# 빈 message이면 기본값 hello를 대문자로 출력
helm template practice ./syntax-lab --set-string message=

# 필수 schoolName이 비어 있으면 오류
helm template practice ./syntax-lab --set-string schoolName=

# resources가 있으면 with 블록의 값을 출력
helm template practice ./syntax-lab \
  --set-string resources.limits.cpu=100m \
  --set-string resources.limits.memory=128Mi</code></pre>
<p><code>schoolName</code>을 비우는 필수값 실험은 의도한 실패다. 메시지와 어느 줄에서 실패했는지 확인하면 된다.</p>
<h3 id="7-17-초보자가-자주-만나는-오류">7-17. 초보자가 자주 만나는 오류</h3>
<table>
<thead>
<tr>
<th>현상</th>
<th>먼저 살펴볼 것</th>
</tr>
</thead>
<tbody><tr>
<td>YAML 파싱 오류</td>
<td>들여쓰기, 콜론, 공백 제거로 붙은 줄</td>
</tr>
<tr>
<td>값이 비어 있음</td>
<td>values 키 이름과 템플릿 경로</td>
</tr>
<tr>
<td>with·range 안에서 <code>.Values</code> 오류</td>
<td>현재 <code>.</code>이 무엇인지, <code>$</code>가 필요한지</td>
</tr>
<tr>
<td>조건을 false로 줬는데 실행됨</td>
<td>문자열 <code>&quot;false&quot;</code>로 입력했는지</td>
</tr>
<tr>
<td>기본값이 의도한 0을 덮어씀</td>
<td>default의 빈 값 처리</td>
</tr>
<tr>
<td>Service는 있는데 접속이 안 됨</td>
<td>selector와 Pod label, 포트, Pod 준비 상태</td>
</tr>
<tr>
<td>설치 성공인데 앱 오류</td>
<td>이미지 실행 로그, 의존 서비스, 앱 설정</td>
</tr>
<tr>
<td>immutable 오류</td>
<td>기존 리소스에서 변경할 수 없는 필드를 바꿨는지</td>
</tr>
</tbody></table>
<p>실패하면 <code>helm template --debug</code>로 최종 YAML을 먼저 읽는다. 템플릿 문제인지, Kubernetes 리소스 문제인지, 앱 실행 문제인지 나눠서 확인하면 원인을 찾기 쉽다.</p>
<h3 id="7-18-이-파트-확인-질문">7-18. 이 파트 확인 질문</h3>
<p><strong>Q. <code>with</code> 안의 <code>.</code>은 항상 전체 Values일까?</strong></p>
<p>A. <code>with</code>가 선택한 객체가 현재 기준점이 된다.</p>
<p><strong>Q. <code>nindent</code>는 왜 필요할까?</strong></p>
<p>A. 새 줄을 시작하고 적절히 들여써 올바른 YAML 구조를 만들기 위해 사용한다.</p>
<p><strong>Q. <code>include</code> 뒤의 <code>.</code>은 무엇일까?</strong></p>
<p>A. 재사용 템플릿에 전달하는 데이터 컨텍스트다.</p>
<hr />
<h2 id="8-조별-토론하기">8. 조별 토론하기</h2>
<h3 id="8-1-토론에서-확인할-학습-목표">8-1. 토론에서 확인할 학습 목표</h3>
<p>교재의 1일차 본문에는 별도의 구체적인 조별 토론 결과가 기록되어 있지 않다. 아래는 <strong>오늘 배운 모든 파트를 연결하기 위해 추가한 토론 질문과 답변 예시</strong>다. 실제 수업에서 어떤 의견이 나왔는지를 주장하는 내용이 아니다.</p>
<p>도구 이름을 외우는 것보다 “왜 이 도구가 필요한가?”를 다른 사람에게 설명해 보자.</p>
<h3 id="8-2-상황-1-내-컴퓨터에서는-되는데-서버에서는-안-된다">8-2. 상황 1: 내 컴퓨터에서는 되는데 서버에서는 안 된다</h3>
<p><strong>상황</strong>: 도서관 API가 개발자의 컴퓨터에서는 실행되지만 배포 서버에서는 실패한다.</p>
<p><strong>토론 질문</strong>: 어떤 차이를 먼저 확인해야 할까?</p>
<p><strong>답변 예시</strong>: 실행 언어 버전, 필요한 파일, 환경변수, CPU 아키텍처, DB 접속, 네트워크, 권한을 확인한다. 컨테이너 이미지는 환경 재현에 도움을 주지만 외부 DB나 네트워크 설정까지 자동으로 같게 만들지는 않는다.</p>
<p><strong>연결 개념</strong>: 실습 환경, 컨테이너 이미지, 멀티 플랫폼 빌드, DevOps 협업.</p>
<h3 id="8-3-상황-2-개발과-운영의-설정-차이">8-3. 상황 2: 개발과 운영의 설정 차이</h3>
<p><strong>상황</strong>: 개발은 Pod 1개, 운영은 3개가 필요하다. 이미지와 Service 구조는 대부분 같다.</p>
<p><strong>토론 질문</strong>: Kustomize와 Helm 각각으로 어떻게 관리할까?</p>
<p><strong>답변 예시</strong>: Kustomize는 Base를 공유하고 Overlay에서 복제 수를 변경한다. Helm은 같은 Chart에 환경별 values를 전달한다. 팀의 기존 설정 형식과 공유·이력 관리 요구를 보고 선택한다.</p>
<p><strong>연결 개념</strong>: Base·Overlay, Chart·Values·Release.</p>
<h3 id="8-4-상황-3-이미지는-창고에-있는데-화면이-예전-버전이다">8-4. 상황 3: 이미지는 창고에 있는데 화면이 예전 버전이다</h3>
<p><strong>상황</strong>: CI 빌드가 성공했고 새 이미지를 push했지만 서비스 화면이 바뀌지 않았다.</p>
<p><strong>토론 질문</strong>: 빌드 성공과 배포 성공 사이에 무엇이 빠졌을까?</p>
<p><strong>답변 예시</strong>: 배포 설정이 새 이미지 태그를 가리키는지, 설정이 적용되었는지, 새 Pod가 준비되었는지 확인한다. 태그 재사용과 이미지 가져오기 정책, Service 연결, 브라우저 캐시도 구분해서 살핀다.</p>
<p><strong>연결 개념</strong>: 이미지 레지스트리, CI/CD, Kustomize images, Helm values.</p>
<h3 id="8-5-상황-4-배포-실패-후-롤백">8-5. 상황 4: 배포 실패 후 롤백</h3>
<p><strong>상황</strong>: 새 버전의 검색 기능에 오류가 있다.</p>
<p><strong>토론 질문</strong>: Helm 롤백만 하면 모든 것이 해결될까?</p>
<p><strong>답변 예시</strong>: 릴리스 이력과 이전 정상 리비전을 확인한다. 이전 이미지가 남아 있는지, 현재 DB 구조와 호환되는지, 외부 변경이 있었는지 살핀다. 배포 정의 롤백 이후 사용자 요청이 정상 처리되는지 다시 검증한다.</p>
<p><strong>연결 개념</strong>: Revision, rollback, 관찰·복구.</p>
<h3 id="8-6-상황-5-ci에서-빌드-권한-문제">8-6. 상황 5: CI에서 빌드 권한 문제</h3>
<p><strong>상황</strong>: Kubernetes의 빌드 Pod에서 Docker 데몬에 접근할 수 없다.</p>
<p><strong>토론 질문</strong>: 어떤 도구와 실행 조건을 검토할까?</p>
<p><strong>답변 예시</strong>: BuildKit·Buildah 등 빌드 방식, rootless 가능 여부, 사용자 권한, 저장소, 캐시, 플랫폼 지원을 검토한다. Kaniko는 교재의 작동 원리를 이해하되 원본 저장소의 보관 상태를 도구 선택에서 고려한다.</p>
<p><strong>연결 개념</strong>: 빌더와 런타임, 데몬, 권한, 도구 유지관리.</p>
<h3 id="8-7-상황-6-배포-설정에-인증정보가-들어-있다">8-7. 상황 6: 배포 설정에 인증정보가 들어 있다</h3>
<p><strong>상황</strong>: 누군가 PAT를 values.yaml에 적어 차트 저장소에 올리려고 한다.</p>
<p><strong>토론 질문</strong>: 공개 가능한 차트로 어떻게 바꿀까?</p>
<p><strong>답변 예시</strong>: 실제 인증값은 제거하고 필요한 Secret 이름이나 외부 주입 방식을 설정으로 받는다. 패키지에 포함될 파일과 Git 변경 내용을 확인한다. 이미 노출됐다면 해당 인증값을 폐기·재발급하고 영향을 확인해야 한다.</p>
<p><strong>연결 개념</strong>: PAT, Secret, 공개 저장소, 차트 패키징.</p>
<h3 id="8-8-토론-기록-양식">8-8. 토론 기록 양식</h3>
<p>팀의 생각을 다음 틀로 정리하면 막연한 도구 추천으로 끝나지 않는다.</p>
<pre><code class="language-text">우리가 해결할 문제:
현재 문제가 생기는 단계:
사용할 도구와 선택 이유:
필요한 환경·권한·입력값:
성공을 확인할 방법:
실패했을 때 확인·복구할 방법:
공개하면 안 되는 정보:</code></pre>
<p>성공 기준은 “명령이 끝났다”보다 구체적으로 적는다. 예를 들어 “새 이미지가 저장되고, Deployment가 그 이미지를 사용하며, 새 Pod를 통해 요청이 정상 처리된다”처럼 쓰자.</p>
<hr />
<h2 id="1일차-연결-연습-한-기능이-사용자에게-도착하는-과정">1일차 연결 연습: 한 기능이 사용자에게 도착하는 과정</h2>
<p>도서관에 새 책 검색 기능을 추가한다고 가정하자. 각 단계에서 무엇을 만드는지 말해 보면 오늘 배운 개념이 연결된다.</p>
<table>
<thead>
<tr>
<th>순서</th>
<th>작업</th>
<th>남는 결과</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>검색 기능 코드 작성·Git 기록</td>
<td>소스 코드와 변경 이력</td>
</tr>
<tr>
<td>2</td>
<td>CI에서 앱 빌드·테스트</td>
<td>검증 결과와 앱 빌드 결과물</td>
</tr>
<tr>
<td>3</td>
<td>이미지 빌더로 포장</td>
<td>컨테이너 이미지</td>
</tr>
<tr>
<td>4</td>
<td>레지스트리에 push</td>
<td>배포 환경이 가져갈 이미지</td>
</tr>
<tr>
<td>5</td>
<td>Kustomize 또는 Helm 설정 변경</td>
<td>사용할 버전과 환경별 배포 정의</td>
</tr>
<tr>
<td>6</td>
<td>Kubernetes에 적용</td>
<td>Deployment·Service 등의 리소스</td>
</tr>
<tr>
<td>7</td>
<td>상태·로그·실제 요청 확인</td>
<td>배포·서비스 정상 여부</td>
</tr>
<tr>
<td>8</td>
<td>사용자 결과를 보고 다음 개선</td>
<td>다음 계획에 쓸 피드백</td>
</tr>
</tbody></table>
<p>각 단계에서 생기는 파일과 실행 대상을 구분해 보자. 코드 저장소, 이미지 레지스트리, 차트 저장소는 서로 다른 물건을 관리한다.</p>
<h2 id="스스로-설명해-보는-복습-문제">스스로 설명해 보는 복습 문제</h2>
<ol>
<li>kubectl과 Kubernetes는 어떤 관계인가?</li>
<li>kubeconfig에서 context는 무엇을 묶는가?</li>
<li>commit과 push는 어떻게 다른가?</li>
<li>DevOps를 도구 이름만으로 설명하면 무엇이 빠지는가?</li>
<li>지속적 전달과 지속적 배포를 구분할 수 있는가?</li>
<li>앱 빌드와 이미지 빌드는 어떻게 다른가?</li>
<li>이미지 빌더와 컨테이너 런타임의 차이는?</li>
<li>build context가 필요한 이유는?</li>
<li>멀티 플랫폼 이미지에 목차가 필요한 이유는?</li>
<li>Kustomize에서 Base·Overlay·Patch는 어떤 관계인가?</li>
<li>YAML 생성과 클러스터 적용은 어떻게 다른가?</li>
<li>이미지 태그 변경 설정이 이미지 자체를 만들지는 않는 이유는?</li>
<li>Chart와 Release를 구분할 수 있는가?</li>
<li>차트 버전·앱 버전·이미지 태그·리비전은 각각 무엇인가?</li>
<li>values 파일을 여러 개 주면 어떤 값이 우선하는가?</li>
<li>Helm dry-run이 실제 앱 실행을 보장하지 않는 이유는?</li>
<li>롤백 후 DB까지 자동 복원됐다고 판단하면 안 되는 이유는?</li>
<li>with·range 안에서 <code>.</code>과 <code>$</code>는 어떻게 다른가?</li>
<li>default가 숫자 0이나 false를 바꿀 수 있는 이유는?</li>
<li>공개 차트 패키지에서 인증정보를 제외해야 하는 이유는?</li>
</ol>
<p>답은 앞의 해당 파트에 있다. 한 문장으로 먼저 설명하고 막히는 부분을 다시 읽으면 된다.</p>
<h2 id="공개-글-작성-범위와-참고-자료">공개 글 작성 범위와 참고 자료</h2>
<p>학습 범위는 제공된 교재의 1일차에 해당하는 실습 환경, DevOps, 컨테이너 빌더, Kustomize, Helm, Helm 문법이다. Helm의 두 수업 파트 구분은 시간표에 맞춰 개념·구조와 배포·관리·공유로 나누었다. 조별 토론은 학습 연결을 위한 추가 질문이다.</p>
<p>원본의 실습 인증정보·내부 서비스 주소·학생 식별 방식·개인 계정·비학습 사진은 공개 예제에 포함하지 않았다. 실습 코드는 공개용으로 새로 구성했다. 도구 상태와 교재에서 혼동하기 쉬운 설명은 본문에 연결한 공식 문서로 보완했다.</p>
<p>추가 학습 시에는 <a href="https://kubernetes.io/docs/">Kubernetes 공식 문서</a>, <a href="https://helm.sh/docs/v3/">Helm 3 공식 문서</a>, <a href="https://docs.docker.com/build/">Docker 빌드 문서</a>에서 자신의 버전과 관련된 항목을 찾아보자.</p>