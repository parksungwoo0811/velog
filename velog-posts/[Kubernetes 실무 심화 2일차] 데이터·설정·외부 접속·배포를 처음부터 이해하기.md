<h2 id="오늘의-학습-지도">오늘의 학습 지도</h2>
<table>
<thead>
<tr>
<th>파트</th>
<th>배울 내용</th>
<th>해결할 질문</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>Persistent Volume, PV</td>
<td>Pod가 바뀌어도 파일을 남기려면?</td>
</tr>
<tr>
<td>2</td>
<td>StatefulSet과 Headless Service</td>
<td>Pod마다 자기 이름과 저장소가 필요하면?</td>
</tr>
<tr>
<td>3</td>
<td>ConfigMap</td>
<td>같은 이미지에 다른 설정을 주려면?</td>
</tr>
<tr>
<td>4</td>
<td>Secret</td>
<td>민감한 설정은 어떻게 전달할까?</td>
</tr>
<tr>
<td>5</td>
<td>HTTP Ingress와 HTTPS</td>
<td>외부 사용자를 알맞은 앱으로 연결하려면?</td>
</tr>
<tr>
<td>6</td>
<td>무중단 배포</td>
<td>새 버전을 서비스하면서 교체하려면?</td>
</tr>
<tr>
<td>7</td>
<td>퀴즈</td>
<td>비슷한 개념을 구분할 수 있을까?</td>
</tr>
<tr>
<td>8</td>
<td>조별 토론</td>
<td>실제 문제에서 무엇부터 확인할까?</td>
</tr>
</tbody></table>
<p>1일차가 “앱을 컨테이너로 실행하고 개수를 유지하는 법”이었다면, 2일차는 <strong>실행 중인 앱에 데이터·설정·접속 경로·배포 정책을 연결하는 법</strong>입니다.</p>
<pre><code class="language-text">사용자 → 외부 연결 → Ingress 규칙에 따른 전달 → Service의 대상 Pod
                                                    │
                          ┌─────────────────────────┼──────────────┐
                          ↓                         ↓              ↓
                       저장소                    ConfigMap       Secret
                     지속할 데이터               일반 설정       민감한 설정</code></pre>
<p>이 그림은 역할을 보여 주는 논리 구조입니다. 실제 패킷 경로는 Controller와 네트워크 구현에 따라 달라집니다.</p>
<h2 id="예제를-읽기-전에">예제를 읽기 전에</h2>
<p>예제의 <code>demo</code>는 학습용 네임스페이스입니다. 개인 클러스터라면 만들 수 있고, 공유 환경이라면 배정받은 네임스페이스로 바꿉니다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl get namespace demo</code></pre>
<p>네임스페이스가 없고 생성 권한이 있는 개인 학습 환경에서만 다음 명령을 사용합니다.</p>
<pre><code class="language-bash">kubectl create namespace demo</code></pre>
<p><code>learning-sc</code>, <code>example-controller</code>, <code>example-issuer</code>는 설명용 이름입니다. 실제 클러스터의 StorageClass·IngressClass·Issuer로 바꿔야 하는 예제에는 전제 조건을 따로 적었습니다. 모든 YAML을 순서대로 무조건 적용해야 하는 실습 묶음은 아닙니다.</p>
<p><code>kubectl</code> 옵션에는 일반 하이픈 <code>-</code>을 사용합니다. PDF에서 복사한 긴 대시 때문에 <code>–n</code>, <code>–l</code>이 되면 명령이 잘못 해석될 수 있습니다.</p>
<hr />
<h2 id="part-1-persistent-volume-앱이-바뀌어도-데이터를-남기기">PART 1. Persistent Volume: 앱이 바뀌어도 데이터를 남기기</h2>
<h3 id="1-1-pod를-새로-만들었는데-사진이-사라진다면">1-1. Pod를 새로 만들었는데 사진이 사라진다면?</h3>
<p>사진 업로드 앱을 만들었다고 생각해 봅시다. 컨테이너 안의 폴더에 사진을 저장했는데 새 버전을 배포하자 사진이 사라졌습니다.</p>
<p>이유는 앱을 실행하는 공간과 오래 보관할 데이터를 같은 수명에 묶었기 때문일 수 있습니다.</p>
<pre><code class="language-text">앱 컨테이너: 고장 나거나 업데이트하면 교체할 수 있어야 함
업로드한 사진: 앱을 교체해도 남아 있어야 함</code></pre>
<p>학교에서 책상을 바꾼다고 학생의 기록까지 버리면 안 됩니다. 기록은 책상과 별도로 관리해야 합니다. 데이터도 같은 원리입니다.</p>
<h3 id="1-2-volume과-mount를-쉬운-말로-이해하기">1-2. Volume과 Mount를 쉬운 말로 이해하기</h3>
<p><strong>Volume은 컨테이너가 사용할 저장 공간을 연결하는 단위</strong>입니다. <strong>Mount는 그 공간을 컨테이너의 특정 경로에서 보이게 하는 작업</strong>입니다.</p>
<pre><code class="language-text">실제 저장 공간
     ↕ 연결
컨테이너에서 보이는 /data 폴더</code></pre>
<p>같은 창고를 한 사람은 왼쪽 문으로, 다른 사람은 오른쪽 문으로 들어갈 수 있습니다. 마찬가지로 같은 볼륨을 컨테이너마다 다른 경로에 연결할 수 있습니다.</p>
<pre><code class="language-yaml"># Pod에서 사용할 볼륨 정의의 일부
volumes:
  - name: shared-data
    emptyDir: {}</code></pre>
<pre><code class="language-yaml"># 특정 컨테이너에서 사용할 연결 경로의 일부
volumeMounts:
  - name: shared-data
    mountPath: /data</code></pre>
<p><code>volumes</code>는 “무엇을 연결할까?”, <code>volumeMounts</code>는 “이 컨테이너의 어디에서 쓸까?”에 대한 답입니다.</p>
<h3 id="1-3-모든-volume이-영구-저장소는-아니다">1-3. 모든 Volume이 영구 저장소는 아니다</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>어디에 연결되는가?</th>
<th>수명과 특징</th>
<th>알맞은 예</th>
</tr>
</thead>
<tbody><tr>
<td>컨테이너 쓰기 가능 계층</td>
<td>컨테이너 실행 환경</td>
<td>교체 시 보존을 기대하지 않음</td>
<td>실행 중 임시 변경</td>
</tr>
<tr>
<td>emptyDir</td>
<td>해당 Pod의 임시 공간</td>
<td>컨테이너 재시작과 Pod 교체를 구분해야 함</td>
<td>임시 파일·같은 Pod의 파일 전달</td>
</tr>
<tr>
<td>hostPath</td>
<td>특정 노드의 폴더</td>
<td>노드에 종속</td>
<td>제한적인 호스트 연동</td>
</tr>
<tr>
<td>외부 저장소</td>
<td>별도 디스크·파일 시스템</td>
<td>저장소·정책에 따라 지속</td>
<td>업무 데이터·업로드 파일</td>
</tr>
</tbody></table>
<p>emptyDir는 <strong>같은 Pod 안의 컨테이너</strong>가 공유할 수 있습니다. 다른 Pod가 같은 노드에 있다는 이유만으로 같은 emptyDir를 사용하는 것은 아닙니다.</p>
<p>hostPath의 경로는 <code>kubectl</code>을 입력한 노트북이 아니라 <strong>Pod가 실행되는 노드의 경로</strong>입니다. 다른 노드에 같은 경로가 있어도 같은 데이터라는 보장은 없습니다.</p>
<h3 id="1-4-메모리-기반-emptydir는-무엇일까">1-4. 메모리 기반 emptyDir는 무엇일까?</h3>
<pre><code class="language-yaml">volumes:
  - name: temporary-data
    emptyDir:
      medium: Memory
      sizeLimit: 64Mi</code></pre>
<p>이 설정은 임시 공간을 메모리 기반 파일 시스템으로 사용합니다. 빠른 임시 작업에 쓸 수 있지만 메모리 자원을 사용합니다. 저장 공간 이름에 Volume이 들어간다고 전원이 꺼져도 남는 디스크가 되는 것은 아닙니다.</p>
<p><code>sizeLimit</code>도 “이만큼의 여유 메모리가 무조건 예약된다”는 뜻으로 이해하지 않습니다. Pod 자원 설정과 노드 여유를 함께 봅니다.</p>
<h3 id="1-5-pv와-pvc-창고와-창고-사용-신청서">1-5. PV와 PVC: 창고와 창고 사용 신청서</h3>
<p>Kubernetes는 앱이 실제 저장 장치의 모든 세부 사항을 직접 알지 않아도 되도록 역할을 나눕니다.</p>
<table>
<thead>
<tr>
<th>리소스</th>
<th>의미</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>PV, PersistentVolume</td>
<td>저장소의 연결 정보·용량·특성을 표현</td>
<td>사용할 수 있는 창고 공간</td>
</tr>
<tr>
<td>PVC, PersistentVolumeClaim</td>
<td>앱이 필요한 저장소 조건을 요청</td>
<td>창고 사용 신청서</td>
</tr>
<tr>
<td>StorageClass</td>
<td>저장소를 준비하는 정책</td>
<td>창고 상품의 종류</td>
</tr>
<tr>
<td>CSI Driver</td>
<td>저장소 기능을 실제로 연결하는 구현</td>
<td>창고 준비·연결 담당자</td>
</tr>
</tbody></table>
<pre><code class="language-text">Pod → PVC → PV → 실제 스토리지</code></pre>
<p>PV라는 Kubernetes 객체 안에 파일 데이터가 전부 들어가는 것이 아닙니다. PV는 실제 저장소와 연결할 정보 등을 표현하고, 파일은 실제 저장 장치에 있습니다.</p>
<h3 id="1-6-pv는-클러스터-범위-pvc는-네임스페이스-범위">1-6. PV는 클러스터 범위, PVC는 네임스페이스 범위</h3>
<p>PV는 클러스터 전체에서 관리하는 리소스입니다. PVC는 특정 네임스페이스에 속합니다.</p>
<p>일반적인 Pod의 PVC 참조는 같은 네임스페이스 안의 PVC를 사용합니다. 다른 네임스페이스의 PVC 이름만 적어서 바로 사용할 수 있는 구조로 생각하지 않습니다.</p>
<p><strong>PV와 PVC의 바인딩은 보통 일대일</strong>입니다. 같은 네임스페이스의 여러 Pod가 하나의 PVC를 참조할 수 있는지는 저장소와 접근 모드, 배치 조건 등에 달려 있습니다. “공유 저장소이므로 여러 PVC가 아무렇게나 같은 PV에 붙는다”는 뜻은 아닙니다.</p>
<h3 id="1-7-바인딩과-마운트는-다르다">1-7. 바인딩과 마운트는 다르다</h3>
<pre><code class="language-text">① PVC가 조건에 맞는 PV와 연결됨 → Bound
② Pod가 배치됨
③ 노드에서 저장소 연결 준비
④ 컨테이너 경로에 마운트
⑤ 앱이 파일 읽기·쓰기</code></pre>
<p>Bound는 “사용 신청과 저장소가 논리적으로 연결됐다”는 뜻입니다. 앱이 이미 파일을 읽고 있다는 뜻은 아닙니다.</p>
<p>예약한 창고 번호가 배정되는 것과 실제로 창고 문을 열어 사용하는 것은 다릅니다. PVC가 Bound여도 네트워크·권한·드라이버 오류 때문에 Pod의 마운트가 실패할 수 있습니다.</p>
<h3 id="1-8-정적-프로비저닝-미리-준비한-저장소-사용">1-8. 정적 프로비저닝: 미리 준비한 저장소 사용</h3>
<p><strong>Provisioning은 필요한 저장 공간을 준비하는 작업</strong>입니다.</p>
<p>정적 프로비저닝에서는 관리자가 실제 저장소와 PV를 미리 준비하고, 사용자가 PVC로 요청합니다.</p>
<pre><code class="language-text">관리자: 저장소 준비 → PV 등록
사용자: PVC 생성 → 조건에 맞는 PV와 바인딩 → Pod에서 사용</code></pre>
<p>다음은 <strong>구조를 읽기 위한 NFS 예제</strong>입니다. <code>192.0.2.10</code>은 문서용 주소이므로 그대로 접속되지 않습니다. 실제 NFS 서버·내보내기 경로·네트워크 권한·노드의 지원 환경이 있어야 합니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: PersistentVolume
metadata:
  name: learning-nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: &quot;&quot;
  nfs:
    server: 192.0.2.10
    path: /exports/learning
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: learning-nfs-claim
  namespace: demo
spec:
  storageClassName: &quot;&quot;
  volumeName: learning-nfs-pv
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi</code></pre>
<p><code>storageClassName: &quot;&quot;</code>는 기본 StorageClass를 자동 선택하지 않겠다는 명시적인 빈 값입니다. 이 예제는 <code>volumeName</code>으로 사용할 PV도 지정했습니다.</p>
<p>용량 선언이 모든 NFS 구현에서 폴더별 디스크 사용량을 실제로 제한해 주는 것은 아닙니다. 실제 할당량은 저장소의 기능과 관리 정책을 확인합니다.</p>
<h3 id="1-9-동적-프로비저닝-신청하면-준비하기">1-9. 동적 프로비저닝: 신청하면 준비하기</h3>
<p>동적 프로비저닝은 PVC 요청을 보고 저장소와 PV를 자동으로 준비하는 방식입니다. 이를 지원하는 프로비저너·드라이버와 StorageClass가 필요합니다.</p>
<pre><code class="language-text">PVC 생성
   ↓ StorageClass 선택
프로비저너·CSI 드라이버가 실제 저장소 준비
   ↓
PV 생성 및 PVC와 바인딩
   ↓
