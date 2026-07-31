<h2 id="목차">목차</h2>
<ol>
<li>Stored Procedure &amp; 함수</li>
<li>Trigger &amp; 이벤트 처리</li>
<li>Cloud DB 개요</li>
<li>서버리스 &amp; 분산 Cloud DB</li>
<li>데이터 웨어하우스 &amp; 분석 DB</li>
<li>현재의 트렌드 (NewSQL·Vector DB·AI)</li>
<li>보안 및 권한 관리</li>
<li>백업·복구 &amp; 고가용성</li>
<li>모니터링 및 운영</li>
</ol>
<hr />
<h2 id="1-stored-procedure--함수">1. Stored Procedure &amp; 함수</h2>
<h3 id="1-1-이-둘의-차이가-뭘까">1-1. 이 둘의 차이가 뭘까?</h3>
<p>DB 안에 저장해두고 실행하는 코드라는 점은 같지만, 목적이 다릅니다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>Stored Procedure</th>
<th>함수(Function, UDF)</th>
</tr>
</thead>
<tbody><tr>
<td>역할</td>
<td>여러 SQL문을 묶어 한 번에 실행(작업 절차)</td>
<td>계산해서 결과값을 돌려줌</td>
</tr>
<tr>
<td>리턴값</td>
<td>있어도 되고 없어도 됨</td>
<td>반드시 1개 이상 반환</td>
</tr>
<tr>
<td>호출 방법</td>
<td><code>CALL</code> / <code>EXEC</code></td>
<td><code>SELECT fn_함수()</code></td>
</tr>
<tr>
<td>트랜잭션 제어</td>
<td>가능 (<code>BEGIN</code>/<code>COMMIT</code>/<code>ROLLBACK</code>)</td>
<td>불가능</td>
</tr>
<tr>
<td>비유</td>
<td><strong>작업 단위</strong>(배치 작업)</td>
<td><strong>계산 도구</strong>(연산기)</td>
</tr>
</tbody></table>
<h3 id="1-2-함수-예시--부가세-10-계산-postgresql">1-2. 함수 예시 — 부가세 10% 계산 (PostgreSQL)</h3>
<pre><code class="language-sql">CREATE OR REPLACE FUNCTION fn_vat(amount NUMERIC)
RETURNS NUMERIC
LANGUAGE sql
IMMUTABLE   -- 같은 입력이면 항상 같은 결과 → 인덱스 표현식에도 쓸 수 있음
AS $$ SELECT amount * 0.1 $$;

SELECT order_id, fn_vat(total_amount) AS vat FROM orders;

-- IMMUTABLE 함수는 함수 기반 인덱스도 만들 수 있음
CREATE INDEX idx_tax ON orders ((fn_vat(total_amount)));</code></pre>
<blockquote>
<p>💡 PostgreSQL은 함수의 &quot;안정성&quot;을 3단계로 구분합니다.</p>
<ul>
<li><strong>IMMUTABLE</strong>: 입력이 같으면 항상 결과가 같음 (예: 세금 계산) → 인덱스·상수 폴딩 가능</li>
<li><strong>STABLE</strong>: 같은 쿼리 안에서는 안 변함 (예: <code>now()</code>가 아닌 <code>CURRENT_DATE</code> 등)</li>
<li><strong>VOLATILE</strong>(기본값): 매번 달라질 수 있음 (예: <code>random()</code>, <code>now()</code>)</li>
</ul>
</blockquote>
<h3 id="1-3-stored-procedure-예시--연도별-매출-상위-5명-postgresql">1-3. Stored Procedure 예시 — 연도별 매출 상위 5명 (PostgreSQL)</h3>
<pre><code class="language-sql">CREATE OR REPLACE PROCEDURE sp_top_customers(p_year INT)
LANGUAGE plpgsql AS $$
BEGIN
    SELECT c.customer_id, SUM(o.total_amount)
    FROM orders o
    JOIN customers c ON c.customer_id = o.customer_id
    WHERE EXTRACT(YEAR FROM o.order_date) = p_year
    GROUP BY c.customer_id
    ORDER BY 2 DESC
    LIMIT 5;
END $$;

CALL sp_top_customers(2024);</code></pre>
<h3 id="1-4-실무-패턴--재고-부족-시-자동-롤백">1-4. 실무 패턴 — 재고 부족 시 자동 롤백</h3>
<pre><code class="language-sql">CREATE OR REPLACE PROCEDURE sp_create_order(
    p_customer_id BIGINT,
    p_items JSONB
) LANGUAGE plpgsql AS $$
DECLARE
    v_order_id BIGINT;
    v_item JSONB;
    v_stock INT;
BEGIN
    -- 1. 재고 확인 → 부족하면 예외 발생 → 전체 ROLLBACK
    -- 2. 주문 헤더 INSERT
    -- 3. 주문 아이템 INSERT
    -- 4. 재고 UPDATE
    -- 모든 단계 성공 시에만 COMMIT
    ...
EXCEPTION WHEN OTHERS THEN
    RAISE NOTICE '주문 실패: %', SQLERRM;
    -- 자동 ROLLBACK
