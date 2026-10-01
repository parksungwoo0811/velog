<blockquote>
<p>목표는 이미 학습된 모델을 FastAPI API로 감싸고, 입력 검증·오류 처리·헬스체크·로깅·로딩 전략까지 갖춘 <strong>작동 가능한 추론 서버</strong>를 이해하는 것이다.</p>
</blockquote>
<hr />
<h2 id="0-오늘-무엇을-만드는가">0. 오늘 무엇을 만드는가?</h2>
<p>머신러닝 모델을 학습했다고 바로 서비스가 되는 것은 아니다. 학습이 끝난 모델은 보통 <code>.pkl</code>, <code>.pt</code>, <code>.keras</code> 같은 <strong>파일</strong>일 뿐이다. 웹·모바일 앱이나 다른 서버가 이 모델을 사용하려면, 정해진 형식으로 요청을 받고 예측 결과를 돌려주는 창구가 필요하다.</p>
<p>Day 1에서는 이 창구를 FastAPI로 만든다.</p>
<pre><code class="language-text">사용자 또는 다른 서비스
        ↓ JSON 요청
FastAPI 추론 서버
        ↓ 입력 검증
학습된 AI 모델 호출
        ↓ 예측 결과
FastAPI가 JSON 응답 반환</code></pre>
<p>오늘의 핵심 결과물은 다음 기능을 갖춘 추론 서버다.</p>
<ul>
<li>URL과 함수를 연결하는 라우팅</li>
<li>요청·응답 데이터의 형식을 정의하는 스키마</li>
<li>잘못된 입력을 막는 검증</li>
<li>정상과 오류를 구분하는 상태 코드 및 예외 처리</li>
<li>서버와 모델의 상태를 확인하는 헬스체크</li>
<li>요청과 결과를 추적하는 로깅</li>
<li>Swagger UI를 이용한 API 문서화와 테스트</li>
<li>Lazy Loading과 Eager Loading 비교</li>
</ul>
<p>전체 3일 과정에서의 위치는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>일차</th>
<th>핵심 질문</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>Day 1 - 모델 서빙</td>
<td>학습된 모델을 어떻게 다른 프로그램이 호출하게 할까?</td>
<td>FastAPI 추론 서버 구축</td>
</tr>
<tr>
<td>Day 2 - MLOps</td>
<td>모델을 어떻게 기록하고 검증하고 안전하게 배포할까?</td>
<td>버전 관리·배포·컨테이너화</td>
</tr>
<tr>
<td>Day 3 - AIOps</td>
<td>배포된 모델이 계속 잘 동작하는지 어떻게 감시하고 대응할까?</td>
<td>모니터링·드리프트 감지·자동 대응</td>
</tr>
</tbody></table>
<p>한 문장으로 줄이면 <strong>서빙은 모델을 쓸 수 있게 만들고, MLOps는 모델을 믿고 바꿀 수 있게 만들며, AIOps는 운영 중 나빠진 모델을 발견하고 회복하게 만든다.</strong></p>
<hr />
<h1 id="part-1-모델-서빙과-aiops-이해">Part 1. 모델 서빙과 AIOps 이해</h1>
<h2 id="11-모델-서빙이란">1.1 모델 서빙이란?</h2>
<p><strong>모델 서빙(Model Serving)</strong>은 학습이 끝난 모델을 실제 사용자의 요청에 응답할 수 있는 형태로 제공하는 단계다.</p>
<p>예를 들어 감성 분석 모델이 있다고 하자.</p>
<ul>
<li>학습 단계: 수많은 리뷰를 이용해 긍정·부정을 구분하도록 모델을 만든다.</li>
<li>서빙 단계: 사용자가 새로운 리뷰를 보내면 모델이 <code>positive</code>, <code>negative</code> 같은 결과를 즉시 돌려준다.</li>
</ul>
<pre><code class="language-text">데이터 수집·전처리
        ↓
모델 학습
        ↓
모델 아티팩트(학습 결과 파일)
        ↓
모델 서빙 ← Day 1
        ↓