Pod에서 사용</code></pre>
<p>호텔로 비유하면 정적 방식은 이미 정해 놓은 방을 배정하고, 동적 방식은 요청 조건에 맞춰 사용할 공간을 준비하는 서비스에 가깝습니다.</p>
<h3 id="1-10-storageclass는-저장소의-주문-양식이다">1-10. StorageClass는 저장소의 주문 양식이다</h3>
<table>
<thead>
<tr>
<th>필드</th>
<th>묻는 질문</th>
</tr>
</thead>
<tbody><tr>
<td><code>provisioner</code></td>
<td>어떤 드라이버·프로비저너로 준비할까?</td>
</tr>
<tr>
<td><code>parameters</code></td>
<td>어떤 디스크 종류·파일 시스템·옵션을 사용할까?</td>
</tr>
<tr>
<td><code>reclaimPolicy</code></td>
<td>사용이 끝난 저장소를 어떻게 처리할까?</td>
</tr>
<tr>
<td><code>volumeBindingMode</code></td>
<td>언제 바인딩·프로비저닝할까?</td>
</tr>
<tr>
<td><code>allowVolumeExpansion</code></td>
<td>지원되는 경우 용량 확장을 허용할까?</td>
</tr>
</tbody></table>
<pre><code class="language-bash">kubectl get storageclass
kubectl describe storageclass learning-sc</code></pre>
<p>StorageClass의 필드가 같아도 <code>parameters</code>의 구체적인 키는 드라이버마다 다릅니다. 다른 클라우드의 설정을 이름만 바꾸어 사용하면 안 됩니다.</p>
<p>용량 확장은 StorageClass의 허용 여부와 드라이버 지원, 파일 시스템 조건을 함께 확인합니다. PVC 요청 용량을 키우는 것과 줄이는 것은 대칭 기능이 아닙니다. 늘릴 수 있다고 같은 방식으로 축소할 수 있다고 가정하지 않습니다.</p>
<h3 id="1-11-immediate와-waitforfirstconsumer">1-11. Immediate와 WaitForFirstConsumer</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>이해할 핵심</th>
</tr>
</thead>
<tbody><tr>
<td>Immediate</td>
<td>PVC가 생기면 즉시 바인딩·프로비저닝을 시작할 수 있음</td>
</tr>
<tr>
<td>WaitForFirstConsumer</td>
<td>실제로 사용할 Pod의 배치 조건을 고려할 때까지 기다림</td>
</tr>
</tbody></table>
<p>특정 가용 영역, 즉 AZ에 묶이는 디스크라면 Pod와 디스크가 서로 맞는 위치에 있어야 합니다. 이때 WaitForFirstConsumer가 도움이 됩니다.</p>
<pre><code class="language-text">잘못 맞춘 경우: Pod는 영역 A, 디스크는 영역 B
위치를 함께 고려한 경우: Pod와 디스크가 사용 가능한 같은 영역에 준비됨</code></pre>
<p>따라서 PVC가 Pending이어도 이 모드에서는 정상적으로 소비할 Pod를 기다리는 것일 수 있습니다. 모든 Pending을 실패라고 판단하지 않습니다. <a href="https://kubernetes.io/docs/concepts/storage/storage-classes/">공식 문서: Storage Classes</a></p>
<h3 id="1-12-동적-pvc와-pod-예제">1-12. 동적 PVC와 Pod 예제</h3>
<p>다음 예제는 실제 클러스터에 사용할 수 있는 <code>learning-sc</code>가 있다는 가정입니다. 없으면 관리자가 안내한 StorageClass 이름으로 바꿉니다.</p>
<p><code>pvc-demo.yaml</code>:</p>
<pre><code class="language-yaml">apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: demo
spec:
  storageClassName: learning-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: storage-reader
  namespace: demo
spec:
  containers:
    - name: toolbox
      image: busybox:1.37
      command: [&quot;sleep&quot;]
      args: [&quot;3600&quot;]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: app-data</code></pre>
<pre><code class="language-bash">kubectl apply -f pvc-demo.yaml
kubectl get pvc app-data -n demo
kubectl get pod storage-reader -n demo
kubectl describe pod storage-reader -n demo</code></pre>
<p>Pod가 실행된 뒤 파일을 만들어 확인합니다.</p>
<pre><code class="language-bash">kubectl exec storage-reader -n demo -- sh -c 'echo hello &gt; /data/message.txt'
kubectl exec storage-reader -n demo -- cat /data/message.txt</code></pre>
<p>Pod만 삭제하고 같은 PVC를 사용하는 Pod를 다시 만들면 파일을 다시 읽는 실습을 할 수 있습니다. <strong>PVC까지 삭제하는 실습과 구분</strong>해야 합니다.</p>
<h3 id="1-13-access-mode를-정확하게-읽기">1-13. Access Mode를 정확하게 읽기</h3>
<table>
<thead>
<tr>
<th>모드</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>ReadWriteOnce, RWO</td>
<td>한 노드에서 읽기·쓰기 마운트 허용</td>
</tr>
<tr>
<td>ReadOnlyMany, ROX</td>
<td>여러 노드에서 읽기 전용 마운트 허용</td>
</tr>
<tr>
<td>ReadWriteMany, RWX</td>
<td>여러 노드에서 읽기·쓰기 마운트 허용</td>
</tr>
<tr>
<td>ReadWriteOncePod, RWOP</td>
<td>지원되는 CSI 환경에서 단일 Pod의 읽기·쓰기 접근으로 제한</td>
</tr>
</tbody></table>
<p><strong>RWO는 “Pod 한 개”와 같지 않습니다.</strong> 같은 노드의 여러 Pod가 해당 볼륨을 사용할 수 있는 경우가 있습니다. 단일 Pod 접근 제한을 원한다면 RWOP의 지원 조건을 확인합니다.</p>
<p>접근 모드는 파일 잠금과 DB 트랜잭션 설계를 대신하지 않습니다. RWX로 연결해도 두 앱이 같은 파일을 동시에 덮어쓰면 충돌할 수 있습니다. <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes">공식 문서: PV 접근 모드</a></p>
<h3 id="1-14-ebs-efs-nfs는-어떻게-다를까">1-14. EBS, EFS, NFS는 어떻게 다를까?</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>저장소 관점</th>
<th>생각할 조건</th>
</tr>
</thead>
<tbody><tr>
<td>EBS</td>
<td>서버에 디스크처럼 연결하는 AWS 블록 스토리지</td>
<td>AZ, 연결 제한, 지원 볼륨·드라이버</td>
</tr>
<tr>
<td>EFS</td>
<td>네트워크로 사용하는 AWS 관리형 파일 시스템</td>
<td>네트워크 접근, 권한, 공유 사용</td>
</tr>
<tr>
<td>NFS</td>
<td>파일 시스템을 네트워크로 공유하는 프로토콜·구성</td>
<td>서버 운영, 연결 가능성, 권한</td>
</tr>
</tbody></table>
<p>블록 스토리지는 새 공책을 받아 파일 시스템이라는 정리 방법을 적용하는 것에 가깝고, 네트워크 파일 스토리지는 이미 있는 공용 파일함에 접속하는 것에 가깝습니다.</p>
<p>“EBS는 어떤 경우에도 단일 연결만 가능하다”, “EFS면 모든 앱의 DB 저장소로 가장 좋다”처럼 단정하지 않습니다. 실제 공유·연결 가능 범위는 제품과 드라이버, 볼륨 종류, 앱의 요구사항에 달려 있습니다.</p>
<h3 id="1-15-efs-file-system-id와-access-point">1-15. EFS File System ID와 Access Point</h3>
<p>File System ID는 어떤 EFS 파일 시스템인지를 구분하는 식별자입니다. Access Point는 그 파일 시스템 안에서 사용할 진입 경로와 POSIX 사용자·그룹, 디렉터리 권한 등을 정하는 데 쓰입니다.</p>
<pre><code class="language-text">파일 시스템
├─ 팀 A용 경로 ← Access Point A
└─ 팀 B용 경로 ← Access Point B</code></pre>
<p>동적 EFS 프로비저닝에서는 기존 파일 시스템 안에 PVC별 Access Point를 준비하는 구성이 가능합니다. <strong>PVC마다 새로운 EFS 파일 시스템 전체를 반드시 만드는 것은 아닙니다.</strong></p>
<p>권한을 쉽게 통과시키려고 모든 경로를 무조건 <code>777</code>로 여는 대신, 앱의 사용자·그룹과 필요한 접근 범위를 맞춥니다. 실제 File System ID와 Access Point ID는 공개 예제에 옮기지 않습니다.</p>
<h3 id="1-16-csi는-저장소와-대화하는-공통-규칙이다">1-16. CSI는 저장소와 대화하는 공통 규칙이다</h3>
<p>CSI는 Container Storage Interface의 약자입니다. 여러 저장소를 일정한 인터페이스로 다루기 위한 규칙이고, CSI Driver는 그 규칙을 구현한 프로그램입니다.</p>
<pre><code class="language-text">Kubernetes의 저장소 요청
        ↓ CSI 규칙
저장소별 드라이버
        ↓
실제 디스크·파일 시스템</code></pre>
<p>흔히 Controller 쪽 구성 요소는 생성·삭제·연결 등 중앙 관리 작업을, Node 쪽 플러그인은 각 노드의 마운트 작업 등을 담당합니다. Controller 플러그인이 반드시 Control Plane 노드에서만 실행되어야 하는 것은 아닙니다.</p>
<p>블록 디스크는 노드에 Attach한 뒤 파일 시스템을 마운트하는 과정이 있을 수 있습니다. 네트워크 파일 시스템은 같은 방식의 블록 장치 Attach 없이 네트워크 마운트하는 등 동작이 다릅니다.</p>
<h3 id="1-17-retain과-delete-지울-때가-더-중요하다">1-17. Retain과 Delete: 지울 때가 더 중요하다</h3>
<table>
<thead>
<tr>
<th>회수 정책</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Retain</td>
<td>사용 종료 후 저장소를 남겨 관리자가 후속 처리</td>
</tr>
<tr>
<td>Delete</td>
<td>지원 프로비저너가 PV와 대응 저장소를 삭제하는 방향으로 처리</td>
</tr>
</tbody></table>
<p>Retain이라고 새 PVC가 기존 데이터를 자동으로 즉시 이어받는 것은 아닙니다. Released 상태와 재사용 절차, 권한 등을 관리해야 합니다.</p>
<p>Delete도 모든 파일 시스템 전체가 같은 방식으로 삭제된다는 뜻은 아닙니다. EFS처럼 공유 파일 시스템과 Access Point를 사용하는 드라이버에서는 실제로 삭제되는 대상과 디렉터리 처리 정책을 확인해야 합니다.</p>
<p>Pod 삭제, PVC 삭제, PV 삭제, 실제 저장소 삭제는 서로 다른 사건입니다. 정리 명령을 실행하기 전에 무엇이 함께 없어지는지 확인합니다.</p>
<h3 id="1-18-저장소-문제를-확인하는-순서">1-18. 저장소 문제를 확인하는 순서</h3>
<pre><code class="language-bash">kubectl get pvc -n demo
kubectl describe pvc app-data -n demo
kubectl get pv
kubectl get storageclass
kubectl describe pod storage-reader -n demo</code></pre>
<table>
<thead>
<tr>
<th>증상</th>
<th>먼저 볼 것</th>
</tr>
</thead>
<tbody><tr>
<td>PVC Pending</td>
<td>StorageClass, 사용 가능한 PV, 바인딩 모드, 이벤트</td>
</tr>
<tr>
<td>PVC Bound인데 Pod 생성 지연</td>
<td>마운트 오류, 노드·영역 조건, 드라이버, 권한</td>
</tr>
<tr>
<td>읽기는 되는데 쓰기 실패</td>
<td>파일 권한, UID/GID, 읽기 전용 설정</td>
</tr>
<tr>
<td>다른 Pod에서 파일이 안 보임</td>
<td>같은 PVC인지, 공유 지원 조건, 마운트 경로</td>
</tr>
<tr>
<td>Pod 교체 후 파일 유실</td>
<td>실제 저장 위치, PVC 연결, 삭제 정책</td>
</tr>
</tbody></table>
<p><strong>이 파트의 핵심:</strong> 앱을 교체하는 수명과 데이터를 보관하는 수명을 분리합니다. PV/PVC를 썼다는 사실만으로 백업과 복구까지 끝나는 것은 아닙니다.</p>
<hr />
<h2 id="part-2-statefulset--headless-service-pod마다-자기-자리-만들기">PART 2. StatefulSet &amp; Headless Service: Pod마다 자기 자리 만들기</h2>
<h3 id="2-1-stateless와-stateful을-먼저-구분하자">2-1. Stateless와 Stateful을 먼저 구분하자</h3>
<p>편의점 계산대에서는 어느 직원에게 계산을 맡겨도 같은 결과를 기대합니다. 반면 개인 사물함은 “비어 있는 아무 사물함”이 아니라 내 번호의 사물함을 찾아야 합니다.</p>
<pre><code class="language-text">Stateless 관점: 어느 복제본이 처리해도 괜찮게 설계
Stateful 관점: 개별 구성원의 이름·저장 데이터·관계가 중요</code></pre>
<p>Stateless 앱이라고 서비스 전체에 데이터가 없다는 뜻은 아닙니다. 데이터와 세션을 외부 저장소로 분리해 앱 복제본 자체를 교체하기 쉽게 설계할 수 있습니다.</p>
<h3 id="2-2-deployment와-statefulset-비교">2-2. Deployment와 StatefulSet 비교</h3>
<table>
<thead>
<tr>
<th>질문</th>
<th>Deployment</th>
<th>StatefulSet</th>
</tr>
</thead>
<tbody><tr>
<td>개별 Pod를 구분해야 하나?</td>
<td>대체 가능한 복제본 관리에 적합</td>
<td>안정적인 개별 정체성 제공</td>
</tr>
<tr>
<td>이름은?</td>
<td>생성 시 접미사 등이 달라질 수 있음</td>
<td>ordinal을 포함한 이름</td>
</tr>
<tr>
<td>Pod별 PVC는?</td>
<td>별도 설계 가능</td>
<td>volumeClaimTemplates로 구성 가능</td>
</tr>
<tr>
<td>순서가 중요한가?</td>
<td>일반적으로 개별 순서를 보장하지 않음</td>
<td>기본 OrderedReady 정책으로 순서 제어</td>
</tr>
<tr>
<td>예시</td>
<td>웹·API 복제본</td>
<td>개별 저장소·네트워크 이름이 필요한 시스템</td>
</tr>
</tbody></table>
<p>Deployment도 PVC를 연결해 사용할 수 있습니다. “Deployment에는 저장소를 붙일 수 없다”는 뜻이 아닙니다. StatefulSet은 <strong>각 Pod의 정체성과 저장소 관계를 관리하기 위한 기능</strong>을 제공하는 것입니다.</p>
<h3 id="2-3-store-0-store-1-store-2의-의미">2-3. <code>store-0</code>, <code>store-1</code>, <code>store-2</code>의 의미</h3>
<p>StatefulSet 이름이 <code>store</code>라면 기본적으로 다음 이름의 Pod를 만들 수 있습니다.</p>
<pre><code class="language-text">store-0
store-1
store-2</code></pre>
<p>뒤의 숫자를 ordinal이라고 합니다. 기본 시작 번호는 0입니다.</p>
<p><code>store-1</code>이 교체되면 같은 이름의 새 Pod가 만들어질 수 있습니다. 다만 <strong>같은 이름이라고 같은 Pod 객체는 아닙니다.</strong> UID와 IP는 바뀔 수 있습니다.</p>
<p>주민이 이사해도 집 주소를 유지하는 것처럼, “역할을 찾는 이름”이 안정적으로 유지되는 데 의미가 있습니다.</p>
<h3 id="2-4-pod마다-자기-pvc를-가진다">2-4. Pod마다 자기 PVC를 가진다</h3>
<pre><code class="language-text">store-0 → data-store-0 → 저장소 0
store-1 → data-store-1 → 저장소 1
store-2 → data-store-2 → 저장소 2</code></pre>
<p><code>volumeClaimTemplates</code>는 Pod별 PVC를 만들기 위한 설계도입니다. 이름에 <code>Claim</code>이 들어가는 것은 PV가 아니라 PVC를 요청하는 템플릿이기 때문입니다.</p>
<p><code>store-1</code>이 다시 만들어져도 해당 PVC가 남아 있고 사용 조건이 맞으면 자기 저장소를 다시 연결할 수 있습니다. 모든 복제본이 같은 디렉터리에 같은 파일을 쓰는 방식과는 다릅니다.</p>
<h3 id="2-5-기본-생성·축소·업데이트-순서">2-5. 기본 생성·축소·업데이트 순서</h3>
<p>기본 OrderedReady 정책을 이해하기 위한 흐름입니다.</p>
<pre><code class="language-text">생성: store-0 준비 → store-1 준비 → store-2 준비
축소: 높은 번호부터 제거
일반 RollingUpdate: 높은 번호부터 교체하고 준비를 확인</code></pre>
<p><code>podManagementPolicy: Parallel</code> 등 설정에 따라 달라질 수 있습니다. 모든 StatefulSet이 어떤 상황에서도 같은 순서로만 움직인다고 단정하지 않습니다.</p>
<p>기본 RollingUpdate는 같은 ordinal의 기존 Pod를 교체하므로 해당 구성원이 잠시 가용하지 않을 수 있습니다. 데이터베이스 클러스터의 중단 허용 범위와 복제·정족수 조건을 함께 설계합니다.</p>
<h3 id="2-6-축소하면-데이터도-자동-삭제될까">2-6. 축소하면 데이터도 자동 삭제될까?</h3>
<p>StatefulSet의 PVC는 기본적으로 Pod 축소나 StatefulSet 삭제와 함께 자동으로 전부 지워지는 방식이 아닙니다. 설정 가능한 PVC 보존 정책도 있으므로 실제 리소스를 확인해야 합니다.</p>
<p>“StatefulSet을 지웠는데 디스크 비용이 계속 나온다”는 현상은 이런 보존 설정과 연결될 수 있습니다. 반대로 삭제 정책을 바꾸면 데이터 손실 범위도 달라집니다. <a href="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/">공식 문서: StatefulSets</a></p>
<h3 id="2-7-노드가-응답하지-않을-때-강제-삭제를-조심하는-이유">2-7. 노드가 응답하지 않을 때 강제 삭제를 조심하는 이유</h3>
<p>어떤 Pod가 네트워크 문제로 보이지 않는다고 해서 그 앱 프로세스가 반드시 멈췄다는 뜻은 아닙니다.</p>
<p>그 상태에서 같은 정체성의 앱을 다른 위치에 강제로 만들면 둘이 동시에 동작하는 상황이 생길 수 있습니다. 데이터 시스템에서는 이를 피하기 위한 격리와 복구 절차가 중요합니다.</p>
<p>따라서 “안 바뀌면 무조건 <code>--force</code>”를 기본 해결책으로 외우지 않습니다. 노드와 저장소 연결 상태, 이전 프로세스의 종료 여부를 판단해야 합니다.</p>
<h3 id="2-8-statefulset은-db-복제를-자동-구현하지-않는다">2-8. StatefulSet은 DB 복제를 자동 구현하지 않는다</h3>
<p>StatefulSet으로 DB Pod 3개를 만들었다고 데이터 동기화와 리더 선출이 자동 완성되지는 않습니다.</p>
<pre><code class="language-text">StatefulSet: 이름·생성 관리·PVC 연결 제공
DB 소프트웨어와 운영 구성: 복제·리더 선출·데이터 일관성 처리</code></pre>
<p>가게에 사물함 3개를 설치하는 것과 세 사물함의 내용물을 자동으로 똑같이 유지하는 장치를 만드는 것은 다른 일입니다.</p>
<h3 id="2-9-headless-service는-왜-필요한가">2-9. Headless Service는 왜 필요한가?</h3>
<p>일반적인 ClusterIP Service는 하나의 가상 IP를 연결 지점으로 제공합니다. Headless Service는 <code>clusterIP: None</code>을 지정해 이런 가상 IP를 할당하지 않습니다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>이해할 관점</th>
</tr>
</thead>
<tbody><tr>
<td>일반 ClusterIP Service</td>
<td>대표 연결 지점으로 접근</td>
</tr>
<tr>
<td>Headless Service</td>
<td>DNS를 통해 개별 Pod의 주소를 찾는 데 활용</td>
</tr>
</tbody></table>
<pre><code class="language-text">대표 번호: 서점에 전화하면 가능한 직원에게 연결
개별 번호: 특정 담당자에게 직접 연락</code></pre>
<p>Selector가 있는 일반적인 Headless Service에서는 DNS가 연결 대상 Pod 주소를 반환할 수 있습니다. 이것이 서버 쪽에서 요청을 반드시 균등 분산한다는 뜻은 아닙니다. 클라이언트가 반환된 주소를 어떻게 선택하는지도 중요합니다.</p>
<h3 id="2-10-servicename과-개별-dns-이름">2-10. <code>serviceName</code>과 개별 DNS 이름</h3>
<p>StatefulSet의 <code>serviceName</code>은 해당 Pod들의 네트워크 정체성을 관리할 Service 이름을 지정합니다. <strong>Service를 자동으로 생성하는 필드는 아닙니다.</strong> 별도 Service가 필요합니다.</p>
<p>예를 들어 다음 이름을 생각할 수 있습니다.</p>
<pre><code class="language-text">store-0.store-headless.demo.svc.cluster.local
│       │              │
Pod     Service        Namespace</code></pre>
<p>위는 클러스터 도메인이 <code>cluster.local</code>인 환경의 예입니다. 실제 클러스터 도메인은 다를 수 있습니다.</p>
<p>Pod가 교체되어 IP가 바뀌어도 이름을 통해 새 주소를 찾을 수 있습니다. DNS 캐시와 준비 상태 때문에 반영에 시간이 걸릴 수는 있습니다.</p>
<h3 id="2-11-headless-service와-statefulset-예제">2-11. Headless Service와 StatefulSet 예제</h3>
<p>다음은 실제 <code>learning-sc</code>를 사용할 수 있는 환경을 가정한 예제입니다. 학습용 웹서버로 이름과 저장소 연결을 확인하며, DB 클러스터를 만드는 예제는 아닙니다.</p>
<p><code>stateful-demo.yaml</code>:</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Service
metadata:
  name: store-headless
  namespace: demo
