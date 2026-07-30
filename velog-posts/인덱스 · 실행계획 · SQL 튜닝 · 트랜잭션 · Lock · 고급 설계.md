<h2 id="목차">목차</h2>
<ol>
<li>인덱스(Index) — 왜 필요한가</li>
<li>실행계획(Execution Plan) 읽는 법</li>
<li>SQL 튜닝 — 자주 하는 실수 8가지</li>
<li>MVCC와 트랜잭션 격리 수준</li>
<li>Lock과 Deadlock</li>
<li>고급 DB 설계 (정규화 심화 / 샤딩 / MSA)</li>
<li>(보너스) 운영 실무 — 백업, VACUUM, Connection Pool</li>
</ol>
<hr />
<h2 id="1-인덱스index--왜-필요한가">1. 인덱스(Index) — 왜 필요한가</h2>
<h3 id="1-1-인덱스가-없다면">1-1. 인덱스가 없다면?</h3>
<p>책 뒤에 &quot;찾아보기(색인)&quot;가 없다고 상상해보세요. &quot;PostgreSQL&quot;이라는 단어를 찾으려면 첫 페이지부터 끝까지 한 장씩 넘겨봐야 합니다. 데이터베이스도 똑같습니다. 인덱스가 없으면 조건에 맞는 행을 찾기 위해 테이블 전체를 처음부터 끝까지 훑습니다 — 이걸 <strong>Full Table Scan(순차 스캔)</strong>이라고 부릅니다.</p>
<table>
<thead>
<tr>
<th>상황</th>
<th>방식</th>
<th>시간 복잡도</th>
</tr>
</thead>
<tbody><tr>
<td>인덱스 없음</td>
<td>Table Full Scan (전체 행 순차 검사)</td>
<td>O(N)</td>
</tr>
<tr>
<td>인덱스 있음</td>
<td>B-Tree 탐색</td>
<td>O(log N)</td>
</tr>
</tbody></table>
<p>행이 몇백만 개면 이 차이가 수십 밀리초 vs 수 초의 차이로 벌어집니다.</p>
<h3 id="1-2-인덱스의-트레이드오프">1-2. 인덱스의 트레이드오프</h3>
<p>인덱스가 &quot;무조건 좋은 것&quot;은 아닙니다. 다음 세 가지 비용을 반드시 기억하세요.</p>
<ul>
<li><strong>읽기 성능 향상</strong> — WHERE / JOIN / ORDER BY에 자주 쓰이는 컬럼을 빠르게 찾음</li>
<li><strong>쓰기 비용 증가</strong> — INSERT/UPDATE/DELETE 시마다 인덱스도 같이 재정렬해야 함</li>
<li><strong>저장 공간 증가</strong> — 인덱스 자체도 디스크 공간을 차지함</li>
</ul>
<blockquote>
<p> 그래서 &quot;아무 컬럼에나 인덱스를 걸면 좋다&quot;가 아니라, <strong>자주 조회하는 컬럼에만 선택적으로</strong> 걸어야 합니다.</p>
</blockquote>
<h3 id="1-3-인덱스-설계-원칙">1-3. 인덱스 설계 원칙</h3>
<ul>
<li>WHERE 조건에 자주 등장하는 컬럼 우선</li>
<li><strong>선택도(Selectivity)</strong>가 높은 컬럼일수록 효과가 큼</li>
<li>FK(외래키) 컬럼에는 반드시 인덱스</li>
<li>조회 안 되는(idx_scan = 0) 인덱스는 제거</li>
</ul>
<h4 id="선택도selectivity란">선택도(Selectivity)란?</h4>
<blockquote>
<p>선택도 = 컬럼의 고유값 수 ÷ 전체 행 수</p>
</blockquote>
<ul>
<li><code>gender</code>(남/여 두 값만 존재) → 선택도 약 0.5 → <strong>나쁜 인덱스 후보</strong> (인덱스를 타도 결국 데이터 절반을 걸러야 함)</li>
<li><code>email</code>(거의 다 고유값) → 선택도 ≈ 1 → <strong>좋은 인덱스 후보</strong></li>
</ul>
<h3 id="1-4-인덱스-종류">1-4. 인덱스 종류</h3>
<table>
<thead>
<tr>
<th>인덱스</th>
<th>구조</th>
<th>최적 용도</th>
</tr>
</thead>
<tbody><tr>
<td><strong>B-Tree</strong></td>
<td>균형 이진 트리</td>
<td><code>=</code>, <code>BETWEEN</code>, <code>LIKE 'prefix%'</code>, <code>ORDER BY</code> — 가장 기본, 모든 DBMS 지원</td>
</tr>
<tr>
<td><strong>Hash</strong></td>
<td>해시 테이블</td>
<td>등호(<code>=</code>)만, 범위 검색 불가</td>
</tr>
<tr>
<td><strong>GIN</strong></td>
<td>역인덱스</td>
<td>JSONB, 배열, 전문검색(Full Text Search)</td>
</tr>
<tr>
<td><strong>GiST</strong></td>
<td>일반화 검색 트리</td>
<td>지리정보, 범위 타입, 벡터 유사도</td>
</tr>
<tr>
<td><strong>BRIN</strong></td>
<td>블록 범위 최소/최대</td>
<td>시계열 데이터, 물리적으로 정렬된 대용량 데이터</td>
</tr>
<tr>
<td><strong>Bitmap</strong></td>
<td>비트 배열</td>
<td>저선택도 컬럼 조합 (DW 환경)</td>
</tr>
</tbody></table>
<blockquote>
<p>실무에서 90%는 B-Tree만 알아도 충분합니다. 나머지는 &quot;이런 상황에 이런 게 있구나&quot; 정도로 기억해두세요.</p>
</blockquote>
<h3 id="1-5-복합-인덱스--컬럼-순서가-중요하다">1-5. 복합 인덱스 — 컬럼 순서가 중요하다</h3>
<p>여러 컬럼으로 인덱스를 만들 때는 <strong>선두 컬럼 원칙(leftmost prefix rule)</strong>을 지켜야 합니다.</p>
<pre><code class="language-sql">-- 복합 인덱스 생성: 등호(=) 조건 컬럼을 앞에, 범위(&lt;,&gt;) 조건 컬럼을 뒤에
CREATE INDEX idx_orders_cust_date ON orders (customer_id, order_date DESC);</code></pre>
<ul>
<li><code>WHERE customer_id = 5 AND order_date &gt; '2024-01-01'</code> → <strong>인덱스 사용 가능</strong> (앞 컬럼부터 순서대로 조건이 있음)</li>
<li><code>WHERE order_date &gt; '2024-01-01'</code> (customer_id 조건 없음) → <strong>인덱스 사용 불가</strong> (선두 컬럼을 건너뜀)</li>
</ul>
<h3 id="1-6-커버링-인덱스--가장-빠른-조회">1-6. 커버링 인덱스 — 가장 빠른 조회</h3>
<pre><code class="language-sql">-- 일반 인덱스: 인덱스로 위치만 찾고, 실제 데이터는 테이블에서 다시 읽어옴 (Index Scan)
CREATE INDEX idx_email ON users (email);

