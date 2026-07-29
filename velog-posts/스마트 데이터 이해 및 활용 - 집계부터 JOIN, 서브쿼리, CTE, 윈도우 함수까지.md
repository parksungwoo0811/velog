<ul>
<li>학과마다 학생이 몇 명인지 계산한다.</li>
<li>강좌별 평균 성적과 수강 인원을 구한다.</li>
<li>학생, 학과, 강좌, 교수 정보를 하나의 결과로 연결한다.</li>
<li>전체 평균보다 성적이 높은 학생을 찾는다.</li>
<li>복잡한 쿼리를 CTE와 View로 읽기 쉽게 만든다.</li>
<li>학과별 순위, 이전 성적과의 차이, 누적값을 계산한다.</li>
</ul>
<p>이 글은 문법을 나열하는 데서 끝나지 않고 <strong>언제 어떤 문법을 선택해야 하는지</strong>, <strong>결과가 왜 그렇게 나오는지</strong>, <strong>초보자가 자주 하는 실수는 무엇인지</strong>까지 함께 설명한다.</p>
<hr />
<h2 id="먼저-보는-학습-지도">먼저 보는 학습 지도</h2>
<table>
<thead>
<tr>
<th align="right">순서</th>
<th>핵심 질문</th>
<th>사용할 기능</th>
</tr>
</thead>
<tbody><tr>
<td align="right">1</td>
<td>여러 행을 하나의 통계로 어떻게 요약할까?</td>
<td><code>COUNT</code>, <code>SUM</code>, <code>AVG</code>, <code>MIN</code>, <code>MAX</code></td>
</tr>
<tr>
<td align="right">2</td>
<td>학과별·강좌별로 어떻게 묶을까?</td>
<td><code>GROUP BY</code>, <code>HAVING</code>, <code>FILTER</code></td>
</tr>
<tr>
<td align="right">3</td>
<td>소계와 총계를 한 번에 어떻게 구할까?</td>
<td><code>ROLLUP</code>, <code>CUBE</code>, <code>GROUPING</code></td>
</tr>
<tr>
<td align="right">4</td>
<td>여러 테이블을 어떻게 연결할까?</td>
<td><code>INNER</code>, <code>LEFT</code>, <code>FULL</code>, <code>CROSS JOIN</code></td>
</tr>
<tr>
<td align="right">5</td>
<td>값이나 존재 여부를 다른 쿼리로 확인하려면?</td>
<td>서브쿼리, <code>IN</code>, <code>EXISTS</code></td>
</tr>
<tr>
<td align="right">6</td>
<td>여러 조회 결과를 합치거나 비교하려면?</td>
<td><code>UNION</code>, <code>INTERSECT</code>, <code>EXCEPT</code></td>
</tr>
<tr>
<td align="right">7</td>
<td>긴 쿼리를 단계별로 나누려면?</td>
<td>CTE, <code>WITH</code>, 재귀 CTE</td>
</tr>
<tr>
<td align="right">8</td>
<td>자주 쓰는 조회를 재사용하려면?</td>
<td>View, Materialized View</td>
</tr>
<tr>
<td align="right">9</td>
<td>원래 행을 유지하면서 순위와 누적값을 구하려면?</td>
<td>Window Function</td>
</tr>
<tr>
<td align="right">10</td>
<td>학과별 상위 3명처럼 실무 문제를 풀려면?</td>
<td>CTE + Window Function</td>
</tr>
</tbody></table>
<pre><code class="language-text">행 조회
  → 그룹별 요약
  → 테이블 연결
  → 다른 쿼리의 결과 활용
  → 복잡한 쿼리 분해·재사용
  → 행을 유지한 채 순위·비교·누적 분석</code></pre>
<hr />
<h2 id="0-실습-환경-확인">0. 실습 환경 확인</h2>
<pre><code class="language-text">departments  1 ─── N students
departments  1 ─── N professors
departments  1 ─── N courses

courses      1 ─── N course_offerings
professors   1 ─── N course_offerings

students     1 ─── N enrollments N ─── 1 course_offerings</code></pre>
<p>핵심 테이블의 역할은 다음과 같다.</p>
<table>
<thead>
<tr>
<th>테이블</th>
<th>역할</th>
<th>대표 키</th>
</tr>
</thead>
<tbody><tr>
<td><code>academic.departments</code></td>
<td>학과 정보</td>
<td><code>department_id</code></td>
</tr>
<tr>
<td><code>academic.students</code></td>
<td>학생 정보</td>
<td><code>student_id</code></td>
</tr>
<tr>
<td><code>academic.professors</code></td>
<td>교수 정보</td>
<td><code>professor_id</code></td>
</tr>
<tr>
<td><code>academic.courses</code></td>
<td>과목 카탈로그</td>
<td><code>course_id</code></td>
</tr>
<tr>
<td><code>academic.course_offerings</code></td>
<td>특정 학기에 실제로 열린 강좌</td>
<td><code>offering_id</code></td>
</tr>
<tr>
<td><code>academic.enrollments</code></td>
<td>학생과 개설강좌를 연결하는 수강신청</td>
<td><code>(student_id, offering_id)</code></td>
</tr>
</tbody></table>
<p>먼저 DBeaver에서 <code>campus_management</code> 데이터베이스의 SQL 편집기를 열고 확인한다.</p>
<pre><code class="language-sql">SELECT
    current_database() AS database_name,
    current_user AS connected_user;</code></pre>
<p>테이블과 데이터 건수도 확인한다.</p>
<pre><code class="language-sql">SELECT 'departments' AS table_name, COUNT(*) AS row_count
FROM academic.departments
UNION ALL
SELECT 'professors', COUNT(*) FROM academic.professors
UNION ALL
SELECT 'students', COUNT(*) FROM academic.students
UNION ALL
SELECT 'courses', COUNT(*) FROM academic.courses
UNION ALL
SELECT 'course_offerings', COUNT(*) FROM academic.course_offerings
UNION ALL
SELECT 'enrollments', COUNT(*) FROM academic.enrollments
ORDER BY table_name;</code></pre>
<table>
<thead>
<tr>
<th>table_name</th>
<th align="right">row_count</th>
</tr>
</thead>
<tbody><tr>
<td><code>course_offerings</code></td>
<td align="right">10</td>
</tr>
<tr>
<td><code>courses</code></td>
<td align="right">10</td>
</tr>
<tr>
<td><code>departments</code></td>
<td align="right">4</td>
</tr>
<tr>
<td><code>enrollments</code></td>
<td align="right">24</td>
</tr>
<tr>
<td><code>professors</code></td>
<td align="right">6</td>
</tr>
<tr>
<td><code>students</code></td>
<td align="right">12</td>
</tr>
</tbody></table>
<blockquote>
<p>결과 건수가 다르더라도 문법 학습에는 문제가 없다. 다만 이후 예시 결과의 행 수와 값은 자신의 데이터에 따라 달라질 수 있다.</p>
</blockquote>
<hr />
<h2 id="1-핵심-복습과-데이터-시스템의-역할">1. 핵심 복습과 데이터 시스템의 역할</h2>
<h3 id="11-ddl과-dml-다시-구분하기">1.1 DDL과 DML 다시 구분하기</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>목적</th>
<th>대표 명령</th>
</tr>
</thead>
<tbody><tr>
<td>DDL</td>
<td>데이터 구조 정의</td>
<td><code>CREATE</code>, <code>ALTER</code>, <code>DROP</code></td>
</tr>
<tr>
<td>DML</td>
<td>데이터 조회·변경</td>
<td><code>SELECT</code>, <code>INSERT</code>, <code>UPDATE</code>, <code>DELETE</code></td>
</tr>
<tr>
<td>TCL</td>
<td>트랜잭션 제어</td>
<td><code>BEGIN</code>, <code>COMMIT</code>, <code>ROLLBACK</code>, <code>SAVEPOINT</code></td>
</tr>
</tbody></table>
<p>중심은 여러 테이블에서 필요한 정보를 찾아내는 <strong>조회 SQL</strong>이다. 따라서 대부분의 예제는 <code>SELECT</code>를 사용한다.</p>
<h3 id="12-select의-작성-순서와-논리적-처리-순서">1.2 SELECT의 작성 순서와 논리적 처리 순서</h3>
<p>SQL은 작성한 순서와 실제로 의미가 처리되는 순서가 다르다.</p>
<pre><code class="language-sql">SELECT ...
FROM ...
JOIN ... ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...;</code></pre>
<p>논리적으로는 다음 흐름으로 이해하면 쉽다.</p>
<pre><code class="language-text">FROM/JOIN
  → WHERE
  → GROUP BY
  → HAVING
  → SELECT
  → ORDER BY
  → LIMIT</code></pre>
<p>이 순서를 알면 다음 질문에 답할 수 있다.</p>
<ul>
<li>왜 집계 결과는 일반적으로 <code>WHERE</code>가 아니라 <code>HAVING</code>에서 거를까?</li>
<li>왜 <code>LEFT JOIN</code> 뒤의 조건을 <code>WHERE</code>에 쓰면 학생이 사라질까?</li>
<li>왜 윈도우 함수로 만든 순위를 같은 단계의 <code>WHERE</code>에서 바로 거를 수 없을까?</li>
</ul>
<p>뒤에서 각 사례를 실제 SQL로 확인한다.</p>
<h3 id="13-dbms-data-warehouse-data-mining의-차이">1.3 DBMS, Data Warehouse, Data Mining의 차이</h3>
<p>세 용어는 모두 데이터를 다루지만 목적이 다르다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>DBMS</th>
<th>Data Warehouse</th>
<th>Data Mining</th>
</tr>
</thead>
<tbody><tr>
<td>주요 목적</td>
<td>일상 업무 처리</td>
<td>분석과 의사결정</td>
<td>패턴과 지식 발견</td>
</tr>
<tr>
<td>대표 작업</td>
<td>주문 등록, 수강신청</td>
<td>월별 매출, 장기 추세</td>
<td>분류, 군집, 예측</td>
</tr>
<tr>
<td>데이터</td>
<td>현재 상태 중심</td>
<td>여러 기간의 이력 통합</td>
<td>분석에 필요한 대용량 데이터</td>
</tr>
<tr>
<td>쿼리 성격</td>
<td>짧은 CRUD와 트랜잭션</td>
<td>복잡한 집계와 다차원 분석</td>
<td>통계·머신러닝 처리</td>
</tr>
<tr>
<td>흔한 표현</td>
<td>OLTP</td>
<td>OLAP</td>
<td>분석 모델링</td>
</tr>
</tbody></table>
<p>학사관리시스템을 예로 들면 다음과 같다.</p>
<ul>
<li>학생이 강좌를 신청한다: <strong>DBMS의 OLTP 작업</strong></li>
<li>최근 5년간 학과별 수강률을 분석한다: <strong>Data Warehouse의 OLAP 작업</strong></li>
<li>중도 이탈 가능성이 높은 학생을 예측한다: <strong>Data Mining 작업</strong></li>
</ul>
<h3 id="14-msa에서-postgresql만으로-모든-일을-하지-않는-이유">1.4 MSA에서 PostgreSQL만으로 모든 일을 하지 않는 이유</h3>
<p>마이크로서비스 환경에서는 도구별 강점을 조합한다.</p>
<table>
<thead>
<tr>
<th>기술</th>
<th>대표 역할</th>
<th>주의할 점</th>
</tr>
</thead>
<tbody><tr>
<td>PostgreSQL</td>
<td>정확한 원본 데이터와 트랜잭션</td>
<td>스키마와 무결성 규칙 관리</td>
</tr>
<tr>
<td>Kafka</td>
<td>이벤트 비동기 전달</td>
<td>DB 커밋과 메시지 발행의 일관성</td>
</tr>
<tr>
<td>Elasticsearch</td>
<td>전문 검색과 로그 검색</td>
<td>원본 DB와 즉시 같지 않을 수 있음</td>
</tr>
<tr>
<td>Redis</td>
<td>캐시, 세션, 만료 데이터</td>
<td>캐시 무효화와 데이터 유실 정책</td>
</tr>
</tbody></table>
<p>중요한 원칙은 <strong>각 도구의 역할을 명확히 하는 것</strong>이다. 예를 들어 결제 원장은 PostgreSQL에 두고, 검색용 색인은 Elasticsearch에 복제하며, 자주 읽는 결과만 Redis에 캐시할 수 있다.</p>
<hr />
<h2 id="2-집계-함수-여러-행을-하나의-값으로-요약하기">2. 집계 함수: 여러 행을 하나의 값으로 요약하기</h2>
<p>집계 함수는 여러 행을 입력받아 하나의 요약값을 만든다.</p>
<h3 id="21-핵심-집계-함수-다섯-가지">2.1 핵심 집계 함수 다섯 가지</h3>
<table>
<thead>
<tr>
<th>함수</th>
<th>의미</th>
<th>예시 질문</th>
</tr>
</thead>
<tbody><tr>
<td><code>COUNT</code></td>
<td>개수</td>
<td>학생은 몇 명인가?</td>
</tr>
<tr>
<td><code>SUM</code></td>
<td>합계</td>
<td>총 수강 학점은 얼마인가?</td>
</tr>
<tr>
<td><code>AVG</code></td>
<td>평균</td>
<td>평균 성적은 몇 점인가?</td>
</tr>
<tr>
<td><code>MIN</code></td>
<td>최솟값</td>
<td>가장 낮은 성적은 몇 점인가?</td>
</tr>
<tr>
<td><code>MAX</code></td>
<td>최댓값</td>
<td>가장 높은 성적은 몇 점인가?</td>
</tr>
</tbody></table>
<p>전체 수강신청 통계를 구해 보자.</p>
<pre><code class="language-sql">SELECT
    COUNT(*) AS enrollment_count,
    COUNT(score) AS scored_count,
    ROUND(AVG(score), 2) AS average_score,
    MIN(score) AS minimum_score,
    MAX(score) AS maximum_score