spec:
  clusterIP: None
  selector:
    app: store
  ports:
    - name: http
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: store
  namespace: demo
spec:
  serviceName: store-headless
  replicas: 2
  selector:
    matchLabels:
      app: store
  template:
    metadata:
      labels:
        app: store
    spec:
      containers:
        - name: web
          image: nginx:1.27-alpine
          ports:
            - name: http
              containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: http
          volumeMounts:
            - name: data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        storageClassName: learning-sc
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi</code></pre>
<pre><code class="language-bash">kubectl apply -f stateful-demo.yaml
kubectl get statefulset,pod,pvc -n demo
kubectl exec store-0 -n demo -- sh -c 'echo store-zero &gt; /data/identity.txt'
kubectl exec store-0 -n demo -- cat /data/identity.txt</code></pre>
<p>Pod <code>store-0</code>, <code>store-1</code>과 PVC <code>data-store-0</code>, <code>data-store-1</code>의 관계를 확인합니다. 실습을 정리할 때는 PVC와 실제 저장소의 회수 정책까지 따로 확인합니다.</p>
<h3 id="2-12-dns와-개별-pod-호출-확인">2-12. DNS와 개별 Pod 호출 확인</h3>
<p>클러스터 내부에서 DNS를 확인하는 보충 예시입니다.</p>
<pre><code class="language-bash">kubectl run dns-check -n demo --rm -i --restart=Never \
  --image=busybox:1.37 -- nslookup store-headless.demo.svc.cluster.local

kubectl run pod-call -n demo --rm -i --restart=Never \
  --image=busybox:1.37 -- wget -qO- \
  http://store-0.store-headless.demo.svc.cluster.local/</code></pre>
<p>Pod가 Ready가 되기 전에는 DNS에 기대한 주소가 보이지 않을 수 있습니다. 초기 클러스터 구성을 위해 준비 전 주소 공개가 필요하다면 <code>publishNotReadyAddresses</code>의 의미와 영향을 따로 검토합니다. 무조건 켜는 옵션은 아닙니다.</p>
<p>일반 Service와 Headless Service를 함께 두어, 일반 앱 접속은 대표 주소로 하고 구성원 간 통신은 개별 주소로 하는 구성도 가능합니다. <a href="https://kubernetes.io/docs/concepts/services-networking/service/#headless-services">공식 문서: Headless Services</a></p>
<p><strong>이 파트의 핵심:</strong> StatefulSet은 Pod마다 자기 이름과 저장소 관계를 유지하고, Headless Service는 개별 구성원을 찾을 수 있게 돕습니다.</p>
<hr />
<h2 id="part-3-configmap-앱과-설정을-분리하기">PART 3. ConfigMap: 앱과 설정을 분리하기</h2>
<h3 id="3-1-설정이-바뀔-때마다-이미지를-다시-만들까">3-1. 설정이 바뀔 때마다 이미지를 다시 만들까?</h3>
<p>개발 환경에서는 로그를 자세히 보고, 운영 환경에서는 꼭 필요한 로그만 보고 싶습니다. 그렇다고 앱 코드를 바꾸고 이미지를 매번 다시 만들어야 할까요?</p>
<pre><code class="language-text">같은 앱 이미지
├─ 개발 설정: LOG_LEVEL=DEBUG
└─ 운영 설정: LOG_LEVEL=INFO</code></pre>
<p><strong>ConfigMap은 민감하지 않은 설정을 키·값 형태로 저장하는 Kubernetes 리소스</strong>입니다. 앱 이미지와 환경별 설정을 분리하는 데 사용합니다.</p>
<p>게임 프로그램은 같고, 난이도·음량·언어 옵션을 다르게 설정하는 것과 비슷합니다.</p>
<h3 id="3-2-어떤-값을-configmap에-넣을까">3-2. 어떤 값을 ConfigMap에 넣을까?</h3>
<table>
<thead>
<tr>
<th>적절한 예</th>
<th>Secret 등 별도 보호가 필요한 예</th>
</tr>
</thead>
<tbody><tr>
<td>로그 수준</td>
<td>실제 비밀번호</td>
</tr>
<tr>
<td>화면 제목</td>
<td>개인키</td>
</tr>
<tr>
<td>공개 가능한 기능 플래그</td>
<td>인증 토큰</td>
</tr>
<tr>
<td>일반 설정 파일</td>
<td>민감한 접속 자격증명</td>
</tr>
</tbody></table>
<p>주소 자체도 내부 시스템 정보라면 공개 문서에는 넣지 않습니다. “ConfigMap에 넣을 수 있는 값”과 “인터넷에 공개해도 되는 값”은 다른 판단입니다.</p>
<h3 id="3-3-configmap-yaml-읽기">3-3. ConfigMap YAML 읽기</h3>
<pre><code class="language-yaml">apiVersion: v1
kind: ConfigMap
metadata:
  name: app-settings
  namespace: demo
data:
  APP_TITLE: &quot;작은 서점&quot;
  LOG_LEVEL: &quot;INFO&quot;
  app.properties: |
    page.size=20
    feature.preview=false</code></pre>
<p><code>APP_TITLE</code>, <code>LOG_LEVEL</code>, <code>app.properties</code>는 모두 키입니다. 여러 줄 문자열도 하나의 값으로 저장할 수 있습니다. <code>|</code>는 아래 여러 줄을 줄바꿈을 유지한 문자열로 읽는 YAML 표기입니다.</p>
<p>ConfigMap의 <code>data</code> 값은 문자열입니다. 숫자나 불리언처럼 보이는 설정도 문자열로 전달한다면 따옴표를 붙이는 것이 읽기 쉽습니다.</p>
<h3 id="3-4-방법-a-환경-변수로-특정-키-전달하기">3-4. 방법 A: 환경 변수로 특정 키 전달하기</h3>
<p>아래는 컨테이너 설정의 일부입니다.</p>
<pre><code class="language-yaml">env:
  - name: APP_TITLE
    valueFrom:
      configMapKeyRef:
        name: app-settings
        key: APP_TITLE</code></pre>
<pre><code class="language-text">ConfigMap의 APP_TITLE 값
          ↓
컨테이너의 APP_TITLE 환경 변수
          ↓
앱이 환경 변수를 읽음</code></pre>
<p><strong>앱이 이 환경 변수 이름을 읽도록 구현되어 있어야 합니다.</strong> ConfigMap에 키를 만들었다고 앱의 모든 기능이 자동으로 바뀌지는 않습니다.</p>
<h3 id="3-5-방법-b-여러-키를-한-번에-전달하기">3-5. 방법 B: 여러 키를 한 번에 전달하기</h3>
<pre><code class="language-yaml">envFrom:
  - configMapRef:
      name: app-settings</code></pre>
<p>설정 묶음을 환경 변수로 전달할 수 있습니다. 실제 환경 변수 이름으로 사용할 키와 충돌·호환성을 고려합니다. 파일 내용까지 들어 있는 ConfigMap 전체를 무조건 <code>envFrom</code>으로 주기보다 필요한 키만 선택하는 편이 목적을 분명하게 만들기도 합니다.</p>
<h3 id="3-6-방법-c-파일로-연결하기">3-6. 방법 C: 파일로 연결하기</h3>
<p>앱이 설정 파일을 읽는다면 ConfigMap을 볼륨으로 연결할 수 있습니다.</p>
<pre><code class="language-yaml">volumes:
  - name: app-config
    configMap:
      name: app-settings
      items:
        - key: app.properties
          path: app.properties</code></pre>
<pre><code class="language-yaml">volumeMounts:
  - name: app-config
    mountPath: /etc/app-config
    readOnly: true</code></pre>
<p>컨테이너에서는 <code>/etc/app-config/app.properties</code>로 파일을 읽습니다. <code>items</code>로 어떤 키를 어떤 파일 이름에 연결할지 고를 수 있습니다.</p>
<h3 id="3-7-전체-보충-예제-환경-변수와-파일을-함께-확인하기">3-7. 전체 보충 예제: 환경 변수와 파일을 함께 확인하기</h3>
<p>앞의 ConfigMap과 다음 Pod를 같은 <code>config-demo.yaml</code>에 <code>---</code>로 구분해 저장하거나 별도 파일로 적용합니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: config-reader
  namespace: demo
spec:
  containers:
    - name: reader
      image: busybox:1.37
      command: [&quot;sleep&quot;]
      args: [&quot;3600&quot;]
      env:
        - name: APP_TITLE
          valueFrom:
            configMapKeyRef:
              name: app-settings
              key: APP_TITLE
      volumeMounts:
        - name: config
          mountPath: /etc/app-config
          readOnly: true
  volumes:
    - name: config
      configMap:
        name: app-settings
        items:
          - key: app.properties
            path: app.properties</code></pre>