-- 커버링 인덱스: SELECT할 컬럼을 인덱스 안에 통째로 포함 → 테이블을 아예 안 읽음! (Index Only Scan)
CREATE INDEX idx_email_cover ON users (email) INCLUDE (id, name);
SELECT id, name FROM users WHERE email = 'a@b.com';
-- → Index Only Scan: 테이블 접근(Heap Fetch) 없이 인덱스만으로 응답 완료</code></pre>
<h3 id="1-7-실전-효과--숫자로-보는-인덱스-위력">1-7. 실전 효과 — 숫자로 보는 인덱스 위력</h3>
<p>100만 건 데이터에서 특정 <code>user_id</code>를 조회하는 쿼리:</p>
<pre><code class="language-sql">-- Before: 인덱스 없이 조회
EXPLAIN ANALYZE SELECT * FROM query_logs WHERE user_id = 'u123';
-- Seq Scan ... (actual time=120..1850ms)

-- 인덱스 생성
CREATE INDEX idx_query_logs_user ON query_logs(user_id);

-- After: 인덱스로 조회
-- Index Scan ... (actual time=0.1..0.8ms)</code></pre>
<p><strong>결과: 1850ms → 0.8ms, 약 2,300배 향상</strong> 🚀</p>
<h3 id="1단원-핵심-정리">1단원 핵심 정리</h3>
<ul>
<li>인덱스는 &quot;찾아보기&quot;다. 없으면 O(N), 있으면 O(log N)</li>
<li>공짜가 아니다 — 쓰기 비용/저장공간 증가</li>
<li>선택도 높은 컬럼(중복 적은 컬럼)에 걸어야 효과가 크다</li>
<li>복합 인덱스는 <strong>등호 조건 → 범위 조건</strong> 순서로 설계</li>
<li>SELECT 컬럼까지 인덱스에 담으면 <code>Index Only Scan</code>으로 최고 속도</li>
</ul>
<hr />
<h2 id="2-실행계획execution-plan-읽는-법">2. 실행계획(Execution Plan) 읽는 법</h2>
<h3 id="2-1-실행계획이란">2-1. 실행계획이란?</h3>
<p>DB 옵티마이저가 SQL을 실제로 실행하기 전에, &quot;어떤 순서로 테이블을 읽고, 어떤 인덱스를 쓰고, 어떤 조인 방식을 택할지&quot; 미리 세우는 계획입니다. <strong>&quot;이 쿼리가 왜 느린지&quot;를 눈으로 보는 창</strong>이라고 생각하면 됩니다.</p>
<h3 id="2-2-explain-vs-explain-analyze">2-2. EXPLAIN vs EXPLAIN ANALYZE</h3>
<table>
<thead>
<tr>
<th>명령어</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>EXPLAIN</code></td>
<td>추정 실행 계획만 보여줌 (쿼리를 실제로 실행하지 않음)</td>
</tr>
<tr>
<td><code>EXPLAIN ANALYZE</code></td>
<td><strong>실제로 쿼리를 실행</strong>하고 실측값까지 보여줌 (UPDATE/DELETE라면 ROLLBACK 권장!)</td>
</tr>
<tr>
<td><code>EXPLAIN (ANALYZE, BUFFERS)</code></td>
<td>여기에 더해 메모리/디스크 읽기까지 측정</td>
</tr>
</tbody></table>
<pre><code class="language-sql">BEGIN;
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT s.name, c.title, e.score
FROM students s
JOIN enrollments e ON e.student_id = s.id
JOIN courses c ON c.id = e.course_id
WHERE s.major_id = 1;
ROLLBACK;  -- DML을 포함한 쿼리라면 실제 반영을 막기 위해 롤백</code></pre>
<h3 id="2-3-실행계획을-꼭-확인해야-하는-순간">2-3. 실행계획을 꼭 확인해야 하는 순간</h3>
<ul>
<li>쿼리가 갑자기 느려졌을 때</li>
<li>인덱스를 만들었는데 안 쓰이는 것 같을 때</li>
<li>JOIN 방식이 이상해 보일 때 (Hash인지 Nested Loop인지)</li>
<li>예측된 rows 수와 실제(actual) rows 수가 크게 다를 때 → 통계 정보 갱신(<code>ANALYZE</code>) 필요</li>
</ul>
<h3 id="2-4-주요-노드-타입-해석표">2-4. 주요 노드 타입 해석표</h3>
<table>
<thead>
<tr>
<th>노드 타입</th>
<th>의미</th>
<th>좋음/나쁨</th>
<th>조치</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Seq Scan</strong></td>
<td>테이블 전체 순차 스캔</td>
<td>보통 Bad</td>
<td>인덱스 추가 (단, 전체의 10% 이상을 읽는 경우엔 오히려 정상일 수 있음)</td>
</tr>
<tr>
<td><strong>Index Scan</strong></td>
<td>인덱스로 위치 찾고 테이블도 접근</td>
<td>Good</td>
<td>FK/WHERE 컬럼에 인덱스가 걸려 있는지 확인</td>
</tr>
<tr>
<td><strong>Index Only Scan</strong></td>
<td>인덱스만으로 결과 반환</td>
<td>Best</td>
<td>커버링 인덱스 설계</td>
</tr>
<tr>
<td><strong>Hash Join</strong></td>
<td>해시 테이블 빌드 후 매칭</td>
<td>대용량 등가 조인에 적합</td>
<td><code>work_mem</code> 부족 시 디스크로 Spill 발생</td>
</tr>
<tr>
<td><strong>Nested Loop</strong></td>
<td>한쪽 테이블 행마다 반복 탐색</td>
<td>소규모+인덱스에 적합</td>
<td>외부 집합이 커지면 급격히 느려짐</td>
</tr>
<tr>
<td><strong>Sort</strong></td>
<td>결과 정렬</td>
<td>확인 필요</td>
<td>ORDER BY 컬럼에 인덱스 추가하면 이 노드 자체가 사라질 수 있음</td>
</tr>
</tbody></table>
<h3 id="2단원-핵심-정리">2단원 핵심 정리</h3>
<ul>
<li><code>EXPLAIN</code>은 예상, <code>EXPLAIN ANALYZE</code>는 실측 (DML엔 ROLLBACK 필수)</li>
<li><code>Seq Scan</code>이 보이면 인덱스를 의심하자</li>
<li><code>Index Only Scan</code>이 가장 빠르다 (테이블 접근 자체가 없음)</li>
<li>예측 rows ↔ 실제 rows 차이가 크면 통계 갱신(<code>ANALYZE</code>)부터</li>
</ul>
<hr />
<h2 id="3-sql-튜닝--자주-하는-실수-8가지">3. SQL 튜닝 — 자주 하는 실수 8가지</h2>
<p>인덱스와 실행계획을 알아도, 쿼리 자체를 잘못 쓰면 인덱스가 있어도 무용지물이 됩니다. 아래 8가지 &quot;안티패턴&quot;은 실무에서 정말 자주 나옵니다.</p>
<table>
<thead>
<tr>
<th>#</th>
<th>안티패턴</th>
<th>문제</th>
<th>올바른 방법</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>인덱스 컬럼에 함수 적용</td>
<td><code>WHERE YEAR(date)=2024</code> → 인덱스 무력화</td>
<td><code>WHERE date &gt;= '2024-01-01' AND date &lt; '2025-01-01'</code></td>
</tr>
<tr>
<td>2</td>
<td><code>SELECT *</code></td>
<td>불필요한 데이터 전송, 커버링 인덱스 불가</td>
<td>필요한 컬럼만 명시</td>
</tr>
<tr>
<td>3</td>
<td>암묵적 타입 변환</td>
<td><code>WHERE user_id = '12345'</code> (컬럼이 INT인데 문자열 비교)</td>
<td><code>WHERE user_id = 12345</code></td>
</tr>
<tr>
<td>4</td>
<td><code>LIKE '%키워드%'</code> (앞 와일드카드)</td>
<td>B-Tree 인덱스 사용 불가</td>
<td><code>LIKE '키워드%'</code>(prefix) 또는 GIN + <code>pg_trgm</code></td>
</tr>
<tr>
<td>5</td>
<td>N+1 문제</td>
<td>루프 안에서 쿼리를 1+N번 실행</td>
<td>JOIN 한 번으로 처리</td>
</tr>
<tr>
<td>6</td>
<td><code>NOT IN</code> + NULL 함정</td>
<td>서브쿼리 결과에 NULL 섞이면 예상과 다르게 동작</td>
<td><code>NOT EXISTS</code> 사용</td>
</tr>
<tr>
<td>7</td>
<td><code>DISTINCT</code> 남용</td>
<td>중복 원인을 파악하지 않고 습관적으로 제거</td>
<td><code>EXISTS</code>로 존재 여부만 확인</td>
</tr>
<tr>
<td>8</td>
<td>대용량 <code>OFFSET</code></td>
<td><code>LIMIT 10 OFFSET 10000</code> → 만 개를 읽고 버림</td>
<td>Keyset(Cursor) 방식: <code>WHERE id &gt; :last_id ORDER BY id LIMIT 10</code></td>
</tr>
</tbody></table>
<h3 id="예시--안티패턴-1-4-상세">예시 — 안티패턴 1, 4 상세</h3>
<pre><code class="language-sql">-- ❌ 인덱스에 함수를 적용하면 인덱스를 못 탐
WHERE YEAR(order_date) = 2024

