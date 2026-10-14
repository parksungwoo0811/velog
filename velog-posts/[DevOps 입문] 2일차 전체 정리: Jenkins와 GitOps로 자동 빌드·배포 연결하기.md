<blockquote>
<p>코드를 바꿀 때마다 사람이 같은 작업을 반복한다면, 그 순서를 문서로 만들고 도구가 실행하게 할 수 있다. 단, 자동으로 실행되는 것과 올바르게 완료되는 것은 구분해야 한다.</p>
</blockquote>
<p>1일차에는 컨테이너 이미지를 만들고 Kubernetes에 배포할 설정을 작성했다. 2일차에는 이 작업을 <strong>Jenkins로 연결하고, Git에 기록한 배포 설정을 Argo CD가 실제 환경에 반영하도록 만드는 과정</strong>을 배운다.</p>
<p>이 글은 「DevOps 이해 및 활용 v2.1」 교재의 2일차 범위를 초보자를 위한 공개용 설명으로 다시 구성한 글이다. 교재의 본문·도표에서 다루는 학습 주제를 빠짐없이 설명하되, 슬라이드와 실습 코드를 그대로 복제하지 않고 새로운 비유와 예제를 사용한다. 수업 시간표, 개인 식별 정보, 내부 접속 주소, 공용 계정, 실제 비밀번호·토큰·인증 파일은 포함하지 않는다.</p>
<p>처음 읽는 사람을 위해 <strong>학교 도서관 서비스</strong>를 예로 사용한다. 학생은 책을 검색하고, 서버는 도서 목록을 돌려준다. 이 서비스에 새 기능을 추가할 때마다 자동으로 검사하고 배포한다고 생각하자.</p>
<p>예제의 <code>registry.example.com</code>, <code>ci.example.com</code>과 <code>YOUR_TEAM</code>은 자신의 환경에 맞춰 바꿀 자리다. 제공되지 않은 주소나 저장소가 실제로 작동한다고 가정하지 않는다. 예제 실행에는 해당 도구·플러그인·권한·프로젝트 파일이 필요하다. 실행을 나타내는 코드와 흐름을 설명하는 코드 조각은 본문에서 구분한다.</p>
<h2 id="목차">목차</h2>
<ol>
<li>Jenkins CI/CD (1): 기본 개념과 Jenkinsfile</li>
<li>Jenkins CI/CD (2): 실제 빌드·업로드·배포 연결</li>
<li>Jenkins CI &amp; Jenkins Poll SCM</li>
<li>Kubernetes에 Jenkins 배포 및 실행하기</li>
<li>GitOps 기반 CD 구성하기</li>
<li>나만의 파이프라인 구성하기</li>
<li>퀴즈: 2일차 복습</li>
<li>조별 토론하기</li>
</ol>
<hr />
<h2 id="1-jenkins-cicd-1-기본-개념과-jenkinsfile">1. Jenkins CI/CD (1): 기본 개념과 Jenkinsfile</h2>
<h3 id="1-1-먼저-1일차와-연결하자">1-1. 먼저 1일차와 연결하자</h3>
<p>앱을 배포할 때 사람이 다음 작업을 했다고 생각해 보자.</p>
<pre><code class="language-text">Git에서 코드 가져오기
    ↓
앱 빌드·테스트
    ↓
컨테이너 이미지 만들기
    ↓
레지스트리에 업로드
    ↓
배포 설정에 새 이미지 지정
    ↓
Kubernetes 적용·상태 확인</code></pre>
<p>한 번은 직접 해 볼 수 있지만 매번 반복하면 명령 누락, 경로 착각, 태그 불일치가 생길 수 있다. 자동화는 이 작업을 <strong>정해진 순서와 조건으로 실행하도록 만드는 것</strong>이다.</p>
<p>단, 잘못된 명령을 자동화하면 잘못된 작업도 자동으로 반복된다. 그래서 자동화 전에 각 단계가 무엇을 입력받고 무엇을 만드는지 이해해야 한다.</p>
<h3 id="1-2-jenkins는-작업을-진행하는-자동화-담당자다">1-2. Jenkins는 작업을 진행하는 자동화 담당자다</h3>
<p>Jenkins는 빌드·테스트·배포 같은 반복 작업을 실행하고 그 결과를 기록하는 자동화 서버다.</p>
<p>급식실에 비유하면 다음과 같다.</p>
<ul>
<li>Git 저장소: 조리법 보관함.</li>
<li>빌드 도구: 조리 도구.</li>
<li>테스트: 음식 검사.</li>
<li>레지스트리: 완성된 제품 보관 창고.</li>
<li>Jenkins: 각 담당자에게 순서대로 일을 시키고 결과를 확인하는 진행 담당자.</li>
</ul>
<p>Jenkins가 직접 모든 언어를 컴파일하는 것은 아니다. Maven, Python, Docker, Buildah 같은 도구를 실행한다. 따라서 <strong>Jenkins가 설치된 것과 필요한 빌드 도구가 준비된 것은 별개</strong>다.</p>
<p>Java 기반으로 여러 운영체제에서 실행할 수 있고, 플러그인으로 Git·컨테이너·Kubernetes 등과 연결할 수 있다. 구체적인 Java 지원 버전과 플러그인 호환성은 사용하는 Jenkins 버전에 맞춰 확인한다.</p>
<h3 id="1-3-controller와-agent">1-3. Controller와 Agent</h3>
<table>
<thead>
<tr>
<th>구성</th>
<th>쉬운 설명</th>
<th>주요 역할</th>
</tr>
</thead>
<tbody><tr>
<td>Controller</td>
<td>작업 접수·관리 담당</td>
<td>Job 설정, 실행 예약, 사용자 화면, 결과 관리</td>
</tr>
<tr>
<td>Agent</td>
<td>실제 작업을 하는 작업자</td>
<td>코드 checkout, 빌드, 테스트, 이미지 생성</td>
</tr>
<tr>
<td>Executor</td>
<td>작업자가 동시에 처리할 수 있는 작업 자리</td>
<td>실행할 빌드를 받아 처리</td>
</tr>
<tr>
<td>Workspace</td>
<td>해당 작업의 책상</td>
<td>소스 코드와 작업 파일이 놓이는 디렉터리</td>
</tr>
</tbody></table>
<pre><code class="language-text">Jenkins Controller
  ├─ Java 빌드 Agent
  ├─ Python 테스트 Agent
  └─ 컨테이너 이미지 빌드 Agent</code></pre>