<pre><code class="language-bash">kubectl apply -f config-demo.yaml
kubectl exec config-reader -n demo -- printenv APP_TITLE
kubectl exec config-reader -n demo -- cat /etc/app-config/app.properties</code></pre>
<p>여기서는 공개 가능한 일반 설정만 출력합니다. Secret 값을 확인하는 습관과 혼동하지 마세요.</p>
<h3 id="3-8-configmap을-고쳤는데-앱-설정이-그대로라면">3-8. ConfigMap을 고쳤는데 앱 설정이 그대로라면?</h3>
<p>설정 전달 방식에 따라 반영이 다릅니다.</p>
<table>
<thead>
<tr>
<th>사용 방식</th>
<th>변경 후 생각할 일</th>
</tr>
</thead>
<tbody><tr>
<td>환경 변수</td>
<td>실행 중 프로세스의 값은 자동 갱신되지 않음. 새 컨테이너·Pod로 반영</td>
</tr>
<tr>
<td>디렉터리 볼륨 마운트</td>
<td>파일이 나중에 갱신될 수 있음. 앱이 다시 읽는지도 확인</td>
</tr>
<tr>
<td><code>subPath</code>로 파일 하나 마운트</td>
<td>ConfigMap 변경의 자동 파일 갱신 대상이 아님</td>
</tr>
</tbody></table>
<p>게시판의 안내문이 바뀌어도 이미 메모한 사람이 다시 읽지 않으면 행동이 바뀌지 않는 것과 같습니다. 파일 내용의 변경과 앱의 재로딩도 구분해야 합니다. <a href="https://kubernetes.io/docs/concepts/configuration/configmap/">공식 문서: ConfigMaps</a></p>
<p>Deployment가 ConfigMap을 사용하는 경우, 설정 변경 후 Pod를 다시 만들기 위한 방법 중 하나는 다음 명령입니다.</p>
<pre><code class="language-bash">kubectl rollout restart deployment/web-app -n demo
kubectl rollout status deployment/web-app -n demo</code></pre>
<p>이 명령은 해당 Deployment가 존재할 때 사용하는 형식 예제입니다. ConfigMap 변경 자체가 모든 Deployment에 자동 rollout을 발생시키는 것은 아닙니다.</p>
<h3 id="3-9-마운트-경로와-기존-파일에-주의하기">3-9. 마운트 경로와 기존 파일에 주의하기</h3>
<p>ConfigMap을 특정 디렉터리에 마운트하면 그 위치의 기존 이미지 파일이 가려질 수 있습니다. 예를 들어 설정 파일 하나만 추가하려다가 디렉터리 전체를 덮어 보이게 할 수 있습니다.</p>
<p><code>subPath</code>는 필요한 파일 하나를 연결하는 데 쓸 수 있지만 앞서 설명한 갱신 제약이 있습니다. 디렉터리 전체 연결과 단일 파일 연결 중 무엇이 필요한지 먼저 정합니다.</p>
<h3 id="3-10-크기·불변-설정·정리">3-10. 크기·불변 설정·정리</h3>
<p>ConfigMap은 큰 파일 보관소가 아닙니다. 저장하는 데이터는 1MiB 제한을 고려해야 하며 대용량 자료는 별도 저장소를 사용합니다. 일반 문자열은 <code>data</code>, 바이너리 데이터는 <code>binaryData</code>로 표현할 수 있습니다. 변경을 막고 싶을 때는 <code>immutable: true</code>를 사용할 수 있지만, 이후 수정 방식도 달라집니다.</p>
<pre><code class="language-bash">kubectl get configmap app-settings -n demo
kubectl describe pod config-reader -n demo</code></pre>
<p>키가 없거나 ConfigMap 이름·네임스페이스가 다르면 컨테이너 시작이 실패할 수 있습니다. 이름과 키, Events부터 확인합니다.</p>
<p><strong>이 파트의 핵심:</strong> 이미지에 앱을 담고, ConfigMap에는 일반 설정을 담습니다. 변경이 언제 앱에 반영되는지는 전달 방식과 앱 동작에 달려 있습니다.</p>
<hr />
<h2 id="part-4-secret-민감한-설정을-앱에-전달하기">PART 4. Secret: 민감한 설정을 앱에 전달하기</h2>
<h3 id="4-1-비밀번호를-이미지에-넣으면-무엇이-문제일까">4-1. 비밀번호를 이미지에 넣으면 무엇이 문제일까?</h3>
<p>앱 코드나 Dockerfile에 실제 비밀번호를 적으면 이미지·소스·빌드 로그를 통해 노출될 수 있습니다. 값을 바꿀 때 이미지도 다시 만들어야 하고, 이전 기록에 남은 정보도 처리해야 합니다.</p>
<pre><code class="language-text">앱 이미지: 프로그램
Secret: 실행에 필요한 민감한 값</code></pre>
<p>Kubernetes Secret은 비밀번호, 토큰, 인증서 개인키 같은 민감한 데이터를 별도 리소스로 다루는 기능입니다. 다만 이름이 Secret이라고 모든 저장·전달·출력 경로가 자동으로 안전해지는 것은 아닙니다.</p>
<h3 id="4-2-configmap과-secret을-비교하자">4-2. ConfigMap과 Secret을 비교하자</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>ConfigMap</th>
<th>Secret</th>
</tr>
</thead>
<tbody><tr>
<td>주 목적</td>
<td>일반 설정</td>
<td>민감한 설정·인증 자료</td>
</tr>
<tr>
<td>앱 전달</td>
<td>환경 변수·파일 등</td>
<td>환경 변수·파일 등</td>
</tr>
<tr>
<td>공개 문서</td>
<td>공개 가능한 값만</td>
<td>실제 값과 개인키를 포함하지 않음</td>
</tr>
<tr>
<td>보안 관점</td>
<td>내부 정보가 있을 수 있음</td>
<td>조회 권한·저장 암호화·앱의 출력 관리까지 필요</td>
</tr>
</tbody></table>
<p>앱은 값을 읽어야 일을 할 수 있으므로 적절한 접근 권한은 필요합니다. 목표는 누구도 읽지 못하게 만드는 것이 아니라 <strong>필요한 대상만 필요한 범위에서 읽게 하는 것</strong>입니다.</p>
<h3 id="4-3-base64는-암호화가-아니다">4-3. Base64는 암호화가 아니다</h3>
<p>Base64는 데이터를 정해진 문자 형식으로 바꾸는 <strong>인코딩</strong>입니다. 다시 돌리는 데 비밀 열쇠가 필요하지 않습니다.</p>
<pre><code class="language-bash">printf 'hello' | base64</code></pre>
<p>결과는 <code>aGVsbG8=</code>입니다. 이를 되돌리면 다시 <code>hello</code>가 됩니다.</p>
<pre><code class="language-bash">python3 -c 'import base64; print(base64.b64decode(&quot;aGVsbG8=&quot;).decode())'</code></pre>
<p>글씨를 다른 표기법으로 적었다고 읽을 권한이 자동으로 제한되는 것은 아닙니다. Base64 문자열도 실제 비밀값과 같은 수준으로 다뤄야 합니다.</p>
<h3 id="4-4-data와-stringdata">4-4. <code>data</code>와 <code>stringData</code></h3>
<table>
<thead>
<tr>
<th>필드</th>
<th>작성 방법</th>
</tr>
</thead>
<tbody><tr>
<td><code>data</code></td>
<td>Base64로 인코딩한 값을 지정</td>
</tr>
<tr>
<td><code>stringData</code></td>
<td>문자열을 적으면 API가 저장 형식으로 변환</td>
</tr>
</tbody></table>
<p>다음 값은 <strong>사용 불가능한 공개 학습용 가짜 값</strong>입니다. 실제 계정 정보가 아닙니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Secret
metadata:
  name: demo-credentials
  namespace: demo
type: Opaque
stringData:
  username: &quot;learning-user&quot;
  password: &quot;learning-only-not-a-real-password&quot;</code></pre>
<p><code>stringData</code>는 입력 편의를 제공할 뿐, 이 YAML 파일 자체의 비밀을 보호해 주지는 않습니다. 실제 비밀을 넣은 파일을 공개 저장소에 올리면 안 됩니다.</p>
<h3 id="4-5-secret의-대표-타입">4-5. Secret의 대표 타입</h3>
<table>
<thead>
<tr>
<th>타입</th>
<th>목적·대표 키</th>
</tr>
</thead>
<tbody><tr>
<td><code>Opaque</code></td>
<td>일반적인 임의 키·값</td>
</tr>
<tr>
<td><code>kubernetes.io/dockerconfigjson</code></td>
<td>레지스트리 인증, <code>.dockerconfigjson</code></td>
</tr>
<tr>
<td><code>kubernetes.io/basic-auth</code></td>
<td>사용자명·비밀번호 인증 자료</td>
</tr>
<tr>
<td><code>kubernetes.io/ssh-auth</code></td>
<td>SSH 개인키, <code>ssh-privatekey</code></td>
</tr>
<tr>
<td><code>kubernetes.io/tls</code></td>
<td>TLS 인증서·개인키, <code>tls.crt</code>·<code>tls.key</code></td>
</tr>
<tr>
<td><code>kubernetes.io/service-account-token</code></td>
<td>ServiceAccount와 연결한 장기 토큰 자료</td>
</tr>
</tbody></table>
<p>타입별로 API가 검사하는 키와 구조가 다릅니다. Opaque와 다른 타입을 모두 “키 이름이 완전히 자유로운 Secret”이라고 생각하지 않습니다. Secret 타입을 지정했다고 해당 앱이 자동으로 인증을 수행하는 것도 아닙니다.</p>
<p>ServiceAccount 토큰 Secret을 수동으로 만드는 방식과, Pod에 짧은 수명의 토큰을 투영하는 현대적인 방식을 구분합니다. ServiceAccount를 만들면 장기 토큰 Secret이 항상 자동 생성된다고 생각하지 않습니다.</p>
<h3 id="4-6-방법-a-secret을-환경-변수로-전달하기">4-6. 방법 A: Secret을 환경 변수로 전달하기</h3>
<p>컨테이너 설정의 일부입니다.</p>
<pre><code class="language-yaml">env:
  - name: APP_PASSWORD
    valueFrom:
      secretKeyRef:
        name: demo-credentials
        key: password</code></pre>
<p>ConfigMap의 <code>configMapKeyRef</code> 대신 <code>secretKeyRef</code>를 사용합니다. 앱이 <code>APP_PASSWORD</code>를 읽도록 구현되어 있어야 합니다.</p>
<p>환경 변수로 전달된 값은 실행 중인 프로세스에서 Secret을 고쳤다고 자동 교체되지 않습니다. 반영을 위해 Pod 교체와 앱 연결 재설정 등이 필요할 수 있습니다.</p>
<h3 id="4-7-방법-b-secret을-파일로-전달하기">4-7. 방법 B: Secret을 파일로 전달하기</h3>
<pre><code class="language-yaml">volumes:
  - name: credentials
    secret:
      secretName: demo-credentials
      items:
        - key: password
          path: password</code></pre>
<pre><code class="language-yaml">volumeMounts:
  - name: credentials
    mountPath: /run/app-credentials
    readOnly: true</code></pre>
<p>앱은 <code>/run/app-credentials/password</code> 파일을 읽을 수 있습니다. 파일을 읽는 사용자 권한과 앱의 재로딩 방식도 고려합니다.</p>
<p>일반적인 Secret 볼륨 갱신에도 지연이 있을 수 있고, <code>subPath</code>를 사용한 마운트는 자동 갱신되지 않습니다. 파일이 바뀌어도 앱이 다시 읽지 않으면 새 값은 적용되지 않을 수 있습니다.</p>
<h3 id="4-8-값-대신-존재-여부를-확인하는-실습">4-8. 값 대신 존재 여부를 확인하는 실습</h3>
<p>앞의 가짜 Secret과 다음 Pod를 <code>secret-demo.yaml</code>에 <code>---</code>로 구분해 저장합니다.</p>
<pre><code class="language-yaml">apiVersion: v1
kind: Pod
metadata:
  name: secret-reader
  namespace: demo
spec:
  containers:
    - name: reader
      image: busybox:1.37
      command: [&quot;sleep&quot;]
      args: [&quot;3600&quot;]
      volumeMounts:
        - name: credentials
          mountPath: /run/app-credentials
          readOnly: true
  volumes:
    - name: credentials
      secret:
        secretName: demo-credentials
        items:
          - key: password
            path: password</code></pre>
<pre><code class="language-bash">kubectl apply -f secret-demo.yaml
kubectl get secret demo-credentials -n demo
kubectl exec secret-reader -n demo -- sh -c \
  'test -s /run/app-credentials/password &amp;&amp; echo &quot;credential file is present&quot;'</code></pre>
<p>이 방법은 값을 화면에 출력하지 않고 비어 있지 않은 파일이 연결됐는지 확인합니다. 실제 값으로 실습할 때 전체 <code>env</code>나 Secret YAML을 출력한 화면을 공유하지 않습니다.</p>
<h3 id="4-9-레지스트리-로그인과-imagepullsecrets는-다르다">4-9. 레지스트리 로그인과 imagePullSecrets는 다르다</h3>
<pre><code class="language-text">개발자 노트북의 docker login
    → 노트북의 push·pull 인증

Pod의 imagePullSecrets
    → 노드가 프라이빗 이미지를 확보할 때 사용할 인증 설정</code></pre>
<p>노트북에서 로그인했다고 모든 Kubernetes 노드에 인증이 자동 전달되지는 않습니다. 환경에 따라 노드의 별도 인증 기능을 사용할 수도 있습니다.</p>
<p>아래는 Pod 설정의 일부입니다. <code>registry-auth</code> Secret이 같은 네임스페이스에 준비되어 있다는 가정입니다.</p>
<pre><code class="language-yaml">imagePullSecrets:
  - name: registry-auth</code></pre>
