<blockquote>
<p>처음 배우는 사람을 위한 개념 중심 학습 노트<br />핵심 흐름: <strong>변화에 대응하는 일하는 방식(Agile) → 그 방식을 운영하는 틀(Scrum) → 일을 표현하는 언어(User Story와 Backlog) → 독립적인 기술 구조(MSA) → 팀 설계 실습</strong></p>
</blockquote>
<hr />
<h2 id="0-오늘-무엇을-배우는가">0. 오늘 무엇을 배우는가</h2>
<p>오늘의 목표는 어려운 기술을 모두 구현하는 것이 아니다. 팀이 <strong>무엇을, 왜, 어떤 순서로 만들지</strong> 정하고, 그 계획을 내일의 MSA 구현으로 연결하는 것이 핵심이다.</p>
<table>
<thead>
<tr>
<th>파트</th>
<th>마지막에 답할 수 있어야 하는 질문</th>
<th>핵심 산출물</th>
</tr>
</thead>
<tbody><tr>
<td>1. Agile 개요</td>
<td>왜 한 번에 완성하지 않고 작게 나누어 만드는가?</td>
<td>Agile 전환 체크리스트</td>
</tr>
<tr>
<td>2. Scrum 핵심 요소</td>
<td>누가, 어떤 회의와 산출물로 Sprint를 운영하는가?</td>
<td>Sprint 캘린더, Daily 진행 규칙</td>
</tr>
<tr>
<td>3. User Story와 Backlog</td>
<td>사용자의 가치를 개발 가능한 일로 어떻게 바꾸는가?</td>
<td>Product Backlog 초안</td>
</tr>
<tr>
<td>4. 모의 프로젝트 소개</td>
<td>각 MSA 컴포넌트는 무엇을 하고 어떻게 통신하는가?</td>
<td>도메인·통신 흐름 이해</td>
</tr>
<tr>
<td>5. 모의 프로젝트 가이드</td>
<td>팀은 어떤 역할로 어떤 Story를 준비하는가?</td>
<td>역할표, 샘플 Story, 체크리스트</td>
</tr>
<tr>
<td>6. 모의 프로젝트 설계 실습</td>
<td>Story를 실제 수행 가능한 Task와 Board로 어떻게 바꾸는가?</td>
<td>Sprint Goal, Sprint Backlog, Board</td>
</tr>
</tbody></table>
<h3 id="먼저-기억할-한-문장">먼저 기억할 한 문장</h3>
<blockquote>
<p><strong>Agile은 변화에 대응하며 일하는 방식이고, Scrum은 그 방식을 반복 실행하는 틀이며, MSA는 빠르고 잦은 변경을 기술적으로 돕는 구조다.</strong></p>
</blockquote>
<h3 id="전체-흐름-지도">전체 흐름 지도</h3>
<pre><code class="language-text">고객의 문제와 가치
      ↓
Epic → User Story → Product Backlog
                         ↓ Sprint Planning
                Sprint Goal + Sprint Backlog
                         ↓
              Task 수행 + Daily Scrum + Board
                         ↓
          동작하는 Increment + Sprint Review
                         ↓
             Retrospective에서 방식 개선
                         ↓
             피드백을 다음 Sprint에 반영</code></pre>