END $$;</code></pre>
<blockquote>
<p>💡 <strong>핵심</strong>: 여러 단계(재고 확인 → 주문 생성 → 재고 차감)를 하나의 트랜잭션으로 묶어서, 중간에 실패하면 전부 취소(ROLLBACK)되게 만드는 것이 Stored Procedure의 가장 큰 장점입니다.</p>
</blockquote>
<h3 id="1-5-stored-procedure를-db에-둘까-앱에-둘까">1-5. Stored Procedure를 DB에 둘까, 앱에 둘까?</h3>
<table>
<thead>
<tr>
<th>DB에 두면 좋은 것</th>
<th>앱에 두면 좋은 것</th>
</tr>
</thead>
<tbody><tr>
<td>공통 계산 로직</td>
<td>자주 바뀌는 비즈니스 로직</td>
</tr>
<tr>
<td>트리거로 자동 호출되는 작업</td>
<td>외부 API 연동</td>
</tr>
<tr>
<td>배치성 대량 작업</td>
<td>복잡한 조건 분기</td>
</tr>
</tbody></table>
<h3 id="✅-1단원-핵심-정리">✅ 1단원 핵심 정리</h3>
<ul>
<li><strong>함수</strong> = 계산기(값 반환), <strong>Procedure</strong> = 작업 절차(트랜잭션 가능)</li>
<li>IMMUTABLE 함수는 인덱스에도 활용 가능</li>
<li>여러 단계를 하나의 원자적 작업으로 묶어야 할 때(재고 차감+주문 생성 등) Procedure가 유리</li>
<li>자주 바뀌는 로직은 앱에, 공통·정형화된 계산은 DB에</li>
</ul>
<hr />
<h2 id="2-trigger--이벤트-처리">2. Trigger &amp; 이벤트 처리</h2>
<h3 id="2-1-trigger란">2-1. Trigger란?</h3>
<p>특정 이벤트(INSERT/UPDATE/DELETE 등)가 발생하면 <strong>DB가 자동으로 실행하는 코드</strong>입니다. &quot;이 테이블에 뭔가 바뀌면 무조건 이 코드도 같이 실행해줘&quot;라고 미리 등록해두는 것이죠.</p>
<h3 id="2-2-trigger가-잘-맞는-경우-vs-피해야-할-경우">2-2. Trigger가 잘 맞는 경우 vs 피해야 할 경우</h3>
<table>
<thead>
<tr>
<th>잘 맞는 경우</th>
<th>피해야 할 경우</th>
</tr>
</thead>
<tbody><tr>
<td>감사 로그(누가 언제 무엇을 바꿨는지 자동 기록)</td>
<td>외부 API 호출 (트랜잭션이 길어지고 장애가 전파됨)</td>
</tr>
<tr>
<td>기본값/유효성 보강 (<code>created_at</code> 자동 세팅)</td>
<td>트리거끼리 서로 참조 (유지보수 지옥)</td>
</tr>
<tr>
<td>비즈니스 규칙 보강 (재고 음수 방지)</td>
<td>대량 DML에 행 단위 트리거 남발 (statement 트리거로 대체)</td>
</tr>
</tbody></table>
<h3 id="2-3-감사-로그-트리거-예시-postgresql-row-level">2-3. 감사 로그 트리거 예시 (PostgreSQL, Row-level)</h3>
<pre><code class="language-sql">-- 감사 로그 테이블
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name TEXT,
    op TEXT,
    row_json JSONB,
    at TIMESTAMPTZ DEFAULT now()
);

-- 트리거 함수: INSERT 발생 시 변경된 행을 JSON으로 기록
CREATE OR REPLACE FUNCTION trg_sales_ai() RETURNS trigger AS $$
BEGIN
    INSERT INTO audit_log(table_name, op, row_json)
    VALUES ('sales', TG_OP, to_jsonb(NEW));
    RETURN NEW;
END $$ LANGUAGE plpgsql;

CREATE TRIGGER sales_ai
AFTER INSERT ON sales
FOR EACH ROW EXECUTE FUNCTION trg_sales_ai();</code></pre>
<blockquote>
<p>💡 <code>TG_OP</code>는 트리거가 어떤 이벤트(<code>INSERT</code>/<code>UPDATE</code>/<code>DELETE</code>)로 실행됐는지 알려주는 예약어입니다.</p>
</blockquote>
<h3 id="2-4-row-level-vs-statement-level">2-4. Row-level vs Statement-level</h3>
<table>
<thead>
<tr>
<th></th>
<th>Row-level</th>
<th>Statement-level</th>
</tr>
</thead>
<tbody><tr>
<td>실행 단위</td>
<td>변경된 행마다 1번씩</td>
<td>문장 전체에 대해 1번만</td>
</tr>
<tr>
<td>대량 INSERT 성능</td>
<td>느림 (1만 건이면 1만 번 실행)</td>
<td>빠름 (1번만 실행)</td>
</tr>
<tr>
<td>PostgreSQL 지원</td>
<td>✅</td>
<td>✅ (Transition Table로 변경분 전체 참조 가능)</td>
</tr>
<tr>
<td>MySQL 지원</td>
<td>✅</td>
<td>❌ (Row-level만 지원)</td>
</tr>
</tbody></table>
<pre><code class="language-sql">-- Statement-level + Transition Table (대량 INSERT에 효율적)
CREATE OR REPLACE FUNCTION trg_sales_stmt_ai() RETURNS trigger AS $$
BEGIN
    INSERT INTO audit_log(table_name, op, row_json)
    SELECT 'sales', TG_OP, to_jsonb(n) FROM new_table AS n;
    RETURN NULL;
END $$ LANGUAGE plpgsql;

CREATE TRIGGER sales_stmt_ai AFTER INSERT ON sales
REFERENCING NEW TABLE AS new_table
FOR EACH STATEMENT EXECUTE FUNCTION trg_sales_stmt_ai();</code></pre>
<h3 id="2-5-실시간-알림--pg_notify">2-5. 실시간 알림 — pg_notify</h3>
<p>트리거가 단순히 로그만 남기는 게 아니라, <strong>앱에 실시간으로 신호를 보낼 수도 있습니다.</strong></p>
<pre><code class="language-sql">CREATE OR REPLACE FUNCTION fn_notify_order() RETURNS trigger AS $$
BEGIN
    PERFORM pg_notify('order_channel', row_to_json(NEW)::TEXT);
    RETURN NEW;
