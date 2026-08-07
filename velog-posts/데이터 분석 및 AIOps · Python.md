<p>파이썬은 <strong>데이터를 변수에 담고 → 원하는 방식으로 처리한 뒤 → 결과를 화면·파일·웹 서비스에 전달하는</strong> 언어입니다. 문법을 외우기보다 아래 흐름을 반복해서 익히는 게 훨씬 남습니다.</p>
<pre><code class="language-text">값 만들기 → 자료구조에 담기 → 함수로 처리하기 → 파일/API와 연결하기 → 오류와 테스트로 확인하기</code></pre>
<h2 id="0-py-파일은-무엇인가요">0. <code>.py</code> 파일은 무엇인가요?</h2>
<p><code>.py</code>는 파이썬 코드를 적어 두는 <strong>텍스트 파일</strong>입니다. <code>hello.py</code>에 아래처럼 저장합니다.</p>
<pre><code class="language-python">name = &quot;민지&quot;
print(f&quot;안녕하세요, {name}님!&quot;)</code></pre>
<p>터미널에서 파일이 있는 폴더로 이동해 실행합니다.</p>
<pre><code class="language-bash">python hello.py</code></pre>
<p>파이썬은 파일을 위에서 아래로 읽고, <code>print()</code>를 만나면 화면에 결과를 출력합니다.</p>
<ul>
<li><strong>스크립트</strong>: <code>python hello.py</code>처럼 직접 실행하는 <code>.py</code> 파일</li>
<li><strong>모듈</strong>: 다른 <code>.py</code> 파일에서 <code>import</code>하여 가져다 쓰는 파일 — 같은 파일이 스크립트도, 모듈도 될 수 있습니다</li>
</ul>
<pre><code class="language-python"># calculator.py
def add(left: float, right: float) -&gt; float:
    return left + right

# main.py
from calculator import add
print(add(2, 3))</code></pre>
<p>파일 이름은 보통 소문자와 밑줄을 써서 <code>sales_report.py</code>처럼 짓습니다.</p>
<h3 id="if-__name__--__main__은-왜-쓰나요"><code>if __name__ == &quot;__main__&quot;:</code>은 왜 쓰나요?</h3>
<p>파일을 <strong>직접 실행했을 때만</strong> 실행할 코드를 구분하는 장치입니다. 다른 파일이 함수만 가져다 쓸 때 예제 코드까지 함께 실행되는 걸 막아줍니다.</p>
<pre><code class="language-python">def greet(name: str) -&gt; None:
    print(f&quot;안녕하세요, {name}님!&quot;)

if __name__ == &quot;__main__&quot;:
    greet(&quot;민지&quot;)  # 이 파일을 직접 실행할 때만 실행됨</code></pre>
<hr />
<h2 id="1-실행-구조--파이썬은-코드를-어떻게-실행하나요">1. 실행 구조 — 파이썬은 코드를 어떻게 실행하나요?</h2>
<p>처음에는 &quot;파이썬이 <code>.py</code> 파일을 읽어 실행한다&quot;고 이해해도 충분합니다. 조금 더 자세히 보면 다음 파이프라인을 거칩니다.</p>
<pre><code class="language-text">소스코드(.py) → 파서(Parser) → AST(추상 구문 트리) → 바이트코드(.pyc) → PVM(Python Virtual Machine) → 실행 결과</code></pre>
<ul>
<li><strong>소스 코드</strong>: 사람이 작성한 <code>.py</code> 파일</li>
<li><strong>AST(Abstract Syntax Tree)</strong>: 파이썬이 코드의 문장 구조를 트리로 이해한 내부 표현. 린터·포매터·트랜스파일러가 코드를 &quot;이해&quot;할 때도 바로 이 AST를 사용합니다</li>
<li><strong>바이트코드</strong>: 더 빠르게 실행하도록 바꾼 중간 표현. <code>__pycache__</code> 폴더의 <code>.pyc</code> 파일이 바로 이 캐시라서, 직접 수정하거나 Git에 올릴 필요 없이 <code>.gitignore</code>에 넣습니다</li>
<li><strong>PVM</strong>: 바이트코드를 하나씩 읽어 해석·실행하는 스택 기반 엔진</li>
</ul>
<p>실제로 눈으로 확인해볼 수 있습니다.</p>
<pre><code class="language-python">import ast, dis

# AST로 코드 구조 보기
tree = ast.parse('x = a + b')
print(ast.dump(tree, indent=2))