<hr />
<h1 id="part-1-agile-개요">Part 1. Agile 개요</h1>
<h2 id="11-agile이란-무엇인가">1.1 Agile이란 무엇인가</h2>
<p>Agile은 특정 도구나 회의 이름이 아니다. 요구사항이 바뀔 수 있다는 사실을 인정하고, 제품을 작은 단위로 자주 완성하여 피드백을 받는 <strong>사고방식과 개발 접근법</strong>이다.</p>
<ul>
<li>한 번에 모든 기능을 완성하려 하지 않는다.</li>
<li>가치가 큰 기능부터 작은 단위로 만든다.</li>
<li>짧은 주기로 동작하는 결과물을 확인한다.</li>
<li>고객과 이해관계자의 피드백으로 다음 계획을 조정한다.</li>
<li>계획을 바꾸는 것을 실패가 아니라 학습의 결과로 본다.</li>
</ul>
<p>쉽게 비유하면 다음과 같다.</p>
<blockquote>
<p>처음 가는 도시에서 목적지까지 이동한다고 하자. 출발 전에 모든 신호와 교통 상황을 완벽히 예측하는 것은 불가능하다. Agile은 전체 방향은 정하되, 몇 블록마다 현재 위치와 교통 상황을 확인하고 경로를 수정하는 방식이다.</p>
</blockquote>
<h2 id="12-왜-필요한가">1.2 왜 필요한가</h2>
<p>소프트웨어 개발에는 네 가지 현실이 있다.</p>
<ol>
<li>시장과 고객의 요구는 계속 바뀐다.</li>
<li>개발 전에 요구사항을 100% 정확히 확정하기 어렵다.</li>
<li>실제로 동작하는 제품을 봐야 고객도 자신이 원하는 것을 더 정확히 말할 수 있다.</li>
<li>문제를 늦게 발견할수록 수정 비용이 커진다.</li>
</ol>
<p>Waterfall 방식에서 분석과 설계를 모두 끝낸 뒤 개발·테스트를 순서대로 진행하면, 후반에 잘못된 가정을 발견했을 때 앞 단계까지 다시 고쳐야 한다. 반면 Agile은 짧은 주기마다 동작하는 결과물을 검증하여 <strong>잘못된 방향으로 오래 달리는 위험</strong>을 줄인다.</p>
<p>예를 들어 온라인 교육 서비스를 만든다고 하자.</p>
<ul>
<li>처음 생각: 결제 기능이 가장 중요하다.</li>
<li>Sprint 1 시연 결과: 사용자는 결제보다 강의 검색이 불편하다고 말한다.</li>
<li>Agile 팀의 대응: Sprint 2에서 결제 고도화보다 검색 개선을 먼저 한다.</li>
</ul>
<p>처음 계획을 그대로 지키는 것보다 <strong>더 가치 있는 방향으로 계획을 바꾸는 것</strong>이 중요하다.</p>
<h2 id="13-waterfall과-agile-비교">1.3 Waterfall과 Agile 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Waterfall</th>
<th>Agile</th>
</tr>
</thead>
<tbody><tr>
<td>진행 방식</td>
<td>분석 → 설계 → 개발 → 테스트의 순차 진행</td>
<td>짧은 Sprint를 반복하는 점진적 진행</td>
</tr>
<tr>
<td>요구사항</td>
<td>초기에 상세히 확정하고 변경을 최소화</td>
<td>계속 수집하고 Backlog 우선순위를 조정</td>
</tr>
<tr>
<td>고객 피드백</td>
<td>프로젝트 후반 또는 종료 시점</td>
<td>매 Sprint Review에서 확인</td>
</tr>
<tr>
<td>결과물 확인</td>
<td>마지막에 통합 결과물 확인</td>
<td>Sprint마다 동작하는 Increment 확인</td>
</tr>
<tr>
<td>변경 대응</td>
<td>변경 절차가 무겁고 비용이 큼</td>
<td>변경을 전제로 다음 Sprint를 재계획</td>
</tr>
<tr>
<td>팀 협업</td>
<td>역할별 분업과 문서 전달 중심</td>
<td>교차 기능 팀의 상시 협업 중심</td>
</tr>
<tr>
<td>주요 위험</td>
<td>늦은 검증으로 잘못된 방향을 늦게 발견</td>
<td>잦은 소통과 우선순위 관리가 필요</td>
</tr>
</tbody></table>
<h3 id="오해하지-말아야-할-점">오해하지 말아야 할 점</h3>
<ul>
<li>Agile은 계획을 세우지 않는 방식이 아니다. <strong>짧게 계획하고 자주 갱신</strong>한다.</li>
<li>문서를 쓰지 않는 방식도 아니다. 의사결정과 협업에 필요한 문서는 남기되, 문서 작성 자체가 목표가 되지 않게 한다.</li>
<li>빠르게 코딩만 하는 방식이 아니다. 피드백, 품질 기준, 회고까지 포함한다.</li>
<li>Waterfall이 항상 틀린 것도 아니다. 요구사항이 안정적이고 변경 비용보다 예측 가능성이 중요한 상황에서는 적합할 수 있다.</li>
</ul>
<h2 id="14-agile이-잘-맞는-상황과-덜-맞는-상황">1.4 Agile이 잘 맞는 상황과 덜 맞는 상황</h2>
<h3 id="잘-맞는-상황">잘 맞는 상황</h3>
<table>
<thead>
<tr>
<th>상황</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>신규 서비스, 스타트업, AI 서비스</td>
<td>고객 반응·모델 성능·데이터에 따라 요구가 자주 변함</td>
</tr>
<tr>
<td>빠른 시장 검증이 필요한 제품</td>
<td>작은 MVP를 먼저 내고 실제 반응을 확인할 수 있음</td>
</tr>
<tr>
<td>기능을 작은 가치 단위로 나눌 수 있음</td>
<td>Sprint마다 사용 가능한 Increment를 만들기 쉬움</td>
</tr>
<tr>
<td>PO와 개발팀이 자주 소통할 수 있음</td>
<td>짧은 피드백 루프가 실제로 작동함</td>
</tr>
<tr>
<td>교차 기능을 가진 소규모 팀</td>
<td>분석·개발·테스트가 한 팀 안에서 빠르게 연결됨</td>
</tr>
</tbody></table>
<h3 id="신중해야-하는-상황">신중해야 하는 상황</h3>
<ul>
<li>요구사항과 승인 절차가 법·계약으로 매우 엄격하게 고정된 경우</li>
<li>중간 결과물을 독립적으로 검증하기 어려운 경우</li>
<li>고객 또는 PO가 Sprint 중간 피드백에 참여할 수 없는 경우</li>
<li>팀에 우선순위를 결정할 권한이 전혀 없는 경우</li>
</ul>
<p>이런 상황에서도 Agile 요소를 일부 적용할 수 있지만, 형식만 도입하면 효과가 작다.</p>
<h2 id="15-agile-도입-5단계-로드맵">1.5 Agile 도입 5단계 로드맵</h2>
<h3 id="1단계---현황-진단">1단계 - 현황 진단</h3>
<p>현재 프로세스와 조직 문화를 살피고 병목을 찾는다.</p>
<ul>
<li>요구사항 승인에 얼마나 걸리는가?</li>
<li>테스트가 개발 마지막에 몰리는가?</li>
<li>의사결정자가 자주 부재하는가?</li>
<li>진행 상태를 팀원이 모두 볼 수 있는가?</li>
</ul>
<h3 id="2단계---파일럿-팀-선정">2단계 - 파일럿 팀 선정</h3>
<p>자발적 참여 의지가 있는 소규모 팀 1~2개부터 시작한다. 처음부터 조직 전체를 바꾸려 하면 학습 비용과 저항이 커진다.</p>
<h3 id="3단계---sprint-0-준비">3단계 - Sprint 0 준비</h3>
<p>Sprint 0는 실제 기능 개발 전에 실행 기반을 준비하는 단계다.</p>
<ul>
<li>Product Backlog 초안 작성</li>
<li>PO, Scrum Master, Development Team 역할 합의</li>
<li>Definition of Done 합의</li>
<li>Board와 협업 도구 준비</li>
<li>첫 Sprint Goal 후보 논의</li>
</ul>
<h3 id="4단계---첫-sprint-실행">4단계 - 첫 Sprint 실행</h3>
<p>1~2주의 짧은 Sprint로 시작한다. Planning → Daily → Review → Retrospective를 실제로 운영하고 곧바로 보정한다.</p>
<h3 id="5단계---확산과-정착">5단계 - 확산과 정착</h3>
<p>파일럿 팀의 성공과 실패를 공유하고, 검증된 방식만 다른 팀으로 점진적으로 확산한다.</p>
<h2 id="16-흔한-실패-패턴과-대응">1.6 흔한 실패 패턴과 대응</h2>
<table>
<thead>
<tr>
<th>실패 패턴</th>
<th>보이는 증상</th>
<th>대응 방법</th>
</tr>
</thead>
<tbody><tr>
<td>무늬만 Agile</td>
<td>Sprint라는 말만 쓰고 실제로는 기존 방식 그대로 진행</td>
<td>이벤트와 산출물의 목적을 다시 합의</td>
</tr>
<tr>
<td>PO 겸직 병목</td>
<td>우선순위와 인수 여부 결정이 계속 지연</td>
<td>PO 전담 또는 의사결정 권한 일부 위임</td>
</tr>
<tr>
<td>회고 없는 반복</td>
<td>매 Sprint 같은 문제가 반복</td>
<td>Retro 액션을 다음 Sprint Backlog에 반영</td>
</tr>
<tr>
<td>과도한 문서화</td>
<td>기존 문서 절차와 Agile 문서를 모두 작성</td>
<td>협업과 추적에 필요한 최소 산출물만 유지</td>
</tr>
<tr>
<td>진행 상황 미가시화</td>
<td>누가 무엇을 하는지 알 수 없음</td>
<td>Board와 Burndown을 매일 갱신</td>
</tr>
<tr>
<td>미완료 작업의 Done 처리</td>
<td>코드는 작성했지만 테스트·통합되지 않음</td>
<td>공통 DoD를 적용하고 미충족 항목은 완료로 보지 않음</td>
</tr>
<tr>
<td>한 사람이 모든 일을 담당</td>
<td>팀 학습과 병렬 작업이 멈춤</td>
<td>API 계약과 Task를 기준으로 역할 분담</td>
</tr>
</tbody></table>
<h3 id="실패의-본질">실패의 본질</h3>
<p>Agile의 이름만 가져오고 <strong>권한, 피드백, 가시성, 개선</strong>이 없으면 회의만 늘어난다. 도구보다 팀 문화와 리더의 지원이 중요하다.</p>
<h2 id="17-우리-팀-agile-전환-체크리스트">1.7 우리 팀 Agile 전환 체크리스트</h2>
<h3 id="sprint-0-이전">Sprint 0 이전</h3>
<ul>
<li><input disabled="" type="checkbox" /> PO, Scrum Master, Development Team의 책임을 명확히 정했다.</li>
<li><input disabled="" type="checkbox" /> Definition of Done 초안을 팀과 합의했다.</li>
<li><input disabled="" type="checkbox" /> Product Backlog에 최소 10~15개의 후보 항목이 있다.</li>
</ul>
<h3 id="sprint-시작-전">Sprint 시작 전</h3>
<ul>
<li><input disabled="" type="checkbox" /> Sprint Goal을 한 문장으로 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 선택한 Story가 작고 검증 가능하다.</li>
<li><input disabled="" type="checkbox" /> Story를 Task 단위로 분할했다.</li>
<li><input disabled="" type="checkbox" /> Sprint Board를 준비했다.</li>
<li><input disabled="" type="checkbox" /> 팀의 가용량을 넘는 일을 억지로 넣지 않았다.</li>
</ul>
<h3 id="첫-sprint-종료-후">첫 Sprint 종료 후</h3>
<ul>
<li><input disabled="" type="checkbox" /> 실제 동작하는 결과물을 Review에서 시연했다.</li>
<li><input disabled="" type="checkbox" /> Retrospective에서 개선 행동을 정했다.</li>
<li><input disabled="" type="checkbox" /> 팀 처리량(Velocity) 기록을 시작했다.</li>
<li><input disabled="" type="checkbox" /> 개선 행동을 다음 Sprint 계획에 반영했다.</li>
</ul>
<hr />
<h1 id="part-2-scrum-핵심-요소">Part 2. Scrum 핵심 요소</h1>
<h2 id="21-scrum이란-무엇인가">2.1 Scrum이란 무엇인가</h2>
<p>Scrum은 Agile의 가치를 실천하기 위한 대표적인 프레임워크다. 정해진 기간인 <strong>Sprint</strong> 안에서 가치 있는 결과물을 만들고, 검토하고, 일하는 방식을 개선하는 과정을 반복한다.</p>
<pre><code class="language-text">Product Backlog
      ↓
Sprint Planning
      ↓
Sprint 실행 ── 매일 Daily Scrum
      ↓
완성된 Increment
      ↓
Sprint Review → 제품 피드백
      ↓
Retrospective → 일하는 방식 개선
      ↓
