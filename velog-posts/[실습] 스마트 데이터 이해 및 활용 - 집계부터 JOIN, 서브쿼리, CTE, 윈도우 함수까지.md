<h3 id="date">Date</h3>
<p>2026-07-28</p>
<h2 id="스키마-요약">스키마 요약</h2>
<table>
<thead>
<tr>
<th>테이블</th>
<th>컬럼</th>
</tr>
</thead>
<tbody><tr>
<td><code>lab.student</code></td>
<td>student_id, name, major, gpa</td>
</tr>
<tr>
<td><code>lab.enroll</code></td>
<td>student_id, course, grade</td>
</tr>
<tr>
<td><code>lab.customers</code></td>
<td>customer_id, customer_name</td>
</tr>
<tr>
<td><code>lab.orders</code></td>
<td>order_id, customer_id, amount</td>
</tr>
<tr>
<td><code>lab.emp</code></td>
<td>emp_id, name, manager_id</td>
</tr>
</tbody></table>
<blockquote>
<p>[!info] 관계</p>
<ul>
<li>student 1:N enroll</li>
<li>customers 1:N orders</li>
<li>emp 1:N emp (Self, manager_id → emp_id)</li>
</ul>
</blockquote>
<h2 id="16은-limit-5-설정">1~6은 limit 5 설정</h2>
<h2 id="1-inner-join--수강-존재-학생의-과목성적-조회">1. INNER JOIN — 수강 존재 학생의 과목/성적 조회</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    e.course,
    e.grade
FROM lab.student AS s
INNER JOIN lab.enroll AS e
    ON s.student_id = e.student_id
    LIMIT 5;</code></pre>
<h2 id="2-left-join--모든-학생-기준-과목-없으면-null">2. LEFT JOIN — 모든 학생 기준, 과목 없으면 NULL</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    e.course,
    e.grade
FROM lab.student AS s
LEFT JOIN lab.enroll AS e
    ON s.student_id = e.student_id
    LIMIT 5;</code></pre>
<h2 id="3-right-join--수강-기준-학생-없으면-null">3. RIGHT JOIN — 수강 기준, 학생 없으면 NULL</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    e.course,
    e.grade
FROM lab.student AS s
RIGHT JOIN lab.enroll AS e
    ON s.student_id = e.student_id
    LIMIT 5;</code></pre>
<h2 id="4-full-outer-join--학생수강-모두-포함">4. FULL OUTER JOIN — 학생/수강 모두 포함</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    e.course,
    e.grade
FROM lab.student AS s
FULL OUTER JOIN lab.enroll AS e
    ON s.student_id = e.student_id
LIMIT 5;</code></pre>
<h2 id="5-한-번도-수강하지-않은-학생-목록-left-join-anti">5. 한 번도 수강하지 않은 학생 목록 (LEFT JOIN Anti)</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    s.major,
    s.gpa
FROM lab.student AS s
LEFT JOIN lab.enroll AS e
    ON s.student_id = e.student_id
WHERE e.student_id IS NULL
LIMIT 5;</code></pre>
<h2 id="6-한-과목-이상-수강한-학생-목록-중복-제거">6. 한 과목 이상 수강한 학생 목록 (중복 제거)</h2>
<pre><code class="language-sql">SELECT DISTINCT
    s.student_id,
    s.name,
    s.major,
    s.gpa
FROM lab.student AS s
INNER JOIN lab.enroll AS e
    ON s.student_id = e.student_id
LIMIT 5;</code></pre>
<h2 id="7-고객별-주문건수--총액">7. 고객별 주문건수 / 총액</h2>
<pre><code class="language-sql">SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.amount), 0) AS total_amount
FROM lab.customers AS c
LEFT JOIN lab.orders AS o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;</code></pre>
<h2 id="8-총액-상위-10명과-금액">8. 총액 상위 10명과 금액</h2>
<pre><code class="language-sql">SELECT
    c.customer_id,
    c.customer_name,
    SUM(o.amount) AS total_amount