# 바이트코드로 함수 내부 보기
def add(x, y):
    return x + y

dis.dis(add)
# LOAD_FAST 0 (x)
# LOAD_FAST 1 (y)
# BINARY_OP 0 (+)
# RETURN_VALUE</code></pre>
<blockquote>
<p><strong>왜 이걸 알아야 하나?</strong>
컴프리헨션이 왜 for 루프보다 빠른지, Polars가 왜 Pandas보다 빠른지를 바이트코드 레벨에서 납득할 수 있게 됩니다. 또 <code>pip install</code> 실패나 <code>ModuleNotFoundError</code>도 결국 인터프리터 경로·가상환경 활성화 문제인 경우가 많아서, 이 구조를 알면 에러를 훨씬 빨리 읽을 수 있습니다.</p>
</blockquote>
<h3 id="오류-메시지traceback-읽는-법">오류 메시지(Traceback) 읽는 법</h3>
<p>오류가 나면 Traceback에서 <strong>내 파일 이름과 줄 번호</strong>를 먼저 찾으세요. 가장 아래쪽에 실제 오류 종류와 이유가 표시됩니다.</p>
<pre><code class="language-text">File &quot;hello.py&quot;, line 3, in &lt;module&gt;
ZeroDivisionError: division by zero</code></pre>
<p><code>hello.py</code> 3번째 줄에서 0으로 나누려 했다는 뜻입니다. Traceback은 사실 PVM이 코드를 실행하며 쌓은 호출 스택을 그대로 보여주는 것이라, 위에서 배운 실행 구조를 알고 있으면 오류를 파일·줄·표현식 단위로 훨씬 빠르게 찾을 수 있습니다.</p>
<h3 id="메모리-모델-참고">메모리 모델 (참고)</h3>
<ul>
<li><strong>Stack</strong>: 함수 호출, 지역 변수 저장</li>
<li><strong>Heap</strong>: 객체, 클래스 인스턴스, 리스트 등 실제 데이터가 저장되는 공간</li>
<li>모든 값은 <code>type</code>(자료형), <code>refcount</code>(참조 수), <code>value</code>를 가진 객체로 표현되며, Reference Counting + Garbage Collector가 메모리를 자동으로 회수합니다</li>
</ul>
<hr />
<h2 id="2-개발-환경-가상환경venv">2. 개발 환경: 가상환경(venv)</h2>
<p>프로젝트마다 필요한 라이브러리 버전이 다를 수 있습니다. 가상환경은 프로젝트별로 독립된 라이브러리 상자를 만드는 기능입니다.</p>
<pre><code class="language-bash">python -m venv .venv
source .venv/bin/activate      # macOS/Linux
# .venv\Scripts\activate       # Windows PowerShell
python -m pip install --upgrade pip</code></pre>
<p>활성화되면 터미널 앞에 보통 <code>(.venv)</code>가 표시됩니다. 작업을 마치면 <code>deactivate</code>로 나올 수 있습니다. <code>.venv/</code>는 <code>.gitignore</code>에 추가하고, 설치한 패키지 목록은 <code>requirements.txt</code>로 반드시 커밋해서 팀원과 동일한 환경을 재현할 수 있게 합니다.</p>
<hr />
<h2 id="3-여러-데이터를-담는-자료구조">3. 여러 데이터를 담는 자료구조</h2>
<table>
<thead>
<tr>
<th>자료구조</th>
<th>쉽게 말하면</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>list</code></td>
<td>순서가 있는 목록</td>
<td>중복 허용, 수정 가능 (동적 배열), 접근 O(1)</td>
</tr>
<tr>
<td><code>dict</code></td>
<td>이름표(키)가 붙은 정보 묶음</td>
<td>키로 값 찾기 O(1), 3.7+부터 삽입 순서 보장</td>
</tr>
<tr>
<td><code>set</code></td>
<td>중복 없는 값 모음</td>
<td>해시 기반, 합집합(<code>|</code>)·교집합(<code>&amp;</code>)·차집합(<code>-</code>)</td>
</tr>
<tr>
<td><code>tuple</code></td>
<td>바꾸지 않는 순서 목록</td>
<td>불변(immutable) → dict 키로도 사용 가능</td>
</tr>
</tbody></table>
<pre><code class="language-python">todos = [&quot;공부&quot;, &quot;운동&quot;]
student = {&quot;name&quot;: &quot;민지&quot;, &quot;age&quot;: 20}
visited_cities = {&quot;서울&quot;, &quot;부산&quot;, &quot;서울&quot;}  # 중복된 서울은 하나만 남음</code></pre>
<p>여러 값을 바꾸거나 순서대로 다룰 땐 <code>list</code>, &quot;이름으로 값 찾기&quot;가 필요할 땐 <code>dict</code>를 가장 자주 씁니다.</p>
<h3 id="조금-더-나아가면-collections-모듈">조금 더 나아가면: collections 모듈</h3>
<ul>
<li><strong><code>deque</code></strong>: 리스트는 맨 앞 삽입/삭제가 O(n)이지만, <code>deque</code>는 내부적으로 &quot;linked list of arrays&quot; 구조라서 <strong>양 끝 삽입/삭제가 모두 O(1)</strong>입니다. 대기열(Queue)이나 슬라이딩 윈도우 문제에 유용합니다.</li>
<li><strong><code>Counter</code></strong>: 값의 개수를 셀 때 사용</li>
<li><strong><code>defaultdict</code></strong>: 키가 없을 때 초기값을 자동으로 만들어주는 딕셔너리</li>
<li><strong><code>heapq</code> / <code>bisect</code></strong>: 우선순위 큐, 정렬된 리스트의 이진 탐색 관리</li>
</ul>
<pre><code class="language-python">from collections import deque
dq = deque([1, 2, 3])
dq.append(4)      # 오른쪽에 추가 O(1)
dq.appendleft(0)   # 왼쪽에 추가 O(1)</code></pre>
<h3 id="컴프리헨션과-제너레이터">컴프리헨션과 제너레이터</h3>
<p>컴프리헨션은 반복문으로 목록을 만드는 짧은 표현입니다.</p>
<pre><code class="language-python">prices = [500, 1200, 800, 1500]
high_prices = [price for price in prices if price &gt;= 1000]
# [1200, 1500]