FROM academic.enrollments;</code></pre>
<p>여기서 <code>COUNT(*)</code>와 <code>COUNT(score)</code>는 결과가 다를 수 있다.</p>
<ul>
<li><code>COUNT(*)</code>: 행 자체를 세므로 <code>score</code>가 <code>NULL</code>이어도 포함한다.</li>
<li><code>COUNT(score)</code>: <code>score</code>가 <code>NULL</code>인 행은 제외한다.</li>
<li><code>AVG(score)</code>: <code>NULL</code>을 제외하고 입력된 점수만 평균 낸다.</li>
</ul>
<p>샘플 데이터에서는 수강신청 24건 중 점수가 입력된 행만 평균 계산에 사용된다.</p>
<h3 id="22-null과-0은-다르다">2.2 NULL과 0은 다르다</h3>
<pre><code class="language-text">score = NULL  → 아직 점수를 모르거나 입력하지 않음
score = 0     → 점수가 실제로 0점임</code></pre>
<p>다음처럼 <code>NULL</code>을 0으로 바꾸고 평균을 내면 질문 자체가 바뀐다.</p>
<pre><code class="language-sql">SELECT
    AVG(score) AS entered_scores_average,
    AVG(COALESCE(score, 0)) AS null_as_zero_average
FROM academic.enrollments;</code></pre>
<p>첫 번째는 <strong>입력된 성적의 평균</strong>, 두 번째는 <strong>미입력 성적을 0점으로 취급한 평균</strong>이다. 어느 것이 맞는지는 업무 규칙에 따라 결정해야 한다.</p>
<h3 id="23-count-distinct로-중복을-제거한-개수-구하기">2.3 COUNT DISTINCT로 중복을 제거한 개수 구하기</h3>
<p>수강신청은 한 학생이 여러 건 가질 수 있다. 수강신청 건수가 아니라 실제 수강 학생 수를 알고 싶다면 <code>DISTINCT</code>가 필요하다.</p>
<pre><code class="language-sql">SELECT
    COUNT(*) AS enrollment_rows,
    COUNT(DISTINCT student_id) AS unique_students
FROM academic.enrollments
WHERE enrollment_status &lt;&gt; 'DROPPED';</code></pre>
<hr />
<h2 id="3-group-by와-having-그룹별-통계-구하기">3. GROUP BY와 HAVING: 그룹별 통계 구하기</h2>
<p>집계 함수만 사용하면 전체를 하나로 요약한다. <code>GROUP BY</code>를 사용하면 학과별, 강좌별, 학년별로 나누어 요약할 수 있다.</p>
<h3 id="31-학과별-학생-수">3.1 학과별 학생 수</h3>
<pre><code class="language-sql">SELECT
    d.department_code,
    d.department_name,
    COUNT(s.student_id) AS student_count
FROM academic.departments AS d
LEFT JOIN academic.students AS s
    ON s.department_id = d.department_id
GROUP BY
    d.department_id,
    d.department_code,
    d.department_name
ORDER BY student_count DESC, d.department_code;</code></pre>
<p><code>LEFT JOIN</code>을 사용했기 때문에 학생이 한 명도 없는 학과도 <code>student_count = 0</code>으로 표시할 수 있다.</p>
<p>여기서 <code>COUNT(*)</code>가 아니라 <code>COUNT(s.student_id)</code>를 사용한 이유가 중요하다.</p>
<ul>
<li><code>LEFT JOIN</code>은 학생이 없어도 학과 행을 하나 남긴다.</li>
<li>그 상태에서 <code>COUNT(*)</code>를 쓰면 학생이 없는 학과도 1로 셀 수 있다.</li>
<li><code>COUNT(s.student_id)</code>는 <code>NULL</code>을 제외하므로 실제 학생 수 0을 얻는다.</li>
</ul>
<h3 id="32-강좌별-수강-인원과-평균-성적">3.2 강좌별 수강 인원과 평균 성적</h3>
<pre><code class="language-sql">SELECT
    c.course_code,
    c.course_title,
    COUNT(e.student_id) FILTER (
        WHERE e.enrollment_status &lt;&gt; 'DROPPED'
    ) AS active_enrollment_count,
    ROUND(AVG(e.score), 2) AS average_score,
    MIN(e.score) AS minimum_score,
    MAX(e.score) AS maximum_score
FROM academic.course_offerings AS o
JOIN academic.courses AS c
    ON c.course_id = o.course_id
LEFT JOIN academic.enrollments AS e
    ON e.offering_id = o.offering_id
WHERE o.academic_year = 2026
  AND o.semester = 'SPRING'
GROUP BY
    o.offering_id,
    c.course_code,
    c.course_title
ORDER BY c.course_code;</code></pre>
<h3 id="33-select에-쓸-수-있는-컬럼-규칙">3.3 SELECT에 쓸 수 있는 컬럼 규칙</h3>
<p><code>GROUP BY</code>를 사용한 쿼리의 <code>SELECT</code>에는 보통 다음 두 종류만 올 수 있다.</p>
<ol>
<li><code>GROUP BY</code>에 포함된 그룹 기준 컬럼</li>
<li><code>COUNT</code>, <code>AVG</code> 같은 집계 함수로 감싼 컬럼</li>
</ol>
<p>다음 쿼리는 잘못된 예다.</p>
<pre><code class="language-sql">-- 잘못된 예: student_name은 그룹 기준도 아니고 집계값도 아니다.
SELECT
    department_id,
    student_name,
    COUNT(*)
FROM academic.students
GROUP BY department_id;</code></pre>
<p>한 학과에 학생이 여러 명이라면 DBMS는 어떤 이름을 한 개만 보여줘야 할지 결정할 수 없다. 따라서 필요한 질문에 맞게 쿼리를 바꿔야 한다.</p>
<pre><code class="language-sql">-- 학과별 학생 수가 목적이라면 이름을 제외한다.
SELECT
    department_id,
    COUNT(*) AS student_count
FROM academic.students
GROUP BY department_id;</code></pre>
<h3 id="34-where와-having의-차이">3.4 WHERE와 HAVING의 차이</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>필터 대상</th>
<th>처리 시점</th>
<th>대표 조건</th>
</tr>
</thead>
<tbody><tr>
<td><code>WHERE</code></td>
<td>집계 전 개별 행</td>
<td><code>GROUP BY</code> 전</td>
<td>재학생만, 2026년 강좌만</td>
</tr>
<tr>
<td><code>HAVING</code></td>
<td>집계 후 생성된 그룹</td>
<td><code>GROUP BY</code> 후</td>
<td>수강생 3명 이상, 평균 80점 이상</td>
</tr>
</tbody></table>
<pre><code class="language-sql">SELECT
    d.department_name,
    COUNT(s.student_id) AS enrolled_student_count
FROM academic.departments AS d
JOIN academic.students AS s
    ON s.department_id = d.department_id
WHERE s.student_status = 'ENROLLED'
GROUP BY d.department_id, d.department_name
HAVING COUNT(s.student_id) &gt;= 3
ORDER BY enrolled_student_count DESC;</code></pre>
<p>이 쿼리의 해석 순서는 다음과 같다.</p>
<ol>
<li>재학생 행만 <code>WHERE</code>로 남긴다.</li>
<li>학과별로 묶는다.</li>
<li>학과별 학생 수를 계산한다.</li>
<li>재학생이 3명 이상인 학과만 <code>HAVING</code>으로 남긴다.</li>
</ol>
<h3 id="35-조건부-집계-filter와-case-when">3.5 조건부 집계: FILTER와 CASE WHEN</h3>
<p>한 번의 조회로 완료·수강중·취소 건수를 나눠 세고 싶을 수 있다.</p>
<p>PostgreSQL에서는 <code>FILTER</code>가 읽기 쉽다.</p>
<pre><code class="language-sql">SELECT
    COUNT(*) AS total_count,
    COUNT(*) FILTER (
        WHERE enrollment_status = 'COMPLETED'
    ) AS completed_count,
    COUNT(*) FILTER (
        WHERE enrollment_status = 'ENROLLED'
    ) AS enrolled_count,
    COUNT(*) FILTER (
        WHERE enrollment_status = 'DROPPED'
    ) AS dropped_count
FROM academic.enrollments;</code></pre>
<p>다른 DBMS까지 고려해야 한다면 <code>CASE WHEN</code>으로 같은 의미를 표현할 수 있다.</p>
<pre><code class="language-sql">SELECT
    COUNT(*) AS total_count,
    SUM(CASE
        WHEN enrollment_status = 'COMPLETED' THEN 1
        ELSE 0
    END) AS completed_count,
    SUM(CASE
        WHEN enrollment_status = 'ENROLLED' THEN 1
        ELSE 0
    END) AS enrolled_count,
    SUM(CASE
        WHEN enrollment_status = 'DROPPED' THEN 1
        ELSE 0
    END) AS dropped_count
FROM academic.enrollments;</code></pre>
<hr />
<h2 id="4-rollup과-cube-소계와-총계를-한-번에-구하기">4. ROLLUP과 CUBE: 소계와 총계를 한 번에 구하기</h2>
<h3 id="41-rollup은-계층적인-소계를-만든다">4.1 ROLLUP은 계층적인 소계를 만든다</h3>
<p><code>ROLLUP(A, B)</code>는 다음 수준의 집계를 만든다.</p>
<pre><code class="language-text">(A, B) 상세 그룹
(A)    A별 소계
()     전체 총계</code></pre>
<p>학과별·학년별 학생 수와 소계, 총계를 한 번에 구해 보자.</p>
<pre><code class="language-sql">SELECT
    CASE
        WHEN GROUPING(d.department_name) = 1 THEN '전체 학과'
        ELSE d.department_name
    END AS department_name,
    CASE
        WHEN GROUPING(s.grade_level) = 1 THEN '소계'
        ELSE s.grade_level::TEXT || '학년'
    END AS grade_group,
    COUNT(*) AS student_count
FROM academic.students AS s
JOIN academic.departments AS d
    ON d.department_id = s.department_id
GROUP BY ROLLUP(d.department_name, s.grade_level)
ORDER BY
    GROUPING(d.department_name),
    d.department_name,
    GROUPING(s.grade_level),
    s.grade_level;</code></pre>
<h3 id="42-grouping이-필요한-이유">4.2 GROUPING이 필요한 이유</h3>
<p><code>ROLLUP</code>이 만든 소계 행은 그룹 기준 컬럼에 <code>NULL</code>을 사용한다. 하지만 원본 데이터에도 실제 <code>NULL</code>이 존재할 수 있다.</p>
<pre><code class="language-sql">GROUPING(column_name)</code></pre>
<ul>
<li><code>0</code>: 원본 그룹 값</li>
<li><code>1</code>: 소계나 총계를 만들기 위해 생성된 값</li>
</ul>
<p>따라서 단순히 <code>COALESCE(column_name, '전체')</code>만 쓰는 것보다 <code>GROUPING</code>으로 집계용 <code>NULL</code>인지 구분하는 것이 정확하다.</p>
<h3 id="43-cube는-모든-조합의-소계를-만든다">4.3 CUBE는 모든 조합의 소계를 만든다</h3>
<p><code>CUBE(A, B)</code>는 다음 집계를 모두 만든다.</p>
<pre><code class="language-text">(A, B) 상세 그룹
(A)    A별 소계
(B)    B별 소계
()     전체 총계</code></pre>
<pre><code class="language-sql">SELECT
    d.department_name,
    s.grade_level,
    COUNT(*) AS student_count
