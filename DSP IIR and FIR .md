# DSP IIR and FIR

## 基本背景知識介紹: 差分方程

  
  $$
  y[n] = -\sum_{k=1}^{N_a} a_k \, y[n-k] + \sum_{k=0}^{N_b} b_k \, x[n-k]
  $$

* 在差分方程裡的 LTI 系統，因為有過去的輸出 $y[n-k]$，所以會形成「分母」
* 而這個過去的輸出 $y[n-k]$ ，其實就是 **feedback**的意思


### 在 FIR system 的定義裡，我們的系統不會有 feedback 的存在
* 沒有 feedback -> 沒有過去的輸出 $y[n-k]$ -> 頻率響應的「分母」=1

$$
y[n] = \sum_{k=0}^{N} b_k \, x[n-k]
$$

* 轉換到 $z$-domain

$$
Y(z) = \left(\sum_{k=0}^N b_k z^{-k}\right) X(z)
$$

 * 所以：
  
  $$
  H(z) = \frac{Y(z)}{X(z)} = \sum_{k=0}^N b_k z^{-k}
  $$


### 結論
* **FIR filter 的分母永遠是 1**，因為系統內沒有 feedback，也就代表它的輸出不依賴於過去的輸出項
    * **導致 pole = 0 or ∞**
* IIR filter 則會因為有 feedback ，導致分母不是常數


***

## FIR linear phase filter 

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

* zeros 為 共軛倒數對 (conjugate reciprocal pair)： $\alpha_k \quad \text{and} \quad \frac{1}{\alpha_k^*}$
* pole: $z = 0 \quad \text{or} \quad z = \infty$

* 補充說明: linear phase 的情況只在 FIR 內考慮
    * 如果在 IIR 內考慮 linear phase，就會導致系統無法 causal stable
 
***

