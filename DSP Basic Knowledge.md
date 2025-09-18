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

