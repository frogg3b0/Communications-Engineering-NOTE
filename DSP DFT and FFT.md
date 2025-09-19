#  DFT 

## 🔹 DTFT (Discrete-Time Fourier Transform)

- **適用對象**：無限長序列 $x[n]$  

- **定義**： $X(e^{j\omega}) = \sum_{n=-\infty}^{\infty} x[n] e^{-j\omega n}$

- **特性**：
  - $x[n]$ 在時域「離散」
  - $X(e^{j\omega})$ 在頻率域是 **連續**，週期為 $2\pi$。

👉 簡單說：**時間離散、頻率連續且週期性**。

---

## 🔹 DFT (Discrete Fourier Transform)

- **適用對象**：有限長序列 $x[n], \; n=0,1,\dots,N-1$

- **定義**： $X[k] = \sum_{n=0}^{N-1} x[n] e^{-j\frac{2\pi}{N}kn}, \quad k=0,1,\dots,N-1$

- **特性**：
  - 輸入是有限長度序列。
  - 輸出是 **離散頻率點** $X[k]$，總共 $N$ 個點。
  - 可以視為對 **DTFT 在 $N$ 個等間隔頻率點**的取樣。