다음 Sprint</code></pre>
<h2 id="22-scrum의-역할">2.2 Scrum의 역할</h2>
<p>교재에서는 이해를 위해 역할(Role)이라는 표현을 사용한다.</p>
<table>
<thead>
<tr>
<th>역할</th>
<th>핵심 책임</th>
<th>하면 안 되는 일</th>
</tr>
</thead>
<tbody><tr>
<td>Product Owner(PO)</td>
<td>제품 가치 극대화, Product Backlog 관리, 우선순위·인수 기준 결정</td>
<td>모든 구현 방법을 일방적으로 지시</td>
</tr>
<tr>
<td>Scrum Master(SM)</td>
<td>Scrum 진행 촉진, 장애물 정리, 팀 보호, Timebox 관리</td>
<td>팀의 상사처럼 Task를 강제 배정</td>
</tr>
<tr>
<td>Development Team</td>
<td>설계·구현·테스트를 통해 Done Increment 완성</td>
<td>자기 영역만 끝내고 통합을 외면</td>
</tr>
</tbody></table>
<h3 id="역할을-쉽게-비유하면">역할을 쉽게 비유하면</h3>
<ul>
<li><strong>PO는 목적지와 우선순위를 정하는 사람</strong>이다.</li>
<li><strong>SM은 팀이 규칙에 맞게 잘 달리도록 길을 정리하는 사람</strong>이다.</li>
<li><strong>개발팀은 목적지에 도달할 방법을 함께 결정하고 제품을 만드는 사람들</strong>이다.</li>
</ul>
<p>실습에서는 역할을 고정할 필요가 없다. 다음 Sprint에서 바꾸어 경험해도 된다. 다만 한 Sprint 안에서는 최종 의사결정자가 누구인지 모호하지 않아야 한다.</p>
<h2 id="23-scrum-산출물">2.3 Scrum 산출물</h2>
<h3 id="product-backlog">Product Backlog</h3>
<p>제품에 필요할 수 있는 모든 요구사항의 우선순위 목록이다. 기능뿐 아니라 오류 수정, 기술 개선, 조사 항목도 포함할 수 있다. PO가 관리하지만 팀과 함께 계속 정제한다.</p>
<h3 id="sprint-backlog">Sprint Backlog</h3>
<p>이번 Sprint에서 달성하기로 선택한 Product Backlog Item과 이를 구현하기 위한 Task 계획이다. 단순한 할 일 목록이 아니라 <strong>Sprint Goal 달성을 위한 실행 계획</strong>이다.</p>
<h3 id="increment">Increment</h3>
<p>이번 Sprint까지 Done 기준을 만족한 모든 결과의 합이다. 코드가 작성되었더라도 통합·테스트 기준을 만족하지 못하면 Increment에 포함할 수 없다.</p>
<h3 id="definition-of-donedod">Definition of Done(DoD)</h3>
<p>팀이 결과물을 “완료”라고 부르기 위한 공통 품질 기준이다.</p>
<p>예시:</p>
<ul>
<li><input disabled="" type="checkbox" /> 코드 작성 완료</li>
<li><input disabled="" type="checkbox" /> 단위 테스트 통과</li>
<li><input disabled="" type="checkbox" /> 팀원 리뷰 완료</li>
<li><input disabled="" type="checkbox" /> 기존 기능 회귀 테스트 통과</li>
<li><input disabled="" type="checkbox" /> API 명세 갱신</li>
<li><input disabled="" type="checkbox" /> 공통 개발 환경에서 실행 확인</li>
</ul>
<h3 id="acceptance-criteria와-dod의-차이">Acceptance Criteria와 DoD의 차이</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>Acceptance Criteria(인수 기준)</th>
<th>Definition of Done(완료의 정의)</th>
</tr>
</thead>
<tbody><tr>
<td>적용 대상</td>
<td>특정 User Story</td>
<td>모든 Story/Increment에 공통</td>
</tr>
<tr>
<td>질문</td>
<td>이 기능이 요구를 만족했는가?</td>
<td>팀의 품질 기준까지 모두 만족했는가?</td>
</tr>
<tr>
<td>예시</td>
<td>잘못된 로그인 정보면 오류 메시지를 표시한다</td>
<td>테스트 통과, 리뷰 완료, 명세 갱신</td>
</tr>
</tbody></table>
<h2 id="24-scrum-이벤트-실전-가이드">2.4 Scrum 이벤트 실전 가이드</h2>
<table>
<thead>
<tr>
<th>이벤트</th>
<th>목적</th>
<th>참석</th>
<th>핵심 진행</th>
</tr>
</thead>
<tbody><tr>
<td>Sprint Planning</td>
<td>이번 Sprint의 목표와 일을 결정</td>
<td>전체 Scrum 팀</td>
<td>Goal 합의 → Story 선택 → Task 분할</td>
</tr>
<tr>
<td>Daily Scrum</td>
<td>Sprint Goal을 향한 진행을 점검·조정</td>
<td>개발팀 중심, SM 촉진</td>
<td>어제/오늘/장애물 공유</td>
</tr>
<tr>
<td>Sprint Review</td>
<td>결과물로 이해관계자 피드백 수집</td>
<td>팀 + 이해관계자</td>
<td>동작 데모, 피드백, Backlog 조정</td>
</tr>
<tr>
<td>Retrospective</td>
<td>팀의 일하는 방식 개선</td>
<td>Scrum 팀</td>
<td>Keep/Problem/Try, 실행 항목 선정</td>
</tr>
<tr>
<td>Backlog Refinement</td>
<td>다음 Sprint 후보를 준비</td>
<td>PO + 필요한 개발팀원</td>
<td>상세화, 분할, 우선순위, 추정</td>
</tr>
</tbody></table>
<p>모든 이벤트에는 Timebox가 있다. 논의가 길어지면 별도 회의로 분리한다. 회의 시간을 지키는 이유는 대화를 막기 위해서가 아니라, 목적이 다른 논의를 섞지 않기 위해서다.</p>
<h2 id="25-sprint-0와-release-planning">2.5 Sprint 0와 Release Planning</h2>
<p>현실적인 프로젝트에서는 본격적인 Sprint 전에 Sprint 0를 두기도 한다. 여기서 요구사항을 User Story로 바꾸고, Product Backlog와 개발 환경을 준비한다.</p>
<p>Release Planning은 “어떤 가치를 어떤 출시 묶음과 Sprint에 배치할 것인가”를 결정한다.</p>
<pre><code class="language-text">Product Backlog 우선순위
      ↓
Release 1: 반드시 먼저 제공할 가치
  ├─ Sprint 1
  ├─ Sprint 2
  └─ Sprint 3
      ↓
Release 2: 이후 확장할 가치
  ├─ Sprint 4
  └─ Sprint 5</code></pre>
<p>Sprint 주기는 보통 고정한다. 교재의 일반 예시는 2~4주지만, 본 실습에서는 학습 목적에 맞게 압축된 Sprint를 사용한다. 기간보다 중요한 것은 <strong>각 Sprint가 검증 가능한 결과를 내는 것</strong>이다.</p>
<h2 id="26-2주-sprint-캘린더-예시">2.6 2주 Sprint 캘린더 예시</h2>
<table>
<thead>
<tr>
<th>시점</th>
<th>활동</th>
<th>확인할 것</th>
</tr>
</thead>
<tbody><tr>
<td>Day 1</td>
<td>Sprint Planning</td>
<td>Goal, 선택 Story, Task, 담당 합의</td>
</tr>
<tr>
<td>Day 2~4</td>
<td>개발 + Daily Scrum</td>
<td>초기 장애물을 빠르게 발견</td>
</tr>
<tr>
<td>Day 5~8</td>
<td>개발 + Daily + 필요 시 Refinement</td>
<td>다음 후보 Story도 준비</td>
</tr>
<tr>
<td>Day 9</td>
<td>개발 마무리 + Daily</td>
<td>DoD 미충족 항목 정리</td>
</tr>
<tr>
<td>Day 10</td>
<td>Review + Retrospective</td>
<td>제품 피드백과 프로세스 개선을 분리</td>
</tr>
</tbody></table>
<h3 id="하루-운영-예시">하루 운영 예시</h3>
<pre><code class="language-text">09:30~09:45  Daily Scrum
09:45~12:00  집중 개발
14:00~17:00  집중 개발 / 필요한 Refinement
17:00~17:15  Sprint Board 최신화</code></pre>
<h2 id="27-daily-scrum-진행법과-시나리오">2.7 Daily Scrum 진행법과 시나리오</h2>
<p>세 질문만 짧게 답한다.</p>
<ol>
<li>어제 Sprint Goal을 위해 무엇을 했는가?</li>
<li>오늘 무엇을 할 것인가?</li>
<li>목표 달성을 막는 장애물은 무엇인가?</li>
</ol>
<h3 id="예시-대화">예시 대화</h3>
<blockquote>
<p><strong>백엔드 A:</strong> 어제 로그인 API를 완료했습니다. 오늘은 토큰 저장 로직을 구현합니다. 장애물은 없습니다.<br /><strong>백엔드 B:</strong> 어제 Gateway 라우팅 설정을 진행했습니다. 오늘 마무리합니다. 서비스 이름 설정을 확인해야 합니다.<br /><strong>프론트 C:</strong> 어제 로그인 화면을 완성했습니다. 오늘 API를 연결합니다. 로그인 응답 형식을 확인해야 합니다.</p>
</blockquote>
<p>여기서 응답 형식이나 설정을 자세히 토론하지 않는다. Daily에서는 장애물을 <strong>발견하고 담당자를 연결</strong>한 뒤, 필요한 사람만 남아 별도 논의를 한다.</p>
<h3 id="나쁜-daily의-신호">나쁜 Daily의 신호</h3>
<ul>
<li>SM에게 업무 보고만 한다.</li>
<li>한 사람이 문제 해결 회의를 길게 시작한다.</li>
<li>Board와 말하는 내용이 다르다.</li>
<li>“진행 중입니다”만 반복하고 완료 조건이 없다.</li>
<li>Sprint Goal과 무관한 개인 업무를 나열한다.</li>
</ul>
<h2 id="28-review와-retrospective를-혼동하지-않기">2.8 Review와 Retrospective를 혼동하지 않기</h2>
<ul>
<li><strong>Review의 대상은 제품</strong>이다: 무엇을 만들었고 어떤 피드백을 받았는가?</li>
<li><strong>Retrospective의 대상은 팀의 방식</strong>이다: 어떻게 일했고 무엇을 개선할 것인가?</li>
</ul>
<p>Retro의 Keep/Problem/Try 예시:</p>
<table>
<thead>
<tr>
<th>Keep</th>
<th>Problem</th>
<th>Try</th>
</tr>
</thead>
<tbody><tr>
<td>API 계약을 먼저 합의해 병렬 작업이 가능했다</td>
<td>JWT 설정 문제를 너무 늦게 공유했다</td>
<td>다음 Sprint부터 막힘이 30분 넘으면 즉시 Board에 표시한다</td>
</tr>
</tbody></table>
<p>Try는 “소통을 잘하자”처럼 추상적으로 쓰지 않는다. <strong>누가, 언제, 무엇을 할지 확인 가능한 행동</strong>으로 정한다.</p>
<hr />
<h1 id="part-3-user-story와-backlog">Part 3. User Story와 Backlog</h1>
<h2 id="31-요구사항이-구현-가능한-일로-바뀌는-과정">3.1 요구사항이 구현 가능한 일로 바뀌는 과정</h2>
<pre><code class="language-text">비즈니스 요구
  ↓