FROM academic.students AS s
JOIN academic.departments AS d
    ON d.department_id = s.department_id
GROUP BY CUBE(d.department_name, s.grade_level);</code></pre>
<p><code>CUBE</code>는 분석 차원이 늘어날수록 조합 수가 빠르게 증가한다. 필요한 소계 조합이 정해져 있다면 <code>GROUPING SETS</code>로 정확히 지정하는 방법도 있다.</p>
<pre><code class="language-sql">SELECT
    d.department_name,
    s.grade_level,
    COUNT(*) AS student_count
FROM academic.students AS s
JOIN academic.departments AS d
    ON d.department_id = s.department_id
GROUP BY GROUPING SETS (
    (d.department_name, s.grade_level),
    (d.department_name),
    ()
);</code></pre>
<hr />
<h2 id="5-페이지네이션-offset과-cursor-방식">5. 페이지네이션: OFFSET과 Cursor 방식</h2>
<p>웹 화면은 데이터를 한 번에 전부 보여주지 않고 페이지 단위로 나눈다.</p>
<h3 id="51-offset-방식">5.1 OFFSET 방식</h3>
<pre><code class="language-sql">SELECT
    student_id,
    student_no,
    student_name
FROM academic.students
ORDER BY student_id
LIMIT 10 OFFSET 20;</code></pre>
<p>장점은 특정 페이지로 바로 이동하기 쉽다는 것이다. 하지만 뒤 페이지로 갈수록 DB가 앞의 행을 읽고 버리는 비용이 커질 수 있다.</p>
<h3 id="52-cursor-또는-keyset-방식">5.2 Cursor 또는 Keyset 방식</h3>
<p>첫 페이지를 조회한다.</p>
<pre><code class="language-sql">SELECT
    student_id,
    student_no,
    student_name
FROM academic.students
ORDER BY student_id
LIMIT 10;</code></pre>
<p>마지막으로 받은 <code>student_id</code>가 1000이라면 다음 페이지는 다음처럼 조회한다.</p>
<pre><code class="language-sql">SELECT
    student_id,
    student_no,
    student_name
FROM academic.students
WHERE student_id &gt; 1000
ORDER BY student_id
LIMIT 10;</code></pre>
<p>정렬값이 중복될 수 있다면 고유 키를 두 번째 기준으로 추가한다.</p>
<pre><code class="language-sql">SELECT
    enrolled_at,
    student_id,
    offering_id,
    score
FROM academic.enrollments
WHERE (enrolled_at, student_id, offering_id)
    &gt; (DATE '2026-02-14', 7, 4)
ORDER BY enrolled_at, student_id, offering_id
LIMIT 10;</code></pre>
<table>
<thead>
<tr>
<th>방식</th>
<th>장점</th>
<th>단점</th>
<th>적합한 화면</th>
</tr>
</thead>
<tbody><tr>
<td>OFFSET</td>
<td>구현이 단순하고 임의 페이지 이동 가능</td>
<td>뒤 페이지가 느려질 수 있고 데이터 변경 시 중복·누락 가능</td>
<td>번호가 있는 짧은 목록</td>
</tr>
<tr>
<td>Keyset</td>
<td>큰 데이터에서도 다음 페이지가 안정적으로 빠름</td>
<td>임의 페이지 이동이 어렵고 커서 관리 필요</td>
<td>무한 스크롤, 다음 버튼</td>
</tr>
</tbody></table>
<p>Keyset 방식도 정렬 조건과 맞는 인덱스가 있어야 장점을 얻는다.</p>
<h3 id="53-order-by와-null-정렬">5.3 ORDER BY와 NULL 정렬</h3>
<p>정렬 방향을 생략하면 기본은 <code>ASC</code>다. 여러 기준은 왼쪽부터 차례대로 적용한다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    student_name,
    grade_level,
    phone
FROM academic.students
ORDER BY
    grade_level DESC,
    student_name ASC;</code></pre>
<p>PostgreSQL은 <code>NULLS FIRST</code>, <code>NULLS LAST</code>로 <code>NULL</code>의 위치를 명시할 수 있다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    student_name,
    phone
FROM academic.students
ORDER BY phone ASC NULLS LAST, student_no;</code></pre>
<p>DBMS마다 기본 <code>NULL</code> 정렬 위치가 다를 수 있다. 결과 순서가 중요하다면 기본값에 의존하지 말고 명시하는 편이 안전하다.</p>
<hr />
<h2 id="6-join-여러-테이블을-하나의-결과로-연결하기">6. JOIN: 여러 테이블을 하나의 결과로 연결하기</h2>
<p>정규화한 데이터베이스는 학생, 학과, 과목을 서로 다른 테이블에 저장한다. <code>JOIN</code>은 나누어 저장한 데이터를 질문에 맞게 다시 연결한다.</p>
<h3 id="61-join을-이해하는-가장-쉬운-질문">6.1 JOIN을 이해하는 가장 쉬운 질문</h3>
<pre><code class="language-text">어떤 테이블을 기준으로 시작할까?
어떤 키끼리 연결할까?
연결되지 않은 행도 보여줄까?</code></pre>
<h3 id="62-inner-join-양쪽에-모두-존재하는-행">6.2 INNER JOIN: 양쪽에 모두 존재하는 행</h3>
<p>** 두 테이블에 모두 매칭되는 행만 반환(교집합)
**수강신청이 실제로 존재하는 학생과 과목만 조회한다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    c.course_code,
    c.course_title,
    e.score
FROM academic.students AS s
INNER JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
INNER JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
INNER JOIN academic.courses AS c
    ON c.course_id = o.course_id
ORDER BY s.student_no, c.course_code;</code></pre>
<p><code>INNER</code>는 생략할 수 있다. 즉 <code>JOIN</code>은 기본적으로 <code>INNER JOIN</code>이다.</p>
<h3 id="63-left-join-왼쪽의-모든-행을-보존한다">6.3 LEFT JOIN: 왼쪽의 모든 행을 보존한다</h3>
<p>*<em>왼쪽 테이블 전체+오른쪽 없으면 NULL(모든 a를 보여주고 b가 없으면 null)
*</em>수강신청이 없는 학생까지 모두 표시하려면 학생을 왼쪽에 두고 <code>LEFT JOIN</code>한다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    c.course_code,
    c.course_title
FROM academic.students AS s
LEFT JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
LEFT JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
LEFT JOIN academic.courses AS c
    ON c.course_id = o.course_id
ORDER BY s.student_no, c.course_code;</code></pre>
<p>수강신청이 없는 학생은 강좌 컬럼이 <code>NULL</code>로 표시된다.</p>
<h3 id="64-right-join과-full-outer-join">6.4 RIGHT JOIN과 FULL OUTER JOIN</h3>
<p>*<em><code>RIGHT JOIN</code>은 오른쪽 테이블의 모든 행을 보존한다. 하지만 테이블 순서를 바꾸어 <code>LEFT JOIN</code>으로 쓰는 편이 읽기 쉬운 경우가 많다.(가독성 문제)
*</em>
*<em><code>FULL OUTER JOIN</code>은 양쪽의 모든 행을 보존한다.
*</em></p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    e.offering_id,
    e.score
FROM academic.students AS s
FULL OUTER JOIN academic.enrollments AS e
    ON e.student_id = s.student_id;</code></pre>
<p>현재  테이블에는 외래 키가 있으므로 존재하지 않는 학생을 참조하는 수강신청은 들어갈 수 없다. 따라서 이 구조에서는 <code>FULL OUTER JOIN</code>의 오른쪽 단독 행이 정상적으로 발생하지 않는다. 실습 자료의 고아 데이터는 외래 키를 제거했거나 별도 실습 테이블을 사용한 경우에만 만들 수 있다.</p>
<h3 id="65-cross-join-가능한-모든-조합">6.5 CROSS JOIN: 가능한 모든 조합</h3>
<p>모든 경우의 수를 만듭니다.
모든 학생과 모든 개설강좌의 조합을 만들 수 있다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    o.offering_id,
    c.course_code
FROM academic.students AS s
CROSS JOIN academic.course_offerings AS o
JOIN academic.courses AS c
    ON c.course_id = o.course_id
ORDER BY s.student_no, c.course_code
LIMIT 100;</code></pre>
<p>학생 12명과 개설강좌 10개를 곱하면 120행이 된다. 데이터가 커지면 행 수가 폭발하므로 의도한 경우에만 사용한다.</p>
<h3 id="66-self-join-같은-테이블을-두-역할로-사용하기">6.6 SELF JOIN: 같은 테이블을 두 역할로 사용하기</h3>
<p>*<em>같은 테이블을 2개처럼 조인
*</em>조직도의 직원과 상사는 모두 직원이므로 같은 테이블을 두 번 사용한다.</p>
<pre><code class="language-sql">-- 별도 조직도 예시 테이블이라고 가정한다.
SELECT
    employee.employee_name,
    manager.employee_name AS manager_name
FROM organization.employees AS employee
LEFT JOIN organization.employees AS manager
    ON manager.employee_id = employee.manager_id;</code></pre>
<p>별칭 <code>employee</code>, <code>manager</code>가 같은 테이블의 역할을 구분한다.</p>
<h3 id="67-anti-join-대응하는-데이터가-없는-행-찾기">6.7 Anti-Join: 대응하는 데이터가 없는 행 찾기</h3>
<p>한 번도 수강하지 않은 학생을 찾는 가장 안전한 방법은 <code>NOT EXISTS</code>다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name
FROM academic.students AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM academic.enrollments AS e
    WHERE e.student_id = s.student_id
)
ORDER BY s.student_no;</code></pre>
<p><code>LEFT JOIN ... IS NULL</code>로도 표현할 수 있다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name
FROM academic.students AS s
LEFT JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
WHERE e.student_id IS NULL
ORDER BY s.student_no;</code></pre>
<h3 id="68-semi-join-대응하는-데이터가-있는지만-확인하기">6.8 Semi-Join: 대응하는 데이터가 있는지만 확인하기</h3>
<p>완료된 과목이 하나라도 있는 학생을 찾는다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name
FROM academic.students AS s
WHERE EXISTS (
    SELECT 1
    FROM academic.enrollments AS e
    WHERE e.student_id = s.student_id
      AND e.enrollment_status = 'COMPLETED'
)
ORDER BY s.student_no;</code></pre>
<p>오른쪽 테이블의 컬럼은 결과에 필요 없고 존재 여부만 확인한다. 일반 <code>JOIN</code>으로 바꾸면 한 학생의 완료 과목 수만큼 중복될 수 있어 <code>DISTINCT</code>가 추가로 필요할 수 있다.</p>
<table>
<thead>
<tr>
<th>JOIN 종류</th>
<th>반환 결과</th>
<th>주 사용 목적</th>
</tr>
</thead>
<tbody><tr>
<td><strong>INNER JOIN</strong></td>
<td>양쪽에 모두 있는 데이터만</td>
<td>가장 많이 사용</td>
</tr>
<tr>
<td><strong>LEFT JOIN</strong></td>
<td>왼쪽 테이블 전체 + 일치하는 오른쪽</td>
<td>기준 테이블을 모두 보고 싶을 때</td>
</tr>
<tr>
<td><strong>RIGHT JOIN</strong></td>
<td>오른쪽 테이블 전체 + 일치하는 왼쪽</td>
<td>오른쪽 테이블을 기준으로 조회할 때</td>
</tr>
<tr>
<td><strong>FULL OUTER JOIN</strong></td>
<td>양쪽 테이블 전체</td>
<td>모든 데이터를 확인할 때</td>
</tr>
<tr>
<td><strong>CROSS JOIN</strong></td>
<td>가능한 모든 조합</td>
<td>조합 생성, 테스트 데이터</td>
</tr>
<tr>
<td><strong>SELF JOIN</strong></td>
<td>자기 자신과 조인</td>
<td>계층 구조(직원-관리자, 부모-자식)</td>
</tr>
</tbody></table>
<hr />
<h2 id="7-join-알고리즘-dbms는-내부에서-어떻게-연결할까">7. JOIN 알고리즘: DBMS는 내부에서 어떻게 연결할까?</h2>
<p>작성자는 <code>JOIN</code>이라는 논리적 요구를 적고, PostgreSQL 옵티마이저는 통계와 비용을 바탕으로 실제 실행 방법을 선택한다.</p>
<h3 id="71-nested-loop-join">7.1 Nested Loop Join</h3>
<pre><code class="language-text">왼쪽 행 하나 선택
  → 오른쪽에서 일치하는 행 탐색
  → 다음 왼쪽 행 반복</code></pre>
