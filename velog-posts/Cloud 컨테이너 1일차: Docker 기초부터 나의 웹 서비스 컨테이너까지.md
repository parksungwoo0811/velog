<blockquote>
<p>이 글은 컨테이너를 처음 배우는 사람을 위한 1일차 수업 정리다.<br />오늘의 흐름은 <strong>환경 준비 → 컨테이너 개념 → MariaDB 실행 → Docker 명령어 → Dockerfile → 볼륨 → CMD 최적화 → 3계층 웹 서비스 구성</strong>이다.</p>
</blockquote>
<hr />
<h2 id="오늘-수업에서-만들게-될-그림">오늘 수업에서 만들게 될 그림</h2>
<p>오늘은 단순히 Docker 명령어를 외우는 날이 아니다. 최종적으로 다음 구조를 이해하고 직접 실행하는 것이 목표다.</p>
<pre><code class="language-text">사용자 브라우저
      |
      | http://localhost:9090
      v
Frontend 컨테이너 (Nginx, HTML/CSS/JavaScript)
      |
      | /api 요청, Docker 네트워크 내부 통신
      v
Backend 컨테이너 (Spring Boot)
      |
      | SQL
      v
MariaDB 컨테이너
      |
      v
호스트 디스크의 데이터 디렉터리 (영속 저장)</code></pre>
<p>각 프로그램을 컨테이너라는 독립된 실행 상자에 넣되, 네트워크와 볼륨을 통해 필요한 부분만 연결한다.</p>
<hr />
<h1 id="1-환경-구성하기-및-컨테이너-이해">1. 환경 구성하기 및 컨테이너 이해</h1>
<h2 id="11-컨테이너-도구의-역할부터-구분하기">1.1 컨테이너 도구의 역할부터 구분하기</h2>
<p>컨테이너 생태계에는 비슷해 보이는 도구가 많다. 처음에는 다음 세 역할만 구분하면 된다.</p>
<table>
<thead>
<tr>
<th>역할</th>
<th>하는 일</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>이미지 빌드 도구</td>
<td>Dockerfile을 읽어 이미지를 만든다</td>
<td>조립 설명서로 제품 생산</td>
</tr>
<tr>
<td>컨테이너 런타임</td>
<td>이미지를 실제 프로세스로 실행한다</td>
<td>제품의 전원을 켬</td>
</tr>
<tr>
<td>컨테이너 플랫폼</td>
<td>빌드, 실행, 네트워크, 볼륨 등을 통합 관리한다</td>
<td>공장 전체 운영 시스템</td>
</tr>
</tbody></table>
<p>Docker Desktop은 로컬 개발자가 Docker Engine, 이미지, 컨테이너, 네트워크, 볼륨을 편리하게 다루도록 제공하는 통합 환경이다.</p>
<h2 id="12-docker-desktop과-로그인">1.2 Docker Desktop과 로그인</h2>
<ol>
<li>Docker Desktop을 설치한다.</li>
<li>Docker Desktop을 실행해 Docker Engine이 동작하도록 한다.</li>
<li>이미지 내려받기 제한을 줄이기 위해 Docker 계정으로 로그인한다.</li>
<li>오늘 실습에서는 Docker Desktop의 Kubernetes 기능을 켜지 않는다. 불필요한 제어-plane 컨테이너가 실행되어 CPU와 메모리를 더 사용할 수 있기 때문이다.</li>
</ol>
<p>설치 후 터미널에서 확인한다.</p>
<pre><code class="language-bash">docker version
docker info</code></pre>
<p><code>docker version</code>은 CLI와 서버 정보를, <code>docker info</code>는 이미지·컨테이너·스토리지·네트워크 등 Docker Engine의 전체 상태를 보여준다.</p>
<h2 id="docker란">Docker란?</h2>
<p>Docker는 애플리케이션과 실행에 필요한 파일·라이브러리를 <strong>컨테이너(Container)</strong>라는 독립된 환경에서 실행하도록 도와주는 도구다.</p>
<p>쉽게 말하면 프로그램을 어디서든 동일하게 실행할 수 있도록 포장하는 기술이다.</p>
<pre><code class="language-text">애플리케이션 코드
+ 필요한 라이브러리
+ 실행 환경
= Docker 이미지</code></pre>
<p>이 이미지를 실행하면 컨테이너가 된다.</p>
<pre><code class="language-text">Dockerfile → 이미지 → 컨테이너
  조리법       밀키트      완성된 요리</code></pre>
<h3 id="주요-용어">주요 용어</h3>
<ul>
<li><strong>Dockerfile</strong>: 이미지를 만드는 방법을 기록한 파일</li>
<li><strong>이미지(Image)</strong>: 애플리케이션 실행 환경을 포장한 읽기 전용 설계도</li>
<li><strong>컨테이너(Container)</strong>: 이미지를 실제로 실행한 상태</li>
<li><strong>Registry</strong>: 이미지를 저장하고 공유하는 장소</li>
<li><strong>Docker Hub</strong>: 대표적인 공개 Registry</li>
</ul>
<h3 id="docker를-사용하는-이유">Docker를 사용하는 이유</h3>
<ul>
<li>개발 PC와 운영 서버에서 동일한 환경을 제공한다.</li>
<li>프로그램을 직접 설치하는 과정을 줄일 수 있다.</li>
<li>컨테이너를 빠르게 생성하고 삭제할 수 있다.</li>
<li>애플리케이션별 환경을 서로 격리할 수 있다.</li>
<li>서버 배포와 확장을 자동화하기 좋다.</li>
</ul>
<p>예를 들어 MariaDB를 직접 설치하지 않고 다음 명령으로 실행할 수 있다.</p>
<pre><code class="language-bash">docker run --name mariadb \
  -e MYSQL_ROOT_PASSWORD=password \
  -d mariadb</code></pre>