모니터링·자동 대응 ← Day 3</code></pre>
<p>여기서 <strong>모델 아티팩트</strong>는 학습 결과를 저장한 파일과 관련 자원을 뜻한다. 모델 파일뿐 아니라 전처리에 사용한 스케일러, 토크나이저, 레이블 정보 등이 함께 필요할 수 있다.</p>
<h3 id="실생활-비유-요리법과-식당">실생활 비유: 요리법과 식당</h3>
<p>학습된 모델이 훌륭한 요리법이라면, 모델 서빙은 그 요리를 손님에게 실제로 제공하는 식당이다.</p>
<ul>
<li>요리법이 좋아도 주문을 받지 못하면 손님은 먹을 수 없다.</li>
<li>주문 형식이 제각각이면 주방이 혼란스러워진다.</li>
<li>손님이 몰려도 감당해야 한다.</li>
<li>재료가 없거나 주방이 고장 났으면 즉시 알아야 한다.</li>
</ul>
<p>모델 정확도만큼 API의 속도·안정성·가용성이 중요한 이유다.</p>
<h2 id="12-학습과-서빙은-무엇이-다른가">1.2 학습과 서빙은 무엇이 다른가?</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>모델 학습</th>
<th>모델 서빙</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>데이터에서 규칙을 학습</td>
<td>새 입력에 예측 결과 제공</td>
</tr>
<tr>
<td>실행 방식</td>
<td>수시간~수일의 배치 작업</td>
<td>보통 24시간 요청 대기</td>
</tr>
<tr>
<td>중요한 지표</td>
<td>정확도, Loss, RMSE 등</td>
<td>지연시간, 처리량, 에러율, 가용성</td>
</tr>
<tr>
<td>실패 영향</td>
<td>수정 후 다시 학습 가능</td>
<td>사용자 요청 실패, 서비스 장애로 이어짐</td>
</tr>
<tr>
<td>자원 패턴</td>
<td>비교적 계획 가능</td>
<td>트래픽에 따라 갑자기 변함</td>
</tr>
<tr>
<td>코드 성격</td>
<td>실험과 반복 변경이 잦음</td>
<td>안정성과 호환성이 중요</td>
</tr>
<tr>
<td>버전 관리</td>
<td>실험·하이퍼파라미터</td>
<td>배포 모델 버전·API 명세</td>
</tr>
<tr>
<td>장애 대응</td>
<td>재실행</td>
<td>즉시 롤백 또는 대체 경로 필요</td>
</tr>
</tbody></table>
<p>학습에서 정확도 99%인 모델도 응답에 30초가 걸리거나 하루에 여러 번 멈춘다면 좋은 서비스라고 하기 어렵다.</p>
<h2 id="13-서빙에서-보는-운영-지표">1.3 서빙에서 보는 운영 지표</h2>
<ul>
<li><strong>Latency(지연시간)</strong>: 요청 한 건이 응답을 받기까지 걸린 시간</li>
<li><strong>Throughput(처리량)</strong>: 초당 또는 분당 처리할 수 있는 요청 수</li>
<li><strong>Error Rate(에러율)</strong>: 전체 요청 중 실패한 요청의 비율</li>
<li><strong>Availability(가용성)</strong>: 사용자가 서비스를 정상적으로 이용할 수 있는 시간의 비율</li>
</ul>
<p>예를 들어 평균 응답시간만 보면 100ms로 좋아 보여도, 일부 사용자가 5초 이상 기다린다면 문제가 될 수 있다. 실무에서는 평균과 함께 p95, p99 같은 상위 지연시간도 살핀다.</p>
<h2 id="14-aiops가-제공하는-가치">1.4 AIOps가 제공하는 가치</h2>
<p>이 과정에서 AIOps는 AI 서비스를 안정적으로 운영하기 위한 감시와 대응 체계로 이해하면 된다.</p>
<ul>
<li><strong>자동 모니터링</strong>: 지연시간, 처리량, 에러율, 모델 상태를 지속적으로 수집</li>
<li><strong>이상 탐지</strong>: 평소보다 느려지거나 오류가 급증하는 현상 감지</li>
<li><strong>자동 대응</strong>: 알림, 재시작, 이전 버전 롤백, 재학습 등의 조치 수행</li>
</ul>
<p>Day 1에서 만드는 <code>/health</code>와 로그는 단순 부가기능이 아니다. Day 3에서 서버를 감시하고 문제의 원인을 찾는 기초 데이터가 된다.</p>
<hr />
<h1 id="part-2-모델-서빙-아키텍처">Part 2. 모델 서빙 아키텍처</h1>
<h2 id="21-배치-추론과-실시간-추론">2.1 배치 추론과 실시간 추론</h2>
<h3 id="배치-추론batch-inference">배치 추론(Batch Inference)</h3>
<p>데이터를 모아 두었다가 정해진 시간에 한꺼번에 예측한다.</p>
<ul>
<li>야간 매출 예측 보고서</li>
<li>전체 고객의 이탈 가능성 계산</li>
<li>하루 한 번 상품 추천 목록 갱신</li>
</ul>
<p>한 건의 즉시 응답보다 대량 처리 효율이 중요할 때 적합하다.</p>
<h3 id="실시간-추론real-time-inference">실시간 추론(Real-time Inference)</h3>
<p>요청이 들어온 순간 모델을 실행해 바로 응답한다.</p>
<ul>
<li>챗봇 답변</li>
<li>결제 직전 이상거래 탐지</li>
<li>사용자가 보는 화면의 실시간 추천</li>
</ul>
<p>사용자가 기다리므로 짧은 지연시간과 높은 가용성이 중요하다.</p>
<table>
<thead>
<tr>
<th>기준</th>
<th>배치 추론</th>
<th>실시간 추론</th>
</tr>
</thead>
<tbody><tr>
<td>처리 시점</td>
<td>예약된 시간 또는 데이터가 모였을 때</td>
<td>요청 직후</td>
</tr>
<tr>
<td>우선 가치</td>
<td>전체 처리량과 비용 효율</td>
<td>빠른 응답과 안정성</td>
</tr>
<tr>
<td>예시</td>
<td>야간 리포트</td>
<td>챗봇, 추천 API</td>
</tr>
</tbody></table>
<h2 id="22-통신-방식과-처리-방식">2.2 통신 방식과 처리 방식</h2>
<table>
<thead>
<tr>
<th>패턴</th>
<th>특징</th>
<th>적합한 상황</th>
</tr>
</thead>
<tbody><tr>
<td>REST + 동기</td>
<td>구현과 디버깅이 비교적 쉬움</td>
<td>작은 API, 빠른 프로토타입</td>
</tr>
<tr>
<td>REST + 비동기</td>
<td>I/O 대기 중 다른 요청 처리 가능</td>
<td>FastAPI 기반 일반 웹 API</td>
</tr>
<tr>
<td>gRPC</td>
<td>바이너리 통신, 강한 스키마, 낮은 오버헤드</td>
<td>내부 서비스 간 고성능 통신</td>
</tr>
<tr>
<td>WebSocket</td>
<td>연결을 유지하며 양방향 메시지 전송</td>
<td>채팅, 상태 갱신, 연결 상대 탐색</td>
</tr>
<tr>
<td>WebRTC</td>
<td>브라우저 간 초저지연 음성·영상·데이터 전송</td>
<td>화상회의, 실시간 미디어 AI</td>
</tr>
</tbody></table>
<p>Day 1의 기본 구조는 <strong>REST API + FastAPI</strong>다.</p>
<h3 id="rest와-http-메서드">REST와 HTTP 메서드</h3>
<ul>
<li><code>GET</code>: 상태나 데이터를 조회할 때 사용</li>
<li><code>POST</code>: 요청 본문을 보내 예측·생성·업로드 같은 작업을 실행할 때 사용</li>
</ul>
<p>예를 들면 <code>GET /health</code>는 상태 조회, <code>POST /predict</code>는 입력 데이터를 보내 예측 실행이라는 의미다.</p>
<h2 id="23-websocket과-webrtc의-관계">2.3 WebSocket과 WebRTC의 관계</h2>
<p>WebSocket과 WebRTC는 같은 것이 아니다.</p>
<ol>
<li>WebSocket으로 방에 들어오고 연결할 상대를 찾는다.</li>
<li>SDP·ICE 같은 연결 정보를 중계한다.</li>
<li>연결이 성립하면 실제 음성·영상은 WebRTC의 P2P 경로로 전달된다.</li>
</ol>
<pre><code class="language-text">클라이언트 A
   │  연결 정보 교환
   ▼
FastAPI WebSocket 시그널링 서버
   ▲
   │  연결 정보 교환
클라이언트 B

연결 후: 클라이언트 A ⇄ 클라이언트 B
         WebRTC 미디어 스트림</code></pre>
<p>따라서 FastAPI WebSocket 서버는 보통 <strong>통화 자체를 운반하는 서버</strong>가 아니라 <strong>누구와 어떻게 연결할지 중개하는 안내원</strong> 역할을 한다.</p>
<blockquote>
<p>실습 범위 구분: WebSocket·WebRTC는 메인 강의의 아키텍처 확장 개념이다. 제공된 Day 1 HAIC 스켈레톤의 필수 구현 범위는 REST 기반 <code>/predict</code>, <code>/health</code>, 데이터 업로드와 모델 로딩이다.</p>
</blockquote>
<hr />
<h1 id="part-3-fastapi-기초">Part 3. FastAPI 기초</h1>
<h2 id="31-fastapi-uvicorn-pydantic의-역할">3.1 FastAPI, Uvicorn, Pydantic의 역할</h2>
<p>세 도구를 한꺼번에 보면 헷갈리기 쉽다.</p>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>FastAPI</td>
<td>URL·요청·응답 규칙으로 API 애플리케이션 구성</td>
<td>식당 운영 규칙과 주문 창구</td>
</tr>
<tr>
<td>Uvicorn</td>
<td>FastAPI 애플리케이션을 실제로 실행하는 ASGI 서버</td>
<td>식당 문을 열고 주문을 받는 직원</td>
</tr>
<tr>
<td>Pydantic</td>
<td>들어오고 나가는 데이터의 타입과 조건 검증</td>
<td>주문서 형식 검사</td>
</tr>
</tbody></table>
<h2 id="32-최소-서버-만들기">3.2 최소 서버 만들기</h2>
<pre><code class="language-python">from fastapi import FastAPI