<p>필요하면 ServiceAccount에 기본 이미지 다운로드 인증을 연결할 수도 있습니다. 실제 Docker 인증 파일을 Secret으로 옮길 때는 다른 레지스트리의 자격증명까지 과하게 포함하지 않도록 범위를 확인합니다.</p>
<h3 id="4-10-secret의-보호-범위와-한계">4-10. Secret의 보호 범위와 한계</h3>
<p>Secret을 안전하게 쓰려면 다음 관계를 이해해야 합니다.</p>
<pre><code class="language-text">누가 Secret을 읽을 수 있는가? → RBAC와 접근 통제
저장된 상태는 어떻게 보호하는가? → 저장 시 암호화 설정
앱은 값을 어디에 남기는가? → 로그·오류 메시지·디버그 출력
값이 바뀌면 어떻게 적용하는가? → 교체·재로딩·자격증명 회전 절차</code></pre>
<p>기본 Secret 리소스 사용만으로 저장소에서의 암호화가 자동 보장되는 것은 아닙니다. 관리형 환경도 실제 설정을 확인합니다. 또한 Pod 생성 권한이 민감 데이터 접근에 어떤 영향을 주는지도 함께 검토합니다. <a href="https://kubernetes.io/docs/concepts/configuration/secret/">공식 문서: Secrets</a></p>
<p><strong>이 파트의 핵심:</strong> Secret은 민감한 값의 관리·전달을 위한 기능입니다. Base64는 암호화가 아니며 권한·저장·로그·변경 반영까지 함께 관리합니다.</p>
<hr />
<h2 id="part-5-http-ingress와-https-외부-요청을-알맞은-앱으로-보내기">PART 5. HTTP Ingress와 HTTPS: 외부 요청을 알맞은 앱으로 보내기</h2>
<h3 id="5-1-service가-있는데-ingress는-왜-필요할까">5-1. Service가 있는데 Ingress는 왜 필요할까?</h3>
<p>하나의 사이트 안에 상품, 주문, 회원 서비스가 있다고 생각해 보세요.</p>
<pre><code class="language-text">shop.example.com/products → 상품 서비스
shop.example.com/orders   → 주문 서비스
account.example.com      → 회원 서비스</code></pre>
<p>사용자는 각 Pod의 IP를 외우지 않습니다. 도메인과 URL 경로로 접속합니다. <strong>Ingress는 HTTP·HTTPS 요청을 어떤 Service로 연결할지 선언하는 규칙</strong>입니다.</p>
<h3 id="5-2-ingress와-ingress-controller는-다르다">5-2. Ingress와 Ingress Controller는 다르다</h3>
<table>
<thead>
<tr>
<th>구성 요소</th>
<th>역할</th>
<th>건물 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Ingress 리소스</td>
<td>도메인·경로별 연결 규칙</td>
<td>안내판의 내용</td>
</tr>
<tr>
<td>Ingress Controller</td>
<td>규칙을 해석해 실제 전달을 구성</td>
<td>안내 체계를 운영하는 담당자</td>
</tr>
<tr>
<td>실제 프록시·로드밸런서</td>
<td>HTTP 요청을 처리</td>
<td>실제 통로·접수 창구</td>
</tr>
</tbody></table>
<p>Ingress YAML만 만들면 외부 접속이 자동 완성되는 것은 아닙니다. 규칙을 처리할 Controller와 외부 연결 경로가 필요합니다.</p>
<p>일부 구현은 Controller와 프록시가 가까이 묶여 동작하고, 다른 구현은 Controller가 클라우드 로드밸런서를 구성합니다. 역할과 실행 위치를 구분합니다.</p>
<h3 id="5-3-l4와-l7을-쉽게-이해하기">5-3. L4와 L7을 쉽게 이해하기</h3>
<table>
<thead>
<tr>
<th>관점</th>
<th>주로 보는 정보</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>L4 전달</td>
<td>IP·포트 등</td>
<td>건물 주소와 출입구 번호</td>
</tr>
<tr>
<td>L7 HTTP 라우팅</td>
<td>Host·URL 경로 등</td>
<td>접수 내용에 따른 부서 배정</td>
</tr>
</tbody></table>
<p>Ingress에서는 “<code>/orders</code> 요청은 주문 서비스로”처럼 HTTP 내용을 보고 규칙을 정할 수 있습니다. Kubernetes의 일반 Ingress 규칙이 모든 TCP·UDP 프로토콜을 동일하게 처리하는 것은 아닙니다.</p>
<h3 id="5-4-외부-요청의-전체-흐름">5-4. 외부 요청의 전체 흐름</h3>
<pre><code class="language-text">① 브라우저가 도메인의 주소를 DNS에서 확인
② 외부에서 접근 가능한 로드밸런서·노드 등의 진입점으로 연결
③ HTTP Host와 경로에 맞는 Ingress 규칙 적용
④ 규칙이 가리키는 Service의 연결 대상 선택
⑤ 대상 Pod의 앱에 요청 전달</code></pre>
<p>이것은 논리 흐름입니다. Controller가 EndpointSlice 등을 보고 Pod IP를 직접 백엔드로 구성하면, 실제 패킷이 Service의 ClusterIP를 항상 경유하지 않을 수 있습니다.</p>
<p>외부 로드밸런서 → NodePort → Ingress 프록시처럼 여러 층을 거칠 수도 있고, 다른 구조를 쓸 수도 있습니다. DNS와 외부 연결 설정도 Ingress YAML 한 장이 전부 대신하지 않습니다.</p>
<h3 id="5-5-host-기반과-path-기반-라우팅">5-5. Host 기반과 Path 기반 라우팅</h3>
<p><strong>Host 기반</strong>은 요청한 사이트 이름으로 나눕니다.</p>
<pre><code class="language-text">shop.example.com → 쇼핑 앱
blog.example.com → 블로그 앱</code></pre>
<p><strong>Path 기반</strong>은 같은 사이트 안에서 경로로 나눕니다.</p>
<pre><code class="language-text">shop.example.com/api → API
shop.example.com/    → 화면 제공 앱</code></pre>
<p>여기서 Host는 요청의 HTTP Host 정보와 연결됩니다. IP로만 요청하면 도메인을 기준으로 작성한 규칙과 맞지 않을 수 있습니다.</p>
<h3 id="5-6-ingressclass는-어느-담당자가-처리할지-고르는-값">5-6. IngressClass는 어느 담당자가 처리할지 고르는 값</h3>
<pre><code class="language-bash">kubectl get ingressclass</code></pre>
<p>Ingress의 <code>spec.ingressClassName</code>에는 사용할 IngressClass의 이름을 지정합니다. 아무 이름이나 적으면 Controller가 처리해 주지 않을 수 있습니다.</p>
<p>아래 <code>example-controller</code>는 설명용입니다. 실제로 안내받은 클래스 이름으로 바꿉니다. 클래스가 하나라는 이유만으로 필드를 무조건 생략해도 된다고 생각하지 않습니다.</p>
<h3 id="5-7-http-ingress-전체-예제">5-7. HTTP Ingress 전체 예제</h3>
<p>아래 예제는 Nginx 정적 웹서버를 실행하고 이를 Service와 Ingress로 연결합니다. 이미지 버전은 예제의 동작 대상을 고정해 읽기 위한 값이며 운영에는 지원·보안 업데이트를 확인합니다.</p>
<p><code>http-demo.yaml</code>:</p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: public-web
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: public-web
  template:
    metadata:
      labels:
        app: public-web
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
---
apiVersion: v1
kind: Service
metadata:
  name: public-web
  namespace: demo
spec:
  selector:
    app: public-web
  ports:
    - name: http
      port: 80
      targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: public-web
  namespace: demo
spec:
  ingressClassName: example-controller
  rules:
    - host: web.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: public-web
                port:
                  number: 80</code></pre>
<p><code>web.example.com</code>은 문서용 예시입니다. 실제 DNS와 라우팅을 연결해야 하며, 이 예제만 복사해서 그 도메인으로 접속되는 것은 아닙니다.</p>
<h3 id="5-8-포트-세-가지를-헷갈리지-말자">5-8. 포트 세 가지를 헷갈리지 말자</h3>
<pre><code class="language-text">브라우저가 연결하는 외부 포트
       ↓ Ingress 전달
Service의 port
       ↓ targetPort
Pod에서 앱이 듣는 포트</code></pre>
<p>예를 들어 앱은 8080에서 동작하지만 Service는 80으로 제공할 수 있습니다. 이때 <code>port: 80</code>, <code>targetPort: 8080</code>을 사용합니다.</p>
<p>Ingress의 <code>backend.service.port.number</code>는 Service의 포트를 참조합니다. 외부 포트나 Pod 포트를 무조건 그대로 적는 필드는 아닙니다.</p>
<h3 id="5-9-exact-prefix-implementationspecific">5-9. <code>Exact</code>, <code>Prefix</code>, <code>ImplementationSpecific</code></h3>
<table>
<thead>
<tr>
<th>pathType</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td>Exact</td>
<td>경로가 정확히 일치하는 규칙</td>
</tr>
<tr>
<td>Prefix</td>
<td>경로 요소를 기준으로 접두 경로 매칭</td>
</tr>
<tr>
<td>ImplementationSpecific</td>
<td>구체적인 해석을 Controller 구현에 맡김</td>
</tr>
</tbody></table>
<p>Prefix의 <code>/api</code>는 <code>/api/users</code>에 맞을 수 있지만 <code>/apix</code>와 같은 문자열을 무조건 같은 것으로 취급하는 단순 문자 접두 비교는 아닙니다.</p>
<p>또한 <code>/api</code>로 라우팅한다고 백엔드에서 <code>/api</code>가 자동으로 제거되지는 않습니다. 경로 재작성은 Controller 기능과 별도 설정입니다. <a href="https://kubernetes.io/docs/concepts/services-networking/ingress/">공식 문서: Ingress</a></p>
<h3 id="5-10-적용-후-확인하는-순서">5-10. 적용 후 확인하는 순서</h3>
<p>클래스와 도메인 등 전제 조건을 맞춘 뒤 적용합니다.</p>
<pre><code class="language-bash">kubectl apply -f http-demo.yaml
kubectl rollout status deployment/public-web -n demo
kubectl get service,ingress -n demo
kubectl get endpointslice -n demo \
  -l kubernetes.io/service-name=public-web
kubectl describe ingress public-web -n demo</code></pre>
<p>DNS가 아직 연결되지 않았을 때 HTTP Host를 지정해 테스트하는 형식 예시는 다음과 같습니다.</p>
<pre><code class="language-bash">curl -i -H 'Host: web.example.com' http://192.0.2.20/</code></pre>
<p><code>192.0.2.20</code> 역시 문서용 주소입니다. 실제 Ingress 진입점으로 바꿔야 합니다. 진입점이 DNS 이름으로 제공되는 환경도 있습니다.</p>
<p>Ingress의 ADDRESS가 어떻게 표시되는지는 구현에 따라 다릅니다. 주소가 보인다고 앱 정상 응답이 자동 보장되는 것도 아닙니다.</p>
<h3 id="5-11-404·502·503을-원인-이름으로-외우지-말자">5-11. 404·502·503을 원인 이름으로 외우지 말자</h3>
<table>
<thead>
<tr>
<th>증상</th>
<th>확인할 방향</th>
</tr>
</thead>
<tbody><tr>
<td>도메인 조회 실패</td>
<td>DNS 연결</td>
</tr>
<tr>
<td>연결 자체가 안 됨</td>
<td>외부 진입점·포트·방화벽·Controller</td>
</tr>
<tr>
<td>404</td>
<td>Host·Path 불일치 또는 앱 자체의 경로 오류</td>
</tr>
<tr>
<td>502·503</td>
<td>연결 대상·준비 상태·포트·앱 오류 등</td>
</tr>
<tr>
<td>인증서 경고</td>
<td>인증서 이름·만료·신뢰 체인·적용 대상</td>
</tr>
</tbody></table>
<p>같은 상태 코드도 Controller와 앱에 따라 원인이 다릅니다. “503이면 무조건 Label 문제”처럼 한 가지로 단정하지 않습니다.</p>
<h3 id="5-12-http와-https의-차이">5-12. HTTP와 HTTPS의 차이</h3>
<p>HTTP는 요청·응답을 주고받는 규칙이고, HTTPS는 TLS로 보호된 연결 위에서 HTTP를 주고받는 방식입니다.</p>
<pre><code class="language-text">HTTP: 내용을 전달하는 대화 규칙
TLS: 상대 확인과 전송 중 데이터 보호를 제공하는 연결
HTTPS: 보호된 연결 안에서 HTTP 사용</code></pre>
<p>봉투에 넣지 않은 편지와, 상대를 확인하고 내용을 보호해 전달하는 편지를 떠올릴 수 있습니다. HTTPS가 앱의 모든 취약점과 저장 데이터 유출을 해결하는 것은 아닙니다.</p>
<h3 id="5-13-공개키·개인키·인증서·ca">5-13. 공개키·개인키·인증서·CA</h3>
<table>
<thead>
<tr>
<th>용어</th>
<th>쉬운 설명</th>
</tr>
</thead>
<tbody><tr>
<td>공개키</td>
<td>공개할 수 있는 키 정보</td>
</tr>
<tr>
<td>개인키</td>
<td>소유자가 보호해야 하는 비밀 키</td>
</tr>
<tr>
<td>인증서</td>
<td>도메인 등의 정보와 공개키를 연결하고 서명을 포함한 자료</td>
</tr>
<tr>
<td>CA</td>
<td>정해진 절차로 인증서를 발급·서명하는 인증기관</td>
</tr>
<tr>
<td>대칭키</td>
<td>통신 데이터를 효율적으로 보호할 때 사용하는 키</td>
</tr>
</tbody></table>
<p>공개키를 무조건 “모든 HTTPS 내용을 암호화하는 열쇠”라고만 이해하지 않습니다. 현대 TLS는 인증과 키 교환·합의를 수행하고, 실제 데이터 보호에는 협의한 대칭 암호 등을 사용합니다.</p>
<p><code>tls.crt</code>는 인증서, <code>tls.key</code>는 개인키입니다. 인증서에 개인키가 포함된다고 생각하지 않습니다. 개인키는 공개하지 않습니다.</p>
<h3 id="5-14-tls-종료는-어디서-일어날까">5-14. TLS 종료는 어디서 일어날까?</h3>
<p>일반적인 Ingress TLS 종료 구성을 생각하면 다음과 같습니다.</p>
<pre><code class="language-text">브라우저 ── HTTPS ── Ingress 프록시 ── HTTP 또는 별도 HTTPS ── 앱</code></pre>
<p>앞 구간에 HTTPS가 적용됐다고 뒤의 모든 구간까지 자동 암호화된 것은 아닙니다. 백엔드 연결도 TLS로 보호할지는 별도로 구성합니다.</p>
<h3 id="5-15-tls-secret을-참조하는-ingress">5-15. TLS Secret을 참조하는 Ingress</h3>
<p>아래는 Ingress 설정의 일부입니다. 같은 네임스페이스의 <code>web-example-tls</code> Secret에 올바른 인증서와 개인키가 준비되어 있어야 합니다.</p>
<pre><code class="language-yaml">spec:
  tls:
    - hosts:
        - web.example.com
      secretName: web-example-tls</code></pre>
<p>인증서가 유효한 호스트 이름과 Ingress의 접속 호스트가 맞아야 합니다. 와일드카드 인증서 <code>*.example.com</code>은 일반적으로 <code>shop.example.com</code> 같은 한 단계 하위 이름을 대상으로 하며, <code>example.com</code> 자체나 <code>a.b.example.com</code>까지 모두 포함한다고 가정하지 않습니다.</p>
<h3 id="5-16-cert-manager-인증서-준비와-갱신-담당자">5-16. cert-manager: 인증서 준비와 갱신 담당자</h3>
<p>cert-manager는 인증서 발급과 갱신을 Kubernetes 리소스로 관리하는 도구입니다. Kubernetes에 기본으로 항상 설치되어 있는 기능은 아닙니다.</p>
<table>
<thead>
<tr>
<th>리소스</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Issuer</td>
<td>같은 네임스페이스에서 사용할 발급자 설정</td>
</tr>
<tr>
<td>ClusterIssuer</td>
<td>클러스터 범위의 발급자 설정</td>
</tr>
<tr>
<td>Certificate</td>
<td>어떤 이름의 인증서를 어느 Secret에 준비할지 요청</td>
</tr>
<tr>
<td>Secret</td>
<td>발급된 인증서·개인키를 앱이나 Ingress가 사용할 형태로 보관</td>
</tr>
</tbody></table>
<pre><code class="language-text">발급자 설정 → Certificate 요청 → 도메인 제어권 확인·발급 → TLS Secret
                                                               ↓
                                                           Ingress 사용</code></pre>
<p>설치는 Helm 등으로 수행할 수 있습니다. 교재의 특정 옛 버전 번호를 현재 환경에 그대로 설치하기보다는 Kubernetes 호환성과 지원 중인 cert-manager 버전을 확인합니다. 이미 구성된 공유 클러스터에서는 관리자 제공 설정을 사용합니다.</p>
<h3 id="5-17-acme와-challenge-이-도메인을-제어하는가">5-17. ACME와 Challenge: 이 도메인을 제어하는가?</h3>
<p>ACME는 인증서 발급·갱신 등을 자동화하는 프로토콜입니다. Let's Encrypt 같은 CA와 연동할 때 사용할 수 있습니다.</p>
<p>CA는 단순히 “이 도메인이 제 것입니다”라는 말만 믿지 않습니다. 요청자가 도메인을 제어하는지 확인하기 위한 과제를 제시합니다. 이것이 Challenge입니다.</p>
<table>
<thead>
<tr>
<th>방법</th>
<th>무엇을 확인할까?</th>
<th>생각할 조건</th>
</tr>
</thead>
<tbody><tr>
<td>HTTP-01</td>
<td>도메인의 정해진 HTTP 경로에 검증 응답 제공</td>
<td>외부에서 HTTP 80번 포트 접근, 올바른 라우팅</td>
</tr>
<tr>
<td>DNS-01</td>
<td>정해진 DNS TXT 레코드에 검증 값 제공</td>
<td>DNS 레코드를 수정할 권한과 반영</td>
</tr>
</tbody></table>
<p>도메인의 HTTP 문 앞에 표지판을 놓는 방식과 DNS 주소록에 확인 문구를 올리는 방식으로 비유할 수 있습니다. 와일드카드 인증서 검증에는 DNS-01을 사용합니다. <a href="https://cert-manager.io/docs/configuration/acme/">공식 문서: ACME</a>, <a href="https://cert-manager.io/docs/configuration/acme/dns01/">DNS-01</a></p>
<h3 id="5-18-acme-계정-키와-서버-개인키는-다르다">5-18. ACME 계정 키와 서버 개인키는 다르다</h3>
<p>Issuer 설정의 <code>privateKeySecretRef</code>는 ACME 계정의 키를 보관하는 Secret 참조입니다. 사이트의 TLS 연결에 사용할 서버 개인키와 같은 목적의 키가 아닙니다.</p>
<pre><code class="language-text">ACME 계정 키: 인증기관과 대화하는 발급 계정의 신원
서버 TLS 개인키: 해당 사이트의 인증서·TLS 연결에 사용하는 키</code></pre>
<p>Solver는 어떤 방식으로 도메인 제어권 확인 과제를 해결할지 정하는 설정입니다. DNS-01에서 DNS 제공자 API를 사용한다면 그 권한도 필요한 범위로 제한해야 합니다.</p>
<h3 id="5-19-certificate-리소스-예제">5-19. Certificate 리소스 예제</h3>
<p>다음은 cert-manager와 사용할 발급자가 설치·설정되어 있다는 가정의 구조 예제입니다. <code>example.com</code>으로 실제 발급을 시도하는 예제가 아니며, 통제하는 실제 도메인으로 바꿔야 합니다.</p>
<pre><code class="language-yaml">apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: web-example-certificate
  namespace: demo