<h3 id="vm과-차이점">VM과 차이점</h3>
<p>VM은 애플리케이션마다 별도의 운영체제를 포함하지만, 컨테이너는 호스트의 Linux 커널을 공유한다. 따라서 일반적으로 VM보다 이미지가 작고 시작이 빠르다.</p>
<p>한 문장으로 정리하면:</p>
<blockquote>
<p><strong>Docker는 애플리케이션과 실행 환경을 이미지로 포장하고, 이를 격리된 컨테이너로 동일하게 실행하도록 도와주는 도구다.</strong></p>
</blockquote>
<h3 id="자주-만나는-환경-오류">자주 만나는 환경 오류</h3>
<h4 id="docker-daemon에-연결할-수-없음">Docker daemon에 연결할 수 없음</h4>
<pre><code class="language-text">Cannot connect to the Docker daemon ...
Is the docker daemon running?</code></pre>
<p>우선 Docker Desktop이 실행 중인지 확인하고 재시작한다. 그래도 해결되지 않을 때만 시스템 재부팅이나 프로세스 점검을 고려한다. 강제 프로세스 종료 명령은 열려 있는 Docker 작업을 잃을 수 있으므로 무조건 실행하지 않는다.</p>
<h4 id="8080-포트를-이미-사용-중임">8080 포트를 이미 사용 중임</h4>
<pre><code class="language-text">Bind for 0.0.0.0:8080 failed: port is already allocated</code></pre>
<p>사용 중인 프로세스를 먼저 확인한다.</p>
<pre><code class="language-bash">lsof -nP -iTCP:8080 -sTCP:LISTEN
docker ps</code></pre>
<p>기존 컨테이너가 사용 중이라면 해당 컨테이너를 정상 종료한다. 다른 프로그램이 사용 중이라면 그 프로그램을 확인한 뒤 종료하거나, 새 컨테이너의 호스트 포트를 <code>8081:8080</code>처럼 바꾼다.</p>
<h2 id="14-가상화란-무엇인가">1.4 가상화란 무엇인가?</h2>
<p>가상화는 <strong>한 대의 물리 컴퓨터 자원을 여러 독립된 컴퓨터처럼 나누어 사용하는 기술</strong>이다.</p>
<p>예를 들어 CPU 32코어, 메모리 128GB인 서버 한 대를 웹 서버, DB 서버, 테스트 서버로 나누어 쓰고 싶다고 하자. 가상화를 사용하면 실제 서버를 세 대 사지 않고도 여러 가상 환경을 만들 수 있다.</p>
<h2 id="15-하이퍼바이저와-vm">1.5 하이퍼바이저와 VM</h2>
<p>하이퍼바이저는 물리 하드웨어와 가상 머신 사이에서 다음 일을 수행한다.</p>
<ul>
<li>CPU, 메모리, 디스크, 네트워크 자원을 나누어 준다.</li>
<li>VM을 생성·삭제하고 스냅샷과 이동을 관리한다.</li>
<li>VM끼리 자원을 침범하지 못하도록 격리한다.</li>
<li>VM이 요청하는 하드웨어 작업을 중재한다.</li>
</ul>
<h3 id="type-1과-type-2-기본-상식">Type 1과 Type 2 (기본 상식)</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>Type 1: Bare-metal</th>
<th>Type 2: Hosted</th>
</tr>
</thead>
<tbody><tr>
<td>설치 위치</td>
<td>하드웨어 위에 직접</td>
<td>기존 운영체제 위에 설치</td>
</tr>
<tr>
<td>장점</td>
<td>성능과 안정성이 높음</td>
<td>개인 PC에서 쉽게 사용</td>
</tr>
<tr>
<td>용도</td>
<td>데이터센터, 클라우드</td>
<td>개발, 테스트</td>
</tr>
<tr>
<td>예</td>
<td>ESXi, Hyper-V, KVM, Xen</td>
<td>VirtualBox, VMware Workstation, Parallels</td>
</tr>
</tbody></table>
<p>Windows의 Hyper-V는 하이퍼바이저와 관리용 Root Partition이 결합된 구조로 이해하면 된다.</p>
<h2 id="16-vm만으로-애플리케이션을-운영할-때의-문제">1.6 VM만으로 애플리케이션을 운영할 때의 문제</h2>
<p>VM은 서버 자원을 나누는 데는 훌륭하지만 애플리케이션 배포 단위로는 무거울 수 있다.</p>
<ul>
<li>Guest OS와 가상 장치를 포함해 이미지가 수 GB 이상으로 커진다.</li>
<li>OS 부팅 때문에 시작에 수십 초에서 수 분이 걸릴 수 있다.</li>
<li>VM을 만든 뒤에도 미들웨어와 애플리케이션을 별도로 설치해야 한다.</li>
<li>개발·테스트·운영 환경의 패키지 버전이 달라질 수 있다.</li>
</ul>
<p>바로 여기서 유명한 문제가 생긴다.</p>
<blockquote>
<p>&quot;제 컴퓨터에서는 되는데요?&quot;</p>
</blockquote>
<h2 id="17-컨테이너의-핵심-개념">1.7 컨테이너의 핵심 개념</h2>
<p>컨테이너는 <strong>애플리케이션과 실행에 필요한 파일·라이브러리를 하나의 이미지로 묶고, 호스트의 Linux 커널을 공유해 격리된 프로세스로 실행하는 방식</strong>이다.</p>
<p>이 문장에서 가장 중요한 단어는 <code>프로세스</code>다. 컨테이너는 작은 VM 한 대라기보다 <strong>격리된 프로세스 실행 환경</strong>에 가깝다.</p>
<h3 id="이삿짐-비유">이삿짐 비유</h3>
<ul>
<li>전통적 서버 설치: 새집에 갈 때마다 가구를 하나씩 구매하고 조립한다.</li>
<li>VM 이미지: 집 전체를 통째로 복제해 옮긴다.</li>
<li>컨테이너 이미지: 생활에 필요한 물건을 규격 상자에 포장하고, 새 장소의 건물 기반 시설은 공유한다.</li>
</ul>
<h2 id="18-vm과-컨테이너-비교">1.8 VM과 컨테이너 비교</h2>
<table>
<thead>
<tr>
<th>비교</th>
<th>VM</th>
<th>컨테이너</th>
</tr>
</thead>
<tbody><tr>
<td>격리 단위</td>
<td>Guest OS를 포함한 가상 머신</td>
<td>호스트 커널을 공유하는 프로세스</td>
</tr>
<tr>
<td>이미지 내용</td>
<td>OS, 가상 장치, 앱</td>
<td>앱과 실행에 필요한 파일·라이브러리</td>
</tr>
<tr>
<td>크기</td>
<td>보통 GB 단위</td>
<td>보통 MB~수백 MB 단위</td>
</tr>
<tr>
<td>시작</td>
<td>OS 부팅 필요</td>
<td>프로세스 시작 수준</td>
</tr>
<tr>
<td>이식성</td>
<td>호환되는 하이퍼바이저 중심</td>
<td>컨테이너 런타임이 있는 환경 중심</td>
</tr>
<tr>
<td>자원 효율</td>
<td>Guest OS 오버헤드 존재</td>
<td>커널 공유로 상대적으로 높음</td>
</tr>
<tr>
<td>서로 다른 OS 커널</td>
<td>Guest OS별 구성 가능</td>
<td>호스트 커널 계열의 제약을 받음</td>
</tr>
</tbody></table>
<p>컨테이너가 VM보다 항상 우월한 것은 아니다. 완전히 다른 OS 커널이나 더 강한 격리 경계가 필요하면 VM이 적절할 수 있다. 실무에서는 <strong>VM 위에서 여러 컨테이너를 실행</strong>하는 조합도 흔하다.</p>
<h2 id="19-이미지와-dockerfile-맛보기">1.9 이미지와 Dockerfile 맛보기</h2>
<pre><code class="language-dockerfile">FROM python:3.10-alpine
RUN apk add --no-cache bash curl
RUN pip install fastapi uvicorn
COPY fastserver.py fastserver.py
CMD [&quot;python3&quot;, &quot;fastserver.py&quot;]</code></pre>
<p>Dockerfile은 “어떤 실행 환경을 이미지로 만들 것인가”를 선언하는 문서다.</p>
<ul>
<li><code>FROM</code>: 출발점이 되는 베이스 이미지</li>
<li><code>RUN</code>: 이미지 빌드 중 실행할 명령</li>
<li><code>COPY</code>: 로컬 파일을 이미지 안으로 복사</li>
<li><code>CMD</code>: 컨테이너가 시작될 때 실행할 기본 명령</li>
</ul>
<h3 id="1교시-체크포인트">1교시 체크포인트</h3>
<ul>
<li>이미지는 실행 전의 설계·포장 결과물이다.</li>
<li>컨테이너는 이미지를 실행해 만든 격리 프로세스다.</li>
<li>VM은 Guest OS를 포함하고, 컨테이너는 호스트 커널을 공유한다.</li>
<li>Dockerfile은 이미지 제작 절차를 선언한다.</li>
</ul>
<hr />
<h1 id="2-나의-데이터베이스-만들기">2. 나의 데이터베이스 만들기</h1>
<h2 id="21-설치하지-않고-mariadb-실행하기">2.1 설치하지 않고 MariaDB 실행하기</h2>
<p>예전 방식이라면 MariaDB 설치 파일을 내려받고 설정 파일과 계정을 직접 구성해야 했다. 컨테이너를 사용하면 이미 만들어진 <code>mariadb</code> 이미지를 실행하면서 설정값만 전달할 수 있다.</p>
<p>먼저 전용 bridge 네트워크를 만든다. 이미 존재하면 생성하지 않는다.</p>
<p>MariaDB를 실행한다.</p>
<pre><code class="language-bash">docker run -d \
  --name mariadb \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=name \
  -e MYSQL_USER=user \
  -e MYSQL_PASSWORD=password \
  --network name \
  -p 3306:3306 \
  mariadb:latest</code></pre>
<h3 id="옵션-해석">옵션 해석</h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>-d</code></td>
<td>백그라운드 실행</td>
</tr>
<tr>
<td><code>--name mariadb</code></td>
<td>컨테이너 이름 지정</td>
</tr>
<tr>
<td><code>-e KEY=VALUE</code></td>
<td>컨테이너에 환경변수 전달</td>
</tr>
<tr>
<td><code>--network skala</code></td>
<td><code>skala</code> 네트워크에 연결</td>
</tr>
<tr>
<td><code>-p 3306:3306</code></td>
<td>호스트 3306 → 컨테이너 3306 연결</td>
</tr>
<tr>
<td><code>mariadb:latest</code></td>
<td>실행할 이미지와 태그</td>
</tr>
</tbody></table>
<p><code>-p</code>의 형식은 항상 다음처럼 읽는다.</p>
<pre><code class="language-text">-p 호스트에서_접속할_포트:컨테이너_프로그램이_듣는_포트</code></pre>
<p>실무에서는 재현성을 위해 <code>latest</code> 대신 검증한 구체적 버전 태그를 사용하는 편이 안전하다.</p>
<h2 id="22-실행-상태와-로그-확인">2.2 실행 상태와 로그 확인</h2>
<pre><code class="language-bash">docker ps
docker logs mariadb</code></pre>
<p>DB는 시작 준비에 시간이 조금 걸릴 수 있다. 컨테이너가 보이는데 바로 접속되지 않는다면 로그에서 준비 완료 메시지나 오류를 확인한다.</p>
<h2 id="23-컨테이너-내부에서-db-사용하기">2.3 컨테이너 내부에서 DB 사용하기</h2>
<pre><code class="language-bash">docker exec -it mariadb /bin/bash
mariadb -u root -p</code></pre>
<p>비밀번호 <code>password</code>를 입력한 후 다음 SQL을 실행한다.</p>
<pre><code class="language-sql">SHOW DATABASES;
USE skala;

CREATE TABLE key_values (
  id INT AUTO_INCREMENT PRIMARY KEY,
  key_column VARCHAR(255) NOT NULL UNIQUE,
  value_column TEXT NOT NULL
);

SHOW TABLES;

INSERT INTO key_values (key_column, value_column)
VALUES ('example_key', 'example_value');