END $$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_notify AFTER INSERT ON orders
FOR EACH ROW EXECUTE FUNCTION fn_notify_order();

-- 앱(Python)에서 수신
-- cur.execute(&quot;LISTEN order_channel&quot;)
-- 이벤트가 오면 conn.notifies에 쌓임 → 실시간 대시보드/웹소켓 알림에 활용</code></pre>
<h3 id="2-6-trigger-vs-application--언제-뭘-쓸까">2-6. Trigger vs Application — 언제 뭘 쓸까</h3>
<ul>
<li><strong>Trigger</strong>: 어떤 경로로 데이터가 바뀌든 반드시 실행돼야 하는 것 (감사, 무결성 규칙)</li>
<li><strong>Application</strong>: 비즈니스 로직, 외부 시스템 호출, 복잡한 흐름 제어</li>
<li>원칙: <strong>트리거는 짧고 결정적으로</strong>, 외부 의존성 없이 작성</li>
</ul>
<h3 id="✅-2단원-핵심-정리">✅ 2단원 핵심 정리</h3>
<ul>
<li>Trigger = &quot;이 테이블 바뀌면 이 코드도 자동 실행&quot;</li>
<li>감사 로그·기본값 세팅엔 좋지만, 외부 API 호출은 절대 금지</li>
<li>대량 데이터 변경엔 Row-level보다 Statement-level(Transition Table)이 효율적</li>
<li><code>pg_notify</code>로 실시간 알림까지 트리거에서 처리 가능</li>
</ul>
<hr />
<h2 id="3-cloud-db-개요">3. Cloud DB 개요</h2>
<h3 id="3-1-on-prem-vs-dbaas-vs-cloud-native--뭐가-다를까">3-1. On-Prem vs DBaaS vs Cloud-Native — 뭐가 다를까</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>On-Prem(직접 구축)</th>
<th>DBaaS(RDS·Cloud SQL)</th>
<th>Cloud-Native(Aurora·Spanner)</th>
</tr>
</thead>
<tbody><tr>
<td>프로비저닝</td>
<td>서버 구매·설치 (느림)</td>
<td>콘솔에서 몇 분 만에 생성</td>
<td>스토리지·복제 구조 자체가 분산 전용 설계</td>
</tr>
<tr>
<td>확장성</td>
<td>수직 확장만(교체 필요)</td>
<td>수직+수평(읽기 복제본)</td>
<td>설계부터 분산, 자동 복구</td>
</tr>
<tr>
<td>비용</td>
<td>CAPEX(선투자)+OPEX</td>
<td>OPEX(사용량 기반)</td>
<td>OPEX + 전용 기능 비용</td>
</tr>
<tr>
<td>운영</td>
<td>DBA가 직접 다 함</td>
<td>백업·패치 자동화</td>
<td>더 높은 자동화</td>
</tr>
</tbody></table>
<blockquote>
<p>💡 <strong>DBaaS를 쓴다고 우리가 아무것도 안 해도 되는 건 아닙니다.</strong> 하드웨어 관리는 클라우드가 해주지만, <strong>스키마 설계·인덱스·쿼리 최적화·보안 정책</strong>은 여전히 우리 몫입니다.</p>
</blockquote>
<h3 id="3-2-aws-rds-vs-aurora--언제-뭘-고를까">3-2. AWS RDS vs Aurora — 언제 뭘 고를까</h3>
<table>
<thead>
<tr>
<th>항목</th>
<th>RDS</th>
<th>Aurora</th>
</tr>
</thead>
<tbody><tr>
<td>아키텍처</td>
<td>엔진 원본을 그대로 관리형으로 운영</td>
<td>Compute/스토리지 분리, 3AZ 6중 복제</td>
</tr>
<tr>
<td>스토리지</td>
<td>EBS 기반</td>
<td>10GB 단위 자동 확장, 최대 128TB</td>
</tr>
<tr>
<td>선택 기준</td>
<td>엔진 호환성 최우선, 중소규모</td>
<td>읽기 부하가 크고 빠른 장애 조치 필요 시</td>
</tr>
</tbody></table>
<h3 id="3-3-cloud-db-선택-프레임워크-4단계">3-3. Cloud DB 선택 프레임워크 (4단계)</h3>
<ol>
<li><strong>규제/컴플라이언스 확인</strong>: 데이터를 특정 국가/리전 밖으로 못 내보내는 규제가 있다면 → On-Prem 또는 특정 리전 클라우드</li>
<li><strong>팀 역량 확인</strong>: DBA 전문가가 없고 DevOps 인력이 적다면 → DBaaS로 운영 부담을 최소화</li>
<li><strong>워크로드 특성</strong>: 트래픽이 예측 가능하면 고정 인스턴스, 급변한다면 서버리스(Aurora Serverless v2 등)</li>
<li><strong>비용 총합(TCO) 계산</strong>: 클라우드는 Egress(데이터 반출)·IOPS 비용을 놓치기 쉬우니 꼭 포함해서 계산</li>
</ol>
<h3 id="✅-3단원-핵심-정리">✅ 3단원 핵심 정리</h3>
<ul>
<li>DBaaS = &quot;하드웨어는 맡기지만, 설계·튜닝·보안은 여전히 내 책임&quot;</li>
<li>Aurora는 읽기 확장·고가용성이 강점, RDS는 엔진 호환성과 익숙함이 강점</li>
<li>Cloud DB 선택은 규제 → 팀 역량 → 워크로드 → 비용 순으로 검토</li>
</ul>
<hr />
<h2 id="4-서버리스--분산-cloud-db">4. 서버리스 &amp; 분산 Cloud DB</h2>
<h3 id="4-1-요즘-뜨는-서버리스분산-db-4종">4-1. 요즘 뜨는 서버리스/분산 DB 4종</h3>
<table>
<thead>
<tr>
<th>서비스</th>
<th>한 줄 특징</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Aurora Serverless v2</strong></td>
<td>트래픽에 따라 자동으로 컴퓨팅 용량(ACU)이 늘고 줄어듦. 간헐적 트래픽에 적합</td>
</tr>
<tr>
<td><strong>Google Cloud Spanner</strong></td>
<td>전 세계 여러 리전에 걸쳐 강한 일관성(Strict Serializability)을 보장하는 진짜 분산 SQL</td>
</tr>
<tr>
<td><strong>Azure Cosmos DB</strong></td>
<td>문서·키값·그래프 등 다중 모델 지원, 일관성 수준을 5단계로 선택 가능</td>
</tr>
<tr>
<td><strong>Neon / Supabase</strong></td>
<td>컴퓨팅과 스토리지를 분리해 <strong>scale-to-zero</strong>(안 쓰면 0으로 줄어듦) 지원, 개발자 친화적</td>
</tr>
</tbody></table>
<h3 id="4-2-전통-rdbms-vs-서버리스분산-db">4-2. 전통 RDBMS vs 서버리스/분산 DB</h3>
<table>
<thead>
<tr>
<th></th>
<th>전통 RDBMS</th>
<th>서버리스/분산</th>
</tr>
</thead>
<tbody><tr>
<td>용량 계획</td>
<td>인스턴스 크기를 수동으로 정함</td>
<td>자동 스케일 (안 쓰면 0까지)</td>
</tr>
<tr>
<td>일관성 모델</td>
<td>단일 노드 강한 일관성</td>
<td>선택형(Cosmos) 또는 글로벌 강한 일관성(Spanner)</td>
</tr>
<tr>
<td>스키마 변경</td>
<td>Lock, 다운타임 위험</td>
<td>Online DDL, 브랜칭(Neon처럼 Git처럼 DB 분기)</td>
</tr>
<tr>
<td>비용</td>
<td>고정(유휴 비용도 계속 나감)</td>
<td>사용량 기반, 유휴 시 절감</td>
</tr>
</tbody></table>
<h3 id="4-3-msa-환경의-db-패턴">4-3. MSA 환경의 DB 패턴</h3>
<p>마이크로서비스에서는 서비스마다 자기 DB를 갖는 게 원칙(Database per Service)입니다. 문제는 <strong>여러 서비스의 DB를 하나의 트랜잭션으로 묶을 수 없다</strong>는 점인데, 이를 해결하는 패턴들이 있습니다.</p>
<table>
<thead>
<tr>
<th>패턴</th>
<th>설명</th>
<th>주의사항</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Saga (Choreography)</strong></td>
<td>각 서비스가 이벤트를 주고받으며 자율적으로 조정</td>
<td>분산 트랜잭션 복잡, 보상 트랜잭션 필요</td>
</tr>
<tr>
<td><strong>Saga (Orchestration)</strong></td>
<td>중앙 코디네이터가 전체 흐름을 제어</td>
<td>코디네이터가 단일 장애점이 될 수 있음</td>
</tr>
<tr>
<td><strong>CQRS</strong></td>
<td>읽기와 쓰기 모델을 분리</td>
<td>이벤트 발생 시 일시적 불일치 허용 필요</td>
</tr>
<tr>
<td><strong>Outbox Pattern</strong></td>
<td>로컬 트랜잭션 안에서 &quot;이벤트 발행&quot;까지 같이 기록</td>
<td>소비자 쪽에서 중복 처리 방지(멱등성) 필요</td>
</tr>
</tbody></table>
<h3 id="✅-4단원-핵심-정리">✅ 4단원 핵심 정리</h3>
<ul>
<li>서버리스 DB = &quot;안 쓰면 비용도 0에 가깝게&quot; (Neon의 scale-to-zero가 대표적)</li>
<li>Spanner는 진짜 여러 대륙에 걸쳐 강한 일관성을 보장하는 특수한 존재</li>
<li>MSA에서 여러 서비스 DB를 넘나드는 트랜잭션엔 Saga/Outbox 패턴이 정답</li>
</ul>
<hr />
<h2 id="5-데이터-웨어하우스--분석-db">5. 데이터 웨어하우스 &amp; 분석 DB</h2>
<h3 id="5-1-oltp-vs-olap--완전히-다른-목적">5-1. OLTP vs OLAP — 완전히 다른 목적</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>OLTP(운영 DB)</th>
<th>OLAP(분석 DB/DWH)</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>트랜잭션 처리(주문, 결제)</td>
<td>다차원 분석·집계(매출 리포트)</td>
</tr>
<tr>
<td>워크로드</td>
<td>잦은 단건 읽기/쓰기, 낮은 지연</td>
<td>대량 스캔·집계, 높은 처리량</td>
</tr>
<tr>
<td>스키마</td>
<td>정규화(3NF)</td>
<td>반정규화(스타/스노우플레이크)</td>
</tr>
<tr>
<td>저장 형식</td>
<td>행 지향(Row store)</td>
<td><strong>컬럼 지향(Columnar)</strong></td>
</tr>
</tbody></table>
<blockquote>
<p>💡 왜 분석 DB는 컬럼 지향일까요? &quot;월별 총 매출&quot;을 구하려면 <code>amount</code> 컬럼만 읽으면 되는데, 행 지향 저장이면 다른 컬럼(주소, 이름 등)까지 같이 읽게 됩니다. 컬럼 지향은 필요한 컬럼만 통째로 읽을 수 있어 집계가 훨씬 빠릅니다.</p>
</blockquote>
<h3 id="5-2-분석-db의-핵심-기술-3가지">5-2. 분석 DB의 핵심 기술 3가지</h3>
<ul>
<li><strong>파티션 프루닝(Partition Pruning)</strong>: <code>WHERE event_time &gt;= '2024-01-01'</code> 조건이 있으면, 2024년 파티션만 읽고 나머지는 통째로 무시</li>
<li><strong>컬럼 프루닝(Column Pruning)</strong>: <code>SELECT amount</code>만 쓰면 다른 컬럼은 디스크에서 아예 읽지 않음 → <strong><code>SELECT *</code>은 분석 DB에서 특히 치명적</strong></li>
<li><strong>벡터화 실행(Vectorized Execution)</strong>: 행 하나씩이 아니라 1000개씩 묶어서 CPU가 한 번에 처리 (BigQuery/Snowflake/ClickHouse 모두 채택)</li>
</ul>
<h3 id="5-3-스타-스키마--분석-db의-기본-설계">5-3. 스타 스키마 — 분석 DB의 기본 설계</h3>
<ul>
<li><strong>팩트 테이블(Fact Table)</strong>: 거래·이벤트 자체 (매출액, 수량 등) — 행이 아주 많고 계속 쌓임</li>
<li><strong>디멘전 테이블(Dimension Table)</strong>: 사용자/상품/시간/지역처럼 &quot;기준이 되는 정보&quot; — 비교적 작음</li>
</ul>
<pre><code class="language-sql">-- BigQuery: 파티션 + 클러스터 설계
CREATE TABLE mart.fact_sales
PARTITION BY DATE(event_ts)          -- 날짜별 파티션 (스캔 비용 절감)
CLUSTER BY user_id, item_id AS       -- 클러스터: 정렬 기준
SELECT event_ts, user_id, item_id, qty, amount, channel
FROM stg.sales_cleaned;