app = FastAPI(title=&quot;Model Serving API&quot;)


@app.get(&quot;/&quot;)
def root():
    return {&quot;status&quot;: &quot;ok&quot;}</code></pre>
<p>실행 명령은 다음 형태다.</p>
<pre><code class="language-bash">uvicorn main:app --host 0.0.0.0 --port 8000 --reload</code></pre>
<ul>
<li><code>main</code>: <code>main.py</code> 파일</li>
<li><code>app</code>: 파일 안의 <code>FastAPI()</code> 객체</li>
<li><code>--host 0.0.0.0</code>: 서버가 모든 네트워크 인터페이스에서 연결을 받도록 설정</li>
<li><code>--port 8000</code>: 8000번 포트 사용</li>
<li><code>--reload</code>: 개발 중 파일이 바뀌면 서버 자동 재시작</li>
</ul>
<p><code>0.0.0.0</code>은 브라우저에서 접속할 주소라기보다 <strong>서버가 어디에서 들어오는 연결을 받을지 정하는 바인딩 주소</strong>다. 로컬 브라우저에서는 보통 <code>http://localhost:8000</code>으로 접속한다.</p>
<blockquote>
<p><code>--reload</code>는 개발 편의 기능이다. 운영 환경에서는 일반적으로 사용하지 않는다.</p>
</blockquote>
<h2 id="33-라우팅-url을-함수와-연결하기">3.3 라우팅: URL을 함수와 연결하기</h2>
<pre><code class="language-python">@app.get(&quot;/health&quot;)
def health():
    return {&quot;status&quot;: &quot;ok&quot;}</code></pre>
<p><code>@app.get(&quot;/health&quot;)</code>는 <code>GET /health</code> 요청을 바로 아래 함수가 처리하도록 연결하는 데코레이터다. 이 연결을 <strong>라우팅</strong>이라고 한다.</p>
<p>프로젝트가 커지면 모든 경로를 <code>main.py</code>에 적지 않고 <code>APIRouter</code>로 기능별 분리한다.</p>
<pre><code class="language-python">from fastapi import APIRouter

router = APIRouter(prefix=&quot;/data&quot;)


@router.get(&quot;/status&quot;)
def status():
    return {&quot;uploaded&quot;: True}</code></pre>
<p>그리고 앱에 라우터를 등록한다.</p>
<pre><code class="language-python">app.include_router(router)</code></pre>
<p>이 경우 최종 URL은 <code>GET /data/status</code>다.</p>
<h2 id="34-요청-한-건의-처리-순서">3.4 요청 한 건의 처리 순서</h2>
<pre><code class="language-text">1. 요청 URL과 HTTP 메서드가 라우터에 매칭된다.
2. Pydantic이 요청 데이터를 파싱하고 검증한다.
3. 검증에 성공하면 핸들러 함수가 실행된다.
4. 모델이 추론한다.
5. 반환값을 응답 스키마에 맞춰 검증한다.
6. FastAPI가 JSON 응답을 만든다.</code></pre>
<h2 id="35-요청을-받는-세-가지-방법">3.5 요청을 받는 세 가지 방법</h2>
<h3 id="1-경로-파라미터">1) 경로 파라미터</h3>
<pre><code class="language-python">@router.get(&quot;/{filename}&quot;)
def read_log(filename: str):
    return {&quot;filename&quot;: filename}</code></pre>
<p><code>/logs/server.log</code>를 호출하면 <code>filename</code>에는 <code>server.log</code>가 들어간다. <code>{filename}</code>과 함수 인자 이름이 같아야 한다.</p>
<h3 id="2-json-요청-바디">2) JSON 요청 바디</h3>
<pre><code class="language-python">from pydantic import BaseModel


class PredictRequest(BaseModel):
    text: str


@app.post(&quot;/predict&quot;)
def predict(req: PredictRequest):
    return {&quot;received&quot;: req.text}</code></pre>
<p>클라이언트가 보내는 JSON은 다음과 같다.</p>
<pre><code class="language-json">{
  &quot;text&quot;: &quot;이 제품은 정말 좋아요&quot;
}</code></pre>
<h3 id="3-파일-업로드">3) 파일 업로드</h3>
<pre><code class="language-python">from fastapi import File, UploadFile


@router.post(&quot;/upload&quot;)
async def upload(file: UploadFile = File(...)):
    raw = await file.read()
    return {&quot;bytes&quot;: len(raw)}</code></pre>
<p>파일 읽기는 I/O 작업이므로 <code>async def</code>와 <code>await</code>를 사용한다.</p>
<h2 id="36-요청·응답-스키마">3.6 요청·응답 스키마</h2>
<pre><code class="language-python">from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class PredictRequest(BaseModel):
    text: str = Field(..., min_length=1, max_length=1000)
    top_k: int = Field(default=1, ge=1, le=10)


class PredictResponse(BaseModel):
    label: str
    score: float


@app.post(&quot;/predict&quot;, response_model=PredictResponse)
def predict(req: PredictRequest):
    return PredictResponse(label=&quot;positive&quot;, score=0.97)</code></pre>
<p><code>Field</code>의 주요 의미는 다음과 같다.</p>
<ul>
<li><code>...</code>: 필수 값</li>
<li><code>default=1</code>: 생략하면 기본값 1</li>
<li><code>min_length</code>, <code>max_length</code>: 문자열 또는 리스트 길이 제한</li>
<li><code>gt=0</code>: 0보다 커야 함</li>
<li><code>ge=0</code>: 0 이상이어야 함</li>
<li><code>le=10</code>: 10 이하여야 함</li>
</ul>
<p>입력이 조건을 어기면 핸들러가 실행되기 전에 FastAPI가 <strong>422 Unprocessable Entity</strong>를 반환한다. 이는 서버 고장이 아니라 스키마 검증이 의도대로 작동했다는 뜻이다. 응답의 <code>loc</code>, <code>msg</code>를 읽으면 어떤 필드가 왜 거절됐는지 알 수 있다.</p>
<h2 id="37-api-스키마와-db-모델은-다르다">3.7 API 스키마와 DB 모델은 다르다</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>Pydantic 모델</th>
<th>SQLAlchemy 모델</th>
</tr>
</thead>
<tbody><tr>
<td>목적</td>
<td>API 입출력 검증</td>
<td>데이터베이스 테이블 매핑</td>
</tr>
<tr>
<td>수명</td>
<td>요청 처리 중 잠시 존재</td>
<td>DB에 영구 저장</td>
</tr>
<tr>
<td>예시</td>
<td><code>PredictRequest</code>, <code>PredictResponse</code></td>
<td><code>PredictionLog</code></td>
</tr>
</tbody></table>
<p>같은 예측 결과라도 사용자에게 보여 줄 필드와 DB에 저장할 필드는 다를 수 있다. 두 계층을 분리하면 API나 테이블 한쪽의 변경이 다른 쪽에 미치는 영향을 줄일 수 있다.</p>
<h3 id="선택-확장-예측-로그를-db에-저장하기">선택 확장: 예측 로그를 DB에 저장하기</h3>
<p>메인 강의에서는 MariaDB 컨테이너와 SQLAlchemy ORM을 연결하는 예시도 다룬다. 개념적 흐름은 다음과 같다.</p>
<pre><code class="language-text">/predict 요청
   ↓
