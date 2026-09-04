<h2 id="전체-지도-아키텍처는-문제-해결의-역사다">전체 지도: 아키텍처는 문제 해결의 역사다</h2>
<p>딥러닝 아키텍처는 이름을 외우기 위한 목록이 아닙니다. 기존 구조가 풀지 못한 문제를 해결하면서 발전한 역사입니다.</p>
<pre><code class="language-text">MLP
→ 이미지의 공간 구조를 잃음
→ CNN: 가까운 픽셀과 위치 공유 규칙 반영

RNN
→ 순서를 다루지만 먼 과거를 잊음
→ LSTM: 기억을 선택하는 Gate 추가

지도학습
→ 라벨 비용과 정보 손실
→ Autoencoder: 입력 자체를 정답으로 표현 학습

Seq2Seq
→ 문장 전체를 하나의 고정 벡터에 압축
→ Attention: 필요한 부분을 직접 참고

Transformer
→ 모든 토큰 관계 계산과 KV Cache 비용 증가
→ Linear Recurrent, MoE, Hybrid Architecture 탐색</code></pre>
<p>교재의 다섯 흐름은 다음과 같습니다.</p>
<ol>
<li><strong>귀납적 편향의 설계</strong><ul>
<li>MLP의 한계</li>
<li>CNN의 지역성·가중치 공유</li>
<li>RNN의 순차성과 시간 의존성</li>
<li>LSTM의 장기 기억과 Gate</li>
</ul>
</li>
<li><strong>표현 학습이라는 다른 길</strong><ul>
<li>지도학습의 한계</li>
<li>Autoencoder</li>
<li>Seq2Seq</li>
</ul>
</li>
<li><strong>CNN의 진화</strong><ul>
<li>AlexNet</li>
<li>ResNet</li>
</ul>
</li>
<li><strong>Attention 등장</strong><ul>
<li>Transformer</li>
<li>Q, K, V와 Multi-head Attention</li>
<li>BERT와 전이학습</li>
</ul>
</li>
<li><strong>규모와 비용의 시대</strong><ul>
<li>Transformer의 계산·메모리 한계</li>
<li>Linear Recurrent Models와 DeltaNet</li>
<li>Mixture of Experts</li>
<li>Kimi Linear 하이브리드 사례</li>
</ul>
</li>
</ol>
<hr />
<h1 id="0-architecture-algorithm-model-구분">0. Architecture, Algorithm, Model 구분</h1>
<p>세 용어를 먼저 구분하면 이후 설명이 쉬워집니다.</p>
<h2 id="architecture">Architecture</h2>
<p>신경망의 구조적 설계 방식입니다.</p>
<ul>
<li>Layer를 어떤 순서로 배치하는가?</li>
<li>층 사이를 어떻게 연결하는가?</li>
<li>데이터가 어떤 경로로 흐르는가?</li>
<li>CNN, RNN, ResNet, Transformer 등이 어떤 뼈대를 갖는가?</li>
</ul>
<p>건물에 비유하면 설계도입니다.</p>
<h2 id="algorithm">Algorithm</h2>
<p>문제를 해결하기 위한 계산 절차입니다.</p>
<ul>
<li>Forward Propagation</li>
<li>Backpropagation</li>
<li>Gradient Descent</li>
<li>Adam</li>
<li>데이터 전처리 절차</li>
</ul>
<p>건물을 짓는 시공 방법이나 작업 절차와 비슷합니다.</p>
<h2 id="model">Model</h2>
<p>Architecture에 데이터를 넣고 학습해 실제 Parameter가 결정된 결과물입니다.</p>
<pre><code class="language-text">Architecture + Algorithm + Data + Training
→ Trained Model</code></pre>
<p>같은 ResNet Architecture를 사용해도 의료 영상과 자동차 이미지를 각각 학습하면 서로 다른 Model이 됩니다.</p>
<hr />
<h1 id="교재가-제시하는-task별-architecture-지도">교재가 제시하는 Task별 Architecture 지도</h1>
<p>딥러닝 초창기에는 데이터와 Task에 따라 전용 Architecture를 구분해 설명하는 경우가 많았습니다.</p>
<table>
<thead>
<tr>
<th>영역</th>
<th>대표 구조</th>
<th>데이터에 넣은 핵심 가정</th>
<th>대표 Task</th>
</tr>
</thead>
<tbody><tr>
<td>Image·Vision</td>
<td>CNN</td>
<td>가까운 Pixel과 공간 Pattern이 중요</td>
<td>분류, 탐지, Segmentation</td>
</tr>
<tr>
<td>Text·Time Series</td>
<td>RNN, LSTM</td>
<td>순서와 과거 상태가 현재에 영향</td>
<td>번역, 예측, 음성</td>
</tr>
<tr>
<td>Generation</td>
<td>GAN</td>
<td>Generator와 Discriminator의 경쟁 학습</td>
<td>Image 생성, Data 생성</td>
</tr>
<tr>
<td>Control·Optimization</td>
<td>Reinforcement Learning</td>
<td>Agent가 Environment에서 Reward로 학습</td>
<td>Game, Robot, 자율주행</td>
</tr>
</tbody></table>
<h3 id="gan의-기본-구조">GAN의 기본 구조</h3>
<ul>
<li>Generator: 실제와 비슷한 Data를 생성</li>
<li>Discriminator: 실제 Data와 생성 Data를 구분</li>
<li>두 Network가 경쟁하며 함께 개선</li>
</ul>
<p>GAN은 본 교재에서 이후 자세히 전개되는 중심 단원은 아니지만, 생성 Architecture가 분류 중심 Network와 다른 문제를 다룬다는 전체 지도에 포함됩니다.</p>
<h3 id="reinforcement-learning의-기본-구조">Reinforcement Learning의 기본 구조</h3>
<ul>
<li>Agent: 행동을 선택하는 주체</li>
<li>Environment: Agent가 상호작용하는 세계</li>
<li>Action: Agent의 선택</li>
<li>Reward: 행동 결과에 대한 Feedback</li>
<li>Policy: 상태에서 행동을 고르는 전략</li>
</ul>
<p>강화학습은 특정 Neural Architecture 하나라기보다 DQN, Policy Network, Actor-Critic처럼 Model과 학습 Algorithm을 결합하는 방법론입니다.</p>
<h3 id="architecture와-학습-방식은-다른-축">Architecture와 학습 방식은 다른 축</h3>
<p>Supervised, Semi-supervised, Self-supervised, Unsupervised, Reinforcement Learning은 <strong>어떤 Feedback으로 학습하는가</strong>를 나타냅니다. CNN, RNN, Transformer는 <strong>Network를 어떻게 연결하는가</strong>를 나타냅니다.</p>
<p>따라서 CNN을 Self-supervised로 학습하거나 Transformer를 Reinforcement Learning으로 추가 학습할 수도 있습니다.</p>
<p>정형 Data도 MLP만 사용하는 것은 아닙니다. TabNet 같은 정형 전용 구조, TCN 같은 Sequence 구조, Transfer Learning·Few-shot·Meta Learning 등 다양한 확장이 존재합니다. 현대 Model은 Text·Image·Audio를 하나의 Multi-modal Architecture에서 함께 처리하기도 합니다.</p>
<hr />
<h1 id="아키텍처-발전의-큰-흐름">아키텍처 발전의 큰 흐름</h1>
<h2 id="computer-vision">Computer Vision</h2>
<table>
<thead>
<tr>
<th>시기</th>
<th>구조</th>
<th>핵심 기여</th>
</tr>
</thead>
<tbody><tr>
<td>초기</td>
<td>CNN</td>
<td>이미지의 지역 패턴을 Filter로 학습</td>
</tr>
<tr>
<td>1990년대</td>
<td>LeNet-5</td>
<td>손글씨 인식용 초기 CNN 구조 정립</td>
</tr>
<tr>
<td>2012</td>
<td>AlexNet</td>
<td>GPU, ReLU, Dropout으로 대규모 CNN의 가능성 입증</td>
</tr>
<tr>
<td>2014</td>
<td>VGGNet</td>
<td>작은 3×3 Filter를 반복하는 단순한 설계</td>
</tr>
<tr>
<td>2014</td>
<td>GoogLeNet/Inception</td>
<td>여러 크기의 연산을 병렬 결합</td>
</tr>
<tr>
<td>2015</td>
<td>ResNet</td>
<td>Skip Connection으로 매우 깊은 망 학습 안정화</td>
</tr>
<tr>
<td>이후</td>
<td>DenseNet</td>
<td>층 사이의 조밀한 연결과 Feature 재사용</td>
</tr>
<tr>
<td>이후</td>
<td>YOLO</td>
<td>한 번의 흐름으로 빠른 객체 탐지</td>
</tr>
<tr>
<td>이후</td>
<td>EfficientNet</td>
<td>깊이·너비·해상도를 균형 있게 확장</td>
</tr>
<tr>
<td>2020 전후</td>
<td>ViT</td>
<td>이미지를 Patch로 나눠 Transformer 적용</td>
</tr>
</tbody></table>
<h2 id="nlp와-sequence">NLP와 Sequence</h2>
<table>
<thead>
<tr>
<th>시기</th>
<th>구조</th>
<th>핵심 기여</th>
</tr>
</thead>
<tbody><tr>
<td>1980년대</td>
<td>RNN</td>
<td>이전 상태를 다음 시점에 전달</td>
</tr>
<tr>
<td>1997</td>
<td>LSTM</td>
<td>Gate와 Cell State로 장기 의존성 개선</td>
</tr>
<tr>
<td>2014</td>
<td>Word2Vec</td>
<td>단어 의미를 연속 벡터 공간에 표현</td>
</tr>
<tr>
<td>2014~2015</td>
<td>Seq2Seq</td>
<td>Encoder-Decoder로 시퀀스를 다른 시퀀스로 변환</td>
</tr>
<tr>
<td>2017</td>
<td>Transformer</td>
<td>Self-Attention과 병렬 처리</td>
</tr>
<tr>
<td>2018</td>
<td>BERT</td>
<td>Encoder 기반 양방향 문맥 사전학습</td>
</tr>
<tr>
<td>이후</td>
<td>GPT 계열</td>
<td>Decoder 기반 Autoregressive 생성 확장</td>
</tr>
<tr>
<td>이후</td>
<td>DALL-E·CLIP 등</td>
<td>텍스트와 이미지의 Multi-modal 학습</td>
</tr>
</tbody></table>
<p>이 연도는 발표·공개·학회 출판 시점을 어떻게 잡는지에 따라 다르게 표기될 수 있습니다. 중요한 것은 정확한 숫자 암기보다 어떤 문제가 다음 구조를 만들었는지 이해하는 것입니다.</p>
<hr />
<h1 id="part-1a-귀납적-편향의-설계---cnn">Part 1A. 귀납적 편향의 설계 - CNN</h1>
<h2 id="1-귀납적-편향이란">1. 귀납적 편향이란?</h2>
<p>귀납적 편향은 모델이 제한된 데이터로도 합리적으로 일반화하도록 미리 넣어 둔 가정이나 구조적 제약입니다.</p>
<p>예를 들어 이미지에는 다음 가정이 유용합니다.</p>
<ul>
<li>가까운 픽셀끼리 의미 있는 패턴을 만든다.</li>
<li>모서리 검출기는 이미지 어디에서나 유용하다.</li>
<li>고양이가 왼쪽에 있든 오른쪽에 있든 고양이라는 정체는 유지된다.</li>
</ul>
<p>CNN은 이런 이미지의 성질을 Architecture에 새긴 모델입니다.</p>
<h2 id="2-mlp가-이미지에-불리한-이유">2. MLP가 이미지에 불리한 이유</h2>
<p>MLP는 한 층의 모든 뉴런이 다음 층의 모든 뉴런과 연결된 Fully Connected 구조입니다. 표 형태의 독립적인 Feature에는 유용하지만 이미지를 그대로 다루면 문제가 생깁니다.</p>
<h3 id="21-공간-구조-소실">2.1 공간 구조 소실</h3>
<p>28×28 이미지를 784개 Vector로 펼치면 어떤 픽셀이 위·아래·옆에 있었는지 구조가 직접 드러나지 않습니다.</p>
<pre><code class="language-text">원래: 28 × 28 격자
Flatten 후: 길이 784의 일렬 Vector</code></pre>
<h3 id="22-parameter-폭발">2.2 Parameter 폭발</h3>
<p>입력 784개와 Hidden Unit 1,000개를 Fully Connected로 연결하면 가중치만 약 784,000개가 필요합니다.</p>
<p>이미지가 640×640×3처럼 커지면 훨씬 심각해집니다.</p>
<h3 id="23-위치-의존">2.3 위치 의존</h3>
<p>MLP는 각 입력 위치마다 별도 가중치를 학습합니다. 왼쪽 위에서 배운 모서리 규칙을 오른쪽 아래에 자동으로 재사용하지 못합니다.</p>
<p>CNN은 격자 유지, Local Connection, Weight Sharing으로 이 문제를 해결합니다.</p>
<h2 id="3-mnist로-보는-cnn">3. MNIST로 보는 CNN</h2>
<p>MNIST는 0부터 9까지의 손글씨 숫자 이미지 데이터입니다. 28×28 흑백 픽셀을 입력받아 숫자 10개 클래스 중 하나를 예측합니다.</p>
<p>CNN은 다음처럼 점점 추상적인 Feature를 학습할 수 있습니다.</p>
<pre><code class="language-text">초기 Layer: 짧은 선, 모서리
중간 Layer: 곡선, 교차점
높은 Layer: 숫자 3의 위·아래 곡선
출력 Layer: 0~9 클래스 확률</code></pre>
<h2 id="4-convolution">4. Convolution</h2>
<p>Convolution은 작은 Filter가 입력 위를 이동하며 지역 영역과 연산해 새로운 Feature Map을 만드는 과정입니다.</p>
<p>딥러닝에서는 엄밀한 수학적 Convolution보다 Kernel을 뒤집지 않는 Cross-correlation 연산을 구현하는 경우가 많지만 관례상 Convolution이라고 부릅니다.</p>
<h2 id="5-convolution-layer">5. Convolution Layer</h2>
<p>이미지 입력과 Filter의 지역 연산으로 Feature Map을 만듭니다.</p>
<h3 id="51-kernel과-filter">5.1 Kernel과 Filter</h3>
<p>초보 단계에서는 둘을 같은 뜻으로 사용해도 무방합니다.</p>
<ul>
<li>작은 크기의 가중치 행렬</li>
<li>이미지 위를 이동</li>
<li>각 위치에서 원소별 곱 후 합산</li>
<li>학습을 통해 Edge, Texture, Shape에 반응</li>
</ul>
<p>3×3 Filter의 계산은 다음과 같습니다.</p>
<p>$$
z_{i,j}=\sum_{u=0}^{2}\sum_{v=0}^{2}
x_{i+u,j+v}k_{u,v}+b
$$</p>
<p>Filter 값은 사람이 항상 정하는 것이 아니라 Backpropagation으로 학습됩니다.</p>
<h3 id="52-stride">5.2 Stride</h3>
<p>Filter가 한 번에 몇 칸 이동하는지 나타냅니다.</p>
<ul>
<li>Stride 1: 한 칸씩 이동, 세밀한 Feature Map</li>
<li>Stride 2: 두 칸씩 이동, 출력 공간 크기 감소</li>
</ul>
<h3 id="53-padding">5.3 Padding</h3>
<p>입력 가장자리에 주로 0을 추가합니다.</p>
<p>목적:</p>
<ul>
<li>가장자리 정보도 충분히 연산</li>
<li>출력 공간 크기 조절</li>
<li>깊은 층에서 크기가 너무 빨리 줄어드는 문제 완화</li>
</ul>
<p>2차원 Convolution의 출력 크기는 한 축 기준으로 다음과 같습니다.</p>
<p>$$
Output=
\left\lfloor
\frac{Input+2P-K}{S}
\right\rfloor+1
$$</p>
<ul>
<li>$P$: Padding</li>
<li>$K$: Kernel Size</li>
<li>$S$: Stride</li>
</ul>
<p>입력 5, Kernel 3, Padding 1, Stride 1이면 출력은 5입니다.</p>
<h3 id="54-channel과-filter-수">5.4 Channel과 Filter 수</h3>
<p>RGB 입력은 Channel 3개입니다. 하나의 2D Feature Map을 만들려는 Filter는 세 Channel 전체를 함께 봅니다.</p>
<p>Filter를 32개 사용하면 출력 Channel은 32개입니다.</p>
<pre><code class="language-text">입력: H × W × 3
Filter: 3 × 3 × 3, 총 32개
출력: H' × W' × 32</code></pre>
<h2 id="6-cnn의-핵심-귀납적-편향">6. CNN의 핵심 귀납적 편향</h2>
<h3 id="locality">Locality</h3>
<p>의미 있는 작은 패턴은 가까운 픽셀 사이에 있다고 가정합니다.</p>
<h3 id="weight-sharing">Weight Sharing</h3>
<p>하나의 Filter를 모든 위치에 반복 사용합니다.</p>
<p>효과:</p>
<ul>
<li>Parameter 절약</li>
<li>한 위치에서 학습한 Pattern을 다른 위치에도 적용</li>
</ul>
<h3 id="translation-equivariance">Translation Equivariance</h3>
<p>입력이 이동하면 Feature Map의 반응도 함께 이동합니다. 이것이 Convolution의 기본 성질입니다.</p>
<h3 id="일부-translation-invariance">일부 Translation Invariance</h3>
<p>Pooling이나 Global Aggregation을 거치면 작은 위치 이동에 출력이 덜 민감해질 수 있습니다.</p>
<blockquote>
<p>Convolution 자체를 완전한 위치 불변이라고 부르기보다 <strong>이동 등변성</strong>, Pooling 등이 일부 <strong>이동 불변성</strong>을 제공한다고 구분하면 정확합니다.</p>
</blockquote>
<h2 id="7-pooling-layer">7. Pooling Layer</h2>
<p>Pooling은 지역 영역을 대표값으로 줄입니다.</p>
<h3 id="max-pooling">Max Pooling</h3>
<p>2×2 영역에서 가장 큰 값을 선택합니다.</p>
<ul>
<li>공간 크기 감소</li>
<li>계산량 감소</li>
<li>강하게 검출된 Feature 유지</li>
<li>작은 위치 변화에 덜 민감</li>
<li>학습 Parameter 없음</li>
</ul>
<p>다만 Pooling은 세부 위치 정보를 버립니다. Segmentation처럼 정밀한 위치가 필요한 문제에서는 과도한 Pooling이 불리할 수 있고 Strided Convolution을 대신 사용하기도 합니다.</p>
<h2 id="8-flatten-dense-softmax">8. Flatten, Dense, Softmax</h2>
<h3 id="flatten">Flatten</h3>
<p>다차원 Feature Map을 1차원 Vector로 모양만 바꿉니다. 학습 Parameter는 없습니다.</p>
<h3 id="dense-또는-fully-connected">Dense 또는 Fully Connected</h3>
<p>Flatten된 Feature를 클래스 점수로 변환하는 학습 Layer입니다.</p>
<blockquote>
<p>Flatten과 Dense는 같은 Layer가 아닙니다. Flatten은 Reshape, Dense는 가중치를 가진 학습 연산입니다.</p>
</blockquote>
<h3 id="softmax">Softmax</h3>
<p>클래스별 Logit을 합이 1인 확률 분포로 바꿉니다.</p>
<p>$$
P(y=i)=\frac{e^{z_i}}{\sum_{j=1}^{K}e^{z_j}}
$$</p>
<ul>
<li>Softmax: 점수를 확률 분포로 변환</li>
<li>Argmax: 가장 큰 값의 위치를 선택</li>
</ul>
<p>Softmax가 최댓값임을 증명하는 것은 아닙니다. 두 기능을 구분해야 합니다.</p>
<h2 id="9-전형적인-cnn-분류-흐름">9. 전형적인 CNN 분류 흐름</h2>
<pre><code class="language-text">Image
→ Conv + ReLU
→ Pooling
→ Conv + ReLU
→ Pooling
→ Flatten 또는 Global Average Pooling
→ Dense
→ Softmax</code></pre>
<ul>
<li>Conv: Feature 추출</li>
<li>Pooling: 공간 크기 축소</li>
<li>Dense: Feature 결합</li>
<li>Softmax: 클래스 확률</li>
</ul>
<p>현대 CNN에서는 Flatten과 큰 Dense Layer 대신 Global Average Pooling을 사용해 Parameter를 줄이는 경우가 많습니다.</p>
<h2 id="10-vision-task-구분">10. Vision Task 구분</h2>
<table>
<thead>
<tr>
<th>Task</th>
<th>질문</th>
<th>출력</th>
</tr>
</thead>
<tbody><tr>
<td>Classification</td>
<td>사진 전체가 무엇인가?</td>
<td>클래스</td>
</tr>
<tr>
<td>Localization</td>
<td>한 물체가 무엇이고 어디 있는가?</td>
<td>클래스 + Bounding Box</td>
</tr>
<tr>
<td>Detection</td>
<td>여러 물체가 무엇이고 어디 있는가?</td>
<td>여러 Box + 클래스</td>
</tr>
<tr>
<td>Segmentation</td>
<td>각 Pixel이 무엇인가?</td>
<td>Pixel 단위 Mask</td>
</tr>
</tbody></table>
<h2 id="part-1a-핵심-정리">Part 1A 핵심 정리</h2>
<ul>
<li>CNN은 이미지의 Locality와 Weight Sharing을 구조에 반영합니다.</li>
<li>Kernel은 지역 영역을 훑어 Feature Map을 만듭니다.</li>
<li>Stride와 Padding이 출력 크기를 조절합니다.</li>
<li>Pooling은 공간 정보를 줄이고 강한 반응을 남깁니다.</li>
<li>Flatten, Dense, Softmax, Argmax는 역할이 서로 다릅니다.</li>
</ul>
<hr />
<h1 id="part-1b-귀납적-편향의-설계---rnn과-lstm">Part 1B. 귀납적 편향의 설계 - RNN과 LSTM</h1>
<h2 id="1-sequence-data">1. Sequence Data</h2>
<p>Sequence는 순서가 의미를 만드는 데이터입니다.</p>
<ul>
<li>자연어 문장</li>
<li>음성</li>
<li>센서 시계열</li>
<li>주가와 수요</li>
<li>사용자 행동 Log</li>
</ul>
<pre><code class="language-text">사람이 사과를 먹었다
≠
사과가 사람을 먹었다</code></pre>
<p>같은 단어라도 순서가 바뀌면 의미가 바뀝니다.</p>
<h2 id="2-recurrent의-의미">2. Recurrent의 의미</h2>
<p>RNN은 같은 Cell 계산을 시간축에서 반복하고, 이전 Hidden State를 현재 계산에 사용합니다.</p>
<p>일기 쓰기에 비유하면 오늘의 일기는 오늘 사건만으로 작성되지 않습니다. 어제까지 쌓인 기억이 오늘 기록에 영향을 줍니다.</p>
<h2 id="3-rnn의-기본-계산">3. RNN의 기본 계산</h2>
<p>$$
h_t=\tanh(W_{hh}h_{t-1}+W_{xh}x_t+b_h)
$$</p>
<p>$$
y_t=W_{hy}h_t+b_y
$$</p>
<ul>
<li>$x_t$: 현재 입력</li>
<li>$h_{t-1}$: 이전 Hidden State</li>
<li>$h_t$: 현재까지의 요약 기억</li>
<li>$y_t$: 현재 출력</li>
</ul>
<h2 id="4-cell과-hidden-state">4. Cell과 Hidden State</h2>
<h3 id="cell">Cell</h3>
<p>각 시점에서 동일한 규칙으로 입력과 이전 상태를 처리하는 연산 단위입니다.</p>
<h3 id="hidden-state">Hidden State</h3>
<p>지금까지 본 Sequence 정보를 고정 크기 Vector로 요약합니다. 다음 시점에 전달되는 기억입니다.</p>
<p>RNN 도식을 펼쳐 그리면 Cell이 여러 개처럼 보이지만 실제로는 시간축에서 같은 Parameter를 공유합니다.</p>
<h2 id="5-시간축-가중치-공유">5. 시간축 가중치 공유</h2>
<p>입력 길이가 늘어도 같은 $W_{xh}$와 $W_{hh}$를 재사용합니다.</p>
<p>장점:</p>
<ul>
<li>Sequence 길이에 비례해 Parameter 수가 증가하지 않음</li>
<li>가변 길이 입력 처리</li>
<li>같은 Pattern을 어느 시점에서도 탐지</li>
</ul>
<p>CNN이 공간 위치에서 가중치를 공유한다면 RNN은 시간 위치에서 가중치를 공유합니다.</p>
<h2 id="6-rnn의-다양한-입출력">6. RNN의 다양한 입출력</h2>
<table>
<thead>
<tr>
<th>구조</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>One-to-One</td>
<td>일반 분류</td>
</tr>
<tr>
<td>One-to-Many</td>
<td>이미지에서 문장 생성</td>
</tr>
<tr>
<td>Many-to-One</td>
<td>문장 감성 분류</td>
</tr>
<tr>
<td>Many-to-Many</td>
<td>번역, 각 시점 Labeling</td>
</tr>
</tbody></table>
<h3 id="단방향과-양방향">단방향과 양방향</h3>
<ul>
<li>단방향 RNN: 과거에서 미래 방향</li>
<li>Bidirectional RNN: 앞과 뒤 문맥을 모두 사용</li>
</ul>
<p>실시간 미래 예측에서는 미래 정보가 없으므로 양방향 구조를 그대로 사용할 수 없습니다.</p>
<h2 id="7-bptt">7. BPTT</h2>
<p>RNN의 학습은 Backpropagation Through Time으로 수행합니다.</p>
<p>시간축으로 펼친 Network를 거꾸로 따라가며 각 시점의 Parameter 기여도를 계산합니다.</p>
<p>긴 Sequence에서는 작은 미분값이 반복해서 곱해져 Gradient Vanishing이, 큰 값이 반복되면 Gradient Exploding이 발생할 수 있습니다.</p>
<h2 id="8-rnn-출력과-softmax">8. RNN 출력과 Softmax</h2>
<ul>
<li>연속값 시계열 예측: 선형 출력으로 수치 예측</li>
<li>다음 단어 예측: Vocabulary 전체에 Softmax 적용</li>
</ul>
<p>단어 예측에서 Hidden State가 Vocabulary 크기의 Logit으로 변환되고 Cross Entropy로 학습됩니다.</p>
<h2 id="9-tanh의-역할과-한계">9. Tanh의 역할과 한계</h2>
<p>Tanh는 Hidden State를 -1과 1 사이로 제한합니다.</p>
<p>입력 Trend가 상승하면 State가 양수 방향으로, 하락 정보가 강하면 음수 방향으로 움직일 수 있습니다.</p>
<p>하지만 입력의 절댓값이 커지면 Tanh가 -1이나 1 근처에서 포화되어 서로 다른 큰 값을 구분하기 어려워지고 Gradient가 작아집니다. 그래서 Hidden State 숫자를 실제 Target 수치와 동일하게 해석하면 안 됩니다. State는 학습된 내부 표현입니다.</p>
<h2 id="10-lstm이-등장한-이유">10. LSTM이 등장한 이유</h2>
<p>기본 RNN은 모든 과거 정보를 하나의 Hidden State에 계속 덮어씁니다. 먼 과거의 중요한 정보가 희미해질 수 있습니다.</p>
<p>LSTM은 별도의 Cell State와 Gate를 추가해 무엇을 기억하고 무엇을 잊을지 학습합니다.</p>
<h2 id="11-lstm의-두-상태">11. LSTM의 두 상태</h2>
<ul>
<li><strong>Cell State $c_t$</strong>: 장기 기억이 흐르는 경로</li>
<li><strong>Hidden State $h_t$</strong>: 현재 시점의 출력·단기 표현</li>
</ul>
<p>일기와 중요 메모장을 함께 쓰는 것과 비슷합니다.</p>
<h2 id="12-lstm-gate">12. LSTM Gate</h2>
<h3 id="forget-gate">Forget Gate</h3>
<p>이전 기억을 얼마나 유지할지 결정합니다.</p>
<p>$$
f_t=\sigma(W_f[h_{t-1},x_t]+b_f)
$$</p>
<h3 id="input-gate">Input Gate</h3>
<p>새 정보를 얼마나 기록할지 정합니다.</p>
<p>$$
i_t=\sigma(W_i[h_{t-1},x_t]+b_i)
$$</p>
<p>새 후보 기억:</p>
<p>$$
\tilde{c}<em>t=\tanh(W_c[h</em>{t-1},x_t]+b_c)
$$</p>
<h3 id="cell-state-update">Cell State Update</h3>
<p>$$
c_t=f_t\odot c_{t-1}+i_t\odot\tilde{c}_t
$$</p>
<h3 id="output-gate">Output Gate</h3>
<p>현재 기억에서 무엇을 출력할지 정합니다.</p>
<p>$$
o_t=\sigma(W_o[h_{t-1},x_t]+b_o)
$$</p>
<p>$$
h_t=o_t\odot\tanh(c_t)
$$</p>
<p>Gate 값은 사람이 고정 규칙으로 작성하지 않고 데이터로 학습됩니다.</p>
<h2 id="13-rnn과-lstm-비교">13. RNN과 LSTM 비교</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>RNN</th>
<th>LSTM</th>
</tr>
</thead>
<tbody><tr>
<td>기억</td>
<td>Hidden State</td>
<td>Hidden + Cell State</td>
</tr>
<tr>
<td>제어</td>
<td>단순 반복 계산</td>
<td>Forget·Input·Output Gate</td>
</tr>
<tr>
<td>장기 의존성</td>
<td>취약</td>
<td>더 잘 유지</td>
</tr>
<tr>
<td>계산량</td>
<td>적음</td>
<td>더 많음</td>
</tr>
<tr>
<td>병렬화</td>
<td>시간 순서 의존</td>
<td>시간 순서 의존</td>
</tr>
<tr>
<td>긴 문맥</td>
<td>정보 손실 가능</td>
<td>개선되지만 여전히 한계</td>
</tr>
</tbody></table>
<p>LSTM이 Gradient 문제를 완전히 없애거나 무한히 긴 문맥을 완벽히 기억하는 것은 아닙니다. 이런 병목이 Attention의 등장 배경이 됩니다.</p>
<h2 id="part-1b-핵심-정리">Part 1B 핵심 정리</h2>
<ul>
<li>RNN은 현재 입력과 이전 Hidden State를 함께 사용합니다.</li>
<li>시간축에서 Parameter를 공유해 가변 길이를 처리합니다.</li>
<li>BPTT는 긴 Sequence에서 Gradient 문제를 겪을 수 있습니다.</li>
<li>LSTM은 Cell State와 Gate로 기억의 보존·삭제·출력을 조절합니다.</li>
</ul>
<hr />
<h1 id="part-2-표현-학습이라는-다른-길">Part 2. 표현 학습이라는 다른 길</h1>
<h2 id="1-지도학습의-한계">1. 지도학습의 한계</h2>
<p>지도학습은 모든 입력에 Label이 필요합니다.</p>
<h3 id="비용">비용</h3>
<ul>
<li>사람이 직접 Labeling</li>
<li>의료·법률·제조처럼 전문가 Label은 더 비쌈</li>
</ul>
<h3 id="정답의-모호함">정답의 모호함</h3>
<p>감정, 품질, 유사성처럼 하나의 정답을 정하기 어려운 문제가 있습니다.</p>
<h3 id="label-이외-정보-손실">Label 이외 정보 손실</h3>
<p>고양이·강아지 Label만 학습하면 배경, 자세, 조명, Texture처럼 데이터에 담긴 풍부한 구조를 충분히 활용하지 못할 수 있습니다.</p>
<p>여기서 나온 발상은 다음과 같습니다.</p>
<blockquote>
<p>정답이 부족하다면 입력 자체에서 학습 목표를 만들자.</p>
</blockquote>
<p>Unsupervised Learning과 Self-supervised Learning의 중요한 출발점입니다.</p>
<h2 id="2-autoencoder">2. Autoencoder</h2>
<p>Autoencoder는 입력을 압축한 뒤 다시 원본으로 복원하도록 학습합니다.</p>
<pre><code class="language-text">입력 x
→ Encoder
→ Latent z
→ Decoder
→ 복원 x_hat</code></pre>
<p>목표:</p>
<p>$$
x\approx\hat{x}
$$</p>
<p>복원을 잘하려면 Bottleneck에 중요한 정보를 보존해야 하므로 표현 학습이 일어납니다.</p>
<h2 id="3-autoencoder-구성">3. Autoencoder 구성</h2>
<h3 id="encoder">Encoder</h3>
<p>입력을 낮은 차원의 표현으로 압축합니다.</p>
<p>$$
z=f_\theta(x)
$$</p>
<h3 id="latent-space와-bottleneck">Latent Space와 Bottleneck</h3>
<p>가장 압축된 내부 표현입니다. 데이터의 핵심 Feature가 담기기를 기대합니다.</p>
<h3 id="decoder">Decoder</h3>
<p>Latent 표현으로 입력을 복원합니다.</p>
<p>$$
\hat{x}=g_\phi(z)
$$</p>
<h3 id="reconstruction-loss">Reconstruction Loss</h3>
<p>입력과 복원 결과의 차이를 줄입니다.</p>
<p>이미지에서는 MSE, BCE, Perceptual Loss 등 목적에 맞는 손실을 사용할 수 있습니다.</p>
<h2 id="4-차원의-저주와-pca-비교">4. 차원의 저주와 PCA 비교</h2>
<p>차원이 커질수록 공간은 희박해지고 충분한 Pattern을 학습하는 데 더 많은 데이터가 필요합니다.</p>
<p>PCA는 분산이 큰 선형 방향으로 압축합니다. Autoencoder는 활성화 함수와 깊은 Network를 사용해 비선형 압축을 학습할 수 있습니다.</p>
<table>
<thead>
<tr>
<th>항목</th>
<th>PCA</th>
<th>Autoencoder</th>
</tr>
</thead>
<tbody><tr>
<td>관계</td>
<td>선형</td>
<td>비선형 가능</td>
</tr>
<tr>
<td>학습</td>
<td>고유벡터 계산</td>
<td>Gradient 기반</td>
</tr>
<tr>
<td>구조</td>
<td>비교적 단순</td>
<td>Architecture 선택 필요</td>
</tr>
<tr>
<td>복원</td>
<td>선형 복원</td>
<td>복잡한 Decoder 가능</td>
</tr>
</tbody></table>
<p>단순 선형 Autoencoder와 MSE를 특정 조건으로 학습하면 PCA와 비슷한 부분 공간을 얻을 수 있습니다.</p>
<h2 id="5-latent-feature의-활용">5. Latent Feature의 활용</h2>
<p>Latent Vector를 다음 과제에 사용할 수 있습니다.</p>
<ul>
<li>Classification</li>
<li>Clustering</li>
<li>유사 이미지 검색</li>
<li>Visualization</li>
<li>Anomaly Detection</li>
<li>생성 모델의 기반</li>
</ul>
<p>복원 학습으로 유용한 Feature를 배웠다면 작은 Classifier를 연결할 수 있습니다.</p>
<h2 id="6-vanilla-autoencoder의-한계">6. Vanilla Autoencoder의 한계</h2>
<h3 id="좋은-복원과-좋은-표현은-다름">좋은 복원과 좋은 표현은 다름</h3>
<p>Pixel을 잘 복사하는 Feature가 분류나 검색에도 유용하다는 보장은 없습니다.</p>
<h3 id="latent-space의-불규칙성">Latent Space의 불규칙성</h3>
<p>Latent 공간의 임의 지점을 Decoder에 넣어도 의미 있는 출력이 나온다는 보장이 없습니다. 따라서 Vanilla AE를 곧바로 생성 모델로 보기는 어렵습니다.</p>
<h3 id="압축-크기-선택">압축 크기 선택</h3>
<ul>
<li>너무 큼: 입력을 단순 복사할 위험</li>
<li>너무 작음: 필요한 정보까지 손실</li>
</ul>
<h3 id="분포-밖-입력">분포 밖 입력</h3>
<p>학습 분포와 다른 입력은 제대로 복원하지 못할 수 있습니다. 이를 Anomaly Detection에 활용할 수도 있지만 신뢰성 검증이 필요합니다.</p>
<h2 id="7-autoencoder의-확장">7. Autoencoder의 확장</h2>
<table>
<thead>
<tr>
<th>종류</th>
<th>핵심 아이디어</th>
<th>활용</th>
</tr>
</thead>
<tbody><tr>
<td>Vanilla AE</td>
<td>기본 압축·복원</td>
<td>차원 축소, Feature 추출</td>
</tr>
<tr>
<td>Denoising AE</td>
<td>손상된 입력에서 깨끗한 원본 복원</td>
<td>Robust Feature, Noise 제거</td>
</tr>
<tr>
<td>Sparse AE</td>
<td>활성값에 희소성 제약</td>
<td>중요한 Feature 선택</td>
</tr>
<tr>
<td>VAE</td>
<td>Latent를 확률 분포로 규제</td>
<td>생성, 연속적인 Latent Space</td>
</tr>
<tr>
<td>Convolutional AE</td>
<td>CNN으로 공간 구조 반영</td>
<td>이미지 압축·복원</td>
</tr>
<tr>
<td>Sequence AE</td>
<td>RNN·LSTM·GRU 사용</td>
<td>Sequence Pattern 압축</td>
</tr>
</tbody></table>
<h3 id="denoising-ae">Denoising AE</h3>
<p>입력에 일부 Noise를 넣고 깨끗한 원본을 복원하게 합니다. 단순 복사보다 본질적인 Feature를 배우도록 유도합니다.</p>
<h3 id="sparse-ae">Sparse AE</h3>
<p>활성 뉴런 수를 제한하거나 L1·KL 제약을 사용해 적은 Feature만 강하게 반응하도록 합니다.</p>
<h3 id="vae">VAE</h3>
<p>Encoder가 하나의 점이 아니라 평균과 분산을 출력하도록 하고 Latent 분포를 정규화합니다. 가까운 Latent 지점이 비슷한 출력을 만들도록 유도해 생성에 적합한 공간을 만듭니다.</p>
<h2 id="8-seq2seq">8. Seq2Seq</h2>
<p>Autoencoder가 입력을 자기 자신으로 복원한다면 Seq2Seq는 입력 Sequence를 다른 Sequence로 변환합니다.</p>
<ul>
<li>기계 번역</li>
<li>문서 요약</li>
<li>초기 Chatbot</li>
<li>음성 인식</li>
<li>시계열 변환</li>
</ul>
<pre><code class="language-text">입력 Sequence
→ Encoder
→ Context Vector
→ Decoder
→ 출력 Sequence</code></pre>
<p>출력 길이는 입력 길이와 다를 수 있습니다.</p>
<h2 id="9-고정-길이-context-병목">9. 고정 길이 Context 병목</h2>
<p>초기 Seq2Seq는 Encoder의 마지막 Hidden State 하나에 입력 전체를 압축했습니다.</p>
<p>문장이 길어지면 하나의 고정 길이 Vector에 모든 정보를 담기 어려워 앞부분이 손실됩니다. Decoder가 각 출력 단어를 만들 때 Encoder의 모든 상태 중 필요한 부분을 직접 참고하게 한 것이 Attention입니다.</p>
<h2 id="part-2-핵심-정리">Part 2 핵심 정리</h2>
<ul>
<li>Label은 비싸고 데이터의 모든 구조를 설명하지 못합니다.</li>
<li>Autoencoder는 입력을 정답으로 삼아 압축과 복원을 학습합니다.</li>
<li>Latent Space는 유용할 수 있지만 좋은 복원이 좋은 표현을 자동 보장하지 않습니다.</li>
<li>Seq2Seq는 입력 Sequence를 다른 Sequence로 바꿉니다.</li>
<li>고정 Context Vector 병목이 Attention의 등장 배경입니다.</li>
</ul>
<hr />
<h1 id="part-3-cnn의-진화">Part 3. CNN의 진화</h1>
<h2 id="1-imagenet-challenge와-깊이">1. ImageNet Challenge와 깊이</h2>
<p>ImageNet 규모의 데이터와 GPU 발전은 더 큰 CNN을 가능하게 했습니다.</p>
<p>초기 흐름:</p>
<pre><code class="language-text">AlexNet
→ VGGNet
→ GoogLeNet
→ ResNet</code></pre>
<p>단순히 Layer를 늘리면 표현력은 커질 수 있지만 실제로는 최적화가 어려워져 Training Error조차 커지는 Degradation 문제가 나타났습니다.</p>
<h2 id="2-alexnet">2. AlexNet</h2>
<p>AlexNet은 대규모 이미지 분류에서 딥러닝의 가능성을 널리 보여준 구조입니다.</p>
<h3 id="주요-구성">주요 구성</h3>
<ul>
<li>5개 Convolution Layer</li>
<li>3개 Fully Connected Layer</li>
<li>ReLU</li>
<li>Dropout</li>
<li>GPU 학습</li>
<li>Data Augmentation</li>
<li>당시 사용된 Local Response Normalization</li>
</ul>
<h3 id="relu">ReLU</h3>
<p>Sigmoid보다 포화 문제를 줄이고 계산이 단순해 학습 속도를 높였습니다.</p>
<h3 id="dropout">Dropout</h3>
<p>Fully Connected Layer의 과적합을 완화했습니다.</p>
<h3 id="gpu">GPU</h3>
<p>대규모 Convolution과 많은 이미지를 현실적인 시간에 학습하게 했습니다.</p>
<h3 id="data-augmentation">Data Augmentation</h3>
<p>Crop, Flip, Color 변화 등으로 입력 다양성을 늘렸습니다.</p>
<p>AlexNet의 중요성은 개별 기술 하나보다 GPU·ReLU·규제·데이터를 결합해 대형 CNN 학습이 실제로 가능하다는 것을 보여준 데 있습니다.</p>
<h2 id="3-왜-단순히-깊게-쌓으면-안-될까">3. 왜 단순히 깊게 쌓으면 안 될까?</h2>
<p>깊은 Plain Network는 다음 문제를 겪습니다.</p>
<ul>
<li>Gradient 흐름 저하</li>
<li>최적화 난도 증가</li>
<li>유용하지 않은 Layer도 입력을 계속 변환</li>
<li>더 깊은 모델의 Training Error가 오히려 높아지는 Degradation</li>
</ul>
<p>Batch Normalization과 ReLU만으로 모든 문제를 해결할 수 없었습니다.</p>
<h2 id="4-resnet의-핵심-발상">4. ResNet의 핵심 발상</h2>
<p>일반 Block은 목표 함수 $H(x)$ 전체를 학습합니다.</p>
<p>ResNet은 변화량을 학습합니다.</p>
<p>$$
F(x)=H(x)-x
$$</p>
<p>따라서 출력은 다음과 같습니다.</p>
<p>$$
H(x)=F(x)+x
$$</p>
<p>입력 $x$가 Skip Connection을 통해 그대로 전달되고 Layer는 필요한 변화 $F(x)$만 학습합니다.</p>
<h2 id="5-residual의-두-의미-구분">5. Residual의 두 의미 구분</h2>
<h3 id="통계-회귀의-residual">통계 회귀의 Residual</h3>
<p>$$
Residual=y-\hat{y}
$$</p>
<p>실제값과 예측값의 차이로 줄여야 할 결과 오차입니다.</p>
<h3 id="resnet의-residual">ResNet의 Residual</h3>
<p>$$
F(x)=H(x)-x
$$</p>
<p>Block이 입력에 더해야 할 변화량입니다.</p>
<p>둘은 이름은 같지만 역할이 다릅니다.</p>
<h2 id="6-skip-connection">6. Skip Connection</h2>
<p>원고 편집에 비유해 봅시다.</p>
<ul>
<li>Plain Block: 매 단계에서 원고 전체를 백지에 다시 작성</li>
<li>Residual Block: 원본을 보존하고 수정할 부분만 표시</li>
</ul>
<p>고칠 것이 없다면 $F(x)\approx0$을 학습해 입력을 거의 그대로 통과시킬 수 있습니다.</p>
<p>효과:</p>
<ul>
<li>정보와 Gradient가 이동할 지름길</li>
<li>불필요한 변환을 하지 않는 선택지</li>
<li>매우 깊은 Network의 최적화 안정화</li>
</ul>
<h2 id="7-resnet의-의미와-주의점">7. ResNet의 의미와 주의점</h2>
<p>ResNet-18, 34, 50, 101, 152 등 다양한 깊이가 있습니다. Bottleneck Block을 사용하면 깊이를 늘리면서 계산량을 관리할 수 있습니다.</p>
<blockquote>
<p>ResNet이 과적합을 자동 해결하거나 Gradient Vanishing을 완전히 없애는 것은 아닙니다. 핵심은 Skip Connection이 깊은 Network의 Optimization과 Gradient 흐름을 크게 개선한다는 점입니다.</p>
</blockquote>
<p>Residual Connection은 Vision뿐 아니라 Transformer와 현대 Deep Network의 표준 요소가 됐습니다.</p>
<h2 id="part-3-핵심-정리">Part 3 핵심 정리</h2>
<ul>
<li>AlexNet은 GPU, ReLU, Dropout, Augmentation의 결합으로 대형 CNN의 성공을 보여줬습니다.</li>
<li>Network는 무조건 깊게 쌓는다고 잘 학습되지 않습니다.</li>
<li>ResNet은 전체 출력 대신 입력에 더할 변화량을 학습합니다.</li>
<li>Skip Connection은 정보와 Gradient가 흐르는 지름길입니다.</li>
</ul>
<hr />
<h1 id="part-4-attention-등장">Part 4. Attention 등장</h1>
<h2 id="1-rnn과-seq2seq의-병목">1. RNN과 Seq2Seq의 병목</h2>
<p>RNN과 LSTM은 Sequence를 순서대로 처리합니다.</p>
<p>문제:</p>
<ul>
<li>앞 계산이 끝나야 다음 계산 가능</li>
<li>긴 문장에서 먼 정보가 약해짐</li>
<li>초기 Seq2Seq는 마지막 Hidden State 하나에 전체 문장 압축</li>
</ul>
<p>Attention의 발상은 간단합니다.</p>
<blockquote>
<p>모든 과거를 하나로 억지로 압축하지 말고, 현재 필요한 부분을 직접 찾아보자.</p>
</blockquote>
<h2 id="2-attention">2. Attention</h2>
<p>Attention은 입력 요소 가운데 현재 작업에 중요한 부분에 높은 가중치를 줍니다.</p>
<p>예문:</p>
<pre><code class="language-text">고양이가 창문에서 햇살을 받으며 잔다.</code></pre>
<p>잔다를 해석할 때 고양이가와의 관계를 강하게 반영할 수 있습니다.</p>
<p>형광펜 비유:</p>
<ul>
<li>중요한 단어: 진하게 표시</li>
<li>관련이 적은 단어: 약하게 표시</li>
<li>가중합: 표시 강도에 따라 정보 조합</li>
</ul>
<h2 id="3-self-attention">3. Self-Attention</h2>
<p>같은 Sequence 안의 각 Token이 다른 모든 Token과 관계를 계산합니다.</p>
<ul>
<li>문법 관계</li>
<li>의미 관계</li>
<li>지시 관계</li>
<li>장거리 의존성</li>
</ul>
<p>RNN처럼 한 Token씩 순차 처리하지 않아 Training에서 병렬 계산하기 쉽습니다.</p>
<h2 id="4-query-key-value">4. Query, Key, Value</h2>
<p>도서관 검색에 비유하면:</p>
<ul>
<li><strong>Query</strong>: 내가 찾는 질문</li>
<li><strong>Key</strong>: 각 책의 검색용 색인</li>
<li><strong>Value</strong>: 책이 실제로 담은 내용</li>
</ul>
<p>현재 Token 표현 $X$에서 세 선형 변환을 만듭니다.</p>
<p>$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$</p>
<p>Query와 Key를 비교해 관련 점수를 만들고, 점수로 Value를 가중합합니다.</p>
<h2 id="5-scaled-dot-product-attention">5. Scaled Dot-Product Attention</h2>
<p>$$
Attention(Q,K,V)
=
softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$</p>
<h3 id="dot-product">Dot Product</h3>
<p>두 Vector가 비슷한 방향이면 큰 값이 됩니다. Query와 Key의 관련 점수로 사용합니다.</p>
<h3 id="scale">Scale</h3>
<p>차원 $d_k$가 커지면 Dot Product 값도 커져 Softmax가 지나치게 뾰족해지고 Gradient가 작아질 수 있습니다. $\sqrt{d_k}$로 나눠 안정화합니다.</p>
<h3 id="softmax-1">Softmax</h3>
<p>한 Query를 기준으로 모든 Key 점수를 합이 1인 가중치로 바꿉니다.</p>
<h3 id="value-합산">Value 합산</h3>
<p>관련도가 높은 Token의 정보를 더 많이 반영합니다.</p>
<h2 id="6-multi-head-attention">6. Multi-Head Attention</h2>
<p>한 가지 관계만 보는 대신 여러 Attention Head가 다른 표현 공간에서 동시에 관계를 찾습니다.</p>
<pre><code class="language-text">Head 1: 주어와 동사
Head 2: 장소와 사건
Head 3: 대명사와 지시 대상
Head 4: 문장 경계와 Topic</code></pre>
<p>$$
head_i=Attention(QW_i^Q,KW_i^K,VW_i^V)
$$</p>
<p>$$
MultiHead=Concat(head_1,\dots,head_h)W^O
$$</p>
<p>Head가 항상 사람이 이해하는 문법 역할 하나와 정확히 대응하는 것은 아닙니다. 데이터가 유용한 관계 표현을 학습합니다.</p>
<h2 id="7-masked-self-attention">7. Masked Self-Attention</h2>
<p>텍스트를 왼쪽에서 오른쪽으로 생성할 때 미래 정답을 보면 안 됩니다.</p>
<pre><code class="language-text">현재 위치 t
→ 1부터 t까지 참고 가능
→ t+1 이후는 Mask</code></pre>
<p>GPT 계열 Decoder의 Causal Mask가 대표적입니다.</p>
<h2 id="8-transformer-encoder">8. Transformer Encoder</h2>
<p>입력 문장을 이해하고 문맥 표현을 만듭니다.</p>
<p>대표 구성:</p>
<ol>
<li>Token Embedding</li>
<li>Positional Information</li>
<li>Multi-head Self-Attention</li>
<li>Residual Connection + Normalization</li>
<li>Feed-forward Network</li>
<li>Residual Connection + Normalization</li>
</ol>
<p>각 Token마다 문맥이 반영된 Vector가 나옵니다. 하나의 고정 Context Vector만 만드는 초기 Seq2Seq와 다릅니다.</p>
<h2 id="9-transformer-decoder">9. Transformer Decoder</h2>
<p>출력 문장을 한 Token씩 생성합니다.</p>
<p>대표 구성:</p>
<ol>
<li>Masked Self-Attention</li>
<li>Encoder-Decoder Cross-Attention</li>
<li>Feed-forward Network</li>
<li>Residual Connection과 Normalization</li>
</ol>
<p>Cross-Attention에서는 Decoder의 Query가 Encoder가 만든 Key와 Value를 참고합니다.</p>
<h2 id="10-positional-encoding">10. Positional Encoding</h2>
<p>Self-Attention만으로는 Token 순서를 자동으로 알 수 없습니다. 모든 Token을 동시에 처리하므로 위치 정보를 추가해야 합니다.</p>
<pre><code class="language-text">고양이 + 위치 1
창문   + 위치 2
앉았다 + 위치 3</code></pre>
<p>고정 Sinusoidal Encoding이나 학습 가능한 Position Embedding을 사용할 수 있습니다.</p>
<h2 id="11-bert와-gpt의-구조-차이">11. BERT와 GPT의 구조 차이</h2>
<h3 id="bert">BERT</h3>
<ul>
<li>Transformer Encoder 기반</li>
<li>앞뒤 문맥을 모두 참고하는 양방향 표현</li>
<li>문장 이해, 분류, 검색, 개체명 인식 등에 강점</li>
</ul>
<h3 id="gpt">GPT</h3>
<ul>
<li>Transformer Decoder 계열 기반</li>
<li>Causal Mask로 이전 Token만 참고</li>
<li>다음 Token 생성에 최적화</li>
</ul>
<blockquote>
<p>BERT는 Transformer Decoder를 양방향으로 확장한 모델이 아니라 <strong>Encoder-only 구조</strong>입니다.</p>
</blockquote>
<h2 id="12-bert-사전학습">12. BERT 사전학습</h2>
<p>BERT는 대규모 Text로 일반 언어 표현을 먼저 학습합니다.</p>
<h3 id="masked-language-modeling">Masked Language Modeling</h3>
<p>일부 Token을 가리고 주변 문맥으로 원래 Token을 예측합니다.</p>
<pre><code class="language-text">그는 [MASK]을 열고 우유를 꺼냈다.
→ 냉장고