FROM lab.customers AS c
INNER JOIN lab.orders AS o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
ORDER BY total_amount DESC
LIMIT 10;</code></pre>
<h2 id="9-모든-직원과-그-매니저-이름-self-join">9. 모든 직원과 그 매니저 이름 (Self Join)</h2>
<pre><code class="language-sql">SELECT
    e.emp_id,
    e.name AS employee_name,
    m.name AS manager_name
FROM lab.emp AS e
LEFT JOIN lab.emp AS m
    ON e.manager_id = m.emp_id;</code></pre>
<h2 id="10-모든-학생-기준-과목-분포-left-join--집계">10. 모든 학생 기준 과목 분포 (LEFT JOIN + 집계)</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    COUNT(e.course) AS course_count
FROM lab.student AS s
LEFT JOIN lab.enroll AS e
    ON s.student_id = e.student_id
GROUP BY s.student_id, s.name;</code></pre>
<h2 id="11-db-과목을-듣지-않은-모든-학생-not-exists-">11. DB 과목을 듣지 않은 모든 학생 (NOT EXISTS )</h2>
<pre><code class="language-sql">SELECT s.student_id, s.name, s.major, s.gpa
FROM lab.student s
WHERE NOT EXISTS (
    SELECT 1
    FROM lab.enroll e
    WHERE e.student_id = s.student_id
      AND e.course = 'DB'
)
ORDER BY s.student_id;</code></pre>
<h2 id="12-과목별-수강-인원--책임-매니저-course_owner-매핑">12. 과목별 수강 인원 + 책임 매니저 (course_owner 매핑)</h2>
<pre><code class="language-sql">SELECT emp_id, name
FROM lab.emp
WHERE name LIKE 'Mgr_%'
ORDER BY emp_id;

INSERT INTO lab.course_owner (course, manager_id)
SELECT 'DB', emp_id FROM lab.emp WHERE name = 'Mgr_2'
UNION ALL
SELECT 'SQL', emp_id FROM lab.emp WHERE name = 'Mgr_3'
UNION ALL
SELECT 'Python', emp_id FROM lab.emp WHERE name = 'Mgr_4';


SELECT co.course,
m.name AS manager_name,
COUNT(e.student_id) AS student_count
FROM lab.course_owner AS co
LEFT JOIN lab.emp AS m ON co.manager_id = m.emp_id
LEFT JOIN lab.enroll AS e ON co.course = e.course
GROUP BY co.course, m.name;</code></pre>
<h2 id="13-학생-×-과목-전체-조합-cross-join-샘플-100건">13. 학생 × 과목 전체 조합 (CROSS JOIN, 샘플 100건)</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name,
    c.course
FROM lab.student AS s
CROSS JOIN (
    SELECT DISTINCT course
    FROM lab.enroll
) AS c
LIMIT 100;</code></pre>
<h2 id="14-스칼라-서브쿼리--학생--소속-학과명">14. 스칼라 서브쿼리 — 학생 + 소속 학과명</h2>
<pre><code class="language-sql">CREATE TABLE lab.department (
major VARCHAR(10),
major_name VARCHAR(50)
);
INSERT INTO lab.department (major, major_name)
VALUES
('BIO', '생명공학과'),
('CE', '컴퓨터공학과'),
('CS', '컴퓨터과학과'),
('EE', '전자공학과'),
('HR', '인사관리학과'),
('ME', '기계공학과');


SELECT
    s.student_id,
    s.name,
(
SELECT d.major_name
FROM lab.department AS d
WHERE d.major = s.major
) AS major_name,
    s.gpa
FROM lab.student AS s;</code></pre>
<h2 id="15-평균-gpa보다-높은-학생-where-서브쿼리">15. 평균 GPA보다 높은 학생 (WHERE 서브쿼리)</h2>
<pre><code class="language-sql">SELECT
    student_id,
    name,
    major,
    gpa