모델 예측
   ↓
PredictionLog 객체 생성
   ↓
DB 세션에 추가하고 commit
   ↓
예측 결과 응답</code></pre>
<p>연결 문자열의 계정·비밀번호는 코드에 직접 적지 말고 실제 프로젝트에서는 환경변수나 Secret 관리 도구로 분리해야 한다.</p>
<blockquote>
<p>실습 범위 구분: DB 연동은 이론 확장 예시다. Day 1 핵심 스켈레톤은 CSV 업로드와 로컬 모델·스케일러 생성에 집중한다.</p>
</blockquote>
<h2 id="38-swagger-ui와-openapi">3.8 Swagger UI와 OpenAPI</h2>
<p>서버 실행 후 <code>http://localhost:8000/docs</code>에 접속하면 Swagger UI가 자동 생성된다.</p>
<ul>
<li>등록된 엔드포인트 확인</li>
<li>요청·응답 필드와 타입 확인</li>
<li><code>Try it out</code>으로 실제 요청 실행</li>
<li>정상 응답과 422·400·500 응답 비교</li>
</ul>
<p>FastAPI는 타입 힌트와 Pydantic 모델을 바탕으로 OpenAPI 명세를 자동 생성한다. 즉 코드와 문서가 따로 놀 가능성을 줄여 준다.</p>
<h2 id="39-라우터와-정적-파일의-등록-순서">3.9 라우터와 정적 파일의 등록 순서</h2>
<p>실습 앱은 API뿐 아니라 대시보드 HTML도 같은 서버에서 제공한다.</p>
<pre><code class="language-python">app.include_router(predict_router)
app.include_router(health_router)
app.include_router(data_router)

# API 라우터를 모두 등록한 뒤 마지막에 정적 파일을 연결한다.
app.mount(&quot;/&quot;, StaticFiles(directory=&quot;static&quot;, html=True), name=&quot;static&quot;)</code></pre>
<p>정적 파일을 <code>/</code>에 먼저 마운트하면 <code>/predict</code> 같은 경로까지 정적 파일 쪽에서 먼저 잡을 수 있다. 따라서 <strong>구체적인 API 라우터를 먼저</strong>, 포괄적인 <code>/</code> 정적 파일 마운트를 마지막에 둔다.</p>
<h2 id="310-day-1에서-다루는-것과-다루지-않는-것">3.10 Day 1에서 다루는 것과 다루지 않는 것</h2>
<h3 id="핵심-범위">핵심 범위</h3>
<ul>
<li>GET·POST 라우팅</li>
<li>Pydantic 요청·응답 모델</li>
<li>입력 검증과 HTTP 예외</li>
<li>Swagger UI</li>
<li>모델 로딩과 추론</li>
<li>헬스체크와 로깅</li>
<li>파일 업로드</li>
<li>Lazy/Eager 로딩</li>
</ul>
<h3 id="이번-실습의-중심이-아닌-것">이번 실습의 중심이 아닌 것</h3>
<ul>
<li>OAuth2·JWT 인증</li>
<li>백그라운드 작업 큐</li>
<li>고급 미들웨어</li>
<li>Jinja2 템플릿</li>
<li>Kubernetes 배포</li>
<li>FastAPI·ASGI 내부 구현</li>
</ul>
<p>목표는 FastAPI의 모든 기능을 외우는 것이 아니라 <strong>모델 서빙 MVP가 안정적으로 동작하는 구조</strong>를 이해하는 것이다.</p>
<hr />
<h1 id="part-4-비동기-처리와-배치-처리">Part 4. 비동기 처리와 배치 처리</h1>
<h2 id="41-동기와-비동기의-차이">4.1 동기와 비동기의 차이</h2>
<p><strong>동기(Sync)</strong> 방식은 한 작업이 끝날 때까지 다음 흐름이 기다린다. <strong>비동기(Async)</strong> 방식은 네트워크·파일처럼 기다리는 시간이 생겼을 때 그동안 다른 요청을 처리할 수 있다.</p>
<p>카페에 비유해 보자.</p>
<ul>
<li>동기: 직원 한 명이 커피가 완성될 때까지 기계 앞에서 기다린 뒤 다음 주문을 받는다.</li>
<li>비동기: 커피 머신이 동작하는 동안 직원이 다음 주문을 받는다.</li>
</ul>
<p>중요한 점은 <strong>비동기가 커피 추출 자체를 빠르게 만드는 것은 아니라는 것</strong>이다. 대기 시간을 활용해 동시에 더 많은 요청을 처리하게 한다.</p>
<h2 id="42-async와-await">4.2 <code>async</code>와 <code>await</code></h2>
<pre><code class="language-python">@app.post(&quot;/upload&quot;)
async def upload(file: UploadFile):
    raw = await file.read()
    return {&quot;size&quot;: len(raw)}</code></pre>
<ul>
<li><code>async def</code>: 비동기 함수 선언</li>
<li><code>await</code>: 기다리는 동안 이벤트 루프가 다른 작업을 처리할 수 있도록 실행권을 돌려줌</li>
</ul>
<h2 id="43-무거운-모델-추론에-주의하기">4.3 무거운 모델 추론에 주의하기</h2>
<p>CPU·GPU를 오래 점유하는 동기 추론 함수를 <code>async def</code> 안에서 그냥 호출하면 이벤트 루프가 막힐 수 있다. 필요하면 스레드풀에 넘긴다.</p>
<pre><code class="language-python">from fastapi.concurrency import run_in_threadpool


@app.post(&quot;/predict-async&quot;)
async def predict_async(req: PredictRequest):
    result = await run_in_threadpool(heavy_model_inference, req.text)
    return result</code></pre>