<ul>
<li>왼쪽 결과가 작을 때 유리하다.</li>
<li>오른쪽 연결 컬럼에 적절한 인덱스가 있으면 빠르다.</li>
<li>큰 집합을 반복 스캔하면 느려질 수 있다.</li>
</ul>
<h3 id="72-hash-join">7.2 Hash Join</h3>
<pre><code class="language-text">작은 입력으로 해시 테이블 생성
  → 큰 입력의 키를 해시에서 탐색</code></pre>
<ul>
<li><code>=</code> 조건의 대용량 조인에 강하다.</li>
<li>적절한 인덱스가 없어도 효율적일 수 있다.</li>
<li>메모리가 부족하면 임시 디스크 작업이 발생할 수 있다.</li>
</ul>
<h3 id="73-merge-join">7.3 Merge Join</h3>
<pre><code class="language-text">양쪽 입력을 조인 키 순서로 준비
  → 두 정렬 흐름을 함께 이동하며 병합</code></pre>
<ul>
<li>양쪽 데이터가 이미 정렬되어 있거나 인덱스 순서를 이용할 수 있을 때 유리하다.</li>
<li>큰 정렬 비용이 필요하면 다른 방식이 더 나을 수 있다.</li>
</ul>
<h3 id="74-알고리즘을-외워서-강제로-선택해야-할까">7.4 알고리즘을 외워서 강제로 선택해야 할까?</h3>
<p>초보 단계에서는 알고리즘을 직접 지정하기보다 다음을 우선한다.</p>
<ol>
<li>올바른 <code>JOIN</code> 조건을 작성한다.</li>
<li>불필요한 행과 컬럼을 줄인다.</li>
<li>조인 키와 자주 필터하는 컬럼의 인덱스를 검토한다.</li>
<li>통계를 최신 상태로 유지한다.</li>
<li><code>EXPLAIN</code>으로 실제 계획을 확인한다.</li>
</ol>
<p>PostgreSQL은 외래 키를 만들었다고 해서 참조하는 쪽 컬럼의 인덱스를 자동 생성하지 않는다. 조회·삭제 패턴을 보고 다음과 같은 인덱스를 검토할 수 있다.</p>
<pre><code class="language-sql">CREATE INDEX IF NOT EXISTS idx_students_department_id
    ON academic.students (department_id);

CREATE INDEX IF NOT EXISTS idx_offerings_course_id
    ON academic.course_offerings (course_id);

CREATE INDEX IF NOT EXISTS idx_offerings_professor_id
    ON academic.course_offerings (professor_id);</code></pre>
<p>무조건 모든 외래 키에 인덱스를 추가하기보다는 테이블 크기와 쿼리 패턴을 함께 본다. 인덱스는 조회를 빠르게 할 수 있지만 저장 공간과 쓰기 비용이 증가한다.</p>
<h3 id="75-explain으로-실행계획-확인하기">7.5 EXPLAIN으로 실행계획 확인하기</h3>
<pre><code class="language-sql">EXPLAIN
SELECT
    s.student_name,
    c.course_title,
    e.score
FROM academic.students AS s
JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
JOIN academic.courses AS c
    ON c.course_id = o.course_id
WHERE s.grade_level = 3;</code></pre>
<p>실제로 실행하면서 시간과 버퍼 사용량까지 보려면 다음과 같이 작성한다.</p>
<pre><code class="language-sql">EXPLAIN (ANALYZE, BUFFERS)
SELECT
    s.student_name,
    c.course_title,
    e.score
FROM academic.students AS s
JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
JOIN academic.courses AS c
    ON c.course_id = o.course_id
WHERE s.grade_level = 3;</code></pre>
<p>초보자는 다음 항목부터 확인하면 된다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>Seq Scan</code></td>
<td>테이블을 순차적으로 읽음</td>
</tr>
<tr>
<td><code>Index Scan</code></td>
<td>인덱스를 이용해 테이블 행을 찾음</td>
</tr>
<tr>
<td><code>Index Only Scan</code></td>
<td>필요한 값을 인덱스 중심으로 해결</td>
</tr>
<tr>
<td><code>Nested Loop</code>, <code>Hash Join</code>, <code>Merge Join</code></td>
<td>선택된 조인 알고리즘</td>
</tr>
<tr>
<td><code>rows</code></td>
<td>옵티마이저가 예상한 행 수</td>
</tr>
<tr>
<td><code>actual rows</code></td>
<td>실제 처리된 행 수</td>
</tr>
<tr>
<td><code>Buffers</code></td>
<td>메모리·디스크 블록 사용 정보</td>
</tr>
</tbody></table>
<blockquote>
<p><code>EXPLAIN ANALYZE</code>는 쿼리를 실제로 실행한다. <code>UPDATE</code>, <code>DELETE</code>와 함께 사용할 때는 테스트 환경이나 롤백 가능한 트랜잭션에서 특히 주의해야 한다.</p>
</blockquote>
<hr />
<h2 id="8-join에서-가장-자주-발생하는-실수">8. JOIN에서 가장 자주 발생하는 실수</h2>
<h3 id="81-on-조건-누락">8.1 ON 조건 누락</h3>
<pre><code class="language-sql">-- 잘못된 예: 연결 조건이 없어 모든 조합이 만들어진다.
SELECT *
FROM academic.students AS s
CROSS JOIN academic.enrollments AS e;</code></pre>
<p>학생 12명과 수강신청 24건이면 288행이 된다. 일반 조인이 목적이었다면 다음 연결 조건이 필요하다.</p>
<pre><code class="language-sql">ON e.student_id = s.student_id</code></pre>
<h3 id="82-left-join-뒤-where가-행을-제거하는-문제">8.2 LEFT JOIN 뒤 WHERE가 행을 제거하는 문제</h3>
<p>모든 학생을 유지하면서 <code>CS101</code> 수강정보만 붙이고 싶다고 하자.</p>
<p>다음 쿼리는 수강하지 않은 학생을 제거한다.</p>
<pre><code class="language-sql">-- 의도와 다른 예: WHERE가 NULL 행을 제거해 사실상 INNER JOIN처럼 된다.
SELECT
    s.student_name,
    c.course_code,
    e.score
FROM academic.students AS s
LEFT JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
LEFT JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
LEFT JOIN academic.courses AS c
    ON c.course_id = o.course_id
WHERE c.course_code = 'CS101';</code></pre>
<p>오른쪽 테이블의 필터를 <code>ON</code> 단계에 포함하면 모든 학생을 유지할 수 있다.</p>
<pre><code class="language-sql">SELECT
    s.student_name,
    c.course_code,
    e.score
FROM academic.students AS s
LEFT JOIN (
    academic.enrollments AS e
    JOIN academic.course_offerings AS o
        ON o.offering_id = e.offering_id
    JOIN academic.courses AS c
        ON c.course_id = o.course_id
       AND c.course_code = 'CS101'
) ON e.student_id = s.student_id
ORDER BY s.student_name;</code></pre>
<p>또는 대상 강좌를 먼저 CTE로 만들면 의도가 더 선명하다.</p>
<pre><code class="language-sql">WITH cs101_enrollments AS (
    SELECT
        e.student_id,
        e.score
    FROM academic.enrollments AS e
    JOIN academic.course_offerings AS o
        ON o.offering_id = e.offering_id
    JOIN academic.courses AS c
        ON c.course_id = o.course_id
    WHERE c.course_code = 'CS101'
)
SELECT
    s.student_name,
    ce.score
FROM academic.students AS s
LEFT JOIN cs101_enrollments AS ce
    ON ce.student_id = s.student_id
ORDER BY s.student_name;</code></pre>
<h3 id="83-1n-join-뒤-count가-부풀어-오르는-문제">8.3 1:N JOIN 뒤 COUNT가 부풀어 오르는 문제</h3>
<p>학과, 학생, 수강신청을 연결하면 학생 한 명이 수강신청 수만큼 반복된다.</p>
<pre><code class="language-sql">SELECT
    d.department_name,
    COUNT(*) AS joined_row_count,
    COUNT(DISTINCT s.student_id) AS actual_student_count
FROM academic.departments AS d
JOIN academic.students AS s
    ON s.department_id = d.department_id
LEFT JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
GROUP BY d.department_id, d.department_name;</code></pre>
<p>무엇을 세는지 먼저 결정해야 한다.</p>
<ul>
<li>수강신청 행 수: <code>COUNT(e.offering_id)</code></li>
<li>학생 수: <code>COUNT(DISTINCT s.student_id)</code></li>
<li>점수가 입력된 수강신청 수: <code>COUNT(e.score)</code></li>
</ul>
<h3 id="84-not-in의-null-함정">8.4 NOT IN의 NULL 함정</h3>
<pre><code class="language-sql">WHERE student_id NOT IN (
    SELECT student_id FROM some_table
)</code></pre>
<p>서브쿼리 결과에 <code>NULL</code>이 하나라도 있으면 SQL의 3값 논리 때문에 기대와 다른 빈 결과가 나올 수 있다. 존재하지 않음을 확인할 때는 보통 <code>NOT EXISTS</code>가 더 안전하다.</p>
<hr />
<h2 id="9-서브쿼리-다른-쿼리의-결과를-재료로-사용하기">9. 서브쿼리: 다른 쿼리의 결과를 재료로 사용하기</h2>
<p>서브쿼리는 SQL 문 안에 포함된 또 다른 <code>SELECT</code>다.</p>
<h3 id="91-스칼라-서브쿼리">9.1 스칼라 서브쿼리</h3>
<p>스칼라 서브쿼리는 한 행, 한 컬럼의 단일 값을 반환해야 한다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    (
        SELECT d.department_name
        FROM academic.departments AS d
        WHERE d.department_id = s.department_id
    ) AS department_name
FROM academic.students AS s
ORDER BY s.student_no;</code></pre>
<p>이 문제는 일반 <code>JOIN</code>이 더 자연스럽다. 스칼라 서브쿼리는 간단한 단일값을 붙일 때 유용하지만 바깥 행마다 반복 평가될 가능성이 있어 큰 데이터에서는 실행계획을 확인한다.</p>
<h3 id="92-where-절의-단일값-서브쿼리">9.2 WHERE 절의 단일값 서브쿼리</h3>
<p>입력된 전체 평균 점수보다 높은 수강성적을 찾는다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    c.course_code,
    e.score
FROM academic.enrollments AS e
JOIN academic.students AS s
    ON s.student_id = e.student_id
JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
JOIN academic.courses AS c
    ON c.course_id = o.course_id
WHERE e.score &gt; (
    SELECT AVG(score)
    FROM academic.enrollments
    WHERE score IS NOT NULL
)
ORDER BY e.score DESC;</code></pre>
<h3 id="93-인라인-뷰">9.3 인라인 뷰</h3>
<p><code>FROM</code> 안의 서브쿼리는 조회 중에 사용하는 임시 결과처럼 동작한다.</p>
<pre><code class="language-sql">SELECT
    ranked.department_name,
    ranked.student_name,
    ranked.average_score,
    ranked.department_rank
FROM (
    SELECT
        d.department_name,
        s.student_name,
        AVG(e.score) AS average_score,
        ROW_NUMBER() OVER (
            PARTITION BY d.department_id
            ORDER BY AVG(e.score) DESC NULLS LAST, s.student_id
        ) AS department_rank
    FROM academic.students AS s
    JOIN academic.departments AS d
        ON d.department_id = s.department_id
    LEFT JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    GROUP BY
        d.department_id,
        d.department_name,
        s.student_id,
        s.student_name
) AS ranked
WHERE ranked.department_rank &lt;= 3
ORDER BY ranked.department_name, ranked.department_rank;</code></pre>
<h3 id="94-상관-서브쿼리">9.4 상관 서브쿼리</h3>
<p>상관 서브쿼리는 바깥 쿼리의 현재 행을 내부 쿼리에서 참조한다.</p>
<pre><code class="language-sql">SELECT
    e.student_id,
    e.offering_id,
    e.score