FROM lab.student
WHERE gpa &gt; (
    SELECT AVG(gpa)
    FROM lab.student
);</code></pre>
<h2 id="16-자신의-학과-평균-gpa보다-높은-학생-상관-서브쿼리">16. 자신의 학과 평균 GPA보다 높은 학생 (상관 서브쿼리)</h2>
<pre><code class="language-sql">SELECT
    s1.student_id,
    s1.name,
    s1.major,
    s1.gpa
FROM lab.student AS s1
WHERE s1.gpa &gt; (
    SELECT AVG(s2.gpa)
    FROM lab.student AS s2
    WHERE s2.major = s1.major
);</code></pre>
<h2 id="17-수강enroll-기록이-있는-학생만-exists">17. 수강(enroll) 기록이 있는 학생만 (EXISTS)</h2>
<pre><code class="language-sql">SELECT
    s.student_id,
    s.name
FROM lab.student AS s
WHERE EXISTS (
    SELECT 1
    FROM lab.enroll AS e
    WHERE e.student_id = s.student_id
);</code></pre>
<h2 id="18-한-번도-수강하지-않은-학생-not-exists-">18. 한 번도 수강하지 않은 학생 (NOT EXISTS )</h2>
<pre><code class="language-sql">SELECT s.student_id, s.name, s.major
FROM lab.student s
WHERE NOT EXISTS (
    SELECT 1 FROM lab.enroll e WHERE e.student_id = s.student_id
)
ORDER BY s.student_id;</code></pre>
<h2 id="19-hr-학과-학생-일부와의-gpa-비교-self-join">19. HR 학과 학생 일부와의 GPA 비교 (Self Join)</h2>
<pre><code class="language-sql">SELECT a.student_id AS &quot;HR_학생&quot;,
       a.gpa         AS &quot;HR_GPA&quot;,
       b.student_id  AS &quot;비교_학생&quot;,
       b.gpa         AS &quot;비교_GPA&quot;,
       b.major       AS &quot;전공&quot;
FROM lab.student a
JOIN lab.student b
     ON a.student_id &lt;&gt; b.student_id
    AND ABS(a.gpa - b.gpa) &lt; 0.1
WHERE a.major = 'HR'
  AND b.major &lt;&gt; 'HR'
  AND a.student_id IN (981, 985, 990)
ORDER BY a.student_id ASC, b.gpa DESC;</code></pre>
<h2 id="20-cs-학과-학생-또는-db-과목-수강한-학생-목록-union">20. CS 학과 학생 또는 DB 과목 수강한 학생 목록 (UNION)</h2>
<pre><code class="language-sql">SELECT student_id, name, major, gpa
FROM lab.student
WHERE major = 'CS'

UNION

SELECT s.student_id, s.name, s.major, s.gpa
FROM lab.student s
JOIN lab.enroll e ON s.student_id = e.student_id
WHERE e.course = 'DB'

ORDER BY student_id;</code></pre>
<h2 id="21-학과별·gpa-구간별-소계총계-group-by-rollup">21. 학과별·GPA 구간별 소계/총계 (GROUP BY ROLLUP)</h2>
<ul>
<li>GPA를 구간 3.0 미만 / 3.0~3.5 / 3.5 초과)으로 분류하는 파생 컬럼을 추가</li>
<li>GROUP BY ROLLUP(major, gpa_tier)로 학과별, 전체 소계를 동시에 조회</li>
<li>GROUPING(major) 함수로 소계 행에 '전체' 라벨을 붙일 것</li>
<li>결과를 major, gpa_tier 순으로 정렬하되소계 행은 하단에 표시</li>
</ul>
<pre><code class="language-sql">SELECT
    CASE WHEN GROUPING(major) = 1 THEN '전체' ELSE major END AS major,
    CASE WHEN GROUPING(gpa_tier) = 1 THEN '소계' ELSE gpa_tier END AS gpa_tier,
    COUNT(*) AS cnt,
    ROUND(AVG(gpa), 2) AS avg_gpa