-- ✅ 범위 조건으로 바꾸면 인덱스 그대로 사용 가능
WHERE order_date &gt;= '2024-01-01' AND order_date &lt; '2025-01-01'

-- ❌ 앞에 %가 붙으면 B-Tree 인덱스가 무력화됨
WHERE name LIKE '%길동%'

-- ✅ 접두어 검색이면 인덱스 사용 가능
WHERE name LIKE '홍%'

-- ✅ 중간 검색이 꼭 필요하다면 GIN + pg_trgm 확장 사용
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_name_trgm ON users USING GIN (name gin_trgm_ops);</code></pre>
<h3 id="느린-쿼리-찾아내기--pg_stat_statements">느린 쿼리 찾아내기 — pg_stat_statements</h3>
<p>&quot;어떤 쿼리가 느린지&quot; 감으로 찾기보다, 통계로 찾는 게 정확합니다.</p>
<pre><code class="language-sql">-- PostgreSQL: 확장 기능 활성화
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 평균 실행 시간 기준 느린 쿼리 TOP 10
SELECT
    ROUND(mean_exec_time::NUMERIC, 2) AS avg_ms,
    calls,
    LEFT(query, 100) AS query_snippet
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;</code></pre>
<h3 id="파티셔닝--테이블을-잘게-쪼개기">파티셔닝 — 테이블을 잘게 쪼개기</h3>
<p>데이터가 아주 많아지면(수천만 건 이상), 테이블 자체를 물리적으로 쪼개서 관리하는 <strong>파티셔닝</strong>을 고려합니다.</p>
<pre><code class="language-sql">-- 날짜 기준 Range 파티셔닝
CREATE TABLE sales (
    id BIGSERIAL,
    sale_date DATE NOT NULL,
    amount NUMERIC
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2024 PARTITION OF sales
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Partition Pruning: 조건에 맞는 파티션만 스캔 (다른 연도는 아예 건드리지 않음)
EXPLAIN SELECT * FROM sales WHERE sale_date BETWEEN '2024-06-01' AND '2024-06-30';
-- → sales_2024만 스캔, sales_2023/2025는 자동 제외</code></pre>
<h3 id="3단원-핵심-정리">3단원 핵심 정리</h3>
<ul>
<li>SELECT는 필요한 컬럼만, WHERE 조건 컬럼엔 함수를 씌우지 말 것</li>
<li><code>NOT EXISTS</code>가 <code>NOT IN</code>보다 안전하고 빠르다</li>
<li>무한 스크롤/페이지네이션엔 OFFSET 대신 Keyset 방식</li>
<li>느린 쿼리는 감이 아니라 <code>pg_stat_statements</code>로 확인</li>
</ul>
<hr />
<h2 id="4-mvcc와-트랜잭션-격리-수준">4. MVCC와 트랜잭션 격리 수준</h2>
<h3 id="4-1-mvcc다중버전-동시성-제어란">4-1. MVCC(다중버전 동시성 제어)란?</h3>
<p>여러 사람이 동시에 같은 데이터를 읽고 쓸 때, &quot;읽기 작업이 쓰기 작업을 막지 않도록&quot; 하는 기술입니다.</p>
<p>핵심 아이디어는 이렇습니다: <strong>데이터를 수정할 때 기존 값을 덮어쓰지 않고, 새 버전을 추가로 만듭니다.</strong> 그리고 각 트랜잭션은 자신이 시작된 시점의 &quot;스냅샷(사진)&quot;을 기준으로 데이터를 봅니다. 이렇게 하면 누군가 데이터를 읽는 동안에도 다른 누군가는 자유롭게 수정할 수 있습니다.</p>
<blockquote>
<p> 비유: 위키백과에서 문서를 &quot;편집 중&quot;이어도, 다른 사람은 편집 전의 예전 버전을 계속 읽을 수 있는 것과 비슷합니다.</p>
</blockquote>
<h3 id="4-2-격리-수준isolation-level--4단계">4-2. 격리 수준(Isolation Level) — 4단계</h3>
<p>동시에 실행되는 트랜잭션들이 서로 얼마나 &quot;간섭&quot;할 수 있는지를 정하는 게 격리 수준입니다. 엄격할수록 안전하지만 느려집니다.</p>
<table>
<thead>
<tr>
<th>격리 수준</th>
<th>Dirty Read</th>
<th>Non-Repeatable Read</th>
<th>Phantom Read</th>
</tr>
</thead>
<tbody><tr>
<td>Read Uncommitted</td>
<td>허용</td>
<td>허용</td>
<td>허용</td>
</tr>
<tr>
<td><strong>Read Committed</strong> (PostgreSQL 기본값)</td>
<td>방지</td>
<td>허용</td>
<td>허용</td>
</tr>
<tr>
<td>Repeatable Read</td>
<td>방지</td>
<td>방지</td>
<td>허용(표준상)</td>
</tr>
<tr>
<td>Serializable</td>
<td>방지</td>
<td>방지</td>
<td>방지</td>
</tr>
</tbody></table>
<p>용어가 낯설 수 있으니 하나씩 풀어볼게요.</p>
<ul>
<li><strong>Dirty Read</strong>: 아직 커밋되지 않은(확정 안 된) 남의 변경사항을 읽어버리는 것</li>
<li><strong>Non-Repeatable Read</strong>: 같은 트랜잭션 안에서 같은 데이터를 두 번 읽었는데 값이 달라지는 것</li>
<li><strong>Phantom Read</strong>: 같은 조건으로 두 번 조회했는데 행의 개수가 달라지는 것(새로 추가된 행이 보임)</li>
</ul>
<h3 id="4-3-예시로-이해하기--read-committed-vs-repeatable-read">4-3. 예시로 이해하기 — Read Committed vs Repeatable Read</h3>
<pre><code class="language-sql">-- 초기 데이터: Alice 잔액 1000원

-- [Read Committed] 기본값
-- Session 1
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM bank WHERE name='Alice';  -- 1000

-- Session 2 (동시에 실행되고 커밋됨)
UPDATE bank SET balance=800 WHERE name='Alice';
COMMIT;

-- Session 1이 다시 조회
SELECT balance FROM bank WHERE name='Alice';  -- 800 (바뀐 값이 바로 보임!)</code></pre>
<pre><code class="language-sql">-- [Repeatable Read] — 트랜잭션 시작 시점 스냅샷 유지
-- Session 1
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM bank WHERE name='Alice';  -- 1000

-- Session 2가 UPDATE 후 COMMIT 해도
SELECT balance FROM bank WHERE name='Alice';  -- 여전히 1000! (스냅샷 유지)</code></pre>
<p><strong>Serializable</strong>은 여기서 한 단계 더 나아가, 두 트랜잭션이 서로 충돌할 것 같으면 아예 하나를 강제로 실패시킵니다(<code>ERROR: could not serialize access</code>). 앱에서는 이 에러를 잡아서 <strong>재시도</strong> 로직을 넣어줘야 합니다.</p>
<h3 id="4-4-실무-적용-가이드">4-4. 실무 적용 가이드</h3>
<table>
<thead>
<tr>
<th>작업 유형</th>
<th>권장 격리 수준</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>단순 분석 SELECT</td>
<td>Read Committed</td>
<td>빠르고 최신 데이터를 봄</td>
</tr>
<tr>
<td>보고서/집계(반복 조회)</td>
<td>Repeatable Read</td>
<td>같은 트랜잭션 내 일관성 보장</td>
</tr>
<tr>
<td>금융 이체, 재고 차감</td>
<td>Serializable</td>
<td>완벽한 무결성이 최우선</td>
</tr>
<tr>
<td>장바구니, 좌석 예약</td>
<td>Serializable 또는 <code>SELECT ... FOR UPDATE</code></td>
<td>동시 접근 충돌 방지</td>
</tr>
</tbody></table>
<h3 id="4단원-핵심-정리">4단원 핵심 정리</h3>
<ul>
<li>MVCC = 덮어쓰지 않고 새 버전을 만들어 &quot;읽기가 쓰기를 막지 않게&quot; 하는 기술</li>
<li>격리 수준이 엄격할수록(Serializable에 가까울수록) 안전하지만 충돌 시 실패/재시도가 늘어남</li>
<li>PostgreSQL 기본값은 Read Committed, 대부분의 일반적인 서비스에 적합</li>
<li>돈이 걸린 로직(결제, 재고)엔 Serializable + 재시도 로직을 고려</li>
</ul>
<hr />
<h2 id="5-lock과-deadlock">5. Lock과 Deadlock</h2>
<h3 id="5-1-lock이-필요한-이유">5-1. Lock이 필요한 이유</h3>
<p>여러 트랜잭션이 같은 데이터를 동시에 변경하려고 하면 데이터 무결성이 깨질 수 있습니다. Lock은 &quot;지금 이 데이터, 내가 쓰고 있으니 잠깐 기다려&quot; 라고 표시해두는 것입니다.</p>
<h3 id="5-2-낙관적-lock-vs-비관적-lock">5-2. 낙관적 Lock vs 비관적 Lock</h3>
<table>
<thead>
<tr>
<th></th>
<th>낙관적 Lock(Optimistic)</th>
<th>비관적 Lock(Pessimistic)</th>
</tr>
</thead>
<tbody><tr>
<td>가정</td>
<td>충돌이 드물다</td>
<td>충돌이 빈번하다</td>
</tr>
<tr>
<td>구현</td>
<td>버전 컬럼으로 충돌 여부 사후 감지</td>
<td><code>SELECT ... FOR UPDATE</code>로 즉시 잠금</td>
</tr>
<tr>
<td>충돌 시</td>
<td>재시도</td>
<td>다른 트랜잭션은 그냥 대기</td>
</tr>
<tr>
<td>적합한 곳</td>
<td>블로그, 설정 변경(읽기 많고 갱신 드묾)</td>
<td>은행 이체, 재고 차감(짧고 확실하게)</td>
</tr>
</tbody></table>
<h3 id="5-3-row-lock-예시">5-3. Row Lock 예시</h3>
<pre><code class="language-sql">-- Session 1
BEGIN;
UPDATE items SET stock = stock - 1 WHERE id = 1;
-- id=1 행에 Lock 획득, 아직 COMMIT 안 함

-- Session 2 (동시에 같은 행을 수정 시도)
BEGIN;
UPDATE items SET stock = stock - 1 WHERE id = 1;
-- → 대기(blocking): Session 1이 COMMIT/ROLLBACK 할 때까지 기다림</code></pre>
<h3 id="5-4-skip-locked--작업-큐-패턴">5-4. SKIP LOCKED — 작업 큐 패턴</h3>
<p>여러 워커(worker)가 하나의 작업 큐에서 중복 없이 작업을 나눠 가져갈 때 유용한 패턴입니다.</p>
<pre><code class="language-sql">WITH picked AS (
    SELECT id FROM job_queue
    WHERE status = 'READY'
    ORDER BY id
    FOR UPDATE SKIP LOCKED   -- 다른 워커가 이미 잡고 있는 행은 건너뜀
    LIMIT 10
)
UPDATE job_queue SET status = 'RUNNING'
FROM picked WHERE job_queue.id = picked.id;</code></pre>
<h3 id="5-5-deadlock교착-상태--서로가-서로를-기다림">5-5. Deadlock(교착 상태) — 서로가 서로를 기다림</h3>
<pre><code class="language-sql">-- Session 1
BEGIN;
UPDATE items SET stock = stock - 1 WHERE id = 1;  -- row 1 Lock 획득
UPDATE items SET stock = stock - 1 WHERE id = 2;  -- row 2 요청 → 대기

-- Session 2 (거의 동시)
BEGIN;
UPDATE items SET stock = stock - 1 WHERE id = 2;  -- row 2 Lock 획득
UPDATE items SET stock = stock - 1 WHERE id = 1;  -- row 1 요청 → 대기

-- → 서로가 서로의 Lock을 기다리는 순환 대기 발생!
-- PostgreSQL이 자동으로 감지: ERROR: deadlock detected
-- 한 쪽 트랜잭션이 자동으로 ROLLBACK되고, 다른 쪽은 계속 진행됨</code></pre>
<h3 id="5-6-deadlock-예방법">5-6. Deadlock 예방법</h3>
<ol>
<li><strong>Lock 획득 순서 통일</strong> — 항상 <code>id</code> 오름차순으로 잠그면 교차 대기 자체가 발생하지 않음</li>
<li><strong>트랜잭션을 짧게 유지</strong> — 비즈니스 로직, HTTP 호출은 트랜잭션 밖에서</li>
<li><strong>Advisory Lock</strong> — 업무 단위(예: user_id)로 논리적 잠금 (<code>pg_advisory_xact_lock(42)</code>)</li>
<li><strong>재시도 전략</strong> — Deadlock 에러 코드(<code>40P01</code>)를 잡아서 짧은 지연 후 재시도</li>
</ol>
<h3 id="5단원-핵심-정리">5단원 핵심 정리</h3>
<ul>
<li>Lock = &quot;지금 내가 쓰고 있으니 기다려&quot; 표시</li>
<li>충돌이 드물면 낙관적, 자주 발생하면 비관적 Lock</li>
<li>Deadlock은 서로가 서로를 기다리는 상황 → <strong>항상 같은 순서로 Lock을 잡는 것</strong>이 가장 확실한 예방법</li>
</ul>
<hr />
<h2 id="6-고급-db-설계--정규화-심화--샤딩--msa">6. 고급 DB 설계 — 정규화 심화 / 샤딩 / MSA</h2>
<h3 id="6-1-bcnf--3nf보다-한-단계-더-엄격한-정규형">6-1. BCNF — 3NF보다 한 단계 더 엄격한 정규형</h3>
<p>3NF까지는 실무에서 자주 다루지만, BCNF는 조금 더 엄격한 조건을 요구합니다.</p>
<blockquote>
<p><strong>BCNF 조건</strong>: 모든 결정자(다른 컬럼값을 결정짓는 컬럼)가 반드시 후보키여야 한다.</p>
</blockquote>
<p>예시: <code>학습지원(교수, 과목, 강의실)</code> 테이블에서 &quot;교수는 항상 같은 강의실만 쓴다&quot;는 규칙이 있다면 → <code>교수 → 강의실</code>이라는 함수 종속이 존재하는데, <code>교수</code>는 후보키가 아니므로 BCNF 위반입니다. 이 경우 <code>교수강의실(교수, 강의실)</code>과 <code>강의배정(교수, 과목)</code>으로 테이블을 분리해야 합니다.</p>
<blockquote>
<p> 실무에서는 대부분 <strong>3NF까지만 적용</strong>합니다. BCNF, 4NF, 5NF는 JOIN 비용과 관리 복잡도를 고려해 선택적으로 적용합니다.</p>
</blockquote>
<h3 id="6-2-정규화-vs-반정규화--언제-무엇을-선택할까">6-2. 정규화 vs 반정규화 — 언제 무엇을 선택할까</h3>
<table>
<thead>
<tr>
<th></th>
<th>정규화 유지 (OLTP)</th>
<th>반정규화 허용 (OLAP)</th>
</tr>
</thead>
<tbody><tr>
<td>중심 작업</td>
<td>쓰기(INSERT/UPDATE)</td>
<td>읽기(집계 SELECT)</td>
</tr>
<tr>
<td>우선순위</td>
<td>데이터 무결성</td>
<td>JOIN 비용 감소</td>
</tr>
<tr>
<td>적합한 예</td>
<td>은행 계좌, 주문 처리</td>
<td>대시보드, 분석 쿼리</td>
</tr>
<tr>
<td>성능 문제 해결법</td>
<td>인덱스로 해결</td>
<td>중복 컬럼, Materialized View</td>
</tr>
</tbody></table>
<h3 id="6-3-scd-slowly-changing-dimension--이력-관리">6-3. SCD (Slowly Changing Dimension) — 이력 관리</h3>
<p>고객의 도시가 &quot;서울 → 부산&quot;으로 바뀔 때, 예전 정보를 남길지 말지 결정하는 전략입니다.</p>
<pre><code class="language-sql">-- Type 1: 그냥 덮어쓰기 (이력 없음)
UPDATE customers SET city = '부산' WHERE id = 42;
-- 이전 도시 정보는 사라짐

-- Type 2: 이력 보존 (기존 행을 종료시키고 새 행 추가)
UPDATE customer_history
SET valid_to = CURRENT_DATE - 1, is_current = FALSE
WHERE customer_id = 42 AND is_current = TRUE;

INSERT INTO customer_history (customer_id, name, city, valid_from, is_current)
VALUES (42, '홍길동', '부산', CURRENT_DATE, TRUE);</code></pre>
<h3 id="6-4-샤딩sharding--여러-대의-db로-분산">6-4. 샤딩(Sharding) — 여러 대의 DB로 분산</h3>
<table>
<thead>
<tr>
<th></th>
<th>파티셔닝</th>
<th>샤딩</th>
</tr>
</thead>
<tbody><tr>
<td>범위</td>
<td>단일 DB 인스턴스 내 논리/물리 분할</td>
<td>여러 DB 인스턴스에 물리적으로 분산</td>
</tr>
<tr>
<td>JOIN</td>
<td>가능</td>
<td>Cross-shard JOIN은 어려움(앱에서 처리)</td>
</tr>
<tr>
<td>목적</td>
<td>관리 편의, 부분 스캔</td>
<td>진짜 수평 확장(Scale-out)</td>
</tr>
</tbody></table>
<p>대표 전략: <strong>Hash 샤딩</strong>(<code>user_id % N</code>으로 균등 분산), <strong>Range 샤딩</strong>(ID 범위별 분산)</p>
<h3 id="6-5-msa-환경의-db-패턴">6-5. MSA 환경의 DB 패턴</h3>
<p>마이크로서비스 구조에서는 서비스마다 자기만의 DB를 갖는 게 원칙(<code>Database per Service</code>)입니다. 문제는 &quot;서비스 A의 DB와 서비스 B의 DB를 하나의 트랜잭션으로 묶을 수 없다&quot;는 점인데, 이걸 해결하는 대표 패턴들입니다.</p>
<ul>
<li><strong>Saga 패턴</strong>: 분산 트랜잭션을 &quot;로컬 트랜잭션 여러 개의 체인&quot;으로 처리하고, 실패하면 보상 트랜잭션(취소 처리)을 실행</li>
<li><strong>CQRS</strong>: 쓰기(Command)와 읽기(Query) 모델을 분리 — 쓰기는 정규화된 RDBMS, 읽기는 비정규화된 캐시나 읽기 전용 복제본 사용</li>
<li><strong>Outbox 패턴</strong>: 메인 로직과 &quot;이벤트 발행&quot;을 같은 트랜잭션 안에서 처리해, 이벤트 유실을 방지</li>
</ul>
<h3 id="6단원-핵심-정리">6단원 핵심 정리</h3>
<ul>
<li>실무에선 3NF까지가 일반적, BCNF/4NF/5NF는 필요할 때만</li>
<li>OLTP는 정규화(무결성), OLAP은 반정규화(속도)가 기본 방향</li>
<li>샤딩은 진짜 여러 대의 서버로 나누는 것 — JOIN이 어려워진다는 트레이드오프를 기억</li>
<li>MSA에서 여러 서비스 DB를 넘나드는 트랜잭션엔 Saga/Outbox 패턴을 사용</li>
</ul>
<hr />
<h2 id="7-운영-실무--백업-vacuum-connection-pool">7. 운영 실무 — 백업, VACUUM, Connection Pool</h2>
<h3 id="7-1-백업-원칙--3-2-1-법칙">7-1. 백업 원칙 — 3-2-1 법칙</h3>
<ul>
<li><strong>3 copies</strong>: 원본 포함 3벌</li>
<li><strong>2 media</strong>: 서로 다른 저장매체 2종류</li>
<li><strong>1 offsite</strong>: 최소 1벌은 물리적으로 다른 장소에</li>
</ul>
<pre><code class="language-sql">-- 논리 백업 (SQL 형태로 저장)
pg_dump -U postgres -d mydb -Fc -f backup.dump   -- 압축 바이너리(권장)

-- 복구
pg_restore -U postgres -d mydb -j 4 backup.dump  -- 병렬 복구</code></pre>
<blockquote>
<p> 백업은 &quot;떴다&quot;가 끝이 아닙니다. <strong>정기적으로 실제 복원 테스트</strong>를 해봐야 진짜 안전한 백업입니다.</p>
</blockquote>
<h3 id="7-2-vacuum--죽은-데이터-청소">7-2. VACUUM — 죽은 데이터 청소</h3>
<p>PostgreSQL은 UPDATE할 때 기존 행을 지우지 않고 &quot;Dead Tuple(죽은 버전)&quot;로 표시만 해둡니다(MVCC 특성 때문). 이게 계속 쌓이면 테이블이 실제보다 부풀어 오르는 <strong>Table Bloat</strong> 현상이 생기고 조회 속도가 느려집니다. <code>VACUUM</code>이 이 죽은 데이터를 정리해줍니다.</p>
<pre><code class="language-sql">VACUUM orders;          -- 죽은 데이터 정리 (공간은 반환 안 됨, 잠금 없음)
VACUUM ANALYZE orders;  -- 정리 + 통계 갱신 동시에
VACUUM FULL orders;     -- 테이블 완전 재작성 (공간 실제 반환, 배타적 잠금 주의)</code></pre>
<p>보통은 <code>autovacuum</code>이 자동으로 처리해주지만, 테이블별로 임계값을 조정할 수도 있습니다.</p>
<h3 id="7-3-connection-pool--pgbouncer">7-3. Connection Pool — pgBouncer</h3>
<p>DB 연결 하나당 약 5~10MB의 메모리가 듭니다. 앱 서버가 많아지고 동시 접속자가 늘면 연결 수만으로 DB가 감당 못 하는 상황이 옵니다. 이때 <strong>연결 풀링 도구(pgBouncer)</strong>를 앞단에 두면, 적은 수의 실제 DB 연결로 훨씬 많은 앱 연결을 처리할 수 있습니다.</p>
<pre><code class="language-ini">[pgbouncer]
pool_mode = transaction     -- 트랜잭션 단위로 연결을 재사용 (가장 효율적)
max_client_conn = 2000      -- 앱에서 받을 수 있는 최대 연결 수
default_pool_size = 20      -- 실제 DB에는 20개 연결만 유지</code></pre>
<p>→ <strong>20개의 실제 DB 연결로 2,000개의 앱 연결을 처리</strong>할 수 있게 됩니다.</p>
<h3 id="7-4-db-성능-최적화-어디부터-손대야-할까--4단계-접근법">7-4. DB 성능 최적화, 어디부터 손대야 할까? — 4단계 접근법</h3>
<table>
<thead>
<tr>
<th>단계</th>
<th>내용</th>
<th>영향력</th>
</tr>
</thead>
<tbody><tr>
<td>1. 설계 단계</td>
<td>정규화 수준, PK 선택, FK 인덱스, 데이터 타입</td>
<td><strong>가장 큼</strong></td>
</tr>
<tr>
<td>2. 쿼리 최적화</td>
<td>SELECT * 제거, 서브쿼리→JOIN, N+1 해결</td>
<td>큼</td>
</tr>
<tr>
<td>3. 인덱스 최적화</td>
<td>선택도 높은 컬럼, 커버링 인덱스, 미사용 인덱스 제거</td>
<td>중간</td>
</tr>
<tr>
<td>4. DB 설정 최적화</td>
<td><code>shared_buffers</code>, <code>work_mem</code>, Connection Pool</td>
<td>상대적으로 작음</td>
</tr>
</tbody></table>
<blockquote>
<p> 많은 사람들이 4단계(서버 설정)부터 손대려 하지만, <strong>가장 효과가 큰 건 1단계(설계)</strong>입니다. 설계가 잘못되면 아무리 서버 스펙을 올려도 한계가 있습니다.</p>
</blockquote>
<h3 id="7단원-핵심-정리">7단원 핵심 정리</h3>
<ul>
<li>백업은 3-2-1 법칙 + 정기 복원 테스트</li>
<li>VACUUM은 MVCC가 남긴 죽은 데이터를 청소하는 필수 유지보수</li>
<li>연결이 많아지면 Connection Pool(pgBouncer)로 완충</li>
<li>최적화는 설계 → 쿼리 → 인덱스 → 설정 순으로 접근하는 게 효율적</li>
</ul>
<hr />
<h2 id="마무리">마무리</h2>
<p> &quot;쿼리를 어떻게 빠르게 만들 것인가&quot;를 인덱스, 실행계획, 튜닝, 트랜잭션, Lock, 고급 설계까지 쭉 훑어봤습니다. 한 번에 다 외우려 하기보다는:</p>
<ol>
<li>쿼리가 느리면 → <strong>EXPLAIN ANALYZE</strong>부터 찍어본다</li>
<li><code>Seq Scan</code>이 보이면 → <strong>인덱스</strong>를 의심한다</li>
<li>동시성 문제가 생기면 → <strong>격리 수준과 Lock</strong>을 점검한다</li>
</ol>