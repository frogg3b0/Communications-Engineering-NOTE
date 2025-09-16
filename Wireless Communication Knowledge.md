# Wireless Communication Knowledge

## Slow/fast fading, flat fading/frequency selective
### Delay Spread
因為 multipath ，造成「同樣資訊的訊號」抵達接收端的時間不同  
<img width="566" height="509" alt="image" src="https://github.com/user-attachments/assets/6b894091-1ede-4079-a7be-9f983f3d5a9b" />  

### Doppler Spread
因為 multipath ，造成「同樣資訊的訊號」所受到的 Doppler effect 不同，因此不同路徑對應的頻譜會有「不同程度的frequency shift」  
* 使得接收端的頻譜從單一頻率「擴展成一段頻帶」
<img width="522" height="285" alt="image" src="https://github.com/user-attachments/assets/18a2a0fa-7855-45b1-a02d-a8125a97a6ba" />

### Coherence Time $T_c$
在通訊系統中， 通訊通道可能會隨著時間的推移而變化。而 Coherence time 指的是「**通道脈衝響應不變化的持續時間**」
* 若 $T_c$ 越大，代表通道不變化的時間持續越長，此時為 Slow fading
* 若 $T_c$ 越小，代表通道不變化的時間持續越短，此時為 Fast fading
* 和 Doppler Spread 成反比

### Coherence Bandwidth $W_c$
在通訊系統中，通訊通道統計性質相似並且coherence的頻段
* 若 $W_c$ 越大，代表通道不變化的頻寬越大，此時為 flat fading
* 若 $W_c$ 越小，代表通道不變化的頻寬越小，此時為 frequency selective
* 和 Delay Spread 成反比

***

## 同樣是 fading，但成因卻大不相同的 Rayleigh/Rician fading

### Rayleight fading
主因: NLOS 的 nultipath  
* 因為同個訊號經由 multipath 之後，會在不同時間點抵達接收端，而這些 path 的振幅也大不相同
* 在接收端把這些 path 的振幅及相位相加之後，可以寫成 $h$ = $h_R$ + $jh_I$
* 當路徑數量趨近於無限大時，根據 CLT 可知 $h_R$ 和 $jh_I$ 皆為高斯分布
* 此時的  $|h| = \sqrt{h_R^2 + h_I^2}$ 就是 Rayleigh distributed


### Rician fading
主因: 有 LOS 的 nultipath
* 概念上和前面的 Rayleigh fading 非常相似，差在這邊多了 LOS 的分量導致振福的分布不相同
* Rician factor $K_r$ = $A^2$ / $2σ^2$  = LOS分量的功率/NLOS分量的功率
    * $K_r$ = 0，表示全部都是 NLOS，此時為 Rayleigh fading
    * $K_r$ -> 無限大，表示 LOS 極強，散射可忽略 → 通道接近 AWGN + 固定衰減

***

有了這些基本觀念之後，接下來就可以進入 OFDM 的領域

## OFDM
### OFDM 定義 (Orthogonal Frequency Division Multiplexing)

將系統的頻寬拆分成很多個 subcarrier ，這些 subcarrier 會互相正交，並且頻寬相對於原本的單一載波還要小很多，也就是說我們把一個 widband channel切成多個 narrowband channel
* 此時每個 subcarrier 的峰值會落在其他 subcarrier 的 null
* subcarrier 會互相正交不干擾彼此
* 因此不需要「guard band」

### 這樣做的用意是甚麼?
* 更小的頻寬代表更接近 coherence bandwidth
* 因此就可以解決 frequency-selective fading 的問題
* 因為這個原因，導致 MIMO 常常會和 OFDM 一起出現

