# Learning Longer-term Dependencies in RNNs with Auxiliary Losses

링크 [[2018trinh](https://arxiv.org/abs/1803.00144)]

한 줄로 요약:
> 보조적인(auxiliary) loss를 추가하여 긴 sequence에서도 학습이 잘(속도/메모리적으로나 성능적으로나) 
> 일어날 수 있도록 하자.

## 1. Introduction
Long term dependency를 배우기 위해 기존에는 주로 gradient descent와 BPTT를 사용하였다.<br/>
여기에는 몇 가지 문제점이 있는데,
<ol>
<li>긴 sequence에서 BPTT로 학습할 시 vanishing/exploding gradient 문제가 발생하기 쉽고,</li>
<li>Sequence 길이가 길어질수록 BPTT에서 잡아먹는 메모리가 비례하여 증가한다는 것이다.</li>
</ol>

이 문제점들을 해결하기 위해서 여러 가지 방법들이 제안되었는데, 문제점 1을 해결하기 위해서는..
<ol>
<li>Long Short-Term Memory(LSTM) 또는</li>
<li>Gradient clipping 등을 사용할 수 있고,</li>
</ol>

문제점 2(메모리 문제!)를 해결하기 위해서는..
<ol start="3">
<li>Truncated BPTT나 synthetic gradients(사실 이건 잘 모르겠다)처럼 sequence의 전체 말고 일부만 사용하는 방식을
채택할 수 있다.</li>
</ol>

사실 CNN도 문제를 완화시킬 수는 있지만 애초에 근본적으로 다른 문제를 해결하기 위해 만들어진 architecture이므로
CNN으로 대체하자는 것은 어불성설이다!

그래서 이 논문에서 제안하는 방식은 원래 목표로 하는 (main) supervised loss에 *unsupervised auxiliary loss*를 추가하는 것이다. <br>
결과부터 얘기하자면, 이를 통해 LSTM의 optimization과 generalization 성능이 향상되었다고 한다. <br>
또한 이 방식을 쓰면 굳이 긴 BPTT를 사용할 필요도 없는데, 이는 truncated BPTT만 써도 성능이 충분히 잘 나오기 때문이다. <br>
심지어 아주아주 긴 sequence에서 실험을 할 때 (제안한 방법을 사용하지 않은 것보다) 메모리도 적게 쓰고 training도 훨씬 빨랐다. <br>

## 2. Related work
제시된 문제를 해결하기 위해 기존에 제안되었던 방법들은
<ul>
<li>RNN with special structures</li>
<li>Skip information at certain steps</li>
</ul>
등이 있는데, 본 논문에서 제안하는 방식은 이러한 방식들과 독립이라서(orthogonal) 기존 것과 병행해서 사용될 수 있다.

## 3. Methodology
간단하게 말하면, 랜덤하게 1개 이상의 anchor position을 정해서 auxiliary loss를 추가하는 것이다.

### 3.1. Reconstruction auxiliary loss
랜덤하게 선택된 anchor point(s)에서 __이전 subsequence를 reconstruct하게끔__ loss를 추가하는 방법이다. <br>
Reconstruct 되어야 하는 부분이 anchor point에서 가까울 경우 비교적 짧은 BPTT만 수행하면 되고,
이 loss를 사용함으로써 anchor point에서 이전 event들에 대한 정보를 기억하게 된다고 한다. <br>
즉, pretraining의 효과가 있으므로 fine tuning을 할 때 긴 BPTT를 수행할 필요가 없다. <br>
이러한 auxiliary loss를 사용한 LSTM을 *r-LSTM*이라고 칭한다.

### 3.2. Prediction auxiliary loss
랜덤하게 선택된 anchor point(s)에서 __다음 subsequence를 예측하게끔__ loss를 추가하는 방법이다. <br>
이러한 auxiliary loss를 사용한 LSTM을 *p-LSTM*이라고 칭한다.

### 3.3. Training
r-LSTM과 p-LSTM은 아래의 두 단계를 거쳐 학습된다.
<ol>
<li> Auxiliary loss를 사용하여 unsupervised pretraining을 수행한다. </li>
<li> Main loss와 auxiliary loss의 합을 minimize함으로써 semi-supervised learning을 수행한다. </li>
</ol>

### 3.4. Sampling frequency and subsequence length
Auxiliary loss를 사용하기 위해서는 추가적으로 고려해야 하는 hyper-parameter들이 발생한다:
<ol>
<li> Anchor point(auxiliary loss)의 개수 </li>
<li> 각 subsequence의 길이 </li>
</ol>

이 논문에서는 주로 anchor point의 개수=1로 두고, 모든 subsequence가 같은 길이를 가지도록 설정하였다.