-- 파티션 프루닝 확인: Q1 파티션만 스캔됨
SELECT SUM(amount) FROM mart.fact_sales
WHERE event_ts &gt;= '2024-01-01' AND event_ts &lt; '2024-04-01';</code></pre>
<h3 id="5-4-분석-db-제품-선택-가이드">5-4. 분석 DB 제품 선택 가이드</h3>
<table>
<thead>
<tr>
<th>요구사항</th>
<th>추천 제품</th>
<th>이유</th>
</tr>
</thead>
<tbody><tr>
<td>완전 서버리스 + GCP 중심</td>
<td>BigQuery</td>
<td>스캔 바이트 과금, 운영 최소</td>
</tr>
<tr>
<td>AWS 생태계 + S3 레이크</td>
<td>Redshift</td>
<td>AWS 밀착, Dist/Sort Key 최적화</td>
</tr>
<tr>
<td>MS PowerBI + Spark 통합</td>
<td>Azure Synapse</td>
<td>MS 생태계, Fabric 연동</td>
</tr>
<tr>
<td>멀티 클라우드 + 운영 단순</td>
<td>Snowflake</td>
<td>JSON, 자동 튜닝</td>
</tr>
<tr>
<td>초저지연 실시간 집계</td>
<td>ClickHouse</td>
<td>이벤트·로그·시계열, 비용 효율</td>
</tr>
</tbody></table>
<h3 id="✅-5단원-핵심-정리">✅ 5단원 핵심 정리</h3>
<ul>
<li>OLTP는 정규화+행지향(쓰기 최적), OLAP은 반정규화+컬럼지향(읽기 최적)</li>
<li>분석 DB에서 <code>SELECT *</code>는 최악의 습관 — 필요한 컬럼만 명시할 것</li>
<li>파티션 프루닝은 &quot;필요한 날짜 폴더만 열어보는 것&quot;이라고 생각하면 이해하기 쉬움</li>
</ul>
<hr />
<h2 id="6-현재의-트렌드--newsql-·-vector-db-·-aidb">6. 현재의 트렌드 — NewSQL · Vector DB · AI+DB</h2>
<h3 id="6-1-newsql--sql과-트랜잭션은-유지하며-수평-확장까지">6-1. NewSQL — SQL과 트랜잭션은 유지하며 수평 확장까지</h3>
<table>
<thead>
<tr>
<th></th>
<th>전통 RDBMS</th>
<th>NoSQL</th>
<th>NewSQL</th>
</tr>
</thead>
<tbody><tr>
<td>ACID+SQL</td>
<td>강함</td>
<td>약함</td>
<td><strong>강함</strong></td>
</tr>
<tr>
<td>수평 확장</td>
<td>어려움</td>
<td>쉬움</td>
<td><strong>쉬움</strong></td>
</tr>
</tbody></table>
<p>대표 주자: <strong>CockroachDB</strong>(모든 노드가 동등, 기본이 Serializable 격리수준), <strong>TiDB</strong>(MySQL 완전 호환 + OLTP/OLAP 동시 지원)</p>
<h3 id="6-2-시계열-db--vector-db">6-2. 시계열 DB &amp; Vector DB</h3>
<ul>
<li><strong>TimescaleDB</strong>: PostgreSQL 확장으로, 시간 기준 파티션(Hypertable)을 자동화하고 <code>Continuous Aggregate</code>로 실시간 집계 뷰까지 제공</li>
<li><strong>Vector DB(pgvector)</strong>: AI 시대의 핵심 — 문장이나 이미지를 숫자 벡터로 바꿔서 저장하고, &quot;가장 비슷한 벡터&quot;를 빠르게 찾는 DB</li>
</ul>
<h3 id="6-3-pgvector로-보는-rag검색-증강-생성-패턴">6-3. pgvector로 보는 RAG(검색 증강 생성) 패턴</h3>
<blockquote>
<p>RAG란? 사용자의 질문을 임베딩(숫자 벡터)으로 바꾼 뒤, DB에 저장된 문서 중 가장 비슷한 것을 찾아서 LLM에게 &quot;이 문서를 참고해서 답해&quot;라고 컨텍스트로 넘겨주는 방식입니다.</p>
</blockquote>
<pre><code class="language-sql">CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE knowledge_base (
    id BIGSERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    embedding VECTOR(1536)   -- OpenAI 임베딩 차원
);