### 為何 MIMO 需要 OFDM (需要 flat fading)
* 如果是 frequency-selective fading(代表$W_c$很小/Doppler Spread很大)會導致嚴重 ISI ，此時的 MIMO 檢測要同時處理「空間 + 時間」的干擾，幾乎不可行
* 因此我們使用 MIMO system 的時候，都會希望它建立在 flat fading 的情況下
* 這也就是 MIMO-OFDM 的由來: OFDM 解決「頻率選擇性衰落」，MIMO 專心發揮空間自由度 (diversity/spatial multiplexing)

### OFDM Tranceiver 架構
<img width="1006" height="403" alt="image" src="https://github.com/user-attachments/assets/c629b1a0-2d1a-4f4c-8667-4eda1ff2dede" />

#### IDFT: 
我們會把一個個的 data symbol(e.g., QAM. PSK) 乘在個別的 subcarrier上，這件事情其實等價於在 time domain 把 symbols 作為 IDFT 基底的係數，最後做 IFFT/IDFT 合成出時域波形  

#### Add CP:
* 為何要使用 Cyclic prefix?
    * 因為「**multipath**」，導致在時域上相鄰的 OFDM symbol 會互相干擾 -> **產生ISI**
    * 為了避免**ISI**，我們會在相鄰的 OFDM symbol 之間預留一段**guard period** -> **解決ISI**
    * 但是**guard period**卻進一步導致 subcarrier 之間的正交性被破壞 -> **產生ICI**
    * 為了改善**ICI**，我們不使用 guard period，而是使用 **Cyclic prefix** ，此時變成 Circular convolution -> **解決ICI**
* 如何使用 CP?
    * CP 的長度設計: 大於最大的 Delay spread，使得多路徑延遲「落在 CP 區間」內，確保不會出現 ISI
    * CP 如何設計: 複製符號尾端的一小段樣本，插入到符號開頭。這段複製出的區域就是 CP
* CP 隱含的缺點:
    * CP 是「冗餘訊號」，並不攜帶額外資訊，導致有效頻譜效率下降

***
## 接收端設計
## Equalizer
Equalizer 的目的就是 補償通道的影響，不管是 ZF 還是 MMSE，都需要知道通道響應 𝐻 
* 所以，沒有通道資訊 (CSI, Channel State Information)，就沒辦法正確設計 equalizer
* 因此在這之前，通常需要進行 channel estimation

### Zero Forcing Equalizer
**原理**：  
希望完全消除通道的影響，把接收訊號除以通道響應：

$$
\hat{x} = \frac{y}{H}
$$

或矩陣情況下：

$$
\hat{\mathbf{x}} = (\mathbf{H}^H \mathbf{H})^{-1} \mathbf{H}^H \mathbf{y}
$$

- 完全去除通道效應 (ISI、cross-talk)。
- 但如果通道響應很小（**deep fade**），會把雜訊放大，造成效能下降。
- 需要 **完整的通道資訊 (CSI)**

***

### MMSE Equalizer (LMMSE)
**原理**：  
在消除通道影響的同時，考慮雜訊的存在，目標是最小化 MSE：

$$
\hat{\mathbf{x}} = (\mathbf{H}^H \mathbf{H} + \sigma_n^2 \mathbf{I})^{-1} \mathbf{H}^H \mathbf{y}
$$


- 兼顧通道消除與雜訊控制 → 避免 ZF 的「雜訊放大問題」
- 在高 SNR 下趨近於 ZF，在低 SNR 下效能比 ZF 好
- 需要 **通道資訊 (CSI)** 和 **雜訊功率 \(\sigma_n^2\)**。

***


### SIC
**原理**：   
逐步偵測多個符號流，每次先偵測一個，再把已檢測出的訊號重建並扣除，然後再偵測剩下的
* MIMO 檢測 → V-BLAST 架構就是 LMMSE + SIC 的典型例子

**SIC 可以在 LMMSE/ZF 的基礎上進一步改善**  
1. 先用 LMMSE 偵測出最可靠的一個流
2. 把它重建後從接收訊號中扣除
3. 對剩下的符號再做 LMMSE 檢測
4. 重複此過程直到所有符號被檢測

***