그는 [MASK]을 열고 여권을 꺼냈다.
→ 서랍 또는 가방</code></pre>
<p>오른쪽 문맥까지 사용하면서 입력에 정답 Token을 그대로 보여주는 부정행위를 막습니다.</p>
<h3 id="next-sentence-prediction">Next Sentence Prediction</h3>
<p>원래 BERT는 두 번째 문장이 첫 번째 문장의 실제 다음 문장인지 예측하는 NSP도 사용했습니다.</p>
<p>후속 BERT 계열 중에는 NSP를 제거하거나 다른 문장 관계 Objective로 바꾼 모델도 많습니다.</p>
<h2 id="13-bert-입력과-출력">13. BERT 입력과 출력</h2>
<p>입력 Embedding은 보통 다음의 합입니다.</p>
<p>$$
E=TokenEmbedding+SegmentEmbedding+PositionEmbedding
$$</p>
<h3 id="특수-token">특수 Token</h3>
<ul>
<li>[CLS]: Sequence 앞에 두는 분류용 대표 Token</li>
<li>[SEP]: 문장 경계를 표시</li>
<li>[MASK]: MLM에서 가린 Token</li>
</ul>
<h3 id="contextual-embedding">Contextual Embedding</h3>
<p>같은 단어라도 주변 문맥에 따라 다른 Vector가 됩니다.</p>
<pre><code class="language-text">은행에서 돈을 찾았다.
강둑의 은행처럼 보이는 지형...</code></pre>
<p>문맥에 따라 의미 표현이 달라집니다.</p>
<h2 id="14-pre-training-fine-tuning-transfer-learning">14. Pre-training, Fine-tuning, Transfer Learning</h2>
<h3 id="pre-training">Pre-training</h3>
<p>대규모 Text로 일반적인 언어 규칙과 의미를 학습합니다.</p>
<h3 id="fine-tuning">Fine-tuning</h3>
<p>사전학습 Model에 작은 Task-specific Head를 연결하고 Label Data로 추가 학습합니다.</p>
<h3 id="transfer-learning">Transfer Learning</h3>
<p>사전학습에서 얻은 지식을 새로운 Task에 재사용하는 전체 개념입니다. Fine-tuning은 그 방법 중 하나입니다.</p>
<p>실무 예:</p>
<ul>
<li>고객 문의 분류</li>
<li>의료 Report 해부학 부위 분류</li>
<li>감성 분석</li>
<li>질문 답변</li>
<li>문서 검색</li>
</ul>
<p>교재의 방사선 보고서 사례는 적은 Label과 비정형 전문 Text에서도 BERT 기반 Transfer Learning이 유용할 수 있음을 보여주는 예입니다. 실제 의료 적용에는 외부 검증, 편향·안전성·개인정보 보호가 추가로 필요합니다.</p>
<h2 id="part-4-핵심-정리">Part 4 핵심 정리</h2>
<ul>
<li>Attention은 현재 작업에 중요한 입력 부분을 가중합합니다.</li>
<li>Q는 질문, K는 단서, V는 전달할 정보입니다.</li>
<li>Multi-head는 여러 표현 공간에서 관계를 병렬 탐색합니다.</li>
<li>Transformer는 Self-Attention으로 Training 병렬화를 가능하게 했습니다.</li>
<li>BERT는 Encoder 기반 이해 Model, GPT는 Decoder 계열 생성 Model입니다.</li>
<li>BERT는 MLM을 통해 양방향 문맥을 학습하고 Fine-tuning으로 Task에 적용합니다.</li>
</ul>
<hr />
<h1 id="part-5-규모와-비용의-시대">Part 5. 규모와 비용의 시대</h1>
<h2 id="1-transformer가-남긴-문제">1. Transformer가 남긴 문제</h2>
<p>Transformer는 정확하고 강력하지만 모든 Token 쌍을 비교하는 Self-Attention 때문에 긴 Context에서 비용이 큽니다.</p>
<h2 id="2-self-attention의-on2">2. Self-Attention의 $O(n^2)$</h2>
<p>Token이 $n$개면 Attention Score Matrix는 대략 $n\times n$입니다.</p>
<pre><code class="language-text">10 Token   → 100 관계
100 Token  → 10,000 관계
1,000 Token → 1,000,000 관계</code></pre>
<p>Training이나 Prompt 전체를 한 번 처리하는 Prefill 단계에서 계산량과 Attention Matrix Memory가 크게 증가합니다.</p>
<h2 id="3-autoregressive-decoding과-kv-cache">3. Autoregressive Decoding과 KV Cache</h2>
<p>생성할 때 이전 Token의 Key와 Value를 Cache에 저장하면 매 Step마다 과거를 다시 계산할 필요는 없습니다.</p>
<ul>
<li>KV Cache Memory: Sequence 길이에 대체로 선형 증가</li>
<li>새 Token 1개의 Attention 조회: 현재 길이에 비례</li>
<li>전체 길이 $n$을 순차 생성하는 누적 Attention 비용: 대략 $O(n^2)$</li>
</ul>
<p>긴 Context의 실시간 Agent에서는 KV Cache가 큰 Memory 병목이 됩니다.</p>
<h2 id="4-transformer와-rnn의-trade-off">4. Transformer와 RNN의 Trade-off</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>Transformer</th>
<th>전통 RNN</th>
</tr>
</thead>
<tbody><tr>
<td>과거 보존</td>
<td>K·V 형태로 자세히 보존</td>
<td>고정 크기 State로 압축</td>
</tr>
<tr>
<td>Recall</td>
<td>원본 정보 조회에 유리</td>
<td>압축 중 정보 손실</td>
</tr>
<tr>
<td>Memory</td>
<td>길이에 따라 증가</td>
<td>고정 크기</td>
</tr>
<tr>
<td>Token당 생성 조회</td>
<td>길이에 비례</td>
<td>고정 State</td>
</tr>
<tr>
<td>Training 병렬성</td>
<td>높음</td>
<td>순차 의존</td>
</tr>
</tbody></table>
<p>Transformer의 비용은 모든 과거를 자세히 참고하려는 설계의 대가이고, RNN의 정보 손실은 과거를 고정 State로 압축하는 대가입니다.</p>
<p>목표는 과거 RNN으로 단순 회귀하는 것이 아니라 RNN의 효율과 Attention의 선택적 Recall을 함께 얻는 것입니다.</p>
<h2 id="5-beyond-transformer의-두-방향">5. Beyond Transformer의 두 방향</h2>
<h3 id="efficiency-oriented">Efficiency-oriented</h3>
<p>Attention 자체를 대체하거나 선형 시간으로 근사합니다.</p>
<ul>
<li>S4와 State Space Models</li>
<li>Long Convolution 계열</li>
<li>RetNet</li>
<li>Mamba</li>
<li>DeltaNet</li>
<li>Gated Delta 계열</li>
<li>Hybrid Linear Attention</li>
</ul>
<h3 id="scaling-oriented">Scaling-oriented</h3>
<p>Parameter는 크게 늘리지만 한 Token이 사용하는 연산은 일부로 제한합니다.</p>
<ul>
<li>Sparsely Gated MoE</li>
<li>GShard</li>
<li>Switch Transformer</li>
<li>GLaM</li>
<li>여러 Open MoE LLM</li>
</ul>
<h2 id="6-state-space-model">6. State Space Model</h2>
<p>SSM은 고정 크기 State로 Sequence 정보를 전달한다는 점에서 RNN과 닮았지만, 선형 System 구조를 이용해 Training 계산을 병렬화할 수 있도록 설계됩니다.</p>
<p>개념적 형태:</p>
<p>$$
h_t=Ah_{t-1}+Bx_t
$$</p>
<p>$$
y_t=Ch_t+Dx_t
$$</p>
<p>순차 Recurrence와 전체 Sequence를 한꺼번에 계산하는 Convolution 관점 사이를 변환할 수 있습니다.</p>
<blockquote>
<p>SSM이 시간 의존성을 없애는 것은 아닙니다. 수학적 구조를 이용해 Training을 병렬화할 수 있게 만든다고 이해해야 합니다.</p>
</blockquote>
<h2 id="7-chunk-wise-처리">7. Chunk-wise 처리</h2>
<p>긴 Sequence를 여러 Chunk로 나눕니다.</p>
<pre><code class="language-text">Chunk A 내부 계산
Chunk B 내부 계산
Chunk C 내부 계산
→ 가능한 부분 병렬화
→ Chunk 사이 State 연결</code></pre>
<p>GPU 병렬성을 활용하면서 Sequence 흐름을 유지하려는 전략입니다.</p>
<h2 id="8-deltanet">8. DeltaNet</h2>
<p>Delta Rule은 State 전체를 매번 새로 쓰지 않고 새 입력이 만든 변화분만 반영합니다.</p>
<p>개념적으로:</p>
<p>$$
h_t=h_{t-1}+\Delta_t
$$</p>
<p>실제 구조에서는 기존 Memory가 예측한 내용과 새 Value의 차이를 이용해 연관된 방향을 수정합니다.</p>
<p>회의록 비유:</p>
<ul>
<li>매 발언마다 전체 회의록을 다시 쓰지 않음</li>
<li>기존 회의록 유지</li>
<li>새 발언이 바꾼 부분만 수정</li>
</ul>
<p>목표:</p>
<ul>
<li>선형 시간</li>
<li>고정 크기 또는 효율적인 State</li>
<li>GPU 친화적 학습</li>
<li>Attention과 유사한 선택적 정보 반영</li>
</ul>
<h2 id="9-linear-recurrent-model의-한계">9. Linear Recurrent Model의 한계</h2>
<p>효율이 좋아도 고정 State에 정보를 압축하므로 정확한 과거 Recall에서 Full Attention보다 불리할 수 있습니다.</p>
<p>따라서 평가할 요소:</p>
<ul>
<li>긴 Context Retrieval</li>
<li>실제 Hardware 처리량</li>
<li>Training 안정성</li>
<li>품질과 Memory Trade-off</li>
<li>Inference Latency</li>
</ul>
<p>이론적 $O(n)$만으로 실제 제품이 항상 빠르다고 단정할 수 없습니다.</p>
<h2 id="10-mixture-of-experts">10. Mixture of Experts</h2>
<p>MoE는 여러 Expert 중 Token마다 일부 Expert만 선택해 계산합니다.</p>
<pre><code class="language-text">Token
→ Router
→ 선택된 Expert 1~2개
→ 결과 결합</code></pre>
<p>모든 Expert를 매번 사용하지 않으므로 전체 Parameter 수는 크게 늘리면서 Active Computation은 제한할 수 있습니다.</p>
<h2 id="11-sparse-gating과-switch-transformer">11. Sparse Gating과 Switch Transformer</h2>
<h3 id="sparsely-gated-moe">Sparsely Gated MoE</h3>
<ul>
<li>Router가 Top-k Expert 선택</li>
<li>Expert가 서로 다른 Pattern 학습</li>
<li>높은 표현력</li>
<li>Expert 간 통신과 Load Balancing이 어려움</li>
</ul>
<h3 id="switch-transformer">Switch Transformer</h3>
<ul>
<li>Token마다 Top-1 Expert 선택</li>
<li>Routing 단순화</li>
<li>계산·통신 효율 개선</li>
<li>잘못된 Routing과 Expert 쏠림 위험</li>
</ul>
<h2 id="12-moe의-핵심-문제">12. MoE의 핵심 문제</h2>
<h3 id="load-imbalance">Load Imbalance</h3>
<p>인기 Expert에 Token이 몰리면 일부 장치만 과부하됩니다.</p>
<h3 id="capacity">Capacity</h3>
<p>Expert가 처리 가능한 Token 수를 넘으면 Token Drop이나 우회 Routing이 필요합니다.</p>
<h3 id="communication">Communication</h3>
<p>분산 장치 사이로 Token을 보내는 All-to-All 통신이 병목이 될 수 있습니다.</p>
<h3 id="router-품질">Router 품질</h3>
<p>적절한 Expert를 선택하지 못하면 전체 품질이 떨어집니다.</p>
<p>MoE는 Parameter가 많다고 항상 Dense Model보다 우수한 것이 아니라 Routing과 시스템 설계가 핵심입니다.</p>
<h2 id="13-mla-개념">13. MLA 개념</h2>
<p>Multi-head Latent Attention은 각 Token의 Key·Value Cache를 더 작은 Latent 표현으로 압축해 Memory 부담을 줄이려는 접근입니다.</p>
<pre><code class="language-text">기존: Token마다 큰 K·V 저장
MLA: 작은 Latent 표현 저장
→ 필요할 때 Attention 계산에 활용</code></pre>
<p>Trade-off:</p>
<ul>
<li>KV Cache Memory 감소</li>
<li>압축·복원 또는 Projection 계산 필요</li>
<li>압축으로 인한 정보 손실 관리</li>
</ul>
<h2 id="14-kimi-linear-사례">14. Kimi Linear 사례</h2>
<p>교재는 Kimi Linear를 다음 요소를 결합한 하이브리드 사례로 설명합니다.</p>
<ul>
<li><strong>KDA</strong>: Delta Update와 Gating으로 효율적인 State 갱신</li>
<li><strong>MLA</strong>: Full Context 관계와 KV Cache 효율 보강</li>
<li><strong>MoE</strong>: 전체 Parameter를 확장하되 일부 Expert만 활성화</li>
</ul>
<p>교재의 설명에서는 KDA Block을 주로 사용하고 일정 간격으로 MLA Block을 섞어 속도·Memory·긴 문맥 품질의 균형을 노립니다.</p>
<pre><code class="language-text">KDA → KDA → KDA → MLA → 반복</code></pre>
<h3 id="역할-분담">역할 분담</h3>
<ul>
<li>KDA: 효율 담당</li>
<li>MLA: 장거리 문맥 보강</li>
<li>MoE: Parameter 확장</li>
</ul>
<h3 id="왜-hybrid인가">왜 Hybrid인가?</h3>
<ul>
<li>순수 Full Attention: Recall은 좋지만 비용 큼</li>
<li>순수 Linear State: 효율은 좋지만 정확한 Recall 손실 가능</li>
<li>Hybrid: 대부분은 저비용 처리, 필요한 Layer에서 Full Context 보강</li>
</ul>
<blockquote>
<p>Kimi Linear 관련 성능·우위 평가는 교재가 소개한 특정 시점의 연구·기사 주장입니다. 최신 Benchmark, 공개 Model 조건, Hardware, Context 길이에 따라 결과가 달라질 수 있으므로 고정된 사실이나 절대적 우위로 해석하면 안 됩니다.</p>
</blockquote>
<h2 id="15-규모에서-효율로">15. 규모에서 효율로</h2>
<p>현대 Architecture 경쟁은 Parameter 수만 늘리는 문제가 아닙니다.</p>
<ul>
<li>Token당 Active Parameter</li>
<li>KV Cache 크기</li>
<li>Memory Bandwidth</li>
<li>Training·Inference Throughput</li>
<li>긴 Context 품질</li>
<li>Energy와 운영 비용</li>
</ul>
<p>AI Agent가 긴 작업 기록과 여러 도구 결과를 처리하려면 품질뿐 아니라 지연 시간과 Memory도 중요합니다. 그래서 Dense Transformer 하나만 키우는 방향과 함께 MoE·SSM·Linear Attention·Hybrid 구조가 연구됩니다.</p>
<h2 id="part-5-핵심-정리">Part 5 핵심 정리</h2>
<ul>
<li>Self-Attention은 Token 쌍 관계 때문에 긴 Context에서 $O(n^2)$ 비용이 큽니다.</li>
<li>KV Cache는 생성 속도를 돕지만 Sequence 길이에 따라 Memory가 증가합니다.</li>
<li>Linear Recurrent와 SSM은 선형 비용과 병렬 학습을 목표로 합니다.</li>
<li>DeltaNet은 State에 변화분을 반영합니다.</li>
<li>MoE는 일부 Expert만 활성화해 Parameter와 계산량을 분리합니다.</li>
<li>Hybrid Architecture는 Attention의 Recall과 Linear Model의 효율을 결합하려 합니다.</li>
</ul>
<hr />
<h1 id="전체-아키텍처-비교">전체 아키텍처 비교</h1>
<table>
<thead>
<tr>
<th>Architecture</th>
<th>데이터 구조에 넣은 가정</th>
<th>강점</th>
<th>대표 한계</th>
</tr>
</thead>
<tbody><tr>
<td>MLP</td>
<td>Feature가 Vector로 주어짐</td>
<td>단순, Tabular Baseline</td>
<td>공간·순서 구조 미반영</td>
</tr>
<tr>
<td>CNN</td>
<td>가까운 Pixel과 위치 공유</td>
<td>Image Feature, Parameter 효율</td>
<td>Global 관계에 Layer 필요</td>
</tr>
<tr>
<td>RNN</td>
<td>현재는 이전 State에 의존</td>
<td>가변 길이, 고정 State</td>
<td>순차 처리, 장기 기억 손실</td>
</tr>
<tr>
<td>LSTM</td>
<td>기억을 Gate로 선택</td>
<td>RNN보다 긴 의존성</td>
<td>계산량, 병렬화 한계</td>
</tr>
<tr>
<td>Autoencoder</td>
<td>입력 복원으로 표현 학습</td>
<td>Label 없이 압축·Feature</td>
<td>유용한 표현 보장 안 됨</td>
</tr>
<tr>
<td>Seq2Seq</td>
<td>입력·출력이 Sequence</td>
<td>번역·요약</td>
<td>고정 Context 병목</td>
</tr>
<tr>
<td>ResNet</td>
<td>변화량만 학습</td>
<td>깊은 Network 안정화</td>
<td>계산량과 과적합은 별도 관리</td>
</tr>
<tr>
<td>Transformer</td>
<td>모든 Token 관계 직접 계산</td>
<td>병렬 학습, 긴 관계</td>
<td>$O(n^2)$, KV Cache</td>
</tr>
<tr>
<td>BERT</td>
<td>양방향 Encoder 표현</td>
<td>문장 이해와 전이학습</td>
<td>생성용 구조 아님</td>
</tr>
<tr>
<td>Linear Recurrent/SSM</td>
<td>과거를 효율적 State로 요약</td>
<td>선형 비용, 긴 Sequence</td>
<td>정확한 Recall 손실 가능</td>
</tr>
<tr>
<td>MoE</td>
<td>Token마다 필요한 Expert만 사용</td>
<td>큰 Parameter, 제한된 연산</td>
<td>Routing·통신·부하 균형</td>
</tr>
</tbody></table>
<hr />
<h1 id="실무에서-architecture를-선택하는-질문">실무에서 Architecture를 선택하는 질문</h1>
<h2 id="1-데이터-구조는-무엇인가">1. 데이터 구조는 무엇인가?</h2>
<ul>
<li>Tabular: MLP, Tree, TabNet 등</li>
<li>Image: CNN, ResNet, ViT</li>
<li>Sequence: RNN, LSTM, TCN, Transformer, SSM</li>
<li>Multi-modal: Vision Encoder + Language Model, Cross-Attention</li>
</ul>
<h2 id="2-출력-task는-무엇인가">2. 출력 Task는 무엇인가?</h2>
<ul>
<li>Classification</li>
<li>Regression</li>
<li>Detection</li>
<li>Segmentation</li>
<li>Generation</li>
<li>Retrieval</li>
<li>Forecasting</li>
</ul>
<h2 id="3-무엇을-우선하는가">3. 무엇을 우선하는가?</h2>
<ul>
<li>정확도</li>
<li>설명 가능성</li>
<li>Latency</li>
<li>Memory</li>
<li>Training 비용</li>
<li>Edge Device 배포</li>
<li>긴 Context</li>
</ul>
<h2 id="4-데이터와-label은-충분한가">4. 데이터와 Label은 충분한가?</h2>
<ul>
<li>Label이 충분: Supervised</li>
<li>Label이 적음: Transfer Learning, Self-supervised</li>
<li>입력 구조 학습: Autoencoder</li>
<li>Foundation Model 활용: Fine-tuning, Adapter, Prompting</li>
</ul>
<h2 id="5-운영-환경은-무엇인가">5. 운영 환경은 무엇인가?</h2>
<ul>
<li>GPU Memory 한계</li>
<li>Batch 처리 또는 실시간 응답</li>
<li>Context 길이</li>
<li>Model Update 주기</li>
<li>개인정보와 보안</li>
<li>Failure 비용</li>
</ul>
<p>최신 Architecture라는 이유만으로 선택하지 말고 문제 구조와 운영 제약에 맞는지 검증해야 합니다.</p>
<hr />
<h1 id="최종-복습-문제-해결의-연대기">최종 복습: 문제 해결의 연대기</h1>
<ol>
<li><strong>MLP는 이미지를 펼치며 공간 구조를 잃었다.</strong></li>
<li><strong>CNN은 Locality와 Weight Sharing으로 이미지 구조를 반영했다.</strong></li>
<li><strong>RNN은 Hidden State로 순서를 기억했다.</strong></li>
<li><strong>LSTM은 Gate와 Cell State로 중요한 기억을 오래 유지했다.</strong></li>
<li><strong>Autoencoder는 입력 자체를 정답으로 압축 표현을 학습했다.</strong></li>
<li><strong>Seq2Seq는 Sequence 변환을 가능하게 했지만 고정 Context 병목이 있었다.</strong></li>
<li><strong>AlexNet은 GPU 기반 대형 CNN의 성공을 보여줬다.</strong></li>
<li><strong>ResNet은 Skip Connection으로 깊은 Network의 학습을 안정화했다.</strong></li>
<li><strong>Attention은 필요한 과거를 직접 참고하게 했다.</strong></li>
<li><strong>Transformer는 Self-Attention으로 Sequence Training을 병렬화했다.</strong></li>
<li><strong>BERT는 Encoder 기반 양방향 사전학습과 전이학습을 확산시켰다.</strong></li>
<li><strong>Linear Recurrent, SSM, MoE, Hybrid 구조는 긴 Context의 비용과 규모 문제를 풀려 한다.</strong></li>
</ol>
<hr />
<h1 id="마무리">마무리</h1>
<p>딥러닝 Architecture를 공부할 때 가장 중요한 질문은 “어떤 모델이 최신인가?”가 아닙니다.</p>
<blockquote>
<p><strong>이 구조는 데이터에 어떤 가정을 넣었고, 이전 구조의 어떤 병목을 해결했으며, 그 대가로 무엇을 잃었는가?</strong></p>
</blockquote>
<p>CNN은 공간 구조를 얻는 대신 일부 위치 정보를 줄이고, RNN은 고정 Memory를 얻는 대신 과거를 압축하며, Transformer는 정확한 관계 조회를 얻는 대신 계산과 Memory 비용을 부담합니다. MoE와 Hybrid Model도 효율을 얻는 대신 Routing과 시스템 복잡성을 감수합니다.</p>
<p>아키텍처의 발전은 완벽한 구조를 찾는 과정이 아니라 <strong>정확도·표현력·속도·Memory·비용 사이의 Trade-off를 더 잘 설계하는 과정</strong>입니다.</p>