<p>Controller가 떠 있어도 실행 가능한 Agent와 Executor가 없으면 작업은 대기할 수 있다.</p>
<p>교재의 실습에서는 Controller에서 직접 빌드하는 구성을 만날 수 있다. 구조 이해를 위한 단순한 실습 방식이다. 실제 운영에서는 작업 부하와 보안을 고려해 빌드 작업을 별도 Agent에 맡기는 구성을 검토한다.</p>
<h3 id="1-4-job-build-stage-step">1-4. Job, Build, Stage, Step</h3>
<table>
<thead>
<tr>
<th>용어</th>
<th>의미</th>
<th>도서관 예시</th>
</tr>
</thead>
<tbody><tr>
<td>Job</td>
<td>등록해 둔 작업 정의</td>
<td>도서관 API 빌드 작업</td>
</tr>
<tr>
<td>Build / Run</td>
<td>그 Job의 한 번의 실행</td>
<td>해당 작업의 실행 12번</td>
</tr>
<tr>
<td>Pipeline</td>
<td>연결된 자동화 흐름</td>
<td>가져오기 → 검사 → 이미지 생성</td>
</tr>
<tr>
<td>Stage</td>
<td>흐름의 큰 단계</td>
<td>테스트 단계</td>
</tr>
<tr>
<td>Step</td>
<td>단계 안의 실제 명령</td>
<td><code>sh './mvnw test'</code></td>
</tr>
</tbody></table>
<p>같은 Job을 여러 번 실행할 수 있다. “작업 이름”과 “몇 번째 실행인지”는 다르다.</p>
<p>학교 운동회에 비유하면 Pipeline은 전체 행사 순서, Stage는 이어달리기나 줄다리기, Step은 실제로 출발 신호를 주거나 기록을 측정하는 행동이다.</p>
<h3 id="1-5-jenkinsfile-작업-순서를-코드로-적기">1-5. Jenkinsfile: 작업 순서를 코드로 적기</h3>
<p>Jenkinsfile은 파이프라인을 코드로 정의한 파일이다. 보통 Git 저장소에 함께 기록한다.</p>
<pre><code class="language-text">library-api/
├── src/
├── pom.xml
├── mvnw
├── Dockerfile
└── Jenkinsfile</code></pre>
<p>코드로 관리하면 변경 비교와 리뷰가 가능하고, 누가 어떤 빌드 절차를 바꿨는지 추적하기 쉽다.</p>
<p>Jenkins Pipeline에는 Declarative와 Scripted 방식이 있다. 이 글에서는 구조가 눈에 잘 보이는 Declarative 방식을 중심으로 설명한다. Jenkinsfile은 Groovy 기반이며 Jenkins가 제공하는 DSL을 사용한다.</p>
<h3 id="1-6-dsl은-특정-일을-쉽게-표현하는-언어다">1-6. DSL은 특정 일을 쉽게 표현하는 언어다</h3>
<p>DSL은 Domain-Specific Language, 특정 분야를 위한 언어다.</p>
<p>식당 주문서의 “메뉴·수량·추가 요청”은 일반 소설 문장보다 주문을 표현하기 쉽다. Jenkins DSL도 자동화 흐름을 표현하기 위해 <code>pipeline</code>, <code>stage</code>, <code>steps</code> 같은 요소를 제공한다.</p>
<pre><code class="language-groovy">pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo '도서관 파이프라인을 시작합니다.'
            }
        }
    }
}</code></pre>
<p>이 예제는 Pipeline 플러그인과 실행 가능한 Agent가 있는 Jenkins에서 등록할 수 있는 최소 파이프라인이다. 실제 앱을 빌드하지 않고 로그만 출력한다.</p>
<h3 id="1-7-agent-어디에서-실행할까">1-7. agent: 어디에서 실행할까?</h3>
<table>
<thead>
<tr>
<th>표현</th>
<th>뜻</th>
</tr>
</thead>
<tbody><tr>
<td><code>agent any</code></td>
<td>사용 가능한 실행 노드에 작업 배정</td>
</tr>
<tr>
<td><code>agent none</code></td>
<td>전체 실행 노드를 잡지 않고 Stage마다 지정</td>
</tr>
<tr>
<td><code>agent { label 'ci-java' }</code></td>
<td>해당 label 조건을 만족하는 노드 사용</td>
</tr>
<tr>
<td><code>agent { docker { image '...' } }</code></td>
<td>준비된 노드에서 Docker 컨테이너 환경 사용</td>
</tr>
</tbody></table>
<p><code>agent any</code>는 모든 노드에 동시에 실행한다는 뜻이 아니다. 실행 가능한 노드에 작업을 배정하는 방식이다.</p>
<p>Docker Agent도 “Jenkins만 있으면 자동으로 Docker가 생긴다”는 뜻이 아니다. 노드에 Docker 접근 환경이 준비되어 있어야 한다.</p>
<h3 id="1-8-environment-이름표를-붙여-값-전달하기">1-8. environment: 이름표를 붙여 값 전달하기</h3>
<pre><code class="language-groovy">pipeline {
    agent any

    environment {
        APP_NAME = 'library-api'
        DEPLOY_ENV = 'dev'
    }

    stages {
        stage('Show settings') {
            environment {
                STAGE_MESSAGE = '이 단계에서만 사용하는 값'
            }
            steps {
                echo &quot;앱: ${env.APP_NAME}&quot;
                echo &quot;환경: ${env.DEPLOY_ENV}&quot;
                echo &quot;설명: ${env.STAGE_MESSAGE}&quot;
            }
        }
    }
}</code></pre>
<p>최상위 <code>environment</code>는 전체 파이프라인에서 사용한다. Stage 내부의 환경 설정은 해당 단계에서 사용한다.</p>
<p>여러 단계로 전달할 값을 실행 중에 만들 때는 다음처럼 <code>env</code>에 넣을 수 있다. 아래는 <code>steps</code> 안에 들어가는 조각이다.</p>
<pre><code class="language-groovy">script {
    env.IMAGE_TAG = &quot;build-${env.BUILD_NUMBER}&quot;
}</code></pre>
<p><code>BUILD_NUMBER</code>는 해당 Job의 실행 번호다. 다른 Job에서도 같은 번호가 나올 수 있으므로 이것만으로 모든 프로젝트의 전역 고유값이 되는 것은 아니다.</p>
<p>단순한 Groovy 지역 변수와 <code>env.IMAGE_TAG</code>를 혼동하지 않는다. Shell에 전달하거나 단계 사이에서 사용할 값이라면 어떤 범위에서 존재하는지 확인한다.</p>
<h3 id="1-9-작은따옴표와-큰따옴표-누가-값을-바꿀까">1-9. 작은따옴표와 큰따옴표: 누가 값을 바꿀까?</h3>
<pre><code class="language-groovy">echo &quot;앱 이름: ${env.APP_NAME}&quot;
echo '$APP_NAME'
sh 'echo &quot;$APP_NAME&quot;'</code></pre>
<table>
<thead>
<tr>
<th>코드</th>
<th>처리</th>
</tr>
</thead>
<tbody><tr>
<td><code>echo &quot;앱 이름: ${env.APP_NAME}&quot;</code></td>
<td>Groovy가 값을 치환</td>
</tr>
<tr>
<td><code>echo '$APP_NAME'</code></td>
<td>글자 <code>$APP_NAME</code>을 그대로 출력</td>
</tr>
<tr>
<td><code>sh 'echo &quot;$APP_NAME&quot;'</code></td>
<td>Groovy는 그대로 전달하고 Shell이 환경변수 치환</td>
</tr>
</tbody></table>
<p>Jenkinsfile 바깥에서 큰따옴표를 쓰느냐, Shell 안에서 따옴표를 쓰느냐는 서로 다른 문제다.</p>
<p>특히 비밀번호가 있는 Shell 명령에서는 Groovy의 문자열 보간으로 비밀값을 명령문에 먼저 끼워 넣지 않도록 주의한다. 실제 비밀값을 로그로 확인하지 않는다. <a href="https://www.jenkins.io/doc/book/pipeline/jenkinsfile/">Jenkinsfile의 문자열·인증정보 처리</a></p>
<h3 id="1-10-options-작업의-운영-규칙">1-10. options: 작업의 운영 규칙</h3>
<pre><code class="language-groovy">options {
    timeout(time: 10, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
}</code></pre>
<p>위 코드는 <code>pipeline</code> 안의 옵션 조각이다.</p>
<table>
<thead>
<tr>
<th>옵션</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>timeout</code></td>
<td>작업이 계속 멈춰 있지 않도록 제한</td>
</tr>
<tr>
<td><code>buildDiscarder</code></td>
<td>오래된 실행 기록 등의 보존 정책 설정</td>
</tr>
<tr>
<td><code>disableConcurrentBuilds()</code></td>
<td>같은 Job의 동시 실행 제한</td>
</tr>
<tr>
<td><code>retry</code></td>
<td>특정 실패 작업을 다시 실행</td>
</tr>
<tr>
<td><code>timestamps()</code></td>
<td>로그에 시각 정보 추가. 관련 플러그인 필요</td>
</tr>
</tbody></table>
<p><code>disableConcurrentBuilds()</code>는 서로 다른 모든 Job의 동시 실행을 막는 옵션은 아니다. 다른 Job이 같은 이미지를 업로드하거나 같은 배포 저장소를 수정한다면 별도 조정이 필요하다.</p>
<p><code>retry</code>는 실패한 작업을 다시 시도하는 기능이다. <code>retry(3) { ... }</code>의 Step은 해당 블록을 최초 시도를 포함해 최대 3번 실행하는 의미다. 이미 외부 시스템에 반영된 작업을 다시 수행해도 안전한지 확인해야 한다. 파이프라인 전체 옵션으로 사용하는 경우에는 다시 실행되는 범위도 구분한다. <a href="https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/#retry-retry-the-body-up-to-n-times">retry Step 공식 설명</a></p>
<h3 id="1-11-post-결과에-따라-마지막-처리">1-11. post: 결과에 따라 마지막 처리</h3>
<pre><code class="language-groovy">post {
    always {
        echo '작업이 끝났습니다.'
    }
    success {
        echo '성공 조건을 통과했습니다.'
    }
    failure {
        echo '실패한 단계의 로그를 확인하세요.'
    }
    aborted {
        echo '작업이 중단되었습니다.'
    }
}</code></pre>
<p><code>always</code>는 성공 여부와 관계없이, <code>success</code>는 성공 시, <code>failure</code>는 실패 시 실행한다. Pipeline 전체와 특정 Stage 뒤에 각각 처리 규칙을 둘 수 있다.</p>
<p>로그 정리, 테스트 결과 수집, 알림 등에도 활용한다. 실제 외부 알림 발송은 받을 사람과 전송 내용을 정해 설정하는 별도 작업이다.</p>
<h3 id="1-12-자주-나오는-step-전체-훑어보기">1-12. 자주 나오는 Step 전체 훑어보기</h3>
<table>
<thead>
<tr>
<th>Step</th>
<th>역할</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>echo</code></td>
<td>로그 출력</td>
<td><code>echo '시작'</code></td>
</tr>
<tr>
<td><code>sh</code></td>
<td>Unix 계열 Shell 명령 실행</td>
<td><code>sh './mvnw test'</code></td>
</tr>
<tr>
<td><code>checkout</code></td>
<td>지정된 SCM 설정으로 코드 가져오기</td>
<td><code>checkout scm</code></td>
</tr>
<tr>
<td><code>git</code></td>
<td>간단한 Git checkout</td>
<td>branch·url·credentialsId 지정</td>
</tr>
<tr>
<td><code>timeout</code></td>
<td>블록 실행 제한</td>
<td>오래 걸리는 작업 중단</td>
</tr>
<tr>
<td><code>retry</code></td>
<td>실패 작업 반복 시도</td>
<td>일시적 네트워크 오류 대응</td>
</tr>
<tr>
<td><code>sleep</code></td>
<td>실행 잠시 대기</td>
<td>비동기 작업 대기</td>
</tr>
<tr>
<td><code>withCredentials</code></td>
<td>인증정보를 필요한 블록에 주입</td>
<td>저장소 로그인</td>
</tr>
<tr>
<td><code>junit</code></td>
<td>JUnit 형식 테스트 결과 게시</td>
<td>Maven 테스트 리포트</td>
</tr>
<tr>
<td><code>archiveArtifacts</code></td>
<td>실행 결과 파일 보관</td>
<td>JAR·리포트 보관</td>
</tr>
</tbody></table>
<p>Step에는 기본 제공 기능과 플러그인이 제공하는 기능이 있다. <code>checkout scm</code>의 <code>scm</code>도 모든 Job에 무조건 존재하는 것은 아니다. SCM에서 Jenkinsfile을 가져오는 Job 등에 맞는 설정이 필요하다.</p>
<p>다음처럼 괄호를 생략한 호출을 볼 수 있다.</p>
<pre><code class="language-groovy">git branch: 'main', url: 'https://github.com/YOUR_TEAM/library-api.git'</code></pre>
<p>이는 이름 있는 인자를 전달하는 Groovy DSL 표현이다. <code>{ ... }</code>로 넘기는 블록은 Closure라고 부르며, Step이 블록 안의 작업을 관리하는 방식으로 이해하면 된다.</p>
<p>Pipeline의 각 요소와 옵션은 <a href="https://www.jenkins.io/doc/book/pipeline/syntax/">공식 문법 안내</a>에서 확인할 수 있다.</p>
<p>그 밖에 교재에서 소개하는 <code>parameters</code>, <code>when</code>, <code>script</code>도 알아 두자.</p>
<ul>
<li><code>parameters</code>: 빌드를 시작할 때 받을 입력 정의. 예를 들어 개발·운영 환경을 선택한다.</li>
<li><code>when</code>: 해당 Stage를 실행할 조건. 브랜치·파라미터 등을 확인할 수 있다.</li>
<li><code>script</code>: Declarative 구조 안에서 Groovy 계산·조건 처리 등을 작성할 블록.</li>
<li><code>tools</code>: Jenkins에 설정한 JDK·Maven 등 도구 구성을 선택하는 지시문.</li>
</ul>
<p>아래는 Pipeline 안의 입력 정의와 조건 Stage를 설명하는 조각이다. 단순 입력 선택으로 운영 배포 승인까지 확보되는 것은 아니다.</p>
<pre><code class="language-groovy">parameters {
    choice(name: 'TARGET_ENV', choices: ['dev', 'prod'], description: '배포 환경')
}

// stages 안에 넣는 조건 Stage
stage('Production Check') {
    when {
        expression { params.TARGET_ENV == 'prod' }
    }
    steps {
        echo '운영 배포에 필요한 별도 검토를 진행합니다.'
    }
}</code></pre>
<p><code>when { branch 'main' }</code> 같은 브랜치 조건은 보통 Multibranch Pipeline의 브랜치 정보를 사용한다. 단일 Pipeline의 환경과 같다고 가정하지 않는다.</p>
<h3 id="1-13-플러그인은-필요한-기능을-추가하는-부품이다">1-13. 플러그인은 필요한 기능을 추가하는 부품이다</h3>
<p>교재에서 소개하는 플러그인은 역할별로 나눠 이해하자. 모두를 설치해야 모든 파이프라인이 작동하는 것은 아니다.</p>
<table>
<thead>
<tr>
<th>플러그인</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>workflow-aggregator</code></td>
<td>Pipeline 관련 기능 묶음</td>
</tr>
<tr>
<td><code>git</code></td>
<td>Git 저장소 연동</td>
</tr>
<tr>
<td><code>git-client</code></td>
<td>Git 실행 지원</td>
</tr>
<tr>
<td><code>github-branch-source</code></td>
<td>GitHub 멀티브랜치·조직 저장소 연동</td>
</tr>
<tr>
<td><code>docker-workflow</code></td>
<td>Docker 기반 Pipeline 기능</td>
</tr>
<tr>
<td><code>pipeline-stage-view</code></td>
<td>단계별 진행 화면</td>
</tr>
<tr>
<td><code>blueocean</code></td>
<td>교재에서 소개하는 Pipeline UI</td>
</tr>
<tr>
<td><code>credentials-binding</code></td>
<td>인증정보를 변수·파일로 전달</td>
</tr>
<tr>
<td><code>timestamper</code></td>
<td>로그 시각 표시</td>
</tr>
<tr>
<td><code>job-dsl</code></td>
<td>Job·Folder 등을 코드로 정의</td>
</tr>
<tr>
<td><code>configuration-as-code</code></td>
<td>Jenkins 설정의 코드 관리</td>
</tr>
<tr>
<td><code>kubernetes</code></td>
<td>Kubernetes Pod를 Agent로 활용</td>
</tr>
<tr>
<td><code>maven-plugin</code></td>
<td>Maven 프로젝트·도구 연동</td>
</tr>
<tr>
<td><code>junit</code></td>
<td>테스트 결과 표시</td>
</tr>
<tr>
<td><code>htmlpublisher</code></td>
<td>HTML 리포트 게시</td>
</tr>
</tbody></table>
<p>플러그인을 설치해도 OS 실행 파일이 자동으로 모두 준비되는 것은 아니다. 예를 들어 Shell에서 <code>mvn</code>을 실행하려면 실행 노드에 Maven이 있어야 한다. UI 플러그인은 교재의 소개 목록이며, 신규 환경에 전부 설치하라는 추천 목록이 아니다.</p>
<h3 id="1-14-jcasc-jenkins-자체의-설정도-문서로-관리">1-14. JCasC: Jenkins 자체의 설정도 문서로 관리</h3>
<p>Jenkinsfile은 <strong>어떤 작업을 할지</strong> 적는다. JCasC는 Jenkins Configuration as Code로, <strong>Jenkins 서버를 어떻게 설정할지</strong> 적는 방식이다.</p>
<table>
<thead>
<tr>
<th>비교</th>
<th>Jenkinsfile</th>
<th>JCasC</th>
</tr>
</thead>
<tbody><tr>
<td>대상</td>
<td>프로젝트의 작업 흐름</td>
<td>Jenkins 시스템 설정</td>
</tr>
<tr>
<td>형식</td>
<td>주로 Groovy DSL</td>
<td>YAML</td>
</tr>
<tr>
<td>예시</td>
<td>테스트 후 이미지 만들기</td>
<td>노드·도구·인증·권한 설정</td>
</tr>
</tbody></table>
<p>사람이 화면에서 설정만 바꾸면 서버를 다시 만들 때 같은 구성을 재현하기 어렵다. 이런 개별 서버를 눈송이처럼 모두 다르다는 의미로 <strong>Snowflake Server</strong>라고 부르기도 한다.</p>
<p>다음은 JCasC의 일부 설정 예제다. 완전한 보안·Agent 구성은 아니다.</p>
<pre><code class="language-yaml">jenkins:
  systemMessage: &quot;Library CI learning environment&quot;
  numExecutors: 0
  mode: NORMAL</code></pre>
<p>여기서 <code>numExecutors: 0</code>은 Controller의 직접 빌드 실행 자리를 두지 않는 설정이다. 이 구성을 사용하려면 별도 Agent가 필요하다.</p>
<p>교재의 다중 사용자 실습 구성은 사용자별 Folder, 권한 규칙, 공용 Credentials 등을 미리 준비한다. 일반적인 설계 목적은 “각자 작업을 구분하고 필요한 권한만 부여하며 반복 설치를 쉽게 하는 것”이다. 실제 사용자 목록과 인증값은 공개하지 않는다.</p>
<p>중요한 구분도 있다. <strong>JCasC 자체가 플러그인 바이너리를 설치하는 도구는 아니다.</strong> 플러그인 설치는 별도 방식으로 준비하고 JCasC는 설치된 기능의 설정을 적용한다. 서버의 빌드 이력·파일·모든 데이터까지 YAML 하나로 복원되는 것도 아니다. 플러그인 버전, 비밀값 공급, 저장소 백업 등이 함께 필요하다. <a href="https://plugins.jenkins.io/configuration-as-code/">JCasC 공식 안내</a></p>
<h3 id="1-15-jenkins와-tekton-비교">1-15. Jenkins와 Tekton 비교</h3>
<table>
<thead>
<tr>
<th>비교</th>
<th>Jenkins</th>
<th>Tekton</th>
</tr>
</thead>
<tbody><tr>
<td>중심 구조</td>
<td>Controller·Agent</td>
<td>Kubernetes CRD·Controller·작업 Pod</td>
</tr>
<tr>
<td>흐름 표현</td>
<td>Jenkinsfile 등</td>
<td>Task·Pipeline 등의 Kubernetes 리소스</td>
</tr>
<tr>
<td>실행 환경</td>
<td>서버·컨테이너·Kubernetes 등</td>
<td>Kubernetes 기반</td>
</tr>
<tr>
<td>장점</td>
<td>다양한 기존 도구와 연동</td>
<td>Kubernetes 방식과 자연스럽게 연결</td>
</tr>
<tr>
<td>배울 것</td>
<td>Pipeline·플러그인·Agent 관리</td>
<td>Kubernetes·CRD·Task·Pipeline 구성</td>
</tr>
</tbody></table>
<p>Tekton 작업이 필요할 때 Pod로 실행된다고 해서 클러스터나 제어 구성요소가 전혀 없는 것은 아니다. 완전히 무상태라거나 항상 비용이 더 적다는 결론도 환경에 따라 달라진다. <a href="https://tekton.dev/docs/pipelines/">Tekton Task·Pipeline 설명</a></p>
<h3 id="1-16-로컬-jenkins-실행과-첫-파이프라인">1-16. 로컬 Jenkins 실행과 첫 파이프라인</h3>
<p>교재는 빌드 도구·플러그인·초기 설정이 들어 있는 교육용 Jenkins 이미지를 사용한다. 공개 글에서는 그 이미지와 공용 로그인 정보 대신 공식 이미지로 시작하는 개념 예제를 사용한다.</p>
<p>Docker가 준비되어 있고 같은 컨테이너 이름이 없는 환경에서 다음처럼 실행할 수 있다.</p>
<pre><code class="language-bash">docker run -d --name jenkins-learning \
  -p 127.0.0.1:8888:8080 \
  -v jenkins-learning-home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk21</code></pre>
<p>브라우저에서 <code>http://localhost:8888</code>로 접속한다. 최초 설정에는 초기 잠금 해제와 플러그인·사용자 준비가 필요하다. 공식 이미지가 교재의 교육용 이미지와 동일한 빌드 환경을 제공하는 것은 아니다. <a href="https://www.jenkins.io/doc/book/installing/docker/">Jenkins Docker 설치 안내</a></p>
<p>첫 Job의 등록 순서는 다음과 같다. 메뉴 표기는 버전·언어 설정에 따라 달라질 수 있다.</p>
<ol>
<li>새 Item을 만든다.</li>
<li>이름을 입력하고 Pipeline 유형을 선택한다.</li>
<li>Pipeline script 방식으로 아래 코드를 입력한다.</li>
<li>저장하고 Build Now로 실행한다.</li>
<li>Stage 화면과 Console Output을 확인한다.</li>
</ol>
<pre><code class="language-groovy">pipeline {
    agent any
    environment {
        APP_NAME = 'library-demo'
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage('Build') {
            steps {
                echo &quot;${env.APP_NAME}: 빌드 순서 연습&quot;
            }
        }
        stage('Test') {
            steps {
                echo '이 단계는 테스트 명령 대신 설명을 출력합니다.'
            }
        }
        stage('Deploy') {
            steps {
                echo '이 단계는 실제 배포를 수행하지 않습니다.'
            }
        }
    }
    post {
        always { echo '파이프라인 연습 종료' }
        success { echo '정의한 단계가 정상 종료됨' }
        failure { echo '실패한 단계의 Console Output 확인' }
    }
}</code></pre>
<p>이 파이프라인의 성공은 로그 출력 단계가 성공했다는 뜻이다. 실제 테스트·배포를 완료했다는 뜻이 아니다.</p>
<p>Docker Agent로 실행하는 실습에서는 적합한 도구가 포함된 이미지와 노드의 Docker 접근 설정을 확인한다. Docker-in-Docker, 호스트 소켓 연결, 원격 Docker 등은 서로 다른 방식이다. 소켓 마운트를 누구나 쓸 수 있는 기본 설정처럼 복사하지 않는다. <a href="https://www.jenkins.io/doc/book/pipeline/docker/">Docker Pipeline 안내</a></p>
<h3 id="1-17-이-파트-확인-질문">1-17. 이 파트 확인 질문</h3>
<p><strong>Q. Jenkinsfile과 JCasC의 차이는?</strong></p>
<p>A. 프로젝트 작업 흐름과 Jenkins 시스템 설정이라는 대상이 다르다.</p>
<p><strong>Q. agent any인데 작업이 계속 대기한다면?</strong></p>
<p>A. 사용 가능한 노드와 Executor, 노드 상태를 확인한다.</p>
<p><strong>Q. Test라는 Stage에서 echo만 실행했다면 테스트 성공인가?</strong></p>
<p>A. 로그 출력 성공이다. 실제 테스트 명령과 결과가 있어야 한다.</p>
<hr />
<h2 id="2-jenkins-cicd-2-실제-빌드·업로드·배포-연결">2. Jenkins CI/CD (2): 실제 빌드·업로드·배포 연결</h2>
<h3 id="2-1-인증정보를-준비하자">2-1. 인증정보를 준비하자</h3>
<p>파이프라인은 Git 코드를 읽고 이미지를 올리며 클러스터에 요청할 수 있다. 이 작업에는 서로 다른 권한이 필요하다.</p>
<table>
<thead>
<tr>
<th>대상</th>
<th>필요한 권한 예시</th>
</tr>
</thead>
<tbody><tr>
<td>앱 Git 저장소</td>
<td>코드 읽기</td>
</tr>
<tr>
<td>배포 Git 저장소</td>
<td>설정 변경 기록·push</td>
</tr>
<tr>
<td>이미지 레지스트리</td>
<td>정해진 저장소로 이미지 업로드</td>
</tr>
<tr>
<td>Kubernetes</td>
<td>허용된 네임스페이스의 리소스 관리</td>
</tr>
</tbody></table>
<p>Git 인증정보가 레지스트리 로그인 정보가 되는 것은 아니다. 레지스트리 push 권한이 Kubernetes 배포 권한이 되는 것도 아니다.</p>
<p>Jenkins에서는 Credentials에 인증정보를 등록하고 <strong>credentialsId라는 식별자</strong>로 참조할 수 있다. ID는 비밀번호 자체가 아니라 보관함을 찾는 이름이다. 해당 Job에서 접근 가능한 저장 범위와 권한도 확인한다. <a href="https://www.jenkins.io/doc/book/using/using-credentials/">Jenkins Credentials 설명</a></p>
<h3 id="2-2-withcredentials-필요한-동안만-인증값-전달">2-2. withCredentials: 필요한 동안만 인증값 전달</h3>
<p>다음은 <code>steps</code> 안의 Shell 로그인 예제다. <code>registry-demo</code>라는 Username with password Credentials와 Docker가 준비되어 있어야 한다.</p>
<pre><code class="language-groovy">withCredentials([usernamePassword(
    credentialsId: 'registry-demo',
    usernameVariable: 'REGISTRY_USER',
    passwordVariable: 'REGISTRY_PASS'
)]) {
    sh '''
        set +x
        printf '%s' &quot;$REGISTRY_PASS&quot; | docker login registry.example.com \
          --username &quot;$REGISTRY_USER&quot; --password-stdin
    '''
}</code></pre>
<p>실제 비밀번호를 코드에 적지 않고, Shell이 주입된 환경변수를 읽는다. <code>--password-stdin</code>은 비밀번호를 표준 입력으로 전달한다.</p>
<p>블록이 끝나면 바인딩 범위가 끝나지만 외부 프로그램이 만든 인증 파일이나 캐시까지 자동 삭제되는 것은 아니다. Docker 설정 파일의 저장 위치와 정리도 함께 관리한다. Jenkins의 로그 마스킹도 모든 정보 유출을 완벽하게 막는 장치는 아니다.</p>
<p>교재의 Credentials 검증 실습은 Git 접근과 레지스트리 인증이 되는지 확인하는 단계다. 검증할 때 <strong>비밀번호를 echo해서 확인하지 않는다.</strong> 실제 읽기·로그인 요청의 성공 여부로 판단한다.</p>
<h3 id="2-3-파이프라인-등록-방식-두-가지">2-3. 파이프라인 등록 방식 두 가지</h3>
<table>
<thead>
<tr>
<th>방식</th>
<th>특징</th>
<th>쓰임</th>
</tr>
</thead>
<tbody><tr>
<td>Pipeline script</td>
<td>Jenkins 화면에 직접 코드 입력</td>
<td>처음 문법을 연습하거나 작은 실험</td>
</tr>
<tr>
<td>Pipeline script from SCM</td>
<td>Git의 Jenkinsfile을 읽음</td>
<td>변경 이력·리뷰·팀 공유</td>
</tr>
</tbody></table>
<p>SCM은 Source Code Management, 소스 코드 관리다. 이 수업에서는 주로 Git을 뜻한다.</p>
<p>SCM 방식에서는 다음이 서로 맞아야 한다.</p>
<ul>
<li>저장소 주소.</li>
<li>접근 Credentials.</li>
<li>Jenkinsfile이 있는 브랜치.</li>
<li>저장소 안의 Script Path.</li>
</ul>
<p>Script Path는 내 노트북의 절대 경로가 아니라 <strong>원격 저장소 내부의 파일 경로</strong>다. 로컬에만 Jenkinsfile을 만들고 push하지 않았다면 Jenkins는 그 파일을 가져올 수 없다.</p>
<h3 id="2-4-cicd의-단계별-입력과-결과">2-4. CI/CD의 단계별 입력과 결과</h3>
<table>
<thead>
<tr>
<th>단계</th>
<th>입력</th>
<th>결과</th>
</tr>
</thead>
<tbody><tr>
<td>Checkout</td>
<td>Git 주소·브랜치·권한</td>
<td>작업 공간의 소스 코드</td>
</tr>
<tr>
<td>앱 빌드·테스트</td>
<td>코드·도구·의존성</td>
<td>JAR·테스트 결과</td>
</tr>
<tr>
<td>이미지 메타 계산</td>
<td>실행 번호·Git 커밋 등</td>
<td>사용할 이미지 태그</td>
</tr>
<tr>
<td>이미지 빌드·push</td>
<td>Dockerfile·앱 결과물·인증</td>
<td>레지스트리의 이미지</td>
</tr>
<tr>
<td>배포 설정 반영</td>
<td>이미지 주소·환경 설정</td>
<td>Kubernetes 배포 정의</td>
</tr>
<tr>
<td>배포·검증</td>
<td>정의·클러스터 권한</td>
<td>새 앱의 실행 상태·응답</td>
</tr>
</tbody></table>
<p>Java의 Maven 빌드와 컨테이너 이미지 빌드는 서로 다른 작업이다. Dockerfile이 <code>target/*.jar</code>를 사용하는데 JAR가 없으면 이미지 빌드도 실패한다.</p>
<h3 id="2-5-이미지-태그를-한-번-정하고-함께-사용하자">2-5. 이미지 태그를 한 번 정하고 함께 사용하자</h3>
<p>1일차 실습에서 <code>1.0</code>, <code>1.0.kaniko-pod</code>, <code>2.0.kaniko</code>는 서로 다른 태그였다. 자동화에서도 이 차이를 그대로 기억해야 한다.</p>
<pre><code class="language-text">빌드·push: library-api:build-12-a1b2c3d4
배포 설정: library-api:build-12-a1b2c3d4</code></pre>
<p>빌더가 사용한 주소와 배포가 요청한 주소가 같아야 한다. 태그 문자열을 단계마다 따로 만들면 어긋나기 쉽다.</p>
<p>커밋 식별자를 태그에 넣으면 어떤 코드로 만든 이미지인지 추적하기 쉽다. 단, 같은 소스여도 의존성·베이스 이미지·빌드 설정이 바뀌면 결과가 달라질 수 있다. 중요한 배포에서는 이미지 digest 등 내용 식별 정보도 관리한다.</p>
<h3 id="2-6-실제-빌드·push-예제">2-6. 실제 빌드·push 예제</h3>
<p>아래는 <strong>SCM에서 불러오는 Jenkinsfile 예제</strong>다. 다음 조건을 먼저 준비해야 한다.</p>
<ul>
<li><code>ci-docker</code> label의 Agent에 Git·JDK·Docker·기본 Shell 도구가 있다.</li>
<li>저장소에 실행 가능한 <code>mvnw</code>, <code>.mvn/</code>, <code>pom.xml</code>, <code>Dockerfile</code>이 있다.</li>
<li>Dockerfile이 생성된 JAR를 이미지에 포함한다.</li>
<li>실제 레지스트리 주소로 예제 값을 바꾼다.</li>
<li><code>registry-demo</code> Credentials에 해당 레지스트리 권한이 있다.</li>
</ul>
<p>이 예제는 이미지 업로드까지 수행한다. 뒤의 직접 배포 조각을 연결하면 CI/CD 방식이 되고, GitOps 설정 갱신을 연결하면 Jenkins CI + Argo CD 구성이 된다.</p>
<pre><code class="language-groovy">pipeline {
    agent { label 'ci-docker' }

    options {
        skipDefaultCheckout(true)
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_REPO = 'registry.example.com/demo/library-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh './mvnw -B clean verify'
            }
        }

        stage('Compute Image') {
            steps {
                script {
                    def commit = sh(
                        script: 'git rev-parse --short=12 HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = &quot;build-${env.BUILD_NUMBER}-${commit}&quot;
                    env.IMAGE_REF = &quot;${env.IMAGE_REPO}:${env.IMAGE_TAG}&quot;
                }
                echo &quot;사용할 이미지: ${env.IMAGE_REF}&quot;
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build --tag &quot;$IMAGE_REF&quot; .'
            }
        }

        stage('Push Image') {
            steps {
                withEnv([&quot;DOCKER_CONFIG=${env.WORKSPACE}/.docker-auth&quot;]) {
                    withCredentials([usernamePassword(
                        credentialsId: 'registry-demo',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASS'
                    )]) {
                        sh '''
                            set -eu
                            set +x
                            mkdir -p &quot;$DOCKER_CONFIG&quot;
                            trap 'rm -f &quot;$DOCKER_CONFIG/config.json&quot;' EXIT
                            printf '%s' &quot;$REGISTRY_PASS&quot; | docker login &quot;$REGISTRY&quot; \
                              --username &quot;$REGISTRY_USER&quot; --password-stdin
                            docker push &quot;$IMAGE_REF&quot;
                        '''
                    }
                }
            }
        }
    }

    post {
        success { echo '앱 검증과 이미지 업로드가 완료되었습니다.' }
        failure { echo '실패 단계의 로그를 확인하세요.' }
    }
}</code></pre>
<p>이 예제의 메시지는 성공한 범위를 명확하게 표현해야 한다. 이미지 push 성공을 사용자 서비스의 정상 배포로 표현하지 않는다.</p>
<p>실행 노드가 서로 다른 CPU를 사용한다면 이미지의 지원 플랫폼도 확인한다. 위의 단일 플랫폼 <code>docker build</code> 예제는 모든 플랫폼을 만드는 명령이 아니다.</p>
<h3 id="2-7-직접-배포하는-cicd-방식">2-7. 직접 배포하는 CI/CD 방식</h3>
<p>교재의 첫 CI/CD 구성은 Jenkins가 이미지 생성부터 Kubernetes 적용까지 수행하는 방식이다.</p>
<pre><code class="language-text">Jenkins → 이미지 빌드·push → 배포 YAML 생성 → kubectl apply</code></pre>
<p>다음은 앞 단계의 <code>IMAGE_REF</code>를 사용해 새 매니페스트를 생성하는 <strong>배포 Stage 내부 조각</strong>이다. <code>kubectl</code>, 사용할 Credentials, 네임스페이스, 실제 앱 포트가 준비되어 있어야 한다. 예시 앱은 HTTP 8080 포트를 사용한다.</p>
<pre><code class="language-groovy">withCredentials([file(
    credentialsId: 'kubeconfig-demo',
    variable: 'KUBECONFIG'
)]) {
    sh '''
        set -eu
        kubectl create deployment library-api \
          --image=&quot;$IMAGE_REF&quot; --port=8080 \
          --namespace=library-dev \
          --dry-run=client -o yaml &gt; deployment.generated.yaml

        kubectl apply -f deployment.generated.yaml
        kubectl rollout status deployment/library-api \
          --namespace=library-dev --timeout=180s
    '''
}</code></pre>
<p>이 조각은 Deployment 중심의 학습 예제다. Service·Ingress·ConfigMap·이미지 pull Secret·상태 점검 등 전체 앱 구성을 자동으로 완성하지 않는다. 실제 수업에서는 준비된 매니페스트나 Kustomize 구성을 사용해 관련 설정을 함께 적용한다.</p>
<p>직접 배포 방식에서는 Jenkins 실행 환경에 클러스터 접근 권한이 필요하다. 다음 파트의 GitOps 방식은 이 책임을 CD 도구로 분리한다.</p>
<h3 id="2-8-docker-대신-buildah·buildkit-사용하기">2-8. Docker 대신 Buildah·BuildKit 사용하기</h3>
<p>교재에는 Docker·Buildah·BuildKit 기반 Jenkinsfile이 나온다. 이름이 달라도 목표는 공통이다.</p>
<pre><code class="language-text">앱 빌드 결과 + Dockerfile → 이미지 생성 → 같은 이미지 주소로 push</code></pre>
<p>바뀌는 것은 빌드 명령과 필요한 실행 환경이다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>먼저 확인할 것</th>
</tr>
</thead>
<tbody><tr>
<td>Docker</td>
<td>Docker CLI·데몬 접근·플랫폼</td>
</tr>
<tr>
<td>Buildah</td>
<td>설치·사용자 권한·스토리지 드라이버</td>
</tr>
<tr>
<td>BuildKit</td>
<td>빌더 접근·캐시·출력·push 설정</td>
</tr>
</tbody></table>
<p>Docker가 없는 노드에 Docker용 Jenkinsfile을 넣거나, Buildah 실행 권한이 없는 Pod에 Buildah용 스크립트를 넣으면 실패한다. 교재 예제 이름만 바꾸는 것으로 실행 조건까지 바뀌지 않는다.</p>
<h3 id="2-9-실행-결과-확인과-실패-위치-찾기">2-9. 실행 결과 확인과 실패 위치 찾기</h3>
<table>
<thead>
<tr>
<th>실패 위치</th>
<th>먼저 확인할 것</th>
</tr>
</thead>
<tbody><tr>
<td>Checkout</td>
<td>주소·브랜치·인증·DNS</td>
</tr>
<tr>
<td>Maven</td>
<td>JDK·pom.xml·의존성·테스트</td>
</tr>
<tr>
<td>이미지 빌드</td>
<td>Dockerfile·컨텍스트·JAR·권한</td>
</tr>
<tr>
<td>push</td>
<td>레지스트리 주소·태그·로그인·쓰기 권한</td>
</tr>
<tr>
<td>Kubernetes 적용</td>
<td>context·namespace·RBAC·리소스 스키마</td>
</tr>
<tr>
<td>Pod 이미지 다운로드</td>
<td>실제 이미지·태그·pull 권한·플랫폼</td>
</tr>
<tr>
<td>앱 시작</td>
<td>앱 로그·설정·DB·포트</td>
</tr>
</tbody></table>
<p>테스트 결과 XML을 <code>junit</code>으로 게시하거나 빌드 결과를 <code>archiveArtifacts</code>로 보관할 수 있다. 실패한 테스트를 결과 수집 단계로 감추거나, 테스트를 생략한 빌드를 “검증 완료”라고 표현하지 않는다.</p>
<p><code>set -e</code> 또는 <code>set -eu</code>를 사용한 Shell은 오류 뒤의 무조건적인 성공 메시지를 줄이는 데 도움이 된다. 복잡한 조건문·파이프에서는 종료 처리 방식도 이해해야 한다. 마지막 <code>echo</code>가 성공했다고 앞의 Git clone까지 성공한 것은 아니다.</p>
<h3 id="2-10-이-파트-확인-질문">2-10. 이 파트 확인 질문</h3>
<p><strong>Q. Registry Credentials와 Git Credentials를 바꿔 써도 될까?</strong></p>
<p>A. 목적과 인증 대상이 다르므로 맞는 Credentials를 사용해야 한다.</p>
<p><strong>Q. Jenkinsfile을 만들었는데 SCM Job이 읽지 못한다면?</strong></p>
<p>A. 원격 push 여부, 브랜치, Script Path를 확인한다.</p>
<p><strong>Q. 이미지 업로드가 성공하면 웹서비스 배포도 성공인가?</strong></p>
<p>A. 배포 설정·클러스터 적용·앱 상태는 다음 단계다.</p>
<hr />
<h2 id="3-jenkins-ci--jenkins-poll-scm">3. Jenkins CI &amp; Jenkins Poll SCM</h2>
<h3 id="3-1-jenkins-ci와-직접-cicd의-차이">3-1. Jenkins CI와 직접 CI/CD의 차이</h3>
<p>앞 파트에서는 Jenkins가 직접 Kubernetes에 배포할 수 있었다. 이제 배포 책임을 Argo CD로 옮겨 보자.</p>
<pre><code class="language-text">Jenkins CI
  코드 가져오기 → 검사 → 이미지 빌드·push → Git 배포 설정 갱신

Argo CD
  Git 배포 설정 읽기 → 실제 Kubernetes 상태와 비교 → 동기화</code></pre>
<p>Jenkins가 배포 설정을 변경했다고 해서 실제 배포가 끝난 것은 아니다. Argo CD가 해당 변경을 읽고 적용해야 한다.</p>
<h3 id="3-2-소스-저장소와-배포-저장소">3-2. 소스 저장소와 배포 저장소</h3>
<table>
<thead>
<tr>
<th>저장소</th>
<th>담는 것</th>
<th>누가 주로 바꾸는가?</th>
</tr>
</thead>
<tbody><tr>
<td>소스 저장소</td>
<td>앱 코드·테스트·Dockerfile·Jenkinsfile</td>
<td>개발자와 CI</td>
</tr>
<tr>
<td>배포 저장소</td>
<td>Kubernetes YAML·Kustomize·Helm 설정</td>
<td>배포 변경 담당과 CI</td>
</tr>
</tbody></table>
<p>교재에서는 한 저장소의 별도 <code>gitops</code> 브랜치를 배포 설정에 사용하는 구성도 등장한다. 반드시 별도 저장소를 만들어야 하는 것은 아니다. 하지만 <strong>어떤 저장소·브랜치·경로가 배포 기준인지</strong> 명확해야 한다.</p>
<pre><code class="language-text">소스 main 브랜치 → 앱 CI 실행
배포 gitops 브랜치 → Argo CD 동기화 기준</code></pre>
<p>브랜치와 경로를 혼동하면 Jenkins가 갱신한 파일을 Argo CD가 보지 못한다.</p>
<h3 id="3-3-이미지-정보를-git에-기록하기">3-3. 이미지 정보를 Git에 기록하기</h3>
<p>배포 저장소의 Kustomize 구성에 새 이미지 태그를 기록하는 예제다. 앞서 CI가 계산한 <code>IMAGE_REF</code>가 Shell 환경변수로 전달되고, 별도의 배포 저장소 작업 폴더에 들어와 있으며 Kustomize CLI가 설치되어 있다고 가정한다.</p>
<pre><code class="language-bash">cd kustomize/overlays/dev
kustomize edit set image &quot;registry.example.com/demo/library-api=$IMAGE_REF&quot;
cd ../..
kubectl kustomize overlays/dev</code></pre>
<p>원본에서 이미지 이름이 일치해야 변환이 적용된다. 수정한 경로를 Git 저장소 루트 기준으로 정확히 stage하고 commit·push한다.</p>
<pre><code class="language-bash"># 배포 저장소 루트에서 실행
git add kustomize/overlays/dev/kustomization.yaml
git diff --cached --quiet || git commit -m &quot;Update development image&quot;
git push origin HEAD:gitops</code></pre>
<p>Git 사용자 이름·이메일, 해당 브랜치의 쓰기 권한, 인증 방식은 사전에 준비한다. Shell의 IMAGE_REF를 임의로 비워 둔 채 실행하지 않는다. 환경·버전이 같아 변경이 없다면 빈 커밋 실패를 적절히 처리한다.</p>
<h3 id="3-4-poll-scm-주기적으로-변경을-물어보기">3-4. Poll SCM: 주기적으로 변경을 물어보기</h3>
<p>SCM Polling은 Jenkins가 Git 저장소에 “새 변경이 있나요?”라고 주기적으로 물어보는 방식이다.</p>
<p>도서관 직원이 일정 간격으로 반납함을 확인하는 것과 비슷하다. <strong>확인할 때마다 무조건 새 작업을 실행하는 것은 아니다.</strong> 일반적인 Poll SCM은 변경을 확인했을 때 빌드를 예약한다.</p>
<pre><code class="language-groovy">triggers {
    pollSCM('H/5 * * * *')
}</code></pre>
<p>위 코드는 <code>pipeline</code> 안의 조각이다. SCM 설정이 있는 Job에서 적용해야 한다. Declarative Jenkinsfile에 정의한 Trigger가 Job 설정에 반영되려면 초기 실행·설정 적용이 필요할 수 있다.</p>
<h3 id="3-5-cron의-다섯-칸">3-5. Cron의 다섯 칸</h3>
<pre><code class="language-text">분  시  일  월  요일
 *   *   *   *    *</code></pre>
<ul>
<li><code>*</code>: 해당 범위 전체.</li>
<li><code>*/n</code>: 일정 간격.</li>
<li><code>a-b</code>: 범위.</li>
<li><code>a,b</code>: 여러 값.</li>
<li><code>H</code>: Jenkins가 Job 이름 등에 기반한 해시로 시점을 분산.</li>
</ul>
<p><code>H</code>는 매번 새 난수를 뽑는 기호가 아니다. 여러 Job이 동시에 저장소를 조회하는 부하를 분산하는 데 사용한다.</p>
<p>교재에서 다루는 표현을 의미 중심으로 정리하면 다음과 같다. 아래 숫자는 자동화 조건이며 수업 시간표가 아니다.</p>
<table>
<thead>
<tr>
<th>표현</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>* * * * *</code></td>
<td>매 분 변경 확인</td>
</tr>
<tr>
<td><code>H/15 * * * *</code></td>
<td>Job별로 분산된 시점에서 약 15분 간격</td>
</tr>
<tr>
<td><code>H(0-29)/10 * * * *</code></td>
<td>각 시간의 앞쪽 분 범위에서 10분 간격</td>
</tr>
<tr>
<td><code>45 9-16/2 * * 1-5</code></td>
<td>평일의 지정된 시 범위에서 2시간 간격, 분은 45</td>
</tr>
<tr>
<td><code>H H(8-15)/2 * * 1-5</code></td>
<td>평일·지정 시 범위에서 시점을 분산해 2시간 간격</td>
</tr>
<tr>
<td><code>H H 1,15 1-11 *</code></td>
<td>지정 월 범위에서 매월 1일·15일, 분·시는 분산</td>
</tr>
</tbody></table>
<p><code>pollSCM</code>과 <code>cron</code> Trigger는 같은 역할이 아니다. <code>cron</code>은 일정에 따라 빌드를 예약하고, <code>pollSCM</code>은 일정에 따라 코드 변경 여부를 확인한다. <a href="https://www.jenkins.io/doc/book/pipeline/syntax/#triggers">Jenkins Trigger·Cron 문법</a></p>
<h3 id="3-6-poll-scm과-webhook-비교">3-6. Poll SCM과 Webhook 비교</h3>
<table>
<thead>
<tr>
<th>비교</th>
<th>Poll SCM</th>
<th>Webhook</th>
</tr>
</thead>
<tbody><tr>
<td>방식</td>
<td>Jenkins가 저장소에 주기적으로 질문</td>
<td>저장소 서비스가 이벤트 발생 시 알림</td>
</tr>
<tr>
<td>비유</td>
<td>반납함을 주기적으로 열어 봄</td>
<td>반납되면 알림벨이 울림</td>
</tr>
<tr>
<td>반영 지연</td>
<td>확인 간격의 영향</td>
<td>전달·처리가 성공하면 빠르게 반응 가능</td>
</tr>
<tr>
<td>연결 조건</td>
<td>Jenkins가 Git에 접근</td>
<td>Git 서비스가 Jenkins의 수신 주소에 접근</td>
</tr>
</tbody></table>
<p>로컬 Jenkins는 외부에서 접근할 수 없을 수 있다. 이때 외부 GitHub가 노트북의 <code>localhost</code>로 Webhook을 보낼 수는 없다. GitHub에서 보는 localhost는 사용자의 컴퓨터 주소가 아니다.</p>
<p>교재의 Poll SCM 페이지에 Webhook이라는 설명이 섞여 있어도 <strong>주기적 조회와 이벤트 알림은 다른 방식</strong>으로 구분하자.</p>
<h3 id="3-7-변경-감지-실습-확인-순서">3-7. 변경 감지 실습 확인 순서</h3>
<ol>
<li>먼저 수동 빌드가 정상 동작하는지 확인한다.</li>
<li>Job의 SCM 저장소·브랜치·Credentials를 확인한다.</li>
<li>Poll SCM을 설정한다.</li>
<li>앱 저장소의 비밀정보 없는 파일을 수정한다.</li>
<li>commit·push한다.</li>
<li>Polling Log와 새 Build의 실행 원인을 확인한다.</li>
<li>이미지와 배포 Git 변경을 확인한다.</li>
</ol>
<p>실습용 README 수정 예시는 다음과 같다.</p>
<pre><code class="language-bash">printf '\nCI trigger practice\n' &gt;&gt; README.md
git add README.md
git commit -m &quot;Practice CI trigger&quot;
git push origin main</code></pre>
<h3 id="3-8-자동-실행이-스스로-반복되는-문제">3-8. 자동 실행이 스스로 반복되는 문제</h3>
<p>CI가 GitOps 브랜치를 변경하고 그 변경이 같은 CI를 다시 실행시키면 불필요한 반복이 생길 수 있다.</p>
<pre><code class="language-text">CI 실행 → 배포 설정 push → 다시 CI 실행 → 또 push → ...</code></pre>
<p>앱 코드와 배포 설정의 Trigger 범위를 분리한다. 저장소·브랜치·경로 필터를 사용할 수 있다. 여러 Job이 같은 배포 브랜치를 동시에 수정하면 push 충돌도 생길 수 있으므로 변경·검토·동시 실행 정책을 정한다.</p>
<h3 id="3-9-이-파트-확인-질문">3-9. 이 파트 확인 질문</h3>
<p><strong>Q. Poll SCM이 실행되면 매번 빌드할까?</strong></p>
<p>A. 일반적으로 변경을 발견해야 빌드를 예약한다.</p>
<p><strong>Q. Jenkins가 배포 Git을 push하면 배포 완료인가?</strong></p>
<p>A. Git 설정 갱신이 완료된 것이다. Argo CD 동기화와 앱 상태는 추가로 확인한다.</p>
<p><strong>Q. H는 매번 달라지는 무작위 시각일까?</strong></p>
<p>A. Job에 기반해 부하를 분산하는 해시 표현이다.</p>
<hr />
<h2 id="4-kubernetes에-jenkins-배포-및-실행하기">4. Kubernetes에 Jenkins 배포 및 실행하기</h2>
<h3 id="4-1-jenkins도-하나의-애플리케이션이다">4-1. Jenkins도 하나의 애플리케이션이다</h3>
<p>Jenkins를 노트북의 컨테이너로 실행할 수도 있고 Kubernetes에 배포할 수도 있다. 위치가 바뀌어도 Pipeline·Credentials·Job이라는 개념은 같다.</p>
<pre><code class="language-text">브라우저
   ↓
Ingress 또는 로컬 포트 연결
   ↓
Jenkins Service
   ↓
Jenkins Controller Pod
   ↓
Agent에서 빌드 작업 실행</code></pre>
<h3 id="4-2-필요한-리소스의-역할">4-2. 필요한 리소스의 역할</h3>
<table>
<thead>
<tr>
<th>리소스</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Deployment 또는 StatefulSet</td>
<td>Jenkins Controller Pod 관리</td>
</tr>
<tr>
<td>Service</td>
<td>Jenkins 접근 경로</td>
</tr>
<tr>
<td>PVC</td>
<td>Jenkins Home 등 영구 데이터 저장</td>
</tr>
<tr>
<td>ConfigMap</td>
<td>일반 설정·JCasC 구성 등</td>
</tr>
<tr>
<td>Secret</td>
<td>비밀 설정과 인증 데이터</td>
</tr>
<tr>
<td>ServiceAccount</td>
<td>Pod에서 사용할 Kubernetes 사용자 정체성</td>
</tr>
<tr>
<td>Role·RoleBinding</td>
<td>허용할 Kubernetes 작업 범위</td>
</tr>
<tr>
<td>Ingress</td>
<td>외부 HTTP/HTTPS 접속 규칙</td>
</tr>
</tbody></table>
<p>교재는 미리 준비한 교육용 이미지와 매니페스트를 사용한다. 공개 환경에서 그대로 재현할 수 있는 이미지라고 가정하지 않는다. 이미지·플러그인·빌드 도구·저장 공간을 자신의 환경에 맞게 준비한다.</p>
<h3 id="4-3-jenkins-home은-지속성이-중요하다">4-3. Jenkins Home은 지속성이 중요하다</h3>
<p>Pod는 다시 만들어질 수 있다. 컨테이너 내부에만 저장한 설정이나 실행 기록이 Pod 교체와 함께 사라질 수 있으므로 Jenkins의 저장 공간을 설계해야 한다.</p>
<p>PVC는 데이터를 저장할 장소를 요청하는 리소스다. PVC를 작성했다고 실제 저장소가 무조건 연결되는 것은 아니다. StorageClass·PV·접근 모드·권한 등이 환경과 맞아야 한다.</p>
<p>JCasC는 설정 재현을 돕고 PVC·백업은 데이터 지속성을 돕는다. 두 개는 대체 관계가 아니다.</p>
<h3 id="4-4-교재의-배포-실습-흐름">4-4. 교재의 배포 실습 흐름</h3>
<pre><code class="language-text">이미지·설정·저장 공간·권한 준비
    ↓
생성된 Kubernetes YAML 확인
    ↓
리소스 적용
    ↓
Pod·PVC·Service 상태 확인
    ↓
Jenkins 접속·Credentials 확인
    ↓
수동 Pipeline 실행</code></pre>
<p><code>jenkins-k8s/</code>에 검토한 매니페스트가 있다는 가정의 명령 예시다.</p>
<pre><code class="language-bash">kubectl apply -f jenkins-k8s/ -n ci-demo
kubectl get pods,services,pvc -n ci-demo
kubectl logs deployment/jenkins -n ci-demo</code></pre>
<p>실제 이름이 <code>jenkins</code>가 아니라면 작성한 리소스 이름으로 바꾼다. 디렉터리에는 적용할 YAML을 모아 두고 임시 파일·다른 환경의 리소스가 섞이지 않게 한다.</p>
<p>Ingress가 준비되지 않은 학습 환경에서는 Service 포트가 8080인 경우 다음처럼 확인할 수 있다.</p>
<pre><code class="language-bash">kubectl port-forward service/jenkins 8888:8080 -n ci-demo</code></pre>
<p>브라우저에서 <code>http://localhost:8888</code>로 접속한다. 실제 Service의 포트는 매니페스트와 맞춰야 한다. 접속이 됐다는 사실과 빌드 Agent가 준비됐다는 사실은 따로 확인한다.</p>
<h3 id="4-5-kubeconfig-우선순위와-context">4-5. kubeconfig 우선순위와 context</h3>
<p>로컬 CLI에서 Kubernetes 설정 파일을 찾는 일반적인 순서는 다음과 같다.</p>
<ol>
<li>명시한 <code>--kubeconfig</code> 파일.</li>
<li><code>KUBECONFIG</code> 환경변수의 파일 목록.</li>
<li>기본 <code>~/.kube/config</code>.</li>
</ol>
<p>그다음 선택한 설정에서 사용할 context를 정한다. <code>--context</code>는 별도의 kubeconfig 파일이 아니라 <strong>설정 안의 연결 조합을 선택</strong>한다.</p>
<pre><code class="language-bash">kubectl config current-context
kubectl --kubeconfig ./config-demo --context demo get pods -n library-dev</code></pre>
<p>여러 kubeconfig 파일을 병합하는 규칙과 충돌 처리도 존재하므로 파일 이름만으로 판단하지 않는다. <a href="https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/">Kubernetes kubeconfig 설명</a></p>
<p>Pod 내부 애플리케이션에서는 ServiceAccount 인증을 사용하는 in-cluster 접근도 가능하다. 하지만 노트북의 kubectl과 모든 라이브러리가 완전히 동일한 우선순위로 동작한다고 일반화하면 안 된다.</p>
<p><strong>내 노트북에 있는 kubeconfig가 Jenkins Pod 안에도 자동으로 존재하는 것은 아니다.</strong> 실행 위치에서 실제로 어떤 인증을 사용하는지 확인한다.</p>
<h3 id="4-6-rbac-인증과-권한은-다르다">4-6. RBAC: 인증과 권한은 다르다</h3>
<p>출입증이 있어도 모든 교실의 물건을 바꿀 수 있는 것은 아니다.</p>
<ul>
<li><strong>인증</strong>: 누구인지 확인.</li>
<li><strong>인가·권한</strong>: 그 사람이 무엇을 할 수 있는지 확인.</li>
</ul>
<p>Kubernetes에서는 ServiceAccount와 Role·RoleBinding 등을 연결해 권한을 정할 수 있다.</p>
<pre><code class="language-bash">kubectl auth can-i create deployments -n library-dev
kubectl auth can-i patch deployments -n library-dev</code></pre>
<p>위 명령은 현재 CLI가 사용하는 사용자 권한을 확인한다. Jenkins가 다른 인증을 쓴다면 Jenkins의 인증 기준으로 확인해야 한다.</p>
<p><code>Forbidden</code>을 만났다고 무조건 관리자 권한을 부여하지 않는다. 파이프라인에서 실제로 필요한 리소스·동작·네임스페이스를 먼저 정한다.</p>
<h3 id="4-7-kubernetes에서-빌드-도구-실행">4-7. Kubernetes에서 빌드 도구 실행</h3>
<p>Kubernetes에 Jenkins를 올렸다고 Docker 데몬이 자동 제공되는 것은 아니다. 빌드를 실행할 Controller·Agent에 적합한 빌드 구성이 필요하다.</p>
<p>교재에서는 Buildah·BuildKit 기반 파이프라인을 등록해 실행하는 과정을 다룬다. 1일차에서 배운 실행 권한·작업 공간·이미지 인증 조건을 같은 방식으로 확인한다.</p>
<p>Kubernetes Plugin은 빌드가 필요할 때 Agent Pod를 생성하는 구성을 지원한다. 필요한 JDK·Maven·빌더를 준비한 Pod 템플릿을 사용하고 작업이 끝난 뒤 정리할 수 있다. 동적 Agent를 사용한다고 Controller의 저장소·가용성 관리가 불필요해지는 것은 아니다.</p>
<h3 id="4-8-webhook으로-git-변경-알림-받기">4-8. Webhook으로 Git 변경 알림 받기</h3>
<p>교재의 Kubernetes Jenkins 실습에서는 외부에서 접근 가능한 Jenkins에 GitHub Webhook을 연결한다.</p>
<ol>
<li>Jenkins의 수신 주소와 GitHub 연동 플러그인을 준비한다.</li>
<li>대상 Job에 적절한 GitHub Trigger를 설정한다.</li>
<li>Git 저장소의 Webhook 설정에서 Jenkins 수신 주소를 등록한다.</li>
<li>Push 이벤트·Content type·필요한 서명 Secret 등을 설정한다.</li>
<li>테스트 요청의 전달 결과와 Jenkins 실행 원인을 확인한다.</li>
</ol>
<p>GitHub 플러그인에서 사용하는 주소 형식은 일반적으로 다음과 같다.</p>
<pre><code class="language-text">https://ci.example.com/github-webhook/</code></pre>
<p>끝의 <code>/</code>까지 포함하는 형태를 사용한다. <strong>실제 공개 접속 주소·네트워크·인증 정책·플러그인 설정이 모두 맞아야 한다.</strong> 주소만 붙여 넣는 것으로 자동 빌드가 보장되지는 않는다. <a href="https://plugins.jenkins.io/github/">GitHub 플러그인 안내</a></p>
<p>Webhook이 전달됐는지, Jenkins가 올바른 저장소의 Job을 찾았는지, 실제 빌드가 시작됐는지는 각 단계에서 확인한다.</p>
<h3 id="4-9-이-파트-확인-질문">4-9. 이 파트 확인 질문</h3>
<p><strong>Q. Jenkins Pod가 Running이면 빌드 환경도 준비된 것일까?</strong></p>
<p>A. 접속 가능한 Controller 상태와 Agent·빌드 도구·권한을 각각 확인해야 한다.</p>
<p><strong>Q. 인증은 되는데 Forbidden이 발생하면?</strong></p>
<p>A. 해당 사용자에게 요청한 작업의 권한이 있는지 확인한다.</p>
<p><strong>Q. JCasC가 있으니 PVC·백업이 필요 없을까?</strong></p>
<p>A. 설정 재현과 데이터 지속성은 다른 문제다.</p>
<hr />
<h2 id="5-gitops-기반-cd-구성하기">5. GitOps 기반 CD 구성하기</h2>
<h3 id="5-1-수동-배포는-어떤-문제가-생길까">5-1. 수동 배포는 어떤 문제가 생길까?</h3>
<p>운영자가 서버에 접속해 설정을 바꾸거나 콘솔에서 직접 수정한다고 생각해 보자.</p>
<ul>
<li>누가 어떤 값을 바꿨는지 찾기 어렵다.</li>
<li>Git의 설정과 실제 환경이 달라질 수 있다.</li>
<li>같은 환경을 다시 만들기 어렵다.</li>
<li>여러 서버를 동시에 맞추기 어렵다.</li>
</ul>
<p>학교 도서관의 공식 도서 목록은 그대로인데, 직원마다 자기 메모로 책 위치를 바꾸는 상황과 비슷하다. 목록과 현실이 달라지면 책을 찾기 어려워진다.</p>
<h3 id="5-2-gitops-원하는-운영-상태를-git에-기록하기">5-2. GitOps: 원하는 운영 상태를 Git에 기록하기</h3>
<p>GitOps는 <strong>원하는 시스템 상태를 선언적으로 기록하고, 버전 관리한 그 상태를 자동화된 조정 과정으로 실제 환경에 반영하는 방식</strong>이다. 수업에서는 Git과 Kubernetes, Argo CD를 중심으로 배운다.</p>
<pre><code class="language-text">Git: 원하는 상태
         ↓ 읽기·비교
CD 도구
         ↓ 적용·조정
Kubernetes: 실제 상태</code></pre>
<p>Git을 썼다는 이유만으로 모든 배포가 GitOps가 되는 것은 아니다. Git에 담긴 배포 상태를 기준으로 실제 환경과 계속 비교·조정하는 흐름을 연결해야 한다.</p>
<h3 id="5-3-desired-state와-actual-state">5-3. Desired State와 Actual State</h3>
<ul>
<li><strong>Desired State</strong>: 원하는 상태. 예를 들어 Pod 2개, 이미지 태그 build-12.</li>
<li><strong>Actual State</strong>: 클러스터에 현재 있는 상태. 예를 들어 Pod 1개, 이미지 태그 build-11.</li>
<li><strong>Drift</strong>: 두 상태가 어긋난 차이.</li>
<li><strong>Reconciliation</strong>: 차이를 확인하고 원하는 상태에 맞추는 조정 과정.</li>
</ul>
<p>온도 조절기에 설정한 온도와 실제 방 온도를 생각하면 쉽다. 센서로 현실을 읽고 목표와 차이가 있으면 조정한다.</p>
<p>단, Kubernetes와 GitOps가 목표를 알고 있다고 해서 자원 부족·이미지 누락·권한 문제까지 없애 주지는 않는다. <strong>목표에 맞추려고 동작하는 것과 목표를 달성한 것은 다르다.</strong></p>
<h3 id="5-4-gitops의-기대효과와-한계">5-4. GitOps의 기대효과와 한계</h3>
<p>교재의 장점은 다음과 같이 이해하자.</p>
<table>
<thead>
<tr>
<th>기대효과</th>
<th>실제로 필요한 조건</th>
</tr>
</thead>
<tbody><tr>
<td>변경 추적</td>
<td>의미 있는 commit·리뷰·권한 관리</td>
</tr>
<tr>
<td>환경 일관성</td>
<td>올바른 선언·동기화·예외 정책</td>
</tr>
<tr>
<td>재현성</td>
<td>이미지·의존성·비밀값·외부 자원도 준비</td>
</tr>
<tr>
<td>복구 지원</td>
<td>이전 상태와 현재 데이터의 호환성</td>
</tr>
<tr>
<td>익숙한 변경 경로</td>
<td>Git 작업 규칙과 승인 과정</td>
</tr>
<tr>
<td>감사·보안 지원</td>
<td>Git 이력 외에 접근 기록·실행 기록도 관리</td>
</tr>
</tbody></table>
<p>Git의 병합 충돌이나 여러 파이프라인의 동시 push가 사라지는 것은 아니다. Git을 사용하면 모든 보안이 자동 해결되거나 변경 이력이 곧 완전한 감사 로그가 되는 것도 아니다.</p>
<h3 id="5-5-argo-cd의-역할">5-5. Argo CD의 역할</h3>
<p>Argo CD는 Kubernetes에서 GitOps 방식의 배포를 관리하는 도구다. Git 등의 선언된 소스에서 원하는 리소스 구성을 만들고 클러스터와 비교·동기화한다.</p>
<pre><code class="language-text">개발자 → 앱 코드 Git
                ↓
           Jenkins CI
           ├─ 검사·이미지 빌드
           ├─ 레지스트리에 push
           └─ 배포 설정 Git 갱신
                          ↓
                      Argo CD
                          ↓
                      Kubernetes</code></pre>
<p>Argo CD가 Java 소스를 JAR로 컴파일하는 역할을 대신하는 것은 아니다. CI의 빌드와 CD의 동기화 책임을 나누어 이해한다.</p>
<p>Git에서 배포 정의를 pull하는 흐름과 Kubernetes가 레지스트리에서 이미지를 pull하는 흐름도 다르다. 각각 다른 주소·권한이 필요하다.</p>
<h3 id="5-6-crd-kubernetes에-새로운-신청서-종류-추가">5-6. CRD: Kubernetes에 새로운 신청서 종류 추가</h3>
<p>Kubernetes에는 Deployment, Service 같은 기본 리소스가 있다. CRD는 Custom Resource Definition으로, 새로운 리소스 종류를 정의하는 기능이다.</p>
<p>Argo CD 설치 시 Application, AppProject, ApplicationSet 같은 종류를 사용할 수 있게 된다.</p>
<table>
<thead>
<tr>
<th>종류</th>
<th>역할</th>
<th>학교 비유</th>
</tr>
</thead>
<tbody><tr>
<td>Application</td>
<td>소스와 배포 대상을 연결하는 배포 단위</td>
<td>도서관 서비스 배포 신청서</td>
</tr>
<tr>
<td>AppProject</td>
<td>허용 소스·대상·리소스 정책</td>
<td>해당 팀의 활동 범위 규칙</td>
</tr>
<tr>
<td>ApplicationSet</td>
<td>입력 목록과 템플릿으로 여러 Application 생성</td>
<td>학년별 신청서를 묶어 만드는 양식</td>
</tr>
</tbody></table>
<p>CRD는 문서 형식을 추가하는 것이고 Controller는 그 문서를 읽어 동작하는 담당자다. Application YAML만 만든다고 Argo CD가 설치되는 것은 아니다.</p>
<h3 id="5-7-appproject-무엇을-어디에-배포할-수-있을까">5-7. AppProject: 무엇을 어디에 배포할 수 있을까?</h3>
<p>AppProject는 Application을 논리적으로 묶고 다음과 같은 범위를 정할 수 있다.</p>
<ul>
<li>가져올 수 있는 Git·Helm 소스.</li>
<li>배포할 수 있는 클러스터와 네임스페이스.</li>
<li>허용·금지할 리소스 종류.</li>
<li>프로젝트 역할과 접근 정책.</li>
</ul>
<p>아래는 구조를 보여 주는 예제다. Argo CD와 CRD가 설치되어 있고 <code>library-dev</code>, <code>library-prod</code>가 준비되어 있어야 한다. 소스 URL은 실제 배포 저장소로 바꾼다.</p>
<pre><code class="language-yaml"># project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: library-project
  namespace: argocd
spec:
  description: Library service deployment policy
  sourceRepos:
    - https://github.com/YOUR_TEAM/library-gitops.git
  destinations:
    - server: https://kubernetes.default.svc
      namespace: library-dev
    - server: https://kubernetes.default.svc
      namespace: library-prod
  clusterResourceWhitelist: []
  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: &quot;&quot;
      kind: Service
    - group: &quot;&quot;
      kind: ConfigMap</code></pre>
<p>이 예제는 허용한 리소스 종류만 사용하는 앱 구성에 맞춘다. Secret·Ingress 등 추가 리소스가 필요하다면 팀 정책에 맞게 범위를 설계한다. 교육기관의 관리자 정책을 사용자 예제로 복사하지 않는다.</p>
<p><code>sourceRepos: ['*']</code>는 모든 소스를 허용하는 넓은 설정이다. <code>clusterResourceBlacklist</code>·<code>namespaceResourceBlacklist</code>는 금지할 종류를 정한다. whitelist와 blacklist의 역할 및 프로젝트 정책의 정확한 동작을 구분한다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/projects/">AppProject 공식 설명</a></p>
<h3 id="5-8-application의-세-질문">5-8. Application의 세 질문</h3>
<p>Application을 만들 때는 세 가지를 먼저 정한다.</p>
<ol>
<li><strong>어떤 프로젝트 정책을 따르는가?</strong> → <code>project</code></li>
<li><strong>어디의 어떤 배포 정의를 읽는가?</strong> → <code>source</code></li>
<li><strong>어디에 배포하는가?</strong> → <code>destination</code></li>
</ol>
<pre><code class="language-yaml"># application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: library-dev
  namespace: argocd
spec:
  project: library-project
  source:
    repoURL: https://github.com/YOUR_TEAM/library-gitops.git
    targetRevision: gitops
    path: kustomize/overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: library-dev
  syncPolicy: {}</code></pre>
<p>핵심 필드를 읽어 보자.</p>
<table>
<thead>
<tr>
<th>필드</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>metadata.namespace: argocd</code></td>
<td>Application 객체가 있는 Argo CD 관리 네임스페이스</td>
</tr>
<tr>
<td><code>project</code></td>
<td>적용할 AppProject</td>
</tr>
<tr>
<td><code>repoURL</code></td>
<td>배포 정의를 읽을 저장소</td>
</tr>
<tr>
<td><code>targetRevision</code></td>
<td>Git 브랜치·태그·커밋 등 대상 리비전</td>
</tr>
<tr>
<td><code>path</code></td>
<td>선택한 Git 리비전 안에서 읽을 디렉터리</td>
</tr>
<tr>
<td><code>destination.server</code></td>
<td>배포할 클러스터 API 대상</td>
</tr>
<tr>
<td><code>destination.namespace</code></td>
<td>앱 리소스의 배포 네임스페이스</td>
</tr>
</tbody></table>
<p><code>metadata.namespace</code>와 <code>destination.namespace</code>는 다르다. 전자는 신청서를 보관할 사무실이고 후자는 실제 일을 수행할 장소다.</p>
<p><code>https://kubernetes.default.svc</code>는 Argo CD가 실행되는 클러스터 내부 대상을 가리키는 일반적인 주소다. 사용자의 노트북에서 웹 브라우저로 접속할 주소가 아니다.</p>
<p>Git 소스에서 <code>targetRevision</code>은 단지 “버전 숫자”가 아니다. 브랜치·커밋 등을 고른다. Helm 저장소를 소스로 사용하는 경우에는 차트 버전 의미가 될 수 있으므로 소스 유형도 함께 확인한다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/application-specification/">Application 필드 안내</a></p>
<h3 id="5-9-repository와-cluster-등록">5-9. Repository와 Cluster 등록</h3>
<p>Argo CD는 배포 정의를 읽을 Repository와 배포 대상 Cluster의 접속 정보를 알아야 한다.</p>
<table>
<thead>
<tr>
<th>등록 대상</th>
<th>제공할 정보</th>
<th>주의할 점</th>
</tr>
</thead>
<tbody><tr>
<td>Repository</td>
<td>URL·접근 방식·필요한 인증</td>
<td>소스를 읽을 권한</td>
</tr>
<tr>
<td>Cluster</td>
<td>API 대상·인증·TLS 정보</td>
<td>리소스를 관리할 권한</td>
</tr>
</tbody></table>
<p>수업에서는 화면으로 저장소를 연결하고, 선언형 YAML로 저장소·클러스터 정보를 등록하는 방법도 소개한다.</p>
<p>Repository·Cluster 정보는 Argo CD가 식별하는 label이 있는 Kubernetes Secret 등으로 관리할 수 있다.</p>
<pre><code class="language-text">argocd.argoproj.io/secret-type: repository
argocd.argoproj.io/secret-type: cluster</code></pre>
<p>공개 글에는 실제 접속 Secret을 넣지 않는다. Base64로 표현했다고 비밀이 안전하게 공개 가능한 문자열로 바뀌는 것도 아니다.</p>
<p>클러스터 등록에는 <code>argocd cluster add</code> 같은 CLI 방식과 선언형 방식이 있다. UI에서 목록을 확인하는 일과 새 클러스터 접속 정보를 등록하는 일은 구분한다. “어떤 환경에서도 Secret을 손으로 작성하는 한 가지 방법만 가능하다”고 일반화하지 않는다. <a href="https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/">Argo CD 선언형 구성 안내</a></p>
<h3 id="5-10-ui로-application-등록하기">5-10. UI로 Application 등록하기</h3>
<p>교재의 화면 실습은 다음 정보가 서로 맞는지 확인하는 과정이다.</p>
<ol>
<li>Repository 연결을 등록하고 연결 상태를 확인한다.</li>
<li>Application 이름과 AppProject를 선택한다.</li>
<li>Repository URL·리비전·경로를 지정한다.</li>
<li>대상 클러스터와 네임스페이스를 지정한다.</li>
<li>수동 또는 자동 동기화 정책을 정한다.</li>
<li>생성 결과와 리소스 트리를 확인한다.</li>
</ol>
<p>이름만 입력하는 것이 아니라 <strong>소스와 목적지를 정확히 연결하는 작업</strong>이다. 같은 저장소에서도 다른 브랜치·경로를 지정하면 다른 구성이 읽힌다.</p>
<h3 id="5-11-수동-동기화와-자동-동기화">5-11. 수동 동기화와 자동 동기화</h3>
<p>수동 모드에서는 변경을 발견하고 사용자가 Sync를 실행한다. 자동 모드에서는 정책에 따라 Argo CD가 동기화를 수행한다.</p>
<pre><code class="language-yaml"># Application spec 아래에 넣는 자동 동기화 설정 조각
syncPolicy:
  automated:
    prune: false
    selfHeal: true
    allowEmpty: false</code></pre>
<p>기본 학습 예제는 자동 삭제를 끈 상태로 동기화를 이해할 수 있게 구성했다. 교재의 <code>prune: true</code>는 삭제 정책까지 켜는 선택이므로 의미를 알고 적용한다.</p>
<p>최신 버전에서는 <code>automated.enabled</code>를 명시할 수도 있다. 생략 여부·지원 버전에 따른 의미는 사용하는 버전의 문서를 확인한다. ApplicationSet이 관리하는 Application은 생성 정책과의 관계도 고려해야 한다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/">자동 동기화 정책</a></p>
<h3 id="5-12-prune-selfheal-allowempty">5-12. prune, selfHeal, allowEmpty</h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>prune</code></td>
<td>이 Application이 관리하던 리소스 중 소스에서 제거된 것을 정리</td>
<td>Git에서 제거한 Service를 삭제</td>
</tr>
<tr>
<td><code>selfHeal</code></td>
<td>실제 리소스만 변경된 경우에도 Git 상태에 맞추도록 자동 조정</td>
<td>직접 바꾼 replica를 Git 값으로 맞춤</td>
</tr>
<tr>
<td><code>allowEmpty</code></td>
<td>자동 prune 상황에서 목표 리소스가 0개가 되는 것을 허용할지</td>
<td>전체 제거를 의도한 배포 변경</td>
</tr>
</tbody></table>
<p><code>prune: true</code>는 Git에 없는 클러스터 전체 리소스를 무차별 삭제한다는 뜻이 아니다. 해당 Application의 관리 범위와 정책을 기준으로 판단한다.</p>
<p><code>selfHeal</code>도 수정된 값을 즉시 무조건 복원하는 마법은 아니다. 비교·동기화 주기와 권한·리소스 조건의 영향을 받는다.</p>
<p><code>allowEmpty: false</code>는 자동 동기화에서 예상치 못한 전체 제거를 막는 보호 장치다. <strong>Git 경로 자체가 사라져 읽기 오류가 난 상황과, 유효한 소스가 빈 리소스 집합을 만든 상황은 다르다.</strong> 이 값이 모든 수동 삭제까지 막는 것도 아니다.</p>
<h3 id="5-13-sync-options-적용·삭제-방식-조정">5-13. Sync Options: 적용·삭제 방식 조정</h3>
<table>
<thead>
<tr>
<th>옵션</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>ApplyOutOfSyncOnly=true</code></td>
<td>선택적으로 차이가 있는 리소스를 적용하는 방식</td>
</tr>
<tr>
<td><code>PrunePropagationPolicy=foreground</code></td>
<td>삭제할 리소스의 종속 리소스 정리를 기다리는 방식</td>
</tr>
<tr>
<td><code>PrunePropagationPolicy=background</code></td>
<td>부모 삭제 후 종속 리소스를 백그라운드에서 정리</td>
</tr>
<tr>
<td><code>PrunePropagationPolicy=orphan</code></td>
<td>종속 리소스를 남기는 방식</td>
</tr>
<tr>
<td><code>Replace=true</code></td>
<td>일반 apply 대신 replace/create 방식을 사용. 영향 범위 검토 필요</td>
</tr>
<tr>
<td><code>Validate=false</code></td>
<td>적용 시 일부 스키마 검증을 끄는 선택. 만능 오류 해결책 아님</td>
</tr>
</tbody></table>
<p>여기서 교재의 예제 오타를 구분하자. foreground를 설명하는 <code>kubectl delete</code> 예시에도 <code>--cascade=background</code>가 적혀 있다. <strong>Foreground 예시에는 <code>--cascade=foreground</code>가 맞고, Background는 <code>--cascade=background</code>다.</strong></p>
<p>변경된 리소스만 적용하는 선택적 동기화는 Hook 등과의 관계도 확인한다. <code>Replace=true</code>를 모든 앱의 기본 설정으로 복사하면 업데이트 중 서비스에 영향을 줄 수 있다. 필요한 이유가 있는 옵션만 선택한다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/sync-options/">Sync Options 공식 설명</a></p>
<h3 id="5-14-synced와-healthy는-다른-질문의-답이다">5-14. Synced와 Healthy는 다른 질문의 답이다</h3>
<table>
<thead>
<tr>
<th>상태</th>
<th>질문</th>
</tr>
</thead>
<tbody><tr>
<td>Synced / OutOfSync</td>
<td>Git의 원하는 배포 정의와 실제 리소스 정의가 맞는가?</td>
</tr>
<tr>
<td>Healthy / Progressing / Degraded 등</td>
<td>지원되는 리소스의 상태 점검 기준을 충족하는가?</td>
</tr>
</tbody></table>
<p>이미지 주소가 존재하지 않아도 그 주소가 Deployment에 적용되면 정의 비교는 Synced일 수 있다. 그러나 Pod는 이미지를 가져오지 못해 앱이 실행되지 않을 수 있다.</p>
<pre><code class="language-text">Git 설정 적용됨 → Synced
이미지 다운로드 실패 → 앱 준비 실패</code></pre>
<p>Healthy도 모든 사용자 시나리오가 정상이라는 증명은 아니다. 리소스 상태 점검과 실제 API·화면 테스트를 함께 확인한다.</p>
<h3 id="5-15-git-변경에서-실제-앱까지-확인하기">5-15. Git 변경에서 실제 앱까지 확인하기</h3>
<p>교재의 Jenkins → GitOps → Argo CD 검증 순서는 다음과 같다.</p>
<ol>
<li>Jenkins의 코드 checkout·앱 빌드·테스트 결과를 본다.</li>
<li>이미지 레지스트리에 정확한 이미지·태그가 있는지 확인한다.</li>
<li>배포 Git의 올바른 브랜치·경로가 갱신되었는지 확인한다.</li>
<li>Argo CD가 해당 리비전을 읽었는지 확인한다.</li>
<li>동기화 결과와 상태를 확인한다.</li>
<li>새 Pod의 이미지·준비 상태·로그를 확인한다.</li>
<li>실제 요청이 정상 처리되는지 확인한다.</li>
</ol>
<pre><code class="language-bash">kubectl get application library-dev -n argocd
kubectl get deployment,service,pods -n library-dev
kubectl get deployment library-api -n library-dev \
  -o jsonpath='{.spec.template.spec.containers[*].image}'</code></pre>
<p>CLI가 준비되어 있고 Argo CD에 로그인한 환경에서는 다음으로 세부 상태를 확인할 수 있다.</p>
<pre><code class="language-bash">argocd app get library-dev
argocd app diff library-dev</code></pre>
<p>메뉴의 초록색 표시 하나만으로 전체 파이프라인의 성공을 판단하지 않는다.</p>
<h3 id="5-16-yaml로-application-등록하기">5-16. YAML로 Application 등록하기</h3>
<p>앞의 프로젝트와 Application 예제를 실제 환경에 맞춰 준비한 경우의 명령이다.</p>
<pre><code class="language-bash">kubectl apply -f project.yaml
kubectl apply -f application.yaml
kubectl get applications -n argocd</code></pre>
<p>Application을 등록했다고 아직 자동 동기화하지 않는 예제다. 검토 후 UI나 적절한 CLI로 Sync한다.</p>
<pre><code class="language-bash">argocd app sync library-dev</code></pre>
<p>이 명령은 실제 앱 리소스를 변경할 수 있다. 대상 클러스터·네임스페이스·리소스를 확인하고 자신의 학습 범위에서 실행한다.</p>
<h3 id="5-17-kustomize와-helm을-argo-cd에-연결">5-17. Kustomize와 Helm을 Argo CD에 연결</h3>
<p>Kustomize 소스라면 <code>path</code>가 올바른 <code>kustomization.yaml</code>이 있는 디렉터리를 가리켜야 한다.</p>
<pre><code class="language-yaml">source:
  repoURL: https://github.com/YOUR_TEAM/library-gitops.git
  targetRevision: gitops
  path: kustomize/overlays/dev</code></pre>
<p>Helm 차트가 Git 안에 있으면 그 차트 경로를 지정하고 values를 연결할 수 있다. 아래는 Application의 소스 조각이다.</p>
<pre><code class="language-yaml">source:
  repoURL: https://github.com/YOUR_TEAM/library-gitops.git
  targetRevision: gitops
  path: charts/library-api
  helm:
    valueFiles:
      - values-dev.yaml</code></pre>
<p>이 예제의 values 파일 경로는 선택한 차트 디렉터리 기준으로 준비한다. 원격 Helm 저장소라면 <code>repoURL</code>, <code>chart</code>, 차트 버전 의미의 <code>targetRevision</code>을 사용하는 구성이 된다.</p>
<p>Argo CD가 Helm을 사용해 YAML을 생성하더라도 일반적으로 Helm CLI의 설치·업그레이드 릴리스 관리가 그대로 실행되는 것으로 생각하면 안 된다. 렌더링과 리소스 수명주기 관리의 담당을 구분한다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/helm/">Argo CD의 Helm 사용 방식</a></p>
<h3 id="5-18-applicationset-비슷한-application-여러-개-만들기">5-18. ApplicationSet: 비슷한 Application 여러 개 만들기</h3>
<p>개발·운영 환경에 각각 Application을 손으로 복사하면 URL이나 경로를 잘못 적을 수 있다. ApplicationSet은 <strong>입력 목록 + 템플릿으로 여러 Application을 생성</strong>한다.</p>
<pre><code class="language-yaml"># application-set.yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: library-environments
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions:
    - missingkey=error
  generators:
    - list:
        elements:
          - environment: dev
            path: kustomize/overlays/dev
            namespace: library-dev
          - environment: prod
            path: kustomize/overlays/prod
            namespace: library-prod
  template:
    metadata:
      name: 'library-{{.environment}}'
    spec:
      project: library-project
      source:
        repoURL: https://github.com/YOUR_TEAM/library-gitops.git
        targetRevision: gitops
        path: '{{.path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{.namespace}}'
      syncPolicy: {}</code></pre>
<p>이 예제는 Go Template 방식을 명시한 학습용 구성이다. 앞에서 만든 동일 이름의 Application을 수동 방식과 ApplicationSet 방식으로 동시에 관리하지 않는다.</p>
<pre><code class="language-text">入力 dev  → library-dev Application
入力 prod → library-prod Application</code></pre>
<p><code>generators</code>가 입력을 만들고 <code>template</code>이 Application의 형태를 만든다. 교재의 기존 템플릿 표현과 Go Template 표현을 섞어 사용하지 않는다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/">ApplicationSet 공식 설명</a>, <a href="https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/GoTemplate/">Go Template 설정</a></p>
<p>멀티 클러스터에서는 입력에 클러스터 대상도 넣을 수 있다. 먼저 해당 클러스터를 Argo CD에 등록하고 AppProject 정책이 허용하는지 확인한다. ApplicationSet이 접속 권한 없는 클러스터를 자동으로 관리하게 만드는 것은 아니다.</p>
<h3 id="5-19-롤백과-삭제의-범위를-구분하자">5-19. 롤백과 삭제의 범위를 구분하자</h3>
<p>GitOps에서는 잘못된 배포 변경을 Git에서 되돌리고 동기화하는 방식으로 복구를 진행할 수 있다.</p>
<pre><code class="language-bash"># 배포 저장소에서 해당 변경을 검토한 뒤 실행하는 예시
git revert DEPLOY_COMMIT_ID
git push origin gitops</code></pre>
<p><code>DEPLOY_COMMIT_ID</code>는 실제로 되돌릴 배포 변경 커밋으로 바꾼다. 이것은 과거 이미지를 다시 지정하는 배포 변경이다. DB 데이터, 외부 결제, 이미 보낸 메시지가 자동으로 과거로 돌아가지는 않는다. 이미지가 남아 있고 데이터 구조와 호환되는지 확인한다.</p>
<p>Application 객체만 삭제하는 경우와 앱 리소스까지 연쇄 삭제하는 경우도 다르다. <code>resources-finalizer.argocd.argoproj.io</code> 같은 finalizer와 삭제 옵션이 동작에 영향을 준다. 앞의 기본 Application 예제에는 연쇄 삭제 finalizer를 넣지 않았다.</p>
<p><strong>정리할 Application과 유지할 리소스 범위, 동기화 관리 주체를 확인한 뒤 삭제한다.</strong> ApplicationSet이 관리하는 객체를 직접 삭제하면 생성 정책에 따라 다시 만들어질 수도 있다. <a href="https://argo-cd.readthedocs.io/en/stable/user-guide/app_deletion/">Application 삭제 정책</a></p>
<h3 id="5-20-적용-사례를-볼-때-질문할-것">5-20. 적용 사례를 볼 때 질문할 것</h3>
<p>교재에는 외부 기술 사례 소개가 있다. 특정 기업의 이름이나 도입 사실만 외우기보다 다음을 확인하자.</p>
<ul>
<li>어느 서비스를 어떤 환경에 배포하는가?</li>
<li>누가 소스와 배포 설정을 변경하는가?</li>
<li>변경 검토·승인 단계가 있는가?</li>
<li>여러 클러스터의 차이는 어떻게 관리하는가?</li>
<li>장애 시 무엇을 기준으로 복구하는가?</li>
</ul>
<p>사례의 숫자나 효과를 자신의 프로젝트가 달성한 성과처럼 옮기지 않는다.</p>
<h3 id="5-21-이-파트-확인-질문">5-21. 이 파트 확인 질문</h3>
<p><strong>Q. Argo CD는 앱 소스를 이미지로 빌드하는 도구일까?</strong></p>
<p>A. 이 구성에서는 배포 정의와 실제 Kubernetes 상태를 비교·동기화하는 CD 담당이다.</p>
<p><strong>Q. Synced인데 ImagePullBackOff가 발생할 수 있을까?</strong></p>
<p>A. 배포 정의는 같아도 이미지 실행 준비가 실패할 수 있다.</p>
<p><strong>Q. Application과 ApplicationSet은 무엇이 다를까?</strong></p>
<p>A. 하나의 배포 연결 정의와 여러 연결 정의를 생성·관리하는 상위 구성이다.</p>
<hr />
<h2 id="6-나만의-파이프라인-구성하기">6. 나만의 파이프라인 구성하기</h2>
<h3 id="6-1-먼저-서비스의-구조를-그리자">6-1. 먼저 서비스의 구조를 그리자</h3>
<p>교재의 마지막 실습은 자신의 Frontend·Backend·AI 서비스 등을 CI/CD 흐름으로 연결하는 것이다.</p>
<pre><code class="language-text">브라우저
  ├─ HTML·CSS·JavaScript 요청 → Frontend 웹서버
  └─ API 요청 → Backend 또는 AI 서비스</code></pre>
<ul>
<li><strong>Frontend</strong>: 화면과 사용자 상호작용.</li>
<li><strong>Backend</strong>: API·업무 로직·데이터 처리.</li>
<li><strong>AI 서비스</strong>: 모델 추론 등의 기능을 API로 제공할 수 있는 별도 서비스.</li>
</ul>
<p>단순 파일 제공과 API 응답은 역할이 다르다. 브라우저에서 Kubernetes 내부 Service 이름을 직접 호출할 수 있다고 생각하면 안 된다. Ingress·프록시·외부 접근 주소를 설계해야 한다.</p>
<h3 id="6-2-정적-자원과-동적-요청">6-2. 정적 자원과 동적 요청</h3>
<p>정적 자원은 준비한 파일을 제공하는 HTML·CSS·이미지 등이다. JavaScript 파일도 파일 제공 자체는 정적일 수 있지만, 브라우저에서 실행되며 API 요청과 화면 갱신을 수행한다.</p>
<pre><code class="language-text">브라우저 → index.html·app.js 가져오기
             ↓ JavaScript 실행
브라우저 → /api/books 요청
             ↓
Backend → JSON 응답
             ↓
브라우저 → 책 목록 화면 갱신</code></pre>
<p>nginx의 일반적인 정적 파일 경로 예시는 <code>/usr/share/nginx/html</code>이다. 교재 도식에 있는 철자·경로 표기를 그대로 코드에 복사하지 말고 실제 이미지·설정 파일을 확인한다.</p>
<h3 id="6-3-앱이-없다면-작은-예제로-시작">6-3. 앱이 없다면 작은 예제로 시작</h3>
<p>교재의 샘플 컨테이너를 활용하거나 기능 하나인 학습용 앱으로 시작할 수 있다.</p>
<ul>
<li>Backend: <code>/health</code>, <code>/api/books</code>처럼 간단한 API.</li>
<li>Frontend: API를 호출해 결과를 표시하는 화면.</li>
<li>AI 서비스: 준비된 모델·실행 환경에서 작은 추론 API.</li>
</ul>
<p>처음부터 모든 기능을 한 파이프라인에 넣지 말고 하나의 앱과 개발 환경으로 흐름을 확인한 뒤 확장한다. 예제·프로토타입을 실제 운영 성과라고 표현하지 않는다.</p>
<h3 id="6-4-저장소의-역할을-정하기">6-4. 저장소의 역할을 정하기</h3>
<pre><code class="language-text">library-source/
├── backend/
│   ├── src/
│   ├── Dockerfile
│   └── Jenkinsfile
└── frontend/
    ├── src/
    ├── Dockerfile
    └── Jenkinsfile

library-gitops/
└── kustomize/
    ├── base/
    └── overlays/
        ├── dev/
        └── prod/</code></pre>
<p>한 저장소에 여러 앱을 모으는 방식과 앱마다 저장소를 나누는 방식 모두 가능하다. 중요한 것은 변경 감지 범위, 작업 디렉터리, 이미지 저장소, 배포 경로를 명확히 하는 것이다.</p>
<p>공개 저장소에는 실제 환경변수 파일·토큰·kubeconfig·생성된 인증 파일을 넣지 않는다. 단순히 <code>.gitignore</code>를 추가했다고 이미 추적되거나 과거 커밋에 있던 비밀값이 사라지는 것도 아니다.</p>
<h3 id="6-5-설계표를-먼저-채우자">6-5. 설계표를 먼저 채우자</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>학습용 계획 예시</th>
</tr>
</thead>
<tbody><tr>
<td>앱</td>
<td>도서관 Backend API</td>
</tr>
<tr>
<td>소스 브랜치</td>
<td>main</td>
</tr>
<tr>
<td>실행 노드</td>
<td>Java·Git·빌더가 있는 CI Agent</td>
</tr>
<tr>
<td>테스트</td>
<td>단위 테스트·핵심 API 확인</td>
</tr>
<tr>
<td>이미지</td>
<td>registry.example.com/demo/library-api</td>
</tr>
<tr>
<td>태그</td>
<td>CI 실행 번호 + 소스 커밋</td>
</tr>
<tr>
<td>배포 기준</td>
<td>배포 저장소의 gitops 브랜치</td>
</tr>
<tr>
<td>배포 경로</td>
<td>kustomize/overlays/dev</td>
</tr>
<tr>
<td>CD</td>
<td>Argo CD Application</td>
</tr>
<tr>
<td>대상</td>
<td>준비된 개발 네임스페이스</td>
</tr>
<tr>
<td>성공 기준</td>
<td>새 이미지 실행·핵심 API 정상 응답</td>
</tr>
<tr>
<td>복구 기준</td>
<td>이전 정상 이미지의 배포 변경</td>
</tr>
</tbody></table>
<p>값을 적지 못한 항목은 아직 연결되지 않은 부분이다. 특히 앱 포트·Service targetPort·Probe 경로를 함께 맞춘다.</p>
<h3 id="6-6-구현-순서">6-6. 구현 순서</h3>
<ol>
<li>로컬에서 앱을 실행하고 핵심 기능을 확인한다.</li>
<li>Dockerfile로 이미지가 만들어지고 실행되는지 확인한다.</li>
<li>레지스트리로 push하고 내려받을 수 있는지 확인한다.</li>
<li>Kubernetes 배포 정의를 생성·검증한다.</li>
<li>수동 배포로 Service·포트·Probe·환경 설정을 확인한다.</li>
<li>Jenkins에서 앱 빌드·테스트·이미지 push를 연결한다.</li>
<li>Jenkins에서 배포 Git의 이미지 정보를 갱신한다.</li>
<li>Argo CD의 소스·대상·동기화 정책을 연결한다.</li>
<li>Poll SCM 또는 Webhook을 붙인다.</li>
<li>코드 변경부터 앱 응답까지 전체 흐름을 다시 확인한다.</li>
</ol>
<p>처음부터 자동 Trigger를 켜면 같은 실패가 반복되어 원인을 찾기 어렵다. 수동 실행으로 단계별 입력·출력을 확인한 뒤 연결하면 문제를 좁히기 쉽다.</p>
<h3 id="6-7-배포-gitops-최소-예제">6-7. 배포 GitOps 최소 예제</h3>
<p>아래는 Argo CD Application이 읽을 수 있는 Kustomize 구성 예제다. 앞의 Application이 지정한 <code>kustomize/overlays/dev</code> 경로에 연결한다.</p>
<pre><code class="language-text">library-gitops/
└── kustomize/
    ├── base/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── kustomization.yaml
    └── overlays/
        └── dev/
            └── kustomization.yaml</code></pre>
<p><strong>① <code>kustomize/base/deployment.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: apps/v1
kind: Deployment
metadata:
  name: library-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: library-api
  template:
    metadata:
      labels:
        app: library-api
    spec:
      containers:
        - name: api
          image: registry.example.com/demo/library-api:1.0.0
          ports:
            - name: http
              containerPort: 8080</code></pre>
<p><strong>② <code>kustomize/base/service.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: v1
kind: Service
metadata:
  name: library-api
spec:
  selector:
    app: library-api
  ports:
    - port: 8080
      targetPort: http</code></pre>
<p><strong>③ <code>kustomize/base/kustomization.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml</code></pre>
<p><strong>④ <code>kustomize/overlays/dev/kustomization.yaml</code></strong></p>
<pre><code class="language-yaml">apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: library-dev
resources:
  - ../../base
images:
  - name: registry.example.com/demo/library-api
    newTag: &quot;build-12-a1b2c3d4&quot;</code></pre>
<p><code>library-gitops</code> 폴더에서 로컬 YAML을 생성한다.</p>
<pre><code class="language-bash">kubectl kustomize kustomize/overlays/dev</code></pre>
<p>이 구성은 구조와 이미지 태그 연결을 연습하는 최소 예제다. 실제로 배포하려면 위 주소의 앱 이미지, 8080 포트의 서버, 네임스페이스, 필요 시 pull Secret을 준비하고 Probe·자원·보안 설정을 보완한다. 가상의 이미지를 그대로 적용하면 이미지 다운로드에 실패한다.</p>
<p>앞의 ApplicationSet 예제를 사용하려면 prod 경로도 별도로 준비해야 한다. 이 최소 예제는 dev만 제공한다.</p>
<h3 id="6-8-문제를-위치별로-좁히는-방법">6-8. 문제를 위치별로 좁히는 방법</h3>
<pre><code class="language-text">Git push 성공?
  ↓
CI Trigger 전달·감지 성공?
  ↓
CI 앱 검증 성공?
  ↓
이미지 build·push 성공?
  ↓
배포 Git 변경 성공?
  ↓
Argo CD 소스 읽기·동기화 성공?
  ↓
Pod 준비·Service 연결 성공?
  ↓
실제 API·화면 정상?</code></pre>
<table>
<thead>
<tr>
<th>오류·현상</th>
<th>확인할 위치</th>
</tr>
</thead>
<tbody><tr>
<td>경로를 찾지 못함</td>
<td><code>pwd</code>, 상대 경로, 실제 파일명</td>
</tr>
<tr>
<td>command not found</td>
<td>실행 노드의 도구·PATH</td>
</tr>
<tr>
<td>Git의 브랜치·파일 없음</td>
<td>원격 저장소의 실제 리비전·Script Path</td>
</tr>
<tr>
<td>Maven은 실패했는데 완료 문구가 나옴</td>
<td>Shell 종료 코드·오류 처리</td>
</tr>
<tr>
<td>ErrImagePull / ImagePullBackOff</td>
<td>레지스트리·태그·인증·플랫폼</td>
</tr>
<tr>
<td>CrashLoopBackOff</td>
<td>앱 시작 로그·실행 명령·환경 설정</td>
</tr>
<tr>
<td>Pod Running인데 READY 0/1</td>
<td>Readiness Probe·앱 준비 상태</td>
</tr>
<tr>
<td>Service 접속 실패</td>
<td>selector·endpoint·포트·네트워크</td>
</tr>
<tr>
<td>Git 변경했는데 Argo CD가 그대로</td>
<td>repoURL·targetRevision·path·동기화 정책</td>
</tr>
<tr>
<td>kubectl은 되는데 Jenkins에서 Forbidden</td>
<td>실행 환경의 실제 인증·RBAC</td>
</tr>
</tbody></table>
<p>교재에서 복사한 긴 대시 <code>–</code>는 명령 옵션의 하이픈 <code>-</code>와 다르다. 이 글의 실행 예제는 일반 하이픈을 사용한다. 예시 Pod 이름을 자신의 리소스 이름으로 바꾸고 컨테이너 로그는 <code>-c</code>로 선택한다.</p>
<h3 id="6-9-완성-기준은-실제-사용자-기능이다">6-9. 완성 기준은 실제 사용자 기능이다</h3>
<p>“파이프라인이 초록색이다”만으로 완료라고 하지 말자. 어떤 단계가 실제로 무엇을 했는지 확인해야 한다.</p>
<ul>
<li>테스트 명령을 실행하고 결과를 확인했다.</li>
<li>만든 이미지와 배포한 이미지가 같다.</li>
<li>Argo CD가 의도한 Git 리비전을 반영했다.</li>
<li>새 앱이 요청을 정상 처리한다.</li>
<li>실패했을 때 원인을 찾고 복구할 절차가 있다.</li>
</ul>
<p>측정하지 않은 속도 향상·장애 감소 수치를 성과로 적지 않는다. 실제 실행 로그와 검증 결과로 설명한다.</p>
<hr />
<h2 id="7-퀴즈-2일차-복습">7. 퀴즈: 2일차 복습</h2>
<p>교재의 2일차 본문에는 별도로 제공된 퀴즈 문항·정답 목록이 없다. 아래는 <strong>모든 파트의 이해를 확인하기 위해 새로 작성한 복습 문제</strong>이며 실제 수업 시험의 정답이라고 주장하지 않는다.</p>
<p>먼저 답을 가리고 자신의 말로 설명해 보자.</p>
<h3 id="7-1-기본-구조">7-1. 기본 구조</h3>
<p><strong>Q1. Jenkins Controller와 Agent는 어떤 차이가 있을까?</strong></p>
<p>A. Controller는 작업을 관리하고, Agent는 코드 가져오기·빌드 등 실제 작업을 수행한다. Controller의 직접 실행 기능은 별도 설정과 운영 판단이다.</p>
<p><strong>Q2. Job, Build, Stage, Step을 큰 것부터 설명해 보자.</strong></p>
<p>A. Job은 작업 정의, Build는 한 번의 실행이다. Pipeline 안의 Stage는 큰 단계, Step은 실제 명령이다.</p>
<p><strong>Q3. Jenkinsfile과 JCasC는 같은 파일일까?</strong></p>
<p>A. 아니다. Jenkinsfile은 프로젝트 작업 흐름, JCasC는 Jenkins 시스템 설정을 주로 정의한다.</p>
<p><strong>Q4. Pipeline의 Test 단계에서 echo만 실행했는데 성공했다. 앱 테스트가 완료됐을까?</strong></p>
<p>A. 아니다. 로그 출력이 성공한 것이고 실제 테스트 명령·결과가 필요하다.</p>
<p><strong>Q5. agent any는 모든 노드에서 동시에 실행한다는 뜻일까?</strong></p>
<p>A. 아니다. 사용 가능한 실행 노드에 작업을 배정한다.</p>
<h3 id="7-2-환경과-인증">7-2. 환경과 인증</h3>
<p><strong>Q6. Jenkins 노트북 설정에 Docker가 있는데 Agent에 없어도 Docker 빌드가 될까?</strong></p>
<p>A. 실행 위치에서 Docker 또는 적절한 빌더를 사용할 수 있어야 한다.</p>
<p><strong>Q7. environment의 값과 Credentials의 비밀값을 같은 방식으로 공개해도 될까?</strong></p>
<p>A. 아니다. 실제 인증정보는 보관 기능과 필요한 범위의 주입을 사용하고 공개하지 않는다.</p>
<p><strong>Q8. credentialsId는 비밀번호일까?</strong></p>
<p>A. 저장된 인증정보를 참조하는 식별자다.</p>
<p><strong>Q9. <code>sh 'echo &quot;$APP_NAME&quot;'</code>에서 환경변수는 누가 치환할까?</strong></p>
<p>A. Groovy는 문자열을 그대로 전달하고 실행 Shell이 환경변수를 치환한다.</p>
<p><strong>Q10. withCredentials 블록이 끝나면 Docker의 인증 파일까지 자동으로 사라질까?</strong></p>
<p>A. 바인딩 범위가 끝나는 것과 외부 프로그램이 만든 파일 삭제는 다르다. 저장 위치와 정리가 필요하다.</p>
<h3 id="7-3-빌드와-자동-실행">7-3. 빌드와 자동 실행</h3>
<p><strong>Q11. Maven 빌드와 이미지 빌드는 같은 단계일까?</strong></p>
<p>A. 앱 실행 결과물을 만드는 작업과 이를 실행 환경과 함께 포장하는 작업이다.</p>
<p><strong>Q12. 빌더는 <code>1.0.kaniko-pod</code>를 push했지만 배포는 <code>1.0</code>을 요청한다. 자동으로 같은 이미지가 선택될까?</strong></p>
<p>A. 아니다. 서로 다른 태그다. 정확한 이미지 주소를 단계 간 공유해야 한다.</p>
<p><strong>Q13. Poll SCM은 정해진 간격마다 무조건 빌드할까?</strong></p>
<p>A. 일반적으로 변경을 확인했을 때 빌드를 예약한다.</p>
<p><strong>Q14. Cron의 H는 매 실행마다 새 난수를 뽑는 기호일까?</strong></p>
<p>A. 아니다. Job 등에 기반한 해시로 조회·실행 시점을 분산한다.</p>
<p><strong>Q15. 외부 GitHub가 사용자의 localhost Jenkins에 바로 알림을 보낼 수 있을까?</strong></p>
<p>A. 일반적으로 없다. 외부에서 도달할 수 있는 적절한 수신 경로가 필요하다.</p>
<h3 id="7-4-kubernetes와-gitops">7-4. Kubernetes와 GitOps</h3>
<p><strong>Q16. 노트북 kubectl이 성공하면 Jenkins Pod도 같은 권한일까?</strong></p>
<p>A. 아니다. 실행 위치의 kubeconfig·ServiceAccount·RBAC를 확인한다.</p>
<p><strong>Q17. Argo CD Application의 metadata.namespace와 destination.namespace는 같은 의미일까?</strong></p>
<p>A. Application 관리 위치와 앱 리소스 배포 위치로 다르다.</p>
<p><strong>Q18. source.path는 내 노트북의 절대 경로일까?</strong></p>
<p>A. 선택한 Git 소스의 해당 리비전 안에서 읽을 경로다.</p>
<p><strong>Q19. Argo CD가 Synced면 앱도 정상 실행된다고 보장할까?</strong></p>
<p>A. 아니다. 배포 정의의 동기화와 앱 상태·기능 검증은 다르다.</p>
<p><strong>Q20. prune은 Git에 없는 클러스터 전체 리소스를 삭제할까?</strong></p>
<p>A. 해당 Application의 관리 리소스와 정책을 기준으로 정리하는 것이다.</p>
<p><strong>Q21. allowEmpty=false가 Git 경로 삭제의 모든 오류를 막을까?</strong></p>
<p>A. 아니다. 자동 prune의 빈 목표 보호와 소스 경로 읽기 오류를 구분한다.</p>
<p><strong>Q22. Git revert로 배포 정의를 되돌리면 DB 데이터도 되돌아갈까?</strong></p>
<p>A. 아니다. 데이터 복구와 앱·DB 호환성은 별도로 관리한다.</p>
<p><strong>Q23. ApplicationSet은 무엇을 생성할까?</strong></p>
<p>A. 입력과 템플릿으로 여러 Application을 생성·관리한다.</p>
<p><strong>Q24. AppProject는 무엇을 제한할 수 있을까?</strong></p>
<p>A. 소스 저장소, 배포 대상, 리소스 종류와 역할 정책 등의 범위를 관리한다.</p>
<p><strong>Q25. 최종 완료 기준으로 가장 적절한 것은?</strong></p>
<p>A. 의도한 변경이 검사·이미지·Git·동기화를 거쳐 실행되고, 실제 사용자 요청까지 검증한 것이다.</p>
<hr />
<h2 id="8-조별-토론하기">8. 조별 토론하기</h2>
<p>아래는 교재의 학습 주제를 연결하기 위해 추가한 토론안이다. 실제 조별 발언이나 토론 결과를 기록한 내용은 아니다.</p>
<h3 id="8-1-상황-a-ci는-성공인데-서비스는-실패">8-1. 상황 A: CI는 성공인데 서비스는 실패</h3>
<p><strong>상황</strong>: Jenkins는 성공했고 이미지도 올라갔지만 사용자가 책 검색을 할 수 없다.</p>
<p><strong>토론 질문</strong>: 성공한 범위와 확인하지 않은 범위를 어떻게 구분할까?</p>
<p><strong>생각할 방향</strong>: 배포 Git 갱신 여부, Argo CD 리비전·동기화, 이미지 pull, 앱 시작, Service·Ingress, 실제 API 응답을 순서대로 확인한다. CI가 echo만 실행했는지도 확인한다.</p>
<h3 id="8-2-상황-b-poll-scm과-webhook-선택">8-2. 상황 B: Poll SCM과 Webhook 선택</h3>
<p><strong>상황</strong>: Jenkins는 노트북에서 실행되고 Git 저장소는 외부 서비스에 있다.</p>
<p><strong>토론 질문</strong>: 어떤 변경 감지 방식이 가능한가? 제약은 무엇인가?</p>
<p><strong>생각할 방향</strong>: Jenkins에서 Git에 접근 가능한지와 Git 서비스에서 Jenkins 수신 주소에 접근 가능한지를 구분한다. 확인 간격, 조회 부하, 알림 전달·검증 방식도 함께 고려한다.</p>
<h3 id="8-3-상황-c-여러-팀이-같은-jenkins를-사용">8-3. 상황 C: 여러 팀이 같은 Jenkins를 사용</h3>
<p><strong>상황</strong>: 서로 다른 프로젝트가 공용 Jenkins와 Kubernetes를 사용한다.</p>
<p><strong>토론 질문</strong>: 작업 공간과 Credentials·배포 권한은 어떻게 나눌까?</p>
<p><strong>생각할 방향</strong>: Folder·Job·Credentials 범위, Agent 구성, RBAC, 네임스페이스, AppProject 정책을 연결한다. 이름을 나누는 것만으로 완전한 보안 격리가 되는 것은 아니다.</p>
<h3 id="8-4-상황-d-두-ci가-같은-배포-브랜치를-수정">8-4. 상황 D: 두 CI가 같은 배포 브랜치를 수정</h3>
<p><strong>상황</strong>: Frontend와 Backend 빌드가 동시에 끝나 같은 GitOps 브랜치에 push한다.</p>
<p><strong>토론 질문</strong>: 변경 충돌과 누락을 어떻게 예방할까?</p>
<p><strong>생각할 방향</strong>: 수정 파일 범위, 최신 브랜치 재확인, 변경 리뷰·PR 방식, 재시도 조건, 프로젝트 전체의 동시 변경 조정 등을 검토한다. 한 Job의 동시 실행 제한만으로 모든 Job의 충돌이 해결되지는 않는다.</p>
<h3 id="8-5-상황-e-운영자가-직접-replica를-변경">8-5. 상황 E: 운영자가 직접 replica를 변경</h3>
<p><strong>상황</strong>: Git에는 2개인데 운영자가 임시로 5개로 바꿨다.</p>
<p><strong>토론 질문</strong>: selfHeal과 충돌할 수 있을까? 응급 변경은 어떻게 기록할까?</p>
<p><strong>생각할 방향</strong>: GitOps의 기준 상태와 수동 변경 정책, 자동 동기화의 범위, 필요한 예외 처리, 최종 Git 반영 절차를 정한다. 다른 자동 확장 도구와 관리 필드가 겹치는지도 확인한다.</p>
<h3 id="8-6-상황-f-잘못된-배포를-복구">8-6. 상황 F: 잘못된 배포를 복구</h3>
<p><strong>상황</strong>: 새 앱 버전 배포 후 오류가 발생했고 DB 스키마도 바뀌었다.</p>
<p><strong>토론 질문</strong>: 이전 이미지로 되돌리면 충분할까?</p>
<p><strong>생각할 방향</strong>: 이전 이미지 보존, DB 호환성, 데이터 백업·마이그레이션, 외부 변경, 실제 사용자 기능 검증을 함께 고려한다.</p>
<h3 id="8-7-상황-g-공개-저장소로-학습-결과-공유">8-7. 상황 G: 공개 저장소로 학습 결과 공유</h3>
<p><strong>상황</strong>: Jenkinsfile, JCasC, Kubernetes YAML을 공개하려고 한다.</p>
<p><strong>토론 질문</strong>: 어떤 정보를 일반화하고 어떤 파일은 제외할까?</p>
<p><strong>생각할 방향</strong>: 실제 계정·토큰·kubeconfig·내부 주소를 제거하고 안전한 예시 값과 주입 방법을 설명한다. 생성 파일·패키지·과거 Git 이력에 비밀값이 포함되어 있는지도 확인한다.</p>
<h3 id="8-8-토론-결과-기록-양식">8-8. 토론 결과 기록 양식</h3>
<pre><code class="language-text">사용자에게 제공할 기능:
현재 수작업 또는 실패 지점:
CI가 담당할 범위:
CD가 담당할 범위:
소스와 배포 설정의 기준 저장소·브랜치·경로:
필요한 도구·권한·인증 방식:
성공 확인 방법:
실패 시 조사·복구 방법:
공개 자료에서 제외할 정보:</code></pre>
<p>도구 이름만 적기보다 “누가 어떤 입력을 받아 무엇을 만들고 어떻게 확인하는가”까지 연결해 보자.</p>
<hr />
<h2 id="2일차-전체-연결-지도">2일차 전체 연결 지도</h2>
<pre><code class="language-text">개발자 코드 변경
      ↓ commit·push
소스 Git 저장소
      ↓ Poll SCM 또는 Webhook
Jenkins Controller → 실행 가능한 Agent
      ↓
코드 가져오기 → 앱 빌드·테스트 → 이미지 빌드
      ↓
이미지 레지스트리로 push
      ↓ 같은 이미지 정보를 기록
배포 Git 저장소·브랜치·경로
      ↓ 읽기·비교
Argo CD Application
      ↓ AppProject 정책·동기화 정책
Kubernetes 리소스 적용
      ↓ 이미지 가져오기·앱 시작
Pod 준비 → Service·Ingress 연결
      ↓
실제 사용자 기능 검증
      ↓
관찰한 결과를 다음 개선에 반영</code></pre>
<h2 id="교재와-설명의-범위">교재와 설명의 범위</h2>
<p>Jenkins CI/CD의 두 파트는 시간표에 맞춰 기본 구조·문법과 실제 자동화 연결로 나눴다. Jenkins CI·Poll SCM, Kubernetes Jenkins, GitOps, 개인 파이프라인은 교재의 순서를 따른다. 교재의 반복 화면은 각각의 입력·실행·검증 단계로 설명했고, 민감한 화면은 일반적인 개념과 안전한 예제로 바꿨다.</p>
<p>퀴즈와 토론은 교재에 별도의 문항·결과가 없어 학습용으로 새로 작성했다. JCasC의 플러그인 설치 범위, Poll SCM과 Webhook의 차이, GitOps 성공·복구의 조건, kubeconfig의 context 구분, Argo CD 삭제 옵션 등 혼동하기 쉬운 내용은 공식 문서로 보완했다.</p>
<p>이 글의 예제는 실제 계정·저장소·이미지·클러스터 권한 없이 실행 완료를 보장하는 설치 스크립트가 아니다. 실행한 단계와 검증한 결과를 구분하며 자신의 환경에 맞게 값을 준비해야 한다.</p>