FROM academic.enrollments AS e
WHERE e.score &gt; (
    SELECT AVG(e2.score)
    FROM academic.enrollments AS e2
    WHERE e2.student_id = e.student_id
);</code></pre>
<p>각 수강성적을 그 학생 자신의 평균과 비교한다. 표현은 직관적이지만 바깥 행마다 내부 집계를 반복할 수 있으므로 CTE나 윈도우 함수로 바꾸는 방법도 검토한다.</p>
<pre><code class="language-sql">WITH scored AS (
    SELECT
        e.*,
        AVG(e.score) OVER (
            PARTITION BY e.student_id
        ) AS student_average
    FROM academic.enrollments AS e
    WHERE e.score IS NOT NULL
)
SELECT
    student_id,
    offering_id,
    score,
    student_average
FROM scored
WHERE score &gt; student_average;</code></pre>
<h3 id="95-in-exists-any-all">9.5 IN, EXISTS, ANY, ALL</h3>
<table>
<thead>
<tr>
<th>연산</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>IN</code></td>
<td>결과 목록 안에 값이 포함되는가?</td>
</tr>
<tr>
<td><code>EXISTS</code></td>
<td>조건을 만족하는 행이 하나라도 존재하는가?</td>
</tr>
<tr>
<td><code>ANY</code></td>
<td>결과 중 하나와 비교 조건을 만족하는가?</td>
</tr>
<tr>
<td><code>ALL</code></td>
<td>결과의 모든 값과 비교 조건을 만족하는가?</td>
</tr>
</tbody></table>
<p>컴퓨터공학과나 인공지능학과 학생을 찾는다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    student_name
FROM academic.students
WHERE department_id IN (
    SELECT department_id
    FROM academic.departments
    WHERE department_code IN ('CS', 'AI')
)
ORDER BY student_no;</code></pre>
<p>모든 입력된 점수보다 높은 기준값을 확인하는 형태는 <code>ALL</code>로 표현할 수 있다.</p>
<pre><code class="language-sql">SELECT 100 AS perfect_score
WHERE 100 &gt;= ALL (
    SELECT score
    FROM academic.enrollments
    WHERE score IS NOT NULL
);</code></pre>
<p>실무에서는 의미가 명확한 <code>EXISTS</code>, <code>NOT EXISTS</code>, <code>IN</code>을 자주 사용하며 <code>ANY</code>, <code>ALL</code>은 결과가 비어 있을 때의 의미까지 생각해야 한다.</p>
<hr />
<h2 id="10-집합-연산자-두-select-결과를-합치고-비교하기">10. 집합 연산자: 두 SELECT 결과를 합치고 비교하기</h2>
<p>집합 연산자는 행 집합과 행 집합을 연결한다. <code>JOIN</code>이 컬럼을 옆으로 연결한다면 집합 연산자는 결과를 주로 위아래로 연결한다.</p>
<h3 id="101-union과-union-all">10.1 UNION과 UNION ALL</h3>
<pre><code class="language-sql">SELECT student_id
FROM academic.enrollments
WHERE enrollment_status = 'COMPLETED'

UNION

SELECT student_id
FROM academic.enrollments
WHERE score &gt;= 90;</code></pre>
<ul>
<li><code>UNION</code>: 중복 제거</li>
<li><code>UNION ALL</code>: 중복 유지</li>
</ul>
<p>중복 제거가 필요 없다면 <code>UNION ALL</code>이 정렬·해시 등의 중복 제거 비용을 줄일 수 있다.</p>
<h3 id="102-intersect">10.2 INTERSECT</h3>
<p>완료된 과목도 있고 90점 이상 성적도 있는 학생의 교집합을 구한다.</p>
<pre><code class="language-sql">SELECT student_id
FROM academic.enrollments
WHERE enrollment_status = 'COMPLETED'

INTERSECT

SELECT student_id
FROM academic.enrollments
WHERE score &gt;= 90;</code></pre>
<h3 id="103-except">10.3 EXCEPT</h3>
<p>재학생 중 완료된 과목이 아직 없는 학생을 구한다.</p>
<pre><code class="language-sql">SELECT student_id
FROM academic.students
WHERE student_status = 'ENROLLED'

EXCEPT

SELECT student_id
FROM academic.enrollments
WHERE enrollment_status = 'COMPLETED';</code></pre>
<p>집합 연산자 양쪽 <code>SELECT</code>는 컬럼 개수가 같고 대응되는 데이터 타입이 호환되어야 한다.</p>
<hr />
<h2 id="11-cte-긴-쿼리를-단계별로-나누기">11. CTE: 긴 쿼리를 단계별로 나누기</h2>
<p>CTE는 Common Table Expression의 약자로 <code>WITH</code> 절에서 이름을 붙인 임시 결과다. 해당 SQL 문 안에서만 사용한다.</p>
<h3 id="111-기본-cte">11.1 기본 CTE</h3>
<p>학생별 평균 성적을 먼저 계산하고 85점 이상인 학생만 조회한다.</p>
<pre><code class="language-sql">WITH student_average AS (
    SELECT
        s.student_id,
        s.student_no,
        s.student_name,
        AVG(e.score) AS average_score
    FROM academic.students AS s
    JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    WHERE e.score IS NOT NULL
    GROUP BY
        s.student_id,
        s.student_no,
        s.student_name
)
SELECT
    student_no,
    student_name,
    ROUND(average_score, 2) AS average_score
FROM student_average
WHERE average_score &gt;= 85
ORDER BY average_score DESC;</code></pre>
<h3 id="112-다중-cte">11.2 다중 CTE</h3>
<p>복잡한 분석을 단계별로 이름 붙이면 데이터 흐름이 보인다.</p>
<pre><code class="language-sql">WITH student_average AS (
    SELECT
        s.student_id,
        s.department_id,
        AVG(e.score) AS student_avg
    FROM academic.students AS s
    JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    WHERE e.score IS NOT NULL
    GROUP BY s.student_id, s.department_id
),
department_average AS (
    SELECT
        department_id,
        AVG(student_avg) AS department_avg
    FROM student_average
    GROUP BY department_id
)
SELECT
    s.student_no,
    s.student_name,
    d.department_name,
    ROUND(sa.student_avg, 2) AS student_avg,
    ROUND(da.department_avg, 2) AS department_avg
FROM student_average AS sa
JOIN department_average AS da
    ON da.department_id = sa.department_id
JOIN academic.students AS s
    ON s.student_id = sa.student_id
JOIN academic.departments AS d
    ON d.department_id = sa.department_id
WHERE sa.student_avg &gt; da.department_avg
ORDER BY d.department_name, sa.student_avg DESC;</code></pre>
<h3 id="113-재귀-cte">11.3 재귀 CTE</h3>
<p>재귀 CTE는 조직도나 카테고리처럼 부모와 자식이 반복되는 구조를 탐색한다.</p>
<pre><code class="language-sql">WITH RECURSIVE organization_tree AS (
    -- 1. Anchor: 최상위 직원
    SELECT
        employee_id,
        employee_name,
        manager_id,
        0 AS depth,
        employee_name::TEXT AS path
    FROM organization.employees
    WHERE manager_id IS NULL

    UNION ALL

    -- 2. Recursive: 현재 직원의 부하 직원을 연결
    SELECT
        child.employee_id,
        child.employee_name,
        child.manager_id,
        parent.depth + 1,
        parent.path || ' &gt; ' || child.employee_name
    FROM organization.employees AS child
    JOIN organization_tree AS parent
        ON parent.employee_id = child.manager_id
    WHERE parent.depth &lt; 10
)
SELECT
    employee_id,
    employee_name,
    depth,
    path
FROM organization_tree
ORDER BY path;</code></pre>
<p>재귀 CTE에는 반드시 다음을 확인한다.</p>
<ul>
<li>시작점인 Anchor가 있는가?</li>
<li>부모와 자식을 연결하는 조건이 정확한가?</li>
<li>순환 데이터나 무한 재귀를 막을 조건이 있는가?</li>
</ul>
<h3 id="114-materialized와-not-materialized">11.4 MATERIALIZED와 NOT MATERIALIZED</h3>
<p>PostgreSQL은 CTE의 사용 방식과 내용에 따라 쿼리 안으로 펼치거나 중간 결과로 계산할 수 있다. 필요한 경우 힌트를 명시할 수 있다.</p>
<pre><code class="language-sql">WITH completed_stats AS MATERIALIZED (
    SELECT
        student_id,
        COUNT(*) AS completed_count,
        AVG(score) AS average_score
    FROM academic.enrollments
    WHERE enrollment_status = 'COMPLETED'
    GROUP BY student_id
)
SELECT
    s.student_name,
    cs.completed_count,
    cs.average_score
FROM academic.students AS s
JOIN completed_stats AS cs
    ON cs.student_id = s.student_id;</code></pre>
<p><code>MATERIALIZED</code>가 항상 빠른 것은 아니다. 중간 결과 저장 비용이 생기며, 옵티마이저가 조건을 안쪽으로 밀어 넣는 최적화를 제한할 수 있다. 반복 참조되는 비싼 계산인지 실행계획으로 확인한 뒤 선택한다.</p>
<hr />
<h2 id="12-view와-materialized-view">12. View와 Materialized View</h2>
<h3 id="121-view는-저장된-select-문이다">12.1 View는 저장된 SELECT 문이다</h3>
<p>학생 대시보드 조회를 View로 만든다.</p>
<pre><code class="language-sql">CREATE OR REPLACE VIEW academic.v_student_dashboard AS
SELECT
    s.student_id,
    s.student_no,
    s.student_name,
    d.department_name,
    COUNT(e.offering_id) AS enrollment_count,
    ROUND(AVG(e.score), 2) AS average_score,
    MAX(e.score) AS best_score,
    COALESCE(SUM(c.credits) FILTER (
        WHERE e.enrollment_status &lt;&gt; 'DROPPED'
    ), 0) AS registered_credits
FROM academic.students AS s
LEFT JOIN academic.departments AS d
    ON d.department_id = s.department_id
LEFT JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
LEFT JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
LEFT JOIN academic.courses AS c
    ON c.course_id = o.course_id
GROUP BY
    s.student_id,
    s.student_no,
    s.student_name,
    d.department_name;</code></pre>
<p>일반 테이블처럼 조회한다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    student_name,
    department_name,
    enrollment_count,
    average_score,
    registered_credits
FROM academic.v_student_dashboard
ORDER BY student_no;</code></pre>
<p>View는 보통 결과 자체를 저장하지 않고 조회할 때 원본 쿼리를 실행한다. 복잡한 SQL 재사용, 민감 컬럼 제외, 안정적인 조회 인터페이스 제공에 유용하다.</p>
<h3 id="122-materialized-view는-조회-결과를-저장한다">12.2 Materialized View는 조회 결과를 저장한다</h3>
<pre><code class="language-sql">CREATE MATERIALIZED VIEW academic.mv_course_statistics AS
SELECT
    o.offering_id,
    c.course_code,
    c.course_title,
    COUNT(e.student_id) AS enrollment_count,
    ROUND(AVG(e.score), 2) AS average_score
FROM academic.course_offerings AS o
JOIN academic.courses AS c
    ON c.course_id = o.course_id
LEFT JOIN academic.enrollments AS e
    ON e.offering_id = o.offering_id
GROUP BY
    o.offering_id,
    c.course_code,
    c.course_title;</code></pre>
<p>결과를 저장하므로 반복되는 무거운 집계를 빠르게 읽을 수 있다. 대신 원본 데이터가 바뀌어도 자동 갱신되지 않는다.</p>
<pre><code class="language-sql">REFRESH MATERIALIZED VIEW academic.mv_course_statistics;</code></pre>
<p>조회 중 잠금을 줄이기 위해 <code>CONCURRENTLY</code>를 사용하려면 Materialized View의 모든 행을 유일하게 식별하는 <strong>UNIQUE 인덱스</strong>가 필요하다.</p>
<pre><code class="language-sql">CREATE UNIQUE INDEX ux_mv_course_statistics_offering
    ON academic.mv_course_statistics (offering_id);

REFRESH MATERIALIZED VIEW CONCURRENTLY
    academic.mv_course_statistics;</code></pre>