FROM (
    SELECT *,
           CASE
               WHEN gpa &lt; 3.0 THEN '3.0 미만'
               WHEN gpa &lt;= 3.5 THEN '3.0~3.5'
               ELSE '3.5 초과'
           END AS gpa_tier
    FROM lab.student
) t
GROUP BY ROLLUP(major, gpa_tier)
ORDER BY GROUPING(major), major, GROUPING(gpa_tier), gpa_tier;</code></pre>
<h2 id="22-조직-계층-경로깊이-with-recursive--매니저별-직속-부하-수">22. 조직 계층 경로/깊이 (WITH RECURSIVE) + 매니저별 직속 부하 수</h2>
<p><strong>- WITH RECURSIVE</strong>로 <strong>CEO</strong>에서 시작하는 조직 트리를 탐색
<strong>-</strong> 각 행에 <strong>depth(0=CEO), path</strong> 컬럼을 포함
<em>예*</em>: 'CEO &gt; Mgr_2 &gt; Dev_15’)
<strong>-</strong> 매니저별 직속 부하 직원 수를 집계하는 쿼리 별도로 작성 <strong>(</strong>컬럼명 **: direct_reports)</p>
<pre><code class="language-sql">WITH RECURSIVE org AS (
    -- Anchor: CEO (manager_id가 NULL인 최상위)
    SELECT emp_id,
           name,
           manager_id,
           0 AS depth,
           name::TEXT AS path
    FROM lab.emp
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive: 부모(org)와 emp를 조인해서 한 단계씩 아래로 내려감
    SELECT e.emp_id,
           e.name,
           e.manager_id,
           o.depth + 1 AS depth,
           o.path || ' &gt; ' || e.name AS path
    FROM lab.emp e
    JOIN org o ON e.manager_id = o.emp_id
)
SELECT emp_id, name, depth, path
FROM org
ORDER BY path;</code></pre>
<h2 id="22-2--매니저별-직속-부하-직원-수">22-2  매니저별 직속 부하 직원 수</h2>
<pre><code class="language-sql">SELECT m.emp_id,
       m.name AS manager_name,
       COUNT(e.emp_id) AS direct_reports