-- HNSW 인덱스: 유사도 검색을 빠르게
CREATE INDEX ON knowledge_base USING HNSW (embedding vector_cosine_ops);

-- 질문 벡터($1)와 가장 비슷한 문서 20개 찾기
SELECT id, 1 - (embedding &lt;=&gt; $1::VECTOR) AS similarity
FROM knowledge_base
ORDER BY embedding &lt;=&gt; $1::VECTOR
LIMIT 20;</code></pre>
<blockquote>
<p>💡 <code>&lt;=&gt;</code> 연산자는 pgvector가 제공하는 &quot;코사인 거리&quot; 연산자입니다. 거리가 가까울수록(값이 작을수록) 두 벡터(=두 문서/문장)가 의미적으로 비슷하다는 뜻이에요.</p>
</blockquote>
<h3 id="6-4-ai-시대-db-엔지니어의-역할-변화">6-4. AI 시대, DB 엔지니어의 역할 변화</h3>
<table>
<thead>
<tr>
<th>전통 DBA</th>
<th>+ AI 시대 추가 역할</th>
</tr>
</thead>
<tbody><tr>
<td>스키마 설계, 인덱스 최적화, 백업/복구</td>
<td>Vector DB 설계(임베딩 차원, 하이브리드 검색)</td>
</tr>
<tr>
<td>실행계획 분석, Lock 관리</td>
<td>Feature Store 관리(학습/서빙 피처 일관성)</td>
</tr>
<tr>
<td></td>
<td>AI 파이프라인 DB 통합, LLM이 만든 SQL 검증</td>
</tr>
</tbody></table>
<h3 id="✅-6단원-핵심-정리">✅ 6단원 핵심 정리</h3>
<ul>
<li>NewSQL = &quot;SQL의 익숙함 + NoSQL의 확장성&quot;을 동시에 잡으려는 시도</li>
<li>Vector DB(pgvector)는 AI 서비스(RAG, 추천, 이미지 검색)의 핵심 인프라로 급부상</li>
<li>목적별 DB 선택: 트랜잭션(PostgreSQL/MySQL), 캐시(Redis), 검색(Elasticsearch), 시계열(TimescaleDB), 벡터(pgvector), 그래프(Neo4j), 분석(BigQuery)</li>
</ul>
<hr />
<h2 id="7-보안-및-권한-관리">7. 보안 및 권한 관리</h2>
<h3 id="7-1-db-보안의-5개-층">7-1. DB 보안의 5개 층</h3>
<ol>
<li><strong>인증(Authentication)</strong> — 누가 접속했는가</li>
<li><strong>인가(Authorization)</strong> — 그 사람이 뭘 할 수 있는가 (SELECT/INSERT 등)</li>
<li><strong>입력검증/파라미터 바인딩</strong> — 악성 입력으로부터 SQL 보호</li>
<li><strong>암호화</strong> — 전송(TLS)과 저장(TDE/컬럼 암호화) 모두 보호</li>
<li><strong>감사(Audit)</strong> — 누가 언제 무엇을 실행했는지 기록</li>
</ol>
<h3 id="7-2-최소-권한-원칙least-privilege">7-2. 최소 권한 원칙(Least Privilege)</h3>
<pre><code class="language-sql">-- 읽기 전용 역할 생성
CREATE ROLE app_reader NOLOGIN;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_reader;