Epic: 큰 기능 묶음
  ↓
User Story: 한 사용자가 얻는 작은 가치
  ↓
Task: 개발자가 실제로 수행할 작업</code></pre>
<p>예를 들어 “온라인 교육 플랫폼 구축”은 비즈니스 요구다.</p>
<ul>
<li>Epic: 사용자 인증</li>
<li>User Story: 수강생이 로그인하여 자신의 수강 목록에 접근한다.</li>
<li>Task: 로그인 화면 작성, API 연동, 토큰 저장, 오류 처리, 테스트 작성</li>
</ul>
<p>Product Backlog는 주로 Epic과 User Story 수준의 항목으로 구성하고, Sprint에 선택된 Story를 Task로 나누어 Sprint Backlog를 만든다.</p>
<h2 id="32-user-story-작성-형식">3.2 User Story 작성 형식</h2>
<pre><code class="language-text">As a [사용자 유형],
I want [원하는 기능 또는 행동],
So that [얻고자 하는 가치 또는 이유].</code></pre>
<p>한국어로 읽으면 다음과 같다.</p>
<pre><code class="language-text">[누구]는
[어떤 가치]를 얻기 위해
[어떤 행동/기능]을 원한다.</code></pre>
<p>좋은 Story는 기능명이 아니라 <strong>사용자와 가치가 포함된 대화의 출발점</strong>이다.</p>
<h2 id="33-invest-원칙">3.3 INVEST 원칙</h2>
<table>
<thead>
<tr>
<th>글자</th>
<th>의미</th>
<th>확인 질문</th>
</tr>
</thead>
<tbody><tr>
<td>I - Independent</td>
<td>다른 Story와 가능한 한 독립적</td>
<td>다른 Story가 끝나지 않아도 만들고 검증할 수 있는가?</td>
</tr>
<tr>
<td>N - Negotiable</td>
<td>구현 방법을 협의할 수 있음</td>
<td>상세 설계를 미리 고정한 계약서가 되어 있지 않은가?</td>
</tr>
<tr>
<td>V - Valuable</td>
<td>사용자 또는 고객 가치가 있음</td>
<td>완료되면 누가 어떤 이익을 얻는가?</td>
</tr>
<tr>
<td>E - Estimable</td>
<td>팀이 규모를 추정할 수 있음</td>
<td>모호하거나 미지의 영역이 너무 크지 않은가?</td>
</tr>
<tr>
<td>S - Small</td>
<td>한 Sprint에 완료 가능한 크기</td>
<td>너무 많은 흐름을 한 Story에 넣지 않았는가?</td>
</tr>
<tr>
<td>T - Testable</td>
<td>완료 여부를 검증할 수 있음</td>
<td>명확한 인수 기준을 작성할 수 있는가?</td>
</tr>
</tbody></table>
<h3 id="모든-내용을-처음부터-상세히-쓰지-않는-이유">모든 내용을 처음부터 상세히 쓰지 않는 이유</h3>
<p>우선순위가 낮은 Story까지 미리 상세화하면, 실제 개발 전에 요구가 바뀌어 문서가 낭비될 수 있다.</p>
<ul>
<li>최초 등록: 제목과 한 줄 가치</li>
<li>Refinement: 상위 후보의 조건과 인수 기준 논의</li>
<li>Planning 직전: INVEST를 만족하고 Task로 나눌 정도로 상세화</li>
<li>Sprint 진행: 필요한 세부사항을 대화로 보완</li>
</ul>
<p>이를 <strong>Just-in-time 상세화</strong>라고 이해하면 된다.</p>
<h2 id="34-backlog-refinement그루밍-프로세스">3.4 Backlog Refinement(그루밍) 프로세스</h2>
<p>Backlog Grooming과 Refinement는 같은 맥락으로 쓰인다. 다음 1~2개 Sprint에 들어올 Story를 Ready 상태로 준비하는 활동이다.</p>
<ol>
<li><strong>우선순위 재정렬</strong>: PO가 비즈니스 가치, 위험, 의존성을 보고 순서를 조정한다.</li>
<li><strong>상위 항목 상세화</strong>: 사용자, 조건, 예외, 인수 기준을 논의한다.</li>
<li><strong>큰 Story 분할</strong>: 한 Sprint 안에 끝내기 어려운 Story를 작은 가치 단위로 나눈다.</li>
<li><strong>INVEST 점검</strong>: 독립성·가치·크기·검증 가능성을 확인한다.</li>
<li><strong>Story Point 추정</strong>: 팀이 상대적인 규모를 함께 추정한다.</li>
</ol>
<h3 id="definition-of-ready-예시">Definition of Ready 예시</h3>
<p>교재가 강조하는 “Ready”를 팀 기준으로 바꾸면 다음과 같다.</p>
<ul>
<li><input disabled="" type="checkbox" /> 사용자와 가치가 명확하다.</li>
<li><input disabled="" type="checkbox" /> 인수 기준이 있다.</li>
<li><input disabled="" type="checkbox" /> 주요 의존성과 위험을 알고 있다.</li>
<li><input disabled="" type="checkbox" /> 한 Sprint에 끝낼 수 있는 크기다.</li>
<li><input disabled="" type="checkbox" /> 팀이 Point를 추정할 수 있다.</li>
</ul>
<h2 id="35-before--after">3.5 Before / After</h2>
<h3 id="before---모호한-요구">Before - 모호한 요구</h3>
<blockquote>
<p>로그인 기능 개발</p>
</blockquote>
<p>문제점:</p>
<ul>
<li>누가 사용하는지 모른다.</li>
<li>왜 필요한지 가치가 없다.</li>
<li>성공과 실패의 기준이 없다.</li>
<li>팀원마다 완성을 다르게 해석한다.</li>
</ul>
<h3 id="after---가치와-검증-기준이-있는-story">After - 가치와 검증 기준이 있는 Story</h3>
<blockquote>
<p><strong>As a</strong> 회원, <strong>I want</strong> 이메일과 비밀번호로 로그인하고 싶다, <strong>So that</strong> 내 계정에 안전하게 접근할 수 있다.</p>
</blockquote>
<p>Acceptance Criteria:</p>
<pre><code class="language-gherkin">Given 가입된 회원이 로그인 화면에 있고
When 올바른 이메일과 비밀번호를 입력하면
Then Access Token이 발급되고 메인 화면으로 이동한다.

Given 가입된 회원이 로그인 화면에 있고
When 잘못된 비밀번호를 입력하면
Then 로그인에 실패하고 이해 가능한 오류 메시지를 표시한다.