<h3 id="123-cte-view-materialized-view-비교">12.3 CTE, View, Materialized View 비교</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>CTE</th>
<th>View</th>
<th>Materialized View</th>
</tr>
</thead>
<tbody><tr>
<td>사용 범위</td>
<td>한 SQL 문</td>
<td>여러 SQL에서 재사용</td>
<td>여러 SQL에서 재사용</td>
</tr>
<tr>
<td>결과 저장</td>
<td>안 함</td>
<td>보통 안 함</td>
<td>저장함</td>
</tr>
<tr>
<td>데이터 최신성</td>
<td>실행 시점</td>
<td>조회 시점</td>
<td>마지막 갱신 시점</td>
</tr>
<tr>
<td>인덱스</td>
<td>직접 생성 불가</td>
<td>일반 View에는 직접 생성 불가</td>
<td>생성 가능</td>
</tr>
<tr>
<td>대표 용도</td>
<td>복잡한 쿼리 분해</td>
<td>공통 조회·권한 제어</td>
<td>대시보드·무거운 집계 캐시</td>
</tr>
</tbody></table>
<hr />
<h2 id="13-window-function-행을-유지하며-분석하기">13. Window Function: 행을 유지하며 분석하기</h2>
<p>강의자료에는 <code>Windows Function</code>이라고 표기된 부분이 있지만 일반적인 용어는 <strong>Window Function</strong>, 한국어로 <strong>윈도우 함수</strong> 또는 <strong>분석 함수</strong>다.</p>
<h3 id="131-group-by와-핵심-차이">13.1 GROUP BY와 핵심 차이</h3>
<table>
<thead>
<tr>
<th>기능</th>
<th>결과 행 수</th>
<th>개별 행 유지</th>
<th>대표 질문</th>
</tr>
</thead>
<tbody><tr>
<td><code>GROUP BY</code></td>
<td>그룹 수만큼 줄어듦</td>
<td>아니요</td>
<td>학과별 평균은?</td>
</tr>
<tr>
<td>Window Function</td>
<td>원래 행 수 유지</td>
<td>예</td>
<td>각 학생 점수와 학과 평균을 함께 보여줘</td>
</tr>
</tbody></table>
<p>기본 문법은 다음과 같다.</p>
<pre><code class="language-sql">function_name(...) OVER (
    PARTITION BY group_column
    ORDER BY sort_column
    ROWS BETWEEN ... AND ...
)</code></pre>
<ul>
<li><code>PARTITION BY</code>: 분석 그룹을 나눈다.</li>
<li><code>ORDER BY</code>: 파티션 안의 순서를 정한다.</li>
<li><code>ROWS</code> 또는 <code>RANGE</code>: 현재 행이 참고할 범위를 정한다.</li>
</ul>
<h3 id="132-학생별-점수와-전체-평균을-함께-표시하기">13.2 학생별 점수와 전체 평균을 함께 표시하기</h3>
<pre><code class="language-sql">SELECT
    student_id,
    offering_id,
    score,
    ROUND(AVG(score) OVER (), 2) AS overall_average,
    ROUND(score - AVG(score) OVER (), 2) AS difference_from_average
FROM academic.enrollments
WHERE score IS NOT NULL
ORDER BY score DESC;</code></pre>
<p>각 수강신청 행은 유지되면서 모든 행에 전체 평균이 함께 표시된다.</p>
<h3 id="133-row_number-rank-dense_rank">13.3 ROW_NUMBER, RANK, DENSE_RANK</h3>
<table>
<thead>
<tr>
<th>함수</th>
<th>동점 처리</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>ROW_NUMBER</code></td>
<td>동점이어도 고유 번호</td>
<td>1, 2, 3, 4</td>
</tr>
<tr>
<td><code>RANK</code></td>
<td>동점은 같은 순위, 다음 순위 건너뜀</td>
<td>1, 2, 2, 4</td>
</tr>
<tr>
<td><code>DENSE_RANK</code></td>
<td>동점은 같은 순위, 다음 순위 연속</td>
<td>1, 2, 2, 3</td>
</tr>
</tbody></table>
<p>학과별 학생 평균 성적 순위를 계산한다.</p>
<pre><code class="language-sql">WITH student_scores AS (
    SELECT
        d.department_name,
        s.student_id,
        s.student_no,
        s.student_name,
        AVG(e.score) AS average_score
    FROM academic.students AS s
    JOIN academic.departments AS d
        ON d.department_id = s.department_id
    LEFT JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    GROUP BY
        d.department_id,
        d.department_name,
        s.student_id,
        s.student_no,
        s.student_name
)
SELECT
    department_name,
    student_no,
    student_name,
    ROUND(average_score, 2) AS average_score,
    ROW_NUMBER() OVER (
        PARTITION BY department_name
        ORDER BY average_score DESC NULLS LAST, student_id
    ) AS row_number,
    RANK() OVER (
        PARTITION BY department_name
        ORDER BY average_score DESC NULLS LAST
    ) AS rank,
    DENSE_RANK() OVER (
        PARTITION BY department_name
        ORDER BY average_score DESC NULLS LAST
    ) AS dense_rank,
    COUNT(*) OVER (
        PARTITION BY department_name
    ) AS total_in_department
FROM student_scores
ORDER BY department_name, row_number;</code></pre>
<p><code>ROW_NUMBER</code>에는 <code>student_id</code>를 두 번째 정렬 기준으로 넣었다. 점수가 같아도 실행할 때마다 순서가 흔들리지 않도록 하는 결정적 정렬 기준이다.</p>
<h3 id="134-순위-상위-3명만-남기기">13.4 순위 상위 3명만 남기기</h3>
<p>윈도우 함수는 논리적으로 <code>WHERE</code>보다 나중에 계산되므로 다음처럼 직접 쓸 수 없다.</p>
<pre><code class="language-sql">-- 오류가 나는 형태
SELECT ...
FROM ...
WHERE ROW_NUMBER() OVER (...) &lt;= 3;</code></pre>
<p>먼저 CTE나 인라인 뷰에서 순위를 만들고 바깥 쿼리에서 거른다.</p>
<pre><code class="language-sql">WITH ranked_students AS (
    SELECT
        d.department_name,
        s.student_id,
        s.student_no,
        s.student_name,
        AVG(e.score) AS average_score,
        ROW_NUMBER() OVER (
            PARTITION BY d.department_id
            ORDER BY AVG(e.score) DESC NULLS LAST, s.student_id
        ) AS department_rank
    FROM academic.students AS s
    JOIN academic.departments AS d
        ON d.department_id = s.department_id
    LEFT JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    GROUP BY
        d.department_id,
        d.department_name,
        s.student_id,
        s.student_no,
        s.student_name
)
SELECT
    department_name,
    student_no,
    student_name,
    ROUND(average_score, 2) AS average_score,
    department_rank
FROM ranked_students
WHERE department_rank &lt;= 3
ORDER BY department_name, department_rank;</code></pre>
<h3 id="135-lag와-lead-이전·다음-행과-비교하기">13.5 LAG와 LEAD: 이전·다음 행과 비교하기</h3>
<p>학생별로 과목 순서상 이전 점수를 가져온다.</p>
<pre><code class="language-sql">WITH score_history AS (
    SELECT
        e.student_id,
        s.student_no,
        s.student_name,
        c.course_code,
        e.score,
        LAG(e.score) OVER (
            PARTITION BY e.student_id
            ORDER BY c.course_code, e.offering_id
        ) AS previous_score,
        LEAD(e.score) OVER (
            PARTITION BY e.student_id
            ORDER BY c.course_code, e.offering_id
        ) AS next_score
    FROM academic.enrollments AS e
    JOIN academic.students AS s
        ON s.student_id = e.student_id
    JOIN academic.course_offerings AS o
        ON o.offering_id = e.offering_id
    JOIN academic.courses AS c
        ON c.course_id = o.course_id
    WHERE e.score IS NOT NULL
)
SELECT
    student_no,
    student_name,
    course_code,
    score,
    previous_score,
    score - previous_score AS score_difference,
    CASE
        WHEN previous_score IS NULL THEN '비교 대상 없음'
        WHEN score &gt; previous_score THEN '상승'
        WHEN score = previous_score THEN '유지'
        ELSE '하락'
    END AS change_status,
    next_score