-- 애플리케이션 계정에 역할 부여 (직접 권한을 주지 않고 역할을 통해서)
CREATE ROLE app_user LOGIN PASSWORD 'secret';
GRANT app_reader TO app_user;</code></pre>
<blockquote>
<p>💡 실무 역할 분리 예시: <code>data_engineer</code>(적재/정제), <code>data_analyst</code>(읽기 전용), <code>api_user</code>(VIEW만 접근, 테이블 직접 접근 불가), <code>etl_user</code>(스케줄러 전용)</p>
</blockquote>
<h3 id="7-3-row-level-security--멀티테넌시-데이터-격리">7-3. Row-Level Security — 멀티테넌시 데이터 격리</h3>
<p>한 테이블에 여러 고객사의 데이터가 섞여 있을 때, &quot;각 고객이 자기 데이터만 보게&quot; 만드는 기능입니다.</p>
<pre><code class="language-sql">ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY customer_policy ON orders
    FOR SELECT USING (customer_id = current_setting('app.customer_id')::int);

-- 앱이 접속 시 세션에 고객 ID를 세팅하면, 그 고객 데이터만 보임
SELECT set_config('app.customer_id', '42', true);</code></pre>
<h3 id="7-4-sql-injection--원인과-방어">7-4. SQL Injection — 원인과 방어</h3>
<pre><code class="language-python"># ❌ 절대 금지: 문자열을 이어붙여서 쿼리 생성
sql = f&quot;SELECT * FROM users WHERE name = '{user_input}'&quot;
cur.execute(sql)
# 공격 예: user_input = &quot;' OR '1'='1&quot;  →  WHERE name = '' OR '1'='1' (전체 조회됨!)

# ✅ 파라미터 바인딩 (Prepared Statement)
cur.execute(&quot;SELECT * FROM users WHERE name = %s&quot;, (user_input,))</code></pre>
<blockquote>
<p>💡 <code>ORDER BY</code>처럼 컬럼명 자체를 사용자 입력으로 받아야 할 때는 파라미터 바인딩이 안 통합니다. 이때는 <strong>화이트리스트 검증</strong>을 씁니다.</p>
</blockquote>
<pre><code class="language-python">if order not in ('name', 'created_at'):
    order = 'created_at'  # 허용되지 않은 값이면 기본값으로 대체
