# DSP IIR and FIR

## 基本背景知識介紹: 差分方程

  
  $$
  y[n] = -\sum_{k=1}^{N_a} a_k \, y[n-k] + \sum_{k=0}^{N_b} b_k \, x[n-k]
  $$

* 在差分方程裡的 LTI 系統，因為有過去的輸出 $y[n-k]$，所以會形成「分母」
* 而這個過去的輸出 $y[n-k]$ ，其實就是 **feedback**的意思


# FIR 與 IIR 基本整理

## 🔹 FIR (Finite Impulse Response Filter)

### 定義
- 脈衝響應長度有限（有限長度序列）。
- 系統輸出僅依賴 **有限個輸入樣本**。
- 無需回授 (no feedback)。

### 數學形式
$$
y[n] = \sum_{k=0}^{M} b_k x[n-k]
$$

- 只有輸入項，沒有輸出回授項。
- 本質上是一個 **有限長度的 convolution sum**。

### Z-transform
$$
H(z) = \sum_{k=0}^{M} b_k z^{-k}
$$

- 只有分子多項式。
- **沒有極點 (除了 z=0)**。


---

## 🔹 IIR (Infinite Impulse Response Filter)

### 定義
- 脈衝響應長度無限。
- 系統輸出依賴 **當前輸入與過去輸出**（有回授 feedback）。

### 數學形式
$$
y[n] = \sum_{k=0}^{M} b_k x[n-k] - \sum_{k=1}^{N} a_k y[n-k]
$$

- 第二項就是回授，讓響應變成無限長。

### Z-transform
$$
H(z) = \frac{\sum_{k=0}^{M} b_k z^{-k}}{1 + \sum_{k=1}^{N} a_k z^{-k}}
$$

- 分子分母都有多項式。
- 分母帶來 **極點 (poles)**，因此可能有無限長響應。

***

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

## 🔹 線性相位 (Linear Phase) 條件

### 定義
- 如果頻率響應的相位為線性函數： $\angle H(e^{j\omega}) = \alpha \omega + \beta$ ，則輸出僅為「延遲 + 常數相位旋轉」，不會產生失真。
  
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

## Filter design

### 數學上常見的例子
* FIR → 長度有限，例如在 time-domain 上的矩形脈衝 (rectangular pulse)。
* Rational IIR → 有閉合形式，例如因果指數 (causal exponential)。
* Non-rational IIR → 例如理想低通濾波器 (sinc)，無法用有限差分實現。

### 設計方法：如何從「規格需求」得到濾波器係數。
* IIR 設計方法：
    * Continuous-time IIR Filter: (Butterworth, Chebyshev, Elliptic)。
    * 利用 bilinear transform 或 impulse invariance 轉換到數位域。
* FIR 設計方法：
    * Window design (Hamming, Kaiser)。
    * 最佳化法 (Parks–McClellan 演算法)。

***

## 設計 Continuous-time IIR Filter

### 🔹 1. Butterworth Filter
- **特性**：最大平坦 (Maximally flat)  
  - 通帶內沒有波動，頻率響應盡可能平滑。  
  - 阻帶衰減比較慢。  
- **優點**：響應自然、平滑，適合對通帶要求平穩的應用（音訊）。  
- **缺點**：要達到高衰減 → 階數要高。  
<img width="228" height="152" alt="image" src="https://github.com/user-attachments/assets/69816497-5d74-4007-ac5b-7e7f69d74497" />

---

### 🔹 2. Chebyshev Filter
分成兩種：

#### Chebyshev Type I
- **特性**：通帶有等波紋 (equiripple)，阻帶單調。  
- **優點**：相較 Butterworth，在相同階數下，過渡區更窄。  
- **缺點**：通帶會有波動 (ripple)。  
<img width="318" height="173" alt="image" src="https://github.com/user-attachments/assets/52ec336f-1535-4e8b-b306-2411099ac576" />

#### Chebyshev Type II
- **特性**：通帶平坦，阻帶有等波紋。  
- **優點**：避免通帶 ripple。  
- **缺點**：設計較少用，因為阻帶 ripple 在很多應用不可接受。  
<img width="197" height="114" alt="image" src="https://github.com/user-attachments/assets/192ade97-9914-406b-8141-01f7336f2a2c" />

---

### 🔹 3. Elliptic (Cauer) Filter
- **特性**：通帶與阻帶都有等波紋 (equiripple)。  
- **優點**：在相同階數下，過渡區最窄（效率最高）。  
- **缺點**：通帶與阻帶 ripple 都存在，相位響應也不理想。

<img width="309" height="114" alt="image" src="https://github.com/user-attachments/assets/19e9981f-7fff-4720-959f-72709d9b91ad" />  

***


