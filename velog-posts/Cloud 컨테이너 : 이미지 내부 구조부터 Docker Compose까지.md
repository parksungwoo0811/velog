<blockquote>
<p>1일차에는 이미지를 만들고 컨테이너로 실행하는 방법을 배웠다.<br />2일차에는 한 단계 더 들어가 <strong>이미지는 실제로 무엇으로 구성되는지</strong>, <strong>컨테이너가 Linux 커널 위에서 어떻게 격리되는지</strong>, <strong>여러 컨테이너를 Docker Compose로 어떻게 관리하는지</strong>를 배운다.</p>
</blockquote>
<blockquote>
<p>실습 경로의 <code>~/path/to/skala-container</code>는 예시이다. 본인의 컴퓨터에서 <code>skala-container</code> 프로젝트를 저장한 경로로 바꿔서 사용하면 된다.</p>
</blockquote>
<hr />
<h2 id="오늘-수업의-전체-흐름">오늘 수업의 전체 흐름</h2>
<pre><code class="language-text">Dockerfile
   |
   | build
   v
OCI Image
├── manifest
├── config
└── filesystem layers
   |
   | run
   v
dockerd → containerd → shim → runc
                           |
                           v
              namespace + cgroup + rootfs
                           |
                           v
                 컨테이너의 PID 1

여러 컨테이너
   |
   | docker compose
   v
Frontend + Backend + Database</code></pre>
<hr />
<blockquote>
<h4 id="2일차의-핵심은-docker-명령을-더-많이-외우는-것이-아니다-이미지-파일이-프로세스로-바뀌는-내부-흐름과-여러-프로세스-그룹을-하나의-서비스로-조율하는-구조를-이해하는-것이다">2일차의 핵심은 Docker 명령을 더 많이 외우는 것이 아니다. <strong>이미지 파일이 프로세스로 바뀌는 내부 흐름과 여러 프로세스 그룹을 하나의 서비스로 조율하는 구조</strong>를 이해하는 것이다.</h4>
</blockquote>
<hr />
<h1 id="1-컨테이너-이미지-구조-이해">1. 컨테이너 이미지 구조 이해</h1>
<h2 id="11-컨테이너-이미지는-무엇인가">1.1 컨테이너 이미지는 무엇인가?</h2>
<p>컨테이너 이미지는 단순한 압축 파일 하나가 아니다. 프로세스를 실행하는 데 필요한 다음 내용을 표준 형식으로 묶은 <strong>배포용 패키지 아카이브</strong>다.</p>
<ul>
<li>컨테이너가 <code>/</code>로 사용할 root filesystem(rootfs)</li>
<li>애플리케이션 실행 파일과 라이브러리</li>
<li>여러 개의 읽기 전용 파일시스템 레이어</li>
<li>실행 명령, 환경변수, 사용자, 포트 등의 메타데이터</li>
<li>레이어의 순서와 무결성을 설명하는 manifest</li>
</ul>
<pre><code class="language-text">Container Image
├── Manifest: 어떤 config와 layer를 조합할 것인가?
├── Config: CMD, ENTRYPOINT, ENV, WORKDIR, USER 등
└── Layers: 실제 파일시스템 변경 내용</code></pre>
<p>이미지는 읽기 전용이며, 여러 이미지가 같은 레이어를 공유할 수 있다. 이 덕분에 저장 공간과 전송량을 줄이고 빠르게 컨테이너를 만들 수 있다.</p>
<h2 id="12-rootfs란">1.2 rootfs란?</h2>
<p>rootfs는 컨테이너 프로세스가 자신의 루트 디렉터리 <code>/</code>라고 인식하는 파일시스템이다.</p>
<pre><code class="language-text">rootfs/
├── bin/
├── etc/
├── lib/
├── usr/
├── var/
└── ...</code></pre>
<p>컨테이너 안에서 <code>ls /</code>를 실행하면 호스트의 실제 <code>/</code> 전체가 아니라, 런타임이 그 컨테이너용으로 준비한 rootfs를 보게 된다.</p>
<h2 id="13-tar로-이미지-구조-감-잡기">1.3 tar로 이미지 구조 감 잡기</h2>
<p><code>tar</code>는 여러 파일과 디렉터리를 하나의 아카이브로 묶거나 다시 푸는 도구다.</p>
<pre><code class="language-bash">cd ~/path/to/skala-container/00.sample-container

tar cvf sample-container.tar .
tar tvf sample-container.tar</code></pre>
<p>자주 쓰는 옵션:</p>
<table>
<thead>
<tr>
<th>옵션</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>c</code></td>
<td>새 아카이브 생성(create)</td>
</tr>
<tr>
<td><code>x</code></td>
<td>아카이브 해제(extract)</td>
</tr>
<tr>
<td><code>t</code></td>
<td>내부 목록 출력(list)</td>
</tr>
<tr>
<td><code>v</code></td>
<td>처리 대상을 자세히 출력(verbose)</td>
</tr>
<tr>
<td><code>f</code></td>
<td>뒤에 오는 값을 파일 이름으로 사용(file)</td>
</tr>
</tbody></table>
<p>Docker 이미지의 레이어도 tar 계열의 아카이브로 보관된다. 다만 실제 이미지는 단순 tar 묶음이 아니라 표준화된 메타데이터와 content digest를 함께 가진다.</p>
<h2 id="14-직접-만든-이미지-분해하기">1.4 직접 만든 이미지 분해하기</h2>
<p>교재 실습 위치:</p>
<pre><code class="language-bash">cd ~/path/to/skala-container/01.answer-code/07.in-images</code></pre>
<p>Dockerfile 확인:</p>
<pre><code class="language-bash">sed -n '1,240p' Dockerfile</code></pre>
<p>이미지 빌드:</p>
<pre><code class="language-bash">docker build -t indepth-container:1.0 .</code></pre>
<p>분석용 디렉터리를 만들고 이미지를 tar 파일로 저장한다.</p>
<pre><code class="language-bash">mkdir -p indepth-container
docker save indepth-container:1.0 \
  -o indepth-container/indepth-container.tar

cd indepth-container
tar tvf indepth-container.tar
tar xvf indepth-container.tar
file *</code></pre>
<blockquote>
<p><code>docker save</code>는 이미지와 메타데이터·레이어를 저장한다. <code>docker export</code>는 실행 중이거나 정지된 컨테이너의 병합된 파일시스템을 내보내므로 목적과 결과가 다르다.</p>
</blockquote>
<h2 id="15-저장된-oci-이미지-구성">1.5 저장된 OCI 이미지 구성</h2>
<p>Docker 버전과 저장 형식에 따라 보이는 파일 이름은 달라질 수 있지만 핵심 개념은 같다.</p>
<pre><code class="language-text">image archive/
├── manifest.json 또는 OCI index/manifest
├── oci-layout
└── blobs/
    └── sha256/
        ├── config JSON
        ├── manifest JSON
        └── layer archives</code></pre>