<p>하지만 스레드풀도 모델 계산을 마법처럼 빠르게 만들지는 않는다. 모델의 CPU/GPU 특성, 라이브러리의 스레드 안전성, 메모리 사용량, 워커 수를 함께 고려해야 한다.</p>
<table>
<thead>
<tr>
<th>기준</th>
<th>동기</th>
<th>비동기</th>
</tr>
</thead>
<tbody><tr>
<td>이해·구현</td>
<td>직관적</td>
<td><code>async</code>·<code>await</code> 이해 필요</td>
</tr>
<tr>
<td>I/O 동시 처리</td>
<td>제한적</td>
<td>효율적</td>
</tr>
<tr>
<td>디버깅</td>
<td>상대적으로 쉬움</td>
<td>흐름 추적이 어려울 수 있음</td>
</tr>
<tr>
<td>적합 작업</td>
<td>단순 로직, 계산 중심 작업</td>
<td>파일·DB·외부 API 등 I/O 대기</td>
</tr>
</tbody></table>
<h2 id="44-배칭으로-처리량-높이기">4.4 배칭으로 처리량 높이기</h2>
<p>배칭은 짧은 시간 동안 요청을 모아 모델에 한꺼번에 넣는 방법이다.</p>
<pre><code class="language-text">개별 처리: 요청 1 → GPU 실행, 요청 2 → GPU 실행, 요청 3 → GPU 실행
배치 처리: 요청 1·2·3을 모음 → GPU 한 번 실행 → 결과를 각 요청자에게 반환</code></pre>
<pre><code class="language-python">BATCH_SIZE = 8
BATCH_TIMEOUT = 0.05  # 최대 50ms 대기


async def batch_worker(queue):
    batch = await collect_until(queue, BATCH_SIZE, BATCH_TIMEOUT)
    results = model.predict_batch([item.text for item in batch])

    for item, result in zip(batch, results):
        item.future.set_result(result)</code></pre>
<ul>
<li>장점: GPU를 효율적으로 사용해 처리량 증가</li>
<li>단점: 요청을 모으는 시간만큼 개별 지연시간이 늘어날 수 있음</li>
</ul>
<p>따라서 <code>BATCH_SIZE</code>와 <code>BATCH_TIMEOUT</code>은 처리량과 지연시간 사이의 균형점이다.</p>
<hr />
<h1 id="part-5-서빙-안정성">Part 5. 서빙 안정성</h1>
<h2 id="51-입력-검증과-에러-처리">5.1 입력 검증과 에러 처리</h2>
<p>Pydantic은 형식과 범위를 검증하고, 비즈니스 규칙은 핸들러에서 추가로 검사할 수 있다.</p>
<pre><code class="language-python">from fastapi import HTTPException


@app.post(&quot;/predict&quot;)
def predict(req: PredictRequest):
    if not req.text.strip():
        raise HTTPException(status_code=400, detail=&quot;text는 비어 있을 수 없습니다&quot;)

    try:
        return model.predict(req.text)
    except Exception:
        # 내부 예외의 상세 내용이나 민감정보를 사용자에게 그대로 노출하지 않는다.
        raise HTTPException(status_code=500, detail=&quot;추론 처리에 실패했습니다&quot;)</code></pre>
<p>주요 상태 코드는 다음처럼 이해할 수 있다.</p>
<table>
<thead>
<tr>
<th>상태 코드</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>200</td>
<td>정상 처리</td>
<td>예측 성공</td>
</tr>
<tr>
<td>400</td>
<td>요청 내용이 업무 규칙에 맞지 않음</td>
<td>공백 문자열</td>
</tr>
<tr>
<td>404</td>
<td>자원을 찾지 못함</td>
<td>존재하지 않는 로그 파일</td>
</tr>
<tr>
<td>422</td>
<td>요청 형식·타입·범위 검증 실패</td>
<td>시퀀스 길이 오류, 음수 값</td>
</tr>
<tr>
<td>500</td>
<td>서버 내부 처리 실패</td>
<td>모델 파일 손상, 예상하지 못한 추론 오류</td>
</tr>
</tbody></table>
<p>서버 로그에는 원인 분석을 위한 상세 예외를 남기되, 클라이언트 응답에는 파일 경로·스택 트레이스·비밀값 같은 내부 정보를 노출하지 않는 것이 좋다.</p>
<h2 id="52-헬스체크">5.2 헬스체크</h2>
<pre><code class="language-python">@app.get(&quot;/health&quot;)
def health_check():
    loaded = model is not None
    return {
        &quot;status&quot;: &quot;ok&quot; if loaded else &quot;degraded&quot;,
        &quot;model_loaded&quot;: loaded,
        &quot;loading_mode&quot;: &quot;eager&quot;,
    }</code></pre>
<p>헬스체크는 외부 시스템이 서버 상태를 자동 확인할 수 있게 한다.</p>
<ul>
<li>서버 프로세스가 살아 있는가?</li>
<li>요청을 받을 준비가 되었는가?</li>
<li>모델이 정상적으로 로드되었는가?</li>
</ul>
<p>로드밸런서, 컨테이너 오케스트레이터, 모니터링 시스템이 이 엔드포인트를 주기적으로 호출할 수 있다.</p>
<h2 id="53-lazy-loading과-eager-loading">5.3 Lazy Loading과 Eager Loading</h2>
<h3 id="eager-loading">Eager Loading</h3>
<p>서버 시작 시 모델을 미리 로드한다.</p>
<pre><code class="language-python">model = None


@app.on_event(&quot;startup&quot;)
def load_model_eager():
    global model
    model = load_model(&quot;model.keras&quot;)</code></pre>
<ul>
<li>서버 시작은 느리다.</li>
<li>첫 요청부터 빠르게 응답할 수 있다.</li>
<li>모델 파일 오류를 시작 단계에서 발견한다.</li>
</ul>
<h3 id="lazy-loading">Lazy Loading</h3>
<p>첫 예측 요청이 들어올 때 모델을 로드하고 캐시에 보관한다.</p>
<pre><code class="language-python">model = None


def get_model():
    global model
    if model is None:
        model = load_model(&quot;model.keras&quot;)
    return model</code></pre>
<ul>
<li>서버 시작은 빠르다.</li>
<li>첫 요청에는 모델 로딩 지연이 포함된다.</li>
<li>모델 파일 문제가 실제 첫 요청 때 뒤늦게 드러날 수 있다.</li>
</ul>
<table>
<thead>
<tr>
<th>측정 항목</th>
<th>Lazy</th>
<th>Eager</th>
</tr>
</thead>
<tbody><tr>
<td>서버 시작 시간</td>
<td>짧음</td>
<td>모델 로딩만큼 길어짐</td>
</tr>
<tr>
<td>첫 요청 시간</td>
<td>길 수 있음</td>
<td>빠름</td>
</tr>
<tr>
<td>두 번째 이후 요청</td>
<td>캐시 후 빠름</td>
<td>계속 빠름</td>
</tr>
<tr>
<td>오류 발견 시점</td>
<td>첫 요청 시</td>
<td>서버 시작 시</td>
</tr>
<tr>
<td>적합한 상황</td>
<td>빠른 기동, 사용 빈도가 낮은 모델</td>
<td>첫 사용자부터 일정한 응답, 배포 전 조기 검증</td>
</tr>
</tbody></table>
<p>어느 하나가 항상 정답은 아니다. 실제 측정 결과와 서비스 요구사항으로 선택해야 한다.</p>
<h2 id="54-로깅">5.4 로깅</h2>
<pre><code class="language-python">import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(&quot;model-serving&quot;)