SELECT * FROM key_values;
EXIT;</code></pre>
<p>교재 일부 화면에는 <code>key_value</code>와 <code>key_values</code>가 섞여 있다. 위 예시는 테이블 이름을 <code>key_values</code>로 통일했다.</p>
<p>컨테이너 셸에서도 나온다.</p>
<pre><code class="language-bash">exit</code></pre>
<h2 id="24-dbeaver로-접속하기">2.4 DBeaver로 접속하기</h2>
<p>DBeaver에서 MariaDB 연결을 만들고 다음 값을 입력한다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>값</th>
</tr>
</thead>
<tbody><tr>
<td>Host</td>
<td><code>localhost</code></td>
</tr>
<tr>
<td>Port</td>
<td><code>3306</code></td>
</tr>
<tr>
<td>Database</td>
<td><code>skala</code></td>
</tr>
<tr>
<td>Username</td>
<td><code>root</code> 또는 <code>user</code></td>
</tr>
<tr>
<td>Password</td>
<td><code>your.password</code></td>
</tr>
</tbody></table>
<p>브라우저 주소와 달리 DB 연결 화면에는 <code>http://</code>를 붙이지 않는다.</p>
<h2 id="25-docker-desktop과-docker-inspect">2.5 Docker Desktop과 <code>docker inspect</code></h2>
<p>Docker Desktop에서는 이미지와 컨테이너를 GUI로 확인·시작·중지·삭제할 수 있다. CLI에서는 <code>inspect</code>로 Docker가 관리하는 원본 메타데이터를 JSON 형태로 본다.</p>
<pre><code class="language-bash">docker inspect mariadb</code></pre>
<p>여기에는 다음 정보가 들어 있다.</p>
<ul>
<li>이미지와 컨테이너 ID</li>
<li>환경변수</li>
<li>실행 명령</li>
<li>네트워크와 IP</li>
<li>포트 바인딩</li>
<li>마운트</li>
<li>상태와 종료 코드</li>
<li>로그 경로</li>
<li>재시작 정책</li>
</ul>
<h2 id="26-중지-삭제-정리의-차이">2.6 중지, 삭제, 정리의 차이</h2>
<pre><code class="language-bash">docker stop mariadb
docker rm mariadb</code></pre>
<ul>
<li><code>stop</code>: 실행 중인 프로세스를 멈추지만 컨테이너 기록은 남는다.</li>
<li><code>rm</code>: 정지된 컨테이너를 삭제한다.</li>
<li>이미지와 볼륨은 별도 자원이므로 컨테이너 삭제만으로 함께 지워지지 않는다.</li>
</ul>
<p>다음 명령은 넓은 범위의 미사용 자원을 삭제한다.</p>
<pre><code class="language-bash">docker system prune -a
docker volume prune -f</code></pre>
<blockquote>
<p>주의: <code>prune</code>은 다른 프로젝트에서 사용하지 않는 이미지·네트워크·볼륨까지 삭제할 수 있다. 공유 개발 환경에서는 삭제 대상을 먼저 확인하고 사용한다. 특히 DB 볼륨 삭제는 데이터 손실로 이어질 수 있다.</p>
</blockquote>
<h3 id="2교시-체크포인트">2교시 체크포인트</h3>
<ul>
<li>컨테이너 환경변수로 초기 DB·사용자·비밀번호를 전달했다.</li>
<li>포트 바인딩 덕분에 호스트의 DBeaver가 컨테이너 DB에 접속했다.</li>
<li>같은 Docker 네트워크의 다른 컨테이너는 이름 <code>mariadb</code>로 DB에 접근할 수 있다.</li>
<li>아직 볼륨을 연결하지 않았다면 컨테이너 삭제 시 DB 데이터도 함께 사라질 수 있다.</li>
</ul>
<hr />
<h1 id="3--컨테이너-기본-활용">3.  컨테이너 기본 활용</h1>
<h2 id="31-이미지-생명주기">3.1 이미지 생명주기</h2>
<pre><code class="language-text">검색(search) → 내려받기(pull) → 실행(run)
                         ↓
                    컨테이너 생성
                         ↓
            시작/정지/재시작/명령 실행
                         ↓
              컨테이너 삭제(rm)
                         ↓
                이미지 삭제(rmi)</code></pre>
<p>직접 만든 이미지라면 다음 흐름도 추가된다.</p>
<pre><code class="language-text">애플리케이션 개발 → Dockerfile 작성 → build → 로컬 실행·검증 → push → 배포 환경의 pull</code></pre>
<p>CI/CD는 이 빌드·검증·레지스트리 등록·배포 흐름을 자동화한다.</p>
<h2 id="32-registry-project-repository-tag">3.2 Registry, Project, Repository, Tag</h2>
<p>이미지 전체 이름을 주소처럼 쪼개 보자.</p>
<pre><code class="language-text">skala-registry.skala-ai.com/class-0/skala-webserver:1.0
|_________________________| |_____| |_____________| |__|
      Registry Domain       Project    Image Name    Tag</code></pre>
<ul>
<li><strong>Registry</strong>: 여러 이미지를 보관하고 제공하는 서버 또는 서비스</li>
<li><strong>Project/Namespace</strong>: 팀이나 조직별 구분</li>
<li><strong>Repository</strong>: 같은 이미지 계열을 모아 둔 저장 단위</li>
<li><strong>Tag</strong>: 이미지 버전이나 변형을 가리키는 이름</li>
<li><strong>Image ID/Digest</strong>: 실제 이미지 내용을 식별하는 값</li>
</ul>
<p><code>nginx</code>처럼 registry와 project가 생략된 이름은 Docker의 기본 규칙에 따라 사실상 <code>docker.io/library/nginx:latest</code>로 해석된다. Podman·Buildah의 기본 검색 registry는 설정에 따라 다를 수 있다.</p>
<h3 id="registry에-이미지-올리고-받는-흐름">Registry에 이미지 올리고 받는 흐름</h3>
<p>직접 만든 로컬 이미지를 registry 주소에 맞게 태그한 뒤 올린다.</p>
<pre><code class="language-bash">docker login skala-registry.skala-ai.com

docker tag myhello:0.1 \
  skala-registry.skala-ai.com/class-0/myhello:0.1

docker push skala-registry.skala-ai.com/class-0/myhello:0.1</code></pre>
<p>다른 환경에서는 전체 이미지 이름으로 내려받는다.</p>
<pre><code class="language-bash">docker pull skala-registry.skala-ai.com/class-0/myhello:0.1</code></pre>
<p>이미지를 수정해 다시 push하거나 pull할 때 모든 데이터를 통째로 전송하지 않는다. 로컬 또는 registry에 이미 존재하는 공통 레이어는 재사용하고 새로 달라진 레이어만 전송한다.</p>
<h2 id="33-이미지-레이어">3.3 이미지 레이어</h2>
<p>이미지는 하나의 거대한 파일이 아니라 여러 <strong>읽기 전용 레이어</strong>의 조합이다.</p>
<pre><code class="language-dockerfile">FROM ubuntu:22.04
RUN apt-get update &amp;&amp; apt-get install -y nginx
COPY ./src /var/www/html
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<ul>
<li><code>FROM</code>, <code>RUN</code>, <code>COPY</code> 등이 이미지 파일시스템을 단계적으로 구성한다.</li>
<li>여러 이미지가 같은 베이스 레이어를 공유하면 한 번 받은 레이어를 재사용한다.</li>
<li>수정되지 않은 빌드 단계는 캐시를 재사용할 수 있다.</li>
<li>컨테이너가 실행되면 읽기 전용 이미지 레이어 위에 쓰기 가능한 컨테이너 레이어가 추가된다.</li>
</ul>
<p>레고에 비유하면, 이미지마다 완성품을 통째로 보관하는 것이 아니라 공통 블록 묶음을 공유하는 셈이다.</p>
<h2 id="34-이미지-검색·다운로드·목록">3.4 이미지 검색·다운로드·목록</h2>
<pre><code class="language-bash">docker search ubuntu
docker pull ubuntu:latest
docker images</code></pre>
<ul>
<li>공식 이미지인지 확인한다.</li>
<li>Tags 목록에서 필요한 버전을 고른다.</li>
<li>태그를 생략하면 일반적으로 <code>latest</code>가 선택되지만, <code>latest</code>가 “가장 안정적”이라는 뜻은 아니다.</li>
</ul>
<p>컨테이너는 호스트의 Linux 커널을 공유하므로 이미지에 전체 커널이 들어 있지 않다. Ubuntu 이미지의 사용자 공간 파일을 다른 Linux 배포판 호스트 위에서 실행할 수 있는 이유다.</p>
<h2 id="35-첫-ubuntu-컨테이너-실행">3.5 첫 Ubuntu 컨테이너 실행</h2>
<pre><code class="language-bash">docker run -it --name hello ubuntu:latest /bin/bash</code></pre>
<p><code>docker run</code>은 필요한 이미지가 로컬에 없으면 내려받고, 컨테이너를 생성한 뒤, 지정한 명령을 실행한다.</p>
<ul>
<li><code>-i</code>: 표준 입력을 열어 둔다.</li>
<li><code>-t</code>: 가상 터미널을 할당한다.</li>
<li><code>--name hello</code>: 기억하기 쉬운 이름을 지정한다.</li>
<li><code>/bin/bash</code>: 컨테이너의 주 프로세스로 실행한다.</li>
</ul>
<p>컨테이너 안에서 둘러본다.</p>
<pre><code class="language-bash">pwd
ls
exit</code></pre>
<p>여기서는 <code>/bin/bash</code>가 주 프로세스다. <code>exit</code>로 Bash가 끝나면 주 프로세스가 사라지므로 컨테이너도 정지한다.</p>
<h2 id="36-상태-확인과-생명주기-명령">3.6 상태 확인과 생명주기 명령</h2>
<pre><code class="language-bash">docker ps       # 실행 중인 컨테이너
docker ps -a    # 정지된 컨테이너까지 전부

docker start hello
docker restart hello
docker stop hello
docker rm hello</code></pre>
<p>핵심 구분은 다음과 같다.</p>
<ul>
<li><code>run</code>: 새 컨테이너를 만들어 실행한다.</li>
<li><code>start</code>: 이미 존재하는 정지 컨테이너를 다시 실행한다.</li>
<li><code>restart</code>: 실행 상태를 다시 시작한다.</li>
<li><code>stop</code>: 정상 종료 신호를 보낸 뒤 정지한다.</li>
<li><code>rm</code>: 컨테이너 기록을 삭제한다.</li>
</ul>
<h2 id="37-실행-중인-컨테이너에서-명령-실행">3.7 실행 중인 컨테이너에서 명령 실행</h2>
<p>한 개 명령만 실행하려면 다음과 같이 한다.</p>
<pre><code class="language-bash">docker exec hello echo &quot;Hello World&quot;</code></pre>
<p>대화형 셸을 하나 더 실행하려면 다음과 같이 한다.</p>
<pre><code class="language-bash">docker exec -it hello /bin/bash</code></pre>
<p><code>exec</code>는 실행 중인 컨테이너에서만 사용할 수 있다. 이것은 컨테이너의 메인 프로세스를 교체하는 것이 아니라 <strong>추가 프로세스</strong>를 실행하는 명령이다.</p>
<h2 id="38-첫-bind-mount">3.8 첫 bind mount</h2>
<pre><code class="language-bash">mkdir -p ./data
echo &quot;hello docker world&quot; &gt; ./data/info.txt

docker run -it \
  --name hello \
  -v &quot;$(pwd)/data:/app/data&quot; \
  ubuntu:latest \
  /bin/bash</code></pre>
