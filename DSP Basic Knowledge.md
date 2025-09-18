# DSP Knowledge

***

# Ch2 Signal System

## Discrete / Digital 個別的定義
### Discrete-time signal
Time discrete / Amplitude continuois

### Digital signal 
Time discrter / Amplitude discrete

***

## Properties of System

* Memoryless
輸出只和「現在」的輸入有關

* Causal
輸出只和「現在」或「過去」的輸入有關 (因果)

* Linear: 
T{ax+b}=aT{x}+b


    * 滿足「可加性」
    * 滿足「齊次性」

* Time Invariant
輸入訊號 x(t) 產生輸出 y(t)，那麼對於任意時間延遲的輸入 x(t+δ) 將得到相同時間延遲的輸出y(t+δ)


* Stable
如果輸入有界，則輸出必然有界

***

##  為什麼 LTI 系統的輸出可以寫成「輸入與脈衝響應的卷積」
<img width="820" height="390" alt="image" src="https://github.com/user-attachments/assets/9ea854c0-7eea-4392-8d3f-c0e3540651b0" />  

### 1. 把輸入展開成 δ 函數的組合
任何離散訊號都可以表示成**一堆單位脈衝的加權和**：

$$
x[n] = \sum_{k=-\infty}^{\infty} x[k] \, \delta[n-k]
$$

- 在 $n=k$ 的位置，有一個 $\delta[n-k]$
- 權重就是 $x[k]$ ，等於把輸入「拆解」成很多 δ


### 2. 系統作用在輸入上
套用系統運算 $T\{\cdot\}$：

$$
T\{x[n]\} = T{ \sum_{k=-\infty}^\infty x[k] \, \delta[n-k] \}
$$

由於系統是 **線性**，可以把加總與係數提出來：

$$
= \sum_{k=-\infty}^\infty x[k] \, T\{\delta[n-k]\}
$$


### 3. LTI 系統對 δ 的反應
對一個 LTI 系統，如果輸入是 $\delta[n-k]$，輸出就是 **脈衝響應 $h[n-k]$**：

$$
T\{\delta[n-k]\} = h[n-k]
$$

#### 為甚麼輸入是 $\delta[n-k]$，輸出就是脈衝響應 $h[n-k]$??
* 我們為了測試通道長甚麼樣子，會在輸入端輸入一個單位脈衝
* 意思是，當輸入是 δ[n]，我們就可以把輸出定義為 h[n]
* 又因為系統是 time-invariant，所以 $T \delta[n-k]$ 的輸出就是脈衝響應 $h[n-k]$


代回去：

$$
T\{x[n]\} = \sum_{k=-\infty}^\infty x[k] \, h[n-k]
$$


### 4. 卷積的定義
根據卷積的定義：

$$
y[n] = (x*h)[n] = \sum_{k=-\infty}^\infty x[k] \ h[n-k]
$$

所以，LTI 系統的輸出 **等於輸入和系統脈衝響應的卷積**。

***
## Ch3 Z-transform

### 基本觀念
* Right-side sequence : x[n]=0, for all n<N (代表只有在右側序列有值)
* Left-side sequence : x[n]=0, for all n>N 
    * 此處的 *N* 不一定是原點

### 收斂範圍
* z-transform 的收斂範圍是圓，這當中又分成圓內/圓外
* 因為不同的 x[n] 可能會有相同的 X(z)
    * 因此要 determined 某個 X(z)， 必須要知道 **x[n]** and **ROC**
 
### 收斂範圍的特性
1. 收斂範圍取決於 |z|
2. ROC 不可以包含 pole
3. "Fourier transform 存在" iff "ROC 包含單位圓"

* Right-side sequence 的 ROC : 在最外側的 pole 的外側
* Left-side sequence 的 ROC : 在最內側的 pole 的內側

* Stability LTI system: ROC 包含單位圓
* Causal LTI system: ROC 在最外側的 pole 的外側