@app.post(&quot;/predict&quot;)
def predict(req: PredictRequest):
    logger.info(&quot;prediction request received&quot;, extra={&quot;text_len&quot;: len(req.text)})
    result = model.predict(req.text)
    logger.info(&quot;prediction response sent&quot;, extra={&quot;label&quot;: result.label})
    return result</code></pre>
<p>좋은 로그는 다음 질문에 답할 수 있어야 한다.</p>
<ul>
<li>언제 요청이 왔는가?</li>
<li>어느 엔드포인트에서 처리했는가?</li>
<li>얼마나 걸렸는가?</li>
<li>성공했는가, 실패했는가?</li>
<li>어떤 모델 버전이 응답했는가?</li>
</ul>
<p>단, 원문 입력 전체, 개인정보, 인증 토큰, DB 비밀번호를 그대로 기록하면 안 된다. 필요한 경우 길이·해시·마스킹된 식별자처럼 최소한의 정보만 남긴다.</p>
<hr />
<h1 id="part-6-fastapi-추론-엔드포인트-실습">Part 6. FastAPI 추론 엔드포인트 실습</h1>
<p>메인 강의의 간단한 감성 분류 예시와 실제 HAIC 스켈레톤은 입력 모양이 다르지만, 흐름은 같다.</p>
<pre><code class="language-text">요청 → 스키마 검증 → 전처리 → 모델 추론 → 후처리 → 응답</code></pre>
<h2 id="61-간단한-감성-분류-예시">6.1 간단한 감성 분류 예시</h2>
<pre><code class="language-python">from fastapi import FastAPI
from pydantic import BaseModel, Field
import joblib

app = FastAPI()
model = joblib.load(&quot;sentiment_model.pkl&quot;)


class PredictRequest(BaseModel):
    text: str = Field(..., min_length=1, max_length=1000)


class PredictResponse(BaseModel):
    label: str
    score: float


@app.post(&quot;/predict&quot;, response_model=PredictResponse)
def predict(req: PredictRequest):
    label, score = model.predict([req.text])[0]
    return PredictResponse(label=label, score=float(score))</code></pre>
<p>요청 예시:</p>
<pre><code class="language-bash">curl -X POST &quot;http://localhost:8000/predict&quot; \
  -H &quot;Content-Type: application/json&quot; \
  -d '{&quot;text&quot;: &quot;이 제품은 정말 좋아요&quot;}'</code></pre>
<p>응답 예시:</p>
<pre><code class="language-json">{
  &quot;label&quot;: &quot;positive&quot;,
  &quot;score&quot;: 0.95
}</code></pre>
<h2 id="62-실제-day-1-haic-실습의-모델">6.2 실제 Day 1 HAIC 실습의 모델</h2>
<p>실습에서는 가상 종목의 최근 시세를 이용해 다음 날 종가를 예측하는 LSTM 모델을 사용한다.</p>
<ul>
<li>입력: 최근 20거래일의 <code>(close, volume)</code> 시퀀스</li>
<li>출력: 다음 날의 예상 종가 1개</li>
<li>Day 1 로컬 모델: baseline 모델 파일</li>
<li>함께 필요한 자원: 학습 때 만든 <code>scaler.pkl</code></li>
</ul>
<p>LSTM의 내부 수식이 이번 수업의 중심은 아니다. 초보자는 우선 다음 함수처럼 생각해도 충분하다.</p>
<pre><code class="language-text">최근 20일의 [종가, 거래량] → 모델 → 다음 날 예상 종가</code></pre>
<h3 id="왜-입력이-정확히-20개여야-할까">왜 입력이 정확히 20개여야 할까?</h3>
<p>모델을 학습할 때 과거 20일을 한 묶음으로 넣도록 입력 모양을 정했기 때문이다. 서빙 때 19개나 21개를 넣으면 학습 때와 모양이 달라져 모델이 처리할 수 없다.</p>
<p>개념적인 요청은 다음과 같다.</p>
<pre><code class="language-json">{
  &quot;sequence&quot;: [
    {&quot;close&quot;: 160.0, &quot;volume&quot;: 1200000},
    {&quot;close&quot;: 161.2, &quot;volume&quot;: 1180000}
  ]
}</code></pre>
<p>실제 요청에서는 같은 형식의 항목이 정확히 20개 필요하다.</p>
<p>응답의 핵심 필드는 다음과 같다.</p>
<pre><code class="language-json">{
  &quot;predicted_close&quot;: 161.23,
  &quot;model_version&quot;: &quot;v1-local&quot;
}</code></pre>
<p><code>model_version</code>을 함께 보내면 현재 어느 모델이 응답했는지 운영자가 확인하기 쉬워진다.</p>
<h3 id="왜-스케일러도-함께-보관할까">왜 스케일러도 함께 보관할까?</h3>
<p>학습 때 가격과 거래량을 특정 범위로 변환했다면, 서빙 때도 <strong>완전히 같은 변환 기준</strong>을 사용해야 한다. 다른 스케일러를 새로 학습하면 같은 가격이 다른 숫자로 변환되어 모델이 배운 기준과 어긋난다.</p>
<hr />
<h1 id="part-7-추론-서버-빌드업-통합-실습">Part 7. 추론 서버 빌드업 통합 실습</h1>
<h2 id="71-프로젝트-구조-읽기">7.1 프로젝트 구조 읽기</h2>
<p>간단한 개념 구조는 다음과 같다.</p>
<pre><code class="language-text">serving_app/
├── main.py              # FastAPI 앱 생성, 라우터 조립, 생명주기
├── schemas.py           # Pydantic 요청·응답 스키마
├── model_loader.py      # Lazy/Eager 모델 로딩과 캐시
├── routers/
│   ├── predict.py       # /predict
│   ├── health.py        # /health
│   ├── data.py          # /data/upload, /data/status
│   └── logs.py          # 로그 조회용 라우터
├── models/              # 모델과 스케일러
└── static/              # 실습 대시보드
scripts/
└── train_baseline_v1.py # Day 1 baseline 학습
data/
└── sample_haic_prices.csv</code></pre>
<p>하나의 <code>FastAPI()</code> 객체에 기능별 라우터를 연결하는 구조다.</p>
<h2 id="72-day-1-핵심-엔드포인트">7.2 Day 1 핵심 엔드포인트</h2>
<table>
<thead>
<tr>
<th>Method</th>
<th>URL</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>GET</td>
<td><code>/</code></td>
<td>실습용 대시보드</td>
</tr>
<tr>
<td>POST</td>
<td><code>/predict</code></td>
<td>20일 시퀀스로 다음 날 종가 예측</td>
</tr>
<tr>
<td>GET</td>
<td><code>/health</code></td>
<td>서버·모델 로딩 상태 확인</td>
</tr>
<tr>
<td>POST</td>
<td><code>/data/upload</code></td>
<td>학습용 CSV 업로드</td>
</tr>
<tr>
<td>GET</td>
<td><code>/data/status</code></td>
<td>최신 업로드 데이터 상태 확인</td>
</tr>
</tbody></table>
<p><code>/predict/batch-test</code>와 <code>/logs</code>는 뒤의 AIOps 흐름에서 본격 사용한다. Day 1에서는 전체 구조 속 위치만 알아 두면 된다.</p>
<h2 id="73-실습-준비">7.3 실습 준비</h2>
<p>모든 명령은 <code>project/</code> 같은 <strong>프로젝트 루트</strong>에서 실행한다. <code>serving_app/</code> 안으로 들어가 실행하면 Python import 경로가 달라질 수 있다.</p>
<pre><code class="language-bash">python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt</code></pre>
<p>TensorFlow가 포함되어 설치 시간이 길 수 있다. Python 버전과 패키지 호환성도 먼저 확인한다.</p>
<h2 id="74-실습-순서">7.4 실습 순서</h2>
<h3 id="step-1-lazy-모드로-서버-기동">Step 1. Lazy 모드로 서버 기동</h3>
<pre><code class="language-bash">uvicorn serving_app.main:app --host 0.0.0.0 --port 8000</code></pre>
<ul>
<li>대시보드: <code>http://localhost:8000/</code></li>
<li>API 문서: <code>http://localhost:8000/docs</code></li>
</ul>
<p>Lazy 모드에서는 모델 파일이 없어도 서버 자체는 시작할 수 있다. 하지만 첫 <code>/predict</code> 호출 시 모델 파일이 없어 실패한다. 이것은 Lazy 방식에서 오류가 늦게 드러나는 사례다.</p>
<h3 id="step-2-데이터-업로드">Step 2. 데이터 업로드</h3>
<p>대시보드에서 예시 CSV를 업로드하거나 다음과 같이 요청한다.</p>
<pre><code class="language-bash">curl -F &quot;file=@data/sample_haic_prices.csv&quot; \
  http://localhost:8000/data/upload</code></pre>