spec:
  secretName: web-example-tls
  dnsNames:
    - web.example.com
  issuerRef:
    name: example-issuer
    kind: ClusterIssuer</code></pre>
<p>cert-manager는 발급 절차를 수행해 지정한 Secret에 결과를 저장합니다. CA에 서버 개인키를 전달해 보관시키는 것이 아니라, 공개키와 요청 정보를 담은 CSR 등으로 발급 절차를 진행합니다. <a href="https://cert-manager.io/docs/usage/certificate/">공식 문서: Certificate</a></p>
<pre><code class="language-bash">kubectl get certificate -n demo
kubectl describe certificate web-example-certificate -n demo
kubectl get secret web-example-tls -n demo</code></pre>
<p>실패하면 필요한 경우 CertificateRequest·Order·Challenge의 상태와 Events도 확인합니다. 인증서 준비 여부를 보려고 TLS Secret의 개인키를 통째로 출력할 필요는 없습니다.</p>
<h3 id="5-20-ingress에-주석을-달아-인증서-요청하기">5-20. Ingress에 주석을 달아 인증서 요청하기</h3>
<p>cert-manager의 ingress-shim 기능을 사용하는 환경에서는 Ingress의 annotation과 TLS 정보를 보고 Certificate를 준비하도록 할 수 있습니다.</p>
<pre><code class="language-yaml">metadata:
  annotations:
    cert-manager.io/cluster-issuer: example-issuer</code></pre>
<p>이때도 <code>spec.tls</code>의 hosts·secretName 등 필요한 설정이 있어야 합니다. 일반적인 Ingress Controller가 모든 인증서를 직접 요청·발급하는 기능이라고 이해하지 않습니다. cert-manager가 Ingress를 관찰해 인증서 관리를 수행하고, Controller·프록시는 준비된 TLS 자료를 사용합니다. <a href="https://cert-manager.io/docs/usage/ingress/">공식 문서: Annotated Ingress</a></p>
<p>인증서를 여러 리소스에서 재사용하거나 세부 정책을 명시적으로 관리하려면 Certificate를 별도로 정의하는 방법이 이해하기 쉬울 수 있습니다.</p>
<h3 id="5-21-교재의-ingress-nginx-예제를-읽을-때의-현재-기준">5-21. 교재의 ingress-nginx 예제를 읽을 때의 현재 기준</h3>
<p>Kubernetes 커뮤니티의 <strong>ingress-nginx 프로젝트는 2026년 3월 24일에 종료되었습니다.</strong> Ingress API 자체가 종료됐다는 뜻은 아닙니다. 이름이 비슷한 다른 NGINX 제품과도 구분해야 합니다. <a href="https://kubernetes.io/blog/2026/03/30/kubernetes-v1-36-sneak-peek/">Kubernetes 공식 발표</a></p>
<p>이 글은 교재의 라우팅과 Canary 개념을 생략하지 않고 설명하되, 해당 Controller 전용 annotation은 기존 예제를 이해하는 문법으로 구분합니다. 신규 운영 환경에는 유지보수 중인 구현과 지원 정책을 확인해야 합니다. <code>ingressClassName</code>을 다른 값으로 바꾼다고 전용 annotation까지 자동 호환되는 것은 아닙니다.</p>
<p><strong>이 파트의 핵심:</strong> Ingress는 접속 규칙이고 Controller가 이를 구현합니다. HTTPS에는 도메인·인증서·개인키·TLS 설정이 필요하며, cert-manager는 발급과 갱신을 관리합니다.</p>
<hr />
<h2 id="part-6-무중단-배포-새-버전을-서비스하면서-교체하기">PART 6. 무중단 배포: 새 버전을 서비스하면서 교체하기</h2>
<h3 id="6-1-배포할-때-왜-서비스가-끊길까">6-1. 배포할 때 왜 서비스가 끊길까?</h3>
<p>기존 앱을 모두 종료한 다음 새 앱을 시작하면 빈 시간이 생깁니다.</p>
<pre><code class="language-text">기존 앱 종료 → 아무도 요청을 처리하지 못함 → 새 앱 준비 완료</code></pre>
<p>식당의 조리대를 바꾼다고 모든 조리대를 동시에 치우면 주문을 받을 수 없습니다. 사용 가능한 조리대를 유지하면서 교체하는 설계가 필요합니다.</p>
<p>무중단 배포는 단순히 옵션 하나를 켜는 일이 아닙니다. <strong>요청을 처리할 수 있는 앱을 유지하고, 준비된 새 앱으로 연결을 옮기며, 기존 작업을 안전하게 마무리하는 과정</strong>입니다.</p>
<h3 id="6-2-세-가지-대표-전략">6-2. 세 가지 대표 전략</h3>
<table>
<thead>
<tr>
<th>전략</th>
<th>바꾸는 방법</th>
<th>장점</th>
<th>생각할 비용·조건</th>
</tr>
</thead>
<tbody><tr>
<td>Rolling Update</td>
<td>기존 Pod를 조금씩 새 Pod로 교체</td>
<td>일반적인 Deployment와 연결하기 쉬움</td>
<td>버전 공존·준비 상태·여유 용량</td>
</tr>
<tr>
<td>Blue/Green</td>
<td>두 환경을 준비하고 연결 대상을 전환</td>
<td>전환·복귀 지점이 분명함</td>
<td>두 환경의 자원, 데이터 호환성</td>
</tr>
<tr>
<td>Canary</td>
<td>일부 요청에 먼저 새 버전 제공</td>
<td>작은 범위로 검증 가능</td>
<td>트래픽 분배 기능, 관측·판단 기준</td>
</tr>
</tbody></table>
<p>어떤 방식도 앱 오류를 자동으로 없애지는 않습니다. 무엇을 보고 성공·실패를 판단할지 정해야 합니다.</p>
<h3 id="6-3-rolling-update-복습">6-3. Rolling Update 복습</h3>
<pre><code class="language-text">v1 v1 v1
   ↓ 새 Pod 준비와 기존 Pod 축소를 반복
v1 v1 v2
   ↓
v1 v2 v2
   ↓
v2 v2 v2</code></pre>
<p>위는 논리적인 그림입니다. 준비 중이거나 종료 중인 Pod 때문에 실제 개수와 중간 순서는 더 복잡할 수 있습니다.</p>
<p>Deployment 설정의 일부입니다.</p>
<pre><code class="language-yaml">replicas: 3
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0</code></pre>
<p><code>maxSurge</code>는 목표 수보다 추가로 만들 수 있는 수·비율, <code>maxUnavailable</code>은 교체 과정에서 허용하는 가용 복제본 부족 수·비율입니다. 후자가 0이라고 외부 장애까지 차단해 가용성을 절대 보장하는 것은 아닙니다.</p>
<h3 id="6-4-readiness는-켜졌는가보다-구체적이어야-한다">6-4. Readiness는 “켜졌는가”보다 구체적이어야 한다</h3>
<p>앱 프로세스가 떠 있어도 DB 연결, 캐시 준비, 필수 파일 로딩이 끝나지 않았을 수 있습니다.</p>
<pre><code class="language-text">Running: 프로세스가 실행 중인가?
Ready: 지금 요청을 정상 처리할 준비가 되었는가?</code></pre>
<p>준비 상태를 제대로 측정하지 않으면 새 앱이 너무 일찍 요청을 받을 수 있습니다. 반대로 항상 실패하도록 설정하면 새 Pod가 끝내 가용해지지 않아 배포가 진행되지 않을 수 있습니다.</p>
<p>Readiness 실패는 일반적으로 그 자체로 컨테이너 재시작을 일으키는 조건이 아닙니다. Liveness·Startup Probe의 역할과 구분합니다.</p>
<h3 id="6-5-정상-종료도-배포의-일부다">6-5. 정상 종료도 배포의 일부다</h3>
<pre><code class="language-text">기존 앱을 연결 대상에서 제외하는 과정
                ↕ 비동기 반영
종료 신호 전달 → 처리 중인 작업 마무리 → 프로세스 종료</code></pre>
<p>EndpointSlice 갱신과 프록시·클라이언트의 반영에는 시간이 걸릴 수 있습니다. 요청 전달 중단과 프로세스 종료가 모든 위치에서 정확히 동시에 일어나는 것은 아닙니다.</p>
<p>앱의 SIGTERM 처리, 연결 종료, 종료 유예 시간, 필요하면 <code>preStop</code> 같은 종료 절차를 함께 설계합니다. 기다리는 시간만 길게 정한다고 데이터 안전과 연결 유지가 자동 해결되지는 않습니다.</p>
<h3 id="6-6-bluegreen-새-가게를-준비한-뒤-안내를-바꾸기">6-6. Blue/Green: 새 가게를 준비한 뒤 안내를 바꾸기</h3>
<pre><code class="language-text">전환 전
사용자 → Service → Blue 환경
                    Green 환경은 별도 준비·점검

전환 후
사용자 → 같은 Service → Green 환경
                         Blue는 복귀 가능 여부를 보며 유지</code></pre>
<p>Blue와 Green은 색 이름일 뿐 항상 Blue가 구버전이라는 고정 규칙은 아닙니다. 현재 서비스 환경과 교체할 환경을 구분하는 이름입니다.</p>
<h3 id="6-7-service-기반-bluegreen의-핵심은-selector">6-7. Service 기반 Blue/Green의 핵심은 Selector</h3>
<pre><code class="language-text">Blue Pod:  app=release-demo, track=blue
Green Pod: app=release-demo, track=green</code></pre>
<p>Service의 Selector를 다음처럼 바꿉니다.</p>
<pre><code class="language-yaml">selector:
  app: release-demo
  track: blue</code></pre>
<pre><code class="language-yaml">selector:
  app: release-demo
  track: green</code></pre>
<p>이름이 같은 Service의 연결 대상만 전환합니다. Selector에 사용하는 Label은 <strong>Pod 템플릿에 실제로 있어야</strong> 합니다. Deployment 자체의 <code>metadata.labels</code>에만 적으면 Pod 선택 조건을 만족시키지 못할 수 있습니다.</p>
<h3 id="6-8-bluegreen-보충-예제">6-8. Blue/Green 보충 예제</h3>
<p>다음은 앱 코드를 따로 준비하지 않고 화면의 버전 표시로 전환을 확인하는 독립 예제입니다. <code>blue-green-demo.yaml</code>로 저장합니다.</p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-blue
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: release-demo
      track: blue
  template:
    metadata:
      labels:
        app: release-demo
        track: blue
    spec:
      containers:
        - name: web
          image: nginx:1.27-alpine
          command: [&quot;sh&quot;, &quot;-c&quot;]
          args:
            - |
              echo 'BLUE release' &gt; /usr/share/nginx/html/index.html
              exec nginx -g 'daemon off;'
          readinessProbe:
            httpGet:
              path: /
              port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-green
  namespace: demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: release-demo
      track: green
  template:
    metadata:
      labels:
        app: release-demo
        track: green
    spec:
      containers:
        - name: web
          image: nginx:1.27-alpine
          command: [&quot;sh&quot;, &quot;-c&quot;]
          args:
            - |
              echo 'GREEN release' &gt; /usr/share/nginx/html/index.html
              exec nginx -g 'daemon off;'
          readinessProbe:
            httpGet:
              path: /
              port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: release-entry
  namespace: demo
spec:
  selector:
    app: release-demo
    track: blue
  ports:
    - port: 80
      targetPort: 80</code></pre>
<p>두 Deployment의 Selector에는 서로 다른 <code>track</code>이 있습니다. 서로 같은 Pod를 관리하려는 혼선을 피하고, Service에서만 전환 대상을 고르는 구조입니다.</p>
<h3 id="6-9-생성-→-green-확인-→-전환-→-복귀">6-9. 생성 → Green 확인 → 전환 → 복귀</h3>
<pre><code class="language-bash">kubectl apply -f blue-green-demo.yaml
kubectl rollout status deployment/release-blue -n demo
kubectl rollout status deployment/release-green -n demo</code></pre>
<p>먼저 현재 Service가 Blue를 가리키는지 확인합니다.</p>
<pre><code class="language-bash">kubectl run blue-check -n demo --rm -i --restart=Never \
  --image=busybox:1.37 -- wget -qO- \
  http://release-entry.demo.svc.cluster.local/</code></pre>
<p>이 명령은 현재 Service의 대상을 확인하므로 아직 Blue가 나오는 것이 정상입니다. Green만 직접 확인하려면 별도 테스트 Service를 두거나 Green Deployment의 Pod에 port-forward를 사용할 수 있습니다.</p>
<pre><code class="language-bash">kubectl port-forward deployment/release-green -n demo 8092:80</code></pre>
<p>다른 터미널에서 확인합니다.</p>
<pre><code class="language-bash">curl http://localhost:8092/</code></pre>
<p>Green이 준비되었다면 Service를 전환합니다.</p>
<pre><code class="language-bash">kubectl patch service release-entry -n demo --type=merge \
  -p '{&quot;spec&quot;:{&quot;selector&quot;:{&quot;app&quot;:&quot;release-demo&quot;,&quot;track&quot;:&quot;green&quot;}}}'

kubectl get endpointslice -n demo \
  -l kubernetes.io/service-name=release-entry</code></pre>
<p>다시 서비스로 새 요청을 보내 응답을 확인합니다.</p>
<pre><code class="language-bash">kubectl run release-check -n demo --rm -i --restart=Never \
  --image=busybox:1.37 -- wget -qO- \
  http://release-entry.demo.svc.cluster.local/</code></pre>
<p>연결 대상 반영이 끝나면 <code>GREEN release</code>가 보입니다. 복귀는 Selector를 Blue로 바꿉니다.</p>
<pre><code class="language-bash">kubectl patch service release-entry -n demo --type=merge \
  -p '{&quot;spec&quot;:{&quot;selector&quot;:{&quot;app&quot;:&quot;release-demo&quot;,&quot;track&quot;:&quot;blue&quot;}}}'</code></pre>