<p>컨테이너 안에서 확인한다.</p>
<pre><code class="language-bash">cd /app/data
cat info.txt
touch add.txt
exit</code></pre>
<p>호스트에서 확인한다.</p>
<pre><code class="language-bash">ls ./data</code></pre>
<p><code>add.txt</code>가 보인다면 호스트와 컨테이너가 같은 디렉터리를 보고 있는 것이다. 경로에 공백이 있을 수 있으므로 <code>&quot;$(pwd)/data:/app/data&quot;</code>처럼 따옴표로 감싸는 습관이 좋다. PowerShell 등 다른 셸에서는 현재 경로 표현이 다를 수 있다.</p>
<h2 id="39-이미지-삭제와-운영-보조-명령">3.9 이미지 삭제와 운영 보조 명령</h2>
<pre><code class="language-bash">docker rmi ubuntu:latest
docker --help
docker stats
docker network ls
docker network inspect skala
docker container prune
docker image prune
docker system prune</code></pre>
<p>이미지를 사용 중인 컨테이너가 남아 있으면 이미지 삭제가 거절될 수 있다. 먼저 어떤 컨테이너가 참조하는지 확인한다.</p>
<p>Docker Desktop에서도 컨테이너의 시작·정지·재시작·삭제, 이미지 관리를 할 수 있다. CLI와 GUI는 같은 Docker Engine 상태를 서로 다른 방식으로 보여준다.</p>
<h3 id="오전-명령어-한눈에-보기">오전 명령어 한눈에 보기</h3>
<table>
<thead>
<tr>
<th>목적</th>
<th>명령</th>
</tr>
</thead>
<tbody><tr>
<td>이미지 검색</td>
<td><code>docker search 이름</code></td>
</tr>
<tr>
<td>이미지 받기</td>
<td><code>docker pull 이름:태그</code></td>
</tr>
<tr>
<td>이미지 보기</td>
<td><code>docker images</code></td>
</tr>
<tr>
<td>새 컨테이너 생성·실행</td>
<td><code>docker run ...</code></td>
</tr>
<tr>
<td>실행 중 목록</td>
<td><code>docker ps</code></td>
</tr>
<tr>
<td>전체 목록</td>
<td><code>docker ps -a</code></td>
</tr>
<tr>
<td>내부 명령 실행</td>
<td><code>docker exec ...</code></td>
</tr>
<tr>
<td>시작·재시작·정지</td>
<td><code>docker start/restart/stop 이름</code></td>
</tr>
<tr>
<td>컨테이너 삭제</td>
<td><code>docker rm 이름</code></td>
</tr>
<tr>
<td>이미지 삭제</td>
<td><code>docker rmi 이름:태그</code></td>
</tr>
<tr>
<td>상세 정보</td>
<td><code>docker inspect 대상</code></td>
</tr>
<tr>
<td>실시간 자원 사용</td>
<td><code>docker stats</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="4-dockerfile-작성하기">4. Dockerfile 작성하기</h1>
<h2 id="41-dockerfile이란">4.1 Dockerfile이란?</h2>
<p>Dockerfile은 <strong>컨테이너 이미지 제작 절차 명세서</strong>다. 누군가의 PC에서 수동으로 서버를 설정하는 대신, 필요한 작업을 코드로 남겨 누구나 같은 이미지를 만들게 한다.</p>
<pre><code class="language-text">Dockerfile + 빌드 컨텍스트
          |
          | docker build
          v
        Image
          |
          | docker run
          v
      Container</code></pre>
<p><code>Dockerfile</code>은 보통 <code>FROM</code>으로 시작한다. 단, <code>FROM</code>에서 사용할 변수를 선언하는 <code>ARG</code>는 그 앞에 올 수 있다.</p>
<h2 id="42-첫-nginx-이미지">4.2 첫 Nginx 이미지</h2>
<pre><code class="language-dockerfile">FROM ubuntu:22.04

RUN apt-get update &amp;&amp; \
    apt-get install -y --no-install-recommends nginx &amp;&amp; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html
COPY index.html .

EXPOSE 80
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p>빌드하고 실행한다.</p>
<pre><code class="language-bash">docker build --tag myhello:0.1 .
docker images

docker run -d \
  --name hello-nginx \
  -p 8080:80 \
  myhello:0.1</code></pre>