<p>그다음 <code>/data/status</code>에서 행 수와 데이터 상태를 확인한다. 실습 CSV에는 최소한 날짜·종가·거래량에 해당하는 필수 열과 충분한 행이 있어야 한다.</p>
<h3 id="step-3-baseline-모델과-스케일러-생성">Step 3. baseline 모델과 스케일러 생성</h3>
<p>별도 터미널에서 실행한다.</p>
<pre><code class="language-bash">python scripts/train_baseline_v1.py</code></pre>
<p>생성되는 핵심 산출물은 로컬 baseline 모델과 <code>scaler.pkl</code>이다. 스케일러는 이후 단계에서도 학습 시점과 동일한 전처리를 보장하기 위해 재사용한다.</p>
<h3 id="step-4-swagger에서-predict-테스트">Step 4. Swagger에서 <code>/predict</code> 테스트</h3>
<p>정상 입력과 실패 입력을 모두 시험한다.</p>
<table>
<thead>
<tr>
<th>테스트</th>
<th>기대 결과</th>
<th>확인할 개념</th>
</tr>
</thead>
<tbody><tr>
<td>시퀀스 20개, 모든 값 정상</td>
<td>200</td>
<td>정상 추론</td>
</tr>
<tr>
<td>시퀀스 19개</td>
<td>422</td>
<td>길이 검증</td>
</tr>
<tr>
<td><code>close = 0</code></td>
<td>422</td>
<td>값 범위 검증</td>
</tr>
<tr>
<td><code>volume &lt; 0</code></td>
<td>422</td>
<td>값 범위 검증</td>
</tr>
</tbody></table>
<p>오류가 났다는 사실만 보지 말고 422 응답의 <code>loc</code>와 <code>msg</code>를 읽어 어느 필드가 왜 거절됐는지 확인한다.</p>
<h3 id="step-5-health-확인">Step 5. <code>/health</code> 확인</h3>
<p>Lazy 모드에서는 첫 추론 전후로 <code>model_loaded</code>가 어떻게 바뀌는지 본다.</p>
<pre><code class="language-json">{
  &quot;status&quot;: &quot;ok&quot;,
  &quot;model_loaded&quot;: true,
  &quot;loading_mode&quot;: &quot;lazy&quot;
}</code></pre>
<h3 id="step-6-eager-모드와-비교">Step 6. Eager 모드와 비교</h3>
<p>서버를 종료한 뒤 Eager 모드로 다시 시작한다.</p>
<pre><code class="language-bash">LOADING_MODE=eager uvicorn serving_app.main:app \
  --host 0.0.0.0 --port 8000</code></pre>