<p>Service 대상으로 port-forward를 열면 특정 Pod로 연결되므로, <strong>이미 열린 port-forward가 Service 전환을 계속 따라간다고 가정하지 않습니다.</strong> 위처럼 클러스터 내부에서 Service로 새 요청을 보내거나 실제 Ingress 경로로 확인합니다.</p>
<h3 id="6-10-ingress-기반-bluegreen">6-10. Ingress 기반 Blue/Green</h3>
<p>Service Selector 대신 Ingress의 백엔드 Service를 바꾸는 방식도 있습니다.</p>
<pre><code class="language-text">전환 전: Ingress → blue-service → Blue Pod
전환 후: Ingress → green-service → Green Pod</code></pre>
<p>두 방식 모두 연결 대상을 바꾸지만 바꾸는 계층이 다릅니다. 전환 후 프록시의 설정 반영, 기존 연결의 유지 여부, 세션 처리도 확인합니다.</p>
<h3 id="6-11-canary-작은-손님-그룹으로-먼저-확인하기">6-11. Canary: 작은 손님 그룹으로 먼저 확인하기</h3>
<p>새 메뉴를 전 손님에게 판매하기 전에 일부 손님에게 먼저 제공해 반응을 보는 것과 비슷합니다.</p>
<pre><code class="language-text">기존 버전: 대부분의 요청
새 버전: 일부 요청 또는 선택한 사용자 그룹</code></pre>
<p>새 버전의 오류율·지연 시간·업무 성공률 등을 확인하고 문제가 없으면 점차 대상을 늘립니다. 문제가 있으면 새 버전으로 가는 요청을 줄이거나 중단합니다.</p>
<h3 id="6-12-service만으로-canary를-흉내-낼-수-있을까">6-12. Service만으로 Canary를 흉내 낼 수 있을까?</h3>
<p>같은 Service Selector에 구버전·신버전 Pod가 모두 선택되게 할 수는 있습니다.</p>
<pre><code class="language-text">기존 Pod 9개 + 신규 Pod 1개
Service는 app=canary-demo만 선택</code></pre>
<p>하지만 이것만으로 요청이 정확히 90:10으로 분배된다고 보장할 수는 없습니다. 연결 재사용, 세션 고정, 클라이언트 특성, 네트워크 구현에 영향을 받습니다.</p>
<p>정밀한 가중치·사용자 그룹별 배분이 필요하면 이를 지원하는 프록시·Gateway·서비스 메시 등의 기능을 사용합니다. Pod 개수 비율과 요청 수 비율을 같은 값이라고 단정하지 않습니다.</p>
<h3 id="6-13-canary의-세-가지-분기-기준">6-13. Canary의 세 가지 분기 기준</h3>
<table>
<thead>
<tr>
<th>기준</th>
<th>예시</th>
<th>활용</th>
</tr>
</thead>
<tbody><tr>
<td>Header</td>
<td><code>X-Preview: always</code></td>
<td>테스트 요청을 명확히 선택</td>
</tr>
<tr>
<td>Cookie</td>
<td><code>preview-group=always</code></td>
<td>특정 사용자 그룹을 유지</td>
</tr>
<tr>
<td>Weight</td>
<td>새 버전으로 약 10% 전달</td>
<td>일반 요청 일부로 관측</td>
</tr>
</tbody></table>
<p>헤더·쿠키가 있다고 인증된 관리자가 되는 것은 아닙니다. 이 조건은 라우팅 기준입니다. 보안 권한 검사를 대체하지 않습니다.</p>
<h3 id="6-14-교재의-ingress-nginx-canary-annotation-읽기">6-14. 교재의 ingress-nginx Canary annotation 읽기</h3>
<p>다음은 종료된 커뮤니티 ingress-nginx의 <strong>기존 교재 문법을 해석하기 위한 설정 조각</strong>입니다. Kubernetes 표준 Ingress 필드가 아니며, 신규 운영용 설치 예제가 아닙니다.</p>
<p>Header 기반:</p>
<pre><code class="language-yaml">metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: &quot;true&quot;
    nginx.ingress.kubernetes.io/canary-by-header: &quot;X-Preview&quot;</code></pre>
<p>Cookie 기반:</p>
<pre><code class="language-yaml">metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: &quot;true&quot;
    nginx.ingress.kubernetes.io/canary-by-cookie: &quot;preview-group&quot;</code></pre>
<p>Weight 기반:</p>
<pre><code class="language-yaml">metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: &quot;true&quot;
    nginx.ingress.kubernetes.io/canary-weight: &quot;10&quot;</code></pre>
<p>이 구현에서는 보통 안정 버전 Ingress와 같은 Host·Path를 사용하는 Canary Ingress가 별도 백엔드 Service를 가리키게 구성합니다. 조건이 여러 개면 우선순위 규칙도 있으므로, 실습에서는 한 가지 기준씩 분리해 확인합니다.</p>
<p>해당 annotation 문법은 <a href="https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#canary">프로젝트의 Canary 문서</a>에서 확인할 수 있습니다. 다른 Controller에서는 그 구현의 필드와 정책을 사용해야 합니다.</p>
<h3 id="6-15-브라우저-확장-기능과-curl로-검증하기">6-15. 브라우저 확장 기능과 curl로 검증하기</h3>
<p>교재처럼 브라우저의 헤더 수정 확장 기능을 사용해 요청 헤더·쿠키를 넣을 수 있습니다. 같은 테스트를 curl로도 표현할 수 있습니다.</p>
<pre><code class="language-bash"># 일반 요청
curl -i https://web.example.com/

# Header 기준 테스트
curl -i -H 'X-Preview: always' https://web.example.com/

# Cookie 기준 테스트
curl -i -b 'preview-group=always' https://web.example.com/</code></pre>
<p>이 명령들은 통제하는 실제 도메인과 구성으로 바꿔 사용하는 형식 예제입니다.</p>
<p>Header 실습을 마치고 Cookie·Weight 실습으로 넘어갈 때는 이전 헤더·쿠키가 남아 있지 않은지 확인합니다. 브라우저 캐시나 고정된 연결 때문에 기대한 분배가 안 보일 수도 있습니다.</p>
<p>가중치 10%라고 요청 10번 중 정확히 1번이 새 버전으로 간다는 뜻은 아닙니다. 충분한 표본과 관측으로 분포를 판단합니다.</p>
<h3 id="6-16-배포-성공을-무엇으로-판단할까">6-16. 배포 성공을 무엇으로 판단할까?</h3>
<table>
<thead>
<tr>
<th>지표·관찰</th>
<th>묻는 질문</th>
</tr>
</thead>
<tbody><tr>
<td>오류율</td>
<td>새 버전에서 실패가 늘었는가?</td>
</tr>
<tr>
<td>지연 시간</td>
<td>일부 요청이 비정상적으로 느려졌는가?</td>
</tr>
<tr>
<td>준비 상태</td>
<td>목표 복제본이 실제 요청을 받을 수 있는가?</td>
</tr>
<tr>
<td>업무 성공률</td>
<td>로그인·조회·결제 같은 핵심 작업이 성공하는가?</td>
</tr>
<tr>
<td>로그·추적</td>
<td>어느 버전·어느 의존 서비스에서 문제가 생기는가?</td>
</tr>
</tbody></table>
<p>허용할 변화와 중단 기준은 서비스 상황에 맞게 정합니다. 임의의 숫자를 모든 서비스에 통용되는 성공 기준으로 쓰지 않습니다.</p>
<h3 id="6-17-이미지-태그만-다르다고-같은-앱의-호환-버전은-아니다">6-17. 이미지 태그만 다르다고 같은 앱의 호환 버전은 아니다</h3>
<p>교육 실습에서는 태그에 따라 서로 다른 구현의 앱을 사용하는 경우도 있습니다. 운영 배포에서는 이름만 보고 호환 버전이라고 판단하지 않습니다.</p>
<p>API 경로, 응답 구조, 포트, 환경 변수, DB 스키마, 건강 상태 경로가 서로 맞는지 확인해야 합니다. 같은 URL로 접속되는 것과 동일한 업무 동작을 제공하는 것은 다른 검증입니다.</p>
<h3 id="6-18-rollback이-데이터도-되돌릴까">6-18. Rollback이 데이터도 되돌릴까?</h3>
<pre><code class="language-text">Deployment rollback → 유지된 이력의 Pod 템플릿 복귀
DB·파일·외부 시스템 변경 → 별도 복구와 호환성 문제</code></pre>
<p>새 버전이 DB 구조를 바꾼 뒤 이전 앱으로 돌아가면 이전 앱이 새 데이터 형식을 읽지 못할 수 있습니다. 데이터 변경을 구·신 버전이 함께 이해할 수 있게 단계적으로 적용하거나 별도 복구 계획을 설계합니다.</p>
<p>Blue/Green도 같은 DB를 사용한다면 같은 문제를 공유할 수 있습니다. 연결을 다시 Blue로 바꾸는 것만으로 모든 변경이 취소되지는 않습니다.</p>
<h3 id="6-19-배포-상태-확인-명령">6-19. 배포 상태 확인 명령</h3>
<pre><code class="language-bash">kubectl get deployment,pod -n demo
kubectl rollout status deployment/release-green -n demo
kubectl rollout history deployment/release-green -n demo
kubectl get endpointslice -n demo \
  -l kubernetes.io/service-name=release-entry
