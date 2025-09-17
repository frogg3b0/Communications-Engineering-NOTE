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

### Memoryless
輸出只和「現在」的輸入有關

### Causal
輸出只和「現在」或「過去」的輸入有關 (因果)

### Linear: 
T{ax+b}=aT{x}+b

### Time Invariant
輸入訊號 x(t) 產生輸出 y(t)，那麼對於任意時間延遲的輸入 x(t+δ) 將得到相同時間延遲的輸出y(t+δ)

### Stable
如果輸入有界，則輸出必然有界

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

