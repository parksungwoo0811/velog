<blockquote>
<p><strong>학습일:</strong> 2026.07.14
<strong>주제:</strong> Git, GitHub, 개발환경 구축, OpenAI Codex</p>
</blockquote>
<hr />
<h1 id="오늘-학습한-내용">오늘 학습한 내용</h1>
<p>오늘은 개발자가 가장 많이 사용하는 <strong>Git과 GitHub의 기본 개념</strong>, 그리고 <strong>VS Code 환경 구축 및 AI를 활용한 개발(Codex CLI)</strong> 에 대해 학습했다.</p>
<p>단순히 명령어를 외우는 것이 아니라 <strong>버전 관리가 왜 필요한지</strong>, <strong>GitHub를 어떻게 활용하는지</strong>, 그리고 <strong>AI와 함께 개발하는 방식(AI Coding)</strong> 을 이해하는 것이 핵심이었다.</p>
<hr />
<h1 id="1-git이란">1. Git이란?</h1>
<p>Git은</p>
<blockquote>
<p><strong>소스코드의 변경 사항을 기록하고 관리하는 버전 관리 시스템(Version Control System)</strong> 이다.</p>
</blockquote>
<p>개발을 하다 보면</p>
<ul>
<li>기능 추가</li>
<li>버그 수정</li>
<li>이전 버전 복원</li>
<li>여러 명이 동시에 개발</li>
</ul>
<p>등이 발생하는데 Git은 이러한 작업을 안전하게 관리해 준다.</p>
<hr />
<h2 id="git을-사용하는-이유">Git을 사용하는 이유</h2>
<p>Git을 사용하면</p>
<ul>
<li>변경 이력 관리</li>
<li>이전 버전 복구</li>
<li>협업</li>
<li>브랜치를 통한 독립적인 개발</li>
<li>충돌(Conflict) 관리</li>
</ul>
<p>등이 가능하다.</p>
<p>즉,</p>
<blockquote>
<p>&quot;파일을 저장하는 것&quot;이 아니라 <strong>프로젝트의 모든 변경 기록을 관리하는 도구</strong>이다.</p>
</blockquote>
<hr />
<h1 id="2-git과-github의-차이">2. Git과 GitHub의 차이</h1>
<p>많은 사람들이 헷갈리는 개념이다.</p>
<table>
<thead>
<tr>
<th>Git</th>
<th>GitHub</th>
</tr>
</thead>
<tbody><tr>
<td>버전 관리 프로그램</td>
<td>Git 저장소를 관리하는 클라우드 서비스</td>
</tr>
<tr>
<td>로컬에서도 사용 가능</td>
<td>원격 저장소 제공</td>
</tr>
<tr>
<td>파일 변경 이력 관리</td>
<td>협업 및 코드 공유</td>
</tr>
</tbody></table>
<p>Git은 프로그램이고</p>
<p>GitHub는 Git 저장소를 인터넷에서 관리하는 서비스이다.</p>
<hr />
<h1 id="3-git의-기본-작업-흐름">3. Git의 기본 작업 흐름</h1>
<p>Git은 아래 순서대로 동작한다.</p>
<pre><code class="language-text">Working Directory
      ↓
git add
      ↓
Staging Area
      ↓
git commit
      ↓
Local Repository
      ↓
git push
      ↓
GitHub(Remote Repository)</code></pre>
<p>이 흐름을 이해하는 것이 Git 학습에서 가장 중요하다.</p>
<hr />
<h1 id="4-자주-사용하는-git-명령어">4. 자주 사용하는 Git 명령어</h1>
<h2 id="저장소-생성">저장소 생성</h2>
<pre><code class="language-bash">git init</code></pre>
<p>현재 폴더를 Git 저장소로 초기화한다.</p>
<hr />
<h2 id="변경-사항-확인">변경 사항 확인</h2>
<pre><code class="language-bash">git status</code></pre>
<p>현재 변경된 파일과 스테이징 상태를 확인한다.</p>
<hr />
<h2 id="파일-추가">파일 추가</h2>
<pre><code class="language-bash">git add .</code></pre>
<p>변경된 파일을 Staging Area에 등록한다.</p>
<hr />
<h2 id="커밋">커밋</h2>
<pre><code class="language-bash">git commit -m &quot;Initial commit&quot;</code></pre>
<p>현재 상태를 하나의 버전으로 저장한다.</p>
<hr />
<h2 id="github-업로드">GitHub 업로드</h2>
<pre><code class="language-bash">git push origin main</code></pre>
<p>로컬 저장소의 변경 내용을 원격 저장소(GitHub)에 업로드한다.</p>
<hr />
<h2 id="최신-내용-가져오기">최신 내용 가져오기</h2>
<pre><code class="language-bash">git pull origin main</code></pre>
<p>원격 저장소의 최신 변경 사항을 가져온다.</p>
<hr />
<h1 id="5-branch브랜치">5. Branch(브랜치)</h1>
<p>브랜치는</p>
<blockquote>
<p><strong>기존 코드에 영향을 주지 않고 독립적으로 개발하는 공간</strong></p>
</blockquote>
<p>이다.</p>
<p>예를 들어</p>
<pre><code class="language-text">main
 ├── feature/login
 ├── feature/signup
 └── feature/payment</code></pre>