sql = f&quot;SELECT * FROM items ORDER BY {order}&quot;</code></pre>
<blockquote>
<p>ORM(SQLAlchemy 등)을 쓰면 기본적으로 안전하지만, <code>.execute(text(...))</code>처럼 raw SQL에 직접 문자열을 넣으면 여전히 취약합니다.</p>
</blockquote>
<h3 id="7-5-암호화--tls--tde--컬럼-암호화">7-5. 암호화 — TLS / TDE / 컬럼 암호화</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>대상</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>TLS(전송 암호화)</td>
<td>클라이언트↔DB 연결</td>
<td>항상 켜두는 게 기본</td>
</tr>
<tr>
<td>TDE(디스크 암호화)</td>
<td>DB 파일 전체</td>
<td>앱은 모르게 자동 암호화, 컬럼 단위 제어는 불가</td>
</tr>
<tr>
<td>컬럼 암호화</td>
<td>민감 컬럼만(주민번호 등)</td>
<td>검색·인덱스에 제약, 성능 저하 있음</td>
</tr>
</tbody></table>
<pre><code class="language-sql">-- PostgreSQL pgcrypto로 컬럼 암호화
CREATE EXTENSION IF NOT EXISTS pgcrypto;
INSERT INTO customers (name, ssn_enc)
VALUES ('Alice', pgp_sym_encrypt('111-22-3333', 'mypassword'));