Given 동일 계정으로 인증에 연속 실패했고
When 실패 횟수가 팀이 정한 기준에 도달하면
Then 계정을 일시 잠그고 사용자에게 안내한다.</code></pre>
<p><code>Given-When-Then</code>은 다음 뜻이다.</p>
<ul>
<li>Given: 어떤 상황에서</li>
<li>When: 어떤 행동을 하면</li>
<li>Then: 어떤 결과가 보여야 하는가</li>
</ul>
<h2 id="36-product-backlog-채우기-가이드">3.6 Product Backlog 채우기 가이드</h2>
<h3 id="1단계---이해관계자와-pain-point-정의">1단계 - 이해관계자와 Pain Point 정의</h3>
<p>“어떤 기능을 만들까?”보다 먼저 “누가 무엇 때문에 불편한가?”를 묻는다.</p>
<p>예시: 수강생은 자신의 목적에 맞는 강의를 찾기 어렵다.</p>
<h3 id="2단계---epic-브레인스토밍">2단계 - Epic 브레인스토밍</h3>
<p>큰 기능 묶음을 나열한다.</p>
<ul>
<li>회원·인증</li>
<li>강의 등록·조회</li>
<li>수강신청</li>
<li>결제</li>
<li>추천</li>
<li>운영·모니터링</li>
</ul>
<h3 id="3단계---epic을-user-story로-분해">3단계 - Epic을 User Story로 분해</h3>
<p>한 사용자가 한 번에 얻는 가치 단위로 줄인다.</p>
<h3 id="4단계---moscow-우선순위-지정">4단계 - MoSCoW 우선순위 지정</h3>
<table>
<thead>
<tr>
<th>등급</th>
<th>뜻</th>
<th>판단 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Must</td>
<td>이번 목표에 반드시 필요</td>
<td>이것이 없으면 핵심 시나리오를 시연할 수 없는가?</td>
</tr>
<tr>
<td>Should</td>
<td>중요하지만 우회 가능</td>
<td>없으면 불편하지만 핵심 가치는 전달되는가?</td>
</tr>
<tr>
<td>Could</td>
<td>여유가 있을 때</td>
<td>있으면 좋지만 미뤄도 되는가?</td>
</tr>
<tr>
<td>Won't</td>
<td>이번 범위에서 하지 않음</td>
<td>명시적으로 제외해 집중력을 지킬 수 있는가?</td>
</tr>
</tbody></table>
<h3 id="5단계---상위-항목의-invest와-인수-기준-점검">5단계 - 상위 항목의 INVEST와 인수 기준 점검</h3>
<p>전체 Backlog를 모두 상세화하지 말고, 우선순위 상위 약 10개부터 준비한다.</p>
<h3 id="6단계---planning-poker로-point-추정">6단계 - Planning Poker로 Point 추정</h3>
<p>팀원이 각자 1, 2, 3, 5, 8 같은 값을 고른 뒤 동시에 공개한다. 가장 큰 값과 작은 값을 고른 사람이 판단 근거를 설명하고 다시 합의한다. 목적은 정답 맞히기가 아니라 <strong>일의 복잡성과 위험에 대한 공통 이해</strong>를 만드는 것이다.</p>
<h2 id="37-story-point를-시간과-혼동하지-않기">3.7 Story Point를 시간과 혼동하지 않기</h2>
<p>Story Point는 “3점 = 3시간”이 아니다. 상대적 크기이며 다음 요소를 함께 반영한다.</p>
<ul>
<li>작업량</li>
<li>기술적 복잡도</li>
<li>불확실성</li>
<li>외부 의존성</li>
</ul>
<p>기준 Story를 하나 정하면 쉽다.</p>
<blockquote>
<p>“정적 강의 목록 화면 만들기”를 2점으로 두었을 때, 인증과 오류 처리가 포함된 로그인 연동은 약 두 배 이상 복잡하므로 5점으로 본다.</p>
</blockquote>
<p>Point는 사람의 능력을 평가하는 점수가 아니다. 다른 팀의 Point와 직접 비교해서도 안 된다.</p>
<h2 id="38-product-backlog-예시">3.8 Product Backlog 예시</h2>
<table>
<thead>
<tr>
<th align="right">우선순위</th>
<th>User Story</th>
<th>인수 기준 요약</th>
<th>MoSCoW</th>
<th align="right">Point</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1</td>
<td>수강생은 계정 접근을 위해 로그인하고 싶다</td>
<td>정상 토큰 발급, 실패 메시지</td>
<td>Must</td>
<td align="right">5</td>
</tr>
<tr>
<td align="right">2</td>
<td>수강생은 선택을 위해 강의 목록을 보고 싶다</td>
<td>목록·가격·카테고리 표시</td>
<td>Must</td>
<td align="right">3</td>
</tr>
<tr>
<td align="right">3</td>
<td>수강생은 학습을 위해 강의를 신청하고 싶다</td>
<td>PENDING 신청 생성</td>
<td>Must</td>
<td align="right">5</td>
</tr>
<tr>
<td align="right">4</td>
<td>강사는 판매를 위해 강의를 등록하고 싶다</td>
<td>강사 권한 검증, 등록 결과 표시</td>
<td>Should</td>
<td align="right">5</td>
</tr>
<tr>
<td align="right">5</td>
<td>수강생은 결제 후 바로 수강하고 싶다</td>
<td>결제 완료 후 ACTIVE 전환</td>
<td>Should</td>
<td align="right">8</td>
</tr>
<tr>
<td align="right">6</td>
<td>수강생은 맞춤 탐색을 위해 추천을 받고 싶다</td>
<td>수강 이력 기반 추천 반환</td>
<td>Could</td>
<td align="right">8</td>
</tr>
</tbody></table>
<hr />
<h1 id="part-4-모의-프로젝트-소개">Part 4. 모의 프로젝트 소개</h1>
<h2 id="41-프로젝트-개요">4.1 프로젝트 개요</h2>
<p>모의 프로젝트는 <strong>온라인 교육 플랫폼</strong>이다.</p>
<table>
<thead>
<tr>
<th>주체</th>
<th>주요 기능</th>
</tr>
</thead>
<tbody><tr>
<td>수강생</td>
<td>회원가입, 로그인, 강의 검색·조회, 수강신청, 결제, 추천 조회</td>
</tr>
<tr>
<td>강사</td>
<td>회원가입, 로그인, 강의 등록</td>
</tr>
<tr>
<td>시스템</td>
<td>결제 완료 후 수강 활성화, 수강 완료 후 추천 갱신</td>
</tr>
</tbody></table>
<p>오늘은 이 시스템을 처음부터 모두 코딩하지 않는다. 제공된 MSA 템플릿을 이해하고, 사용자 가치와 API 계약을 기준으로 팀 아이디어에 맞게 계획하는 것이 중요하다.</p>
<h2 id="42-왜-agile과-msa를-함께-배우는가">4.2 왜 Agile과 MSA를 함께 배우는가</h2>
<ul>
<li>Agile은 기능을 작게 나누어 자주 변경·배포하려 한다.</li>
<li>모놀리식 시스템은 작은 변경에도 전체 프로그램을 다시 테스트·배포해야 할 수 있다.</li>
<li>MSA는 기능을 독립 서비스로 나누어 변경 범위를 줄인다.</li>
</ul>
<p>따라서 MSA는 Agile의 빠르고 잦은 변경을 기술적으로 돕는다. 하지만 MSA가 항상 정답은 아니다.</p>
<table>
<thead>
<tr>
<th>구조</th>
<th>적합한 상황</th>
<th>장점</th>
<th>비용</th>
</tr>
</thead>
<tbody><tr>
<td>Monolith</td>
<td>작은 팀, 단순한 초기 제품</td>
<td>빠른 시작, 낮은 운영 복잡도</td>
<td>규모가 커지면 결합도 증가</td>
</tr>
<tr>
<td>Modular Monolith</td>
<td>하나로 배포하되 내부 경계가 필요한 경우</td>
<td>비교적 단순한 운영 + 모듈 경계</td>
<td>독립 배포에는 한계</td>
</tr>
<tr>
<td>MSA</td>
<td>여러 팀, 서비스별 독립 배포·확장이 중요한 경우</td>
<td>독립 변경·배포·확장</td>
<td>통신, 데이터 일관성, 관측, 운영 복잡도 증가</td>
</tr>
</tbody></table>
<p>“얼마나 잘게 나눌까?”는 기술 유행이 아니라 <strong>도메인 경계, 데이터 소유권, 변경 빈도, 팀 구조</strong>로 판단한다.</p>
<h2 id="43-도메인-서비스와-포트">4.3 도메인 서비스와 포트</h2>
<table>
<thead>
<tr>
<th>서비스</th>
<th>기술</th>
<th align="right">포트</th>
<th>책임</th>
<th>핵심 데이터</th>
</tr>
</thead>
<tbody><tr>
<td>User Service</td>
<td>Spring Boot</td>
<td align="right">8081</td>
<td>회원가입, 사용자 조회</td>
<td>users</td>
</tr>
<tr>
<td>Course Service</td>
<td>Spring Boot</td>
<td align="right">8082</td>
<td>강의 등록, 목록, 검색</td>
<td>courses</td>
</tr>
<tr>
<td>Enrollment Service</td>
<td>Spring Boot</td>
<td align="right">8083</td>
<td>수강신청, 상태 관리</td>
<td>enrollments</td>
</tr>
<tr>
<td>Payment Service</td>
<td>Spring Boot</td>
<td align="right">8084</td>
<td>결제 처리, 내역 조회</td>
<td>payments</td>
</tr>
<tr>
<td>Recommend Service</td>
<td>FastAPI</td>
<td align="right">8085</td>
<td>수강 이력 기반 추천</td>
<td>enrollments·courses 조회</td>
</tr>
</tbody></table>
<h3 id="중요한-경계-원칙">중요한 경계 원칙</h3>
<p>각 서비스는 자신의 책임과 데이터를 가진다. 다른 서비스의 내부 클래스나 모델을 직접 공유하기보다, 공개된 API 또는 이벤트 계약으로 통신한다. 이 원칙이 있어야 각 팀이 서로의 내부 구현을 몰라도 병렬로 작업할 수 있다.</p>
<h2 id="44-인프라-컴포넌트">4.4 인프라 컴포넌트</h2>
<table>
<thead>
<tr>
<th>컴포넌트</th>
<th align="right">포트</th>
<th>쉬운 비유</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>API Gateway</td>
<td align="right">8080</td>
<td>건물 안내 데스크</td>
<td>모든 외부 요청의 단일 진입점, 경로 라우팅, 인증 필터</td>
</tr>
<tr>
<td>Eureka Server</td>
<td align="right">8761</td>
<td>서비스 전화번호부</td>
<td>서비스가 현재 어디에 있는지 등록·조회</td>
</tr>
<tr>
<td>Auth Server</td>
<td align="right">9000</td>
<td>출입증 발급소</td>
<td>로그인·OAuth2 처리, JWT 발급, 공개키 제공</td>
</tr>
<tr>
<td>Kafka</td>
<td align="right">9092</td>
<td>사건 알림 게시판</td>
<td>Producer가 발행한 이벤트를 Consumer에게 비동기 전달</td>
</tr>
<tr>
<td>MariaDB</td>
<td align="right">3306</td>
<td>업무 기록 보관소</td>
<td>실습에서는 단일 인스턴스 안에 업무 테이블 구성</td>
</tr>
<tr>
<td>Docker Compose</td>
<td align="right">-</td>
<td>전체 무대 관리자</td>
<td>여러 컨테이너의 구성·의존성·기동을 한 파일로 관리</td>
</tr>
</tbody></table>
<h3 id="이-실습에서-이해해야-하는-범위">이 실습에서 이해해야 하는 범위</h3>
<ul>
<li>로그인하면 토큰을 받고, 이후 요청에 토큰을 넣는다.</li>
<li>클라이언트는 개별 서비스가 아니라 Gateway를 호출한다.</li>
<li>Gateway는 요청을 올바른 서비스로 보낸다.</li>
<li>Eureka는 실행 중인 서비스 위치를 찾게 해 준다.</li>
<li>결제 완료 이벤트가 발생하면 수강 상태가 자동으로 바뀐다.</li>
<li>Swagger에서 요청과 응답 계약을 확인한다.</li>
</ul>
<p>처음부터 모든 인프라 코드를 직접 구현하거나 한 줄씩 이해하는 것은 오늘의 목표가 아니다.</p>
<h2 id="45-gateway-라우팅">4.5 Gateway 라우팅</h2>
<table>
<thead>
<tr>
<th>외부 경로</th>
<th>대상 서비스</th>
</tr>
</thead>
<tbody><tr>
<td><code>/users/**</code></td>
<td>User Service</td>
</tr>
<tr>
<td><code>/courses/**</code></td>
<td>Course Service</td>
</tr>
<tr>
<td><code>/enrollments/**</code></td>
<td>Enrollment Service</td>
</tr>
<tr>
<td><code>/payments/**</code></td>
<td>Payment Service</td>
</tr>
<tr>
<td><code>/recommend/**</code></td>
<td>Recommend Service</td>
</tr>
</tbody></table>
<p>브라우저가 <code>http://localhost:8080/courses/...</code>를 호출하면 Gateway가 Course Service로 전달한다. 이 단일 진입점 덕분에 인증, 로깅, CORS 같은 공통 처리를 한곳에서 일관되게 적용할 수 있다.</p>
<h2 id="46-인증-흐름">4.6 인증 흐름</h2>
<pre><code class="language-text">1. Client → Auth Server: 로그인/인증 요청
2. Auth Server → Client: Access Token(JWT) 발급
3. Client → API Gateway: Authorization: Bearer &lt;token&gt;과 함께 API 요청
4. Gateway: 공개키(JWK)를 이용해 토큰 검증
5. Gateway → 대상 Service: 검증된 사용자 정보를 헤더와 함께 전달
6. Service: 권한과 요청을 확인하고 응답</code></pre>
<ul>
<li><strong>Authentication(인증)</strong>: “당신은 누구인가?”를 확인한다.</li>
<li><strong>Authorization(인가)</strong>: “당신이 이 행동을 할 권한이 있는가?”를 확인한다.</li>
<li><strong>JWT</strong>: 사용자의 식별 정보와 권한 등을 서명된 형태로 담는 토큰이다.</li>
<li><strong>JWK</strong>: 서비스들이 JWT 서명을 검증할 수 있도록 공개키 정보를 제공하는 형식이다.</li>
</ul>
<h2 id="47-동기-rest와-비동기-kafka">4.7 동기 REST와 비동기 Kafka</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>REST 동기 통신</th>
<th>Kafka 비동기 통신</th>
</tr>
</thead>
<tbody><tr>
<td>응답</td>
<td>호출한 쪽이 결과를 기다림</td>
<td>이벤트를 발행하고 다른 작업을 계속할 수 있음</td>
</tr>
<tr>
<td>적합한 상황</td>
<td>지금 즉시 결과가 필요</td>
<td>사건을 여러 서비스에 알리고 느슨하게 연결</td>
</tr>
<tr>
<td>예시</td>
<td>강의가 존재하는지 확인</td>
<td>결제가 완료되었다는 사건 전파</td>
</tr>
<tr>
<td>주요 위험</td>
<td>상대 서비스 지연·장애가 호출자에 영향</td>
<td>중복 처리, 순서, 재시도, 최종 일관성 고려</td>
</tr>
</tbody></table>
<p>실습의 통신 구간:</p>
<table>
<thead>
<tr>
<th>호출/이벤트</th>
<th>방식</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>Enrollment → Course</td>
<td>REST</td>
<td>신청 전에 강의 존재 여부를 즉시 알아야 함</td>
</tr>
<tr>
<td>Enrollment → Payment</td>
<td>REST</td>
<td>결제 요청 결과가 현재 흐름에 필요</td>
</tr>
<tr>
<td>Recommend → Enrollment</td>
<td>REST</td>
<td>추천 계산에 수강 이력이 필요</td>
</tr>
<tr>
<td>Recommend → Course</td>
<td>REST</td>
<td>추천 대상 강의 목록이 필요</td>
</tr>
<tr>
<td><code>payment.completed</code></td>
<td>Kafka</td>
<td>결제 완료 사실을 수강 서비스에 느슨하게 전달</td>
</tr>
<tr>
<td><code>enrollment.completed</code></td>
<td>Kafka</td>
<td>수강 완료 사실로 추천 갱신을 촉발</td>
</tr>
</tbody></table>
<h2 id="48-수강신청-전체-통신-흐름">4.8 수강신청 전체 통신 흐름</h2>
<pre><code class="language-text">Client
  │ POST /enrollments + JWT
  ▼
API Gateway
  │ 토큰 검증, Enrollment Service로 라우팅
  ▼
Enrollment Service
  ├─ REST → Course Service: 강의 존재 확인
  ├─ enrollments에 PENDING 생성
  └─ REST → Payment Service: 결제 요청
                         │
                         ├─ payments에 COMPLETED 기록
                         └─ Kafka에 payment.completed 발행
                                           │
                                           ▼
                              Enrollment Service가 소비
                              PENDING → ACTIVE 변경
                              enrollment.completed 발행
                                           │
                                           ▼
                              Recommend Service가 소비
                              수강 이력·강의 목록 REST 조회
                              추천 결과 갱신</code></pre>
<h3 id="꼭-구분하기">꼭 구분하기</h3>
<p>Payment Service가 결제 완료 응답을 주는 것은 동기 흐름이고, 이후 <code>payment.completed</code>로 다른 서비스의 상태를 바꾸는 것은 비동기 흐름이다. 비동기 이벤트를 쓴다고 해서 전체 요청이 모두 비동기가 되는 것은 아니다.</p>
<h2 id="49-핵심-데이터-모델">4.9 핵심 데이터 모델</h2>
<table>
<thead>
<tr>
<th>테이블</th>
<th>주요 필드</th>
<th>관계/상태</th>
</tr>
</thead>
<tbody><tr>
<td>users</td>
<td>id, name, email, password, role</td>
<td>STUDENT / INSTRUCTOR</td>
</tr>
<tr>
<td>courses</td>
<td>id, title, category, price, instructor_id</td>
<td>강사가 여러 강의를 등록</td>
</tr>
<tr>
<td>enrollments</td>
<td>id, user_id, course_id, status</td>
<td>PENDING → ACTIVE</td>
</tr>
<tr>
<td>payments</td>
<td>id, user_id, course_id, amount, status</td>
<td>COMPLETED / FAILED</td>
</tr>
</tbody></table>
<p>교육용 템플릿은 MariaDB 단일 인스턴스를 사용한다. 실무의 MSA에서는 서비스별 데이터 소유권과 독립성을 더 엄격히 설계해야 하며, 서비스가 다른 서비스의 테이블을 마음대로 수정하면 경계가 무너질 수 있다.</p>
<h2 id="410-day-1과-day-2의-연결">4.10 Day 1과 Day 2의 연결</h2>
<table>
<thead>
<tr>
<th>Day 1에서 정하는 것</th>
<th>Day 2에서 사용하는 방법</th>
</tr>
</thead>
<tbody><tr>
<td>Pain Point와 이해관계자 가치</td>
<td>무엇을 구현해야 하는지 판단하는 근거</td>
</tr>
<tr>
<td>User Story</td>
<td>구현할 사용자 흐름</td>
</tr>
<tr>
<td>Acceptance Criteria</td>
<td>테스트와 데모 성공 기준</td>
</tr>
<tr>
<td>Sprint Goal</td>
<td>팀 전체가 맞출 방향</td>
</tr>
<tr>
<td>Sprint Backlog와 Task</td>
<td>실제 구현 작업 목록</td>
</tr>
<tr>
<td>Story Point와 우선순위</td>
<td>Sprint 범위 조정 기준</td>
</tr>
<tr>
<td>API 계약</td>
<td>프론트·백엔드 병렬 작업의 약속</td>
</tr>
</tbody></table>
<p>교재의 핵심 문장대로, <strong>Day 1의 상세한 인수 기준이 Day 2의 테스트 기준</strong>이 된다.</p>
<h2 id="411-sprint-1과-sprint-2-범위">4.11 Sprint 1과 Sprint 2 범위</h2>
<h3 id="sprint-1---walking-skeleton--mvp">Sprint 1 - Walking Skeleton / MVP</h3>
<p>회원·강의·수강신청 등 핵심 2~3개 도메인을 골라 사용자가 처음부터 끝까지 실제로 사용할 수 있는 한 흐름을 완성한다.</p>
<p>예시:</p>
<pre><code class="language-text">로그인 → 강의 목록 → 강의 선택 → 수강신청(PENDING 확인)</code></pre>
<p>여러 서비스를 조금씩 건드리고 아무 흐름도 끝내지 않는 것이 아니라, 좁더라도 끝까지 동작하는 뼈대를 만든다.</p>
<h3 id="sprint-2---확장">Sprint 2 - 확장</h3>
<ul>
<li>결제 도메인 확장</li>
<li><code>payment.completed</code> 이벤트와 상태 자동 변경</li>
<li>추천 규칙 확장</li>
<li>필요한 경우 새 서비스와 Gateway 라우팅 추가</li>
</ul>
<p>인프라 역할의 Auth Server, API Gateway, Eureka Server는 제공된 구조를 사용하며 실습의 주 수정 대상에서 제외한다.</p>
<hr />
<h1 id="part-5-모의-프로젝트-가이드">Part 5. 모의 프로젝트 가이드</h1>
<h2 id="51-팀-역할-배분">5.1 팀 역할 배분</h2>
<table>
<thead>
<tr>
<th>역할</th>
<th align="right">권장 인원</th>
<th>실습 책임</th>
<th>산출물 예시</th>
</tr>
</thead>
<tbody><tr>
<td>Product Owner</td>
<td align="right">1</td>
<td>Pain Point·가치 정의, 우선순위, Acceptance Criteria 확정</td>
<td>Product Backlog, Sprint Goal</td>
</tr>
<tr>
<td>Scrum Master</td>
<td align="right">1</td>
<td>Planning/Daily 진행, Timebox, 장애물·Board 관리</td>
<td>진행 규칙, 장애물 목록, Retro 액션</td>
</tr>
<tr>
<td>Backend Dev</td>
<td align="right">팀 상황에 맞게</td>
<td>서비스 API 확인·구현·명세</td>
<td>API, Swagger 문서, 테스트</td>
</tr>
<tr>
<td>Frontend Dev</td>
<td align="right">팀 상황에 맞게</td>
<td>사용자 흐름과 API 연결</td>
<td>동작 화면, 요청·응답 처리</td>
</tr>
<tr>
<td>공통 Dev Team</td>
<td align="right">전원</td>
<td>추정, 리뷰, 통합, Done 달성</td>
<td>동작하는 Increment</td>
</tr>
</tbody></table>
<p>역할별 할 일은 달라도 괜찮다. 중요한 것은 결과가 같은 Sprint Goal로 합쳐지는 것이다. API 명세가 공통 계약이 되면 백엔드 구현과 프론트 화면 작업을 병렬로 진행할 수 있다.</p>
<h2 id="52-팀-아이디어로-치환하는-방법">5.2 팀 아이디어로 치환하는 방법</h2>
<p>제공 템플릿의 명칭만 바꾸는 것이 아니라 <strong>역할과 책임의 대응 관계</strong>를 찾는다.</p>
<table>
<thead>
<tr>
<th>교육 플랫폼 개념</th>
<th>쇼핑 서비스 예시</th>
<th>우리 팀 아이디어</th>
</tr>
</thead>
<tbody><tr>
<td>강사</td>
<td>판매자</td>
<td></td>
</tr>
<tr>
<td>강의</td>
<td>상품</td>
<td></td>
</tr>
<tr>
<td>수강신청</td>
<td>주문</td>
<td></td>
</tr>
<tr>
<td>결제</td>
<td>결제</td>
<td></td>
</tr>
<tr>
<td>수강 권한 활성화</td>
<td>주문 확정·상품 접근</td>
<td></td>
</tr>
<tr>
<td>추천 강의</td>
<td>추천 상품</td>
<td></td>
</tr>
</tbody></table>
<p>표를 채우면 화면, API, 서비스 경계가 자연스럽게 드러난다. 단순 명사 치환 뒤에는 반드시 “누가 어떤 가치를 얻는가?”가 있어야 한다.</p>
<h2 id="53-샘플-user-story-3종">5.3 샘플 User Story 3종</h2>
<h3 id="story-a---인증">Story A - 인증</h3>
<blockquote>
<p><strong>As a</strong> 수강생, <strong>I want</strong> 안전하게 로그인하고 싶다, <strong>So that</strong> 내 수강 정보에 접근할 수 있다.</p>
</blockquote>
<p>Acceptance Criteria:</p>
<ul>
<li>올바른 계정이면 Access Token이 발급된다.</li>
<li>잘못된 계정이면 명확한 오류가 표시된다.</li>
<li>이후 보호 API 요청에는 Bearer Token이 포함된다.</li>
</ul>
<h3 id="story-b---gateway">Story B - Gateway</h3>
<blockquote>
<p><strong>As a</strong> 시스템 운영자, <strong>I want</strong> 외부 요청이 API Gateway를 거치게 하고 싶다, <strong>So that</strong> 인증과 라우팅 정책을 한곳에서 일관되게 적용할 수 있다.</p>
</blockquote>
<p>Acceptance Criteria:</p>
<ul>
<li><code>/courses/**</code> 요청이 Course Service로 전달된다.</li>
<li>보호 경로에 토큰이 없으면 요청이 거부된다.</li>
<li>정상 토큰의 사용자 정보가 하위 서비스로 전달된다.</li>
</ul>
<h3 id="story-c---kafka-이벤트">Story C - Kafka 이벤트</h3>
<blockquote>
<p><strong>As a</strong> 수강생, <strong>I want</strong> 결제가 끝나면 수강 상태가 자동으로 활성화되길 원한다, <strong>So that</strong> 별도의 재신청 없이 바로 학습을 시작할 수 있다.</p>
</blockquote>
<p>Acceptance Criteria:</p>
<ul>
<li>결제 완료 시 <code>payment.completed</code> 이벤트가 발행된다.</li>
<li>Enrollment Service가 이벤트를 받아 상태를 ACTIVE로 바꾼다.</li>
<li>같은 이벤트가 재전달되어도 상태가 잘못 중복 변경되지 않는다.</li>
</ul>
<h2 id="54-실행-체크리스트">5.4 실행 체크리스트</h2>
<h3 id="팀-킥오프">팀 킥오프</h3>
<ul>
<li><input disabled="" type="checkbox" /> 팀원과 역할을 정했다.</li>
<li><input disabled="" type="checkbox" /> 이해관계자와 Pain Point를 한 문장으로 정의했다.</li>
<li><input disabled="" type="checkbox" /> 교육 플랫폼과 우리 도메인의 매핑표를 채웠다.</li>
<li><input disabled="" type="checkbox" /> Sprint 1의 핵심 사용자 흐름 하나를 정했다.</li>
</ul>
<h3 id="backlog-준비">Backlog 준비</h3>
<ul>
<li><input disabled="" type="checkbox" /> 컴포넌트 또는 도메인별 Epic을 나열했다.</li>
<li><input disabled="" type="checkbox" /> 각 Epic에서 최소 2개의 User Story를 작성했다.</li>
<li><input disabled="" type="checkbox" /> 상위 Story에 Acceptance Criteria를 작성했다.</li>
<li><input disabled="" type="checkbox" /> INVEST와 MoSCoW를 적용했다.</li>
<li><input disabled="" type="checkbox" /> Sprint 1과 Sprint 2로 나눈 이유를 설명할 수 있다.</li>
</ul>
<h3 id="기술-연결-확인">기술 연결 확인</h3>
<ul>
<li><input disabled="" type="checkbox" /> 각 Story가 어느 서비스와 API에 연결되는지 표시했다.</li>
<li><input disabled="" type="checkbox" /> REST와 Kafka를 사용하는 구간을 구분했다.</li>
<li><input disabled="" type="checkbox" /> Swagger의 Method, URL, Request, Response를 확인했다.</li>
<li><input disabled="" type="checkbox" /> 인증 요청과 공개 요청을 구분했다.</li>
<li><input disabled="" type="checkbox" /> 외부 호출은 Gateway 주소를 사용한다.</li>
</ul>
<h3 id="환경-확인">환경 확인</h3>
<ul>
<li><input disabled="" type="checkbox" /> 제공된 인프라 이미지와 소스가 준비되어 있다.</li>
<li><input disabled="" type="checkbox" /> <code>docker compose up -d</code> 후 컨테이너 상태를 확인했다.</li>
<li><input disabled="" type="checkbox" /> Eureka Dashboard에서 서비스 등록 상태를 확인했다.</li>
<li><input disabled="" type="checkbox" /> 필요한 API를 Swagger <code>Try it out</code>으로 호출했다.</li>
<li><input disabled="" type="checkbox" /> 민감한 값은 저장소에 직접 올리지 않는다.</li>
</ul>
<h2 id="55-막혔을-때-좋은-질문법">5.5 막혔을 때 좋은 질문법</h2>
<p>나쁜 질문:</p>
<blockquote>
<p>“Eureka가 왜 안 되나요?”</p>
</blockquote>
<p>좋은 질문:</p>
<blockquote>
<p>“Gateway를 통해 <code>GET /courses</code>를 호출했습니다. 요청 URL은 <code>http://localhost:8080/...</code>이고 응답 상태는 503입니다. Eureka에는 COURSE-SERVICE가 등록되어 있습니다. 어느 구간을 더 확인해야 하나요?”</p>
</blockquote>
<p>문제를 <strong>내가 보낸 요청, 기대 결과, 실제 결과, 이미 확인한 내용</strong>으로 좁히면 해결이 빨라진다.</p>
<h2 id="56-발표-기획서의-논리-구조">5.6 발표 기획서의 논리 구조</h2>
<ol>
<li><strong>왜</strong>: 이해관계자의 Pain Point</li>
<li><strong>무엇을</strong>: 이를 해결할 서비스와 AI의 역할</li>
<li><strong>어떻게 나누어</strong>: Sprint 1 MVP와 Sprint 2 확장</li>
<li><strong>어떤 구조로</strong>: MSA 서비스와 통신 구성도</li>
<li><strong>어떤 계약으로</strong>: API 명세</li>
<li><strong>무슨 결과가</strong>: 동작 화면과 상태 변화</li>
</ol>
<p>이 순서를 지키면 기술 나열이 아니라 가치에서 구현까지 이어지는 설명이 된다.</p>
<hr />
<h1 id="7-초보자가-자주-헷갈리는-개념-정리">7. 초보자가 자주 헷갈리는 개념 정리</h1>
<table>
<thead>
<tr>
<th>헷갈리는 개념</th>
<th>구분</th>
</tr>
</thead>
<tbody><tr>
<td>Agile vs Scrum</td>
<td>Agile은 가치와 원칙에 가까운 접근법, Scrum은 이를 운영하는 프레임워크</td>
</tr>
<tr>
<td>Product Backlog vs Sprint Backlog</td>
<td>전체 제품 후보 목록 vs 이번 Sprint 실행 계획</td>
</tr>
<tr>
<td>User Story vs Task</td>
<td>사용자 가치 단위 vs 그것을 만들기 위한 기술 작업</td>
</tr>
<tr>
<td>Acceptance Criteria vs DoD</td>
<td>Story별 기능 성공 조건 vs 모든 작업에 공통인 품질 기준</td>
</tr>
<tr>
<td>Sprint Review vs Retro</td>
<td>제품 결과와 피드백 검토 vs 팀의 일하는 방식 개선</td>
</tr>
<tr>
<td>Story Point vs 시간</td>
<td>상대적 규모·불확실성 vs 절대 시간</td>
</tr>
<tr>
<td>Authentication vs Authorization</td>
<td>누구인지 확인 vs 무엇을 할 수 있는지 확인</td>
</tr>
<tr>
<td>API Gateway vs Eureka</td>
<td>요청을 받아 전달하는 문지기 vs 서비스 위치를 알려주는 전화번호부</td>
</tr>
<tr>
<td>REST vs Kafka</td>
<td>즉시 응답을 기다리는 호출 vs 사건을 비동기로 알리는 이벤트</td>
</tr>
<tr>
<td>Monolith vs MSA</td>
<td>하나의 배포 단위 vs 독립적인 여러 서비스 단위</td>
</tr>
</tbody></table>
<hr />
<h1 id="8-최종-자가-점검-문제">8. 최종 자가 점검 문제</h1>
<h2 id="개념-확인">개념 확인</h2>
<ol>
<li>요구사항 변화가 큰 프로젝트에서 Agile이 Waterfall보다 유리할 수 있는 이유는 무엇인가?</li>
<li>Sprint Review와 Retrospective는 무엇을 각각 검토하는가?</li>
<li>“로그인 기능 개발”이 좋은 User Story가 아닌 이유를 세 가지 말해 보자.</li>
<li>Story Point를 사람의 성과 점수나 시간으로 쓰면 안 되는 이유는 무엇인가?</li>
<li>Gateway와 Eureka의 역할 차이를 건물 비유로 설명해 보자.</li>
<li>Enrollment가 Course 존재 여부를 REST로 확인하고, 결제 완료를 Kafka로 받는 이유는 무엇인가?</li>
<li>Sprint 1에 모든 기능을 조금씩 넣는 것보다 하나의 Walking Skeleton을 완성해야 하는 이유는 무엇인가?</li>
</ol>
<h2 id="답을-말할-때-포함해야-할-핵심">답을 말할 때 포함해야 할 핵심</h2>
<ol>
<li>짧은 피드백 주기, 위험 조기 발견, 우선순위 변경</li>
<li>Review는 제품, Retro는 프로세스와 협업 방식</li>
<li>사용자·가치·검증 기준 부재</li>
<li>상대적 복잡도와 불확실성의 팀 기준이기 때문</li>
<li>Gateway는 안내 데스크, Eureka는 전화번호부</li>
<li>즉시 판단이 필요한 조회 vs 사건의 느슨한 전파</li>
<li>실제 사용 가능한 흐름으로 피드백을 받아야 하기 때문</li>
</ol>
<hr />
<h1 id="9-day-1-핵심-요약">9. Day 1 핵심 요약</h1>
<ol>
<li>요구사항은 변한다. Agile은 짧은 주기로 동작하는 결과물을 검증하며 위험을 줄인다.</li>
<li>Scrum은 PO·SM·개발팀이 Product Backlog를 기반으로 Planning, Daily, Review, Retro를 반복하는 틀이다.</li>
<li>User Story는 “누가, 무엇을, 왜” 원하는지를 표현하고, Acceptance Criteria는 성공 조건을 정한다.</li>
<li>Product Backlog는 전체 후보, Sprint Backlog는 이번 Sprint의 실행 계획이다.</li>
<li>Story Point는 시간이 아니라 팀이 합의한 상대적 규모다.</li>
<li>MSA는 서비스를 책임 단위로 분리해 독립 변경·배포를 돕지만, 통신과 운영 복잡도라는 비용이 있다.</li>
<li>Gateway는 단일 진입점, Eureka는 서비스 탐색, Auth는 토큰 발급·검증, Kafka는 비동기 이벤트 전달을 담당한다.</li>
<li>모의 프로젝트의 핵심 흐름은 로그인 → 강의 조회 → 수강신청 → 결제 → 수강 활성화 → 추천이다.</li>
<li>Sprint 1은 끝까지 동작하는 최소 흐름, Sprint 2는 결제·이벤트·추천 확장에 집중한다.</li>
<li>Day 1의 Goal, Story, Acceptance Criteria, Task, Point, Board가 Day 2 구현과 테스트의 기준이 된다.</li>
</ol>
<blockquote>
<p>오늘의 진짜 성과는 문서를 많이 작성하는 것이 아니다. <strong>팀이 사용자 가치 하나를 선택하고, 그것을 검증 가능한 Story와 실행 가능한 Task로 바꾸어, 내일 실제로 만들 준비를 끝내는 것</strong>이다.</p>
</blockquote>
<hr />
<h2 id="자료-기준">자료 기준</h2>
<p>이 글은 다음 제공 자료를 교차 확인해 작성했다. 수업 구성과 개념은 메인 교재를 기준으로 하고, 실제 온라인 교육 플랫폼의 최신 서비스 구조와 실습 범위는 가이드 자료로 보강했다.</p>
<ul>
<li>메인: <code>교재.Cloud_Agile 방법론 및 MSA 개발_임성열.pdf</code> - Day 1 범위 3~46쪽</li>
<li>진행 보강: <code>가이드1.Agile_MSA_진행_가이드.pdf</code></li>
<li>실습 범위·초보자 관점: <code>가이드2.Agile_MSA_실습_가이드_수정_v2.pdf</code></li>
<li>실제 시스템·API·통신 흐름: <code>가이드3.코드 템플릿 설명 문서.msa-practice_scenario.pdf</code></li>
<li>사례·아키텍처 선택 관점: <code>보강1.Agile_MSA_사례로_살펴보는_이해_v2.pdf</code></li>
<li>컨테이너 실행 개념 보조: <code>보강2.Docker_컨테이너_기초_가이드_수정.pdf</code></li>
</ul>