<p>운영체제에 따라 환경변수 설정 문법은 다를 수 있다. 측정해야 할 값은 다음과 같다.</p>
<table>
<thead>
<tr>
<th>측정 항목</th>
<th align="right">Lazy</th>
<th align="right">Eager</th>
</tr>
</thead>
<tbody><tr>
<td>서버 시작 시간</td>
<td align="right">직접 측정</td>
<td align="right">직접 측정</td>
</tr>
<tr>
<td>첫 <code>/predict</code> 응답 시간</td>
<td align="right">직접 측정</td>
<td align="right">직접 측정</td>
</tr>
<tr>
<td>두 번째 <code>/predict</code> 응답 시간</td>
<td align="right">직접 측정</td>
<td align="right">직접 측정</td>
</tr>
</tbody></table>
<p>측정값 뒤에 결론을 한 줄로 적는다.</p>
<blockquote>
<p>예: 첫 사용자의 응답시간이 중요한 상시 서비스이므로 시작은 다소 느려도 Eager가 적합하다. 반대로 자주 사용하지 않는 여러 모델을 한 서버에 올린다면 Lazy로 메모리를 아끼는 방안을 검토할 수 있다.</p>
</blockquote>
<h2 id="75-swagger-통합-점검-순서">7.5 Swagger 통합 점검 순서</h2>
<ol>
<li>서버가 정상 기동했는지 확인한다.</li>
<li><code>/docs</code>에 접속한다.</li>
<li><code>/health</code>로 로딩 상태를 확인한다.</li>
<li><code>/predict</code>에 정상 20개 시퀀스를 보내 200 응답을 확인한다.</li>
<li>시퀀스 길이와 값 범위를 일부러 틀려 422 응답을 확인한다.</li>
<li>콘솔 로그에서 모델 로딩 시점과 요청 흐름을 확인한다.</li>
<li>Lazy/Eager의 시작·첫 요청·두 번째 요청 시간을 기록한다.</li>
</ol>
<hr />
<h1 id="실습에서-자주-만나는-문제">실습에서 자주 만나는 문제</h1>
<h2 id="1-서버는-켜졌는데-predict가-500이다">1. 서버는 켜졌는데 <code>/predict</code>가 500이다</h2>
<p>Lazy 모드는 모델 없이도 서버가 시작될 수 있다. baseline 학습을 실행해 모델과 스케일러가 실제 예상 경로에 생성됐는지 확인한다.</p>
<pre><code class="language-bash">python scripts/train_baseline_v1.py</code></pre>
<h2 id="2-업로드-데이터가-없다는-오류가-난다">2. 업로드 데이터가 없다는 오류가 난다</h2>
<p>먼저 CSV를 업로드하고 <code>/data/status</code>로 확인한다. 명령을 프로젝트 루트에서 실행했는지도 점검한다. 상대 경로는 현재 작업 폴더에 영향을 받는다.</p>
<h2 id="3-predict가-422를-반환한다">3. <code>/predict</code>가 422를 반환한다</h2>
<p>서버 장애가 아니라 입력 스키마가 요청을 막은 것이다.</p>
<ul>
<li>시퀀스가 정확히 20개인가?</li>
<li>종가는 0보다 큰가?</li>
<li>거래량은 0 이상인가?</li>
<li>JSON 필드 이름과 타입이 맞는가?</li>
</ul>
<h2 id="4-포트가-달라-연결이-거절된다">4. 포트가 달라 연결이 거절된다</h2>
<p>서버, 브라우저, <code>curl</code>, 테스트 스크립트가 같은 포트를 사용하는지 확인한다. 이 정리에서는 혼동을 줄이기 위해 <code>8000</code>으로 통일했다.</p>
<h2 id="5-재학습-후에도-예전-모델이-응답한다">5. 재학습 후에도 예전 모델이 응답한다</h2>
<p>이 문제는 뒤의 일차에서 더 중요하다. 전역 캐시에 예전 모델 객체가 남아 있으면 레지스트리의 새 모델이 생겨도 서버는 기존 객체를 계속 사용할 수 있다. 캐시 무효화나 안전한 재로딩 정책이 필요하다.</p>
<h2 id="6-재학습-요청이-너무-오래-걸린다">6. 재학습 요청이 너무 오래 걸린다</h2>
<p>긴 재학습을 HTTP 요청 처리 중 동기로 실행하면 사용자가 계속 기다리게 된다. 실무에서는 백그라운드 작업 큐나 별도 워커로 분리하는 방안을 고려한다. 이는 Day 1의 동기·비동기 개념이 운영 설계로 연결되는 예다.</p>
<hr />
<h1 id="초보자가-반드시-구분해야-하는-개념">초보자가 반드시 구분해야 하는 개념</h1>
<table>
<thead>
<tr>
<th>헷갈리는 쌍</th>
<th>차이</th>
</tr>
</thead>
<tbody><tr>
<td>학습 vs 서빙</td>
<td>규칙을 만드는 과정 vs 만들어진 규칙으로 새 요청에 답하는 과정</td>
</tr>
<tr>
<td>모델 파일 vs API</td>
<td>저장된 계산 자원 vs 다른 프로그램이 사용할 수 있는 통신 창구</td>
</tr>
<tr>
<td>FastAPI vs Uvicorn</td>
<td>API 애플리케이션 프레임워크 vs 애플리케이션을 실행하는 서버</td>
</tr>
<tr>
<td>Pydantic 스키마 vs DB 모델</td>
<td>요청·응답 검증 vs 영구 저장 구조</td>
</tr>
<tr>
<td>422 vs 500</td>
<td>입력이 계약을 위반함 vs 서버 내부 처리 실패</td>
</tr>
<tr>
<td>비동기 vs 병렬 계산</td>
<td>대기 중 다른 일을 처리함 vs 여러 계산을 실제로 동시에 수행함</td>
</tr>
<tr>
<td>Lazy vs Eager</td>
<td>첫 요청 때 로드 vs 서버 시작 때 로드</td>
</tr>
<tr>
<td>WebSocket vs WebRTC</td>
<td>지속적인 메시지·시그널링 채널 vs 실시간 P2P 미디어 전송</td>
</tr>
<tr>
<td>모니터링 vs 로깅</td>
<td>상태를 지속적으로 관찰·판정하는 활동 vs 사건을 기록하는 데이터</td>
</tr>
</tbody></table>
<hr />
<h1 id="day-1-완료-체크리스트">Day 1 완료 체크리스트</h1>
<ul>
<li><input disabled="" type="checkbox" /> 모델 서빙이 학습과 어떻게 다른지 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 배치 추론과 실시간 추론의 사용 사례를 구분할 수 있다.</li>
<li><input disabled="" type="checkbox" /> REST, gRPC, WebSocket, WebRTC의 역할 차이를 큰 그림에서 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> FastAPI 앱·라우터·핸들러의 조립 구조를 읽을 수 있다.</li>
<li><input disabled="" type="checkbox" /> Pydantic 스키마가 요청을 검증하고 422를 만드는 과정을 설명할 수 있다.</li>
<li><input disabled="" type="checkbox" /> API 스키마와 DB 모델의 역할 차이를 안다.</li>
<li><input disabled="" type="checkbox" /> Swagger UI에서 <code>/predict</code>와 <code>/health</code>를 직접 호출할 수 있다.</li>
<li><input disabled="" type="checkbox" /> 정상 입력은 200, 잘못된 시퀀스와 값은 422가 되는 것을 확인했다.</li>
<li><input disabled="" type="checkbox" /> CSV 업로드 후 baseline 모델과 스케일러를 생성했다.</li>
<li><input disabled="" type="checkbox" /> Lazy와 Eager에서 서버 시작·첫 요청·두 번째 요청 시간을 측정했다.</li>
<li><input disabled="" type="checkbox" /> 로그와 헬스체크가 AIOps의 기반이 되는 이유를 설명할 수 있다.</li>
</ul>
<h2 id="남겨-두면-좋은-실습-증빙">남겨 두면 좋은 실습 증빙</h2>
<ul>
<li><code>/docs</code>의 <code>/predict</code> 정상 요청·응답 화면</li>
<li>시퀀스 길이 오류 또는 잘못된 값에 대한 422 화면</li>
<li><code>/health</code>의 <code>model_loaded</code>, <code>loading_mode</code> 변화</li>
<li>baseline 학습 실행 결과</li>
<li>Lazy/Eager 측정표</li>
<li>시행착오를 <code>오류 메시지 → 원인 → 해결 방법 → 해결 후 결과</code> 순서로 적은 기록</li>
</ul>
<p>개인정보, 인증정보, 내부 경로, 비밀번호, 원본 사용자 입력 등은 캡처와 로그를 공개하기 전에 반드시 마스킹한다.</p>
<hr />
<h1 id="최종-요약">최종 요약</h1>
<p>Day 1의 핵심은 FastAPI 문법을 많이 외우는 것이 아니다. <strong>학습된 모델을 외부 요청과 연결하고, 잘못된 입력을 차단하고, 상태를 관찰하며, 문제가 생겼을 때 원인을 찾을 수 있는 서비스 형태로 만드는 것</strong>이다.</p>
<pre><code class="language-text">모델 파일
  ↓ FastAPI로 감싸기
입력 계약(Pydantic)
  ↓
추론 엔드포인트(/predict)
  ↓
상태 확인(/health) + 오류 처리 + 로그
  ↓
Swagger로 검증
  ↓
Day 2의 버전 관리·컨테이너화로 연결</code></pre>
<p>모델이 파일에서 서비스가 되는 순간부터 정확도 외에도 지연시간, 처리량, 가용성, 오류율, 버전, 로그가 중요해진다. 이것이 모델 개발에서 모델 운영으로 넘어가는 첫걸음이다.</p>