kubectl get events -n demo --sort-by=.metadata.creationTimestamp</code></pre>
<p>Rolling Update 중 정상 진전이 없는 상태가 기록된다고 기본 Deployment가 자동으로 이전 버전으로 복귀하는 것은 아닙니다. 배포 자동화 도구가 별도 정책을 실행하는지 확인해야 합니다.</p>
<p><strong>이 파트의 핵심:</strong> 배포 방식은 교체 순서와 트래픽을 조절합니다. 앱 준비·정상 종료·데이터 호환성·관측과 복구가 함께 맞아야 중단 위험을 줄일 수 있습니다.</p>
<hr />
<h2 id="part-7-복습-퀴즈-개념을-내-말로-설명하기">PART 7. 복습 퀴즈: 개념을 내 말로 설명하기</h2>
<blockquote>
<p>아래 문제는 학습을 위해 새로 만들었습니다. 교재의 내부 평가 문항이나 정답지를 옮긴 것이 아닙니다. 먼저 답을 생각한 뒤 해설을 읽어 보세요.</p>
</blockquote>
<h3 id="7-1-저장소-퀴즈">7-1. 저장소 퀴즈</h3>
<p><strong>문제 1.</strong> Pod 교체 후에도 사진을 보관해야 합니다. emptyDir를 쓰면 충분할까요?</p>
<p><strong>답:</strong> 충분하지 않습니다. emptyDir는 해당 Pod의 임시 공간입니다. 지속 보관이 가능한 저장소를 연결하고 보존·백업 정책을 확인해야 합니다.</p>
<p><strong>문제 2.</strong> PVC가 Bound이면 앱에서 파일 쓰기가 성공했다고 판단해도 될까요?</p>
<p><strong>답:</strong> 아닙니다. Bound는 PVC와 PV의 논리 연결 상태입니다. 실제 Pod 배치와 마운트, 파일 권한, 앱의 쓰기 성공을 확인해야 합니다.</p>
<p><strong>문제 3.</strong> RWO는 오직 Pod 한 개만 볼륨을 사용할 수 있다는 뜻일까요?</p>
<p><strong>답:</strong> 아닙니다. 한 노드에서의 읽기·쓰기 마운트 조건입니다. 단일 Pod 제한인 RWOP와 구분합니다.</p>
<p><strong>문제 4.</strong> 동적 프로비저닝에서는 사용자가 PVC마다 PV를 직접 작성해야 할까요?</p>
<p><strong>답:</strong> 일반적으로 지원 프로비저너가 StorageClass에 따라 PV와 저장소를 준비합니다. 드라이버·권한·StorageClass가 먼저 구성되어 있어야 합니다.</p>
<p><strong>문제 5.</strong> WaitForFirstConsumer인 PVC가 Pod 생성 전 Pending이면 반드시 실패일까요?</p>
<p><strong>답:</strong> 아닙니다. 소비할 Pod의 위치·조건을 기다리는 정상 상태일 수 있습니다. PVC 이벤트와 바인딩 모드를 확인합니다.</p>
<p><strong>문제 6.</strong> Retain 정책이면 새 PVC가 이전 파일을 자동으로 즉시 이어받을까요?</p>
<p><strong>답:</strong> 아닙니다. 데이터를 남기는 정책과 재사용·바인딩 절차는 다릅니다. 기존 리소스 상태와 권한을 처리해야 합니다.</p>
<h3 id="7-2-statefulset과-설정-퀴즈">7-2. StatefulSet과 설정 퀴즈</h3>
<p><strong>문제 7.</strong> StatefulSet이 교체한 Pod의 이름이 같으면 UID와 IP도 같을까요?</p>
<p><strong>답:</strong> 아닙니다. 안정적인 이름을 재사용할 수 있지만 새 Pod 객체의 UID와 IP는 달라질 수 있습니다.</p>
<p><strong>문제 8.</strong> StatefulSet의 <code>serviceName</code>을 적으면 Headless Service가 자동 생성될까요?</p>
<p><strong>답:</strong> 아닙니다. 참조할 Service를 별도로 준비해야 합니다.</p>
<p><strong>문제 9.</strong> Headless Service는 일반적인 ClusterIP 가상 IP를 할당할까요?</p>
<p><strong>답:</strong> 아닙니다. <code>clusterIP: None</code>을 지정합니다. DNS로 Pod 주소를 찾는 데 활용합니다.</p>
<p><strong>문제 10.</strong> ConfigMap을 바꾸면 실행 중 앱의 환경 변수도 즉시 바뀔까요?</p>
<p><strong>답:</strong> 아닙니다. 환경 변수는 일반적으로 컨테이너 시작 시 전달됩니다. 새 Pod로 반영하는 등의 절차가 필요합니다.</p>
<p><strong>문제 11.</strong> ConfigMap 파일이 바뀌면 앱의 동작도 반드시 바뀔까요?</p>
<p><strong>답:</strong> 아닙니다. 앱이 파일을 다시 읽는지 확인해야 합니다. <code>subPath</code> 마운트는 자동 갱신되지 않는다는 점도 구분합니다.</p>
<p><strong>문제 12.</strong> Secret의 Base64 값을 공개해도 원문을 모르니 괜찮을까요?</p>
<p><strong>답:</strong> 아닙니다. Base64는 되돌릴 수 있는 인코딩입니다. 실제 비밀과 같은 수준으로 보호해야 합니다.</p>
<p><strong>문제 13.</strong> 노트북에서 Docker 로그인에 성공하면 Pod도 프라이빗 이미지를 자동으로 받을까요?</p>
<p><strong>답:</strong> 그렇지 않습니다. 노드의 이미지 다운로드 인증은 imagePullSecrets나 환경별 인증 체계 등을 따로 확인합니다.</p>
<h3 id="7-3-외부-접속과-배포-퀴즈">7-3. 외부 접속과 배포 퀴즈</h3>
<p><strong>문제 14.</strong> Ingress YAML만 만들면 인터넷 접속이 완성될까요?</p>
<p><strong>답:</strong> 아닙니다. 이를 처리할 Controller, 외부 진입점, DNS, 연결 정책 등이 필요합니다.</p>
<p><strong>문제 15.</strong> Ingress의 백엔드 포트는 항상 컨테이너의 포트일까요?</p>
<p><strong>답:</strong> 아닙니다. 백엔드 Service의 포트를 가리킵니다. Service의 targetPort가 실제 앱 포트로 연결됩니다.</p>
<p><strong>문제 16.</strong> HTTPS를 사용하면 Ingress에서 앱까지의 연결도 자동으로 암호화될까요?</p>
<p><strong>답:</strong> 아닙니다. TLS 종료 이후 백엔드 연결 방식은 별도 설정에 달려 있습니다.</p>
<p><strong>문제 17.</strong> Certificate와 TLS Secret은 같은 리소스일까요?</p>
<p><strong>답:</strong> 아닙니다. Certificate는 발급·관리 요청을 표현하고, TLS Secret에는 인증서와 개인키가 저장됩니다.</p>
<p><strong>문제 18.</strong> HTTP-01로 와일드카드 인증서의 도메인 검증을 할 수 있을까요?</p>
<p><strong>답:</strong> 해당 용도에는 DNS-01을 사용합니다. HTTP-01과 DNS-01의 접근·권한 조건도 다릅니다.</p>
<p><strong>문제 19.</strong> <code>maxUnavailable: 0</code>이면 모든 경우에 무중단이 보장될까요?</p>
<p><strong>답:</strong> 아닙니다. 앱 준비 상태, 종료 처리, 외부 장애, 여유 자원, 데이터 호환성 등을 함께 고려해야 합니다.</p>
<p><strong>문제 20.</strong> Canary 가중치 10%라면 요청 10번 중 정확히 1번은 새 버전일까요?</p>
<p><strong>답:</strong> 아닙니다. 분배 정책과 연결 특성에 따라 달라집니다. 작은 표본을 정확한 비율 보장으로 해석하지 않습니다.</p>
<p><strong>문제 21.</strong> Blue/Green에서 Service Selector를 바꾸면 DB 변경도 취소될까요?</p>
<p><strong>답:</strong> 아닙니다. 라우팅 복귀와 데이터 복구는 별도 작업입니다.</p>
<p><strong>문제 22.</strong> <code>kubectl port-forward service/...</code>를 열어 두면 이후 Service Selector 전환을 계속 추적할까요?</p>
<p><strong>답:</strong> 그렇게 가정하면 안 됩니다. 특정 Pod로 연결된 터널일 수 있습니다. 실제 Service·Ingress 경로로 새 요청을 보내 전환을 확인합니다.</p>
<hr />
<h2 id="part-8-조별-토론-장애-원인을-단계별로-좁히기">PART 8. 조별 토론: 장애 원인을 단계별로 좁히기</h2>
<blockquote>
<p>다음 상황도 공개용으로 새로 만든 보충 토론입니다. 실제 참여자나 내부 시스템의 정보는 사용하지 않습니다.</p>
</blockquote>
<h3 id="8-1-토론-방법">8-1. 토론 방법</h3>
<p>정답 용어부터 말하기보다 다음 순서로 이야기해 보세요.</p>
<pre><code class="language-text">관찰한 현상 → 관련 계층 → 확인할 증거 → 원인 후보 → 조치와 검증</code></pre>
<p>각 팀원은 하나의 원인만 주장하지 말고, 자신의 추측을 확인하거나 기각할 관찰을 제시합니다.</p>
<h3 id="8-2-사례-a-pvc가-pending이다">8-2. 사례 A: PVC가 Pending이다</h3>
<p><strong>상황:</strong> 앱은 만들었지만 저장소를 사용할 수 없습니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>요청한 StorageClass가 실제로 존재하나요?</li>
<li>정적 PV가 필요하다면 용량·접근 모드·클래스가 맞나요?</li>
<li>WaitForFirstConsumer로 소비할 Pod를 기다리나요?</li>
<li>프로비저너에 필요한 권한과 드라이버가 있나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p><code>kubectl describe pvc</code>의 이벤트를 먼저 확인합니다. Pending이라는 단어만으로 용량 부족이나 드라이버 오류를 확정하지 않습니다. Pod의 배치 실패와 연결되어 있을 수도 있습니다.</p>
<h3 id="8-3-사례-b-두-pod가-같은-파일을-쓰다가-내용이-깨졌다">8-3. 사례 B: 두 Pod가 같은 파일을 쓰다가 내용이 깨졌다</h3>
<p><strong>상황:</strong> 공유 저장소를 RWX로 연결했는데 파일 내용이 기대와 다릅니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>두 앱이 같은 경로와 파일 이름을 쓰나요?</li>
<li>덮어쓰기와 추가 쓰기 중 무엇을 하나요?</li>
<li>잠금·원자적 변경·동시 접근 제어가 있나요?</li>
<li>파일 공유 방식이 앱의 요구에 맞나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>RWX는 여러 노드의 마운트 지원 조건이지 데이터 충돌 해결 기능이 아닙니다. 저장소가 정상이어도 앱의 동시 쓰기 설계 때문에 문제가 발생할 수 있습니다.</p>
<h3 id="8-4-사례-c-statefulset을-만들었는데-db-데이터가-다르다">8-4. 사례 C: StatefulSet을 만들었는데 DB 데이터가 다르다</h3>
<p><strong>상황:</strong> Pod 3개가 모두 Running인데 서로 데이터가 다릅니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>각 Pod가 독립적인 PVC를 사용하는 것은 의도한 구조인가요?</li>
<li>DB 복제 설정과 구성원 연결이 되어 있나요?</li>
<li>Headless DNS를 앱이 실제로 사용하나요?</li>
<li>클러스터 초기화·리더 선출이 끝났나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>StatefulSet은 안정적인 정체성과 저장소 연결을 제공하지만 DB 복제 프로토콜을 대신하지 않습니다. Kubernetes의 Running과 DB 클러스터의 정상 상태를 구분해야 합니다.</p>
<h3 id="8-5-사례-d-configmap은-바뀌었는데-로그-수준은-그대로다">8-5. 사례 D: ConfigMap은 바뀌었는데 로그 수준은 그대로다</h3>
<p><strong>상황:</strong> 설정 리소스에서는 새 값이 보이지만 앱은 예전 값으로 동작합니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>환경 변수로 전달했나요, 파일로 연결했나요?</li>
<li>subPath를 사용하나요?</li>
<li>앱이 시작할 때 한 번만 읽나요?</li>
<li>Pod가 새 설정으로 다시 생성됐나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>리소스 저장 → 전달 파일·환경 변수 → 앱의 읽기라는 단계를 나눕니다. 환경 변수라면 Pod 교체, 파일이라면 갱신과 재로딩을 확인합니다.</p>
<h3 id="8-6-사례-e-tls-secret이-있는데-브라우저가-경고한다">8-6. 사례 E: TLS Secret이 있는데 브라우저가 경고한다</h3>
<p><strong>상황:</strong> Secret은 존재하지만 HTTPS 접속에 문제가 있습니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>Secret이 Ingress와 같은 네임스페이스에 있나요?</li>
<li><code>tls.hosts</code>, 규칙의 Host, 접속 도메인이 맞나요?</li>
<li>인증서의 유효 이름과 기간, 신뢰 체인이 맞나요?</li>
<li>DNS가 기대한 Controller 진입점으로 가나요?</li>
<li>실제 프록시가 해당 인증서를 사용하고 있나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>Secret 존재 여부와 인증서의 올바른 적용은 다릅니다. 개인키 내용을 공유하는 대신 인증서 준비 상태, 호스트, Events와 외부에 제시되는 인증서를 확인합니다.</p>
<h3 id="8-7-사례-f-canary-요청이-전부-구버전으로-간다">8-7. 사례 F: Canary 요청이 전부 구버전으로 간다</h3>
<p><strong>상황:</strong> 가중치나 헤더를 설정했지만 새 버전이 보이지 않습니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>현재 Controller가 해당 기능을 지원하나요?</li>
<li>IngressClass와 Host·Path가 맞나요?</li>
<li>Canary Service에 실제 Ready 대상이 있나요?</li>
<li>이전 헤더·쿠키·세션 고정 조건이 남아 있나요?</li>
<li>충분한 표본으로 새 연결을 만들어 확인했나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>전용 annotation을 지원하지 않는 Controller에서는 설정을 적어도 원하는 동작이 나오지 않을 수 있습니다. 새 버전의 응답을 구분하는 표시도 필요합니다. 화면 캐시와 실제 서버 응답을 구분합니다.</p>
<h3 id="8-8-사례-g-같은-label-때문에-다른-pod에-연결된다">8-8. 사례 G: 같은 Label 때문에 다른 Pod에 연결된다</h3>
<p><strong>상황:</strong> 이전 독립 Pod와 새 Deployment의 Pod가 같은 Label을 가집니다. <code>exec deployment/...</code>나 port-forward가 기대하지 않은 Pod를 고릅니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>Selector가 어떤 Pod를 모두 선택하나요?</li>
<li>독립 Pod와 ReplicaSet 소유 Pod를 구분했나요?</li>
<li>필요한 컨테이너가 선택된 Pod에 실제로 있나요?</li>
<li>실행 중인 앱이 대상 포트에서 응답하나요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>리소스 이름과 Selector 기반 선택은 다른 문제입니다. Pod 목록·Label·ownerReferences를 확인하고, 진단 시 정확한 Pod를 지정합니다. 실습을 나누는 Label을 설계하고, 끝난 자원을 정리할 때도 소유 범위를 확인합니다.</p>
<p>포트포워딩의 <code>connection refused</code>는 선택된 Pod의 해당 포트에 앱이 듣지 않는 상황 등에서 발생할 수 있습니다. Ingress 장애와 동일하게 단정하지 않습니다.</p>
<h3 id="8-9-사례-h-새-버전-복귀-후에도-업무-오류가-계속된다">8-9. 사례 H: 새 버전 복귀 후에도 업무 오류가 계속된다</h3>
<p><strong>상황:</strong> 앱을 이전 이미지로 돌렸지만 기능이 정상화되지 않았습니다.</p>
<p><strong>토론 질문</strong></p>
<ul>
<li>DB 형식과 외부 설정도 바뀌었나요?</li>
<li>이전 앱이 변경된 데이터를 읽을 수 있나요?</li>
<li>복귀한 이미지 태그가 실제로 기대한 digest인가요?</li>
<li>클라이언트 캐시·기존 세션·비동기 작업은 어떤 상태인가요?</li>
</ul>
<p><strong>풀이 방향</strong></p>
<p>이미지 변경, 설정 변경, 데이터 변경, 트래픽 전환을 따로 추적합니다. “rollback 명령 성공”과 “업무 정상화 확인”은 서로 다른 완료 조건입니다.</p>
<h3 id="8-10-설계-토론-작은-온라인-서점-만들기">8-10. 설계 토론: 작은 온라인 서점 만들기</h3>
<p>다음 요구사항을 한 장의 구조로 연결해 보세요.</p>
<table>
<thead>
<tr>
<th>요구</th>
<th>관련 개념</th>
<th>판단할 점</th>
</tr>
</thead>
<tbody><tr>
<td>웹 API 여러 개 운영</td>
<td>Deployment·Service</td>
<td>복제본 수와 준비 상태</td>
</tr>
<tr>
<td>업로드한 표지 이미지 보관</td>
<td>외부 저장소·PVC 등</td>
<td>공유 필요성, 삭제·백업 정책</td>
</tr>
<tr>
<td>로그 수준을 환경별로 변경</td>
<td>ConfigMap</td>
<td>환경 변수·파일 중 전달 방식</td>
</tr>
<tr>
<td>외부 시스템 인증 자료</td>
<td>Secret</td>
<td>권한, 변경 반영, 로그 노출</td>
</tr>
<tr>
<td>도메인으로 접속</td>
<td>DNS·Ingress·Controller</td>
<td>클래스·호스트·경로·외부 연결</td>
</tr>
<tr>
<td>HTTPS 제공</td>
<td>TLS Secret·cert-manager</td>
<td>도메인 제어권과 갱신</td>
</tr>
<tr>
<td>새 버전 일부 검증</td>
<td>Canary</td>
<td>분기 기준과 성공·중단 지표</td>
</tr>
<tr>
<td>개별 DB 구성원 관리</td>
<td>StatefulSet·Headless Service</td>
<td>DB 복제와 복구는 별도 설계</td>
</tr>
</tbody></table>
<p>처음부터 모든 리소스를 써야 좋은 설계가 되는 것은 아닙니다. “이 요구에 이 기능이 필요한 이유”를 설명할 수 있어야 합니다.</p>
<hr />
<h2 id="전체-내용을-한-번에-연결하기">전체 내용을 한 번에 연결하기</h2>
<pre><code class="language-text">앱 이미지
  ↓ Deployment 또는 StatefulSet으로 실행
Pod
  ├─ PV/PVC: 지속 데이터 연결
  ├─ ConfigMap: 일반 설정 전달
  ├─ Secret: 민감한 설정 전달
  └─ Readiness: 요청 처리 준비 상태 표시
       ↑
Service 또는 Headless Service
       ↑
Ingress 규칙과 실제 전달 구현
       ↑
DNS·외부 연결·HTTPS
       ↑
사용자

버전 교체: Rolling Update / Blue·Green / Canary
검증: 준비 상태 + 응답 + 지표 + 데이터·업무 정상 여부</code></pre>
<p>중요한 구분은 네 가지입니다.</p>
<ol>
<li><strong>실행과 데이터:</strong> Pod 교체와 데이터 보관은 다른 수명입니다.</li>
<li><strong>설정 저장과 반영:</strong> 리소스 수정과 앱 동작 변경은 다른 단계입니다.</li>
<li><strong>규칙과 구현:</strong> Ingress 선언과 실제 접속 처리 담당자는 다릅니다.</li>
<li><strong>배포 명령과 서비스 정상화:</strong> 명령 성공 뒤에도 요청과 업무 결과를 검증해야 합니다.</li>
</ol>
<h2 id="실습-정리와-공개-글-점검">실습 정리와 공개 글 점검</h2>
<p>학습용 Pod·Deployment·Service는 본인이 적용한 YAML을 대상으로 정리합니다. 다른 실습과 같은 이름·Label을 사용했다면 삭제 범위를 먼저 확인합니다.</p>
<p>PVC·PV·StorageClass는 일반 앱 리소스와 삭제 영향을 구분합니다. 특히 PVC 삭제 전에 회수 정책과 실제 저장소의 데이터·비용 영향을 확인합니다. 공유 클러스터에서 전체 네임스페이스를 지우는 방식으로 정리하지 않습니다.</p>
<p>공개 글에는 실제 Secret 값, 인증 파일, 개인키, 내부 도메인, 클라우드 계정·저장소 식별자, 참여자 정보가 들어 있지 않은지 확인합니다. Base64로 보이는 값도 실제 인증 자료라면 공개하지 않습니다.</p>
<h2 id="학습-범위와-검증-안내">학습 범위와 검증 안내</h2>
<p>이 글은 첨부 Kubernetes 심화 교재의 2일차 주제를 개념 중심으로 다시 설명한 학습 자료입니다. 퀴즈·토론·보충 YAML은 새로 구성했고, 교재 내부 평가나 실습 화면을 재배포하지 않았습니다.</p>
<p>이 문서의 예제를 실제 클러스터에 일괄 배포해 검증한 것은 아닙니다. 실제 적용에는 권한, 네임스페이스, StorageClass, 드라이버, IngressClass, DNS, 발급자와 저장소 접근 조건이 필요합니다. 설명용 이름과 문서용 주소를 그대로 사용하는 예제와 환경에 맞게 바꿔야 하는 예제를 구분해서 읽습니다.</p>
<p>교재 뒤의 별도 Self-Study인 TLS 상세 메커니즘, Job·CronJob, Pod Lifecycle 심화, Service 심화, SecurityContext, QoS·PriorityClass는 이번 시간표의 독립 파트가 아닙니다. 2일차 주제를 이해하는 데 필요한 TLS·건강 상태·종료 처리 개념은 관련 파트 안에 설명했습니다.</p>