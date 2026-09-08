<pre><code class="language-text">앱 소스 → 컨테이너 이미지 → 레지스트리에 저장
                              ↓
                    쿠버네티스에서 Pod로 실행
                              ↓
                    상태를 관찰하고 문제 해결</code></pre>
<p>예제의 <code>demo</code>, <code>study-web</code>, <code>registry.example.com</code>은 설명용 이름이다. 실제 수업에서는 안내받은 네임스페이스와 본인만의 소문자 리소스 이름을 사용한다. <code>&lt;...&gt;</code>가 있는 명령은 해당 값을 바꾸어 읽는 형식 예제다.</p>
<hr />
<h2 id="들어가기-앞서">들어가기 앞서</h2>
<h3 id="1쿠버네티스">1.쿠버네티스</h3>
<p>쿠버네티스(Kubernetes, K8s)는 <strong>여러 서버에서 많은 컨테이너를 자동으로 배포하고 운영하는 시스템</strong>입니다.</p>
<p>예를 들어 쇼핑몰 서버를 컨테이너 3개로 운영한다고 생각해 보겠습니다. 사람이 직접 관리하면 컨테이너가 종료될 때 다시 실행하고, 사용자가 증가하면 개수를 늘리고, 새 버전도 하나씩 교체해야 합니다. 쿠버네티스는 이런 작업을 대신 수행합니다.</p>
<p>쿠버네티스에서 사용자는 다음처럼 <strong>원하는 상태</strong>를 선언합니다.</p>
<pre><code class="language-yaml">spec:
  replicas: 3</code></pre>
<p>이는 “컨테이너를 지금 3번 실행해”라는 일회성 명령보다 <strong>“앱의 실행 복제본을 항상 3개로 유지해”</strong>라는 뜻에 가깝습니다.</p>
<p>쿠버네티스는 현재 상태를 계속 확인합니다.</p>
<pre><code class="language-text">원하는 상태: Pod 3개
현재 상태:   Pod 2개
        ↓
부족한 Pod 1개 생성
        ↓