<p>manifest를 확인한다.</p>
<pre><code class="language-bash">cat manifest.json | jq</code></pre>
<p><code>manifest.json</code>이 없는 OCI layout이면 <code>index.json</code>과 <code>blobs/sha256</code>의 JSON을 확인한다.</p>
<pre><code class="language-bash">cat index.json | jq
find blobs/sha256 -type f -maxdepth 1 -print
file blobs/sha256/*</code></pre>
<h3 id="manifest-config-layer-구분">Manifest, Config, Layer 구분</h3>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>질문</th>
<th>포함 내용</th>
</tr>
</thead>
<tbody><tr>
<td>Manifest</td>
<td>무엇을 조립할까?</td>
<td>config digest, layer digest와 순서</td>
</tr>
<tr>
<td>Config</td>
<td>어떻게 실행할까?</td>
<td>CMD, ENTRYPOINT, ENV, USER, WORKDIR, EXPOSE, history</td>
</tr>
<tr>
<td>Layer</td>
<td>어떤 파일이 바뀌었나?</td>
<td><code>RUN</code>, <code>COPY</code>, <code>ADD</code>로 추가·수정된 파일</td>
</tr>
</tbody></table>
<p>digest는 파일 내용으로 계산한 식별자다. 내용이 달라지면 digest도 달라지므로 무결성 검증과 중복 제거에 사용된다.</p>
<h2 id="16-dockerfile-지시어와-레이어의-관계">1.6 Dockerfile 지시어와 레이어의 관계</h2>
<p>예제 Dockerfile:</p>
<pre><code class="language-dockerfile">ARG UBUNTU_VERSION=22.04
FROM ubuntu:${UBUNTU_VERSION}
RUN apt-get update &amp;&amp; apt-get install -y curl lsb-release nginx
ARG UBUNTU_VERSION
RUN echo &quot;Ubuntu version: ${UBUNTU_VERSION}&quot;
LABEL maintainer=&quot;student@example.com&quot;
EXPOSE 8080/tcp
EXPOSE 80/tcp
WORKDIR /var/www/html
COPY index.html .
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p>모든 지시어가 같은 형태의 파일 레이어를 만드는 것은 아니다.</p>
<table>
<thead>
<tr>
<th>종류</th>
<th>대표 지시어</th>
<th>결과</th>
</tr>
</thead>
<tbody><tr>
<td>파일시스템 변경</td>
<td><code>RUN</code>, <code>COPY</code>, <code>ADD</code></td>
<td>실제 파일이 들어 있는 레이어 생성</td>
</tr>
<tr>
<td>실행 설정·메타데이터</td>
<td><code>CMD</code>, <code>ENTRYPOINT</code>, <code>ENV</code>, <code>USER</code>, <code>WORKDIR</code>, <code>EXPOSE</code>, <code>LABEL</code></td>
<td>image config에 기록</td>
</tr>
<tr>
<td>기반 이미지 선택</td>
<td><code>FROM</code></td>
<td>부모 이미지 레이어를 상속</td>
</tr>
<tr>
<td>빌드 변수</td>
<td><code>ARG</code></td>
<td>빌드 과정에서 사용, 일반적으로 런타임 환경변수로 남지 않음</td>
</tr>
</tbody></table>
<p><code>docker history</code>로 빌드 이력을 쉽게 볼 수 있다.</p>
<pre><code class="language-bash">docker history --no-trunc indepth-container:1.0
docker image inspect indepth-container:1.0 | jq</code></pre>
<p>레이어 파일의 내부 목록은 해당 digest를 찾아 다음처럼 확인한다.</p>
<pre><code class="language-bash">file blobs/sha256/&lt;layer-digest&gt;
tar tvf blobs/sha256/&lt;layer-digest&gt;</code></pre>
<p>교재의 긴 SHA-256 값은 예시 이미지에서 생성된 값이다. 직접 빌드한 환경에서는 digest가 달라질 수 있으므로 그대로 복사하지 말고 manifest에서 자신의 값을 확인한다.</p>
<h2 id="17-mariadb-이미지-분해하기">1.7 MariaDB 이미지 분해하기</h2>
<p>MariaDB처럼 다른 사람이 만든 공식 이미지도 같은 방식으로 분석할 수 있다.</p>
<pre><code class="language-bash">docker pull mariadb:10.11

mkdir -p /tmp/indepth-mariadb
docker save mariadb:10.11 \
  -o /tmp/indepth-mariadb/mariadb.tar

cd /tmp/indepth-mariadb
tar xvf mariadb.tar</code></pre>
<p>manifest와 blob 유형을 확인한다.</p>
<pre><code class="language-bash">if test -f manifest.json; then
  cat manifest.json | jq
else
  cat index.json | jq
fi

file blobs/sha256/*</code></pre>
<p>MariaDB 이미지에서는 다음과 같은 레이어·설정의 흔적을 찾을 수 있다.</p>
<ul>
<li>기반 OS 파일</li>
<li><code>mysql</code> 또는 <code>mariadb</code> 사용자와 그룹 생성</li>
<li>MariaDB 패키지 설치</li>
<li><code>/docker-entrypoint-initdb.d</code> 디렉터리</li>
<li><code>/usr/local/bin/docker-entrypoint.sh</code></li>
<li>실행 환경변수와 volume 설정</li>
</ul>
<p>특정 layer의 내용을 확인한다.</p>
<pre><code class="language-bash">tar tvf blobs/sha256/&lt;layer-digest&gt; | less</code></pre>
<h3 id="1교시-체크포인트">1교시 체크포인트</h3>
<ul>
<li>이미지는 rootfs 레이어와 실행 메타데이터를 묶은 표준 패키지다.</li>
<li>Manifest는 config와 layer의 digest·순서를 연결한다.</li>
<li>Config는 컨테이너 실행 방법을 설명한다.</li>
<li>Layer는 파일시스템 변경 내용을 담는다.</li>
<li><code>docker save</code>와 <code>docker export</code>의 대상과 목적은 다르다.</li>
<li>Dockerfile의 모든 지시어가 파일 레이어를 만드는 것은 아니다.</li>
</ul>
<hr />
<h1 id="2--컨테이너-구조-이해-1">2.  컨테이너 구조 이해 (1)</h1>
<h2 id="21-docker-architecture">2.1 Docker Architecture</h2>
<p>Docker의 기본 구성은 Client, Docker Host, Registry로 나눌 수 있다.</p>
<pre><code class="language-text">Docker Client
  docker build / pull / run
          |
          v
Docker Host
  dockerd
  ├── Images
  └── Containers
          ^
          |
Registry
  Docker Hub / Private Registry</code></pre>
<ul>
<li><strong>Client</strong>: 사용자가 <code>docker</code> 명령을 입력하는 진입점</li>
<li><strong>dockerd</strong>: 이미지·컨테이너·네트워크·볼륨 관리</li>
<li><strong>Registry</strong>: 이미지를 push/pull하는 저장소</li>
<li><strong>containerd와 runc</strong>: 실제 컨테이너 생명주기와 프로세스 실행 담당</li>
</ul>
<h2 id="22-컨테이너는-linux-커널-기능의-조합이다">2.2 컨테이너는 Linux 커널 기능의 조합이다</h2>
<p>컨테이너는 작은 VM이 아니다. 일반 Linux 프로세스에 여러 커널 기능을 조합해 격리된 실행 환경처럼 보이게 만든 것이다.</p>
<table>
<thead>
<tr>
<th>기술</th>
<th>담당 역할</th>
</tr>
</thead>
<tbody><tr>
<td>Namespace</td>
<td>프로세스가 보는 시스템 공간 분리</td>
</tr>
<tr>
<td>cgroup</td>
<td>CPU, 메모리, I/O, PID 수 제한·측정</td>
</tr>
<tr>
<td>rootfs + OverlayFS</td>
<td>컨테이너별 파일시스템 제공</td>
</tr>
<tr>
<td>Capabilities</td>
<td>root 권한을 세부 기능으로 분리</td>
</tr>
<tr>
<td>SELinux/AppArmor</td>
<td>프로세스가 접근할 대상과 행위 통제</td>
</tr>
<tr>
<td>Netfilter</td>
<td>NAT, 포트 포워딩, 패킷 필터링</td>
</tr>
</tbody></table>
<h2 id="23-namespace-보이는-세상을-분리한다">2.3 Namespace: 보이는 세상을 분리한다</h2>
<p>Namespace는 하나의 커널을 공유하면서도 프로세스마다 서로 다른 시스템 view를 제공한다.</p>
<table>
<thead>
<tr>
<th>Namespace</th>
<th>격리 대상</th>
<th>주요 기능·쓰임</th>
<th>컨테이너에서 보이는 결과</th>
</tr>
</thead>
<tbody><tr>
<td>PID</td>
<td>프로세스 번호와 트리</td>
<td>다른 컨테이너와 호스트의 프로세스를 보거나 제어하지 못하게 함</td>
<td>주 프로세스가 PID 1</td>
</tr>
<tr>
<td>NET</td>
<td>NIC, IP, port, routing</td>
<td>컨테이너별 네트워크 장치와 IP를 제공하고 포트 충돌을 분리</td>
<td>독립된 <code>eth0</code>, IP, routing table</td>
</tr>
<tr>
<td>MNT</td>
<td>mount table</td>
<td>컨테이너마다 보이는 파일시스템을 다르게 구성하고 필요한 볼륨만 연결</td>
<td>컨테이너별 rootfs와 mount</td>
</tr>
<tr>
<td>UTS</td>
<td>hostname/domain name</td>
<td>컨테이너마다 독립적인 호스트 이름을 부여</td>
<td>컨테이너별 hostname</td>
</tr>
<tr>
<td>IPC</td>
<td>공유 메모리, queue, semaphore</td>
<td>프로세스 간 통신 자원이 다른 컨테이너와 섞이는 것을 방지</td>
<td>IPC 자원 분리</td>
</tr>
<tr>
<td>USER</td>
<td>UID/GID 매핑</td>
<td>내부 사용자와 호스트 사용자를 다르게 매핑해 권한 피해를 줄임</td>
<td>내부 root를 외부 UID와 다르게 매핑 가능</td>
</tr>
</tbody></table>
<p>비유하면 Namespace는 같은 건물 안에서 각 세입자에게 서로 다른 호수와 출입문, 주소록을 보여주는 기능이다.</p>
<h2 id="24-cgroup-사용할-수-있는-자원을-제한한다">2.4 cgroup: 사용할 수 있는 자원을 제한한다</h2>
<p>Namespace가 “무엇을 볼 수 있는가?”를 정한다면 cgroup은 “얼마나 사용할 수 있는가?”를 정한다.</p>
<pre><code class="language-bash">docker run -d \
  --name resource-demo \
  --cpus=&quot;2.0&quot; \
  --memory=&quot;512m&quot; \
  nginx</code></pre>
<p>주요 제한:</p>
<ul>
<li>CPU quota와 shares</li>
<li>Memory와 swap</li>
<li>Block I/O 읽기·쓰기</li>
<li>생성 가능한 PID 수</li>
<li>장치 접근</li>
</ul>
<p>메모리 제한을 넘으면 Linux OOM Killer가 프로세스를 종료할 수 있다. Docker에서는 흔히 종료 코드 <code>137</code>로 보이며, Kubernetes에서는 <code>OOMKilled</code> 상태로 나타날 수 있다.</p>
<pre><code class="language-text">memory limit 초과
      ↓
Kernel OOM Killer
      ↓
PID 1 종료
      ↓
shim이 종료 감지
      ↓
containerd/Docker 상태 갱신</code></pre>
<h2 id="25-linux-capability와-selinuxapparmor">2.5 Linux Capability와 SELinux/AppArmor</h2>
<p>전통적 권한 모델은 root에게 지나치게 많은 권한을 준다. Capability는 root 권한을 약 40개의 기능으로 분리하여 필요한 것만 줄 수 있게 한다.</p>
<ul>
<li><code>CAP_NET_BIND_SERVICE</code>: 1024 미만 포트 바인딩</li>
<li><code>CAP_NET_ADMIN</code>: 네트워크 인터페이스·라우팅 관리</li>
<li><code>CAP_SYS_ADMIN</code>: mount 등 강력한 시스템 관리 기능</li>
<li><code>CAP_KILL</code>: 다른 프로세스에 signal 전송</li>
</ul>
<pre><code class="language-yaml">securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE</code></pre>
<p>SELinux/AppArmor는 “이 프로세스가 어떤 파일·포트·프로세스에 어떤 행동을 할 수 있는가?”를 정책으로 강제한다. UID 0이거나 Capability가 있어도 보안 정책이 허용하지 않으면 커널이 접근을 막을 수 있다.</p>
<h2 id="26-overlayfs-여러-레이어를-하나의-로-보이게-한다">2.6 OverlayFS: 여러 레이어를 하나의 <code>/</code>로 보이게 한다</h2>
<p>OverlayFS의 네 구성요소를 구분해야 한다.</p>
<table>
<thead>
<tr>
<th>구성</th>
<th>접근</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>LowerDir</td>
<td>Read-only</td>
<td>이미지의 원본 레이어</td>
</tr>
<tr>
<td>UpperDir</td>
<td>Read-write</td>
<td>해당 컨테이너의 변경 내용</td>
</tr>
<tr>
<td>WorkDir</td>
<td>내부 작업 공간</td>
<td>rename·copy-up 등 OverlayFS 작업 지원</td>
</tr>
<tr>
<td>MergedDir</td>
<td>통합 view</td>
<td>컨테이너가 rootfs <code>/</code>로 보는 결과</td>
</tr>
</tbody></table>
<pre><code class="language-text">MergedDir (/)
     ↑ overlay
┌───────────────┐
│ UpperDir (RW) │  컨테이너의 생성·수정·삭제
├───────────────┤
│ LowerDir N    │  이미지 레이어 (RO)
│ LowerDir N-1  │
│ LowerDir 1    │
└───────────────┘</code></pre>
<h3 id="copy-on-write">Copy-on-Write</h3>
<p>LowerDir의 파일을 수정하면 원본을 직접 바꾸지 않는다.</p>
<ol>
<li>원본 파일을 UpperDir로 복사한다.</li>
<li>UpperDir의 복사본을 수정한다.</li>
<li>MergedDir에서는 UpperDir의 파일이 우선 보인다.</li>
</ol>
<p>따라서 이미지의 불변성을 지키면서 컨테이너마다 독립적인 변경사항을 가질 수 있다.</p>
<h2 id="27-overlayfs-직접-실습">2.7 OverlayFS 직접 실습</h2>
<blockquote>
<p>주의: <code>--privileged</code>는 컨테이너에 매우 강한 호스트 권한을 준다. 개인 로컬 실습 환경에서만 사용하고, 출처를 모르는 이미지에는 절대 적용하지 않는다.</p>
</blockquote>
<p>Ubuntu 실습 컨테이너 실행:</p>
<pre><code class="language-bash">docker run --rm -it \
  --privileged \
  ubuntu:24.04 \
  /bin/bash</code></pre>
<p>이후 명령은 컨테이너 내부에서 실행한다.</p>
<pre><code class="language-bash">mkdir -p /mnt/ovtest
mount -t tmpfs tmpfs /mnt/ovtest
df -T /mnt/ovtest

mkdir -p /mnt/ovtest/{lower,upper,work,merged}

echo &quot;AAA from lower&quot; &gt; /mnt/ovtest/lower/a.txt
echo &quot;BBB from lower&quot; &gt; /mnt/ovtest/lower/b.txt

mount -t overlay overlay \
  -o lowerdir=/mnt/ovtest/lower,upperdir=/mnt/ovtest/upper,workdir=/mnt/ovtest/work \
  /mnt/ovtest/merged</code></pre>
<p>병합 결과를 확인한다.</p>
<pre><code class="language-bash">mountpoint /mnt/ovtest/merged
ls -l /mnt/ovtest/lower
ls -l /mnt/ovtest/merged</code></pre>
<p>MergedDir의 파일을 수정한다.</p>
<pre><code class="language-bash">echo &quot;add line test&quot; &gt;&gt; /mnt/ovtest/merged/a.txt

cat /mnt/ovtest/lower/a.txt
cat /mnt/ovtest/upper/a.txt
cat /mnt/ovtest/merged/a.txt</code></pre>
<p>확인 결과:</p>
<ul>
<li>LowerDir의 원본은 그대로다.</li>
<li>UpperDir에 수정된 <code>a.txt</code>가 생성된다.</li>
<li>MergedDir에서는 UpperDir의 수정본이 보인다.</li>
</ul>
<p>컨테이너를 나가면 <code>--rm</code> 때문에 실습 컨테이너가 자동 삭제된다.</p>
<pre><code class="language-bash">exit</code></pre>
<h2 id="28-dockerd-containerd-shim-runc">2.8 dockerd, containerd, shim, runc</h2>
<p><code>docker run</code> 한 줄 뒤에서는 여러 구성요소가 역할을 나눈다.</p>
<pre><code class="language-text">docker CLI
   ↓
dockerd
   ↓
containerd
   ↓
containerd-shim
   ↓
runc
   ↓
container init process (PID 1)</code></pre>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>dockerd</td>
<td>Docker API 진입점, 이미지·네트워크·볼륨 등 전체 관리</td>
</tr>
<tr>
<td>containerd</td>
<td>이미지 준비, rootfs snapshot, OCI bundle, 생명주기 관리</td>
</tr>
<tr>
<td>containerd-shim</td>
<td>컨테이너 PID 감시, 표준 I/O·종료 상태 연결</td>
</tr>
<tr>
<td>runc</td>
<td>Namespace·cgroup·rootfs·Capability를 설정하고 PID 1 실행</td>
</tr>
</tbody></table>
<p>runc는 과거 libcontainer 계열의 기술을 바탕으로 만들어진 OCI Runtime Spec의 대표 구현체다.</p>
<h2 id="29-oci-세-가지-표준">2.9 OCI 세 가지 표준</h2>
<p>OCI(Open Container Initiative)는 컨테이너 생태계의 호환성을 위한 표준을 정의한다.</p>
<table>
<thead>
<tr>
<th>표준</th>
<th>정의</th>
</tr>
</thead>
<tbody><tr>
<td>OCI Image Spec</td>
<td>이미지 manifest, config, filesystem layer 구조</td>
</tr>
<tr>
<td>OCI Runtime Spec</td>
<td><code>config.json + rootfs</code>로 프로세스를 실행하는 방법</td>
</tr>
<tr>
<td>OCI Distribution Spec</td>
<td>Registry에서 manifest와 blob을 push/pull하는 API</td>
</tr>
</tbody></table>
<p>OCI Bundle:</p>
<pre><code class="language-text">mybundle/
├── config.json  # 어떻게 실행할 것인가
└── rootfs/      # 무엇을 실행할 것인가</code></pre>
<p>공통 표준 덕분에 BuildKit·Buildah·Kaniko가 만든 이미지를 Docker Hub·Harbor에 저장하고, Docker·containerd·CRI-O 같은 런타임에서 실행할 수 있다.</p>
<h3 id="2교시-체크포인트">2교시 체크포인트</h3>
<ul>
<li>컨테이너는 Namespace와 cgroup으로 격리·제한된 프로세스다.</li>
<li>OverlayFS는 이미지 레이어와 컨테이너 변경사항을 하나의 rootfs로 합친다.</li>
<li>runc는 OCI bundle을 읽고 실제 PID 1을 실행한다.</li>
<li>dockerd, containerd, shim, runc는 서로 다른 책임을 가진다.</li>
<li>OCI는 이미지·실행·배포 형식을 표준화한다.</li>
</ul>
<hr />
<h1 id="3컨테이너-구조-이해-2">3.컨테이너 구조 이해 (2)</h1>
<h2 id="31-docker-네트워크-유형">3.1 Docker 네트워크 유형</h2>
<table>
<thead>
<tr>
<th>유형</th>
<th>격리</th>
<th>내장 DNS</th>
<th>NAT</th>
<th>주 용도</th>
</tr>
</thead>
<tbody><tr>
<td>기본 bridge</td>
<td>있음</td>
<td>이름 해석 제약</td>
<td>있음</td>
<td>간단한 단일 컨테이너 테스트</td>
</tr>
<tr>
<td>Custom bridge</td>
<td>있음</td>
<td>서비스·컨테이너 이름 해석</td>
<td>있음</td>
<td>여러 컨테이너 애플리케이션</td>
</tr>
<tr>
<td>host</td>
<td>없음</td>
<td>별도 Docker DNS 없음</td>
<td>없음</td>
<td>격리보다 네트워크 성능이 중요한 특수 상황</td>
</tr>
<tr>
<td>none</td>
<td>네트워크 없음</td>
<td>없음</td>
<td>없음</td>
<td>완전한 네트워크 차단 실험</td>
</tr>
</tbody></table>
<p>Custom bridge 생성:</p>
<pre><code class="language-bash">docker network create my-bridge
docker run --network my-bridge nginx</code></pre>
<p>실무에서는 기본 bridge보다 Custom bridge를 권장한다. 같은 네트워크의 컨테이너끼리 이름으로 통신할 수 있기 때문이다.</p>
<h2 id="32-bridge-veth-eth0">3.2 bridge, veth, eth0</h2>
<p>Bridge 네트워크의 기본 연결은 다음과 같다.</p>
<pre><code class="language-text">Container eth0
      |
   veth pair
      |
docker0 또는 custom bridge
      |
Host routing + Netfilter/NAT
      |
Host physical NIC
      |
External network</code></pre>
<ul>
<li>컨테이너마다 Network Namespace와 가상 NIC <code>eth0</code>가 생긴다.</li>
<li>호스트 쪽에는 짝이 되는 veth 인터페이스가 생긴다.</li>
<li>Linux bridge가 veth들을 L2 switch처럼 연결한다.</li>
<li>외부 통신과 <code>-p</code> 포트 공개는 host routing과 Netfilter/NAT 규칙을 사용한다.</li>
</ul>
<h2 id="33--p-88888080-패킷-전달-흐름">3.3 <code>-p 8888:8080</code> 패킷 전달 흐름</h2>
<pre><code class="language-text">Client → HostIP:8888
            ↓
Netfilter DNAT
            ↓
172.17.0.3:8080
            ↓
Routing table에서 docker bridge 결정
            ↓
Bridge FDB에서 목적지 MAC의 veth 검색
            ↓
Container eth0 → application:8080</code></pre>
<ul>
<li><strong>DNAT</strong>: 목적지 IP와 port를 컨테이너 주소로 변경</li>
<li><strong>ARP/Neighbor table</strong>: IP와 MAC 주소 대응</li>
<li><strong>FDB</strong>: MAC 주소와 bridge port(veth) 대응</li>
<li><strong>NAPT</strong>: IP 주소뿐 아니라 port까지 함께 변환</li>
</ul>
<h2 id="34-dhcp-cidr-calico-참고-개념">3.4 DHCP, CIDR, Calico 참고 개념</h2>
<h3 id="dhcp">DHCP</h3>
<p>DHCP는 IP 주소, subnet mask, gateway, DNS 서버 정보를 자동 할당하는 프로토콜이다. 가정용 공유기가 노트북과 휴대전화에 사설 IP를 나눠 주는 것이 대표적인 예다.</p>
<h3 id="cidr">CIDR</h3>
<p><code>192.168.0.0/24</code>에서 <code>/24</code>는 앞의 24비트가 network 영역이라는 뜻이다.</p>
<pre><code class="language-text">192.168.0.0/24
├── network address: 192.168.0.0
├── host range: 일반적으로 192.168.0.1 ~ 192.168.0.254
└── broadcast: 192.168.0.255</code></pre>
<h3 id="docker-network와-kubernetes-cni">Docker Network와 Kubernetes CNI</h3>
<p>Docker bridge는 기본적으로 단일 노드 컨테이너 통신을 다룬다. Kubernetes는 여러 노드의 Pod 통신을 위해 Calico, Amazon VPC CNI 같은 CNI 플러그인을 사용한다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>Docker bridge</th>
<th>Calico 등의 CNI</th>
</tr>
</thead>
<tbody><tr>
<td>대상</td>
<td>Docker 컨테이너</td>
<td>Kubernetes Pod</td>
</tr>
<tr>
<td>범위</td>
<td>단일 노드 중심</td>
<td>멀티 노드</td>
</tr>
<tr>
<td>이름 해석</td>
<td>Docker DNS</td>
<td>CoreDNS</td>
</tr>
<tr>
<td>보안 정책</td>
<td>제한적</td>
<td>NetworkPolicy</td>
</tr>
</tbody></table>
<p>Calico는 환경에 따라 routing, BGP, IP-in-IP tunnel 등을 사용해 다른 노드의 Pod CIDR로 패킷을 전달한다.</p>
<h2 id="35-unix-domain-socket">3.5 Unix Domain Socket</h2>
<p>UDS는 IP와 port 대신 파일 경로를 주소로 사용하는 같은 호스트 내부 IPC 방식이다.</p>
<pre><code class="language-text">TCP socket:         127.0.0.1:8080
Unix domain socket: /var/run/docker.sock</code></pre>
<p>특징:</p>
<ul>
<li>IP stack과 routing을 거치지 않아 로컬 통신에 효율적</li>
<li>socket file의 소유권과 permission으로 접근 통제</li>
<li>외부 네트워크에서 직접 접근 불가</li>
</ul>
<p>대표 경로:</p>
<ul>
<li>Docker: <code>/var/run/docker.sock</code></li>
<li>containerd: <code>/run/containerd/containerd.sock</code></li>
<li>CRI-O: <code>/run/crio/crio.sock</code></li>
<li>MySQL: <code>/var/run/mysqld/mysqld.sock</code></li>
</ul>
<blockquote>
<p>Docker socket을 컨테이너에 마운트하면 그 컨테이너가 사실상 호스트 Docker를 제어할 수 있다. 신뢰할 수 있는 실습 이미지에서만 사용해야 한다.</p>
</blockquote>
<h2 id="36-runc-실습의-목표">3.6 runc 실습의 목표</h2>
<p>Docker가 자동으로 수행하는 다음 과정을 직접 확인한다.</p>
<ol>
<li>이미지에서 rootfs 준비</li>
<li>OCI <code>config.json</code> 생성</li>
<li>Namespace와 cgroup 설정</li>
<li>rootfs를 컨테이너 <code>/</code>로 전환</li>
<li>PID 1 실행</li>
</ol>
<p>현재 저장소의 실습 위치:</p>
<pre><code class="language-bash">cd ~/path/to/skala-container/01.answer-code/08.runc</code></pre>
<blockquote>
<p>교재 일부 페이지에는 <code>01.answer-code/runc</code>로 표시되지만 현재 저장소 경로는 <code>01.answer-code/08.runc</code>다.</p>
</blockquote>
<h2 id="37-runc-실습-환경-준비">3.7 runc 실습 환경 준비</h2>
<p>파일 확인:</p>
<pre><code class="language-bash">ls -l
sed -n '1,260p' Dockerfile
sed -n '1,240p' build.sh
sed -n '1,240p' run.sh</code></pre>
<p>실행 권한이 없다면:</p>
<pre><code class="language-bash">chmod +x build.sh run.sh</code></pre>
<p>빌드와 실행:</p>
<pre><code class="language-bash">./build.sh
./run.sh
docker ps --filter name=runc-test
docker exec -it runc-test /bin/bash</code></pre>
<blockquote>
<p>이 실습은 <code>--privileged</code>와 Docker socket mount를 사용한다. 개인 학습용 로컬 환경에서만 실행하고, 실습 후 컨테이너를 제거한다.</p>
</blockquote>
<p>컨테이너 내부에서 확인:</p>
<pre><code class="language-bash">runc --version</code></pre>
<h2 id="38-oci-bundle-만들기">3.8 OCI Bundle 만들기</h2>
<p>다음은 <code>runc-test</code> 컨테이너 내부에서 실행한다.</p>
<pre><code class="language-bash">cd
mkdir -p mybundle/rootfs
cd mybundle</code></pre>
<p>Alpine 컨테이너 파일시스템을 rootfs로 추출한다.</p>
<pre><code class="language-bash">alpine_id=$(docker create alpine:latest)
docker export &quot;$alpine_id&quot; | tar -C rootfs -xf -
docker rm &quot;$alpine_id&quot;

ls -al rootfs</code></pre>
<p>OCI 기본 설정 생성:</p>
<pre><code class="language-bash">runc spec
ls -l</code></pre>
<p>결과:</p>
<pre><code class="language-text">mybundle/
├── config.json
└── rootfs/</code></pre>
<h2 id="39-runc로-직접-컨테이너-실행">3.9 runc로 직접 컨테이너 실행</h2>
<p><code>config.json</code>과 <code>rootfs</code>가 있는 bundle 디렉터리에서 실행한다.</p>
<pre><code class="language-bash">runc run mycontainer</code></pre>
<p>다른 터미널에서 <code>runc-test</code>에 다시 들어가 상태를 확인한다.</p>
<pre><code class="language-bash">docker exec -it runc-test /bin/bash
cd ~/mybundle
runc list</code></pre>
<p><code>runc run</code>으로 열린 컨테이너 안에서는 다음을 확인한다.</p>
<pre><code class="language-bash">ps -ef
ip addr 2&gt;/dev/null || ifconfig
exit</code></pre>
<h2 id="310-runc-실습-2-bind-mount">3.10 runc 실습 2: bind mount</h2>
<p>먼저 실행 중인 <code>mycontainer</code>가 없는지 확인한다.</p>
<pre><code class="language-bash">runc list</code></pre>
<p><code>runc-test</code> 내부에 공유할 파일을 만든다.</p>
<pre><code class="language-bash">mkdir -p /mydata
echo &quot;Hello from host&quot; &gt; /mydata/test.txt
cd ~/mybundle</code></pre>
<p><code>config.json</code>에 mount를 추가한다.</p>
<pre><code class="language-bash">jq '.mounts += [{
  &quot;destination&quot;: &quot;/data&quot;,
  &quot;type&quot;: &quot;bind&quot;,
  &quot;source&quot;: &quot;/mydata&quot;,
  &quot;options&quot;: [&quot;rbind&quot;, &quot;rw&quot;]
}]' config.json &gt; config_volume.json

jq '.mounts[] | select(.destination == &quot;/data&quot;)' config_volume.json
cp config_volume.json config.json</code></pre>
<p>실행 후 확인:</p>
<pre><code class="language-bash">runc run mycontainer</code></pre>
<p>runc 컨테이너 안에서:</p>
<pre><code class="language-bash">cd /data
ls -l
cat test.txt
exit</code></pre>
<h2 id="311-runc-실습-3-pid-namespace-제거">3.11 runc 실습 3: PID Namespace 제거</h2>
<p>PID Namespace 설정을 제거하면 runc 컨테이너가 바깥 <code>runc-test</code> 환경의 프로세스 목록까지 볼 수 있다.</p>
<p>수동 편집 대신 백업 후 <code>jq</code>로 변경한다.</p>
<pre><code class="language-bash">cd ~/mybundle
cp config.json config.before-pid-share.json

jq 'del(.linux.namespaces[] | select(.type == &quot;pid&quot;))' \
  config.json &gt; config.pid-share.json

cp config.pid-share.json config.json
runc run mycontainer</code></pre>
<p>runc 컨테이너 안에서:</p>
<pre><code class="language-bash">ps -ef
exit</code></pre>
<p>격리가 사라지면 바깥쪽 프로세스들이 보인다. 이 실습은 Namespace가 “프로세스가 보는 view”를 만든다는 사실을 보여준다.</p>
<h2 id="312-runc-실습-4-network-namespace-제거">3.12 runc 실습 4: Network Namespace 제거</h2>
<p>Network Namespace 설정을 제거하면 runc 컨테이너가 <code>runc-test</code>의 network stack을 공유한다.</p>
<pre><code class="language-bash">cd ~/mybundle
cp config.json config.before-network-share.json

jq 'del(.linux.namespaces[] | select(.type == &quot;network&quot;))' \
  config.json &gt; config.network-share.json

cp config.network-share.json config.json
runc run mycontainer</code></pre>
<p>runc 컨테이너 안에서:</p>
<pre><code class="language-bash">ip addr 2&gt;/dev/null || ifconfig
exit</code></pre>
<p>바깥 <code>runc-test</code>와 같은 인터페이스와 IP가 보이는지 비교한다.</p>
<h2 id="313-runc-실습-5-직접-pid-1-실행">3.13 runc 실습 5: 직접 PID 1 실행</h2>
<p>rootfs 안에 init script를 만든다.</p>
<pre><code class="language-bash">cd ~/mybundle

cat &gt; rootfs/init.sh &lt;&lt;'EOF'
#!/bin/sh
echo &quot;[init] hello from runc&quot;
date
sleep 1
echo &quot;[init] done&quot;
sleep 3600
EOF

chmod +x rootfs/init.sh</code></pre>
<p><code>config.json</code>의 process를 다음 핵심 값으로 수정한다.</p>
<pre><code class="language-json">{
  &quot;terminal&quot;: false,
  &quot;user&quot;: {
    &quot;uid&quot;: 0,
    &quot;gid&quot;: 0
  },
  &quot;args&quot;: [&quot;sh&quot;, &quot;/init.sh&quot;]
}</code></pre>
<p>실행 및 확인:</p>
<pre><code class="language-bash">runc run -d mycontainer
runc list
runc state mycontainer
runc exec -t mycontainer /bin/sh</code></pre>
<p>runc 컨테이너 안에서:</p>
<pre><code class="language-bash">ps -ef
exit</code></pre>
<p>정리:</p>
<pre><code class="language-bash">runc kill mycontainer KILL
runc delete mycontainer
runc list</code></pre>
<p>호스트로 나온 뒤 실습용 Docker 컨테이너도 정리한다.</p>
<pre><code class="language-bash">exit
docker stop runc-test
docker rm runc-test</code></pre>
<h2 id="314-그래서-컨테이너란-무엇인가">3.14 그래서 컨테이너란 무엇인가?</h2>
<p>컨테이너는 별도의 물리적 실행 개체가 아니다.</p>
<blockquote>
<p>컨테이너는 런타임이 논리적으로 관리하는 프로세스 그룹이며, Linux Namespace로 view가 격리되고 cgroup으로 자원이 제한되며 별도 rootfs를 사용하는 상태다.</p>
</blockquote>
<p>컨테이너 안의 프로세스도 결국 호스트 Linux 커널이 스케줄링하는 일반 프로세스다. Docker와 containerd가 이미지, 설정, PID, 로그, 네트워크를 묶어 “컨테이너”라는 단위로 추상화하여 관리한다.</p>
<h3 id="오전-체크포인트">오전 체크포인트</h3>
<ul>
<li>Custom bridge에서 컨테이너 이름으로 통신할 수 있다.</li>
<li><code>-p</code>는 host Netfilter/NAT를 통해 port를 전달한다.</li>
<li>UDS는 파일 경로를 사용하는 로컬 IPC다.</li>
<li>OCI Bundle은 <code>config.json + rootfs</code>다.</li>
<li>runc는 bundle을 읽어 Namespace·cgroup을 구성하고 PID 1을 실행한다.</li>
<li>컨테이너는 본질적으로 격리·제한된 프로세스의 모음이다.</li>
</ul>
<hr />
<h1 id="4-docker-compose-1">4. Docker Compose (1)</h1>
<h2 id="41-docker-compose가-필요한-이유">4.1 Docker Compose가 필요한 이유</h2>
<p>컨테이너가 하나일 때는 <code>docker run</code> 한 줄로 충분하다. Frontend, Backend, DB가 생기면 다음 설정을 반복해야 한다.</p>
<ul>
<li>이미지 또는 build context</li>
<li>port binding</li>
<li>환경변수</li>
<li>volume</li>
<li>network</li>
<li>restart 정책</li>
<li>시작 순서와 healthcheck</li>
</ul>
<p>Docker Compose는 이 구성을 YAML 파일 하나에 선언하고 애플리케이션 전체를 한 번에 관리한다.</p>
<pre><code class="language-text">docker-compose.yaml
├── db service
├── backend service
├── frontend service
├── volumes
└── networks</code></pre>
<p>Compose는 주로 단일 Docker host의 로컬 개발·교육·통합 테스트 환경에 적합하다. 여러 서버에 걸친 운영 orchestration은 Kubernetes 같은 별도 플랫폼의 영역이다.</p>
<h2 id="42-기본-yaml-구조">4.2 기본 YAML 구조</h2>
<pre><code class="language-yaml">services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: postgres
    volumes:
      - ./db_data:/var/lib/postgresql/data

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.backend
    depends_on:
      - db

  frontend:
    image: nginx:alpine
    ports:
      - &quot;8080:80&quot;</code></pre>
<ul>
<li><code>services</code>: 컨테이너 실행 단위</li>
<li>service 이름: Compose 내부 DNS 이름</li>
<li><code>image</code>: 기존 이미지 사용</li>
<li><code>build</code>: Dockerfile로 이미지 빌드</li>
<li><code>ports</code>: host와 container port 연결</li>
<li><code>volumes</code>: 데이터·설정 파일 mount</li>
<li><code>environment</code>: 환경변수 전달</li>
<li><code>depends_on</code>: 시작 의존성 표현</li>
</ul>
<p>YAML은 들여쓰기가 문법이다. Tab 대신 space를 사용하고 다음 명령으로 먼저 검증한다.</p>
<pre><code class="language-bash">docker compose config</code></pre>
<h2 id="43-현재-저장소의-compose-실습-경로">4.3 현재 저장소의 Compose 실습 경로</h2>
<p>교재 PDF에는 <code>10.docker-compose</code>로 표기된 페이지가 있지만 현재 저장소에는 다음 경로로 존재한다.</p>
<pre><code class="language-text">01.answer-code/09.docker-compose/
├── 01.start/
└── 02.services/</code></pre>
<p>첫 실습:</p>
<pre><code class="language-bash">cd ~/path/to/skala-container/01.answer-code/09.docker-compose/01.start
sed -n '1,240p' docker-compose.yaml
docker compose config</code></pre>
<p>실행:</p>
<pre><code class="language-bash">docker compose up -d
docker compose ps</code></pre>
<p>종료:</p>
<pre><code class="language-bash">docker compose down</code></pre>
<h2 id="44-build와-up">4.4 build와 up</h2>
<pre><code class="language-bash"># 이미지 빌드만
docker compose build

# 캐시 없이 빌드
docker compose build --no-cache

# 특정 서비스만 빌드
docker compose build backend

# foreground 실행
docker compose up

# background 실행
docker compose up -d

# 빌드 후 background 실행
docker compose up --build -d

# 일부 서비스만 실행
docker compose up -d db backend</code></pre>
<p>Foreground 실행 중에는 <code>Control + C</code>로 중지할 수 있다. Background 실행은 <code>docker compose stop</code> 또는 <code>down</code>으로 관리한다.</p>
<h2 id="45-stop-start-restart">4.5 stop, start, restart</h2>
<pre><code class="language-bash">docker compose stop
docker compose start
docker compose restart
docker compose restart backend</code></pre>
<ul>
<li><code>stop</code>: 컨테이너를 보존한 채 중지</li>
<li><code>start</code>: 기존 컨테이너 재시작</li>
<li><code>restart</code>: stop 후 start</li>
<li>설정 파일 변경 반영에는 단순 restart보다 recreate가 필요할 수 있음</li>
</ul>
<h2 id="46-상태와-로그">4.6 상태와 로그</h2>
<pre><code class="language-bash">docker compose ps
docker compose ps -a

docker compose logs
docker compose logs -f
docker compose logs -f backend</code></pre>
<p><code>logs -f</code>에서 빠져나갈 때 <code>Control + C</code>를 눌러도 서비스는 계속 실행된다.</p>
<p>실제 컨테이너 메타데이터:</p>
<pre><code class="language-bash">docker compose ps -q backend
docker inspect &quot;$(docker compose ps -q backend)&quot; | less</code></pre>
<p>확인할 내용:</p>
<ul>
<li>State와 exit code</li>
<li>CMD와 ENTRYPOINT</li>
<li>network와 IP</li>
<li>mount</li>
<li>환경변수</li>
<li>restart policy</li>
<li>healthcheck 결과</li>
</ul>
<h2 id="47-rm과-down-구분">4.7 rm과 down 구분</h2>
<pre><code class="language-bash">docker compose rm
docker compose down
docker compose down -v</code></pre>
<table>
<thead>
<tr>
<th>명령</th>
<th>컨테이너</th>
<th>Compose network</th>
<th>Named volume</th>
</tr>
</thead>
<tbody><tr>
<td><code>rm</code></td>
<td>중지 컨테이너 제거</td>
<td>유지</td>
<td>유지</td>
</tr>
<tr>
<td><code>down</code></td>
<td>제거</td>
<td>제거</td>
<td>기본적으로 유지</td>
</tr>
<tr>
<td><code>down -v</code></td>
<td>제거</td>
<td>제거</td>
<td>제거</td>
</tr>
</tbody></table>
<blockquote>
<p><code>down -v</code>는 DB의 named volume까지 삭제할 수 있다. 데이터가 필요하면 실행하지 않는다. Bind mount의 호스트 디렉터리는 <code>down -v</code>가 직접 삭제하지 않는다.</p>
</blockquote>
<h2 id="48-설정-변경과-장애-복구">4.8 설정 변경과 장애 복구</h2>
<p>특정 서비스를 강제로 재생성한다.</p>
<pre><code class="language-bash">docker compose up -d --force-recreate backend</code></pre>
<p>전체를 다시 빌드·생성한다.</p>
<pre><code class="language-bash">docker compose down
docker compose up --build -d</code></pre>
<p>문제 해결 순서:</p>
<pre><code class="language-text">docker compose config
        ↓
docker compose ps -a
        ↓
docker compose logs 서비스명
        ↓
docker inspect 컨테이너
        ↓
network / volume / environment 확인</code></pre>
<h2 id="49-compose-서비스-디버깅">4.9 Compose 서비스 디버깅</h2>
<pre><code class="language-bash">docker compose exec backend sh</code></pre>
<p>Ubuntu/Debian 이미지에 Bash가 있으면:</p>
<pre><code class="language-bash">docker compose exec backend bash</code></pre>
<p>내부 DNS와 네트워크 확인:</p>
<pre><code class="language-bash">docker compose exec backend cat /etc/resolv.conf
docker compose exec backend getent hosts db
docker compose exec backend ping -c 3 db</code></pre>
<p>이미지에 <code>ping</code>이나 <code>getent</code>가 없을 수 있다. 이 경우 애플리케이션 로그나 해당 이미지에 포함된 <code>wget</code>, <code>curl</code>, DB client를 사용한다.</p>
<h2 id="410-compose-network-확인">4.10 Compose network 확인</h2>
<pre><code class="language-bash">docker network ls
docker network inspect &lt;compose-network-name&gt;</code></pre>
<p>Compose는 기본적으로 <code>&lt;project&gt;_default</code> 형태의 Custom bridge를 만든다. 어떤 service가 연결됐는지 <code>docker network inspect</code>에서 확인할 수 있다.</p>
<p>사용하지 않는 network만 정리:</p>
<pre><code class="language-bash">docker network prune</code></pre>
<p>다른 프로젝트 network도 대상이 될 수 있으므로 목록과 삭제 확인 질문을 읽고 진행한다.</p>
<h2 id="411-특정-compose-파일과-project-지정">4.11 특정 Compose 파일과 project 지정</h2>
<pre><code class="language-bash">docker compose -f docker-compose-app1.yaml ps
docker compose -f docker-compose-app2.yaml logs -f

docker compose -p app1 up -d
docker compose -p app2 up -d</code></pre>
<ul>
<li><code>-f</code>: 사용할 Compose 파일 지정</li>
<li><code>-p</code>: project 이름 지정</li>
<li>기본 project 이름: 일반적으로 Compose 파일이 있는 디렉터리 이름</li>
</ul>
<h2 id="412-frontend와-nginx-reverse-proxy-확장">4.12 Frontend와 Nginx reverse proxy 확장</h2>
<p>Frontend service 예시:</p>
<pre><code class="language-yaml">services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.frontend
    ports:
      - &quot;8080:80&quot;
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro</code></pre>
<p><code>nginx.conf</code>:</p>
<pre><code class="language-nginx">server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /api/ {
        proxy_pass http://backend:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}</code></pre>
<ul>
<li><code>/</code>: 정적 파일 제공</li>
<li><code>/api/</code>: Backend service로 전달</li>
<li><code>$host</code>: 요청받은 host header</li>
<li><code>$remote_addr</code>: client IP</li>
<li><code>:ro</code>: 설정 파일을 read-only로 mount</li>
</ul>
<p>실행:</p>
<pre><code class="language-bash">docker compose up --build -d
docker compose ps</code></pre>
<h3 id="4교시-체크포인트">4교시 체크포인트</h3>
<ul>
<li>Compose의 service는 컨테이너 실행·통신 단위다.</li>
<li><code>up</code>은 생성·실행, <code>down</code>은 컨테이너와 network 제거다.</li>
<li>YAML 수정 후에는 <code>docker compose config</code>로 검증한다.</li>
<li><code>down -v</code>는 DB 데이터를 삭제할 수 있다.</li>
<li>같은 Compose network에서는 service 이름으로 통신한다.</li>
</ul>
<hr />
<h1 id="5--docker-compose-2--컨테이너-관련-도구들">5.  Docker Compose (2) &amp; 컨테이너 관련 도구들</h1>
<h2 id="51-compose의-command와-entrypoint">5.1 Compose의 <code>command</code>와 <code>entrypoint</code></h2>
<p><code>command</code>는 Dockerfile의 CMD를 대체한다.</p>
<pre><code class="language-dockerfile">FROM nginx
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon on;&quot;]</code></pre>
<pre><code class="language-yaml">services:
  web:
    image: nginx
    command: [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p><code>entrypoint</code>까지 지정하면 Dockerfile의 ENTRYPOINT도 대체한다.</p>
<pre><code class="language-yaml">services:
  web:
    image: nginx
    entrypoint: [&quot;/docker-entrypoint.sh&quot;]
    command: [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p>원칙:</p>
<ul>
<li>이미지의 기본 실행 방식이 적절하면 override하지 않는다.</li>
<li>환경별 인자만 달라질 때 <code>command</code>를 사용한다.</li>
<li><code>entrypoint</code> 변경은 이미지의 본질적 실행 흐름까지 바꾸므로 신중하게 사용한다.</li>
</ul>
<h2 id="52-compose에서-이미지-빌드">5.2 Compose에서 이미지 빌드</h2>
<pre><code class="language-yaml">services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile.prod
      args:
        APP_ENV: prod
    ports:
      - &quot;8080:8080&quot;</code></pre>
<ul>
<li><code>context</code>: Docker build context</li>
<li><code>dockerfile</code>: context 안에서 사용할 Dockerfile</li>
<li><code>args</code>: Dockerfile의 ARG에 전달할 빌드 변수</li>
</ul>
<pre><code class="language-bash">docker compose build
docker compose up --build -d</code></pre>
<h2 id="53-restart-policy">5.3 Restart policy</h2>
<table>
<thead>
<tr>
<th>값</th>
<th>동작</th>
<th>적합한 예</th>
</tr>
</thead>
<tbody><tr>
<td><code>no</code></td>
<td>자동 재시작 안 함</td>
<td>migration, 일회성 batch</td>
</tr>
<tr>
<td><code>on-failure</code></td>
<td>비정상 종료 시 재시작</td>
<td>실패 시 재시도가 필요한 작업</td>
</tr>
<tr>
<td><code>always</code></td>
<td>daemon·서버 재시작 후에도 항상 재시작</td>
<td>상시 서비스</td>
</tr>
<tr>
<td><code>unless-stopped</code></td>
<td>사용자가 명시적으로 멈춘 경우 제외하고 재시작</td>
<td>일반적인 상시 서비스</td>
</tr>
</tbody></table>
<pre><code class="language-yaml">services:
  backend:
    image: my-backend
    restart: unless-stopped</code></pre>
<p>Restart policy는 애플리케이션의 PID 1이 종료된 뒤 어떻게 할지를 정한다. 애플리케이션 자체의 graceful shutdown과는 다른 개념이다.</p>
<h2 id="54-healthcheck">5.4 Healthcheck</h2>
<p>Healthcheck는 “프로세스가 떠 있는가?”가 아니라 “서비스가 실제 요청을 처리할 준비가 됐는가?”를 확인한다.</p>
<pre><code class="language-yaml">services:
  backend:
    image: my-backend
    healthcheck:
      test: [&quot;CMD&quot;, &quot;wget&quot;, &quot;-qO-&quot;, &quot;http://localhost:8080/health&quot;]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s</code></pre>
<ul>
<li><code>test</code>: 검사 명령</li>
<li><code>interval</code>: 검사 주기</li>
<li><code>timeout</code>: 한 번의 검사 제한 시간</li>
<li><code>retries</code>: 연속 실패 허용 횟수</li>
<li><code>start_period</code>: 초기 기동 유예 시간</li>
</ul>
<p>Docker healthcheck는 상태를 <code>healthy</code> 또는 <code>unhealthy</code>로 표시하지만, 그 자체가 컨테이너를 자동 교체하거나 복구하지는 않는다. Restart policy와 orchestration의 복구 기능은 별개다.</p>
<h2 id="55-환경변수-주입-방법">5.5 환경변수 주입 방법</h2>
<h3 id="map-형태">Map 형태</h3>
<pre><code class="language-yaml">services:
  web:
    image: nginx
    environment:
      APP_ENV: production
      PORT: &quot;8080&quot;</code></pre>
<h3 id="list-형태">List 형태</h3>
<pre><code class="language-yaml">environment:
  - APP_ENV=production
  - PORT=8080</code></pre>
<h3 id="env_file"><code>env_file</code></h3>
<pre><code class="language-dotenv"># env.dev
DB_HOST=db
DB_USER=admin
DB_PASSWORD=change-me</code></pre>
<pre><code class="language-yaml">services:
  backend:
    image: my-backend
    env_file:
      - env.dev</code></pre>
<h3 id="env-interpolation"><code>.env</code> interpolation</h3>
<pre><code class="language-dotenv">PORT=3000
DB_PASSWORD=change-me</code></pre>
<pre><code class="language-yaml">services:
  web:
    ports:
      - &quot;${PORT}:80&quot;
    environment:
      DATABASE_PASSWORD: &quot;${DB_PASSWORD}&quot;</code></pre>
<p>다른 interpolation 파일 사용:</p>
<pre><code class="language-bash">docker compose --env-file .env.production up -d</code></pre>
<p>현재 shell 변수로 전달:</p>
<pre><code class="language-bash">APP_ENV=prod docker compose up -d</code></pre>
<blockquote>
<p>비밀번호를 Compose 파일과 Git 저장소에 직접 기록하지 않는다. <code>.env</code>를 사용할 때도 <code>.gitignore</code>에 포함하고, 운영 환경에서는 secret 관리 수단을 사용한다.</p>
</blockquote>
<h2 id="56-service-discovery-localhost가-아닌-service-이름">5.6 Service discovery: <code>localhost</code>가 아닌 service 이름</h2>
<pre><code class="language-yaml">services:
  frontend:
    ports:
      - &quot;8080:80&quot;

  backend:
    expose:
      - &quot;8080&quot;

  db:
    image: postgres:15</code></pre>
<p>호출 주소:</p>
<pre><code class="language-text">Browser → http://localhost:8080
Frontend → http://backend:8080
Backend → jdbc:postgresql://db:5432/postgres</code></pre>
<p>컨테이너 안의 <code>localhost</code>는 그 컨테이너 자신이다. Backend에서 <code>localhost:5432</code>를 사용하면 DB가 아니라 Backend 컨테이너 자신의 5432번 포트를 찾는다.</p>
<p>컨테이너 IP는 재생성 시 바뀔 수 있으므로 고정 IP 대신 service 이름을 사용한다.</p>
<h2 id="57-ports와-expose">5.7 <code>ports</code>와 <code>expose</code></h2>
<h3 id="외부-공개">외부 공개</h3>
<pre><code class="language-yaml">ports:
  - &quot;8080:80&quot;</code></pre>
<p>모든 host interface에 bind된다.</p>
<pre><code class="language-text">0.0.0.0:8080 → container:80</code></pre>
<h3 id="host-로컬에서만-공개">host 로컬에서만 공개</h3>
<pre><code class="language-yaml">ports:
  - &quot;127.0.0.1:8080:80&quot;</code></pre>
<h3 id="내부-통신만-사용">내부 통신만 사용</h3>
<pre><code class="language-yaml">expose:
  - &quot;8080&quot;</code></pre>
<p><code>expose</code>는 host port를 열지 않는다. 같은 network의 다른 컨테이너가 container port로 접근할 수 있다는 의도를 문서화한다. 실제로는 애플리케이션이 해당 port에서 listen해야 한다.</p>
<p>DB처럼 외부 공개가 필요 없는 서비스에는 <code>ports</code>를 지정하지 않는 편이 안전하다.</p>
<h2 id="58-public·private-network-분리">5.8 Public·Private network 분리</h2>
<p>목표:</p>
<pre><code class="language-text">Browser
   |
   v
Frontend ── public ── Backend
                         |
                      private
                         |
                         DB</code></pre>
<ul>
<li>Frontend는 Backend와 통신 가능</li>
<li>Backend는 Frontend와 DB 모두와 통신 가능</li>
<li>Frontend는 DB와 직접 통신 불가</li>
<li>DB는 host에 port를 공개하지 않음</li>
</ul>
<pre><code class="language-yaml">services:
  frontend:
    image: my-frontend
    ports:
      - &quot;8080:80&quot;
    networks:
      - public

  backend:
    image: my-backend
    networks:
      - public
      - private

  db:
    image: postgres:15
    networks:
      - private

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true</code></pre>
<p><code>internal: true</code>는 해당 network에서 외부로 나가는 기본 연결까지 제한한다. 단순히 service끼리 분리하는 것보다 강한 설정이므로 이미지 pull이 아니라 <strong>실행 중 서비스의 외부 API 호출 필요성</strong>을 고려해야 한다.</p>
<p>이미 만들어 둔 외부 Docker network를 Compose가 사용하게 하려면:</p>
<pre><code class="language-yaml">networks:
  my-network:
    external: true</code></pre>
<h2 id="59-depends_on과-준비-완료-조건">5.9 <code>depends_on</code>과 준비 완료 조건</h2>
<p>단순 <code>depends_on</code>은 시작 순서를 표현할 뿐 애플리케이션 준비 완료를 보장하지 않는다. DB healthcheck와 <code>condition: service_healthy</code>를 함께 사용한다.</p>
<pre><code class="language-yaml">services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: postgres
    healthcheck:
      test: [&quot;CMD&quot;, &quot;pg_isready&quot;, &quot;-U&quot;, &quot;postgres&quot;]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  backend:
    image: my-backend
    depends_on:
      db:
        condition: service_healthy</code></pre>
<pre><code class="language-text">db container start
      ↓
DB가 실제 연결 가능
      ↓ healthy
backend start</code></pre>
<h2 id="510-현재-저장소의-전체-compose-실습">5.10 현재 저장소의 전체 Compose 실습</h2>
<p>실습 위치:</p>
<pre><code class="language-bash">cd ~/path/to/skala-container/01.answer-code/09.docker-compose/02.services</code></pre>
<p>구조:</p>
<pre><code class="language-text">02.services/
├── docker-compose.yaml
├── nginx.conf
├── db_data/
├── backend/
│   ├── Dockerfile.backend
│   └── app.py
└── frontend/
    ├── Dockerfile.frontend
    └── index.html</code></pre>
<p>핵심 Compose 구성:</p>
<pre><code class="language-yaml">services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: postgres
    volumes:
      - ./db_data:/var/lib/postgresql/data
    healthcheck:
      test: [&quot;CMD&quot;, &quot;pg_isready&quot;, &quot;-U&quot;, &quot;postgres&quot;]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    networks:
      - private

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.backend
    ports:
      - &quot;9090:8080&quot;
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: [&quot;CMD&quot;, &quot;wget&quot;, &quot;-qO-&quot;, &quot;http://localhost:8080/health&quot;]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    networks:
      - public
      - private

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.frontend
    ports:
      - &quot;8080:80&quot;
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - public

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true</code></pre>
<p>검증·실행:</p>
<pre><code class="language-bash">docker compose config
docker compose up --build -d
docker compose ps</code></pre>
<p>시작 순서:</p>
<pre><code class="language-text">DB healthy → Backend healthy → Frontend start</code></pre>
<p>로그와 health 확인:</p>
<pre><code class="language-bash">docker compose logs -f backend

docker inspect &quot;$(docker compose ps -q db)&quot; \
  --format '{{json .State.Health}}' | jq

docker inspect &quot;$(docker compose ps -q backend)&quot; \
  --format '{{json .State.Health}}' | jq</code></pre>
<p>접속:</p>
<pre><code class="language-text">Frontend: http://localhost:8080
Backend:  http://localhost:9090
Health:   http://localhost:9090/health
Users:    http://localhost:9090/users</code></pre>
<p>교재 일부 페이지의 <code>localhost:8080/heath</code>는 <code>health</code>의 오타일 수 있다. 실제 backend route와 Nginx proxy 경로를 확인한다.</p>
<p>Network 확인:</p>
<pre><code class="language-bash">docker network ls
docker network inspect 02services_public
docker network inspect 02services_private</code></pre>
<p>실제 network 이름은 Compose project 이름에 따라 달라질 수 있으므로 먼저 <code>docker network ls</code>로 확인한다.</p>
<p>정리:</p>
<pre><code class="language-bash">docker compose down</code></pre>
<p>DB 데이터까지 삭제할 의도가 있을 때만:</p>
<pre><code class="language-bash">docker compose down -v</code></pre>
<p>현재 구성은 bind mount <code>./db_data</code>를 사용하므로 호스트 <code>db_data</code> 디렉터리는 별도로 남는다.</p>
<h2 id="511-컨테이너-관련-도구-분류">5.11 컨테이너 관련 도구 분류</h2>
<p>비슷해 보이는 도구도 계층별 역할이 다르다.</p>
<pre><code class="language-text">Orchestration
  Kubernetes, Docker Compose
         ↓
High-level Runtime / Engine
  Docker Engine, containerd, CRI-O, Podman
         ↓
Low-level OCI Runtime
  runc, crun, youki, runsc, kata-runtime
         ↓
Linux Kernel
  namespace, cgroup, mount, capability, seccomp</code></pre>
<h3 id="통합-컨테이너-플랫폼">통합 컨테이너 플랫폼</h3>
<p>이미지 빌드, 실행, network, volume 관리 기능을 묶어 제공한다.</p>
<ul>
<li>Docker Engine/Desktop: Docker daemon과 Desktop UI 중심</li>
<li>Podman: daemonless·rootless 실행을 지원하는 컨테이너 도구</li>
<li>Rancher Desktop: 로컬 containerd/Docker와 Kubernetes 개발 환경</li>
<li>Colima: macOS에서 경량 VM을 이용해 Docker/containerd 환경 제공</li>
</ul>
<h3 id="이미지-빌드-도구">이미지 빌드 도구</h3>
<table>
<thead>
<tr>
<th>도구</th>
<th>실행 방식·특징</th>
<th>주요 용도</th>
</tr>
</thead>
<tbody><tr>
<td>Docker Build</td>
<td>Docker daemon 사용</td>
<td>일반 로컬 빌드</td>
</tr>
<tr>
<td>BuildKit</td>
<td>병렬·cache 중심 builder</td>
<td>빠른 현대적 빌드</td>
</tr>
<tr>
<td>Docker Buildx</td>
<td>BuildKit 기반 Docker plugin</td>
<td>multi-architecture 이미지</td>
</tr>
<tr>
<td>Kaniko</td>
<td>컨테이너 안에서 daemon 없이 빌드</td>
<td>Kubernetes CI 환경</td>
</tr>
<tr>
<td>Buildah</td>
<td>daemonless·rootless 가능</td>
<td>Podman 생태계</td>
</tr>
<tr>
<td>Buildpacks</td>
<td>Dockerfile 없이 앱 분석 후 빌드</td>
<td>애플리케이션 자동 패키징</td>
</tr>
<tr>
<td>Jib</td>
<td>Maven·Gradle에서 Java 이미지 생성</td>
<td>Java 프로젝트</td>
</tr>
<tr>
<td>Bazel</td>
<td>재현 가능한 대규모 build graph</td>
<td>대형 monorepo</td>
</tr>
<tr>
<td>img</td>
<td>rootless BuildKit 계열</td>
<td>daemonless Dockerfile 빌드</td>
</tr>
</tbody></table>
<h3 id="고수준-컨테이너-런타임">고수준 컨테이너 런타임</h3>
<p>이미지를 가져오고 container lifecycle을 관리하며 low-level runtime을 호출한다.</p>
<table>
<thead>
<tr>
<th>런타임</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>containerd</td>
<td>Docker와 Kubernetes에서 널리 사용</td>
</tr>
<tr>
<td>CRI-O</td>
<td>Kubernetes CRI 전용 경량 runtime</td>
</tr>
<tr>
<td>Docker Engine</td>
<td>dockerd가 containerd를 통해 관리</td>
</tr>
</tbody></table>
<h3 id="저수준-oci-런타임">저수준 OCI 런타임</h3>
<p>실제로 Namespace, cgroup, rootfs, security 설정을 적용하고 프로세스를 실행한다.</p>
<table>
<thead>
<tr>
<th>런타임</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>runc</td>
<td>대표적인 OCI runtime</td>
</tr>
<tr>
<td>crun</td>
<td>C 기반의 빠르고 경량인 runtime</td>
</tr>
<tr>
<td>youki</td>
<td>Rust 기반 OCI runtime</td>
</tr>
<tr>
<td>runsc</td>
<td>gVisor sandbox를 통한 격리 강화</td>
</tr>
<tr>
<td>kata-runtime</td>
<td>경량 VM 기반의 강한 격리</td>
</tr>
</tbody></table>
<h3 id="오케스트레이션-플랫폼">오케스트레이션 플랫폼</h3>
<p>여러 컨테이너의 배포·확장·복구·service discovery를 관리한다.</p>
<ul>
<li>Docker Compose: 단일 host 중심 개발·테스트</li>
<li>Kubernetes: 여러 node에 걸친 배포와 운영</li>
</ul>
<p>Kubernetes의 일반적인 실행 흐름:</p>
<pre><code class="language-text">kubelet
   ↓ CRI
containerd 또는 CRI-O
   ↓
runc/crun 등 OCI runtime
   ↓
Pod container process</code></pre>
<p>과거의 dockershim 경로와 달리 현대 Kubernetes는 CRI를 통해 containerd·CRI-O와 직접 연동한다.</p>
<p>Kubernetes에서는 control plane의 API Server, Scheduler, Controller Manager, etcd가 원하는 상태를 관리하고, 각 node의 kubelet이 CRI를 통해 container runtime에 Pod 실행을 요청한다. CNI는 network, CSI는 storage, OCI runtime은 최종 container process 실행 계층을 담당한다.</p>
<h2 id="512-종합-실습-웹-서비스를-compose로-전환">5.12 종합 실습: 웹 서비스를 Compose로 전환</h2>
<p>1일차에 만든 Frontend, Backend, DB를 하나의 Compose project로 구조화한다.</p>
<pre><code class="language-text">Browser
   |
   v
Frontend(Nginx)
   |
   v
Backend(Spring/FastAPI)
   |
   v
Database(Postgres/MariaDB)</code></pre>
<p>완료 기준:</p>
<ul>
<li>각 서비스에 Dockerfile 또는 image가 지정됨</li>
<li>DB 데이터 volume 구성</li>
<li>Frontend만 필요한 host port 공개</li>
<li>service 이름으로 내부 통신</li>
<li>public/private network 분리</li>
<li>DB와 Backend healthcheck</li>
<li><code>depends_on: condition: service_healthy</code></li>
<li>적절한 restart policy</li>
<li>비밀값을 source에 직접 기록하지 않음</li>
<li><code>docker compose config</code> 통과</li>
<li><code>docker compose up --build -d</code> 성공</li>
<li><code>docker compose ps</code>에서 healthy 확인</li>
<li>브라우저 요청이 DB 저장까지 연결됨</li>
</ul>
<hr />
<h1 id="6-2일차-핵심-개념-비교">6. 2일차 핵심 개념 비교</h1>
<h2 id="image-manifest-vs-config-vs-layer">Image Manifest vs Config vs Layer</h2>
<table>
<thead>
<tr>
<th>대상</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Manifest</td>
<td>config와 layer의 digest·순서 연결</td>
</tr>
<tr>
<td>Config</td>
<td>실행 명령과 환경 설정</td>
</tr>
<tr>
<td>Layer</td>
<td>파일시스템 변경 내용</td>
</tr>
</tbody></table>
<h2 id="namespace-vs-cgroup">Namespace vs cgroup</h2>
<table>
<thead>
<tr>
<th>Namespace</th>
<th>cgroup</th>
</tr>
</thead>
<tbody><tr>
<td>무엇이 보이는지 격리</td>
<td>얼마나 쓸 수 있는지 제한</td>
</tr>
<tr>
<td>PID, NET, MNT, UTS 등</td>
<td>CPU, memory, I/O, PID 수 등</td>
</tr>
</tbody></table>
<h2 id="image-layer-vs-container-layer">Image layer vs Container layer</h2>
<table>
<thead>
<tr>
<th>Image layer</th>
<th>Container UpperDir</th>
</tr>
</thead>
<tbody><tr>
<td>읽기 전용</td>
<td>읽기·쓰기 가능</td>
</tr>
<tr>
<td>여러 컨테이너가 공유</td>
<td>해당 컨테이너에 종속</td>
</tr>
<tr>
<td>이미지의 불변 원본</td>
<td>실행 중 변경 내용</td>
</tr>
</tbody></table>
<h2 id="dockerd-vs-containerd-vs-runc">dockerd vs containerd vs runc</h2>
<table>
<thead>
<tr>
<th>구성</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>dockerd</td>
<td>Docker 전체 API와 자원 관리</td>
</tr>
<tr>
<td>containerd</td>
<td>이미지·snapshot·lifecycle 관리</td>
</tr>
<tr>
<td>runc</td>
<td>OCI bundle로 실제 프로세스 실행</td>
</tr>
</tbody></table>
<h2 id="ports-vs-expose"><code>ports</code> vs <code>expose</code></h2>
<table>
<thead>
<tr>
<th><code>ports</code></th>
<th><code>expose</code></th>
</tr>
</thead>
<tbody><tr>
<td>host port에 실제 공개</td>
<td>내부 port 의도 문서화</td>
</tr>
<tr>
<td>브라우저 등 외부 접근 가능</td>
<td>같은 network의 service 통신용</td>
</tr>
</tbody></table>
<h2 id="stop-vs-down"><code>stop</code> vs <code>down</code></h2>
<table>
<thead>
<tr>
<th><code>docker compose stop</code></th>
<th><code>docker compose down</code></th>
</tr>
</thead>
<tbody><tr>
<td>컨테이너 보존</td>
<td>컨테이너와 Compose network 제거</td>
</tr>
<tr>
<td><code>start</code>로 재사용</td>
<td>다음 <code>up</code>에서 재생성</td>
</tr>
</tbody></table>
<hr />
<h1 id="7-초보자가-자주-하는-실수">7. 초보자가 자주 하는 실수</h1>
<h2 id="71-이미지의-sha-256-값을-교재-그대로-사용한다">7.1 이미지의 SHA-256 값을 교재 그대로 사용한다</h2>
<p>digest는 빌드 결과와 architecture에 따라 달라질 수 있다. 자신의 <code>manifest.json</code>, <code>index.json</code>, <code>docker inspect</code> 결과에서 찾아야 한다.</p>
<h2 id="72-docker-export로-이미지-구조를-분석한다">7.2 <code>docker export</code>로 이미지 구조를 분석한다</h2>
<p>이미지 레이어와 config까지 보려면 <code>docker save</code>를 사용한다. <code>docker export</code>는 병합된 컨테이너 파일시스템 중심이다.</p>
<h2 id="73---privileged와-docker-socket을-일반-서비스에-사용한다">7.3 <code>--privileged</code>와 Docker socket을 일반 서비스에 사용한다</h2>
<p>둘 다 호스트 장악으로 이어질 수 있는 강력한 권한이다. runc 학습용 로컬 환경 외에는 사용하지 않는다.</p>
<h2 id="74-compose-내부에서-localhost로-db에-접속한다">7.4 Compose 내부에서 <code>localhost</code>로 DB에 접속한다</h2>
<pre><code class="language-text">잘못된 주소: localhost:5432
올바른 주소: db:5432</code></pre>
<h2 id="75-depends_on만으로-db-준비-완료를-기대한다">7.5 <code>depends_on</code>만으로 DB 준비 완료를 기대한다</h2>
<p>시작 순서와 준비 완료는 다르다. DB healthcheck와 <code>service_healthy</code> 조건을 함께 사용한다.</p>
<h2 id="76-expose-또는-compose-expose를-host-port-공개로-생각한다">7.6 <code>EXPOSE</code> 또는 Compose <code>expose</code>를 host port 공개로 생각한다</h2>
<p>외부 공개에는 <code>ports</code>가 필요하다.</p>
<h2 id="77-compose-파일-수정-후-restart만-한다">7.7 Compose 파일 수정 후 <code>restart</code>만 한다</h2>
<p>환경변수, mount, port, image 설정 변경은 recreate가 필요할 수 있다.</p>
<pre><code class="language-bash">docker compose up -d --force-recreate</code></pre>
<h2 id="78-down--v로-db-데이터를-지운다">7.8 <code>down -v</code>로 DB 데이터를 지운다</h2>
<p>Named volume 삭제 의도를 확인한 뒤 실행한다. 데이터가 필요하면 먼저 backup한다.</p>
<hr />
<h1 id="8-2일차-완료-체크리스트">8. 2일차 완료 체크리스트</h1>
<ul>
<li><input disabled="" type="checkbox" /> rootfs, manifest, config, layer의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> <code>docker save</code>로 이미지를 저장하고 내부 파일을 확인했다.</li>
<li><input disabled="" type="checkbox" /> Dockerfile 지시어와 image layer/config의 관계를 이해했다.</li>
<li><input disabled="" type="checkbox" /> MariaDB 이미지에서 entrypoint layer를 찾았다.</li>
<li><input disabled="" type="checkbox" /> Namespace와 cgroup의 역할을 구분한다.</li>
<li><input disabled="" type="checkbox" /> Capability와 SELinux/AppArmor의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> OverlayFS의 LowerDir, UpperDir, WorkDir, MergedDir를 구분한다.</li>
<li><input disabled="" type="checkbox" /> Copy-on-Write를 직접 확인했다.</li>
<li><input disabled="" type="checkbox" /> dockerd, containerd, shim, runc의 역할을 구분한다.</li>
<li><input disabled="" type="checkbox" /> OCI Image/Runtime/Distribution Spec의 목적을 안다.</li>
<li><input disabled="" type="checkbox" /> bridge, custom bridge, host network를 구분한다.</li>
<li><input disabled="" type="checkbox" /> veth, bridge, DNAT를 통한 port 전달 흐름을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> UDS와 TCP socket의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> OCI Bundle을 만들고 runc로 프로세스를 실행했다.</li>
<li><input disabled="" type="checkbox" /> 컨테이너가 격리된 프로세스 그룹이라는 의미를 이해했다.</li>
<li><input disabled="" type="checkbox" /> <code>docker compose config</code>, <code>up</code>, <code>ps</code>, <code>logs</code>, <code>down</code>을 사용할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Compose service 이름으로 컨테이너 간 통신한다.</li>
<li><input disabled="" type="checkbox" /> Restart policy와 healthcheck를 구분한다.</li>
<li><input disabled="" type="checkbox" /> Public·Private network를 분리할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 이미지 builder, high-level runtime, low-level runtime, orchestrator를 구분한다.</li>
</ul>
<hr />
<h1 id="9-마무리-오늘-배운-내용을-한-문장씩">9. 마무리: 오늘 배운 내용을 한 문장씩</h1>
<ol>
<li><strong>컨테이너 이미지는 rootfs 레이어와 실행 메타데이터를 표준 형식으로 묶은 배포 패키지다.</strong></li>
<li><strong>컨테이너는 VM이 아니라 Namespace와 cgroup으로 격리·제한된 Linux 프로세스 그룹이다.</strong></li>
<li><strong>OverlayFS는 읽기 전용 이미지와 쓰기 가능한 컨테이너 변경사항을 하나의 rootfs로 보여준다.</strong></li>
<li><strong>dockerd가 전체를 관리하고 containerd가 생명주기를 관리하며 runc가 실제 PID 1을 실행한다.</strong></li>
<li><strong>OCI 표준 덕분에 서로 다른 builder, registry, runtime 사이에서 이미지를 호환할 수 있다.</strong></li>
<li><strong>Docker bridge는 veth, Linux bridge, routing, Netfilter/NAT로 컨테이너 통신을 구현한다.</strong></li>
<li><strong>Docker Compose는 여러 컨테이너의 실행 설정과 수명주기를 YAML 하나로 관리한다.</strong></li>
<li><strong>Compose 내부에서는 IP나 localhost 대신 service 이름으로 통신한다.</strong></li>
<li><strong>Healthcheck는 준비 상태를 측정하고 restart policy는 종료 후 재시작 방법을 정한다.</strong></li>
<li><strong>빌드 도구, 고수준 런타임, 저수준 런타임, 오케스트레이터는 서로 다른 계층의 책임을 가진다.</strong></li>
</ol>