<p>각 기능을 별도로 개발한 뒤</p>
<p>완성되면 main에 병합(Merge)한다.</p>
<hr />
<h2 id="git-flow">Git Flow</h2>
<p>대표적인 브랜치 전략이다.</p>
<ul>
<li><strong>main</strong> : 운영(배포) 버전</li>
<li><strong>develop</strong> : 개발 통합 브랜치</li>
<li><strong>feature/</strong> : 기능 개발</li>
<li><strong>release/</strong> : 출시 준비</li>
<li><strong>hotfix/</strong> : 긴급 버그 수정</li>
</ul>
<p>프로젝트 규모에 따라 GitHub Flow(main + feature + Pull Request)처럼 더 단순한 전략을 사용하기도 한다.</p>
<hr />
<h1 id="6-gitignore의-역할">6. .gitignore의 역할</h1>
<p>모든 파일을 Git으로 관리하면 안 된다.</p>
<p>대표적으로</p>
<ul>
<li>API Key</li>
<li>.env</li>
<li>node_modules</li>
<li>로그 파일</li>
</ul>
<p>등은 제외해야 한다.</p>
<pre><code class="language-text">.env
node_modules/</code></pre>
<p><code>.gitignore</code> 파일에 등록하면 Git이 해당 파일을 추적하지 않는다.</p>
<hr />
<h1 id="7-personal-access-tokenpat">7. Personal Access Token(PAT)</h1>
<p>GitHub는 더 이상 계정 비밀번호로 Push를 허용하지 않는다.</p>
<p>대신</p>
<p><strong>Personal Access Token(PAT)</strong> 을 사용한다.</p>
<p>PAT는</p>
<blockquote>
<p>GitHub 계정을 대신하는 인증 토큰</p>
</blockquote>
<p>이다.</p>
<hr />
<h2 id="주의사항">주의사항</h2>
<p>PAT는 비밀번호와 동일하게 취급해야 한다.</p>
<ul>
<li>Git에 커밋 금지</li>
<li>.env에 저장</li>
<li>유출 시 즉시 폐기 후 재발급</li>
</ul>
<p>GitHub CLI(<code>gh auth login</code>)를 이용한 인증도 권장된다.</p>
<hr />
<h1 id="8-vs-code와-live-server">8. VS Code와 Live Server</h1>
<p>HTML 프로젝트를 실행하기 위해</p>
<p><strong>Live Server</strong> 확장을 사용하였다.</p>
<p>장점</p>
<ul>
<li>자동 새로고침</li>
<li>간단한 웹 서버 실행</li>
<li>HTML/CSS/JS 실시간 확인</li>
</ul>
<p>웹 프론트엔드 개발에서 가장 많이 사용하는 확장 프로그램 중 하나이다.</p>
<hr />
<h1 id="9-openai-codex-cli">9. OpenAI Codex CLI</h1>
<p>Codex CLI는</p>
<blockquote>
<p>자연어 명령을 이용해 코드 생성, 수정, 테스트 등을 수행하는 AI 개발 도구</p>
</blockquote>
<p>이다.</p>
<p>설치</p>
<pre><code class="language-bash">npm install -g @openai/codex</code></pre>
<p>실행</p>
<pre><code class="language-bash">codex</code></pre>
<p>최초 실행 시 로그인 후 사용할 수 있다.</p>
<hr />
<h1 id="10-ai와-함께-개발하는-방식vibe-coding">10. AI와 함께 개발하는 방식(Vibe Coding)</h1>
<p>이번 실습에서는 AI에게 다음과 같은 작업을 요청하였다.</p>
<ul>
<li>HTML 생성</li>
<li>코드 설명</li>
<li>코드 리뷰</li>
<li>테스트 수행</li>
<li>새로운 기능 추가</li>
<li>정규표현식 설명</li>
<li>비밀번호 검증 코드 작성</li>
<li>Mermaid 다이어그램 생성</li>
</ul>
<p>이를 통해 AI를 단순한 코드 생성기가 아니라 <strong>개발을 함께 수행하는 협업 도구</strong>로 활용할 수 있음을 확인했다.</p>
<hr />
<h1 id="오늘의-핵심-정리">오늘의 핵심 정리</h1>
<ul>
<li>Git은 소스코드의 버전을 관리하는 시스템이다.</li>
<li>GitHub는 Git 저장소를 원격에서 관리하는 서비스이다.</li>
<li>Git은 <code>add → commit → push</code> 흐름을 이해하는 것이 가장 중요하다.</li>
<li>브랜치를 활용하면 안전하게 기능을 개발할 수 있다.</li>
<li><code>.gitignore</code>를 통해 민감한 정보와 불필요한 파일을 관리 대상에서 제외해야 한다.</li>
<li>GitHub 인증은 PAT 또는 GitHub CLI를 사용하는 것이 일반적이다.</li>
<li>Live Server를 이용하면 웹 페이지를 실시간으로 확인할 수 있다.</li>
<li>Codex CLI를 활용하면 자연어 기반으로 코드 생성, 수정, 리뷰, 테스트를 수행할 수 있다.</li>
<li>앞으로의 개발은 AI를 활용한 <strong>AI Assisted Development</strong>가 중요한 개발 방식이 될 것이다.</li>
</ul>
<hr />
<h1 id="느낀-점">느낀 점</h1>
<p>이번 학습을 통해 Git은 단순한 명령어 모음이 아니라 <strong>프로젝트의 변경 이력을 안전하게 관리하고 협업을 가능하게 하는 핵심 도구</strong>라는 점을 이해했다.</p>
<p>또한 AI는 단순히 코드를 대신 작성하는 도구가 아니라, <strong>코드 생성부터 설명, 리뷰, 테스트까지 함께 수행하는 개발 파트너</strong>가 될 수 있다는 점이 인상적이었다. 앞으로는 Git을 활용한 버전 관리와 AI 기반 개발을 함께 익혀 생산성을 높이는 개발 습관을 만들어가고자 한다.</p>