# 딕셔너리 컴프리헨션
fruit_prices = {&quot;apple&quot;: 1000, &quot;banana&quot;: 500}
discounted = {name: price * 0.9 for name, price in fruit_prices.items()}</code></pre>
<p>한 줄이 길어지거나 조건이 복잡하면 읽기 쉬운 <code>for</code>문으로 풀어 쓰세요. 위 1번에서 본 것처럼 컴프리헨션은 내부적으로 최적화된 바이트코드로 반복되기 때문에 일반 <code>for</code> 루프보다 빠릅니다.</p>
<p>큰 데이터를 모두 메모리에 올리기 어렵다면 리스트 <code>[]</code> 대신 제너레이터 표현식 <code>()</code> 또는 <code>yield</code>로 한 항목씩 처리합니다.</p>
<pre><code class="language-python">def fib(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b</code></pre>
<pre><code class="language-python">import sys
lst = list(range(10000))
gen = (x for x in range(10000))
print(sys.getsizeof(lst))  # 약 87KB
print(sys.getsizeof(gen))  # 약 104B — 메모리 차이가 극명함</code></pre>
<hr />
<h2 id="4-함수-반복-작업에-이름-붙이기">4. 함수: 반복 작업에 이름 붙이기</h2>
<p>함수는 입력을 받아 처리하고 결과를 돌려주는 작은 작업 단위입니다. &quot;읽기, 정제, 저장&quot;처럼 역할을 나누면 코드가 읽기 쉽고 테스트도 쉬워집니다.</p>
<pre><code class="language-python">def calculate_average(total: float, count: int) -&gt; float:
    return total / count

average = calculate_average(100, 4)</code></pre>
<ul>
<li><code>total</code>, <code>count</code>: 함수에 넣는 <strong>입력값(매개변수)</strong></li>
<li><code>return</code>: 처리 결과를 돌려주는 키워드</li>
<li><code>-&gt; float</code>: 결과가 실수라는 <strong>타입 힌트</strong>. 실행을 바꾸지는 않지만 실수를 찾는 데 도움을 줍니다 (7번에서 더 다룹니다)</li>
</ul>
<h3 id="인자를-몇-개든-받고-싶다면-args-kwargs">인자를 몇 개든 받고 싶다면: <code>*args</code>, <code>**kwargs</code></h3>
<pre><code class="language-python">def greet(*args, **kwargs):
    print(args)    # 위치 인자 → 튜플
    print(kwargs)  # 키워드 인자 → 딕셔너리</code></pre>
<h3 id="클로저와-데코레이터--기존-함수를-건드리지-않고-기능-덧붙이기">클로저와 데코레이터 — 기존 함수를 건드리지 않고 기능 덧붙이기</h3>
<p>클로저는 내부 함수가 외부 함수의 지역 변수를 기억하고 계속 사용할 수 있는 특성입니다. 데코레이터는 이 클로저 개념을 활용해 함수 앞뒤로 로직을 감싸는 패턴입니다.</p>
<pre><code class="language-python">def timer(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f&quot;실행 시간: {time.time() - start:.4f}초&quot;)
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(2)
    print(&quot;작업 완료!&quot;)</code></pre>
<p>이 패턴이 왜 중요한지는 실전 프레임워크를 보면 체감됩니다. <strong>FastAPI의 라우팅 자체가 데코레이터 기반</strong>입니다.</p>
<pre><code class="language-python">from fastapi import FastAPI
app = FastAPI()

@app.get(&quot;/items/{item_id}&quot;)
def read_item(item_id: int):
    return {&quot;item_id&quot;: item_id}</code></pre>
<hr />
<h2 id="5-파일-읽기와-쓰기">5. 파일 읽기와 쓰기</h2>
<p>경로는 문자열을 직접 이어 붙이기보다 <code>pathlib.Path</code>로 다루고, 파일은 <code>with</code>로 엽니다. <code>with</code> 블록이 끝나면 파일이 자동으로 닫히므로 안전합니다.</p>
<pre><code class="language-python">from pathlib import Path

path = Path(&quot;memo.txt&quot;)
path.write_text(&quot;첫 번째 메모&quot;, encoding=&quot;utf-8&quot;)
text = path.read_text(encoding=&quot;utf-8&quot;)
print(text)</code></pre>
<ul>
<li><strong>CSV</strong>: 행과 열로 된 표 데이터에 적합 (<code>csv</code> 모듈)</li>
<li><strong>JSON</strong>: API 응답처럼 목록과 정보 묶음이 중첩된 데이터에 적합 (<code>json</code> 모듈)</li>
</ul>
<p>운영 환경에서는 <code>print()</code> 대신 <strong><code>logging</code></strong> 모듈을 씁니다. 로그 레벨(DEBUG/INFO/WARNING/ERROR)에 따라 콘솔·파일·에러 파일로 나눠 기록하면 문제 원인을 훨씬 빨리 추적할 수 있고, API 키 같은 민감 정보는 코드에 직접 쓰지 않고 <code>.env</code> 파일 + 환경변수로 분리해서 관리합니다.</p>
<hr />
<h2 id="6-예외-처리-문제가-생겨도-이유를-알기">6. 예외 처리: 문제가 생겨도 이유를 알기</h2>
<p><code>try</code>에는 실패할 수 있는 코드를, <code>except</code>에는 그 실패에 대한 대응을 둡니다. 모든 오류를 무시하면 원인을 찾기 어려우므로 예상 가능한 오류만 구체적으로 처리하세요.</p>
<pre><code class="language-python">from pathlib import Path

def load_text(path: Path) -&gt; str:
    try:
        return path.read_text(encoding=&quot;utf-8&quot;)
    except FileNotFoundError as error:
        raise FileNotFoundError(f&quot;파일을 찾을 수 없습니다: {path}&quot;) from error
    finally:
        print(&quot;로딩 종료&quot;)</code></pre>
<ul>
<li><code>try</code>: 시도할 코드</li>
<li><code>except</code>: 특정 오류가 났을 때의 처리</li>
<li><code>else</code>: 오류 없이 끝났을 때 실행할 코드</li>
<li><code>finally</code>: 성공·실패와 관계없이 마지막에 실행할 코드</li>
</ul>
<p>필요하면 예외를 직접 만들 수도 있습니다.</p>
<pre><code class="language-python">class DataValidationError(ValueError):
    def __init__(self, col, val):
        self.col, self.val = col, val
        super().__init__(f&quot;{col} 컬럼 값 오류: {val}&quot;)</code></pre>
<hr />
<h2 id="7-타입-힌트와-입력-데이터-검증">7. 타입 힌트와 입력 데이터 검증</h2>
<p>타입 힌트는 &quot;이 함수에는 어떤 값이 들어오고 무엇이 나오는가&quot;를 코드에 적는 메모이자 안전장치입니다. 실행에는 영향을 주지 않지만, <code>mypy</code> 같은 정적 검사 도구가 실행 전에 타입 오류를 미리 찾아줍니다.</p>
<pre><code class="language-python">from typing import Union, Optional, Literal, Any

def parse(value: Union[str, int]) -&gt; str:   # 여러 타입 중 하나
    return str(value)

def find_user(id: int) -&gt; Optional[str]:    # None이 허용될 수도 있음
    ...

def turn(direction: Literal[&quot;left&quot;, &quot;right&quot;]):  # 정해진 값만 허용
    print(f&quot;Turning {direction}&quot;)</code></pre>
<pre><code class="language-bash">$ mypy analysis.py
# analysis.py:4: error: Argument 1 has incompatible type 'list[str]'; expected 'list[float]'</code></pre>
<h3 id="pydantic--외부-데이터-검증의-표준">Pydantic — 외부 데이터 검증의 표준</h3>
<p>외부 CSV나 API 데이터는 형식이 틀릴 수 있으므로 Pydantic으로 검증하면 좋습니다.</p>
<pre><code class="language-python">from pydantic import BaseModel, Field

class SalesRecord(BaseModel):
    region: str
    amount: float = Field(gt=0)

record = SalesRecord.model_validate({&quot;region&quot;: &quot;서울&quot;, &quot;amount&quot;: 1500})</code></pre>
<p><code>amount</code>가 0 이하이거나 숫자가 아니면 검증 오류가 발생합니다. CSV 한 줄 한 줄을 이 모델로 검증하며 통과한 것과 실패한 것을 나눠 처리하는 패턴이 실전에서 자주 쓰입니다.</p>
<pre><code class="language-python">valid, errors = [], []
for i, row in enumerate(rows):
    try:
        valid.append(SalesRecord(**row))
    except Exception as e:
        errors.append({&quot;row&quot;: i, &quot;error&quot;: str(e)})</code></pre>
<blockquote>
<p>이 스키마 검증 구조는 그대로 FastAPI의 API 스키마와 연결되기 때문에, 지금 익혀두면 후속 과목에서 그대로 재활용할 수 있습니다.</p>
</blockquote>
<hr />
<h2 id="8-코드-품질과-테스트">8. 코드 품질과 테스트</h2>
<p>Ruff는 자주 생기는 코드 실수와 스타일 문제를 찾아주고(포매팅까지 통합), pytest는 &quot;이 함수가 기대한 대로 동작하는가&quot;를 자동으로 확인합니다.</p>
<pre><code class="language-bash">ruff check .
ruff format .
pytest -v</code></pre>
<p>pytest는 아래 규칙만 지키면 알아서 테스트를 찾아 실행해줍니다.</p>
<ul>
<li>파일 이름: <code>test_*.py</code> 또는 <code>*_test.py</code></li>
<li>함수 이름: <code>test_</code>로 시작</li>
</ul>
<pre><code class="language-python">def clean_amount(value: float | None) -&gt; float:
    return 0.0 if value is None else value

def test_clean_amount() -&gt; None:
    assert clean_amount(None) == 0.0
    assert clean_amount(12.5) == 12.5</code></pre>
<p>테스트는 정상 입력뿐 아니라 경계값과 실패 상황도 확인하는 게 좋습니다. <code>pytest-cov</code>로 커버리지(테스트가 코드의 몇 %를 실행해봤는지)를 측정할 수 있고, <code>pre-commit</code>으로 <code>git commit</code> 시점에 자동으로 포매팅·린팅 검사를 돌리거나 GitHub Actions CI에 연결해두면, &quot;코드 스타일은 사람이 아니라 도구가 통일한다&quot;는 원칙을 팀 전체에 적용할 수 있습니다.</p>
<hr />
<h2 id="9-비동기와-병렬-처리-기다리는-동안-다른-일-하기">9. 비동기와 병렬 처리: 기다리는 동안 다른 일 하기</h2>
<p>웹 API 요청처럼 결과를 기다리는 작업은 <code>asyncio</code>로 여러 개를 동시에 시작할 수 있습니다. 이는 여러 CPU로 계산하는 &quot;병렬 처리&quot;와는 다른 개념입니다.</p>
<pre><code class="language-python">import asyncio

async def say_hello() -&gt; None:
    await asyncio.sleep(1)
    print(&quot;1초 후 인사&quot;)

asyncio.run(say_hello())</code></pre>
<ul>
<li><code>async def</code>: 비동기 함수를 정의</li>
<li><code>await</code>: 작업이 끝날 때까지 기다리되, 그 사이 다른 비동기 작업이 진행될 수 있게 함</li>
<li><code>asyncio.run()</code>: 비동기 프로그램의 시작점</li>
</ul>
<h3 id="왜-대기-시간이-많은-작업에-asyncio를-쓰는지--gil-이야기">왜 &quot;대기 시간이 많은 작업&quot;에 asyncio를 쓰는지 — GIL 이야기</h3>
<p>CPython은 한 번에 <strong>하나의 스레드만</strong> 파이썬 코드를 실행할 수 있는 제약이 있습니다. 이걸 GIL(Global Interpreter Lock)이라고 합니다.</p>
<pre><code class="language-python"># CPU bound 작업 — threading을 써도 GIL 때문에 별 효과 없음
def cpu_task():
    return sum(range(10**7))

# I/O bound 작업 — 대기 중엔 GIL이 풀리기 때문에 효과적
import time
def io_task():
    time.sleep(1)  # 이 순간 GIL이 해제됨</code></pre>
<p>그래서 <strong>대기 시간이 많은 작업(API 호출, 파일 대기)에는 asyncio/threading</strong>, <strong>CPU 계산이 많은 작업(대량 전처리)에는 multiprocessing</strong>을 우선 검토합니다.</p>
<pre><code class="language-python">from concurrent.futures import ProcessPoolExecutor
import multiprocessing as mp

def process_chunk(chunk):
    return [transform(row) for row in chunk]

n_cores = mp.cpu_count()
chunks = split_chunks(all_data, n_cores)

with ProcessPoolExecutor(max_workers=n_cores) as exe:
    results = list(exe.map(process_chunk, chunks))
# 8코어 환경이면 이론상 8배 빨라짐</code></pre>
<p>여러 API를 순차로 호출하면 100개에 100초가 걸리지만, <code>asyncio</code>로 동시에 호출하면 약 2초면 끝납니다.</p>
<pre><code class="language-python">import asyncio, httpx

async def fetch(client, url):
    try:
        r = await client.get(url, timeout=10)
        return r.json()
    except Exception as e:
        return {&quot;error&quot;: str(e)}

async def fetch_all(urls):
    async with httpx.AsyncClient() as c:
        tasks = [fetch(c, u) for u in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)</code></pre>
<p>최적화는 &quot;느낌&quot;이 아니라 &quot;측정&quot;으로 합니다. <code>cProfile</code>로 어느 함수가 진짜 병목인지 먼저 확인하고 나서 최적화하세요.</p>
<pre><code class="language-python">import cProfile
cProfile.run('heavy_analysis(df)', sort='cumtime')</code></pre>
<blockquote>
<p>뒤에 나올 LangChain·RAG·AI Agent 과목이 전부 <code>async</code> 기반이기 때문에, 지금 이 개념 없이는 그 코드들을 읽을 수조차 없습니다.</p>
</blockquote>
<hr />
<h2 id="한눈에-보는-핵심-요약">한눈에 보는 핵심 요약</h2>
<ol>
<li><code>.py</code> 파일은 파이썬 코드 파일이며, <code>python 파일명.py</code>로 실행합니다. 실행 시 소스코드 → AST → 바이트코드 → PVM 순으로 처리됩니다.</li>
<li>여러 파일은 <code>import</code>로 연결하고, 직접 실행할 코드는 <code>if __name__ == &quot;__main__&quot;:</code> 아래에 둡니다.</li>
<li>목록은 <code>list</code>, 이름표가 있는 데이터는 <code>dict</code>, 중복 제거는 <code>set</code>을 우선 사용하고, 반복 작업은 컴프리헨션·제너레이터로 짧고 메모리 효율적으로 처리합니다.</li>
<li>반복 작업은 함수로 나누고, 데코레이터·클로저로 기능을 조립하며, 파일은 <code>Path</code>와 <code>with</code>로 안전하게 다룹니다.</li>
<li>오류는 Traceback의 파일·줄 번호부터 확인하고, 예상 가능한 오류는 <code>try-except-finally</code>로 처리합니다.</li>
<li>외부 입력은 타입 힌트와 Pydantic으로 검증하고, Ruff·pytest·pre-commit으로 품질을 확인합니다.</li>
<li>API처럼 기다림이 많은 작업은 <code>asyncio</code>로, CPU 연산이 많은 작업은 <code>multiprocessing</code>으로 처리합니다 (그 기준은 GIL의 한계에서 나옵니다).</li>
</ol>