<p>브라우저에서 <code>http://localhost:8080</code>으로 접속한다. <code>8080:80</code>은 <strong>호스트 8080번 포트가 컨테이너 Nginx의 80번 포트로 전달된다</strong>는 뜻이다.</p>
<pre><code class="language-bash">docker ps
docker stop hello-nginx
docker rm hello-nginx</code></pre>
<h2 id="43-주요-dockerfile-지시어">4.3 주요 Dockerfile 지시어</h2>
<table>
<thead>
<tr>
<th>지시어</th>
<th>시점</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>FROM</code></td>
<td>빌드</td>
<td>베이스 이미지 선택</td>
</tr>
<tr>
<td><code>RUN</code></td>
<td>빌드</td>
<td>패키지 설치, 파일 변경, 빌드 작업</td>
</tr>
<tr>
<td><code>ARG</code></td>
<td>빌드</td>
<td>빌드할 때 사용할 변수</td>
</tr>
<tr>
<td><code>LABEL</code></td>
<td>빌드</td>
<td>버전·작성자·용도 등 메타데이터</td>
</tr>
<tr>
<td><code>COPY</code></td>
<td>빌드</td>
<td>빌드 컨텍스트의 파일을 이미지로 복사</td>
</tr>
<tr>
<td><code>ADD</code></td>
<td>빌드</td>
<td>복사 + 일부 tar 자동 해제 등의 부가 기능</td>
</tr>
<tr>
<td><code>WORKDIR</code></td>
<td>빌드/실행 설정</td>
<td>이후 명령의 작업 디렉터리</td>
</tr>
<tr>
<td><code>ENV</code></td>
<td>빌드/실행</td>
<td>이미지와 컨테이너에 남는 환경변수</td>
</tr>
<tr>
<td><code>USER</code></td>
<td>빌드/실행</td>
<td>이후 명령과 프로세스의 사용자</td>
</tr>
<tr>
<td><code>EXPOSE</code></td>
<td>실행 설정</td>
<td>컨테이너가 사용하는 포트 문서화</td>
</tr>
<tr>
<td><code>VOLUME</code></td>
<td>실행 설정</td>
<td>익명 볼륨 마운트 지점 선언</td>
</tr>
<tr>
<td><code>CMD</code></td>
<td>실행</td>
<td>기본 명령 또는 기본 인자</td>
</tr>
<tr>
<td><code>ENTRYPOINT</code></td>
<td>실행</td>
<td>컨테이너의 고정 실행 프로그램</td>
</tr>
<tr>
<td><code>SHELL</code></td>
<td>빌드/실행 설정</td>
<td>shell form에서 사용할 기본 셸 변경</td>
</tr>
</tbody></table>
<h2 id="44-from과-multi-stage-build">4.4 <code>FROM</code>과 Multi-stage build</h2>
<p>Vue 애플리케이션을 만들 때 Node.js는 빌드에는 필요하지만 배포 후에는 필요하지 않다. 빌드용 이미지와 실행용 이미지를 분리하면 결과 이미지가 작고 공격 표면도 줄어든다.</p>
<pre><code class="language-dockerfile">FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY default.conf /etc/nginx/conf.d/
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p>1단계에는 Node.js, 소스 코드, <code>node_modules</code>가 있다. 2단계에는 완성된 <code>dist</code>와 Nginx만 들어간다.</p>
<pre><code class="language-bash">docker buildx build --tag vue-frontend:1.0 .
docker run -d --name vue-frontend -p 8080:80 vue-frontend:1.0
docker stop vue-frontend
docker rm vue-frontend</code></pre>
<h2 id="45-run-빌드-중-실행과-캐시">4.5 <code>RUN</code>: 빌드 중 실행과 캐시</h2>
<pre><code class="language-dockerfile">RUN apt-get update &amp;&amp; apt-get install -y \
    curl \
    git \
    &amp;&amp; rm -rf /var/lib/apt/lists/*</code></pre>
<ul>
<li><code>RUN</code> 결과는 이미지 파일시스템 레이어에 반영된다.</li>
<li>관련 명령을 묶으면 불필요한 레이어와 캐시 파일을 줄일 수 있다.</li>
<li>앞 단계가 바뀌면 그 뒤 캐시도 다시 계산된다.</li>
<li>패키지 목록 갱신과 설치는 같은 <code>RUN</code>에 두어 오래된 캐시 문제를 피한다.</li>
</ul>
<p>Exec form도 가능하다.</p>
<pre><code class="language-dockerfile">RUN [&quot;mkdir&quot;, &quot;-p&quot;, &quot;/var/www/html&quot;]</code></pre>
<p>확인 실습:</p>
<pre><code class="language-dockerfile">FROM ubuntu:22.04
RUN apt-get update &amp;&amp; apt-get install -y curl lsb-release</code></pre>
<pre><code class="language-bash">docker buildx build --tag linux-container:1.0 .
docker run -it --name linux-container linux-container:1.0 /bin/bash
lsb_release -a
exit
docker rm linux-container</code></pre>
<p>컨테이너 이름 충돌 오류가 나면 <code>docker ps -a</code>로 같은 이름이 남아 있는지 확인하고, 실행 중이면 먼저 중지한 뒤 삭제한다.</p>
<h2 id="46-arg-빌드-시점-변수">4.6 <code>ARG</code>: 빌드 시점 변수</h2>
<pre><code class="language-dockerfile">ARG UBUNTU_VERSION=22.04
FROM ubuntu:${UBUNTU_VERSION}

RUN apt-get update &amp;&amp; apt-get install -y curl lsb-release

# FROM 뒤에서 다시 사용하려면 이 stage 안에서 재선언
ARG UBUNTU_VERSION
RUN echo &quot;Build Ubuntu version: ${UBUNTU_VERSION}&quot;</code></pre>
<pre><code class="language-bash">docker buildx build \
  --tag linux-container:1.0 \
  --build-arg UBUNTU_VERSION=24.04 \
  .</code></pre>
<p><code>ARG</code>는 빌드에 쓰는 값이다. 비밀번호나 API 키를 <code>ARG</code>로 넣으면 이미지 기록에 노출될 수 있으므로 비밀 저장 수단으로 사용하지 않는다.</p>
<h2 id="47-label-이미지의-설명-정보">4.7 <code>LABEL</code>: 이미지의 설명 정보</h2>
<pre><code class="language-dockerfile">LABEL maintainer=&quot;team@example.com&quot; \
      description=&quot;Container training image&quot; \
      version=&quot;1.0.0&quot;</code></pre>
<pre><code class="language-bash">docker inspect linux-container:1.0</code></pre>
<p>LABEL은 실행 동작을 바꾸지 않지만 이미지 분류, 운영 관리, 자동화 정책에 유용하다.</p>
<p>과거 Dockerfile 예제의 <code>MAINTAINER</code> 지시어는 현재 <code>LABEL maintainer=&quot;...&quot;</code> 형태로 표현하는 것이 일반적이다.</p>
<h2 id="48-cmd-기본-실행-명령">4.8 <code>CMD</code>: 기본 실행 명령</h2>
<pre><code class="language-dockerfile">CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<ul>
<li>컨테이너가 시작될 때 실행한다.</li>
<li>Dockerfile에 여러 개가 있으면 마지막 <code>CMD</code>만 유효하다.</li>
<li><code>docker run 이미지 새명령</code>을 입력하면 CMD는 새 명령으로 대체된다.</li>
<li>서버 프로세스는 컨테이너가 살아 있도록 foreground에서 실행해야 한다.</li>
</ul>
<h2 id="49-expose-문서이지-포트-연결이-아니다">4.9 <code>EXPOSE</code>: 문서이지 포트 연결이 아니다</h2>
<pre><code class="language-dockerfile">EXPOSE 80/tcp</code></pre>
<p><code>EXPOSE</code>는 이미지 사용자가 “이 앱이 컨테이너 내부의 80번 포트를 사용한다”는 사실을 알게 하는 메타데이터다. 호스트에 포트를 실제 공개하려면 실행할 때 <code>-p</code>가 필요하다.</p>
<pre><code class="language-bash">docker image inspect --format='{{.Config.ExposedPorts}}' myhello:0.1
docker run -d -p 8080:80 myhello:0.1</code></pre>
<p>대문자 <code>-P</code>는 EXPOSE된 포트를 임의의 호스트 포트에 연결한다.</p>
<h2 id="410-copy-workdir-add">4.10 <code>COPY</code>, <code>WORKDIR</code>, <code>ADD</code></h2>
<pre><code class="language-dockerfile">WORKDIR /var/www/html
COPY index.html .</code></pre>
<ul>
<li><code>WORKDIR</code>은 폴더가 없으면 만들고 이후 명령의 기준 위치가 된다.</li>
<li><code>COPY</code>의 원본은 빌드 컨텍스트 안에 있어야 한다.</li>
<li><code>COPY --chown=user:group</code>으로 소유권을 지정할 수 있다.</li>
<li>Multi-stage에서는 <code>COPY --from=builder</code>로 이전 단계의 결과만 가져온다.</li>
<li>단순 파일 복사는 동작이 명확한 <code>COPY</code>를 우선한다. tar 자동 해제 같은 <code>ADD</code> 기능이 실제로 필요할 때만 <code>ADD</code>를 쓴다.</li>
</ul>
<p>예제 <code>index.html</code>:</p>
<pre><code class="language-html">&lt;!doctype html&gt;
&lt;html lang=&quot;ko&quot;&gt;
  &lt;head&gt;
    &lt;meta charset=&quot;utf-8&quot; /&gt;
    &lt;title&gt;SKALA Container&lt;/title&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;h1&gt;SKALA Container에 오신 것을 환영합니다.&lt;/h1&gt;
  &lt;/body&gt;
&lt;/html&gt;</code></pre>
<h2 id="411-user-root가-아닌-사용자로-실행">4.11 <code>USER</code>: root가 아닌 사용자로 실행</h2>
<p>컨테이너 기본 사용자는 보통 root다. 최소 권한 원칙에 따라 앱은 가능하면 일반 사용자로 실행한다.</p>
<pre><code class="language-dockerfile">RUN groupadd skala &amp;&amp; \
    useradd --create-home --shell /bin/bash --gid skala skala

RUN chown -R skala:skala /var/lib/nginx /var/log/nginx /run

RUN sed -i \
    -e 's/listen 80 default_server;/listen 8080 default_server;/' \
    -e 's/listen \[::\]:80 default_server;/listen [::]:8080 default_server;/' \
    /etc/nginx/sites-enabled/default

USER skala
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<p>일반 사용자는 root 소유 디렉터리에 쓸 수 없고, Linux 설정에 따라 1024 미만의 privileged port 바인딩도 제한된다. 그래서 필요한 디렉터리의 소유권과 Nginx 수신 포트를 함께 조정한다. <code>USER</code>는 계정을 자동 생성하지 않으므로 먼저 <code>useradd</code> 또는 이미지에 맞는 계정 생성 명령이 필요하다.</p>
<h2 id="412-env-실행할-때까지-남는-환경변수">4.12 <code>ENV</code>: 실행할 때까지 남는 환경변수</h2>
<pre><code class="language-dockerfile">ENV APP_HOME=/opt/myapp \
    SPRING_PROFILES_ACTIVE=prod \
    JAVA_OPTS=&quot;-Xms256m -Xmx512m&quot;

WORKDIR $APP_HOME</code></pre>
<p>실행할 때 덮어쓸 수 있다.</p>
<pre><code class="language-bash">docker run -e SPRING_PROFILES_ACTIVE=local myapp:1.0</code></pre>
<p>이미지에 남아도 되는 기본 설정만 ENV에 둔다. 비밀번호는 Dockerfile에 직접 기록하지 않는다.</p>
<h2 id="413-entrypoint와-cmd">4.13 <code>ENTRYPOINT</code>와 <code>CMD</code></h2>
<pre><code class="language-dockerfile">FROM alpine:latest
ENTRYPOINT [&quot;ping&quot;]
CMD [&quot;-c&quot;, &quot;3&quot;, &quot;localhost&quot;]</code></pre>
<pre><code class="language-bash">docker run --rm check-ping
docker run --rm check-ping google.com</code></pre>
<p>첫 명령은 <code>ping -c 3 localhost</code>, 둘째는 CMD를 대체하여 <code>ping google.com</code>처럼 동작한다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>ENTRYPOINT</th>
<th>CMD</th>
</tr>
</thead>
<tbody><tr>
<td>의미</td>
<td>컨테이너의 고정된 주 프로그램</td>
<td>기본 명령 또는 기본 인자</td>
</tr>
<tr>
<td>이미지 뒤에 값 입력</td>
<td>ENTRYPOINT 뒤의 인자가 됨</td>
<td>기존 CMD가 대체됨</td>
</tr>
<tr>
<td>강제 변경</td>
<td><code>--entrypoint</code> 필요</td>
<td>이미지 뒤에 새 명령 입력</td>
</tr>
</tbody></table>
<h2 id="414-volume과-shell">4.14 <code>VOLUME</code>과 <code>SHELL</code></h2>
<p>Dockerfile에서도 마운트 지점을 선언할 수 있다.</p>
<pre><code class="language-dockerfile">VOLUME [&quot;/data&quot;]</code></pre>
<p>이 선언은 호스트의 특정 경로를 직접 지정하지 않는다. 컨테이너 실행 시 별도 마운트를 주지 않으면 Docker가 익명 볼륨을 만들 수 있다. 어떤 데이터를 어디에 보관할지는 배포 환경의 책임인 경우가 많으므로, 실무에서는 <code>docker run --mount</code>나 Compose 설정에서 named volume 또는 bind mount를 명시하는 방식이 더 관리하기 쉽다.</p>
<p><code>SHELL</code>은 이후 shell form 명령이 사용할 기본 셸을 바꾼다.</p>
<pre><code class="language-dockerfile">SHELL [&quot;/bin/bash&quot;, &quot;-c&quot;]
RUN echo &quot;bash로 실행&quot;</code></pre>
<p>운영체제와 베이스 이미지에 따라 Bash가 없을 수 있다. 특별한 셸 기능이 필요할 때만 명시적으로 사용한다.</p>
<h2 id="415-레이어를-만드는-명령과-실행-설정을-만드는-명령">4.15 레이어를 만드는 명령과 실행 설정을 만드는 명령</h2>
<table>
<thead>
<tr>
<th>분류</th>
<th>대표 지시어</th>
<th>결과</th>
</tr>
</thead>
<tbody><tr>
<td>파일시스템 레이어</td>
<td><code>RUN</code>, <code>COPY</code>, <code>ADD</code></td>
<td>이미지 내부 파일 생성·변경</td>
</tr>
<tr>
<td>컨테이너 실행 설정</td>
<td><code>CMD</code>, <code>ENTRYPOINT</code>, <code>ENV</code>, <code>WORKDIR</code>, <code>EXPOSE</code>, <code>USER</code>, <code>VOLUME</code>, <code>LABEL</code></td>
<td>실행 시 사용할 설정 메타데이터</td>
</tr>
</tbody></table>
<p>이 구분을 알면 “왜 <code>CMD</code>를 바꿨는데 이미지 용량은 거의 그대로인가?”, “왜 <code>EXPOSE</code>만 했는데 접속이 안 되는가?” 같은 질문에 답할 수 있다.</p>
<hr />
<h1 id="5-볼륨-연결하기--cmd-명령-최적화">5. 볼륨 연결하기 &amp; CMD 명령 최적화</h1>
<h2 id="51-컨테이너는-일회성이다">5.1 컨테이너는 일회성이다</h2>
<p>컨테이너의 쓰기 가능한 레이어는 컨테이너에 종속된다. 컨테이너를 삭제하면 그 안에만 저장한 DB 파일도 사라진다.</p>
<p>컨테이너를 카페의 임시 작업대로 생각해 보자. 작업대는 언제든 교체할 수 있지만 중요한 장부는 작업대에 붙여 두지 않고 별도 금고에 보관해야 한다. 그 금고가 볼륨이다.</p>
<h2 id="52-세-가지-볼륨-타입">5.2 세 가지 볼륨 타입</h2>
<table>
<thead>
<tr>
<th>타입</th>
<th>누가 위치를 관리하나?</th>
<th>추천 용도</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>Named volume</td>
<td>Docker</td>
<td>DB 데이터, 앱의 영속 데이터</td>
<td>이름으로 관리하기 편함</td>
</tr>
<tr>
<td>Bind mount</td>
<td>사용자</td>
<td>소스 코드, 설정 파일, 로컬 개발</td>
<td>호스트 경로가 명확함</td>
</tr>
<tr>
<td>Anonymous volume</td>
<td>Docker</td>
<td>임시 마운트</td>
<td>자동 생성 이름 때문에 추적이 어려움</td>
</tr>
</tbody></table>
<h2 id="53-named-volume-실습">5.3 Named volume 실습</h2>
<pre><code class="language-bash">docker run -it --name busybox -v demo:/usr/share busybox</code></pre>
<p>컨테이너 내부에서 확인한다.</p>
<pre><code class="language-bash">mount | grep /usr/share
exit</code></pre>
<p>호스트에서 마운트 메타데이터를 확인한다.</p>
<pre><code class="language-bash">docker inspect busybox --format '{{json .Mounts}}' | jq
docker rm busybox
docker volume ls</code></pre>
<p>컨테이너를 지워도 <code>demo</code>라는 named volume은 남는다. 이것이 데이터와 컨테이너의 수명주기를 분리하는 핵심이다.</p>
<h2 id="54-bind-mount-실습">5.4 Bind mount 실습</h2>
<pre><code class="language-bash">docker run -it \
  --name busybox \
  -v &quot;$(pwd):/usr/share&quot; \
  busybox</code></pre>
<p>컨테이너 안에서 호스트 디렉터리에 파일을 만든다.</p>
<pre><code class="language-bash">touch /usr/share/bind-mount.txt
exit</code></pre>
<p>호스트에서 확인한다.</p>
<pre><code class="language-bash">ls bind-mount.txt
docker inspect busybox --format '{{json .Mounts}}' | jq
docker rm busybox</code></pre>
<p>bind mount는 개발 중 소스를 즉시 공유하기 좋지만, 호스트의 디렉터리 구조와 권한에 영향을 받는다.</p>
<h2 id="55-mariadb-데이터-영속화">5.5 MariaDB 데이터 영속화</h2>
<pre><code class="language-bash">mkdir -p db-data

docker run -d \
  --name mariadb \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=skala \
  -e MYSQL_USER=user \
  -e MYSQL_PASSWORD=password \
  --network skala \
  -p 3306:3306 \
  -v &quot;$(pwd)/db-data:/var/lib/mysql&quot; \
  mariadb:latest</code></pre>
<pre><code class="language-bash">ls ./db-data</code></pre>
<p>MariaDB가 <code>/var/lib/mysql</code>에 쓴 파일이 호스트 <code>db-data</code>에 저장된다. 이제 컨테이너를 교체해도 같은 디렉터리를 새 컨테이너에 연결하면 데이터를 다시 사용할 수 있다.</p>
<blockquote>
<p>DB 데이터 디렉터리를 직접 편집하지 않는다. 백업·복구는 DB가 지원하는 절차를 사용하는 것이 안전하다.</p>
</blockquote>
<h2 id="56-로컬-파일을-python-컨테이너와-공유">5.6 로컬 파일을 Python 컨테이너와 공유</h2>
<pre><code class="language-bash">mkdir -p mydata

docker run -d \
  --name linux-container \
  -p 8080:8080 \
  -p 8888:80 \
  -v &quot;$(pwd)/mydata:/mydata&quot; \
  linux-container:1.0

cp webserver.py ./mydata/
docker exec -it linux-container /bin/bash</code></pre>
<p>컨테이너 내부에서 <code>/mydata/webserver.py</code>가 보인다. 교재 실습은 컨테이너 안에서 Python을 수동 설치하고 웹 서버를 실행한다.</p>
<pre><code class="language-bash">apt-get update
apt-get install -y python3 python3-pip
python3 /mydata/webserver.py</code></pre>
<p>브라우저에서 <code>http://localhost:8080</code>으로 접속한다.</p>
<p>이 방식은 원리를 확인하기에는 좋지만 매번 수동 설치해야 한다. 그 작업을 Dockerfile로 옮긴다.</p>
<pre><code class="language-dockerfile">FROM ubuntu:22.04

RUN apt-get update &amp;&amp; \
    apt-get install -y python3 python3-pip &amp;&amp; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY webserver.py .
EXPOSE 8080
CMD [&quot;python3&quot;, &quot;webserver.py&quot;]</code></pre>
<pre><code class="language-bash">docker build --tag linux-container:1.0 .
docker run -d --name linux-container -p 8080:8080 linux-container:1.0</code></pre>
<h2 id="57-왜-pid-1이-중요한가">5.7 왜 PID 1이 중요한가?</h2>
<p>Docker는 컨테이너를 멈출 때 주 프로세스인 PID 1에 먼저 <code>SIGTERM</code>을 보낸다. 애플리케이션은 이 신호를 받아 다음 일을 정리할 수 있다.</p>
<ul>
<li>새 요청 수신 중단</li>
<li>처리 중인 요청 완료</li>
<li>DB 연결과 파일 닫기</li>
<li>메모리 상태나 로그 정리</li>
<li>정상 종료</li>
</ul>
<p>정해진 시간 안에 끝나지 않으면 Docker는 강제 종료 신호인 <code>SIGKILL</code>을 보낸다. 이 때문에 PID 1이 신호를 제대로 받아야 graceful shutdown이 가능하다.</p>
<h2 id="58-sigterm-처리-예시">5.8 SIGTERM 처리 예시</h2>
<pre><code class="language-python">import signal
import sys
import time
from datetime import datetime

def handle_sigterm(signum, frame):
    print(f&quot;[{datetime.now()}] SIGTERM 수신&quot;)
    time.sleep(1)  # 처리 중인 작업 정리 예시
    print(f&quot;[{datetime.now()}] 정상 종료&quot;)
    sys.exit(0)

signal.signal(signal.SIGTERM, handle_sigterm)</code></pre>
<p>Spring Boot, FastAPI 등 많은 프레임워크도 종료 신호를 받았을 때 graceful shutdown을 수행하도록 설정할 수 있다.</p>
<h2 id="59-cmd의-세-실행-방식">5.9 CMD의 세 실행 방식</h2>
<h3 id="a-exec-form-권장">A. Exec form: 권장</h3>
<pre><code class="language-dockerfile">CMD [&quot;python3&quot;, &quot;webserver.py&quot;]</code></pre>
<pre><code class="language-text">PID 1: python3 webserver.py</code></pre>
<p>애플리케이션이 PID 1이므로 Docker의 SIGTERM을 직접 받는다.</p>
<h3 id="b-shell-form-일반적으로-비권장">B. Shell form: 일반적으로 비권장</h3>
<pre><code class="language-dockerfile">CMD [&quot;/bin/sh&quot;, &quot;-c&quot;, &quot;python3 -u webserver.py&quot;]</code></pre>
<p>또는 다음과 같다.</p>
<pre><code class="language-dockerfile">CMD python3 -u webserver.py</code></pre>
<pre><code class="language-text">PID 1: /bin/sh -c python3 -u webserver.py
└─ child: python3 -u webserver.py</code></pre>
<p>셸이 PID 1이고 Python은 자식이다. 셸이 신호를 자식에게 적절히 전달하지 않으면 앱이 SIGTERM을 받지 못해 강제 종료될 수 있다.</p>
<h3 id="c-shell-with-exec-조건부-권장">C. Shell with <code>exec</code>: 조건부 권장</h3>
<pre><code class="language-dockerfile">CMD [&quot;/bin/sh&quot;, &quot;-c&quot;, &quot;exec python3 -u webserver.py&quot;]</code></pre>
<p>환경변수 확장 같은 셸 기능이 꼭 필요하다면 셸이 마지막에 <code>exec</code>로 자기 자신을 Python 프로세스로 치환하게 한다.</p>
<pre><code class="language-text">처음: /bin/sh
 exec 이후: PID 1 자리를 python3가 이어받음</code></pre>
<h2 id="510-세-방식-직접-비교하기">5.10 세 방식 직접 비교하기</h2>
<pre><code class="language-bash">docker build --tag linux-container:1.0 .
docker run --rm -d --name linux-container linux-container:1.0
docker exec -it linux-container /bin/bash
ps -ef
exit</code></pre>
<p>다른 터미널에서 로그를 보고 종료한다.</p>
<pre><code class="language-bash">docker logs -f linux-container</code></pre>
<pre><code class="language-bash">docker stop linux-container</code></pre>
<p>Exec form과 shell-with-exec에서는 앱이 PID 1이며 SIGTERM 처리 로그가 보여야 한다. 단순 shell form에서는 <code>/bin/sh</code>가 PID 1인 것을 확인할 수 있다.</p>
<h2 id="511--i--t의-정확한-의미">5.11 <code>-i</code>, <code>-t</code>의 정확한 의미</h2>
<ul>
<li><code>-i</code>, <code>--interactive</code>: 표준 입력(STDIN)을 열어 둔다.</li>
<li><code>-t</code>, <code>--tty</code>: 가상 터미널을 할당한다.</li>
<li><code>-it</code>: 사람이 컨테이너 셸과 대화할 때 보통 함께 쓴다.</li>
</ul>
<pre><code class="language-bash">docker exec -it linux-container /bin/bash</code></pre>
<p>컨테이너를 백그라운드 서비스로 실행하는 <code>-d</code>와, 사람이 터미널로 상호작용하는 <code>-it</code>는 목적이 다르다.</p>
<h3 id="5교시-체크포인트">5교시 체크포인트</h3>
<ul>
<li>중요한 데이터는 컨테이너의 쓰기 레이어에만 두지 않는다.</li>
<li>운영 데이터는 named volume, 개발 소스·설정 공유는 bind mount가 흔하다.</li>
<li>컨테이너의 PID 1이 종료 신호를 받아야 정상 종료가 가능하다.</li>
<li>특별한 이유가 없다면 JSON 배열 형태의 exec form을 사용한다.</li>
</ul>
<hr />
<h1 id="6-나의-서비스-컨테이너-만들기">6. 나의 서비스 컨테이너 만들기</h1>
<h2 id="61-최종-실습-시나리오">6.1 최종 실습 시나리오</h2>
<ol>
<li>Spring Boot Backend 이미지를 빌드하고 실행한다.</li>
<li><code>skala</code> bridge 네트워크로 Backend와 MariaDB를 연결한다.</li>
<li>정적 Frontend 이미지를 빌드하고 Nginx에서 실행한다.</li>
<li>Vue.js를 multi-stage로 빌드해 Nginx 이미지로 실행한다.</li>
<li>브라우저 → Frontend → Backend → MariaDB 전체 흐름을 확인한다.</li>
</ol>
<h2 id="62-3계층-구조-이해">6.2 3계층 구조 이해</h2>
<h3 id="frontend">Frontend</h3>
<ul>
<li>Nginx가 HTML, CSS, JavaScript, 이미지 같은 정적 파일을 제공한다.</li>
<li><code>/api</code> 요청을 Backend 컨테이너로 프록시한다.</li>
</ul>
<h3 id="backend">Backend</h3>
<ul>
<li>Spring Boot가 비즈니스 로직과 REST API를 제공한다.</li>
<li>MariaDB에 데이터를 저장하고 조회한다.</li>
</ul>
<h3 id="database">Database</h3>
<ul>
<li>MariaDB가 영속 데이터를 관리한다.</li>
<li>외부 공개가 필요 없다면 운영 환경에서는 DB 포트를 호스트에 열지 않고 내부 네트워크에서만 연결하는 편이 안전하다.</li>
</ul>
<p>브라우저는 <code>spring-backend</code>라는 Docker 내부 이름을 알지 못한다. Nginx가 같은 Docker 네트워크 안에서 그 이름을 DNS처럼 찾아 요청을 전달한다.</p>
<h2 id="63-spring-boot-backend-이미지-만들기">6.3 Spring Boot Backend 이미지 만들기</h2>
<p>예제 위치:</p>
<pre><code class="language-text">00.sample-container/01.spring-backend-v1.0</code></pre>
<p>Dockerfile:</p>
<pre><code class="language-dockerfile">FROM eclipse-temurin:21-jre

WORKDIR /app
ENV JAVA_OPTS=&quot;-Xms256m -Xmx512m&quot;

COPY ./target/*.jar app.jar

EXPOSE 8080
CMD [&quot;sh&quot;, &quot;-c&quot;, &quot;exec java $JAVA_OPTS -jar app.jar&quot;]</code></pre>
<p>단순 로컬 파일 복사에는 <code>ADD</code>보다 <code>COPY</code>가 의도를 더 분명히 보여준다. 여기서는 <code>$JAVA_OPTS</code> 확장이 필요하므로 shell-with-exec를 사용했다.</p>
<p>JAR을 만든 뒤 이미지를 빌드한다.</p>
<pre><code class="language-bash">mvn clean install -DskipTests
docker buildx build --tag spring-backend:1.0 .</code></pre>
<h2 id="64-mariadb와-backend-연결">6.4 MariaDB와 Backend 연결</h2>
<p>먼저 상태를 확인한다.</p>
<pre><code class="language-bash">docker ps
docker network ls
docker network inspect skala</code></pre>
<p>Backend를 같은 네트워크에서 실행한다.</p>
<pre><code class="language-bash">docker run -d \
  --name spring-backend \
  --network skala \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=local-mariadb \
  spring-backend:1.0</code></pre>
<pre><code class="language-bash">docker ps
docker logs -f spring-backend</code></pre>
<p>같은 <code>skala</code> 네트워크 안에서는 Backend가 DB 호스트 주소로 <code>localhost</code>가 아니라 <code>mariadb</code>를 사용해야 한다.</p>
<blockquote>
<p>컨테이너 안의 <code>localhost</code>는 그 컨테이너 자신이다. Backend 컨테이너에서 <code>localhost:3306</code>은 MariaDB가 아니라 Backend 컨테이너의 3306번 포트를 뜻한다.</p>
</blockquote>
<p>브라우저에서 <code>http://localhost:8080</code>에 접속해 사용자 등록 기능을 확인하고, DBeaver에서 <code>skala.users</code> 테이블의 데이터가 생겼는지 확인한다.</p>
<h2 id="65-spring-boot도-multi-stage로-빌드하기">6.5 Spring Boot도 Multi-stage로 빌드하기</h2>
<p>호스트에 Maven이 없어도 컨테이너 안에서 JAR 빌드부터 실행 이미지 생성까지 할 수 있다.</p>
<pre><code class="language-dockerfile">FROM ubuntu:24.04 AS builder
RUN apt-get update &amp;&amp; \
    apt-get install -y openjdk-21-jdk maven &amp;&amp; \
    apt-get clean &amp;&amp; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src ./src
RUN mvn package -DskipTests -q

FROM eclipse-temurin:21-jre
WORKDIR /app
ENV SPRING_PROFILES_ACTIVE=local
COPY --from=builder /build/target/*.jar app.jar
ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;app.jar&quot;]</code></pre>
<p>최종 이미지에는 소스, JDK, Maven, Maven 캐시가 들어가지 않고 실행에 필요한 JRE와 JAR만 남는다.</p>
<h2 id="66-nginx의-정적-frontend-만들기">6.6 Nginx의 정적 Frontend 만들기</h2>
<p>Nginx 설정 <code>default.conf</code>:</p>
<pre><code class="language-nginx">server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
        expires -1;
        add_header Cache-Control &quot;no-store, no-cache, must-revalidate&quot;;
        add_header Pragma &quot;no-cache&quot;;
    }

    location /api {
        resolver 127.0.0.11 valid=10s;
        set $backend &quot;spring-backend:8080&quot;;
        proxy_pass http://$backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }
}</code></pre>
<ul>
<li><code>/</code> 요청은 정적 파일로 응답한다.</li>
<li>SPA의 경로를 새로고침해도 <code>index.html</code>로 돌아가도록 <code>try_files</code>를 설정한다.</li>
<li><code>/api</code> 요청은 <code>spring-backend:8080</code>으로 전달한다.</li>
<li><code>127.0.0.11</code>은 Docker의 내장 DNS를 사용하는 구성이다.</li>
</ul>
<p>Dockerfile:</p>
<pre><code class="language-dockerfile">FROM nginx:alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY default.conf /etc/nginx/conf.d/
COPY src/ /usr/share/nginx/html/

EXPOSE 80
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<pre><code class="language-bash">docker buildx build --tag frontend:1.0 .

docker run -d \
  --name frontend \
  --network skala \
  -p 9090:80 \
  frontend:1.0</code></pre>
<p>브라우저에서 <code>http://localhost:9090</code>으로 접속한다.</p>
<h2 id="67-vuejs-frontend를-multi-stage로-만들기">6.7 Vue.js Frontend를 Multi-stage로 만들기</h2>
<p>Vue 소스는 브라우저가 바로 실행할 수 있도록 먼저 정적 리소스로 빌드해야 한다.</p>
<pre><code class="language-text">Vue 소스(App.vue, main.js, ...)
        |
        | npm ci &amp;&amp; npm run build
        v
dist(index.html, JS, CSS, ...)
        |
        | Nginx 이미지에 복사
        v
Vue Frontend 컨테이너</code></pre>
<p>Dockerfile:</p>
<pre><code class="language-dockerfile">FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY default.conf /etc/nginx/conf.d/
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD [&quot;nginx&quot;, &quot;-g&quot;, &quot;daemon off;&quot;]</code></pre>
<pre><code class="language-bash">docker buildx build --tag vue-frontend:1.0 .

docker run -d \
  --name vue-frontend \
  --network skala \
  -p 8090:80 \
  vue-frontend:1.0</code></pre>
<p>브라우저에서 <code>http://localhost:8090</code>으로 접속한다.</p>
<h2 id="68-요청-한-번이-이동하는-경로">6.8 요청 한 번이 이동하는 경로</h2>
<p>사용자가 Vue 화면에서 사용자 등록 버튼을 누르면 다음 일이 일어난다.</p>
<ol>
<li>브라우저가 <code>http://localhost:8090/api/users</code>로 요청한다.</li>
<li>호스트 8090 포트가 Vue Frontend 컨테이너의 80 포트로 연결된다.</li>
<li>Nginx가 <code>/api</code> 요청을 <code>spring-backend:8080</code>으로 프록시한다.</li>
<li>Docker 내장 DNS가 <code>spring-backend</code>를 Backend 컨테이너 IP로 해석한다.</li>
<li>Spring Boot가 요청을 처리하고 <code>mariadb:3306</code>으로 SQL을 보낸다.</li>
<li>MariaDB가 데이터를 볼륨에 기록한다.</li>
<li>결과 JSON이 Backend → Nginx → 브라우저 순서로 돌아온다.</li>
</ol>
<h2 id="69-최종-확인-명령">6.9 최종 확인 명령</h2>
<pre><code class="language-bash">docker ps
docker network inspect skala
docker logs mariadb
docker logs spring-backend
docker logs frontend</code></pre>
<p>문제가 생기면 요청 경로의 앞에서부터 한 단계씩 확인한다.</p>
<ol>
<li>브라우저에서 Frontend가 열리는가?</li>
<li>Nginx 로그에 <code>/api</code> 요청이 보이는가?</li>
<li>Backend 컨테이너가 실행 중인가?</li>
<li>Backend 로그에 DB 연결 오류가 있는가?</li>
<li>두 컨테이너가 모두 <code>skala</code> 네트워크에 연결되어 있는가?</li>
<li>DB 이름, 사용자, 비밀번호, 프로필이 일치하는가?</li>
<li>DBeaver에서 실제 데이터가 조회되는가?</li>
</ol>
<hr />
<h1 id="7-오늘-반드시-구분해야-하는-개념">7. 오늘 반드시 구분해야 하는 개념</h1>
<h2 id="이미지-vs-컨테이너">이미지 vs 컨테이너</h2>
<table>
<thead>
<tr>
<th>이미지</th>
<th>컨테이너</th>
</tr>
</thead>
<tbody><tr>
<td>실행 전의 읽기 전용 템플릿</td>
<td>이미지를 실행한 프로세스 환경</td>
</tr>
<tr>
<td>하나로 여러 컨테이너 생성 가능</td>
<td>각자 상태와 쓰기 레이어를 가짐</td>
</tr>
<tr>
<td><code>docker images</code></td>
<td><code>docker ps -a</code></td>
</tr>
<tr>
<td>삭제: <code>docker rmi</code></td>
<td>삭제: <code>docker rm</code></td>
</tr>
</tbody></table>
<h2 id="run-vs-cmd"><code>RUN</code> vs <code>CMD</code></h2>
<table>
<thead>
<tr>
<th><code>RUN</code></th>
<th><code>CMD</code></th>
</tr>
</thead>
<tbody><tr>
<td>이미지 빌드 중 실행</td>
<td>컨테이너 시작 때 실행</td>
</tr>
<tr>
<td>결과가 이미지 레이어에 반영</td>
<td>기본 실행 설정으로 저장</td>
</tr>
<tr>
<td>패키지 설치, 빌드</td>
<td>서버 프로세스 시작</td>
</tr>
</tbody></table>
<h2 id="expose-vs--p"><code>EXPOSE</code> vs <code>-p</code></h2>
<table>
<thead>
<tr>
<th><code>EXPOSE 8080</code></th>
<th><code>-p 9090:8080</code></th>
</tr>
</thead>
<tbody><tr>
<td>이미지가 사용할 포트를 문서화</td>
<td>호스트와 컨테이너 포트를 실제 연결</td>
</tr>
<tr>
<td>이것만으로 외부 접속 불가</td>
<td><code>localhost:9090</code>으로 접근 가능</td>
</tr>
</tbody></table>
<h2 id="arg-vs-env"><code>ARG</code> vs <code>ENV</code></h2>
<table>
<thead>
<tr>
<th><code>ARG</code></th>
<th><code>ENV</code></th>
</tr>
</thead>
<tbody><tr>
<td>주로 빌드 시점</td>
<td>빌드와 런타임</td>
</tr>
<tr>
<td><code>--build-arg</code>로 전달</td>
<td><code>docker run -e</code>로 덮어쓰기</td>
</tr>
<tr>
<td>stage 범위에 주의</td>
<td>최종 이미지 설정에 남음</td>
</tr>
</tbody></table>
<h2 id="named-volume-vs-bind-mount">Named volume vs Bind mount</h2>
<table>
<thead>
<tr>
<th>Named volume</th>
<th>Bind mount</th>
</tr>
</thead>
<tbody><tr>
<td>Docker가 저장 위치 관리</td>
<td>사용자가 호스트 경로 지정</td>
</tr>
<tr>
<td>운영 데이터에 편리</td>
<td>개발 소스·설정 공유에 편리</td>
</tr>
<tr>
<td>호스트 구조 의존성이 낮음</td>
<td>호스트 권한·경로에 영향 받음</td>
</tr>
</tbody></table>
<hr />
<h1 id="8-초보자가-자주-하는-실수">8. 초보자가 자주 하는 실수</h1>
<h2 id="81-컨테이너끼리-localhost로-접속한다">8.1 컨테이너끼리 <code>localhost</code>로 접속한다</h2>
<p>각 컨테이너의 <code>localhost</code>는 자기 자신이다. 같은 사용자 정의 네트워크에서 상대 컨테이너 이름을 사용한다.</p>
<pre><code class="language-text">잘못된 DB 주소: localhost:3306
올바른 DB 주소: mariadb:3306</code></pre>
<h2 id="82-expose만-쓰면-외부-접속된다고-생각한다">8.2 <code>EXPOSE</code>만 쓰면 외부 접속된다고 생각한다</h2>
<p><code>EXPOSE</code>는 설명 정보다. <code>docker run -p 호스트:컨테이너</code>가 필요하다.</p>
<h2 id="83-서버를-백그라운드로-보내-컨테이너가-종료된다">8.3 서버를 백그라운드로 보내 컨테이너가 종료된다</h2>
<p>컨테이너는 메인 프로세스가 끝나면 종료된다. Nginx는 <code>daemon off;</code>처럼 foreground에서 실행한다.</p>
<h2 id="84-실행-중인-컨테이너를-이미지처럼-수정한다">8.4 실행 중인 컨테이너를 이미지처럼 수정한다</h2>
<p>컨테이너 안에서 수동 설치한 내용은 재현하기 어렵다. 필요한 변경은 Dockerfile에 기록하고 이미지를 다시 빌드한다.</p>
<h2 id="85-latest만-사용한다">8.5 <code>latest</code>만 사용한다</h2>
<p>시간이 지나 다른 이미지가 받아질 수 있다. 중요한 환경은 검증한 버전 태그나 digest로 고정한다.</p>
<h2 id="86-비밀번호를-dockerfile에-기록한다">8.6 비밀번호를 Dockerfile에 기록한다</h2>
<p>이미지 기록과 registry를 통해 노출될 수 있다. 실행 환경의 secret 관리 방식을 사용한다. 오늘 실습의 <code>password</code>는 로컬 학습용 값일 뿐이다.</p>
<h2 id="87-docker-system-prune--a를-습관적으로-실행한다">8.7 <code>docker system prune -a</code>를 습관적으로 실행한다</h2>
<p>다른 프로젝트가 다시 받아야 할 이미지와 빌드 캐시까지 삭제할 수 있다. 볼륨 정리는 실제 데이터 삭제 가능성을 특히 주의한다.</p>
<h2 id="88-shell-form으로-서버를-실행한다">8.8 shell form으로 서버를 실행한다</h2>
<p>셸이 PID 1이 되어 종료 신호가 앱에 전달되지 않을 수 있다. 기본은 exec form, 셸 기능이 필요하면 <code>exec</code> 치환을 사용한다.</p>
<hr />
<h1 id="9-1일차-전체-실습-완료-체크리스트">9. 1일차 전체 실습 완료 체크리스트</h1>
<ul>
<li><input disabled="" type="checkbox" /> Docker Desktop이 실행되고 <code>docker info</code>가 성공한다.</li>
<li><input disabled="" type="checkbox" /> 실습 저장소를 내려받았다.</li>
<li><input disabled="" type="checkbox" /> VM과 컨테이너의 구조 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 이미지와 컨테이너의 차이를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> <code>skala</code> bridge 네트워크를 만들었다.</li>
<li><input disabled="" type="checkbox" /> MariaDB 컨테이너를 실행하고 SQL을 입력했다.</li>
<li><input disabled="" type="checkbox" /> DBeaver에서 MariaDB에 접속했다.</li>
<li><input disabled="" type="checkbox" /> <code>pull</code>, <code>run</code>, <code>ps</code>, <code>exec</code>, <code>stop</code>, <code>rm</code>, <code>rmi</code>를 구분한다.</li>
<li><input disabled="" type="checkbox" /> bind mount를 통해 호스트와 컨테이너가 파일을 공유했다.</li>
<li><input disabled="" type="checkbox" /> Dockerfile로 Nginx 또는 Python 이미지를 빌드했다.</li>
<li><input disabled="" type="checkbox" /> <code>FROM</code>, <code>RUN</code>, <code>ARG</code>, <code>LABEL</code>, <code>COPY</code>, <code>WORKDIR</code>, <code>USER</code>, <code>ENV</code>, <code>EXPOSE</code>, <code>CMD</code>, <code>ENTRYPOINT</code>를 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> Multi-stage build의 목적을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> named volume, bind mount, anonymous volume의 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> Exec form에서 애플리케이션이 PID 1인 것을 확인했다.</li>
<li><input disabled="" type="checkbox" /> Spring Boot, Frontend, MariaDB를 같은 네트워크로 연결했다.</li>
<li><input disabled="" type="checkbox" /> 브라우저에서 사용자 등록 후 DB 데이터까지 확인했다.</li>
</ul>
<hr />
<h1 id="10-마무리-오늘-배운-내용을-한-문장씩">10. 마무리: 오늘 배운 내용을 한 문장씩</h1>
<ol>
<li><strong>컨테이너는 작은 VM이 아니라, 호스트 커널을 공유하며 격리된 애플리케이션 프로세스 환경이다.</strong></li>
<li><strong>이미지는 실행 템플릿이고 컨테이너는 그 이미지를 실행한 인스턴스다.</strong></li>
<li><strong>Dockerfile은 반복 가능한 이미지 제작 절차를 코드로 기록한다.</strong></li>
<li><strong>볼륨은 데이터의 수명을 컨테이너의 수명과 분리한다.</strong></li>
<li><strong>사용자 정의 bridge 네트워크에서는 컨테이너 이름으로 서로 통신할 수 있다.</strong></li>
<li><strong>PID 1과 종료 신호를 올바르게 처리해야 서비스가 안전하게 종료된다.</strong></li>
<li><strong>Multi-stage build는 빌드 도구를 최종 이미지에서 제외해 더 작고 안전한 이미지를 만든다.</strong></li>
<li><strong>Frontend, Backend, DB를 각 컨테이너로 분리하고 네트워크로 연결하면 하나의 서비스를 구성할 수 있다.</strong></li>
</ol>
<p>오늘의 핵심은 명령어 암기가 아니다. <strong>이미지를 만들고, 컨테이너로 실행하고, 네트워크로 연결하고, 볼륨으로 데이터를 지키며, 종료 신호까지 안전하게 처리하는 전체 흐름</strong>을 머릿속에 그리는 것이다.</p>