### Zero and pole of rational system

$$
H(z) = \frac{B(z)}{A(z)} 
= c \frac{\prod_{i=1}^M (z - \bar{z}_i)}{\prod_{i=1}^N (z - z_i)}
$$

* Zero 的數量 = Pole 的數量
* 若 $N > M$，則在 $z = \infty$ 有 $N-M$ 個零點 (zeros)
* 若 $N < M$，則在 $z = \infty$ 有 $M-N$ 個極點 (poles)


***

## Ch4 Sampling 

<img width="953" height="187" alt="image" src="https://github.com/user-attachments/assets/b42bd991-ba7b-4e53-b13a-8429799aa6f9" />


### 系統總覽
- **輸入**： $x_c(t)$（連續時間訊號）。
- **C/D converter**：
  1. 採樣脈衝列 $s(t)=\sum_{n}\delta(t-nT_s)$  
  2. 得到 $x_s(t)=\sum_n x_c(nT_s)\,\delta(t-nT_s)$  
  3. 轉為離散序列 $x[n]=x_c(nT_s)$  
  4. 通過數位濾波器 $H(e^{j\omega})$


<img width="991" height="1009" alt="image" src="https://github.com/user-attachments/assets/e268ea98-aef7-47e7-8d08-fb1d9e342604" />

***

- **D/C converter**：
  1. $x[n] \;\Rightarrow\; x_s(t)=\sum_n x[n]\delta(t-nT_s)$  
  2. 通過類比濾波器 $H_v(j\Omega)$（矩形低通，高度 $T_s$），得到 $x_r(t)$
 
  <img width="959" height="734" alt="image" src="https://github.com/user-attachments/assets/d85c1731-e4a5-491e-a65b-82f4552f3218" />

***

## Ch5 LTI Analysis

### Effect of Phase Response

* Zero phase : 不影響系統 e.g., Ideal lowpass filter (frequency domain上)
* Linear phase : 在  time-domain 為 **delay**，代表輸出只是「輸入延遲 + 相位平移」，**不會造成波形失真**
* Nonlinear phase : 需透過 **group delay**理解
    * group delay : 用來衡量相位響應是否「接近線性」，如果群延遲是常數，就代表相位是線性的。

***

### 濾波器設計中的 Pole 與 Zero 配置對頻帶的影響
* Pole 會讓增益變大 → 因此會放在 passband
* Zero 會讓增益變小 → 因此會放在 stopband
    * 所以，透過配置 pole 與 zero，可以控制濾波器的幅度響應
    * 如果 transition band 很窄代表一群 pole 會聚集在 passband edge 附近；一群 zero 會聚集在 stopband edge 附近
  
<img width="490" height="171" alt="image" src="https://github.com/user-attachments/assets/e2a648fc-671f-483a-b52b-778149eb6f5c" />  

***
## FIR Linear Phase 專區

### Linear Phase -> FIR linear phase filter 的四種 type

#### 1. 若系統頻率響應為 linear phase

$$
\angle H(e^{j\omega}) = \alpha \omega + \beta
$$



#### 2. 推出時域條件
因為系統頻率響應為 linear phase，所以經過推導可得：  

$$
h[n] = e^{j\theta} h^*[N-n]
$$

* 依照這個公式，我們可以進一步為FIR linear filter分成四種 type

#### 3. Real-coefficient FIR Linear Phase Filter 的四種類型

* 因為 h[n] 是實係數，所以 $h[n] = e^{j\theta} h^*[N-n]$ 可以退化成 $h[n] = \pm h[N-n]$
* 因此 **real coefficient FIR filter *h[n]* 有 linear phase** iff **$h[n] = \pm h[N-n]$**
    * 衍伸出四種 type

<img width="783" height="797" alt="image" src="https://github.com/user-attachments/assets/e8198bae-b2a4-48b2-a6c5-69b01c9443a9" />   

***

### FIR Linear Phase Filter 的 zero point