SELECT pgp_sym_decrypt(ssn_enc, 'mypassword') FROM customers;</code></pre>
<h3 id="✅-7단원-핵심-정리">✅ 7단원 핵심 정리</h3>
<ul>
<li>인증→인가→입력검증→암호화→감사, 5개 층을 모두 갖춰야 진짜 보안</li>
<li>SQL Injection의 유일한 근본 해법은 <strong>파라미터 바인딩</strong>, 컬럼명 등은 화이트리스트 검증</li>
<li>여러 고객사 데이터가 섞인 테이블엔 Row-Level Security로 격리</li>
<li>앱에는 DB 관리자 권한을 주지 말고, 딱 필요한 역할만 부여(최소 권한 원칙)</li>
</ul>
<hr />
<h2 id="8-백업·복구--고가용성">8. 백업·복구 &amp; 고가용성</h2>
<h3 id="8-1-백업-유형-비교">8-1. 백업 유형 비교</h3>
<table>
<thead>
<tr>
<th>유형</th>
<th>설명</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>Full Backup</td>
<td>전체 데이터 스냅샷</td>
<td>가장 느리지만 가장 완전</td>
</tr>
<tr>
<td>Incremental</td>
<td>마지막 백업 이후 변경분만</td>
<td>빠름, 복구 시 Full+증분 조합 필요</td>
</tr>
<tr>
<td>PITR</td>
<td>특정 시점으로 복구</td>
<td>WAL(변경 기록) 기반, &quot;실수로 삭제한 직전&quot;으로 복구 가능</td>
</tr>
</tbody></table>
<pre><code class="language-bash"># PostgreSQL 논리 백업/복구
pg_dump -U postgres -d mydb -Fc -f backup.dump
pg_restore -U postgres -d mydb -j 4 backup.dump</code></pre>
<blockquote>
<p>💡 <strong>3-2-1 백업 법칙</strong>: 원본 포함 3벌, 서로 다른 매체 2종류, 최소 1벌은 다른 장소에. 그리고 백업은 &quot;떴다&quot;가 끝이 아니라 <strong>정기적으로 실제 복원 테스트</strong>를 해봐야 진짜 안전합니다.</p>
</blockquote>
<h3 id="8-2-streaming-replication--실시간-복제">8-2. Streaming Replication — 실시간 복제</h3>
<pre><code>[postgresql.conf]
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'</code></pre><ul>
<li><strong>Synchronous(동기)</strong>: COMMIT 시 Standby까지 응답을 기다림 → 데이터 손실 없음(RPO=0), 대신 느림</li>
<li><strong>Asynchronous(비동기, 기본값)</strong>: COMMIT 즉시 응답 → 빠르지만 아주 약간의 데이터 손실 가능성(RPO&gt;0)</li>
</ul>
<h3 id="8-3-multi-az-구조-aws-예시">8-3. Multi-AZ 구조 (AWS 예시)</h3>
<pre><code>[Primary DB (AZ-A, 읽기/쓰기)] ──(동기 복제)── [Standby DB (AZ-B, 대기)]</code></pre><p>Primary가 죽으면 AWS가 자동으로 Standby를 새 Primary로 승격시킵니다(Failover). 쓰기가 완료되려면 두 DB 모두 써야 하므로 RPO(데이터 손실)가 거의 0입니다.</p>
<h3 id="8-4-dr재해-복구-목표--rpo--rto">8-4. DR(재해 복구) 목표 — RPO / RTO</h3>
<ul>
<li><strong>RPO(Recovery Point Objective)</strong>: 최대 허용 데이터 손실 시간 (&quot;장애 5분 전 데이터까지는 살릴 수 있어야 한다&quot;)</li>
<li><strong>RTO(Recovery Time Objective)</strong>: 최대 허용 서비스 다운 시간 (&quot;장애 나도 10분 안에 복구해야 한다&quot;)</li>
</ul>
<h3 id="✅-8단원-핵심-정리">✅ 8단원 핵심 정리</h3>
<ul>
<li>백업은 Full/Incremental/PITR을 조합해서 쓰고, 3-2-1 법칙 + 복원 테스트가 핵심</li>
<li>복제는 동기(안전하지만 느림) vs 비동기(빠르지만 손실 가능) 트레이드오프</li>
<li>RPO(데이터 손실 허용치)와 RTO(다운타임 허용치)를 먼저 정해야 백업 전략이 나옴</li>
</ul>
<hr />
<h2 id="9-모니터링-및-운영">9. 모니터링 및 운영</h2>
<h3 id="9-1-모니터링--db의-건강검진">9-1. 모니터링 = DB의 건강검진</h3>
<table>
<thead>
<tr>
<th>서버 지표</th>
<th>사람에 비유하면</th>
</tr>
</thead>
<tbody><tr>
<td>CPU, 메모리</td>
<td>몸의 체력</td>
</tr>
<tr>
<td>TPS, QPS</td>
<td>심장 박동 수</td>
</tr>
<tr>
<td>Latency(지연시간)</td>
<td>혈압</td>
</tr>
<tr>
<td>로그</td>
<td>건강기록</td>
</tr>
</tbody></table>
<h3 id="9-2-기본-지표-3가지">9-2. 기본 지표 3가지</h3>
<table>
<thead>
<tr>
<th>용어</th>
<th>뜻</th>
<th>카페로 비유하면</th>
</tr>
</thead>
<tbody><tr>
<td><strong>TPS</strong>(Transactions Per Second)</td>
<td>초당 완료된 거래(트랜잭션) 수</td>
<td>초당 결제 완료 수</td>
</tr>
<tr>
<td><strong>QPS</strong>(Queries Per Second)</td>
<td>초당 DB에 던진 쿼리 수</td>
<td>초당 주문 수</td>
</tr>
<tr>
<td><strong>Latency</strong></td>
<td>요청부터 응답까지 걸린 시간</td>
<td>한 잔 나오기까지 걸리는 시간</td>
</tr>
</tbody></table>
<h3 id="9-3-모니터링-도구">9-3. 모니터링 도구</h3>
<table>
<thead>
<tr>
<th>도구</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Prometheus</td>
<td>지표 수집</td>
</tr>
<tr>
<td>Grafana</td>
<td>시각화(그래프 대시보드)</td>
</tr>
<tr>
<td>CloudWatch(AWS)</td>
<td>RDS/EC2 상태 자동 모니터링</td>
</tr>
<tr>
<td>Datadog</td>
<td>DB·서버·네트워크 통합 관리(유료 SaaS)</td>
</tr>
</tbody></table>
<h3 id="9-4-이상-징후-대응-표">9-4. 이상 징후 대응 표</h3>
<table>
<thead>
<tr>
<th>증상</th>
<th>원인 가능성</th>
<th>확인 방법</th>
</tr>
</thead>
<tbody><tr>
<td>쿼리가 갑자기 느려짐</td>
<td>인덱스 없음, 통계 오래됨</td>
<td><code>EXPLAIN ANALYZE</code>로 실행계획 확인</td>
</tr>
<tr>
<td>CPU 100%</td>
<td>느린 쿼리, 풀스캔</td>
<td>Slow Query Log 확인</td>
</tr>
</tbody></table>
<h3 id="9-5-알람alert-설정-예시">9-5. 알람(Alert) 설정 예시</h3>
<table>
<thead>
<tr>
<th>상황</th>
<th>경고 기준</th>
<th>행동</th>
</tr>
</thead>
<tbody><tr>
<td>평균 응답 시간 &gt; 500ms</td>
<td>5분 이상 지속</td>
<td>쿼리 분석 시작</td>
</tr>
<tr>
<td>Replica Lag &gt; 30초</td>
<td>10분 지속</td>
<td>복제 트래픽 점검</td>
</tr>
<tr>
<td>CPU 사용률 &gt; 80%</td>
<td>10분 지속</td>
<td>인덱스, 캐시 점검</td>
</tr>
<tr>
<td>Free Storage &lt; 10%</td>
<td>즉시</td>
<td>로그 정리, 용량 증설</td>
</tr>
</tbody></table>
<h3 id="✅-9단원-핵심-정리">✅ 9단원 핵심 정리</h3>
<ul>
<li>TPS/QPS = &quot;얼마나 바쁜가&quot;, Latency = &quot;얼마나 빠른가&quot;, 로그 = &quot;왜 느렸는가의 증거&quot;</li>
<li>On-Prem은 Prometheus+Grafana를 직접 구성, Cloud는 CloudWatch/Query Insights가 자동 제공</li>
<li>임계치를 미리 정해두고 알람을 걸어야 장애를 &quot;터지기 전에&quot; 발견할 수 있음</li>
</ul>
<hr />
<h2 id="마무리--4일간의-여정-정리">마무리 — 4일간의 여정 정리</h2>
<table>
<thead>
<tr>
<th>Day</th>
<th>핵심 질문</th>
</tr>
</thead>
<tbody><tr>
<td>Day1</td>
<td>데이터를 어떻게 <strong>설계</strong>할 것인가 (ERD, 정규화, DDL)</td>
</tr>
<tr>
<td>Day2</td>
<td>데이터를 어떻게 <strong>조회</strong>할 것인가 (JOIN, 서브쿼리, 윈도우함수)</td>
</tr>
<tr>
<td>Day3</td>
<td>조회를 어떻게 <strong>빠르게</strong> 만들 것인가 (인덱스, 실행계획, 트랜잭션)</td>
</tr>
<tr>
<td>Day4</td>
<td>DB를 어떻게 <strong>운영</strong>할 것인가 (SP/Trigger, Cloud, 보안, 백업, 모니터링)</td>
</tr>
</tbody></table>
<p>4일 동안의 흐름을 한 문장으로 요약하면: <strong>&quot;설계 잘하고 → 잘 조회하고 → 빠르게 만들고 → 안전하게 운영한다.&quot;</strong> 이 네 가지가 실무 DB 엔지니어링의 기본기입니다. 다음은 마지막 종합실습(E-Commerce 매출 분석 — RFM, Materialized View, 실행계획 튜닝)으로 4일 과정을 마무리하게 됩니다.</p>