FROM lab.emp m
JOIN lab.emp e ON e.manager_id = m.emp_id   
WHERE m.manager_id IS NOT NULL              
GROUP BY m.emp_id, m.name
ORDER BY m.emp_id;</code></pre>
<h2 id="23-학과별-gpa-상위-3명-window-function-cte서브쿼리-방식">23. 학과별 GPA 상위 3명 (Window Function, CTE/서브쿼리 방식)</h2>
<p>(서브쿼리 방식과 CTE 방식 모두 작성)</p>
<ul>
<li>학과 내 순위를 계산
ROW_NUMBER() OVER (PARTITION BY major ORDER BY gpa DESC)</li>
<li>GPA가 동일한 경우 student_id 오름차순을 2차 기준으로 사용</li>
<li>RANK()와 DENSE_RANK()를 함께 계산해 동점 처리 방식 차이를 비교</li>
<li>결과에 전체 학과별 학생 수(total_in_major)도 COUNT() OVER(PARTITION BY major)로 추가</li>
</ul>
<h2 id="cte방식">cte방식</h2>
<pre><code class="language-sql">WITH ranked AS (
    SELECT student_id,
           name,
           major,
           gpa,
           ROW_NUMBER() OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS rn,
           RANK()       OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS rnk,
           DENSE_RANK() OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS drk,
           COUNT(*)     OVER (PARTITION BY major) AS total_in_major
    FROM lab.student
)
SELECT student_id, name, major, gpa, rn, rnk, drk, total_in_major
FROM ranked
WHERE rn &lt;= 3
ORDER BY major, rn;</code></pre>
<h2 id="서브쿼리-방식">서브쿼리 방식</h2>
<pre><code class="language-sql">SELECT student_id, name, major, gpa, rn, rnk, drk, total_in_major
FROM (
    SELECT student_id,
           name,
           major,
           gpa,
           ROW_NUMBER() OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS rn,
           RANK()       OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS rnk,
           DENSE_RANK() OVER (PARTITION BY major ORDER BY gpa DESC, student_id ASC) AS drk,
           COUNT(*)     OVER (PARTITION BY major) AS total_in_major
    FROM lab.student
) t
WHERE rn &lt;= 3
ORDER BY major, rn;
</code></pre>
<h2 id="24-이전-수강-과목-대비-성적-변화-lag">24. 이전 수강 과목 대비 성적 변화 (LAG)</h2>
<p>(성적은 A=4, B=3, C=2, D=1 점수로 변환)</p>
<ul>
<li>grade를 숫자 점수(A=4, B=3, C=2, D=1)로 변환하는 CASE 식을 작성</li>
<li>LAG(score) OVER (PARTITION BY student_id ORDER BY course)로 이전 과목 점수를 가져올 것</li>
<li>현재 점수 - 이전 점수를 diff 컬럼으로 추가하고, 상승/유지/하락을 텍스트로 표시</li>
<li>학생별 최고점과 최저점의 차이(score_range)를 Window Function으로 계산<pre><code class="language-sql">WITH scored AS (
  SELECT student_id,
         course,
         grade,
         CASE grade
             WHEN 'A' THEN 4
             WHEN 'B' THEN 3
             WHEN 'C' THEN 2
             ELSE 1
         END AS score
  FROM lab.enroll
),
lagged AS (
  SELECT student_id,
         course,
         grade,
         score,
         LAG(score) OVER (PARTITION BY student_id ORDER BY course) AS prev_score,
         MAX(score) OVER (PARTITION BY student_id)
       - MIN(score) OVER (PARTITION BY student_id) AS score_range
  FROM scored
)
SELECT student_id,
     course,
     grade,
     score,
     prev_score,
     score - prev_score AS diff,
     CASE
         WHEN prev_score IS NULL THEN '첫 과목'
         WHEN score &gt; prev_score THEN '상승'
         WHEN score = prev_score THEN '유지'
         ELSE '하락'
     END AS trend,
     score_range
FROM lagged
ORDER BY student_id, course;</code></pre>
</li>
</ul>
<h2 id="25-누적-주문금액--이동평균--고객별-누적--50-초과-지점">25. 누적 주문금액 / 이동평균 / 고객별 누적 / 50% 초과 지점</h2>
<ul>
<li>SUM(amount) OVER (ORDER BY order_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)로 누적합 계산</li>
<li>AVG(amount) OVER (ORDER BY order_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)로 3개 이동평균 계산</li>
<li>customer_id별로 PARTITION을 나눠 고객별 누적 구매금액도 함께 계산</li>
<li>누적합이 전체 합의 50%를 초음하는 첫 번째 order_id를 찾는 쿼리를 작성<pre><code class="language-sql">WITH cum AS (
  SELECT order_id,
         customer_id,
         amount,
         SUM(amount) OVER (
             ORDER BY order_id
             ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
         ) AS cum_total,
         AVG(amount) OVER (
             ORDER BY order_id
             ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
         ) AS moving_avg_3,
         SUM(amount) OVER (
             PARTITION BY customer_id
             ORDER BY order_id
             ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
         ) AS cust_cum
  FROM lab.orders
)
SELECT order_id, customer_id, amount, cum_total, moving_avg_3, cust_cum
FROM cum
ORDER BY order_id;</code></pre>
</li>
</ul>
<h2 id="25-2--50-초과-지점의-첫-order_id-찾기">25-2  <strong>50% 초과 지점의 첫 order_id 찾기</strong></h2>
<pre><code class="language-sql">WITH cum AS (
    SELECT order_id,
           amount,
           SUM(amount) OVER (
               ORDER BY order_id
               ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
           ) AS cum_total
    FROM lab.orders
),
total AS (
    SELECT SUM(amount) AS total_sum
    FROM lab.orders
)
SELECT order_id, cum_total, total.total_sum
FROM cum, total
WHERE cum_total &gt;= total.total_sum * 0.5
ORDER BY order_id
LIMIT 1;</code></pre>