현재 상태:   Pod 3개</code></pre>
<p>주요 기능은 다음과 같습니다.</p>
<ul>
<li>컨테이너를 실행할 서버 선택</li>
<li>장애가 발생한 앱 복구</li>
<li>여러 복제본으로 트래픽 분산</li>
<li>앱 개수 확장과 축소</li>
<li>새 버전의 점진적 배포</li>
<li>설정과 비밀 데이터 전달</li>
<li>네트워크와 저장 공간 연결</li>
</ul>
<p>여기서 <strong>Pod</strong>는 쿠버네티스가 컨테이너를 실행하는 기본 단위입니다. 일반적인 웹 앱의 관리 구조는 다음과 같습니다.</p>
<pre><code class="language-text">Deployment → ReplicaSet → Pod → 컨테이너</code></pre>
<p>식당에 비유하면 컨테이너는 요리사, Pod는 요리사가 일하는 조리 공간, 노드는 주방이 있는 매장입니다. 쿠버네티스는 중앙 운영실처럼 필요한 조리 공간의 수를 유지하고 어느 매장에 배치할지 관리합니다.</p>
<p>따라서 쿠버네티스는 단순히 컨테이너를 실행하는 도구가 아니라, <strong>컨테이너로 만든 서비스를 원하는 상태로 계속 유지하는 운영 플랫폼</strong>이라고 이해하면 됩니다.</p>
<hr />
<h2 id="2-컨테이너와-이미지">2. 컨테이너와 이미지</h2>
<h3 id="이미지">이미지</h3>
<p>앱 실행에 필요한 파일과 기본 설정을 담은 읽기 전용 묶음이다.</p>
<pre><code class="language-text">Java 실행 환경 + 애플리케이션 JAR + 기본 설정</code></pre>
<h3 id="컨테이너">컨테이너</h3>
<p>이미지를 바탕으로 실행한 프로세스다.</p>
<pre><code class="language-text">이미지 = 실행 설계도
컨테이너 = 설계도로 만든 실행 인스턴스</code></pre>
<p>동일한 이미지로 여러 컨테이너를 실행할 수 있다.</p>
<p>컨테이너 안에 생성한 파일은 컨테이너가 교체될 때 사라질 수 있다. 중요한 데이터는 외부 저장소나 볼륨에 저장해야 한다.</p>
<hr />
<h2 id="3-dockerfile">3. Dockerfile</h2>
<p>Dockerfile은 이미지를 만드는 방법을 기록한 파일이다.</p>
<pre><code class="language-dockerfile">FROM eclipse-temurin:21-jre
WORKDIR /app
COPY app.jar /app/app.jar
EXPOSE 0
EXPOSE 8080
EXPOSE 808920,80920,9200
ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app/app.jar&quot;]</code></pre>
<blockquote>
<p>위 예시에서 실제로 필요한 포 하는 포 server; <code>EXPOSE</code>는 앱의 포 포트를 문서화입니다.</p>
</blockquote>
<p>핵심 지시어는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>지시어</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>FROM</code></td>
<td>기반 이미지 선택</td>
</tr>
<tr>
<td><code>WORKDIR</code></td>
<td>작업 디렉터리 설정</td>
</tr>
<tr>
<td><code>COPY</code></td>
<td>파일을 이미지에 복사</td>
</tr>
<tr>
<td>`RUN``</td>
<td>이미지 빌드 중 중 명령 실행</td>
</tr>
<tr>
<td><code>ENV</code></td>
<td>기본 환경변수 설정</td>
</tr>
<tr>
<td><code>USER</code></td>
<td>실행 사용자 지정</td>
</tr>
<tr>
<td><code>EXPOSE</code></td>
<td>사용하는 포트 메 문서화</td>
</tr>
<tr>
<td><code>ENTRYPOINT</code></td>
<td>컨테이너의 주 실행 명령</td>
</tr>
<tr>
<td><code>CMD</code></td>
<td>기본 명령 또는 기본 인자</td>
</tr>
</tbody></table>
<p><code>ENTRYPOINT</code>는 종료 신호가 앱에 잘 전달되도록 배열 형태를 사용하는 것이 좋다.</p>
<pre><code class="language-dockerfile">ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app/app.jar&quot;]</code></pre>
<p>이미지 태그는 <code>latest</code>보다 버 커밋 ID나 버전을 활용해야 변경 내용을 추적하고 되돌리기 쉽다.</p>
<pre><code class="language-text">app:1.2.0
app:1.2.0-a1b2c3d</code></pre>
<hr />
<h2 id="4-클러스터-구조">4. 클러스터 구조</h2>
<p>쿠버네티스 클러스터는 크게 두 영역으로 나뉜다.</p>
<pre><code class="language-text">클러스터
├── 컨트롤 플레인: 결정하고 상태를 관리
└── 워커 노드: 실제 앱을 실행</code></pre>
<h3 id="컨트롤-플레인">컨트롤 플레인</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>API 서버</td>
<td>모든 관리 요청을 받는 입구</td>
</tr>
<tr>
<td>etcd</td>
<td>쿠버네티스 리소스 상태 저장</td>
</tr>
<tr>
<td>스케줄러</td>
<td>새 Pod를 실행할 노드 선택</td>
</tr>
<tr>
<td>컨트롤러 매니저</td>
<td>원하는 상태와 현재 상태의 차이 조정</td>
</tr>
</tbody></table>
<h3 id="워커-노드">워커 노드</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>kubelet</td>
<td>배정된 Pod 실행 및 상태 관리</td>
</tr>
<tr>
<td>컨테이너 런타임</td>
<td>이미지 다운로드와 컨테이너 실행</td>
</tr>
<tr>
<td>네트워크 구성요소</td>
<td>Pod 및 Service 통신 지원</td>
</tr>
</tbody></table>
<p>중요한 역할 분담은 다음과 같다.</p>
<pre><code class="language-text">스케줄러: 어디서 실행할지 결정
kubelet: 결정된 내용을 노드에서 실행
런타임: 컨테이너를 실제로 실행</code></pre>
<hr />
<h2 id="5-kubectl과-kubelet">5. kubectl과 kubelet</h2>
<h3 id="kubectl">kubectl</h3>
<p>사용자가 쿠버네티스 API 서버에 명령을 전달하는 도구다.</p>
<pre><code class="language-bash">kubectl get pods
kubectl apply -f pod.yaml
kubectl logs my-pod</code></pre>
<h3 id="kubelet">kubelet</h3>
<p>각 워커 노드에서 실행되는 에이전트다.</p>
<ul>
<li>자기 노드에 배정된 Pod 실행</li>
<li>컨테이너 상태 확인</li>
<li>프로브 실행</li>
<li>상태를 API 서버에 보고</li>
<li>재시작 정책에 따라 컨테이너 재실행</li>
</ul>
<pre><code class="language-text">사용자 → kubectl → API 서버 → kubelet → 컨테이너 런타임</code></pre>
<hr />
<h2 id="6-매니페스트와-리소스">6. 매니페스트와 리소스</h2>
<h3 id="매니페스트">매니페스트</h3>
<p>원하는 리소스를 작성한 YAML 또는 JSON 파일이다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: study-web
spec:
  containers:
    - name: web
      image: nginx:1.28</code></pre>
<h3 id="리소스">리소스</h3>
<p>매니페스트를 API 서버에 적용하여 클러스터에 생성된 객체다.</p>
<pre><code class="language-bash">kubectl apply -f pod.yaml</code></pre>
<p>파일을 수정하는 것만으로 클러스터가 변경되지는 않는다. <code>apply</code> 등을 통해 전달해야 한다.</p>
<pre><code class="language-text">YAML 파일 수정 → kubectl apply → 클러스터 리소스 변경</code></pre>
<hr />
<h2 id="7-쿠버네티스-객체의-기본-구조">7. 쿠버네티스 객체의 기본 구조</h2>
<p>대부분의 객체는 다음 구조를 가진다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: study-web
spec:
  containers:
    - name: web
      image: nginx:1.28</code></pre>
<table>
<thead>
<tr>
<th>필드</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>apiVersion</code></td>
<td>사용할 API 규격</td>
</tr>
<tr>
<td><code>kind</code></td>
<td>리소스 종류</td>
</tr>
<tr>
<td><code>metadata</code></td>
<td>이름, 네임스페이스, 라벨</td>
</tr>
<tr>
<td><code>spec</code></td>
<td>원하는 상태</td>
</tr>
<tr>
<td><code>status</code></td>
<td>쿠버네티스가 관찰한 현재 상태</td>
</tr>
</tbody></table>
<pre><code class="language-text">spec   = 원하는 상태
status = 현재 상태</code></pre>
<p>컨트롤러는 둘의 차이를 줄이는 일을 반복한다.</p>
<hr />
<h2 id="8-pod">8. Pod</h2>
<p>Pod는 쿠버네티스에서 컨테이너를 실행하는 최소 배포 단위다.</p>
<pre><code class="language-text">노드
└── Pod
    └── 컨테이너</code></pre>
<p>같은 Pod의 컨테이너는 다음을 공유한다.</p>
<ul>
<li>Pod IP</li>
<li>네트워크와 포트 공간</li>
<li>연결하도록 설정한 볼륨</li>
<li>같은 노드에서 실행되는 수명</li>
</ul>
<p>같은 Pod 안에서는 <code>localhost</code>로 서로 통신할 수 있다.</p>
<p>대부분의 웹 애플리케이션은 Pod 하나에 주 앱 컨테이너 하나를 둔다.</p>
<h3 id="pod를-직접-운영하지-않는-이유">Pod를 직접 운영하지 않는 이유</h3>
<p>독립적으로 생성한 Pod는 삭제되거나 노드가 고장 나면 대체 Pod를 유지할 상위 관리자가 없다.</p>
<p>운영에서는 일반적으로 Deployment가 Pod를 관리한다.</p>
<pre><code class="language-text">Deployment → ReplicaSet → Pod → 컨테이너</code></pre>
<hr />
<h2 id="9-pod-재생성과-컨테이너-재시작">9. Pod 재생성과 컨테이너 재시작</h2>
<p>둘은 반드시 구분해야 한다.</p>
<h3 id="컨테이너-재시작">컨테이너 재시작</h3>
<p>같은 Pod 안에서 컨테이너만 다시 실행된다.</p>
<ul>
<li>Pod 이름과 UID 유지</li>
<li><code>RESTARTS</code> 증가</li>
<li>liveness 실패나 앱 종료 등으로 발생 가능</li>
</ul>
<h3 id="pod-재생성">Pod 재생성</h3>
<p>기존 Pod를 대신하여 새로운 Pod가 만들어진다.</p>
<ul>
<li>새 Pod 이름과 UID</li>
<li>IP가 바뀔 수 있음</li>
<li>Deployment 같은 상위 관리자가 생성</li>
</ul>
<pre><code class="language-text">컨테이너 장애 → 같은 Pod에서 컨테이너 재시작 가능
Pod 삭제 → 관리자가 새로운 Pod 생성</code></pre>
<hr />
<h2 id="10-pod-상태와-ready">10. Pod 상태와 Ready</h2>
<p><code>Running</code>은 앱이 정상적으로 요청을 처리한다는 뜻이 아니다.</p>
<pre><code class="language-text">Running = 컨테이너가 실행 중
Ready   = 요청을 받을 준비가 됨</code></pre>
<pre><code class="language-bash">kubectl get pods</code></pre>
<pre><code class="language-text">NAME        READY   STATUS    RESTARTS
study-web   0/1     Running   0</code></pre>
<p>위 Pod는 컨테이너가 실행 중이지만 요청을 받을 준비는 되지 않았다.</p>
<p>자주 보는 표시:</p>
<table>
<thead>
<tr>
<th>표시</th>
<th>우선 확인할 내용</th>
</tr>
</thead>
<tbody><tr>
<td><code>Pending</code></td>
<td>노드 배치, 자원, 볼륨</td>
</tr>
<tr>
<td><code>ContainerCreating</code></td>
<td>네트워크, 볼륨, 이미지 준비</td>
</tr>
<tr>
<td><code>ImagePullBackOff</code></td>
<td>이미지 경로, 태그, 인증</td>
</tr>
<tr>
<td><code>CrashLoopBackOff</code></td>
<td>앱 로그, 실행 명령, 설정</td>
</tr>
<tr>
<td><code>Running 0/1</code></td>
<td>readinessProbe</td>
</tr>
<tr>
<td><code>OOMKilled</code></td>
<td>메모리 제한과 실제 사용량</td>
</tr>
</tbody></table>
<hr />
<h2 id="11-세-가지-프로브">11. 세 가지 프로브</h2>
<p>프로브는 kubelet이 컨테이너의 상태를 확인하는 검사다.</p>
<table>
<thead>
<tr>
<th>프로브</th>
<th>확인하는 내용</th>
<th>실패 결과</th>
</tr>
</thead>
<tbody><tr>
<td><code>startupProbe</code></td>
<td>앱의 초기 기동 완료 여부</td>
<td>컨테이너 재시작</td>
</tr>
<tr>
<td><code>livenessProbe</code></td>
<td>앱이 회복 불가능하게 멈췄는지</td>
<td>컨테이너 재시작</td>
</tr>
<tr>
<td><code>readinessProbe</code></td>
<td>지금 요청을 받아도 되는지</td>
<td>Service 연결 대상에서 제외 가능</td>
</tr>
</tbody></table>
<p>핵심 차이는 다음과 같다.</p>
<pre><code class="language-text">readiness 실패 → 트래픽 대상에서 제외
liveness 실패  → 컨테이너 재시작</code></pre>
<p>liveness 검사에 DB 상태를 포함하면 DB 장애 때 모든 앱이 재시작하는 문제가 생길 수 있다.</p>
<hr />
<h2 id="12-라벨과-셀렉터">12. 라벨과 셀렉터</h2>
<p>라벨은 리소스에 붙이는 분류표다.</p>
<pre><code class="language-yaml">metadata:
  labels:
    app: study-web</code></pre>
<p>셀렉터는 라벨로 대상을 찾는 조건이다.</p>
<pre><code class="language-bash">kubectl get pods -l app=study-web</code></pre>
<p>Deployment와 Service는 Pod 이름보다 라벨을 이용하여 대상을 찾는다. 그래서 Pod가 재생성되어 이름이 바뀌어도 연결을 유지할 수 있다.</p>
<hr />
<h2 id="13-namespace">13. Namespace</h2>
<p>Namespace는 클러스터 안의 리소스를 논리적으로 나누는 공간이다.</p>
<pre><code class="language-bash">kubectl get pods -n demo</code></pre>
<p>팀이나 환경을 구분하고 이름 충돌을 줄이며 권한과 자원 정책을 적용하는 기준으로 사용한다.</p>
<p>다만 Namespace만 만들었다고 네트워크까지 자동으로 격리되지는 않는다.</p>
<p>공유 실습 환경에서는 반드시 다음을 확인한다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl get pods -n &lt;내-네임스페이스&gt;</code></pre>
<p>본인 식별자가 붙은 리소스만 수정하거나 삭제해야 한다.</p>
<hr />
<h2 id="14-필수-kubectl-명령">14. 필수 kubectl 명령</h2>
<pre><code class="language-bash"># 목록 확인
kubectl get pods
kubectl get pods -o wide

# 상세 정보와 이벤트 확인
kubectl describe pod &lt;Pod-이름&gt;

# 현재 로그
kubectl logs &lt;Pod-이름&gt;

# 재시작 전 컨테이너 로그
kubectl logs &lt;Pod-이름&gt; --previous

# 컨테이너 안에서 명령 실행
kubectl exec -it &lt;Pod-이름&gt; -- /bin/sh

# 매니페스트 적용
kubectl apply -f app.yaml

# 적용 전 차이 확인
kubectl diff -f app.yaml

# 서버에서 저장 없이 검증
kubectl apply --dry-run=server -f app.yaml

# 리소스 삭제
kubectl delete -f app.yaml

# 권한 확인
kubectl auth can-i get pods -n &lt;네임스페이스&gt;</code></pre>
<p>장애 진단은 다음 순서를 기본으로 한다.</p>
<pre><code class="language-text">get → describe와 Events → logs → exec</code></pre>
<hr />
<h2 id="15-1일차-최종-암기-문장">15. 1일차 최종 암기 문장</h2>
<ol>
<li>쿠버네티스는 컨테이너의 원하는 상태를 유지한다.</li>
<li>이미지는 설계도이고 컨테이너는 실행 인스턴스다.</li>
<li>컨트롤 플레인은 결정하고 워커 노드는 실행한다.</li>
<li>kubectl은 사용자의 명령 도구이고 kubelet은 노드의 실행 관리자다.</li>
<li>스케줄러는 노드를 선택하고 kubelet과 런타임이 컨테이너를 실행한다.</li>
<li>매니페스트는 파일이고 리소스는 클러스터 안의 객체다.</li>
<li><code>spec</code>은 원하는 상태, <code>status</code>는 현재 상태다.</li>
<li>Pod는 컨테이너를 실행하는 최소 배포 단위다.</li>
<li>컨테이너 재시작과 Pod 재생성은 서로 다르다.</li>
<li>Running과 Ready는 서로 다르다.</li>
<li>readiness 실패는 트래픽에서 제외하고 liveness 실패는 컨테이너를 재시작한다.</li>
<li>쿠버네티스는 이름보다 라벨과 셀렉터로 대상을 연결한다.</li>
<li>문제가 생기면 <code>get → describe → logs</code> 순서로 확인한다.</li>
</ol>
<hr />
<h2 id="1장-왜-쿠버네티스인가">1장. 왜 쿠버네티스인가</h2>
<h3 id="1-1-우리가-최종적으로-만들려는-것">1-1. 우리가 최종적으로 만들려는 것</h3>
<p>개발한 웹 API를 내 노트북에서 실행하는 것과 다른 사람이 계속 사용할 수 있게 운영하는 것은 다르다. 운영하려면 서버에서 실행하고, 주소를 연결하고, 새 버전을 배포하고, 장애에도 대응해야 한다.</p>
<p>전체 과정의 목표는 <strong>앱을 이미지로 만들어 배포하고, 상태를 확인하며, 문제가 생겼을 때 고칠 수 있는 것</strong>이다. 1일차에는 이미지와 클러스터 구조, kubectl, Pod를 익힌다. 다음 날에는 여러 Pod 관리, 서비스 연결, 외부 노출, 설정과 저장소를 연결한다.</p>
<p>식당으로 비유하면 오늘은 음식 포장법과 주방 운영 구조를 배우고, 다음에는 주문을 받아 여러 조리대에 나누어 보내는 방법을 배우는 셈이다.</p>
<h3 id="1-2-물리-서버-→-가상머신-→-컨테이너">1-2. 물리 서버 → 가상머신 → 컨테이너</h3>
<p>앱을 실행하는 방식은 자원을 더 효율적으로 나누는 방향으로 발전했다. 세 방식은 지금도 함께 쓰이며, 실제로 가상머신 위에서 컨테이너를 실행하는 구성이 흔하다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>나누는 단위</th>
<th>비유</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>물리 서버</td>
<td>컴퓨터 자체</td>
<td>가게마다 건물 한 채</td>
<td>독립적이지만 자원이 남기 쉽다</td>
</tr>
<tr>
<td>가상머신, VM</td>
<td>가상 하드웨어와 게스트 OS</td>
<td>한 건물 안의 독립된 사무실</td>
<td>OS까지 분리하지만 그만큼 자원을 쓴다</td>
</tr>
<tr>
<td>컨테이너</td>
<td>격리된 프로세스 실행 환경</td>
<td>공용 시설을 이용하는 작업 공간</td>
<td>호스트 커널을 공유하여 가볍게 실행한다</td>
</tr>
</tbody></table>
<p><strong>커널</strong>은 CPU, 메모리, 파일, 네트워크 같은 자원을 관리하는 운영체제의 핵심이다. 컨테이너에는 필요한 사용자 공간 파일과 라이브러리가 들어갈 수 있지만, 보통 각 컨테이너가 자기 커널을 따로 부팅하지 않는다.</p>
<p>그래서 컨테이너는 실행 준비 비용이 적다. 다만 앱 자체가 준비되는 시간까지 항상 짧은 것은 아니다. Spring Boot 앱이 초기화와 DB 연결에 시간이 걸린다면 컨테이너에서도 그 시간은 필요하다. 커널을 공유하므로 격리와 보안 설정도 중요하다.</p>
<h3 id="1-3-컨테이너만으로는-부족한-이유-오케스트레이션">1-3. 컨테이너만으로는 부족한 이유: 오케스트레이션</h3>
<p><code>docker run</code>으로 앱 하나를 띄울 수 있다. 그런데 앱을 여러 서버에 수십 개 실행한다면 다음 일을 누가 할까?</p>
<table>
<thead>
<tr>
<th>운영 문제</th>
<th>필요한 관리</th>
</tr>
</thead>
<tbody><tr>
<td>앱 프로세스가 종료됨</td>
<td>다시 실행</td>
</tr>
<tr>
<td>서버 자체가 고장 남</td>
<td>다른 서버에 대체 실행</td>
</tr>
<tr>
<td>사용자가 늘어남</td>
<td>실행 개수 확장</td>
</tr>
<tr>
<td>복제본 주소가 바뀜</td>
<td>연결 대상 갱신</td>
</tr>
<tr>
<td>새 버전을 배포함</td>
<td>기존 버전과 차례로 교체</td>
</tr>
<tr>
<td>배포 후 문제가 생김</td>
<td>이전 버전으로 복구</td>
</tr>
<tr>
<td>설정과 비밀번호가 필요함</td>
<td>실행 환경에 적절히 전달</td>
</tr>
</tbody></table>
<p>이처럼 여러 실행 단위를 배치하고 운영하는 일을 <strong>오케스트레이션</strong>이라고 한다. 오케스트라의 지휘자가 각 악기의 연주를 조율하듯, 쿠버네티스는 여러 컨테이너의 실행을 조율한다.</p>
<p>단, 설치만 하면 모든 운영 문제가 해결되지는 않는다. 자동 확장에는 지표와 정책이 필요하고, 장애 복구에는 여유 자원과 적절한 워크로드 구성이 필요하다. 앱 버그와 데이터 백업도 별도 과제다.</p>
<h3 id="1-4-선언형-api와-조정-루프">1-4. 선언형 API와 조정 루프</h3>
<p>쿠버네티스의 핵심은 <strong>원하는 상태를 선언하면 실제 상태가 그에 가까워지도록 계속 조정한다</strong>는 것이다.</p>
<pre><code class="language-yaml"># Deployment 설정의 일부
spec:
  replicas: 3</code></pre>
<p>이 뜻은 “지금 세 번 실행해”보다 “이 앱의 복제본 수를 3개로 유지해”에 가깝다.</p>
<pre><code class="language-text">원하는 개수: 3
현재 개수: 2
    ↓ 차이를 발견
부족한 Pod 1개 생성
    ↓ 다시 관찰
현재 개수: 3</code></pre>
<p>에어컨에 목표 온도를 설정하는 것과 비슷하다. 사람이 매 순간 냉방을 켜고 끄는 대신, 제어 장치가 목표와 현재 온도를 비교한다. 이 반복 과정이 <strong>조정 루프, reconciliation loop</strong>다.</p>
<p>Deployment가 관리하는 Pod를 삭제하면 같은 Pod가 살아나는 것이 아니라 <strong>새 Pod가 만들어져 개수를 채운다</strong>. 직접 만든 독립 Pod에는 이런 복제본 관리자가 없다.</p>
<h3 id="1-5-명령형과-선언형">1-5. 명령형과 선언형</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>명령형</th>
<th>선언형</th>
</tr>
</thead>
<tbody><tr>
<td>표현</td>
<td>지금 할 행동</td>
<td>유지할 목표 상태</td>
</tr>
<tr>
<td>예</td>
<td><code>kubectl scale ... --replicas=3</code></td>
<td>YAML에 <code>replicas: 3</code>을 쓰고 apply</td>
</tr>
<tr>
<td>장점</td>
<td>빠른 실험과 즉시 조작</td>
<td>변경 검토, 기록, 재현에 유리</td>
</tr>
<tr>
<td>주의점</td>
<td>파일과 실제 상태가 달라질 수 있음</td>
<td>파일 관리와 검토 절차가 필요</td>
</tr>
</tbody></table>
<p>명령형은 “직원 두 명 더 불러”, 선언형은 “근무 인원을 세 명으로 유지해”에 해당한다.</p>
<p>같은 선언을 반복 적용해도 목표 상태가 불필요하게 누적되지 않는 성질을 <strong>멱등성</strong>이라고 한다. 다만 모든 명령형 명령이 반복 시 오류가 나는 것은 아니다. <code>create</code>는 같은 이름이 있으면 실패할 수 있지만 <code>scale</code>은 같은 값을 다시 지정할 수 있다.</p>
<p>또한 명령형으로 만든 Deployment도 생성 이후에는 조정 루프의 관리를 받는다. <strong>생성 방법과 생성된 리소스의 관리 방식은 구분해야 한다.</strong> 운영에서는 선언 파일을 Git으로 관리하면 변경 이유와 복원 기준을 남기기 좋다.</p>
<h3 id="1-6-manifest와-resource">1-6. Manifest와 Resource</h3>
<p><strong>매니페스트</strong>는 만들고 싶은 대상을 설명하는 YAML 또는 JSON 문서다. <strong>리소스 객체</strong>는 API 서버를 통해 클러스터에 등록된 실제 관리 대상이다.</p>
<p>식당의 주문서와 주문 관리 시스템에 접수된 주문으로 생각해 보자. 주문서를 집에서 수정해도 식당에는 전달되지 않는다. 매니페스트도 수정 후 <code>apply</code>해야 클러스터에 반영된다.</p>
<pre><code class="language-bash">kubectl apply -f deployment.yaml
kubectl get deployment</code></pre>
<p>로컬 파일을 삭제해도 이미 생성한 리소스는 남는다. 반대로 리소스를 삭제해도 Git에 있는 파일은 남는다. 둘의 수명은 서로 다르다.</p>
<h3 id="1-7-spec과-status">1-7. spec과 status</h3>
<table>
<thead>
<tr>
<th>필드</th>
<th>의미</th>
<th>주로 기록하는 주체</th>
</tr>
</thead>
<tbody><tr>
<td><code>spec</code></td>
<td>원하는 동작과 상태</td>
<td>사용자와 도구</td>
</tr>
<tr>
<td><code>status</code></td>
<td>관찰된 현재 상태</td>
<td>쿠버네티스 구성요소</td>
</tr>
</tbody></table>
<p>예를 들어 목표가 3개인데 준비된 복제본은 2개라면 다음처럼 이해한다.</p>
<pre><code class="language-yaml">spec:
  replicas: 3
status:
  readyReplicas: 2</code></pre>
<p>“3명을 요청했지만 업무 가능한 직원은 현재 2명”이라는 뜻이다. <code>status</code>를 3으로 적는다고 실제 앱이 건강해지지 않는다. 준비되지 않은 이유를 찾아야 한다.</p>
<p>사용자가 작성하는 필드는 <code>spec</code>만은 아니다. <code>apiVersion</code>, <code>kind</code>, <code>metadata</code>도 작성하며, ConfigMap처럼 <code>data</code>를 사용하는 리소스도 있다.</p>
<h3 id="1-8-yaml과-json">1-8. YAML과 JSON</h3>
<p>YAML과 JSON은 같은 구조의 데이터를 표현하는 서로 다른 문법이다. YAML은 사람이 읽고 작성하기 편하고, JSON은 프로그램으로 가공하기 편하다.</p>
<pre><code class="language-bash">kubectl get pod study-web -o yaml
kubectl get pod study-web -o json</code></pre>
<p>YAML을 읽을 때는 들여쓰기를 폴더 구조처럼 보면 된다.</p>
<pre><code class="language-yaml">metadata:
  name: study-web
  labels:
    app: study-web</code></pre>
<p>여기서 <code>name</code>과 <code>labels</code>는 <code>metadata</code> 안에 있고, <code>app</code>은 <code>labels</code> 안에 있다.</p>
<p>주의할 것은 들여쓰기에 탭 대신 공백을 사용하고, <code>-</code>로 목록을 표현하며, 필드가 요구하는 자료형을 맞추는 것이다. <code>replicas: 3</code>은 숫자이고, 환경변수의 <code>value: &quot;3&quot;</code>은 문자열이다. 쿠버네티스 API는 JSON 외의 직렬화 형식도 지원하지만, 입문 단계에서는 YAML과 JSON의 대응을 이해하면 충분하다.</p>
<h3 id="1-9-먼저-알아둘-리소스-8가지">1-9. 먼저 알아둘 리소스 8가지</h3>
<table>
<thead>
<tr>
<th>리소스</th>
<th>역할</th>
<th>쉬운 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Pod</td>
<td>컨테이너를 함께 실행하는 최소 배포 단위</td>
<td>작업 공간</td>
</tr>
<tr>
<td>Deployment</td>
<td>Pod 복제본과 버전 교체 관리</td>
<td>인원·교대 계획</td>
</tr>
<tr>
<td>Service</td>
<td>Pod 집합에 안정적인 연결 지점 제공</td>
<td>대표 전화번호</td>
</tr>
<tr>
<td>Ingress</td>
<td>HTTP(S) 호스트·경로에 따른 연결 규칙</td>
<td>건물 안내 데스크의 분기표</td>
</tr>
<tr>
<td>ConfigMap</td>
<td>비밀이 아닌 설정 보관</td>
<td>공용 업무 안내문</td>
</tr>
<tr>
<td>Secret</td>
<td>비밀 데이터를 위한 별도 리소스</td>
<td>접근을 통제해야 하는 보관함</td>
</tr>
<tr>
<td>PVC</td>
<td>영속 저장 공간 요청</td>
<td>창고 임대 신청서</td>
</tr>
<tr>
<td>StatefulSet</td>
<td>안정적인 식별자와 저장소 연결 등이 필요한 Pod 관리</td>
<td>고정 번호와 보관함을 가진 작업 공간</td>
</tr>
</tbody></table>
<p>Secret은 이름만으로 안전이 보장되는 금고가 아니다. 읽기 권한, 저장 시 암호화, 비밀값 배포 절차가 함께 필요하다. StatefulSet 역시 DB의 복제·백업을 자동으로 해결하는 기능은 아니다.</p>
<h3 id="1-10-리소스의-전체-관계">1-10. 리소스의 전체 관계</h3>
<p><strong>관리 관계</strong>와 <strong>사용자 요청 경로</strong>를 나누어 보면 이해하기 쉽다.</p>
<pre><code class="language-text">[관리 관계]
Deployment → ReplicaSet → Pod → 컨테이너

[웹 요청의 개념적 경로]
사용자 → 외부 진입점 → Ingress 규칙을 처리하는 구현 → 앱 Pod
                          └ Service를 통해 대상을 식별·연결

[Pod에 연결되는 것]
ConfigMap: 일반 설정
Secret: 비밀 데이터
PVC: 영속 저장 공간 요청</code></pre>
<p>Deployment와 ReplicaSet은 HTTP 요청을 전달하는 프록시가 아니다. Pod를 관리하는 객체다. Service를 통한 실제 패킷 경로도 네트워크 구현과 Ingress Controller에 따라 달라진다.</p>
<p>Pod 네트워크와 Service 가상 IP 영역을 구분해서 이해하되, Pod 네트워크가 언제나 오버레이인 것은 아니다. 클라우드 네트워크의 IP를 직접 사용하는 방식도 있다.</p>
<h3 id="1-11-노드·pod·컨테이너의-차이">1-11. 노드·Pod·컨테이너의 차이</h3>
<pre><code class="language-text">노드: 앱을 실행할 컴퓨터
└ Pod: 같은 노드에서 함께 실행할 묶음
  ├ 컨테이너 A: 실제 앱 프로세스
  └ 컨테이너 B: 보조 프로세스</code></pre>
<p>같은 Pod의 컨테이너는 네트워크 공간을 공유하므로 <code>localhost</code>로 통신할 수 있다. IP와 포트 공간을 공유하기 때문에 같은 주소와 포트에 서로 다른 서버를 동시에 바인딩할 수는 없다.</p>
<p>파일시스템은 기본적으로 각 컨테이너의 것이고, 공통 볼륨을 각각 마운트해야 그 볼륨의 파일을 공유한다. 한 Pod에 담긴 컨테이너 중 하나만 다른 노드로 이동시킬 수도 없다.</p>
<h3 id="1-12-개발자와-쿠버네티스의-책임-경계">1-12. 개발자와 쿠버네티스의 책임 경계</h3>
<p>개발자는 무엇을 실행할지, 얼마나 필요한지, 건강을 어떻게 판단할지 정의한다. 쿠버네티스는 그 설정을 바탕으로 실행하고 관찰하고 조정한다.</p>
<table>
<thead>
<tr>
<th>개발자가 정할 것</th>
<th>클러스터가 수행할 것</th>
</tr>
</thead>
<tbody><tr>
<td>이미지와 실행 명령</td>
<td>이미지 가져오기와 실행</td>
</tr>
<tr>
<td>복제본 수</td>
<td>컨트롤러를 통한 개수 유지</td>
</tr>
<tr>
<td>자원 요구와 배치 조건</td>
<td>조건에 맞는 노드 선택</td>
</tr>
<tr>
<td>건강 검사 방식</td>
<td>프로브 실행과 결과 반영</td>
</tr>
<tr>
<td>설정 및 저장 공간 요구</td>
<td>설정 전달과 볼륨 연결</td>
</tr>
</tbody></table>
<p>“프로세스는 살아 있지만 주문 처리를 못하는 상태”를 어떤 검사로 판별할지는 앱을 아는 사람이 정해야 한다. 쿠버네티스가 비즈니스 정상 여부를 저절로 이해하지는 않는다.</p>
<h3 id="1-13-관리형-쿠버네티스와-eks">1-13. 관리형 쿠버네티스와 EKS</h3>
<p>직접 구축하면 API 서버, etcd, 인증서, 가용성, 백업 등을 직접 운영해야 한다. 관리형 서비스는 그중 일부를 클라우드 제공자가 담당한다. Amazon EKS는 AWS의 관리형 쿠버네티스 서비스다.</p>
<p>건물을 직접 짓고 관리하는 방식과 관리 서비스를 포함한 공간을 사용하는 방식의 차이라고 생각하면 된다. 관리형을 사용해도 앱 설정, 접근 권한, 데이터 보호, 워크로드 자원과 배포는 계속 관리해야 한다. 노드 관리 책임은 선택한 실행 방식에 따라 달라진다.</p>
<p>가격·지원 기간·서비스별 기본 설정은 변하므로 여기서는 고정 수치로 외우지 않는다. 핵심은 <strong>무엇을 맡기고 무엇을 직접 운영하는지</strong>다.</p>
<h3 id="1-14-공유-실습-환경을-이해하기">1-14. 공유 실습 환경을 이해하기</h3>
<p>여러 사람이 같은 클러스터나 네임스페이스를 사용하면 리소스 이름과 라벨이 서로 겹칠 수 있다. 네임스페이스가 같다면 <code>study-web</code>이라는 이름도 하나의 리소스를 가리킨다.</p>
<p>실제 실습에서는 안내받은 소문자 식별자를 이름에 붙이고, 조회한 뒤 본인의 것만 수정·삭제한다. 예를 들어 <code>study-web-a17</code> 같은 형식을 사용할 수 있다. 이 글에는 실제 반 이름, 개인 식별자, 클러스터 이름, 레지스트리 주소를 넣지 않는다.</p>
<p>공유 네임스페이스의 전체 삭제는 개인 실습 정리 방법으로 사용하지 않는다.</p>
<h3 id="1-15-내-앱은-어디에-놓일까">1-15. 내 앱은 어디에 놓일까?</h3>
<p>앱을 위한 Pod와 Service는 네임스페이스 안의 리소스다. 이미지가 저장되는 레지스트리는 별도 시스템이며, 볼륨의 실체도 외부 저장 서비스일 수 있다.</p>
<pre><code class="language-text">레지스트리 ─ 이미지 다운로드 → 노드에서 컨테이너 실행
                                      ↑
네임스페이스 안의 Pod 설정 ────────────┘
       └ PVC → 저장소 시스템의 볼륨 연결</code></pre>
<p>여기서 “네임스페이스 안”은 API 객체의 논리적 소속을 뜻한다. Pod의 실제 프로세스는 노드에서 실행되고, 데이터는 연결한 저장소에 있을 수 있다.</p>
<h3 id="1-16-1장-핵심-정리">1-16. 1장 핵심 정리</h3>
<p>쿠버네티스는 원하는 상태를 계속 유지하도록 여러 구성요소가 협력하는 시스템이다. 파일과 클러스터 객체, 목표 상태와 현재 상태를 구분하면 동작이 읽히기 시작한다.</p>
<p>도입 여부는 서비스 개수 하나로 결정하지 않는다. 배포 빈도, 가용성 요구, 팀의 운영 역량, 관리 비용을 함께 본다. 작은 앱에는 단순한 실행 환경이나 PaaS가 더 적합할 수도 있다.</p>
<p><strong>이해 확인:</strong> Deployment가 관리하는 Pod 하나를 지웠는데 왜 새 Pod가 생길까? → 복제본 수가 목표보다 줄었으므로 컨트롤러가 새 Pod를 생성하기 때문이다.</p>
<hr />
<h2 id="2장-컨테이너와-이미지">2장. 컨테이너와 이미지</h2>
<h3 id="2-1-컨테이너를-만드는-리눅스-기능">2-1. 컨테이너를 만드는 리눅스 기능</h3>
<p>컨테이너를 “작은 가상머신”으로만 생각하면 자원과 장애를 이해하기 어렵다. 리눅스 컨테이너는 커널 기능으로 실행 환경을 격리한 프로세스다.</p>
<table>
<thead>
<tr>
<th>기능</th>
<th>하는 일</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>Linux namespace</td>
<td>프로세스·네트워크·마운트 등의 보이는 범위 분리</td>
<td>각 방에서 보이는 공간 분리</td>
</tr>
<tr>
<td>cgroup</td>
<td>CPU·메모리 등 사용량 관리</td>
<td>방별 전력 사용량 관리</td>
</tr>
<tr>
<td>레이어 파일시스템</td>
<td>공통 파일과 변경 내용을 겹쳐 사용</td>
<td>공통 교재에 개인 메모 덧붙이기</td>
</tr>
<tr>
<td>capabilities</td>
<td>관리자 권한을 세부 기능으로 나눔</td>
<td>마스터키 대신 필요한 열쇠만 지급</td>
</tr>
<tr>
<td>seccomp</td>
<td>사용할 수 있는 시스템 호출 제한</td>
<td>허용한 업무 요청만 접수</td>
</tr>
</tbody></table>
<p>여기서 Linux namespace와 쿠버네티스 Namespace는 서로 다른 개념이다. 전자는 운영체제의 격리 기능이고, 후자는 클러스터 리소스를 논리적으로 나누는 단위다.</p>
<p>macOS나 Windows에서 리눅스 컨테이너를 실행할 때는 일반적으로 중간에 리눅스 VM 환경이 있다. 컨테이너의 커널 공유는 그 리눅스 실행 환경을 기준으로 이해한다.</p>
<h3 id="2-2-이미지와-컨테이너의-관계">2-2. 이미지와 컨테이너의 관계</h3>
<p><strong>이미지</strong>는 실행에 필요한 파일과 기본 설정을 담은 읽기 전용 묶음이다. <strong>컨테이너</strong>는 그 이미지를 바탕으로 생성한 실행 인스턴스다.</p>
<p>쿠키 틀 하나로 쿠키 여러 개를 만들듯, 이미지 하나로 컨테이너 여러 개를 실행할 수 있다. 같은 호스트의 이미지 저장소에서 공통 레이어를 재사용해 공간을 아낄 수 있지만, 서로 다른 노드는 각각 다운로드하거나 캐시해야 한다.</p>
<pre><code class="language-text">공통 이미지 레이어
├ 컨테이너 A의 쓰기층
└ 컨테이너 B의 쓰기층</code></pre>
<p>컨테이너의 쓰기층을 영속 저장소로 믿으면 안 된다. 특히 쿠버네티스에서 컨테이너가 새로 만들어지거나 Pod가 대체되면 쓰기층의 데이터를 유지할 수 없다. 다만 Docker에서 동일한 컨테이너를 단순히 stop/start하는 것과 삭제 후 재생성하는 것은 다르다.</p>
<p>남길 데이터에는 볼륨이나 외부 저장소를 사용한다. 로그는 stdout/stderr로 내보내고 수집 시스템이 보관하게 구성한다. stdout에 기록했다는 이유만으로 영구 보관되는 것은 아니다.</p>
<h3 id="2-3-이미지-레이어와-빌드-캐시">2-3. 이미지 레이어와 빌드 캐시</h3>
<p>도시락을 만들 때 밥과 반찬이 같고 소스만 바뀌었다면 모든 재료를 다시 준비할 필요가 없다. 이미지 빌드도 변경되지 않은 단계의 결과를 캐시로 재사용할 수 있다.</p>
<p>자주 바뀌지 않는 의존성 관련 파일을 먼저 처리하고, 자주 바뀌는 소스는 뒤에서 복사하면 유리하다. 앞 단계가 변경되면 그 결과에 의존하는 뒤 단계도 다시 실행해야 하기 때문이다. 빌드 컨텍스트에서 불필요한 파일은 <code>.dockerignore</code>로 제외한다. <a href="https://docs.docker.com/build/cache/optimize/">Docker 빌드 캐시 최적화</a></p>
<pre><code class="language-text">권장하는 사고 순서
빌드 도구·의존성 정의 준비
→ 의존성 처리
→ 앱 소스 복사
→ 앱 빌드</code></pre>
<p>“Dockerfile 한 줄마다 파일 레이어가 정확히 하나 생긴다”는 설명은 단순화다. <code>RUN</code>, <code>COPY</code>처럼 파일 변경을 만드는 지시어와 <code>ENV</code>, <code>CMD</code>처럼 이미지 설정을 변경하는 지시어를 구분해야 한다.</p>
<h3 id="2-4-dockerfile-핵심-지시어">2-4. Dockerfile 핵심 지시어</h3>
<table>
<thead>
<tr>
<th>지시어</th>
<th>의미</th>
<th>기억할 점</th>
</tr>
</thead>
<tbody><tr>
<td><code>FROM</code></td>
<td>출발점이 되는 이미지</td>
<td>실행 환경 선택</td>
</tr>
<tr>
<td><code>WORKDIR</code></td>
<td>이후 작업의 기준 디렉터리</td>
<td>다음 단계에도 작업 경로 유지</td>
</tr>
<tr>
<td><code>COPY</code></td>
<td>빌드 컨텍스트의 파일 복사</td>
<td>일반 파일 복사의 기본 선택</td>
</tr>
<tr>
<td><code>ADD</code></td>
<td>복사와 추가 기능</td>
<td>원격 소스·압축 처리 등을 의도할 때 사용</td>
</tr>
<tr>
<td><code>RUN</code></td>
<td>빌드 중 명령 실행</td>
<td>컨테이너 시작 명령과 다름</td>
</tr>
<tr>
<td><code>ENV</code></td>
<td>기본 환경변수 설정</td>
<td>비밀번호를 이미지에 넣지 않기</td>
</tr>
<tr>
<td><code>EXPOSE</code></td>
<td>사용할 포트에 대한 메타데이터</td>
<td>호스트 포트 공개 기능이 아님</td>
</tr>
<tr>
<td><code>USER</code></td>
<td>이후 실행 사용자 설정</td>
<td>불필요한 root 실행 줄이기</td>
</tr>
<tr>
<td><code>ENTRYPOINT</code></td>
<td>주 실행 명령</td>
<td>exec 형식으로 앱에 종료 신호 전달</td>
</tr>
<tr>
<td><code>CMD</code></td>
<td>기본 실행 명령 또는 ENTRYPOINT의 기본 인자</td>
<td>실행 시 재정의 가능</td>
</tr>
</tbody></table>
<pre><code class="language-dockerfile">ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app/app.jar&quot;]</code></pre>
<p>대괄호 형식은 별도 셸을 거치지 않고 프로그램을 실행한다. 셸 형식은 셸이 신호를 전달하는 방식에 따라 정상 종료가 어려워질 수 있다. 셸 스크립트를 사용한다면 마지막 실행에 <code>exec</code>를 사용하는 식으로 신호 전달을 설계한다.</p>
<p><code>RUN</code>은 “도시락을 만들 때 하는 일”, <code>ENTRYPOINT</code>는 “도시락을 전달받은 뒤 실행할 일”로 구분하면 쉽다.</p>
<h3 id="2-5-spring-boot와-멀티스테이지-빌드">2-5. Spring Boot와 멀티스테이지 빌드</h3>
<p>앱을 만드는 데 필요한 도구와 실행하는 데 필요한 도구는 다르다. Java 앱은 빌드에 JDK와 Gradle 등이 필요하지만, 최종 실행 이미지에는 JRE와 JAR만 있으면 되는 경우가 많다.</p>
<p>멀티스테이지 빌드는 <strong>작업실에서 완성품을 만든 뒤, 배송 상자에는 완성품만 담는 방식</strong>이다.</p>
<pre><code class="language-dockerfile"># Gradle Wrapper를 포함한 Groovy DSL 프로젝트의 설명용 예제
FROM eclipse-temurin:21-jdk AS build
WORKDIR /workspace
COPY gradlew build.gradle settings.gradle ./
COPY gradle ./gradle
RUN chmod +x gradlew
COPY src ./src
RUN ./gradlew --no-daemon bootJar

FROM eclipse-temurin:21-jre
WORKDIR /app
RUN groupadd --system app &amp;&amp; useradd --system --gid app app
COPY --from=build /workspace/build/libs/*.jar /app/app.jar
USER app
EXPOSE 8080
ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app/app.jar&quot;]</code></pre>
<p>이 예제는 구조 설명을 위해 단순화했다. <code>build.gradle.kts</code>를 사용하는 프로젝트, 다중 모듈 프로젝트, JAR가 여러 개 생성되는 프로젝트는 복사 경로와 빌드 명령을 맞춰야 한다. 사용자 생성 명령도 베이스 이미지의 배포판에 따라 다르다.</p>
<p>빌드 캐시를 더 활용하려면 의존성 사전 처리나 BuildKit 캐시 마운트 등을 추가한다. 최종 이미지에는 소스, Gradle 캐시, 빌드용 JDK를 복사하지 않아 크기와 불필요한 도구를 줄인다. 구체적인 감소량은 프로젝트에 따라 달라진다.</p>
<h3 id="2-6-컨테이너-안의-jvm-메모리">2-6. 컨테이너 안의 JVM 메모리</h3>
<p>JVM 메모리는 객체를 담는 <strong>힙</strong>만으로 구성되지 않는다.</p>
<pre><code class="language-text">컨테이너가 사용하는 메모리
= Java 힙
+ 클래스 메타데이터
+ 스레드 스택
+ JIT 코드 캐시
+ 직접 버퍼와 기타 네이티브 메모리</code></pre>
<p>월세 예산이 100만 원인데 방값으로 전부 쓰면 관리비를 낼 수 없는 것과 같다. 컨테이너 제한이 1GiB라고 힙을 1GiB로 설정하면 힙 외 메모리가 사용할 여유가 부족하다.</p>
<pre><code class="language-bash">JAVA_TOOL_OPTIONS=&quot;-XX:MaxRAMPercentage=70&quot;</code></pre>
<p>컨테이너 제한을 인식하는 JVM에서 위 설정은 최대 힙 계산에 사용할 비율을 지정하는 예다. 70%나 75%는 모든 앱에 맞는 정답이 아니다. 스레드 수, 라이브러리, 부하를 측정하여 조절해야 한다.</p>
<p><code>-Xmx</code>도 환경변수나 시작 옵션으로 주입할 수 있으므로 값을 바꿀 때 항상 이미지를 다시 빌드해야 하는 것은 아니다. 힙이 다 차지 않았더라도 전체 메모리 사용량 때문에 <code>OOMKilled</code>가 발생할 수 있다는 점이 중요하다.</p>
<h3 id="2-7-이미지-태그-전략">2-7. 이미지 태그 전략</h3>
<p>태그는 사람이 이미지를 구분하려고 붙인 이름이다.</p>
<table>
<thead>
<tr>
<th>예</th>
<th>의미</th>
<th>주의점</th>
</tr>
</thead>
<tbody><tr>
<td><code>app:latest</code></td>
<td>latest라는 이름의 태그</td>
<td>최신 시각을 보장하지 않음</td>
</tr>
<tr>
<td><code>app:1.2.0</code></td>
<td>릴리스 버전</td>
<td>덮어쓰기를 막아야 추적 가능</td>
</tr>
<tr>
<td><code>app:a1b2c3d</code></td>
<td>소스 커밋 식별자 활용</td>
<td>같은 커밋의 빌드도 환경에 따라 달라질 수 있음</td>
</tr>
<tr>
<td><code>app:build-42</code></td>
<td>빌드 실행 번호</td>
<td>어느 파이프라인인지 함께 기록</td>
</tr>
<tr>
<td><code>app:1.2.0-a1b2c3d</code></td>
<td>버전과 소스 추적 정보</td>
<td>사람이 읽고 추적하기 편함</td>
</tr>
</tbody></table>
<p>“오늘의 도시락”이라는 이름보다 “9월 7일 제조, 42번 배치”가 문제 추적에 유리하다. 태그를 덮어쓰지 않는 정책과 배포 기록을 함께 사용해야 이전 버전 복원이 확실해진다.</p>
<h3 id="2-8-태그와-다이제스트">2-8. 태그와 다이제스트</h3>
<p>태그가 바꿔 붙일 수 있는 이름표라면, 다이제스트는 내용으로 계산한 지문이다.</p>
<pre><code class="language-text">태그 참조:       registry.example.com/team/app:1.2.0
다이제스트 참조: registry.example.com/team/app@sha256:&lt;실제 해시&gt;</code></pre>
<p>레지스트리 이미지 매니페스트나 이미지 인덱스의 다이제스트를 지정하면 정확한 내용을 가리킬 수 있다. 멀티플랫폼 이미지에서는 여러 플랫폼을 묶는 인덱스와 플랫폼별 매니페스트가 각각 존재할 수 있다.</p>
<p>태그가 삭제되어도 대상 매니페스트가 저장소에 남아 있다면 다이제스트로 받을 수 있지만, 삭제·보존 정책에 의해 실제 내용이 제거되면 받을 수 없다.</p>
<h3 id="2-9-image-id-repodigest-pod의-imageid">2-9. IMAGE ID, RepoDigest, Pod의 imageID</h3>
<table>
<thead>
<tr>
<th>값</th>
<th>무엇을 구분하는가</th>
</tr>
</thead>
<tbody><tr>
<td>태그</td>
<td>사람이 지정한 이미지 참조 이름</td>
</tr>
<tr>
<td>Docker <code>IMAGE ID</code></td>
<td>로컬 이미지 구성 정보의 식별자</td>
</tr>
<tr>
<td><code>RepoDigest</code></td>
<td>레지스트리에서 내용으로 지정한 참조</td>
</tr>
<tr>
<td>Pod 상태의 <code>imageID</code></td>
<td>런타임이 보고한 실제 사용 이미지 식별 정보</td>
</tr>
</tbody></table>
<p>이 값들은 모두 같은 종류의 문서를 해시한 값이 아니므로 문자열이 다르다고 즉시 잘못 배포된 것은 아니다. 런타임과 이미지 저장 방식에 따른 표현 차이도 살펴야 한다.</p>
<pre><code class="language-bash">kubectl get pod study-web \
  -o jsonpath='{.status.containerStatuses[0].imageID}{&quot;\n&quot;}'</code></pre>
<p>실제 배포 검증에서는 요청한 태그뿐 아니라 빌드 기록, 레지스트리 다이제스트, 런타임의 식별 정보를 함께 확인한다.</p>
<h3 id="2-10-베이스-이미지도-고정하기">2-10. 베이스 이미지도 고정하기</h3>
<p>앱 소스가 같아도 베이스 이미지가 바뀌면 라이브러리와 실행 환경이 달라질 수 있다. 베이스를 다이제스트로 고정하면 그 입력을 일정하게 만들 수 있다. 대신 보안 업데이트를 반영하려면 다이제스트를 의도적으로 갱신해야 한다. <a href="https://docs.docker.com/build/building/best-practices/">Docker 이미지 작성 권장사항</a></p>
<pre><code class="language-dockerfile"># 아래 해시는 설명용 자리표시자이므로 실제 다이제스트로 교체해야 한다.
FROM eclipse-temurin:21-jre@sha256:&lt;실제 다이제스트&gt;</code></pre>
<p>베이스 고정만으로 전체 빌드의 완전한 재현성이 보장되지는 않는다. 의존성 버전, 빌드 도구, 외부 다운로드, 타임스탬프 등 다른 입력도 영향을 준다.</p>
<h3 id="2-11-레지스트리-인증은-실행-장소마다-다르다">2-11. 레지스트리 인증은 실행 장소마다 다르다</h3>
<p>내 노트북이 레지스트리에 로그인한 것과 클러스터 노드가 이미지를 받을 권한이 있는 것은 별개다. 내가 창고 출입증을 가지고 있다고 배송 기사도 자동으로 출입할 수 있는 것은 아니다.</p>
<table>
<thead>
<tr>
<th>위치</th>
<th>인증을 마련하는 방법의 예</th>
</tr>
</thead>
<tbody><tr>
<td>로컬 Docker</td>
<td><code>docker login</code>, 자격증명 저장소</td>
</tr>
<tr>
<td>쿠버네티스</td>
<td><code>imagePullSecrets</code>, 노드 권한, 자격증명 제공 플러그인 등</td>
</tr>
</tbody></table>
<pre><code class="language-yaml"># Pod spec의 일부: 같은 네임스페이스에 해당 Secret이 있어야 한다.
imagePullSecrets:
  - name: registry-credentials</code></pre>
<p>필요한 인증 방식은 레지스트리와 클러스터 구성에 따라 다르다. <code>ImagePullBackOff</code>가 나오면 태그·경로와 함께 다운로드 주체의 권한을 확인한다. 인증 파일이나 Secret의 실제 값은 블로그에 게시하지 않는다.</p>
<h3 id="2-12-클러스터에-올리기-전에-로컬에서-실행하기">2-12. 클러스터에 올리기 전에 로컬에서 실행하기</h3>
<p>로컬 컨테이너 실행은 앱 문제와 클러스터 문제를 분리하는 데 도움이 된다.</p>
<pre><code class="language-bash">docker build -t demo-api:dev .
docker run --rm --name demo-api \
  -p 127.0.0.1:8080:8080 \
  --memory=1g --cpus=1 \
  -e JAVA_TOOL_OPTIONS=&quot;-XX:MaxRAMPercentage=70&quot; \
  demo-api:dev</code></pre>
<p>다른 터미널에서 앱이 제공하는 경로를 호출하고 로그를 확인한다.</p>
<pre><code class="language-bash">curl http://localhost:8080/
docker logs demo-api --tail 100</code></pre>
<p><code>-p</code>의 앞 포트는 내 컴퓨터, 뒤 포트는 컨테이너 포트다. <code>EXPOSE</code>만 적었다고 이 연결이 생기지는 않는다.</p>
<p>컨테이너 안의 <code>localhost</code>는 기본적으로 그 컨테이너의 네트워크 공간이다. 호스트 DB를 연결할 때 그대로 사용하면 엉뚱한 대상을 찾게 된다. CPU 아키텍처도 대상 노드와 맞춰야 한다. amd64 노드용으로 빌드할 때만 <code>--platform=linux/amd64</code>처럼 의도한 플랫폼을 지정한다.</p>
<h3 id="2-13-2장-핵심-정리">2-13. 2장 핵심 정리</h3>
<p>이미지는 실행 재료와 기본 설정이고 컨테이너는 실행 인스턴스다. 빌드 환경과 실행 환경을 분리하고, 변경을 추적할 수 있는 이미지 참조를 사용하며, 컨테이너의 임시 저장 공간에 중요한 데이터를 맡기지 않는다. 불필요한 root 권한을 줄이고, 이미지 취약점 스캔으로 의존성을 점검하며, 비밀값을 이미지에 포함하지 않는다.</p>
<p><strong>이해 확인:</strong> 힙을 넉넉하게 설정했는데 왜 OOMKilled가 날까? → 컨테이너의 전체 메모리에는 힙 외의 메모리도 포함되기 때문이다.</p>
<hr />
<h2 id="3장-쿠버네티스-아키텍처">3장. 쿠버네티스 아키텍처</h2>
<h3 id="3-1-클러스터의-전체-구조">3-1. 클러스터의 전체 구조</h3>
<p><strong>클러스터</strong>는 여러 컴퓨터와 관리 구성요소를 하나의 실행 환경처럼 묶은 것이다. 크게 결정을 담당하는 <strong>컨트롤 플레인</strong>과 앱을 실행하는 <strong>워커 노드</strong>로 나눈다.</p>
<pre><code class="language-text">사용자 / 배포 도구
       ↓ kubectl 또는 API 요청
컨트롤 플레인
  API 서버 ↔ etcd: 객체 저장
      ↕ 컨트롤러: 목표 상태로 조정
      ↕ 스케줄러: 실행할 노드 결정
       ↕
워커 노드
  kubelet → 컨테이너 런타임 → Pod의 컨테이너
  네트워크 구성요소 / 스토리지 구성요소</code></pre>
<p>물류 센터로 비유하면 컨트롤 플레인은 중앙 관리실이고, 워커 노드는 실제 작업장이다. 관리실은 주문과 배치를 관리하고 작업장은 실제 작업을 수행한다.</p>
<p>리소스 상태를 공유하는 중심은 API 서버지만 <strong>모든 종류의 통신이 API 서버를 통과하는 것은 아니다</strong>. 앱 간 통신, kubelet과 런타임 간 통신, 클라우드 API 호출은 각각의 경로를 가진다.</p>
<h3 id="3-2-컨트롤-플레인-구성요소">3-2. 컨트롤 플레인 구성요소</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
<th>물류 센터 비유</th>
</tr>
</thead>
<tbody><tr>
<td>kube-apiserver</td>
<td>요청을 받고 인증·인가·검증 후 객체 처리</td>
<td>접수 창구</td>
</tr>
<tr>
<td>etcd</td>
<td>클러스터 API 객체의 상태를 저장</td>
<td>중앙 장부</td>
</tr>
<tr>
<td>kube-scheduler</td>
<td>아직 배정되지 않은 Pod의 노드 선택</td>
<td>작업장 배치 담당</td>
</tr>
<tr>
<td>kube-controller-manager</td>
<td>여러 컨트롤러의 조정 루프 실행</td>
<td>분야별 운영 관리자</td>
</tr>
<tr>
<td>cloud-controller-manager</td>
<td>클라우드와 관련된 제어 기능 연결</td>
<td>외부 시설 담당 창구</td>
</tr>
</tbody></table>
<p>스케줄러는 직접 컨테이너를 실행하지 않는다. “이 Pod는 이 노드에서 실행한다”는 배정 결과를 기록한다. 이후 해당 노드의 kubelet과 런타임이 실행을 담당한다. <a href="https://kubernetes.io/docs/concepts/overview/components/">쿠버네티스 구성요소</a></p>
<p>etcd에는 앱의 주문 테이블이나 업로드 파일이 자동으로 저장되는 것이 아니다. etcd는 쿠버네티스 객체의 저장소이고, 앱 데이터는 별도 DB와 저장소가 맡는다.</p>
<h3 id="3-3-워커-노드-구성요소">3-3. 워커 노드 구성요소</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
<th>자주 연결되는 증상</th>
</tr>
</thead>
<tbody><tr>
<td>kubelet</td>
<td>배정된 Pod의 실행과 상태 관리, 프로브 수행</td>
<td>컨테이너 준비·건강 검사 문제</td>
</tr>
<tr>
<td>컨테이너 런타임</td>
<td>이미지 다운로드, 컨테이너 생성·실행</td>
<td>이미지와 실행 단계 오류</td>
</tr>
<tr>
<td>kube-proxy 또는 대체 구현</td>
<td>Service 연결을 위한 데이터 경로 구성</td>
<td>Service 접근 문제</td>
</tr>
<tr>
<td>CNI 플러그인</td>
<td>Pod 네트워크 연결</td>
<td>IP 할당과 Pod 통신 문제</td>
</tr>
<tr>
<td>CSI 드라이버</td>
<td>저장소 프로비저닝·연결·마운트 지원</td>
<td>볼륨 연결 문제</td>
</tr>
<tr>
<td>로그·모니터링 에이전트</td>
<td>관찰 데이터 수집</td>
<td>로그와 지표 누락</td>
</tr>
</tbody></table>
<p>kubelet은 API의 변경 감시와 동기화 과정을 통해 담당 Pod의 상태를 맞춘다. 단순히 매번 전체 정보를 새로 조회하는 방식으로만 이해하지 않는다.</p>
<h3 id="3-4-kubectl-apply-한-줄의-여정">3-4. <code>kubectl apply</code> 한 줄의 여정</h3>
<p>Deployment 매니페스트를 적용한다고 가정하자.</p>
<ol>
<li>kubectl이 파일을 읽고 API 서버에 요청한다.</li>
<li>API 서버가 요청을 확인하고 Deployment 객체를 저장한다.</li>
<li>Deployment 컨트롤러가 필요한 ReplicaSet을 관리한다.</li>
<li>ReplicaSet 컨트롤러가 부족한 Pod 객체를 만든다.</li>
<li>스케줄러가 Pod를 실행할 노드를 선택한다.</li>
<li>노드의 kubelet이 런타임과 협력해 컨테이너를 실행한다.</li>
<li>kubelet이 상태와 프로브 결과를 보고한다.</li>
<li>일치하는 Service가 있다면 관련 컨트롤러가 연결 대상 정보를 갱신한다.</li>
</ol>
<p>각 단계는 독립된 구성요소가 상태 변화를 보고 진행한다. 하나의 프로그램이 위에서 아래로 모든 작업을 직접 호출하는 단일 함수가 아니다. API 요청 방식도 생성·갱신과 apply 방식에 따라 달라지므로 항상 POST라고 외우지 않는다.</p>
<pre><code class="language-text">Pending          → 아직 배치·실행 준비를 끝내지 못했나?
ImagePullBackOff → 이미지 다운로드에서 막혔나?
CrashLoopBackOff → 실행한 앱이 반복해서 종료되나?
Running, 0/1    → 실행은 했지만 준비 검사를 통과하지 못했나?</code></pre>
<p>표는 진단 방향이다. 상태 이름만으로 원인을 확정하지 말고 Events와 로그를 확인한다.</p>
<h3 id="3-5-pod-하나를-직접-만들-때의-여정">3-5. Pod 하나를 직접 만들 때의 여정</h3>
<p><code>kubectl run</code>으로 독립 Pod를 만들면 Deployment와 ReplicaSet 단계는 없다.</p>
<pre><code class="language-text">Pod 생성 요청 → API 저장 → 노드 배정 → 컨테이너 준비·실행 → 상태 보고</code></pre>
<p>그래서 독립 Pod를 삭제해도 자동으로 복제본을 채워 줄 ReplicaSet이 없다. 다만 Pod가 남아 있는 동안 개별 컨테이너가 종료되면 kubelet이 재시작 정책에 따라 컨테이너를 다시 실행할 수 있다.</p>
<p>또한 Pod를 만들었다고 Service가 자동으로 만들어지지는 않는다. Service가 있어야 그 선택 조건과 준비 상태에 따라 서비스 연결 대상에 포함될 수 있다.</p>
<h3 id="3-6-오브젝트의-공통-구조">3-6. 오브젝트의 공통 구조</h3>
<pre><code class="language-yaml">apiVersion: apps/v1 #어느 api 그룹/버전인가
kind: Deployment # 무슨 종류인가
metadata:     #누구인가  
  name: study-web
  labels:
    app: study-web
spec:    #원하는 상태- 사람이 작성
  replicas: 2
  selector:
    matchLabels:
      app: study-web
  template:
    metadata:
      labels:
        app: study-web
    spec:
      containers:
        - name: web
          image: nginx:1.28</code></pre>
<p>읽는 순서는 “어떤 규격인지 → 어떤 종류인지 → 이름이 무엇인지 → 어떤 상태를 원하는지”다.</p>
<p><code>template</code>은 새 Pod를 만들 때 사용할 틀이다. 그 안의 <code>metadata</code>는 Pod의 이름표이고, 그 안의 <code>spec</code>은 Pod의 실행 설정이다. 바깥 Deployment의 라벨과 안쪽 Pod의 라벨은 서로 다른 위치에 있다.</p>
<p>위 이미지 태그는 문법 학습용 예시다. 실제 실행 시에는 해당 환경에서 허용하고 가져올 수 있는 이미지 참조를 사용한다.</p>
<h3 id="3-7-라벨과-셀렉터">3-7. 라벨과 셀렉터</h3>
<p><strong>라벨</strong>은 리소스에 붙이는 분류표이고, <strong>셀렉터</strong>는 그 분류표로 대상을 고르는 조건이다.</p>
<pre><code class="language-yaml"># Pod에 붙이는 라벨
labels:
  app: study-web
  tier: frontend</code></pre>
<pre><code class="language-bash">kubectl get pods -l app=study-web
kubectl get pods -l 'app=study-web,tier=frontend'</code></pre>
<p>도서관에서 책의 일련번호가 달라도 “프로그래밍” 분류로 찾을 수 있는 것과 같다. Pod 이름이 재생성 때 바뀌어도 라벨이 같으면 Service가 새 Pod를 찾을 수 있다.</p>
<p>Deployment의 <code>selector.matchLabels</code>는 Pod 템플릿의 라벨과 맞아야 한다. Service 셀렉터도 대상으로 삼을 Pod 라벨과 맞아야 한다. 앱이 실행 중인데 연결 대상이 없다면 이 연결을 확인한다. Deployment의 셀렉터는 생성 후 변경할 수 없는 필드다.</p>
<h3 id="3-8-namespace로-논리-공간-나누기">3-8. Namespace로 논리 공간 나누기</h3>
<p>Namespace는 하나의 클러스터 안에서 리소스를 논리적으로 묶는 공간이다. 팀별 공간, 개발·검증 환경 등을 나눌 수 있다.</p>
<pre><code class="language-bash">kubectl get pods -n demo
kubectl api-resources --namespaced=false</code></pre>
<p>Pod, Deployment, Service는 네임스페이스에 속한다. Node, PV, StorageClass처럼 클러스터 범위에 속하는 리소스도 있다.</p>
<p>네임스페이스는 이름과 권한·쿼터를 나누는 기반이지만, 이것만으로 네트워크가 차단되지는 않는다. 네트워크 격리에는 NetworkPolicy와 이를 집행하는 구현 등이 필요하다. 비유하면 교실을 나눴다고 출입문에 자동으로 잠금장치가 생기는 것은 아니다.</p>
<h3 id="3-9-추가로-필요한-애드온">3-9. 추가로 필요한 애드온</h3>
<table>
<thead>
<tr>
<th>애드온</th>
<th>제공하는 기능</th>
</tr>
</thead>
<tbody><tr>
<td>CNI</td>
<td>일반적인 Pod 네트워크 구성</td>
</tr>
<tr>
<td>CoreDNS</td>
<td>Service 이름 등을 IP로 해석</td>
</tr>
<tr>
<td>metrics-server</td>
<td><code>kubectl top</code> 등에 필요한 자원 사용 지표</td>
</tr>
<tr>
<td>Ingress Controller</td>
<td>Ingress 규칙을 실제 연결 동작으로 구현</td>
</tr>
<tr>
<td>CSI 드라이버</td>
<td>외부 저장소와 쿠버네티스 연결</td>
</tr>
<tr>
<td>cert-manager</td>
<td>인증서 발급·갱신 자동화</td>
</tr>
<tr>
<td>로그 수집기</td>
<td>여러 노드의 로그 수집·전달</td>
</tr>
</tbody></table>
<p>“쿠버네티스가 있으니 그 기능도 있겠지”라고 가정하지 말고 실제 구성을 확인한다. 예를 들어 <code>top</code>이 실패했다고 앱 CPU 사용량이 0인 것은 아니다. 지표를 제공하는 구성요소가 없거나 문제가 있을 수 있다.</p>
<p>내부 애드온 설치 여부나 보안 정책 설정은 공개 글에 그대로 나열하지 않고, 위와 같은 일반적인 역할로 설명한다.</p>
<h3 id="3-10-3장-핵심-정리">3-10. 3장 핵심 정리</h3>
<p>클러스터를 읽을 때는 “요청을 받는 곳, 상태를 저장하는 곳, 배치를 정하는 곳, 실행하는 곳”을 구분한다. 리소스 간 연결은 이름뿐 아니라 라벨과 셀렉터로 형성된다.</p>
<p><strong>이해 확인:</strong> Pod가 오래 Pending이라면? → <code>kubectl describe pod &lt;이름&gt;</code>으로 스케줄링·볼륨·이미지 등 어느 준비 단계에서 막혔는지 확인한다.</p>
<hr />
<h2 id="4장-클러스터-내부-동작">4장. 클러스터 내부 동작</h2>
<h3 id="4-1-eks-아키텍처-그림-읽기">4-1. EKS 아키텍처 그림 읽기</h3>
<p>복잡한 클라우드 그림은 다섯 층으로 나누면 읽기 쉽다. 교재의 참조 아키텍처는 여러 구성 선택지를 담은 그림이며, 모든 클러스터가 그 요소를 전부 사용하는 것은 아니다.</p>
<table>
<thead>
<tr>
<th>층</th>
<th>구성 예</th>
<th>질문</th>
</tr>
</thead>
<tbody><tr>
<td>외부 접근</td>
<td>사용자, DNS, 인증서, 웹 방화벽</td>
<td>사용자가 어떤 이름으로 들어오는가?</td>
</tr>
<tr>
<td>진입점</td>
<td>ALB·NLB 같은 로드밸런서</td>
<td>외부 요청을 어디로 전달하는가?</td>
</tr>
<tr>
<td>네트워크와 실행</td>
<td>VPC, 서브넷, 노드 그룹, Pod</td>
<td>앱은 어디에서 실행되는가?</td>
</tr>
<tr>
<td>관리</td>
<td>EKS 컨트롤 플레인</td>
<td>누가 배포와 상태를 관리하는가?</td>
</tr>
<tr>
<td>부가 기능</td>
<td>IAM, 스토리지, 로그·지표·추적</td>
<td>권한·데이터·관찰은 누가 맡는가?</td>
</tr>
</tbody></table>
<p><strong>VPC</strong>는 클라우드의 가상 네트워크이고 <strong>서브넷</strong>은 그 안의 IP 구간이다. <strong>AZ</strong>는 장애를 분리하도록 구성한 가용 영역이다. 여러 AZ에 분산한다는 것은 한 장소의 장애가 모든 복제본에 동시에 영향을 주지 않도록 설계한다는 의미다.</p>
<p>NAT Gateway는 사설 네트워크의 외부 통신에, 관리 접속 경로는 운영자가 시스템을 관리하는 데 쓰인다. 노드 그룹이나 서버리스 실행 방식은 앱을 실행할 기반의 선택지다. IAM 역할은 AWS 자원을 사용할 권한과 관련된다.</p>
<p>저장소 그림의 EBS·EFS·S3는 성격이 서로 다르고, 관찰 영역의 로그·지표·분산 추적도 서로 다른 질문에 답한다. 오늘은 제품 이름보다 <strong>역할과 책임의 경계</strong>를 파악한다.</p>
<h3 id="4-2-일반-쿠버네티스-아키텍처-그림-읽기">4-2. 일반 쿠버네티스 아키텍처 그림 읽기</h3>
<p>클라우드 제품 이름을 지우면 기본 구조는 다음처럼 남는다.</p>
<pre><code class="language-text">관리 요청: kubectl / UI / CI → API 서버
상태 공유: API 서버 ↔ etcd, 컨트롤러, 스케줄러, kubelet
실행 요청: kubelet → CRI 런타임
앱 통신:  Pod ↔ 네트워크 구현 ↔ 다른 Pod / 외부 시스템
저장소:   스토리지 제어 구성요소 ↔ CSI 드라이버 ↔ 저장소</code></pre>
<p>관리 경로를 <strong>제어 경로</strong>, 실제 사용자 요청이 흐르는 경로를 <strong>데이터 경로</strong>라고도 부른다. 배포 요청이 실패해도 기존 웹 요청은 처리될 수 있는 이유가 이 구분에 있다.</p>
<h3 id="4-3-api-서버가-요청을-처리하는-관문">4-3. API 서버가 요청을 처리하는 관문</h3>
<p>학교 시설을 이용할 때 신분 확인, 이용 권한 확인, 신청서 검사, 시설 규칙 검사를 거치는 것과 비슷하다.</p>
<table>
<thead>
<tr>
<th>관문</th>
<th>확인하는 내용</th>
<th>오류의 예</th>
</tr>
</thead>
<tbody><tr>
<td>요청 해석</td>
<td>HTTP 요청과 데이터가 해석 가능한가</td>
<td>Bad Request</td>
</tr>
<tr>
<td>인증, Authentication</td>
<td>누구의 요청인가</td>
<td>Unauthorized</td>
</tr>
<tr>
<td>인가, Authorization</td>
<td>이 동작을 할 권한이 있는가</td>
<td>Forbidden</td>
</tr>
<tr>
<td>변경 Admission</td>
<td>기본값·정책에 따라 보완할 내용이 있는가</td>
<td>Webhook 오류</td>
</tr>
<tr>
<td>스키마 검증</td>
<td>필드 이름과 자료형이 맞는가</td>
<td>잘못된 필드·자료형</td>
</tr>
<tr>
<td>검증 Admission</td>
<td>추가 정책을 만족하는가</td>
<td>보안 정책·쿼터 거부</td>
</tr>
<tr>
<td>저장</td>
<td>승인된 객체를 영속화</td>
<td>저장소 관련 오류</td>
</tr>
</tbody></table>
<p>이는 역할을 이해하기 위한 구분이다. 실제 처리에는 디코딩, 기본값 처리, 검증 등이 결합되므로 내부 구현을 고정된 일곱 함수로 외우지는 않는다.</p>
<p><strong>인증과 인가의 차이:</strong> 학생증을 확인하는 것은 인증이고, 그 학생에게 실험실 사용 권한이 있는지 확인하는 것은 인가다. <code>Forbidden</code>도 RBAC 외 정책 문맥에서 나타날 수 있으므로 메시지 전체를 읽어야 한다.</p>
<h3 id="4-4-컨트롤러-매니저-안의-여러-관리자">4-4. 컨트롤러 매니저 안의 여러 관리자</h3>
<p>컨트롤러는 자기가 맡은 객체를 관찰하고 필요한 변경을 API 서버에 요청한다.</p>
<table>
<thead>
<tr>
<th>컨트롤러</th>
<th>담당 업무</th>
</tr>
</thead>
<tbody><tr>
<td>Deployment</td>
<td>ReplicaSet과 배포 상태 관리</td>
</tr>
<tr>
<td>ReplicaSet</td>
<td>필요한 Pod 수 유지</td>
</tr>
<tr>
<td>Node 관련 제어</td>
<td>노드 상태 감지와 장애 대응에 관여</td>
</tr>
<tr>
<td>EndpointSlice</td>
<td>Service 연결 대상 정보 갱신</td>
</tr>
<tr>
<td>Namespace</td>
<td>네임스페이스 삭제 시 내부 리소스 정리</td>
</tr>
<tr>
<td>PersistentVolume 관련 제어</td>
<td>PV·PVC 연결 등의 수명 관리</td>
</tr>
<tr>
<td>Job·CronJob</td>
<td>완료해야 할 작업과 예약 실행 관리</td>
</tr>
</tbody></table>
<p>교재의 Endpoints는 Service 뒤에 있는 실제 연결 대상 목록을 이해하기 위한 개념이다. 이 글의 조회 예제는 EndpointSlice를 사용한다. 목록과 Ready 조건은 관련되지만 실제 포함·전달 동작에는 Service 설정과 종료 상태 등의 조건도 영향을 준다.</p>
<p>컨트롤러는 각자 전문 업무를 맡은 직원들이다. 한 직원이 전체 클러스터를 직접 실행하는 구조가 아니다.</p>
<h3 id="4-5-스케줄러-후보를-거르고-점수를-매기기">4-5. 스케줄러: 후보를 거르고 점수를 매기기</h3>
<p>새 직원을 어느 지점에 배치할지 정한다고 생각해 보자. 일단 자리가 없는 지점은 제외하고, 남은 지점 중 조건이 좋은 곳을 선택한다.</p>
<p><strong>필터링</strong>은 배치 가능한 노드만 남기는 과정이고, <strong>스코어링</strong>은 가능한 후보 중 더 적합한 노드에 점수를 주는 과정이다.</p>
<table>
<thead>
<tr>
<th>판단 재료</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>requests</td>
<td>Pod가 요청한 자원을 수용할 수 있는가</td>
</tr>
<tr>
<td>nodeSelector·필수 affinity</td>
<td>반드시 맞아야 하는 노드 조건</td>
</tr>
<tr>
<td>taint·toleration</td>
<td>해당 노드의 제한을 허용하는가</td>
</tr>
<tr>
<td>Pod 수 제한</td>
<td>노드에 더 배치할 수 있는가</td>
</tr>
<tr>
<td>볼륨 토폴로지</td>
<td>저장소를 해당 위치에서 연결할 수 있는가</td>
</tr>
<tr>
<td>선호 affinity·분산 선호</td>
<td>가능한 후보 중 어느 쪽이 유리한가</td>
</tr>
</tbody></table>
<p>affinity와 topology spread는 설정에 따라 필수 조건 또는 선호 조건이 될 수 있다. toleration이 있다고 그 노드로 배치가 보장되는 것도 아니다.</p>
<p>모든 후보가 제외되면 Pod가 Pending으로 남을 수 있다. <code>describe</code>의 <code>0/N nodes are available</code> 뒤에 어떤 조건이 걸렸는지 확인한다.</p>
<h3 id="4-6-etcd-클러스터의-장부">4-6. etcd: 클러스터의 장부</h3>
<p>etcd는 분산 키-값 저장소다. 여러 서버가 일관된 데이터를 유지하기 위해 합의 과정을 사용한다. 쿠버네티스의 리소스 선언과 보고된 상태가 여기에 보관된다.</p>
<p>다른 구성요소는 보통 etcd를 직접 수정하지 않고 API 서버를 통해 객체를 다룬다. API의 <strong>watch</strong>는 변경을 계속 전달받는 방식이다. 주문 현황판을 매초 새로 출력하기보다 새 주문 알림을 받는 것에 가깝다. 실제 클라이언트는 초기 조회, watch 재연결, 동기화도 수행한다.</p>
<p>etcd가 일시적으로 중단되는 것과 데이터가 영구 소실되는 것은 다르다. 전자는 제어 기능을 막을 수 있고, 후자는 상태 복구 문제로 이어진다. 이미 실행 중인 프로세스가 즉시 모두 종료되는 것은 아니다. 또한 etcd 백업과 앱 DB 백업은 별개다.</p>
<h3 id="4-7-클라우드-연동-담당자를-나누어-이해하기">4-7. 클라우드 연동: 담당자를 나누어 이해하기</h3>
<p>쿠버네티스의 선언이 실제 클라우드 자원 생성으로 이어질 수 있다. 그러나 관련 업무를 전부 cloud-controller-manager 하나가 하는 것은 아니다.</p>
<table>
<thead>
<tr>
<th>연동 종류</th>
<th>주로 관여하는 구성요소</th>
</tr>
</thead>
<tbody><tr>
<td>클라우드 노드 정보·일부 경로·Service 연동</td>
<td>cloud-controller-manager</td>
</tr>
<tr>
<td>클라우드 로드밸런서</td>
<td>클라우드 제공자 또는 전용 LB 컨트롤러</td>
</tr>
<tr>
<td>블록·파일 볼륨</td>
<td>CSI 드라이버와 스토리지 컨트롤러</td>
</tr>
<tr>
<td>Pod IP와 네트워크</td>
<td>CNI 플러그인</td>
</tr>
</tbody></table>
<p>cloud-controller-manager는 클라우드 관련 제어를 쿠버네티스 핵심 기능에서 분리한다. 실제 담당 범위는 클러스터 구성에 따라 확인한다. <a href="https://kubernetes.io/docs/concepts/architecture/cloud-controller/">Cloud Controller Manager 공식 설명</a></p>
<p>리소스 삭제가 외부 자원 삭제로 이어지는지도 정책과 컨트롤러에 따라 달라진다. 공유 로드밸런서를 사용하는 Ingress 하나를 삭제했다고 공용 로드밸런서가 삭제되는 것은 아니다. PVC도 회수 정책에 따라 실제 데이터의 수명이 달라진다.</p>
<h3 id="4-8-kubelet이-실제로-하는-일">4-8. kubelet이 실제로 하는 일</h3>
<p>kubelet은 노드의 현장 관리자다. 자기 노드에 배정된 Pod를 실행하고 관찰하기 위해 다른 구성요소와 협력한다.</p>
<ul>
<li>런타임에 이미지와 컨테이너 실행을 요청한다.</li>
<li>프로브를 실행하여 시작·생존·준비 상태를 확인한다.</li>
<li>Pod와 노드 상태를 API 서버에 보고한다.</li>
<li>볼륨의 노드 측 연결·마운트 과정에 관여한다.</li>
<li>이미지 정리와 노드 자원 압박 대응을 수행한다.</li>
</ul>
<p>따라서 HTTP 프로브는 보통 kubelet이 Pod 주소로 직접 수행한다. 외부 Ingress를 통해 사용자가 호출하는 경로와 같지 않다. 프로브 성공은 중요한 정보지만 전체 외부 접속 경로가 정상이라는 증거는 아니다.</p>
<h3 id="4-9-런타임과-cri-oci의-차이">4-9. 런타임과 CRI, OCI의 차이</h3>
<p><strong>CRI</strong>는 kubelet이 컨테이너 런타임과 대화하는 규격이다. <strong>OCI</strong>는 이미지 형식과 실행 환경 등에 관한 표준을 정의한다.</p>
<pre><code class="language-text">Docker로 이미지 생성
         ↓ 이미지 호환 규격
레지스트리에 저장
         ↓
kubelet → CRI → containerd 등의 런타임 → 컨테이너 실행</code></pre>
<p>Docker Engine을 사용하지 않는 노드에서도 Docker로 만든 호환 이미지를 실행할 수 있다. 워드프로세서를 바꾸어도 호환 문서 형식을 읽을 수 있는 것과 비슷하다.</p>
<p>노드 런타임을 직접 진단할 때는 <code>crictl</code> 같은 도구를 사용할 수 있다. 일반 앱 개발자는 우선 <code>kubectl get</code>, <code>describe</code>, <code>logs</code>로 진단하고, 노드 직접 접근은 주어진 권한과 운영 절차에 따른다.</p>
<h3 id="4-10-kube-proxy와-service-연결-규칙">4-10. kube-proxy와 Service 연결 규칙</h3>
<p>일반적인 ClusterIP Service는 Pod 집합에 가상 IP를 제공한다. 해당 주소로 보낸 트래픽이 실제 Pod로 전달되도록 노드의 데이터 경로를 구성해야 한다.</p>
<pre><code class="language-text">Service의 가상 주소:포트
       ↓ 연결 규칙
Pod A:앱 포트 또는 Pod B:앱 포트</code></pre>
<p>kube-proxy는 Service와 EndpointSlice 변화를 보고 규칙을 갱신하는 역할을 한다. 구체적인 구현에는 iptables, nftables 등의 방식이 있으며, kube-proxy를 대체하는 네트워크 구현도 있다.</p>
<pre><code class="language-bash">kubectl get service study-web
kubectl get endpointslice -l kubernetes.io/service-name=study-web</code></pre>
<p>Service라는 API 객체가 없는 것은 아니다. API 객체는 실제로 존재하며, 패킷 전달 방식이 별도의 전용 서버 프로세스가 아닐 수 있다는 의미다. 연결 대상이 비었다면 라벨과 Ready 상태부터 확인한다.</p>
<h3 id="4-11-구성요소가-멈추면-무엇이-영향을-받을까">4-11. 구성요소가 멈추면 무엇이 영향을 받을까?</h3>
<table>
<thead>
<tr>
<th>멈춘 구성요소</th>
<th>주로 어려워지는 일</th>
<th>남아 있을 수 있는 동작</th>
</tr>
</thead>
<tbody><tr>
<td>API 서버</td>
<td>조회·변경·제어 상태 공유</td>
<td>기존 컨테이너 실행과 일부 기존 트래픽</td>
</tr>
<tr>
<td>etcd</td>
<td>객체 저장·조회와 제어 기능</td>
<td>기존 프로세스 실행</td>
</tr>
<tr>
<td>스케줄러</td>
<td>새 Pod의 노드 배정</td>
<td>배정된 Pod 실행</td>
</tr>
<tr>
<td>컨트롤러 매니저</td>
<td>복제본 보충·롤아웃 등</td>
<td>기존 Pod와 kubelet의 개별 컨테이너 관리</td>
</tr>
<tr>
<td>특정 노드 kubelet</td>
<td>그 노드의 상태 보고·관리</td>
<td>런타임에서 이미 실행 중인 컨테이너가 당장 유지될 수 있음</td>
</tr>
<tr>
<td>특정 노드 kube-proxy</td>
<td>Service 규칙 갱신</td>
<td>이미 설치된 규칙에 의한 통신</td>
</tr>
<tr>
<td>CoreDNS</td>
<td>새로운 이름 해석</td>
<td>IP 직접 통신·캐시된 이름을 통한 일부 통신</td>
</tr>
</tbody></table>
<p>이는 보장표가 아니라 장애 범위를 나누는 출발점이다. 구성, 장애 기간, 의존성, 후속 장애에 따라 사용자 영향이 달라진다.</p>
<p>특히 kube-proxy 프로세스가 멈췄다고 기존 규칙이 자동으로 모두 삭제되는 것은 아니다. 반대로 현재 요청이 된다는 이유만으로 제어 기능 장애를 방치해서도 안 된다. 이후 재배치나 대상 변경을 따라가지 못할 수 있다.</p>
<h3 id="4-12-4장-핵심-정리">4-12. 4장 핵심 정리</h3>
<p>장애를 볼 때는 “어느 단계까지 성공했는가?”를 묻는다. API 요청, 객체 생성, 노드 배정, 이미지 다운로드, 컨테이너 실행, 준비 검사, 서비스 연결은 서로 다른 단계다.</p>
<p><strong>이해 확인:</strong> 스케줄러가 컨테이너를 실행할까? → 아니다. 노드를 선택하고 배정하며, 실행은 kubelet과 런타임이 담당한다.</p>
<hr />
<h2 id="5장-실습-환경과-클러스터-접속">5장. 실습 환경과 클러스터 접속</h2>
<h3 id="5-1-필요한-도구와-버전-확인">5-1. 필요한 도구와 버전 확인</h3>
<table>
<thead>
<tr>
<th>도구</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td>kubectl</td>
<td>쿠버네티스 API 조작</td>
</tr>
<tr>
<td>AWS CLI</td>
<td>AWS 인증과 EKS 접속 설정</td>
</tr>
<tr>
<td>Docker 등 이미지 도구</td>
<td>이미지 빌드와 로컬 실행</td>
</tr>
<tr>
<td>Helm</td>
<td>차트 기반 설치·관리, 이 범위에서는 선택 도구</td>
</tr>
</tbody></table>
<pre><code class="language-bash">kubectl version --client
aws --version
docker --version
helm version --short</code></pre>
<p>kubectl은 API 서버와 호환되는 버전을 사용해야 한다. 지원되는 기본 버전 차이 기준은 마이너 버전 한 단계 이내이며, 여러 API 서버 버전이 섞여 있으면 범위가 더 좁아질 수 있다. 수업에서 지정한 버전과 환경 기준을 먼저 확인한다. <a href="https://kubernetes.io/releases/version-skew-policy/">kubectl 버전 호환 정책</a></p>
<p>EKS 실습에는 안내받은 AWS CLI v2를 사용한다. “모든 AWS CLI v1은 토큰을 만들 수 없다”처럼 일반화하는 대신, 실제 서비스의 지원 버전과 설치 안내를 따른다.</p>
<h3 id="5-2-eks-접속-설정-만들기">5-2. EKS 접속 설정 만들기</h3>
<pre><code class="language-bash">aws eks update-kubeconfig \
  --region &lt;사용할-리전&gt; \
  --name &lt;클러스터-이름&gt; \
  --profile &lt;로컬-프로파일&gt;</code></pre>
<p>이 명령은 클러스터를 새로 만드는 명령이 아니다. <strong>kubectl이 어느 API 서버에 어떤 방식으로 인증할지 로컬 설정을 준비하는 명령</strong>이다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl cluster-info
kubectl get pods -n &lt;내-네임스페이스&gt;</code></pre>
<p>kubeconfig에는 접속 대상과 인증 방법 등이 들어간다. EKS에서는 보통 AWS CLI 실행을 통해 토큰을 얻는다. kubeconfig는 환경에 따라 자격증명 자체를 담을 수도 있으므로 파일 전체를 공개하지 않는다.</p>
<p><code>get nodes</code>가 거부된다고 클러스터 접속 전체가 실패한 것은 아니다. 노드는 볼 권한이 없지만 자기 네임스페이스의 Pod는 볼 수 있는 계정도 있다.</p>
<h3 id="5-3-context와-kubeconfig">5-3. Context와 KUBECONFIG</h3>
<p><strong>컨텍스트</strong>는 클러스터, 사용자 인증 정보, 기본 네임스페이스를 묶은 접속 선택지다. 휴대전화에서 현재 어느 계정으로 어느 업무 공간에 들어와 있는지와 비슷하다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl config get-contexts
kubectl config use-context &lt;컨텍스트-이름&gt;
kubectl config set-context --current --namespace=&lt;내-네임스페이스&gt;</code></pre>
<p>별도 파일을 사용하려면 다음처럼 지정할 수 있다.</p>
<pre><code class="language-bash">export KUBECONFIG=&quot;$HOME/.kube/study-config&quot;</code></pre>
<p>이 파일에도 올바른 접속 설정이 들어 있어야 한다. 변수를 지정한 것만으로 인증 정보가 만들어지지는 않는다. 파일을 나누는 것은 사고 방지에 도움을 주지만 권한을 제한하는 보안 장치 자체는 아니다.</p>
<h3 id="5-4-kubectl-문법과-권한-확인">5-4. kubectl 문법과 권한 확인</h3>
<p>대부분의 리소스 명령은 다음 구조로 읽는다.</p>
<pre><code class="language-text">kubectl &lt;동사&gt; &lt;리소스 종류&gt; &lt;이름&gt; &lt;옵션&gt;</code></pre>
<pre><code class="language-bash">kubectl get pods -n demo
kubectl describe pod study-web -n demo
kubectl get pods -n demo -l app=study-web -o wide</code></pre>
<p><code>get</code>은 조회, <code>pod</code>는 종류, <code>study-web</code>은 이름, <code>-n</code>은 네임스페이스다. 다만 <code>logs</code>, <code>exec</code>, <code>config</code>처럼 하위 명령별 문법이 다른 경우도 있으므로 <code>--help</code>를 함께 사용한다.</p>
<pre><code class="language-bash">kubectl auth can-i get pods -n demo
kubectl auth can-i create deployments.apps -n demo
kubectl auth can-i delete nodes</code></pre>
<p>권한이 의심되면 추측보다 <code>auth can-i</code>로 확인한다. 이 결과는 권한 판단을 돕지만, 개별 생성 요청이 쿼터·검증 정책까지 모두 통과한다는 보장은 아니다.</p>
<h3 id="5-5-get과-describe">5-5. get과 describe</h3>
<p><code>get</code>은 현재 목록과 상태를 빠르게 보는 도구이고, <code>describe</code>는 한 대상의 설정·상태·관련 이벤트를 설명하는 도구다.</p>
<pre><code class="language-bash">kubectl get pods
kubectl get pods -o wide
kubectl get pods -w
kubectl describe pod study-web
kubectl get events --sort-by=.metadata.creationTimestamp</code></pre>
<p>병원에 비유하면 <code>get</code>은 환자 현황판이고 <code>describe</code>는 한 환자의 진료 기록에 가깝다. 특히 Events는 이미지 다운로드 실패, 자원 부족, 볼륨 문제 등을 찾는 출발점이다.</p>
<p>Events는 영구 감사 기록이 아니고 보존 시간도 제한될 수 있다. <code>describe</code>도 앱 내부 예외를 전부 보여주지 않으므로 앱 로그와 함께 읽는다.</p>
<h3 id="5-6-정렬과-필터링">5-6. 정렬과 필터링</h3>
<pre><code class="language-bash">kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'
kubectl get pods --field-selector=status.phase=Running
kubectl get events --field-selector=type=Warning
kubectl get pods --show-labels
kubectl get pods -l 'app in (study-web,study-worker)'</code></pre>
<p>라벨 셀렉터는 붙여 둔 분류로, 필드 셀렉터는 지원되는 객체 필드로 고른다. 필드 셀렉터가 모든 필드의 임의 검색을 지원하는 것은 아니다.</p>
<p><strong>중요:</strong> <code>status.phase!=Running</code>은 “문제 있는 Pod 전부”가 아니다. Running이면서 Ready가 false이거나 컨테이너가 반복 재시작 중인 Pod가 빠질 수 있다. 정상 완료한 작업도 포함될 수 있다. phase, READY, RESTARTS, 컨테이너 상태를 함께 본다.</p>
<h3 id="5-7-원하는-출력-형식으로-조회하기">5-7. 원하는 출력 형식으로 조회하기</h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>-o wide</code></td>
<td>노드·IP 등 추가 열</td>
</tr>
<tr>
<td><code>-o yaml</code></td>
<td>객체 구조를 읽기</td>
</tr>
<tr>
<td><code>-o json</code></td>
<td>프로그램으로 가공</td>
</tr>
<tr>
<td><code>-o name</code></td>
<td>종류/이름만 출력</td>
</tr>
<tr>
<td><code>-o custom-columns=...</code></td>
<td>필요한 열만 표로 구성</td>
</tr>
<tr>
<td><code>-o jsonpath=...</code></td>
<td>필요한 값 추출</td>
</tr>
</tbody></table>
<pre><code class="language-bash">kubectl get pods -o custom-columns='NAME:.metadata.name,NODE:.spec.nodeName,IP:.status.podIP'
kubectl get pods -l app=study-web -o name</code></pre>
<p>첫 번째는 “Pod 이름, 배치 노드, IP만 보여줘”라는 뜻이다. 블로그에 실제 출력 전체를 붙이기보다 설명에 필요한 가상 출력으로 바꾸는 편이 좋다.</p>
<h3 id="5-8-jsonpath로-값-꺼내기">5-8. JSONPath로 값 꺼내기</h3>
<p>JSONPath는 중첩된 데이터 안의 값을 찾아가는 경로다. 파일 탐색기에서 폴더를 순서대로 여는 것과 비슷하다.</p>
<pre><code class="language-bash">kubectl get pod study-web -o jsonpath='{.status.podIP}{&quot;\n&quot;}'
kubectl get pods -o jsonpath='{.items[*].metadata.name}{&quot;\n&quot;}'
kubectl get configmap study-config -o jsonpath='{.data.app\.properties}{&quot;\n&quot;}'
kubectl get deployment study-web \
  -o jsonpath='{.spec.template.spec.containers[?(@.name==&quot;web&quot;)].image}{&quot;\n&quot;}'</code></pre>
<ul>
<li><code>.</code>은 안쪽 필드로 들어간다.</li>
<li><code>[0]</code>은 첫 번째 항목, <code>[*]</code>는 모든 항목이다.</li>
<li><code>?(@.name==&quot;web&quot;)</code>는 이름이 web인 항목을 고른다.</li>
<li>키 이름 자체에 점이 있으면 <code>app\.properties</code>처럼 구분자와 구별한다.</li>
</ul>
<p>필드가 기억나지 않으면 먼저 <code>-o yaml</code>로 구조를 본 뒤 경로를 작성한다.</p>
<h3 id="5-9-로그-exec-포트포워딩">5-9. 로그, exec, 포트포워딩</h3>
<pre><code class="language-bash">kubectl logs study-web --tail=100
kubectl logs -f study-web
kubectl logs study-web --previous
kubectl exec -it study-web -- /bin/sh
kubectl port-forward pod/study-web 18080:80</code></pre>
<p><code>logs</code>는 앱이 남긴 기록, <code>exec</code>는 실행 중인 컨테이너 안에서 명령 실행, <code>port-forward</code>는 로컬 포트로 특정 Pod에 임시 연결하는 기능이다.</p>
<p><code>--previous</code>는 같은 Pod에서 직전에 종료된 해당 컨테이너의 로그를 확인하는 옵션이다. 삭제된 다른 Pod의 모든 과거 로그를 검색하는 기능은 아니다. Pod에 컨테이너가 여러 개면 <code>-c &lt;컨테이너&gt;</code>로 대상을 지정한다.</p>
<h3 id="5-10-만들고-고치고-지우기">5-10. 만들고 고치고 지우기</h3>
<pre><code class="language-bash">kubectl diff -f deployment.yaml
kubectl apply -f deployment.yaml
kubectl get deployment study-web
kubectl delete -f deployment.yaml</code></pre>
<p><code>diff</code>는 적용 전 차이를, <code>apply</code>는 원하는 상태의 반영을 담당한다. 차이가 없더라도 애플리케이션이 정상이라는 뜻은 아니므로 상태 확인이 이어져야 한다.</p>
<p>디렉터리의 여러 파일에는 <code>apply -f &lt;디렉터리&gt;</code>를, Kustomize 구성에는 <code>apply -k &lt;디렉터리&gt;</code>를 사용할 수 있다. 즉석으로 <code>scale</code>이나 <code>set image</code>를 썼다면 관리 중인 선언 파일과의 차이를 정리한다.</p>
<p>Pod 삭제는 앱 전체 종료와 다르다. Deployment가 관리하면 Pod를 보충한다. 앱을 중지할 의도라면 관리 객체의 복제본 수나 수명에 접근해야 한다.</p>
<h3 id="5-11-파일-없이-매니페스트-전달하기">5-11. 파일 없이 매니페스트 전달하기</h3>
<p>다음은 표준입력으로 ConfigMap을 전달하는 예다.</p>
<pre><code class="language-bash">kubectl apply -n demo -f - &lt;&lt;'EOF_CONFIG'
apiVersion: v1
kind: ConfigMap
metadata:
  name: study-config
data:
  app.properties: |
    app.name=StudyApp
  max.connections: &quot;100&quot;
EOF_CONFIG</code></pre>
<p><code>-f -</code>는 파일 대신 표준입력을 읽으라는 뜻이다. <code>&lt;&lt;'EOF_CONFIG'</code>의 따옴표는 셸이 본문의 <code>$변수</code> 등을 먼저 치환하지 않도록 한다. 변수 치환이 필요하다면 의도한 템플릿 방식으로 처리해야 한다.</p>
<p>실습에는 편하지만 반복 관리할 설정은 파일로 보관한다. ConfigMap 값은 문자열이라는 점도 함께 기억한다.</p>
<h3 id="5-12-첫-배포로-조정-루프-확인하기">5-12. 첫 배포로 조정 루프 확인하기</h3>
<pre><code class="language-bash">kubectl create deployment study-hello --image=nginx:1.28 --replicas=3
kubectl get deployment,replicaset,pod -l app=study-hello
kubectl get pods -l app=study-hello -o wide</code></pre>
<p>여기서 확인할 것은 세 가지다. Deployment가 있고, 그 아래 ReplicaSet이 있고, 실제로 실행하는 Pod가 있다는 점이다.</p>
<p>본인 실습용 Pod 하나를 이름으로 삭제하면 새 이름의 Pod가 생긴다. 단, 클러스터의 자원과 정책이 허용해야 생성이 완료된다. 실험 후에는 본인의 <code>study-hello</code> Deployment를 삭제하여 정리한다.</p>
<h3 id="5-13-실습-리소스-정리">5-13. 실습 리소스 정리</h3>
<p><code>kubectl get all</code>은 이름과 달리 모든 종류를 보여주지 않는다. ConfigMap, Secret, Ingress, PVC 등은 별도로 확인해야 한다.</p>
<pre><code class="language-bash">kubectl get all -l app=study-web
kubectl get ingress,configmap,secret,pvc -l app=study-web</code></pre>
<p>정리는 본인이 만든 파일이나 정확한 이름을 기준으로 한다. Deployment를 지워도 별도로 생성한 Service나 PVC는 남을 수 있다. 클라우드 자원을 생성한 실습이라면 연결된 외부 자원의 수명도 확인한다.</p>
<p>공유 네임스페이스에서는 전체 삭제를 사용하지 않는다. 라벨 삭제도 라벨이 정말 본인 것만 가리키는지 같은 조건으로 먼저 조회한다.</p>
<h3 id="5-14-5장-핵심-정리">5-14. 5장 핵심 정리</h3>
<p>접속 대상과 네임스페이스를 확인하고, 권한을 확인한 뒤 조작한다. 문제를 만나면 목록 → 상세 이벤트 → 로그 순서로 좁힌다.</p>
<p><strong>이해 확인:</strong> <code>aws eks update-kubeconfig</code>는 무엇을 만들까? → 클러스터가 아니라 로컬 접속 설정을 준비한다.</p>
<hr />
<h2 id="6장-kubectl-실무">6장. kubectl 실무</h2>
<h3 id="6-1-포트포워딩으로-직접-붙기">6-1. 포트포워딩으로 직접 붙기</h3>
<p>외부 공개 주소 없이 앱 하나에 접속해 보고 싶을 때 사용한다.</p>
<pre><code class="language-bash">kubectl port-forward pod/study-web 18080:80
# 다른 터미널
curl http://localhost:18080/</code></pre>
<p><code>18080</code>은 내 컴퓨터의 포트, <code>80</code>은 Pod 쪽 포트다. Service나 Deployment를 대상으로 지정할 수도 있다.</p>
<pre><code class="language-bash">kubectl port-forward svc/study-web 18080:80
kubectl port-forward deploy/study-web 18080:80</code></pre>
<p>이 경우에도 실제로는 선택한 Pod에 연결한다. Service의 평소 부하분산 경로 전체를 시험하는 것과 같지 않다. 연결 대상 Pod가 종료되면 다시 실행해야 할 수 있고, 명령을 <code>Ctrl+C</code>로 종료하면 연결도 끝난다. <a href="https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/">쿠버네티스 포트포워딩</a></p>
<p>예를 들어 포트포워딩으로 정상 응답을 받았는데 외부 주소는 실패한다면 앱 자체 외의 Service·Ingress·DNS 경로를 추가로 살펴볼 근거가 된다.</p>
<h3 id="6-2-컨테이너-안에서-명령-실행하기">6-2. 컨테이너 안에서 명령 실행하기</h3>
<pre><code class="language-bash">kubectl exec study-web -- ls -al /tmp
kubectl exec -it study-web -- /bin/sh
kubectl exec -it study-web -c web -- /bin/sh</code></pre>
<p><code>--</code> 앞은 kubectl의 옵션이고 뒤는 컨테이너 안에서 실행할 명령이다. <code>-i</code>는 입력 전달, <code>-t</code>는 터미널 할당이다.</p>
<p>경량 이미지에는 bash나 sh가 없을 수 있다. 이런 경우에는 권한과 정책이 허용하는 환경에서 임시 디버그 컨테이너를 사용할 수 있다.</p>
<pre><code class="language-bash">kubectl debug -it study-web --image=busybox:1.36 --target=web</code></pre>
<p>임시 컨테이너도 Pod에 대한 변경이고 권한이 필요하다. <code>--target</code>의 프로세스 관찰 지원 등은 런타임 조건에 영향을 받는다. 네트워크 진단용 별도 Pod를 만드는 방법도 있으나, 다른 네임스페이스·라벨·정책을 사용하면 원래 앱과 조건이 달라질 수 있다.</p>
<p>안에서 파일을 고치는 것은 원인 확인용으로만 생각한다. 지속적인 수정은 이미지, 설정, 매니페스트에 반영해야 한다. 환경변수 전체 출력에는 비밀값이 섞일 수 있어 공개하지 않는다.</p>
<h3 id="6-3-자원-사용량-보기">6-3. 자원 사용량 보기</h3>
<pre><code class="language-bash">kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods --sort-by=memory</code></pre>
<p>CPU와 메모리를 많이 사용하는 대상을 살펴보는 명령이다. metrics-server 등 필요한 지표 제공 구성이 있어야 한다.</p>
<p>requests는 주로 배치 시 자원 수요 계산에 사용되고 limits는 실행 중 자원 사용을 제한한다. <code>top</code>은 관찰된 최근 사용량이다. <strong>예약·요청값과 실제 사용값은 다르다.</strong></p>
<p>버스 좌석을 예약한 인원과 지금 타고 있는 인원이 다른 것과 비슷하다. 한 번의 사용량만 보고 적정 설정을 정하기보다 평소·최대 부하와 장기간 추이를 확인한다. <code>top</code>은 장기 모니터링을 대체하지 않는다.</p>
<h3 id="6-4-라벨과-애노테이션">6-4. 라벨과 애노테이션</h3>
<p>라벨은 대상을 선택하고 분류하는 데, 애노테이션은 설명·도구 설정 등 부가 정보를 붙이는 데 사용한다.</p>
<pre><code class="language-bash">kubectl label pod study-web purpose=practice
kubectl label pod study-web purpose=review --overwrite
kubectl get pods -l purpose=review
kubectl label pod study-web purpose-

kubectl annotate pod study-web study-note=&quot;readiness 실험&quot;</code></pre>
<p><code>purpose-</code>는 해당 라벨 삭제를 뜻한다. 애노테이션은 일반적인 <code>-l</code> 라벨 셀렉터의 대상이 아니다. 컨트롤러가 특정 애노테이션을 읽어 동작을 바꾸기도 하므로 아무 키에나 설정을 적는다고 기능이 생기지는 않는다.</p>
<p>Deployment가 관리하는 Pod에만 즉석으로 라벨을 붙이면 대체 Pod에는 그 라벨이 없을 수 있다. 계속 유지할 라벨은 Pod 템플릿에도 작성한다.</p>
<h3 id="6-5-리소스를-수정하는-방법">6-5. 리소스를 수정하는 방법</h3>
<table>
<thead>
<tr>
<th>방법</th>
<th>의미</th>
<th>사용 시 생각할 점</th>
</tr>
</thead>
<tbody><tr>
<td><code>apply</code></td>
<td>선언을 적용</td>
<td>저장된 선언과 실제 상태를 관리</td>
</tr>
<tr>
<td><code>edit</code></td>
<td>현재 객체를 에디터에서 수정</td>
<td>원본 파일과 어긋나기 쉬움</td>
</tr>
<tr>
<td><code>patch</code></td>
<td>특정 필드를 부분 수정</td>
<td>자동화·부분 변경에 활용</td>
</tr>
<tr>
<td><code>set image</code></td>
<td>이미지 필드 수정</td>
<td>이미지 변경의 편의 명령</td>
</tr>
<tr>
<td><code>replace</code></td>
<td>객체 정의를 교체</td>
<td>최신 객체와 필드 조건 확인</td>
</tr>
<tr>
<td><code>replace --force</code></td>
<td>삭제 후 다시 생성</td>
<td>중단과 식별자 변경 가능</td>
</tr>
</tbody></table>
<p>운영에서는 어떤 도구를 쓰더라도 변경의 기준과 이력이 있어야 한다. 선언형 관리 환경에서 즉석 변경을 했다면 원본 설정에도 반영하거나 원래 상태로 되돌린다.</p>
<p>특히 <code>replace --force</code>는 단순한 덮어쓰기라고 생각하면 안 된다. 삭제와 재생성을 수반하므로 연결과 실행이 끊길 수 있다.</p>
<h3 id="6-6-리소스를-삭제하는-방법">6-6. 리소스를 삭제하는 방법</h3>
<pre><code class="language-bash">kubectl get deployment study-web
kubectl delete deployment study-web</code></pre>
<p>이름, 파일, 라벨로 대상을 지정할 수 있다. 라벨로 삭제할 때도 먼저 같은 조건으로 조회한다.</p>
<pre><code class="language-bash">kubectl get pods,services -l app=study-web
# 조회 결과가 본인 실습 리소스일 때만 해당 조건으로 정리</code></pre>
<p>강제 삭제는 정상 종료를 기다리는 과정을 건너뛸 수 있다. API에서 객체가 없어졌다고 노드의 프로세스 종료까지 반드시 확인된 것은 아니다. 특히 노드 통신 장애나 상태를 저장하는 앱에서는 중복 실행 위험을 고려해야 한다.</p>
<p>네임스페이스 전체를 지우거나 공유 공간에서 <code>--all</code>을 사용하는 것은 개인 리소스 정리가 아니다.</p>
<h3 id="6-7-kubectl-cp로-파일-복사하기">6-7. <code>kubectl cp</code>로 파일 복사하기</h3>
<pre><code class="language-bash">kubectl cp ./sample.txt study-web:/tmp/sample.txt -c web
kubectl cp study-web:/tmp/sample.txt ./sample-copy.txt -c web</code></pre>
<p>콜론 앞은 Pod 이름, 뒤는 컨테이너 안 경로다. 네임스페이스가 필요하면 <code>demo/study-web:/tmp/sample.txt</code>처럼 표현할 수 있다.</p>
<p>보통 컨테이너에 <code>tar</code>가 있어야 동작하므로 최소 이미지에서는 실패할 수 있다. 파일 복사는 새 이미지 배포를 대신하는 방식이 아니다.</p>
<p>실무에서는 진단용 파일이나 덤프를 꺼내 분석하는 데 활용한다. Java 힙덤프 생성에는 <code>jcmd</code> 등 도구와 여유 디스크가 필요하고, 덤프에는 메모리에 있던 개인정보나 토큰이 포함될 수 있다. 공개 학습 글에는 원본 덤프를 첨부하지 않는다.</p>
<h3 id="6-8-k9s-터미널에서-보는-대시보드">6-8. k9s: 터미널에서 보는 대시보드</h3>
<p>k9s는 반복 조회를 화면으로 편하게 수행하게 해 주는 별도 도구다. Kubernetes 자체 구성요소는 아니다.</p>
<table>
<thead>
<tr>
<th>조작의 예</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>k9s -n demo</code></td>
<td>해당 네임스페이스로 시작</td>
</tr>
<tr>
<td><code>:pods</code>, <code>:svc</code>, <code>:deploy</code></td>
<td>리소스 화면 이동</td>
</tr>
<tr>
<td><code>/</code></td>
<td>표시 목록 필터</td>
</tr>
<tr>
<td><code>d</code></td>
<td>상세 설명</td>
</tr>
<tr>
<td><code>l</code></td>
<td>로그</td>
</tr>
<tr>
<td><code>s</code></td>
<td>셸 접근</td>
</tr>
<tr>
<td><code>?</code></td>
<td>현재 화면의 키 도움말</td>
</tr>
<tr>
<td><code>Esc</code>, <code>:q</code></td>
<td>뒤로 이동·종료</td>
</tr>
</tbody></table>
<p>단축키는 화면과 버전에 따라 도움말로 확인한다. <a href="https://k9scli.io/topics/commands/">k9s 명령 안내</a> 삭제·수정 기능도 있으므로 관찰 도구라고 해서 모든 조작이 조회인 것은 아니다. 어떤 화면에서 어떤 리소스를 선택했는지 확인한다.</p>
<h3 id="6-9-kubectl-아래의-api-직접-보기">6-9. kubectl 아래의 API 직접 보기</h3>
<p>kubectl은 쿠버네티스 API를 사용하는 클라이언트다. 서버의 API 경로를 직접 조회할 수도 있다.</p>
<pre><code class="language-bash">kubectl get --raw /api
kubectl get --raw /apis
kubectl get --raw /api/v1/namespaces/demo/pods</code></pre>
<p><code>/api</code>와 <code>/apis</code>는 API 그룹 구조를 파악하는 출발점이다. Pod 로그처럼 객체의 하위 기능도 별도 API 경로와 권한으로 다뤄진다.</p>
<pre><code class="language-bash">kubectl proxy --port=18001
# 다른 터미널
curl http://127.0.0.1:18001/api/</code></pre>
<p><code>kubectl proxy</code>는 <strong>쿠버네티스 API</strong>에 접근하도록 돕는다. 앱 포트에 연결하는 <code>port-forward</code>와 목적이 다르다. 상세 통신 로그를 볼 때는 <code>--v=8</code> 등을 사용할 수 있지만, 서버 주소와 요청·응답 내용이 포함될 수 있으므로 원본 출력은 공개하지 않는다.</p>
<h3 id="6-10-api-리소스와-필드를-스스로-찾기">6-10. API 리소스와 필드를 스스로 찾기</h3>
<pre><code class="language-bash">kubectl api-resources
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
kubectl api-versions
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy</code></pre>
<p><code>api-resources</code>는 사용할 수 있는 리소스 종류, <code>api-versions</code>는 API 버전, <code>explain</code>은 필드 설명을 찾는 데 사용한다.</p>
<p>처음 보는 YAML을 만났을 때 외우려 하지 말고 “이 필드가 어떤 부모 아래에 있는가?”를 따라간다. <code>no matches for kind</code> 오류라면 종류와 API 버전, 추가 리소스 정의의 설치 여부를 확인한다.</p>
<h3 id="6-11-적용-전-검증-diff와-dry-run">6-11. 적용 전 검증: diff와 dry-run</h3>
<pre><code class="language-bash">kubectl diff -f deployment.yaml
kubectl apply --dry-run=server -f deployment.yaml
kubectl apply -f deployment.yaml</code></pre>
<p>세 명령의 질문은 각각 다르다.</p>
<ul>
<li><code>diff</code>: 지금 상태와 무엇이 달라지는가?</li>
<li><code>--dry-run=server</code>: 저장하지 않고 서버의 검증을 통과하는가?</li>
<li><code>apply</code>: 실제로 반영한다.</li>
</ul>
<p>서버 dry-run 성공은 이미지 다운로드나 앱 기동까지 성공한다는 뜻이 아니다. 아직 Pod를 실제로 실행하지 않았기 때문이다.</p>
<pre><code class="language-bash">kubectl create deployment study-web \
  --image=nginx:1.28 \
  --dry-run=client -o yaml &gt; deployment.yaml</code></pre>
<p>클라이언트 dry-run은 이처럼 초안을 만드는 데도 유용하다. 모든 명령 조합에서 서버와의 통신이 전혀 없다고 단정하지 말고, 주된 차이를 “변경을 저장하지 않고 어디까지 처리하는가”로 이해한다.</p>
<h3 id="6-12-6장-핵심-정리">6-12. 6장 핵심 정리</h3>
<table>
<thead>
<tr>
<th>알고 싶은 것</th>
<th>먼저 고를 도구</th>
</tr>
</thead>
<tbody><tr>
<td>지금 무엇이 있나</td>
<td>get</td>
</tr>
<tr>
<td>왜 실행되지 않나</td>
<td>describe와 Events</td>
</tr>
<tr>
<td>앱은 어떤 오류를 냈나</td>
<td>logs</td>
</tr>
<tr>
<td>이전 컨테이너가 왜 종료됐나</td>
<td>logs --previous</td>
</tr>
<tr>
<td>특정 앱에 임시 연결하고 싶다</td>
<td>port-forward</td>
</tr>
<tr>
<td>내부 파일·프로세스를 확인하고 싶다</td>
<td>exec, debug</td>
</tr>
<tr>
<td>자원을 얼마나 쓰나</td>
<td>top</td>
</tr>
<tr>
<td>무엇이 바뀌나</td>
<td>diff</td>
</tr>
<tr>
<td>이 필드는 무엇인가</td>
<td>explain</td>
</tr>
<tr>
<td>내 권한이 있나</td>
<td>auth can-i</td>
</tr>
</tbody></table>
<p>명령어를 통째로 암기하기보다 질문에서 도구로 이어지는 길을 익힌다.</p>
<hr />
<h2 id="7장-pod">7장. Pod</h2>
<h3 id="7-1-pod는-컨테이너-자체가-아니다">7-1. Pod는 컨테이너 자체가 아니다</h3>
<p>Pod는 하나 이상의 컨테이너를 같은 노드에 함께 배치하는 단위다. 보통 앱 컨테이너 하나로 시작하고, 밀접하게 보조해야 하는 컨테이너가 있을 때 함께 묶는다.</p>
<table>
<thead>
<tr>
<th>공유하는 것</th>
<th>기본적으로 따로인 것</th>
</tr>
</thead>
<tbody><tr>
<td>네트워크 공간과 포트 공간</td>
<td>각 컨테이너의 이미지 파일시스템</td>
</tr>
<tr>
<td>같은 노드에 배치되는 운명</td>
<td>각 컨테이너의 실행 상태</td>
</tr>
<tr>
<td>각자 마운트한 공통 볼륨</td>
<td>프로세스 공간은 별도이며 공유 설정 가능</td>
</tr>
</tbody></table>
<p>같은 집의 룸메이트처럼 주소는 공유하지만 개인 서랍은 따로다. 공용 선반을 마련하면 그곳의 물건을 함께 사용할 수 있는데, 이것이 공통 볼륨에 해당한다.</p>
<p><strong>컨테이너 재시작과 Pod 재생성은 다르다.</strong> Pod 안의 앱 하나가 실패했을 때 해당 컨테이너만 재시작될 수 있다. Pod를 새로 만들면 UID가 달라지는 새로운 객체가 된다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/">Pod 공식 설명</a></p>
<h3 id="7-2-pod-매니페스트-읽기">7-2. Pod 매니페스트 읽기</h3>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: study-web
  labels:
    app: study-web
spec:
  containers:
    - name: web
      image: nginx:1.28
      ports:
        - name: http
          containerPort: 80
      resources:
        requests:
          cpu: 100m
          memory: 64Mi
        limits:
          cpu: 500m
          memory: 128Mi
  restartPolicy: Always</code></pre>
<p><code>containers</code>가 목록인 이유는 Pod 안에 컨테이너를 여러 개 둘 수 있기 때문이다. <code>100m</code>은 CPU 0.1개에 해당하는 단위이고, <code>Mi</code>는 1024를 기반으로 한 메모리 단위다. 수치는 문법 학습용이며 앱별로 측정하여 정한다.</p>
<p><code>containerPort: 80</code>은 앱 프로세스를 실행하거나 방화벽을 열어 주는 설정이 아니다. 앱이 실제로 그 포트에서 수신해야 한다. 포트에 이름을 주면 Service나 프로브 등에서 참조할 수 있다.</p>
<h3 id="7-3-실행-중인-pod의-yaml에는-왜-필드가-많을까">7-3. 실행 중인 Pod의 YAML에는 왜 필드가 많을까?</h3>
<pre><code class="language-bash">kubectl get pod study-web -o yaml</code></pre>
<p>처음 작성한 파일보다 훨씬 많은 정보가 나올 수 있다. 사용자 입력, API 기본값, admission 변경, 배정 결과, 실행 상태가 한 객체에 함께 보이기 때문이다.</p>
<table>
<thead>
<tr>
<th>필드의 예</th>
<th>읽는 의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>metadata.uid</code></td>
<td>이 객체 인스턴스의 고유 식별자</td>
</tr>
<tr>
<td><code>spec.nodeName</code></td>
<td>실행하도록 배정된 노드</td>
</tr>
<tr>
<td><code>spec.serviceAccountName</code></td>
<td>Pod가 사용하는 서비스 계정</td>
</tr>
<tr>
<td><code>spec.containers[].imagePullPolicy</code></td>
<td>이미지 다운로드 정책</td>
</tr>
<tr>
<td><code>status.podIP</code></td>
<td>할당된 Pod IP</td>
</tr>
<tr>
<td><code>status.containerStatuses</code></td>
<td>컨테이너별 상태·재시작·이미지 정보</td>
</tr>
</tbody></table>
<p>이미지 pull 정책의 기본값은 태그 등의 조건에 영향을 받는다. 항상 <code>IfNotPresent</code>라고 외우지 않는다. 전체 YAML을 다음 배포 파일로 그대로 복사하기보다 관리할 필드와 서버가 채운 필드를 구별한다.</p>
<h3 id="7-4-pod-생명주기-phase와-ready">7-4. Pod 생명주기: phase와 Ready</h3>
<table>
<thead>
<tr>
<th>phase</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Pending</td>
<td>객체는 받아들여졌지만 컨테이너 실행 준비를 완료하지 못함</td>
</tr>
<tr>
<td>Running</td>
<td>노드에 배정되고 컨테이너가 생성되어, 적어도 하나가 실행·시작·재시작 중</td>
</tr>
<tr>
<td>Succeeded</td>
<td>모든 컨테이너가 성공적으로 종료했고 다시 시작하지 않음</td>
</tr>
<tr>
<td>Failed</td>
<td>모든 컨테이너가 종료했고 하나 이상 실패했으며 재시작하지 않는 상황 등</td>
</tr>
<tr>
<td>Unknown</td>
<td>상태를 확인하지 못하는 상황</td>
</tr>
</tbody></table>
<p><strong>Running은 앱이 정상 요청을 처리한다는 보장이 아니다.</strong> 업무를 시작한 직원이 로그인과 장비 준비까지 완료한 것은 아닌 것과 같다. 요청을 받을 준비 여부는 Ready 조건으로 따로 본다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/">Pod 생명주기</a></p>
<p><code>kubectl get pods</code>의 <code>READY 1/2</code>는 일반적으로 표시 대상 컨테이너 둘 중 하나만 준비되었다는 뜻이다. Pod의 최종 Ready 판정에는 컨테이너 준비 상태 외에 readiness gate 같은 조건도 관여할 수 있다.</p>
<h3 id="7-5-status-열의-이름-읽기">7-5. STATUS 열의 이름 읽기</h3>
<p>STATUS 열은 phase만 그대로 표시하는 열이 아니다. 컨테이너 대기 이유와 종료 정보를 읽기 좋게 요약하기도 한다.</p>
<table>
<thead>
<tr>
<th>표시</th>
<th>의미와 첫 확인 지점</th>
</tr>
</thead>
<tbody><tr>
<td>Pending</td>
<td>배치·준비가 끝나지 않음 → Events</td>
</tr>
<tr>
<td>ContainerCreating</td>
<td>이미지 외에도 네트워크·볼륨 등 준비 과정 → Events</td>
</tr>
<tr>
<td>ImagePullBackOff</td>
<td>이미지 다운로드 실패 후 재시도 대기 → 경로·태그·인증·네트워크</td>
</tr>
<tr>
<td>Running, READY 부족</td>
<td>준비 조건 미충족 → readiness 결과</td>
</tr>
<tr>
<td>CrashLoopBackOff</td>
<td>컨테이너가 반복 종료되어 재시작을 기다림 → 이전 로그와 종료 이유</td>
</tr>
<tr>
<td>Completed</td>
<td>정상 종료 → 배치라면 정상, 계속 실행할 서버라면 실행 명령 확인</td>
</tr>
<tr>
<td>Evicted</td>
<td>노드 자원 압박 등으로 축출 → Pod 상태와 노드 상태</td>
</tr>
<tr>
<td>Terminating</td>
<td>삭제 처리 중 → 종료 유예·노드 연결·finalizer·볼륨 등</td>
</tr>
</tbody></table>
<p><code>CrashLoopBackOff</code>와 <code>ImagePullBackOff</code>는 Pod phase 이름이 아니다. <code>OOMKilled</code>도 주로 컨테이너 종료 이유에서 찾는다. 재시도 지연은 설정과 기능에 영향을 받으므로 고정된 초 단위 수열보다 “반복 실패 시 재시도 간격이 늘어날 수 있다”는 개념을 기억한다.</p>
<p>Pending 단계라도 이미 실행한 초기화 컨테이너의 로그가 있을 수 있다. 따라서 “Pending이면 무조건 로그가 없다”도 지나친 단순화다.</p>
<h3 id="7-6-초기화-컨테이너-initcontainer">7-6. 초기화 컨테이너, initContainer</h3>
<p>일반적인 initContainer는 앱 컨테이너보다 먼저 순서대로 실행되며, 앞 단계가 성공적으로 끝나야 다음 단계로 넘어간다.</p>
<p>식당 개점 전 “바닥 청소 → 재료 준비 → 영업 시작”을 순서대로 수행하는 것과 비슷하다.</p>
<pre><code class="language-yaml"># Pod spec의 일부
initContainers:
  - name: prepare
    image: busybox:1.36
    command: [&quot;sh&quot;, &quot;-c&quot;, &quot;echo ready &gt; /work/ready.txt&quot;]
    volumeMounts:
      - name: work
        mountPath: /work
containers:
  - name: app
    image: busybox:1.36
    command: [&quot;sh&quot;, &quot;-c&quot;, &quot;cat /work/ready.txt; sleep 3600&quot;]
    volumeMounts:
      - name: work
        mountPath: /work
volumes:
  - name: work
    emptyDir: {}</code></pre>
<p>초기화 실패 시 일반적으로 해당 initContainer를 정책에 따라 재시도한다. 항상 Pod 객체 전체를 새로 만드는 것은 아니다. <code>restartPolicy: Never</code> 등 정책에 따른 차이가 있으며, 초기화 작업은 반복 실행해도 안전하게 설계한다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/init-containers/">Init Containers 공식 설명</a></p>
<p>DB 마이그레이션을 각 Pod의 init에 넣으면 복제본 수만큼 동시에 실행할 수 있다. 중복 실행 방지와 작업 분리를 고려해야 한다. 네이티브 사이드카처럼 init 영역에 정의하지만 계속 실행하는 별도 패턴은 일반 initContainer와 구별한다.</p>
<h3 id="7-7-세-가지-프로브">7-7. 세 가지 프로브</h3>
<p><strong>프로브</strong>는 kubelet이 컨테이너의 상태를 검사하는 방법이다.</p>
<table>
<thead>
<tr>
<th>프로브</th>
<th>묻는 질문</th>
<th>실패가 누적되면</th>
</tr>
</thead>
<tbody><tr>
<td>startupProbe</td>
<td>초기 기동이 완료되었는가?</td>
<td>해당 컨테이너 종료·재시작 정책 적용</td>
</tr>
<tr>
<td>livenessProbe</td>
<td>재시작이 필요할 정도로 동작을 멈췄는가?</td>
<td>해당 컨테이너 종료·재시작 정책 적용</td>
</tr>
<tr>
<td>readinessProbe</td>
<td>지금 요청을 받아도 되는가?</td>
<td>준비되지 않은 상태로 표시하여 일반적인 Service 전달 대상에서 제외</td>
</tr>
</tbody></table>
<p>식당에 비유하면 startup은 개점 준비, readiness는 지금 주문을 받을 수 있는지, liveness는 조리 시스템이 회복 불가능하게 멈췄는지 확인하는 일이다.</p>
<p><strong>readiness 실패 자체는 재시작 명령이 아니다.</strong> startup이 성공하기 전에는 liveness와 readiness 검사를 시작하지 않는다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/probes/">프로브 공식 설명</a></p>
<p>readiness로 빠져도 모든 종류의 트래픽이 물리적으로 차단되는 것은 아니다. Pod IP 직접 호출, 기존 연결, 특수 Service 설정 등은 구별해야 한다.</p>
<h3 id="7-8-프로브-설정-읽기">7-8. 프로브 설정 읽기</h3>
<p>다음은 해당 건강 확인 경로를 실제로 제공하는 Spring Boot 앱의 컨테이너 설정 조각이다.</p>
<pre><code class="language-yaml">startupProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 30
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3</code></pre>
<ul>
<li><code>periodSeconds</code>: 검사 주기.</li>
<li><code>timeoutSeconds</code>: 한 번의 응답 대기 시간.</li>
<li><code>failureThreshold</code>: 연속 실패 판정 횟수.</li>
<li><code>initialDelaySeconds</code>: 설정했을 때 첫 검사 전 지연.</li>
</ul>
<p>startup의 <code>5 × 30</code>은 약 150초 규모의 기동 허용 시간을 이해하는 계산이다. 실제 실행 시각과 종료까지의 시간을 정확히 보장하는 타이머로 해석하지 않는다.</p>
<p>liveness에 외부 DB 장애를 그대로 연결하면, DB가 잠시 멈췄을 때 모든 앱을 재시작하여 상황을 악화시킬 수 있다. <strong>재시작으로 해결 가능한 앱 내부 이상인지</strong>를 기준으로 설계한다. readiness에 어떤 의존성을 포함할지도 서비스 특성에 맞춰 결정해야 한다. Spring Boot의 readiness 그룹에 DB가 자동으로 포함된다고 가정하지 않는다. <a href="https://docs.spring.io/spring-boot/reference/actuator/endpoints.html#actuator.endpoints.kubernetes-probes">Spring Boot 건강 검사와 Kubernetes 프로브</a></p>
<h3 id="7-9-pod-종료와-graceful-shutdown">7-9. Pod 종료와 graceful shutdown</h3>
<p>정상 종료는 단순히 프로세스를 끊는 것이 아니다. 새 요청을 줄이고, 처리 중인 작업을 마무리하고, 정해진 시간 안에 종료하는 과정이다.</p>
<pre><code class="language-text">삭제 요청과 종료 표시
  ├ 연결 대상 상태 갱신이 비동기로 전파됨
  └ kubelet이 종료 절차 시작
       → preStop 훅 실행(있다면)
       → 종료 신호, 보통 SIGTERM
       → 앱이 처리 중인 작업 마무리
       → 유예 시간이 끝나도 남으면 강제 종료</code></pre>
<p>종료 유예 시간은 preStop을 실행하는 시간도 포함한다. 예를 들어 preStop 5초와 앱 정리 20초를 예상한다면 둘을 포함할 충분한 여유가 필요하다. 앱의 정상 종료와 컨테이너 신호 전달도 함께 구성한다.</p>
<p>고정된 <code>sleep 5</code>가 모든 환경의 502 오류를 없애 주지는 않는다. 라우팅 전파 시간, 진행 중인 요청, 앱 종료 동작을 확인해야 한다. <a href="https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination">Pod 종료 절차</a></p>
<h3 id="7-10-재시작-정책과-백오프">7-10. 재시작 정책과 백오프</h3>
<table>
<thead>
<tr>
<th>restartPolicy</th>
<th>기본적인 동작</th>
<th>예</th>
</tr>
</thead>
<tbody><tr>
<td>Always</td>
<td>종료 코드와 관계없이 컨테이너를 다시 실행</td>
<td>계속 실행할 서버</td>
</tr>
<tr>
<td>OnFailure</td>
<td>실패 종료 시 다시 실행</td>
<td>실패 시 재시도할 작업</td>
</tr>
<tr>
<td>Never</td>
<td>컨테이너를 자동 재시작하지 않음</td>
<td>종료 결과를 관찰할 일회성 작업</td>
</tr>
</tbody></table>
<p>Deployment의 일반적인 Pod 템플릿은 <code>Always</code>를 사용한다. 완료가 목적인 작업은 Job 등의 워크로드를 고려한다.</p>
<p>백오프는 계속 실패하는 대상을 무한히 빠르게 재시도하지 않도록 기다리는 방식이다. 재시작 횟수와 직전 종료 이유를 함께 읽어야 한다. Job 컨트롤러가 새 Pod를 만들어 재시도하는 것과, 같은 Pod 안에서 컨테이너를 재시작하는 것도 다른 과정이다.</p>
<h3 id="7-11-멀티컨테이너-패턴">7-11. 멀티컨테이너 패턴</h3>
<table>
<thead>
<tr>
<th>패턴</th>
<th>역할</th>
<th>예</th>
</tr>
</thead>
<tbody><tr>
<td>사이드카</td>
<td>주 앱 보조</td>
<td>프록시, 인증서 갱신, 특정 로그 처리</td>
</tr>
<tr>
<td>앰배서더</td>
<td>외부 연결을 대신 담당</td>
<td>외부 DB로 연결하는 로컬 프록시</td>
</tr>
<tr>
<td>어댑터</td>
<td>출력 형식을 변환</td>
<td>기존 지표를 수집 시스템이 읽는 형식으로 변환</td>
</tr>
</tbody></table>
<p>하나의 Pod에 묶으면 같은 노드에 배치되고 Pod 단위로 확장한다. 그래서 API 서버와 독립적인 배치 워커처럼 <strong>서로 다른 속도로 확장·배포해야 하는 앱</strong>은 보통 분리한다.</p>
<p>같은 건물에 있어야 협업이 되는 팀은 함께 두되, 인원과 근무 시간이 독립적인 팀까지 한 자리로 묶지 않는 것과 같다. 서비스 메시도 구현에 따라 사이드카를 사용할 수 있지만 모든 서비스 메시가 반드시 사이드카 구조인 것은 아니다.</p>
<h3 id="7-12-downward-api-pod가-자기-정보-알기">7-12. Downward API: Pod가 자기 정보 알기</h3>
<p>앱 로그에 Pod 이름이 있으면 어느 복제본에서 문제가 생겼는지 찾기 쉽다. 쿠버네티스 API를 앱이 직접 호출하지 않고도 자기 정보를 주입받을 수 있다.</p>
<pre><code class="language-yaml">env:
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: POD_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
  - name: NODE_NAME
    valueFrom:
      fieldRef:
        fieldPath: spec.nodeName
  - name: POD_IP
    valueFrom:
      fieldRef:
        fieldPath: status.podIP
  - name: MEMORY_LIMIT_MIB
    valueFrom:
      resourceFieldRef:
        containerName: app
        resource: limits.memory
        divisor: 1Mi</code></pre>
<p><code>fieldRef</code>는 Pod 필드, <code>resourceFieldRef</code>는 컨테이너 자원 설정을 참조한다. 위 예제의 <code>app</code>은 실제 컨테이너 이름과 맞아야 하며 메모리 limit도 적절히 정의한다.</p>
<p>이는 필요한 일부 필드를 제공하는 기능이지 클러스터의 모든 정보를 자유롭게 읽는 통로는 아니다. 수집한 실제 노드명·IP 등은 공개 글에서는 일반화한다.</p>
<h3 id="7-13-운영에서-독립-pod를-잘-사용하지-않는-이유">7-13. 운영에서 독립 Pod를 잘 사용하지 않는 이유</h3>
<p>독립 Pod는 자신의 복제본 수를 관리할 상위 컨트롤러가 없다. 컨테이너 재시작은 가능해도 Pod 자체가 삭제되거나 노드가 고장 났을 때 다른 노드에 대체 Pod를 유지하는 역할이 부족하다.</p>
<table>
<thead>
<tr>
<th>워크로드</th>
<th>사용하는 목적</th>
</tr>
</thead>
<tbody><tr>
<td>Deployment</td>
<td>상태 없는 서버 앱의 복제본과 롤아웃</td>
</tr>
<tr>
<td>StatefulSet</td>
<td>안정적인 식별자·저장소 연결 등이 필요한 앱</td>
</tr>
<tr>
<td>DaemonSet</td>
<td>대상 노드마다 에이전트 실행</td>
</tr>
<tr>
<td>Job</td>
<td>완료해야 하는 작업</td>
</tr>
<tr>
<td>CronJob</td>
<td>일정에 따른 Job 실행</td>
</tr>
<tr>
<td>독립 Pod</td>
<td>학습·임시 진단 등 제한된 용도</td>
</tr>
</tbody></table>
<p>Pod가 다른 노드로 그대로 이사하는 것이 아니라, 상위 관리자가 새 Pod를 만들고 그 새 Pod가 배정된다. 메모리와 임시 파일이 그대로 따라가는 이사로 생각하면 안 된다.</p>
<h3 id="7-14-pod-디버깅-도구상자">7-14. Pod 디버깅 도구상자</h3>
<pre><code class="language-bash"># 1. 상태, 준비 여부, 재시작 횟수, 노드 확인
kubectl get pod study-web -o wide

# 2. 준비·실행 실패 이유 확인
kubectl describe pod study-web

# 3. 앱 로그 확인
kubectl logs study-web --tail=100

# 4. 이전 컨테이너가 있었다면 종료 전 로그 확인
kubectl logs study-web --previous

# 5. 필요할 때 컨테이너 내부 확인
kubectl exec -it study-web -- /bin/sh</code></pre>
<p>모든 증상에서 마지막까지 실행할 필요는 없다. 이미지 다운로드 실패라면 앱 컨테이너가 아직 없어서 exec보다 이미지 경로와 인증을 먼저 봐야 한다.</p>
<pre><code class="language-text">어디까지 진행됐나? → get
어느 단계에서 막혔나? → describe
앱이 어떤 말을 남겼나? → logs
추가로 무엇을 관찰해야 하나? → exec / debug / 연결 검사</code></pre>
<h3 id="7-15-7장-핵심-정리">7-15. 7장 핵심 정리</h3>
<p>Pod는 함께 배치되는 컨테이너 묶음이고, 컨테이너마다 재시작될 수 있다. 실행 중인 상태와 요청 가능한 상태는 다르며, 프로브별 실패 결과도 다르다.</p>
<p><strong>이해 확인:</strong> readiness가 실패하면 무슨 일이 일어날까? → 준비되지 않은 상태가 되어 일반적인 Service 트래픽 대상에서 빠질 수 있다. readiness 실패 자체가 컨테이너를 재시작하지는 않는다.</p>
<hr />
<h2 id="8-집중실습-시간-보충-실습-18">8. 집중실습 시간: 보충 실습 1~8</h2>
<blockquote>
<p><strong>원본 문제와의 구분:</strong> 아래 8개는 별도 원본 문제지가 없어 이 글에서 구성한 복습용 실습이다. 실제 수업의 ‘집중실습 1~8’과 번호·내용이 같다는 뜻은 아니다. 원본 문제별 요구사항과 해설은 현재 자료만으로 확인할 수 없다.</p>
<p>다음 명령은 학습용 절차이며 이 글을 작성하면서 실제 클러스터에 실행하지 않았다. 결과 설명은 예상 관찰이다. 이미지 접근, 자원, 권한, 보안 정책에 따라 생성이 거부되거나 시간이 더 걸릴 수 있다.</p>
</blockquote>
<h3 id="실습-공통-준비">실습 공통 준비</h3>
<p>이미 접속이 허용된 실습 클러스터와 안내받은 네임스페이스를 사용한다. 이 실습을 위해 공유 네임스페이스를 새로 만들거나 삭제하지 않는다.</p>
<pre><code class="language-bash"># 실제로 배정받은 네임스페이스로 바꾼다.
export NS=demo
# 다른 사람과 겹치지 않는 소문자 식별자로 바꾼다.
export PREFIX=day1-a17

mkdir -p k8s-day1-practice
cd k8s-day1-practice

kubectl config current-context
kubectl get pods -n &quot;$NS&quot;</code></pre>
<p>이후 명령은 같은 터미널에서 이어서 실행한다. 새 터미널을 사용하면 변수도 다시 설정한다. 이름이 겹치는 리소스가 이미 있다면 PREFIX를 다른 값으로 정한다. 여기서 사용하는 Nginx·BusyBox 버전은 실습 예시이며 운영 이미지 권장 버전의 의미는 아니다.</p>
<table>
<thead>
<tr>
<th>실습</th>
<th>목표</th>
<th>권장 시간</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>접속 대상과 권한 이해</td>
<td>10분</td>
</tr>
<tr>
<td>2</td>
<td>이미지와 컨테이너 구분</td>
<td>15분</td>
</tr>
<tr>
<td>3</td>
<td>선언형 Deployment 적용</td>
<td>15분</td>
</tr>
<tr>
<td>4</td>
<td>Pod 삭제와 조정 루프 관찰</td>
<td>15분</td>
</tr>
<tr>
<td>5</td>
<td>조회·라벨·출력·포트포워딩</td>
<td>15분</td>
</tr>
<tr>
<td>6</td>
<td>반복 종료 원인 진단</td>
<td>15분</td>
</tr>
<tr>
<td>7</td>
<td>readiness와 liveness 차이</td>
<td>20분</td>
</tr>
<tr>
<td>8</td>
<td>init·공유 볼륨·자기 정보 및 정리</td>
<td>15분</td>
</tr>
</tbody></table>
<h3 id="보충-실습-1-나는-어디에-접속했을까">보충 실습 1. 나는 어디에 접속했을까?</h3>
<p><strong>목표:</strong> 컨텍스트, 네임스페이스, 권한이 서로 다른 개념임을 확인한다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl config get-contexts
kubectl get pods -n &quot;$NS&quot;
kubectl auth can-i get pods -n &quot;$NS&quot;
kubectl auth can-i create deployments.apps -n &quot;$NS&quot;
kubectl auth can-i create pods -n &quot;$NS&quot;</code></pre>
<p><strong>예상 관찰:</strong> 현재 선택한 접속 정보와 해당 네임스페이스의 Pod 목록, 각 동작의 권한 결과가 나온다. 클러스터 전체 조회 권한이 없어도 자기 공간은 조회 가능할 수 있다.</p>
<p><strong>해설:</strong> 컨텍스트는 목적지와 인증 선택, 네임스페이스는 리소스의 논리 공간, 권한은 허용된 동작이다. 세 가지를 한 번에 “접속 된다/안 된다”로 묶지 않는다.</p>
<p><strong>확인 질문:</strong> Pod 조회는 되는데 생성이 안 되면 네트워크 문제일까? → 조회가 성공했다면 연결 자체보다 생성 권한이나 정책을 먼저 확인한다.</p>
<h3 id="보충-실습-2-이미지-하나로-컨테이너-두-개-만들기">보충 실습 2. 이미지 하나로 컨테이너 두 개 만들기</h3>
<p><strong>목표:</strong> 공통 이미지에서 서로 다른 실행 인스턴스가 만들어지는 것을 관찰한다. 이 실습은 로컬 Docker를 사용한다.</p>
<pre><code class="language-bash">mkdir -p image-demo
cat &gt; image-demo/index.html &lt;&lt;'EOF_HTML'
&lt;h1&gt;Hello, container!&lt;/h1&gt;
EOF_HTML
cat &gt; image-demo/Dockerfile &lt;&lt;'EOF_DOCKER'
FROM nginx:1.28
COPY index.html /usr/share/nginx/html/index.html
EOF_DOCKER

docker build -t &quot;$PREFIX-web:study&quot; image-demo
docker run -d --name &quot;$PREFIX-local-a&quot; \
  -p 127.0.0.1:18081:80 &quot;$PREFIX-web:study&quot;
docker run -d --name &quot;$PREFIX-local-b&quot; \
  -p 127.0.0.1:18082:80 &quot;$PREFIX-web:study&quot;

curl http://127.0.0.1:18081/
curl http://127.0.0.1:18082/
docker ps --filter &quot;name=$PREFIX-local&quot;</code></pre>
<p>A에만 임시 파일을 만든다.</p>
<pre><code class="language-bash">docker exec &quot;$PREFIX-local-a&quot; sh -c 'echo only-a &gt; /tmp/note.txt'
docker exec &quot;$PREFIX-local-a&quot; cat /tmp/note.txt
docker exec &quot;$PREFIX-local-b&quot; sh -c 'test -f /tmp/note.txt || echo no-file'</code></pre>
<p><strong>예상 관찰:</strong> 두 컨테이너는 같은 웹 문서를 제공하지만 A의 <code>/tmp/note.txt</code>는 B에 없다. 이미지가 같아도 각각의 쓰기층은 다르기 때문이다.</p>
<p><strong>정리:</strong> 본인 실습 컨테이너를 정상 종료한 후 제거한다.</p>
<pre><code class="language-bash">docker stop &quot;$PREFIX-local-a&quot; &quot;$PREFIX-local-b&quot;
docker rm &quot;$PREFIX-local-a&quot; &quot;$PREFIX-local-b&quot;</code></pre>
<p><strong>확인 질문:</strong> A에서 만든 파일이 이미지에 자동 반영될까? → 아니다. 실행 중 쓰기층의 변화와 이미지 빌드는 별개다.</p>
<h3 id="보충-실습-3-yaml을-만들고-적용하기">보충 실습 3. YAML을 만들고 적용하기</h3>
<p><strong>목표:</strong> 매니페스트, Deployment, ReplicaSet, Pod를 연결한다.</p>
<p>먼저 클러스터에 저장하지 않고 초안을 만든다.</p>
<pre><code class="language-bash">kubectl create deployment &quot;$PREFIX-web&quot; \
  --image=nginx:1.28 --replicas=2 \
  -n &quot;$NS&quot; --dry-run=client -o yaml &gt; 03-web.yaml</code></pre>
<p>파일에서 <code>kind</code>, <code>metadata.name</code>, <code>spec.replicas</code>, <code>selector</code>, <code>template</code>을 찾아본다. 환경에서 requests·limits 또는 보안 설정을 요구한다면 정책에 맞는 이미지와 설정을 추가한다.</p>
<pre><code class="language-bash">kubectl apply -n &quot;$NS&quot; --dry-run=server -f 03-web.yaml
kubectl diff -n &quot;$NS&quot; -f 03-web.yaml
kubectl apply -n &quot;$NS&quot; -f 03-web.yaml
kubectl get deployment,replicaset,pod -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot;
kubectl rollout status deployment/&quot;$PREFIX-web&quot; -n &quot;$NS&quot; --timeout=120s</code></pre>
<p><code>diff</code>는 차이가 있으면 종료 코드 1을 반환할 수 있다. 차이를 발견한 정상 결과와 실행 오류를 구분한다. rollout 시간이 초과되면 성공으로 간주하지 말고 Pod Events를 확인한다.</p>
<p>같은 파일을 다시 적용해 본다.</p>
<pre><code class="language-bash">kubectl apply -n &quot;$NS&quot; -f 03-web.yaml</code></pre>
<p><strong>예상 관찰:</strong> 다시 적용했다고 Pod가 2개 더 늘어나지 않는다. 목표가 계속 2개이기 때문이다. 변경 도구나 서버 기본값에 따라 출력 문구는 다를 수 있다.</p>
<p><strong>확인 질문:</strong> YAML을 3개로 수정만 하고 저장하면 클러스터가 즉시 바뀔까? → 별도 자동화가 없는 이 실습에서는 apply해야 전달된다.</p>
<h3 id="보충-실습-4-pod를-지우면-왜-다시-생길까">보충 실습 4. Pod를 지우면 왜 다시 생길까?</h3>
<p><strong>목표:</strong> 동일한 Pod의 부활과 새로운 Pod 생성의 차이를 관찰한다. 실습 3이 준비된 상태에서 진행한다.</p>
<pre><code class="language-bash">export TARGET_POD=$(kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot; \
  -o jsonpath='{.items[0].metadata.name}')

kubectl get pod &quot;$TARGET_POD&quot; -n &quot;$NS&quot; \
  -o custom-columns='NAME:.metadata.name,UID:.metadata.uid,NODE:.spec.nodeName'
kubectl delete pod &quot;$TARGET_POD&quot; -n &quot;$NS&quot;
kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot; -w</code></pre>
<p>변화를 확인한 뒤 <code>Ctrl+C</code>로 관찰을 종료한다.</p>
<p><strong>예상 관찰:</strong> 새 이름과 UID를 가진 Pod가 만들어져 복제본 수를 채운다. 기존 Pod의 종료와 새 Pod 생성이 겹쳐 잠시 목록이 다르게 보일 수 있다.</p>
<p><strong>해설:</strong> 사람이 없어진 직원을 살리는 것이 아니라 빈 자리에 새 직원을 배치하는 것과 같다. 같은 노드 또는 다른 노드에 배치될 수 있으며, 이 실험이 반드시 노드 이동을 보여주는 것은 아니다.</p>
<p><strong>확인 질문:</strong> 계속 Pod를 삭제하면 앱이 멈출까? → 관리자가 계속 보충할 수 있다. 목표 개수나 관리 객체를 변경해야 의도가 유지된다.</p>
<h3 id="보충-실습-5-질문에-맞는-조회-도구-선택하기">보충 실습 5. 질문에 맞는 조회 도구 선택하기</h3>
<p><strong>목표:</strong> 같은 앱을 여러 관점에서 관찰한다.</p>
<pre><code class="language-bash">kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot; -o wide
kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot; --show-labels
kubectl get deployment &quot;$PREFIX-web&quot; -n &quot;$NS&quot; \
  -o jsonpath='{.spec.template.spec.containers[0].image}{&quot;\n&quot;}'

export TARGET_POD=$(kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web&quot; \
  -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod &quot;$TARGET_POD&quot; -n &quot;$NS&quot;
kubectl logs &quot;$TARGET_POD&quot; -n &quot;$NS&quot; --tail=20
kubectl label pod &quot;$TARGET_POD&quot; -n &quot;$NS&quot; study=observed
kubectl get pods -n &quot;$NS&quot; -l &quot;app=$PREFIX-web,study=observed&quot;</code></pre>
<p>포트포워딩을 시작한다.</p>
<pre><code class="language-bash">kubectl port-forward -n &quot;$NS&quot; deployment/&quot;$PREFIX-web&quot; 18080:80</code></pre>
<p>브라우저에서 <code>http://127.0.0.1:18080</code>을 열거나 다른 터미널에서 curl로 호출한다. 확인 후 포트포워딩 터미널에서 <code>Ctrl+C</code>를 누른다.</p>
<p><strong>예상 관찰:</strong> 이미지 참조, 노드·IP, 이벤트, 앱 로그가 각각 다른 명령으로 보인다. 라벨을 붙인 Pod만 선택할 수 있고, 로컬 브라우저에서 웹 응답도 확인한다.</p>
<p><strong>해설:</strong> <code>get -o wide</code>가 답하는 질문은 “어디에 있는가”, logs는 “앱이 무엇을 기록했는가”다. Pod에 즉석으로 붙인 <code>study</code> 라벨은 Deployment 템플릿에는 반영되지 않는다.</p>
<h3 id="보충-실습-6-일부러-종료하는-앱-진단하기">보충 실습 6. 일부러 종료하는 앱 진단하기</h3>
<p><strong>목표:</strong> CrashLoopBackOff를 원인 이름이 아닌 반복 실패의 결과로 이해한다.</p>
<pre><code class="language-bash">kubectl run &quot;$PREFIX-crash&quot; -n &quot;$NS&quot; \
  --image=busybox:1.36 --restart=Always \
  --dry-run=client -o yaml \
  --command -- sh -c 'echo intentional-failure; exit 1' &gt; 06-crash.yaml

kubectl apply -n &quot;$NS&quot; -f 06-crash.yaml
kubectl get pod &quot;$PREFIX-crash&quot; -n &quot;$NS&quot; -w</code></pre>
<p>여러 차례 상태가 바뀌는 것을 보고 <code>Ctrl+C</code>로 관찰을 종료한다.</p>
<pre><code class="language-bash">kubectl describe pod &quot;$PREFIX-crash&quot; -n &quot;$NS&quot;
kubectl logs &quot;$PREFIX-crash&quot; -n &quot;$NS&quot;
kubectl logs &quot;$PREFIX-crash&quot; -n &quot;$NS&quot; --previous</code></pre>
<p><strong>예상 관찰:</strong> 이미지 다운로드가 가능하다면 프로세스가 <code>intentional-failure</code>를 남기고 종료 코드 1로 끝난다. kubelet이 재시작을 반복하며 RESTARTS가 증가하고 BackOff 상태를 볼 수 있다. 첫 재시작 전에는 이전 로그가 없을 수 있다.</p>
<p><strong>해설:</strong> 원인은 <code>exit 1</code>이라는 실행 내용이다. CrashLoopBackOff라는 단어 자체가 코드 오류인지 설정 오류인지 알려주지는 않는다. 종료 코드와 로그가 원인을 좁혀 준다.</p>
<p>관찰 후 반복 실행을 끝낸다.</p>
<pre><code class="language-bash">kubectl delete -n &quot;$NS&quot; -f 06-crash.yaml</code></pre>
<h3 id="보충-실습-7-readiness와-liveness-비교하기">보충 실습 7. readiness와 liveness 비교하기</h3>
<p><strong>목표:</strong> 준비 실패는 트래픽 수신 자격에, 생존 실패는 컨테이너 재시작에 영향을 준다는 차이를 관찰한다.</p>
<p>아래 셸 문서는 <code>$PREFIX</code>를 이름에 넣기 위해 의도적으로 변수 치환을 사용한다.</p>
<pre><code class="language-bash">cat &gt; 07-probes.yaml &lt;&lt;EOF_PROBES
apiVersion: v1
kind: Pod
metadata:
  name: ${PREFIX}-probes
spec:
  containers:
    - name: app
      image: busybox:1.36
      command:
        - sh
        - -c
        - 'touch /tmp/live /tmp/ready; echo started; sleep 3600'
      resources:
        requests:
          cpu: 10m
          memory: 16Mi
        limits:
          cpu: 100m
          memory: 64Mi
      startupProbe:
        exec:
          command: [&quot;test&quot;, &quot;-f&quot;, &quot;/tmp/live&quot;]
        periodSeconds: 2
        failureThreshold: 15
      readinessProbe:
        exec:
          command: [&quot;test&quot;, &quot;-f&quot;, &quot;/tmp/ready&quot;]
        periodSeconds: 2
        failureThreshold: 1
      livenessProbe:
        exec:
          command: [&quot;test&quot;, &quot;-f&quot;, &quot;/tmp/live&quot;]
        periodSeconds: 2
        failureThreshold: 3
  restartPolicy: Always
EOF_PROBES

kubectl apply -n &quot;$NS&quot; -f 07-probes.yaml
kubectl wait -n &quot;$NS&quot; --for=condition=Ready \
  pod/&quot;$PREFIX-probes&quot; --timeout=120s
kubectl get pod &quot;$PREFIX-probes&quot; -n &quot;$NS&quot;</code></pre>
<p>먼저 준비 표시만 지운다.</p>
<pre><code class="language-bash">kubectl exec -n &quot;$NS&quot; &quot;$PREFIX-probes&quot; -- rm /tmp/ready
kubectl get pod &quot;$PREFIX-probes&quot; -n &quot;$NS&quot; -w</code></pre>
<p>READY가 <code>0/1</code>로 바뀌는 것을 보고 <code>Ctrl+C</code>로 관찰을 종료한다. 이 실습에서는 다른 실패가 없다면 RESTARTS는 그대로다. 준비 표시를 복원한다.</p>
<pre><code class="language-bash">kubectl exec -n &quot;$NS&quot; &quot;$PREFIX-probes&quot; -- touch /tmp/ready
kubectl wait -n &quot;$NS&quot; --for=condition=Ready \
  pod/&quot;$PREFIX-probes&quot; --timeout=60s</code></pre>
<p>이제 생존 표시를 지운다.</p>
<pre><code class="language-bash">kubectl exec -n &quot;$NS&quot; &quot;$PREFIX-probes&quot; -- rm /tmp/live
kubectl get pod &quot;$PREFIX-probes&quot; -n &quot;$NS&quot; -w</code></pre>
<p>연속 실패 판정과 종료 유예 때문에 즉시 재시작하지 않을 수 있다. 재시작 횟수가 증가하고 다시 준비되면 <code>Ctrl+C</code>로 관찰을 종료한다.</p>
<pre><code class="language-bash">kubectl describe pod &quot;$PREFIX-probes&quot; -n &quot;$NS&quot;</code></pre>
<p><strong>예상 관찰:</strong> liveness 실패 후 컨테이너가 다시 시작하면 시작 명령이 두 파일을 다시 만든다. Pod 이름은 유지될 수 있지만 RESTARTS는 증가한다.</p>
<p><strong>해설:</strong> 준비 실패와 재시작을 직접 구분한 것이다. 여기서는 Service를 만들지 않았으므로 실제 서비스 트래픽 제외를 측정한 것은 아니다. 관찰한 것은 컨테이너 준비 상태와 재시작 동작이다.</p>
<h3 id="보충-실습-8-initcontainer-공통-볼륨-자기-정보-확인-및-정리">보충 실습 8. initContainer, 공통 볼륨, 자기 정보 확인 및 정리</h3>
<p><strong>목표:</strong> 초기화가 끝난 뒤 앱이 시작되는 과정과 공유 파일, Downward API를 확인한다.</p>
<pre><code class="language-bash">cat &gt; 08-init.yaml &lt;&lt;EOF_INIT
apiVersion: v1
kind: Pod
metadata:
  name: ${PREFIX}-init
spec:
  initContainers:
    - name: prepare
      image: busybox:1.36
      command: [&quot;sh&quot;, &quot;-c&quot;, &quot;echo prepared &gt; /work/message.txt&quot;]
      volumeMounts:
        - name: work
          mountPath: /work
  containers:
    - name: app
      image: busybox:1.36
      command: [&quot;sh&quot;, &quot;-c&quot;, &quot;cat /work/message.txt; sleep 3600&quot;]
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      volumeMounts:
        - name: work
          mountPath: /work
  volumes:
    - name: work
      emptyDir: {}
  restartPolicy: Always
EOF_INIT

kubectl apply -n &quot;$NS&quot; -f 08-init.yaml
kubectl wait -n &quot;$NS&quot; --for=condition=Ready \
  pod/&quot;$PREFIX-init&quot; --timeout=120s
kubectl describe pod &quot;$PREFIX-init&quot; -n &quot;$NS&quot;
kubectl logs &quot;$PREFIX-init&quot; -n &quot;$NS&quot; -c app
kubectl exec &quot;$PREFIX-init&quot; -n &quot;$NS&quot; -c app -- printenv POD_NAME
kubectl exec &quot;$PREFIX-init&quot; -n &quot;$NS&quot; -c app -- cat /work/message.txt</code></pre>
<p><strong>예상 관찰:</strong> initContainer가 성공 종료하고, 앱 로그에 <code>prepared</code>가 보이며, 앱 환경변수에 자신의 Pod 이름이 들어 있다. 초기화가 빨라 중간 STATUS를 못 봐도 describe에서 초기화 컨테이너의 종료 상태를 확인할 수 있다.</p>
<p><strong>해설:</strong> 파일을 이미지에 함께 넣은 것이 아니라, initContainer가 실행 중 만든 파일을 두 컨테이너가 같은 <code>emptyDir</code>로 공유한 것이다. <code>emptyDir</code>은 같은 Pod 안의 컨테이너 재시작 동안 유지될 수 있지만, Pod 수명이 끝나면 영속 데이터로 남지 않는다.</p>
<p><strong>마지막 정리:</strong> 먼저 이 실습에서 만든 대상을 확인한다.</p>
<pre><code class="language-bash">kubectl get -n &quot;$NS&quot; -f 03-web.yaml
kubectl get -n &quot;$NS&quot; -f 07-probes.yaml
kubectl get -n &quot;$NS&quot; -f 08-init.yaml</code></pre>
<p>확인한 본인 리소스를 파일 기준으로 삭제한다.</p>
<pre><code class="language-bash">kubectl delete -n &quot;$NS&quot; -f 03-web.yaml --ignore-not-found
kubectl delete -n &quot;$NS&quot; -f 06-crash.yaml --ignore-not-found
kubectl delete -n &quot;$NS&quot; -f 07-probes.yaml --ignore-not-found
kubectl delete -n &quot;$NS&quot; -f 08-init.yaml --ignore-not-found</code></pre>
<p>삭제 후 같은 파일로 조회하면 대상이 없다는 결과가 나와야 한다. Deployment의 하위 Pod 삭제가 완료되는 데에는 시간이 걸릴 수 있다. 로컬에서 만든 실습 이미지도 더 필요 없을 때 해당 태그만 제거할 수 있다.</p>
<pre><code class="language-bash">docker image rm &quot;$PREFIX-web:study&quot;</code></pre>
<p>이 실습은 로드밸런서나 PVC를 생성하지 않는다. 다른 수업 실습에서 만들었다면 그 리소스의 별도 정리 절차를 따른다.</p>
<hr />
<h2 id="9-헷갈리는-개념-비교표">9. 헷갈리는 개념 비교표</h2>
<table>
<thead>
<tr>
<th>자주 혼동하는 표현</th>
<th>구분해서 이해하기</th>
</tr>
</thead>
<tbody><tr>
<td>이미지 = 컨테이너</td>
<td>이미지는 실행 재료, 컨테이너는 실행 인스턴스</td>
</tr>
<tr>
<td>컨테이너 = 가상머신</td>
<td>리눅스 컨테이너는 커널을 공유하는 격리된 프로세스</td>
</tr>
<tr>
<td>매니페스트 = 클러스터 객체</td>
<td>파일과 API에 등록된 객체는 별도</td>
</tr>
<tr>
<td>파일 저장 = 배포 반영</td>
<td>직접 관리하는 환경에서는 apply 등 전달이 필요</td>
</tr>
<tr>
<td>Pod 삭제 = 앱 영구 종료</td>
<td>상위 컨트롤러가 대체 Pod를 만들 수 있음</td>
</tr>
<tr>
<td>컨테이너 재시작 = Pod 재생성</td>
<td>같은 Pod 안의 컨테이너만 다시 실행될 수 있음</td>
</tr>
<tr>
<td>Running = 서비스 정상</td>
<td>Ready와 실제 요청 결과를 따로 확인</td>
</tr>
<tr>
<td>readiness 실패 = 재시작</td>
<td>준비 상태 변경이며 재시작은 별도</td>
</tr>
<tr>
<td>모든 장애 Pod는 phase!=Running</td>
<td>Running 중인 장애도 있으므로 READY·RESTARTS 등 확인</td>
</tr>
<tr>
<td>localhost = 내 노트북</td>
<td>명령을 실행하는 네트워크 공간 기준</td>
</tr>
<tr>
<td>containerPort = 포트 공개</td>
<td>프로세스 수신·Service·공개 경로는 별도</td>
</tr>
<tr>
<td>Namespace = 완전한 격리</td>
<td>권한·네트워크·자원 정책이 함께 필요</td>
</tr>
<tr>
<td>Service 경로에 Deployment가 있음</td>
<td>Deployment는 관리 객체이지 요청 프록시가 아님</td>
</tr>
<tr>
<td>kube-proxy 중단 = 기존 통신 즉시 중단</td>
<td>설치된 규칙이 남아 있을 수 있음</td>
</tr>
<tr>
<td>base64 = 암호화</td>
<td>되돌릴 수 있는 인코딩</td>
</tr>
<tr>
<td>kubectl get all = 전부 조회</td>
<td>일부 리소스 종류는 별도로 조회</td>
</tr>
<tr>
<td>dry-run 성공 = 앱 정상 실행</td>
<td>API 검증과 실제 기동은 다른 단계</td>
</tr>
</tbody></table>
<h2 id="10-처음-배울-때-스스로-설명해-볼-질문">10. 처음 배울 때 스스로 설명해 볼 질문</h2>
<ol>
<li><strong>쿠버네티스는 왜 필요한가?</strong> 여러 실행 단위를 배치·복구·교체하면서 목표 상태를 유지하기 위해서다.</li>
<li><strong><code>replicas: 3</code>의 의미는?</strong> 관리할 복제본의 목표 수를 3으로 선언한다.</li>
<li><strong>API 서버, 스케줄러, kubelet은 어떻게 다른가?</strong> 요청과 상태 관리의 관문, 노드 선택 담당, 노드의 실행 관리 담당이다.</li>
<li><strong>라벨이 중요한 이유는?</strong> 이름이 바뀌어도 조건에 맞는 대상을 찾고 연결할 수 있다.</li>
<li><strong>이미지가 로컬에서는 되는데 클러스터에서는 실패한다면?</strong> 다운로드 권한·경로·태그·플랫폼·실행 설정·자원 등을 단계별로 확인한다.</li>
<li><strong>Pod가 Running인데 요청을 못 받는다면?</strong> Ready, 포트와 바인딩, Service 선택 조건, 실제 연결 경로를 확인한다.</li>
<li><strong>로그가 없으면 무엇부터 볼까?</strong> describe와 Events로 실행 전 단계에서 막혔는지 확인한다.</li>
<li><strong>왜 앱 데이터를 Pod 밖에 두는가?</strong> Pod가 대체되거나 사라져도 보존할 데이터이기 때문이다.</li>
</ol>
<h2 id="11-공개-학습-기록에-남길-정보">11. 공개 학습 기록에 남길 정보</h2>
<p>이 글의 예제처럼 <strong>개념, 일반화한 명령 형식, 직접 만든 샘플 YAML, 가상 결과와 해설</strong>을 남긴다. 원본 내부 화면이나 접속 결과를 그대로 공개하지 않는다.</p>
<p>실제 계정·이름·식별자, 클러스터 API 주소, 내부 도메인과 IP, 인증 토큰·키·비밀번호, kubeconfig·Docker 인증 파일, Secret 값, 운영 로그·덤프, 내부 보안 설정은 게시물에서 제외한다. 이미지에도 비밀값을 넣지 않고, 공개 전에는 코드 블록과 첨부 파일을 함께 확인한다.</p>
<p>예를 들어 아래 정도면 오류를 설명하기 충분하다.</p>
<pre><code class="language-text">증상: ImagePullBackOff
확인: Pod Events에서 이미지 다운로드 인증 실패 확인
원인: 로컬 로그인과 클러스터 다운로드 권한을 같은 것으로 생각함
학습: 실제 다운로드 주체의 인증 구성을 확인해야 함</code></pre>
<h2 id="12-참고-자료와-정리-범위">12. 참고 자료와 정리 범위</h2>
<p>수업 자료의 1<del>7장을 바탕으로 개념을 재설명했으며, 제공 PDF 기준 3</del>101쪽의 장별 소주제를 다뤘다. PDF 쪽수는 표지부터 센 순서이므로 슬라이드에 인쇄된 번호와 한 쪽 차이가 있다. 각 장의 요약과 아키텍처 그림이 담은 역할도 본문에 포함했다.</p>
<p>공개된 공식 문서로 용어와 동작을 보완했다. 특정 버전에 따라 달라질 수 있는 명령·기능은 사용하는 클러스터의 API와 문서를 기준으로 확인한다.</p>
<ul>
<li><a href="https://kubernetes.io/docs/home/">Kubernetes 공식 문서</a></li>
<li><a href="https://docs.docker.com/">Docker 공식 문서</a></li>
</ul>
<p>원본 교재의 내부 운영 값과 도표 이미지는 게시용 글에 포함하지 않았다. 별도 실습 문제지는 제공되지 않아, 원본 집중실습 1~8의 요구사항과의 일치 여부는 확인하지 못했다.</p>
<p>추천 태그: <code>Kubernetes</code>, <code>Docker</code>, <code>Cloud</code>, <code>DevOps</code>, <code>Pod</code>, <code>kubectl</code>, <code>학습정리</code></p>