FROM score_history
ORDER BY student_no, course_code;</code></pre>
<p>실제 시간 흐름을 분석한다면 과목 코드보다 학기, 시험일, 이벤트 시각 같은 시간 컬럼으로 정렬해야 한다.</p>
<h3 id="136-누적합">13.6 누적합</h3>
<p>신청일 순서로 누적 수강신청 수를 계산한다.</p>
<pre><code class="language-sql">WITH daily_enrollments AS (
    SELECT
        enrolled_at,
        COUNT(*) AS daily_count
    FROM academic.enrollments
    GROUP BY enrolled_at
)
SELECT
    enrolled_at,
    daily_count,
    SUM(daily_count) OVER (
        ORDER BY enrolled_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_count
FROM daily_enrollments
ORDER BY enrolled_at;</code></pre>
<h3 id="137-이동평균">13.7 이동평균</h3>
<p>최근 3개 신청일의 평균 신청 건수를 구한다.</p>
<pre><code class="language-sql">WITH daily_enrollments AS (
    SELECT
        enrolled_at,
        COUNT(*) AS daily_count
    FROM academic.enrollments
    GROUP BY enrolled_at
)
SELECT
    enrolled_at,
    daily_count,
    ROUND(AVG(daily_count) OVER (
        ORDER BY enrolled_at
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_average_3_rows
FROM daily_enrollments
ORDER BY enrolled_at;</code></pre>
<p>초반 두 행은 이전 행이 충분하지 않으므로 1개 또는 2개 행만으로 평균을 계산한다.</p>
<h3 id="138-rows와-range의-차이">13.8 ROWS와 RANGE의 차이</h3>
<table>
<thead>
<tr>
<th>프레임</th>
<th>기준</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>ROWS</code></td>
<td>물리적인 행 수</td>
<td>정확히 앞의 N행을 포함</td>
</tr>
<tr>
<td><code>RANGE</code></td>
<td><code>ORDER BY</code> 값의 범위</td>
<td>같은 정렬값의 동료 행을 함께 포함할 수 있음</td>
</tr>
</tbody></table>
<pre><code class="language-sql">-- 현재 행을 포함한 최근 3행
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

-- 현재 날짜를 포함한 최근 3일 범위
RANGE BETWEEN INTERVAL '2 days' PRECEDING AND CURRENT ROW</code></pre>
<h3 id="139-first_value와-last_value의-프레임-주의">13.9 FIRST_VALUE와 LAST_VALUE의 프레임 주의</h3>
<p><code>LAST_VALUE</code>는 기본 프레임에서 파티션의 진짜 마지막 값이 아니라 <strong>현재 프레임의 마지막 값</strong>을 돌려줄 수 있다. 파티션 전체의 마지막 값을 원한다면 프레임을 명시한다.</p>
<pre><code class="language-sql">SELECT
    student_id,
    offering_id,
    score,
    FIRST_VALUE(score) OVER (
        PARTITION BY student_id
        ORDER BY offering_id
    ) AS first_score,
    LAST_VALUE(score) OVER (
        PARTITION BY student_id
        ORDER BY offering_id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND UNBOUNDED FOLLOWING
    ) AS last_score
FROM academic.enrollments
WHERE score IS NOT NULL;</code></pre>
<h3 id="1310-ntile-percent_rank-cume_dist">13.10 NTILE, PERCENT_RANK, CUME_DIST</h3>
<pre><code class="language-sql">SELECT
    student_id,
    offering_id,
    score,
    NTILE(4) OVER (
        ORDER BY score DESC
    ) AS quartile,
    PERCENT_RANK() OVER (
        ORDER BY score DESC
    ) AS percent_rank,
    CUME_DIST() OVER (
        ORDER BY score DESC
    ) AS cumulative_distribution
FROM academic.enrollments
WHERE score IS NOT NULL
ORDER BY score DESC;</code></pre>
<ul>
<li><code>NTILE(4)</code>: 행을 네 그룹으로 나눈다.</li>
<li><code>PERCENT_RANK</code>: 순위를 0~1 범위로 표현한다.</li>
<li><code>CUME_DIST</code>: 현재 값까지 포함된 행의 누적 비율을 구한다.</li>
</ul>
<p>표본이 적을 때는 분위 구간의 크기가 균등하지 않을 수 있으므로 결과를 그대로 등급으로 단정하지 않는다.</p>
<hr />
<h2 id="14-실무에서-자주-쓰는-고급-패턴">14. 실무에서 자주 쓰는 고급 패턴</h2>
<h3 id="141-학과별-상위-n명-window-function">14.1 학과별 상위 N명: Window Function</h3>
<p>그룹마다 상위 N개를 찾는 문제를 <strong>Top-N per group</strong>이라고 한다. 가장 일반적인 해결법은 <code>ROW_NUMBER</code>다.</p>
<pre><code class="language-sql">WITH student_average AS (
    SELECT
        s.student_id,
        s.department_id,
        s.student_name,
        AVG(e.score) AS average_score
    FROM academic.students AS s
    LEFT JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    GROUP BY
        s.student_id,
        s.department_id,
        s.student_name
),
ranked AS (
    SELECT
        sa.*,
        ROW_NUMBER() OVER (
            PARTITION BY sa.department_id
            ORDER BY sa.average_score DESC NULLS LAST,
                     sa.student_id
        ) AS row_number
    FROM student_average AS sa
)
SELECT
    d.department_name,
    r.student_name,
    ROUND(r.average_score, 2) AS average_score,
    r.row_number
FROM ranked AS r
JOIN academic.departments AS d
    ON d.department_id = r.department_id
WHERE r.row_number &lt;= 3
ORDER BY d.department_name, r.row_number;</code></pre>
<h3 id="142-lateral-join-왼쪽-행마다-제한된-결과-가져오기">14.2 LATERAL JOIN: 왼쪽 행마다 제한된 결과 가져오기</h3>
<p>학과별 학생 평균 성적 상위 2명을 가져온다.</p>
<pre><code class="language-sql">SELECT
    d.department_name,
    top_students.student_name,
    ROUND(top_students.average_score, 2) AS average_score
FROM academic.departments AS d
CROSS JOIN LATERAL (
    SELECT
        s.student_name,
        AVG(e.score) AS average_score
    FROM academic.students AS s
    LEFT JOIN academic.enrollments AS e
        ON e.student_id = s.student_id
    WHERE s.department_id = d.department_id
    GROUP BY s.student_id, s.student_name
    ORDER BY AVG(e.score) DESC NULLS LAST, s.student_id
    LIMIT 2
) AS top_students
ORDER BY d.department_name, top_students.average_score DESC NULLS LAST;</code></pre>
<p><code>LATERAL</code> 안의 서브쿼리는 왼쪽의 현재 학과 <code>d.department_id</code>를 참조할 수 있다.</p>
<h3 id="143-upsert-없으면-insert-있으면-update">14.3 UPSERT: 없으면 INSERT, 있으면 UPDATE</h3>
<p>다음 통계 테이블이 있다고 가정한다.</p>
<pre><code class="language-sql">CREATE TABLE IF NOT EXISTS academic.student_login_stats (
    student_id BIGINT PRIMARY KEY
        REFERENCES academic.students(student_id),
    login_count INTEGER NOT NULL DEFAULT 0,
    last_login_at TIMESTAMPTZ NOT NULL
);</code></pre>
<p>로그인 통계를 한 문장으로 추가하거나 갱신한다.</p>
<pre><code class="language-sql">INSERT INTO academic.student_login_stats (
    student_id,
    login_count,
    last_login_at
)
VALUES (1, 1, CURRENT_TIMESTAMP)
ON CONFLICT (student_id)
DO UPDATE SET
    login_count = academic.student_login_stats.login_count
                  + EXCLUDED.login_count,
    last_login_at = EXCLUDED.last_login_at;</code></pre>
<p><code>EXCLUDED</code>는 충돌로 인해 새로 삽입되지 못한 입력값을 가리킨다.</p>
<h3 id="144-string_agg와-json_agg">14.4 STRING_AGG와 JSON_AGG</h3>
<p>학생별 수강 과목을 한 문자열로 묶는다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    STRING_AGG(
        c.course_title,
        ', ' ORDER BY c.course_title
    ) AS course_titles
FROM academic.students AS s
JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
JOIN academic.courses AS c
    ON c.course_id = o.course_id
GROUP BY s.student_id, s.student_no, s.student_name
ORDER BY s.student_no;</code></pre>
<p>API에서 사용할 수 있는 JSON 배열 형태로도 집계할 수 있다.</p>
<pre><code class="language-sql">SELECT
    s.student_no,
    s.student_name,
    JSONB_AGG(
        JSONB_BUILD_OBJECT(
            'courseCode', c.course_code,
            'courseTitle', c.course_title,
            'score', e.score,
            'status', e.enrollment_status
        )
        ORDER BY c.course_code
    ) AS enrollments
FROM academic.students AS s
JOIN academic.enrollments AS e
    ON e.student_id = s.student_id
JOIN academic.course_offerings AS o
    ON o.offering_id = e.offering_id
JOIN academic.courses AS c
    ON c.course_id = o.course_id
GROUP BY s.student_id, s.student_no, s.student_name
ORDER BY s.student_no;</code></pre>
<p>DB에서 JSON을 만들 수 있다고 해서 모든 API 응답 조립을 DB가 담당해야 하는 것은 아니다. 재사용성, 응답 크기, 애플리케이션 책임을 함께 고려한다.</p>
<h3 id="145-pivot-행을-열로-바꾸기">14.5 PIVOT: 행을 열로 바꾸기</h3>
<p>PostgreSQL에서는 조건부 집계를 사용해 상태별 건수를 열로 펼칠 수 있다.</p>
<pre><code class="language-sql">SELECT
    c.course_code,
    COUNT(*) FILTER (
        WHERE e.enrollment_status = 'ENROLLED'
    ) AS enrolled_count,
    COUNT(*) FILTER (
        WHERE e.enrollment_status = 'COMPLETED'
    ) AS completed_count,
    COUNT(*) FILTER (
        WHERE e.enrollment_status = 'DROPPED'
    ) AS dropped_count
FROM academic.courses AS c
JOIN academic.course_offerings AS o
    ON o.course_id = c.course_id
LEFT JOIN academic.enrollments AS e
    ON e.offering_id = o.offering_id
GROUP BY c.course_id, c.course_code
ORDER BY c.course_code;</code></pre>
<h3 id="146-날짜-처리">14.6 날짜 처리</h3>
<p>월별 수강신청 건수를 집계한다.</p>
<pre><code class="language-sql">SELECT
    DATE_TRUNC('month', enrolled_at) AS enrollment_month,
    COUNT(*) AS enrollment_count
FROM academic.enrollments
GROUP BY DATE_TRUNC('month', enrolled_at)
ORDER BY enrollment_month;</code></pre>
<p>최근 30일처럼 현재 시점 기준 범위를 조회할 수도 있다.</p>
<pre><code class="language-sql">SELECT
    student_id,
    offering_id,
    enrolled_at
FROM academic.enrollments
WHERE enrolled_at &gt;= CURRENT_DATE - INTERVAL '30 days'
ORDER BY enrolled_at DESC;</code></pre>
<p>시간대가 있는 <code>TIMESTAMPTZ</code>를 화면용 한국 시간으로 표현하는 예는 다음과 같다.</p>
<pre><code class="language-sql">SELECT
    student_id,
    created_at,
    created_at AT TIME ZONE 'Asia/Seoul' AS created_at_kst
FROM academic.enrollments;</code></pre>
<p><code>TIMESTAMPTZ</code>는 절대 시점을 저장하고 세션 시간대에 맞게 표시한다. <code>AT TIME ZONE</code> 결과의 타입과 의미가 입력 타입에 따라 달라지므로 저장 정책과 표시 정책을 구분한다.</p>
<h3 id="147-문자열-처리">14.7 문자열 처리</h3>
<p>이메일에서 도메인을 추출한다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    email,
    SPLIT_PART(email, '@', 2) AS email_domain
FROM academic.students
ORDER BY email_domain, student_no;</code></pre>
<p>전화번호에서 숫자만 남길 수 있다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    phone,
    REGEXP_REPLACE(phone, '[^0-9]', '', 'g') AS normalized_phone
FROM academic.students
WHERE phone IS NOT NULL;</code></pre>
<p>개인정보를 보고서에 표시해야 한다면 원본을 직접 노출하기보다 필요한 범위만 마스킹한다.</p>
<pre><code class="language-sql">SELECT
    student_no,
    LEFT(student_name, 1)
        || REPEAT('*', GREATEST(LENGTH(student_name) - 2, 0))
        || CASE
            WHEN LENGTH(student_name) &gt;= 2
            THEN RIGHT(student_name, 1)
            ELSE ''
           END AS masked_name
FROM academic.students;</code></pre>
<p>마스킹은 화면 표시를 줄일 뿐 원본 접근 권한을 대체하지 않는다. 데이터베이스 권한, View, 감사 로그와 함께 설계한다.</p>
<h3 id="148-gap-and-island-연속-구간-찾기">14.8 Gap and Island: 연속 구간 찾기</h3>
<p>연속 로그인 날짜처럼 붙어 있는 기간을 찾는 문제를 Gap and Island라고 한다.</p>
<pre><code class="language-sql">WITH login_dates AS (
    SELECT DISTINCT
        student_id,
        login_at::DATE AS login_date
    FROM academic.student_login_history
),
grouped AS (
    SELECT
        student_id,
        login_date,
        login_date
        - ROW_NUMBER() OVER (
            PARTITION BY student_id
            ORDER BY login_date
        )::INTEGER AS island_key
    FROM login_dates
)
SELECT
    student_id,
    MIN(login_date) AS streak_start,
    MAX(login_date) AS streak_end,
    COUNT(*) AS consecutive_days
FROM grouped
GROUP BY student_id, island_key
ORDER BY student_id, streak_start;</code></pre>
<p>연속 날짜에서 <code>ROW_NUMBER</code>를 빼면 같은 연속 구간은 동일한 키가 된다는 원리를 이용한다. 이 예시는 <code>student_login_history</code> 테이블이 별도로 있을 때 실행할 수 있다.</p>
<hr />
<h2 id="15-트랜잭션과-savepoint-복습">15. 트랜잭션과 SAVEPOINT 복습</h2>
<p>여러 변경을 하나의 작업으로 보호하려면 트랜잭션을 사용한다.</p>
<pre><code class="language-sql">BEGIN;

UPDATE academic.enrollments
SET enrollment_status = 'DROPPED'
WHERE student_id = 1
  AND offering_id = 3;

-- 결과 확인
SELECT
    student_id,
    offering_id,
    enrollment_status
FROM academic.enrollments
WHERE student_id = 1
  AND offering_id = 3;

ROLLBACK;</code></pre>
<p><code>ROLLBACK</code>했으므로 변경은 취소된다.</p>
<p>작업 중간에 복구 지점을 만들 수도 있다.</p>
<pre><code class="language-sql">BEGIN;

UPDATE academic.students
SET phone = '010-9999-9999'
WHERE student_no = '2026001';

SAVEPOINT phone_updated;

UPDATE academic.students
SET grade_level = 5
WHERE student_no = '2026001';

-- CHECK 제약조건 오류가 발생했다면 중간 지점으로 되돌린다.
ROLLBACK TO SAVEPOINT phone_updated;

COMMIT;</code></pre>
<p>오류가 발생한 PostgreSQL 트랜잭션은 롤백하기 전까지 실패 상태가 될 수 있다. DBeaver에서 자동 커밋 설정과 현재 트랜잭션 상태를 확인하는 습관이 필요하다.</p>
<hr />
<h2 id="16-sql-성능을-위한-기본-습관">16. SQL 성능을 위한 기본 습관</h2>
<h3 id="161-select--대신-필요한-컬럼만-선택한다">16.1 SELECT * 대신 필요한 컬럼만 선택한다</h3>
<pre><code class="language-sql">-- 필요한 데이터보다 훨씬 많이 읽을 수 있다.
SELECT *
FROM academic.students;

-- 화면과 로직에 필요한 컬럼만 선택한다.
SELECT
    student_id,
    student_no,
    student_name
FROM academic.students;</code></pre>
<h3 id="162-인덱스-컬럼을-함수로-감싸는-조건을-주의한다">16.2 인덱스 컬럼을 함수로 감싸는 조건을 주의한다</h3>
<pre><code class="language-sql">-- 컬럼에 함수를 적용한 형태
WHERE EXTRACT(YEAR FROM enrolled_at) = 2026

-- 범위로 표현한 형태
WHERE enrolled_at &gt;= DATE '2026-01-01'
  AND enrolled_at &lt;  DATE '2027-01-01'</code></pre>
<p>두 번째 형태는 <code>enrolled_at</code>의 일반 B-Tree 인덱스를 활용하기 쉬운 조건이다.</p>
<h3 id="163-n1-쿼리를-피한다">16.3 N+1 쿼리를 피한다</h3>
<p>학생 목록 12건을 조회한 뒤 애플리케이션 반복문에서 학생마다 수강신청 쿼리를 한 번씩 실행하면 총 13번의 쿼리가 발생한다.</p>
<pre><code class="language-text">학생 목록 1번
+ 학생별 수강신청 N번
= N+1 문제</code></pre>
<p>한 번의 <code>JOIN</code>, 필요한 키 목록을 사용한 일괄 조회, ORM의 적절한 로딩 전략으로 줄일 수 있다.</p>
<h3 id="164-distinct로-원인을-숨기지-않는다">16.4 DISTINCT로 원인을 숨기지 않는다</h3>
<p>중복이 나타났을 때 무조건 <code>DISTINCT</code>를 붙이기 전에 확인한다.</p>
<ul>
<li>1:N 관계 때문에 정상적으로 행이 늘었는가?</li>
<li><code>JOIN</code> 조건이 빠졌는가?</li>
<li>필요한 단위보다 세부 테이블을 먼저 연결했는가?</li>
<li>먼저 집계한 뒤 연결해야 하는가?</li>
</ul>
<h3 id="165-큰-offset을-피한다">16.5 큰 OFFSET을 피한다</h3>
<p>무한 스크롤이나 다음 페이지 방식에는 Keyset 페이지네이션을 검토한다.</p>
<h3 id="166-예상-행-수와-실제-행-수를-비교한다">16.6 예상 행 수와 실제 행 수를 비교한다</h3>
<p><code>EXPLAIN (ANALYZE, BUFFERS)</code>에서 예상 <code>rows</code>와 <code>actual rows</code> 차이가 매우 크면 통계나 데이터 분포를 점검한다.</p>
<pre><code class="language-sql">ANALYZE academic.students;
ANALYZE academic.enrollments;</code></pre>
<hr />
<h2 id="17-ai-시대의-데이터베이스-활용-맛보기">17. AI 시대의 데이터베이스 활용 맛보기</h2>
<h3 id="171-jsonb">17.1 JSONB</h3>
<p>상품처럼 속성이 자주 달라지는 일부 데이터를 JSONB로 저장할 수 있다.</p>
<pre><code class="language-sql">CREATE TABLE product_catalog (
    product_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_name TEXT NOT NULL,
    attributes JSONB NOT NULL DEFAULT '{}'::JSONB
);

INSERT INTO product_catalog (product_name, attributes)
VALUES (
    '노트북 A',
    '{&quot;color&quot;:&quot;silver&quot;,&quot;memoryGB&quot;:16,&quot;storageGB&quot;:512}'
);

SELECT
    product_name,
    attributes -&gt;&gt; 'color' AS color,
    (attributes -&gt;&gt; 'memoryGB')::INTEGER AS memory_gb
FROM product_catalog
WHERE attributes @&gt; '{&quot;color&quot;:&quot;silver&quot;}';</code></pre>
<p>JSONB는 유연하지만 모든 구조를 JSON으로 넣으면 외래 키, 타입, 정규화의 장점을 잃을 수 있다. 관계와 규칙이 분명한 핵심 데이터는 일반 컬럼과 테이블을 우선한다.</p>
<h3 id="172-벡터-검색과-rag">17.2 벡터 검색과 RAG</h3>
<p>문장을 임베딩이라는 숫자 벡터로 변환하면 의미가 가까운 문서를 검색할 수 있다. PostgreSQL에서는 벡터 확장을 사용해 관계형 조건과 벡터 유사도 검색을 함께 구성할 수 있다.</p>
<pre><code class="language-text">사용자 질문
  → 질문 임베딩 생성
  → 유사한 문서 조각 검색
  → 검색 결과를 LLM 문맥으로 전달
  → 근거를 바탕으로 답변 생성</code></pre>
<p>이 과정을 RAG라고 한다. 실무에서는 다음을 함께 설계해야 한다.</p>
<ul>
<li>문서를 어떤 단위로 나눌지</li>
<li>벡터 검색과 키워드 검색을 어떻게 조합할지</li>
<li>사용자 권한에 따라 검색 결과를 어떻게 제한할지</li>
<li>임베딩 모델 변경 시 기존 벡터를 어떻게 갱신할지</li>
<li>답변과 검색 근거를 어떻게 추적할지</li>
</ul>
<p>AI가 SQL을 작성해 주더라도 결과의 정확성, 권한, 개인정보, 실행 비용은 사람이 검증해야 한다.</p>
<hr />
<h2 id="18-어떤-문법을-선택해야-할까">18. 어떤 문법을 선택해야 할까?</h2>
<table>
<thead>
<tr>
<th>해결하려는 문제</th>
<th>우선 고려할 문법</th>
</tr>
</thead>
<tbody><tr>
<td>전체 또는 그룹별 통계</td>
<td>집계 함수 + <code>GROUP BY</code></td>
</tr>
<tr>
<td>집계 결과 필터</td>
<td><code>HAVING</code></td>
</tr>
<tr>
<td>조건별 개수를 열로 표시</td>
<td><code>FILTER</code> 또는 <code>SUM(CASE ...)</code></td>
</tr>
<tr>
<td>여러 테이블 컬럼 함께 조회</td>
<td><code>JOIN</code></td>
</tr>
<tr>
<td>관련 행 존재 여부</td>
<td><code>EXISTS</code></td>
</tr>
<tr>
<td>관련 행 부재 여부</td>
<td><code>NOT EXISTS</code></td>
</tr>
<tr>
<td>간단한 단일값 비교</td>
<td>스칼라 서브쿼리</td>
</tr>
<tr>
<td>복잡한 쿼리 단계 분리</td>
<td>CTE</td>
</tr>
<tr>
<td>공통 조회 재사용</td>
<td>View</td>
</tr>
<tr>
<td>무거운 결과를 저장해 빠르게 조회</td>
<td>Materialized View</td>
</tr>
<tr>
<td>원래 행을 유지한 순위·누적·이전값</td>
<td>Window Function</td>
</tr>
<tr>
<td>그룹별 상위 N개</td>
<td><code>ROW_NUMBER</code> + CTE</td>
</tr>
<tr>
<td>조직도·카테고리 트리</td>
<td>재귀 CTE</td>
</tr>
<tr>
<td>결과 두 개의 합집합·교집합·차집합</td>
<td>집합 연산자</td>
</tr>
</tbody></table>
<p>문법을 많이 쓰는 쿼리가 좋은 쿼리는 아니다. 질문의 의미가 분명하고, 정확한 결과를 내며, 다른 사람이 읽고 수정하기 쉬운 쿼리가 좋은 쿼리다.</p>
<hr />
<h2 id="19-초보자가-자주-하는-오해">19. 초보자가 자주 하는 오해</h2>
<h3 id="group-by를-쓰면-select에-아무-컬럼이나-넣어도-된다">“GROUP BY를 쓰면 SELECT에 아무 컬럼이나 넣어도 된다”</h3>
<p>아니다. 그룹 기준 컬럼이나 집계된 컬럼을 사용해야 한다.</p>
<h3 id="where와-having은-위치만-다르고-같다">“WHERE와 HAVING은 위치만 다르고 같다”</h3>
<p>아니다. <code>WHERE</code>는 행을 먼저 줄이고, <code>HAVING</code>은 만들어진 그룹을 거른다.</p>
<h3 id="left-join을-쓰면-왼쪽-행은-항상-남는다">“LEFT JOIN을 쓰면 왼쪽 행은 항상 남는다”</h3>
<p>오른쪽 테이블 조건을 <code>WHERE</code>에 쓰면 <code>NULL</code> 행이 제거되어 왼쪽 행이 사라질 수 있다.</p>
<h3 id="join-결과가-많으면-distinct를-붙이면-된다">“JOIN 결과가 많으면 DISTINCT를 붙이면 된다”</h3>
<p>먼저 1:N 관계, 누락된 연결 조건, 잘못된 집계 단위를 확인해야 한다.</p>
<h3 id="cte를-사용하면-무조건-빨라진다">“CTE를 사용하면 무조건 빨라진다”</h3>
<p>CTE의 가장 확실한 장점은 가독성과 구조화다. 성능은 PostgreSQL 버전, 참조 횟수, 쿼리 내용과 실행계획에 따라 달라진다.</p>
<h3 id="window-function은-group-by의-더-좋은-버전이다">“Window Function은 GROUP BY의 더 좋은 버전이다”</h3>
<p>역할이 다르다. 그룹별 한 행이 필요하면 <code>GROUP BY</code>, 개별 행과 분석값을 함께 보고 싶으면 Window Function이 적합하다.</p>
<h3 id="explain-analyze는-결과를-보여주지-않으니-실행도-안-한다">“EXPLAIN ANALYZE는 결과를 보여주지 않으니 실행도 안 한다”</h3>
<p><code>ANALYZE</code>가 붙으면 실제로 실행한다. 변경 쿼리에서는 특히 주의한다.</p>
<hr />
<h2 id="20-핵심-정리">20. 핵심 정리</h2>
<ul>
<li>집계 함수는 여러 행을 개수, 합계, 평균, 최솟값, 최댓값으로 요약한다.</li>
<li><code>COUNT(*)</code>는 행을 세고 <code>COUNT(column)</code>은 <code>NULL</code>이 아닌 값만 센다.</li>
<li><code>GROUP BY</code>는 행을 그룹으로 묶고 <code>HAVING</code>은 집계된 그룹을 필터링한다.</li>
<li>조건부 집계는 PostgreSQL의 <code>FILTER</code> 또는 <code>CASE WHEN</code>으로 표현할 수 있다.</li>
<li><code>ROLLUP</code>은 계층적 소계와 총계를, <code>CUBE</code>는 가능한 차원 조합의 소계를 만든다.</li>
<li><code>GROUPING</code>은 원본의 <code>NULL</code>과 집계가 만든 소계용 <code>NULL</code>을 구분한다.</li>
<li>Keyset 페이지네이션은 적절한 인덱스와 안정적인 정렬 키가 있을 때 큰 목록의 다음 페이지 조회에 유리하다.</li>
<li><code>INNER JOIN</code>은 양쪽 일치 행을, <code>LEFT JOIN</code>은 왼쪽 전체를 보존한다.</li>
<li><code>NOT EXISTS</code>는 관련 데이터가 없는 행을, <code>EXISTS</code>는 관련 데이터가 있는 행을 찾는 데 적합하다.</li>
<li><code>LEFT JOIN</code> 뒤 오른쪽 테이블의 조건을 <code>WHERE</code>에 쓰면 의도치 않게 행이 제거될 수 있다.</li>
<li>1:N 조인에서는 행이 늘어나므로 무엇을 세는지에 따라 <code>COUNT</code>, <code>COUNT(column)</code>, <code>COUNT(DISTINCT ...)</code>를 구분한다.</li>
<li>PostgreSQL 옵티마이저는 Nested Loop, Hash, Merge Join 등의 물리 알고리즘을 선택한다.</li>
<li><code>EXPLAIN</code>은 계획을 보고 <code>EXPLAIN ANALYZE</code>는 실제 실행 정보까지 측정한다.</li>
<li>서브쿼리는 다른 쿼리의 결과를 값, 목록, 존재 여부 또는 임시 결과로 사용한다.</li>
<li><code>UNION</code>, <code>INTERSECT</code>, <code>EXCEPT</code>는 행 집합의 합집합, 교집합, 차집합을 만든다.</li>
<li>CTE는 긴 쿼리를 단계별로 나누고 재귀 CTE는 계층 구조를 탐색한다.</li>
<li>View는 공통 조회를 재사용하고 Materialized View는 결과를 저장해 반복 조회 비용을 줄인다.</li>
<li>Window Function은 원래 행을 유지하면서 순위, 이전값, 누적합, 이동평균을 계산한다.</li>
<li>윈도우 함수의 결과를 필터링하려면 CTE나 인라인 뷰에서 먼저 계산한다.</li>
<li><code>ROWS</code>는 행 수 기준, <code>RANGE</code>는 정렬값 범위 기준 프레임이다.</li>
<li>좋은 SQL은 정확성, 가독성, 성능과 업무 의미를 함께 만족해야 한다.</li>
</ul>
<hr />
<h2 id="마무리">마무리</h2>
<p>처음에는 <code>JOIN</code>, 서브쿼리, CTE, Window Function이 서로 비슷해 보일 수 있다. 하지만 각 기능은 해결하는 문제가 다르다.</p>
<pre><code class="language-text">테이블을 연결한다        → JOIN
다른 쿼리의 결과를 쓴다 → 서브쿼리
복잡한 단계를 나눈다    → CTE
공통 조회를 재사용한다  → View
행을 유지하며 분석한다  → Window Function</code></pre>
<p>문법을 외우기보다 “내가 원하는 결과의 행 단위가 무엇인가?”를 먼저 질문하면 SQL을 훨씬 쉽게 설계할 수 있다.</p>