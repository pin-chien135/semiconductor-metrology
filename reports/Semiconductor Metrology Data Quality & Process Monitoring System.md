### **Semiconductor Metrology Data Quality \& Process Monitoring System**

### Measurement System Analysis（MSA）技術報告

**專案名稱：** Wafer Metrology Data Simulation  
**分析模組：** Measurement System Analysis（MSA）  
**分析方法：** Crossed ANOVA Gauge R\&R、Interaction Pooling、Paired Operator Comparison、Residual Diagnostics  
**資料來源：** Python 模擬之 Wafer CD 量測資料  
**分析工具：** Python、Pandas、NumPy、SciPy、Statsmodels、Matplotlib



#### 1\. Executive Summary（摘要）

本專案建立一套模擬半導體晶圓關鍵尺寸（Critical Dimension, CD）量測資料，透過 Measurement System Analysis（MSA）評估量測系統的重複性（Repeatability）與再現性（Reproducibility），並辨識操作員之間是否存在系統性量測差異。

本次 MSA 採用平衡交叉實驗設計（Balanced Crossed Design），由 3 位操作員分別對相同的 10 片 wafer 進行 2 次重複量測，共取得 60 筆量測紀錄。

主要分析結果如下：

|評估項目|分析結果|
|-|-:|
|Total Gauge R\&R % Study Variation|14.14%|
|Total Gauge R\&R % Contribution|2.00%|
|Number of Distinct Categories（ndc）|9|
|Operator Effect（Crossed ANOVA）|p = 0.00381|
|Part × Operator Interaction|p = 0.21678|
|Shapiro-Wilk Normality Test|p = 0.96853|

主要發現：

1. 量測系統的 Gauge R\&R 占總變異約 2.00%，顯示本次模擬資料中，wafer 間差異是總變異的主要來源。
2. GRR % Study Variation 為 14.14%，落在常見的 10%–30% 條件式評估區間，仍須依實際量測用途與規格判斷。
3. 操作員主效應達統計顯著，進一步配對比較發現 OP03 與 OP01、OP02 之間存在顯著平均量測差異。
4. Part × Operator 交互作用未達顯著，將交互作用合併至誤差項後，GRR 評估結果沒有明顯改變。
5. 殘差圖與常態性檢定未顯示明顯違反常態性假設的證據，但仍需注意尾端觀測值及有限的樣本數。

整體而言，本次分析顯示量測系統具有一定的 wafer 區分能力，但仍存在值得改善的操作員間差異。

\---

#### 2\. Project Objective

在半導體製程中，量測數據是製程監控、異常判斷及製程改善的重要依據。

然而，量測結果除了受到實際製程差異影響，也可能受到量測設備、操作員及重複量測誤差影響。

若量測系統本身存在較大的變異，可能造成：

* 將量測誤差誤判為製程變化。
* 不同操作員取得不一致的量測結果。
* 降低製程監控與異常偵測的可靠性。
* 影響後續 SPC 與製程能力分析的判讀。

因此，本模組的目標為：

1. 評估量測系統的 Repeatability 與 Reproducibility。
2. 分析 wafer、operator 及兩者交互作用所造成的變異。
3. 量化 Gauge R\&R 占整體變異的比例。
4. 找出不同操作員之間是否存在系統性量測差異。
5. 檢查統計模型殘差，評估分析結果的合理性。
6. 提出量測系統改善方向，作為後續製程監控分析的基礎。

\---

#### 3\. Dataset \& Experimental Design

##### 3.1 資料結構

本次使用模擬 CD 量測資料，目標尺寸設定為 45 nm。

實驗設計如下：

|項目|設定|
|-|-:|
|Wafer 數量|10|
|Operator 數量|3|
|Repeats per Operator|2|
|Tool 數量|1|
|Total Records|60|
|Target CD|45 nm|

所有操作員均量測相同的 wafer，每位操作員對每片 wafer 進行兩次重複量測。

此設計屬於 Balanced Crossed Design，能夠分別估計 wafer、operator、交互作用及重複量測所造成的變異。

##### 3.2 模擬資料設定

資料模擬包含以下變異來源：

|變異來源|模擬設定|
|-|-|
|Wafer-to-Wafer Variation|SD = 0.8 nm|
|Operator OP01 Bias|0.00 nm|
|Operator OP02 Bias|+0.08 nm|
|Operator OP03 Bias|-0.06 nm|
|Repeatability Error|SD = 0.12 nm|

此設計刻意加入操作員間的偏差，使分析能夠評估操作員差異是否可被統計方法辨識。

注意：上述設定是模擬資料的生成條件，不是實際量測系統的已知規格。

##### 3.3 主要欄位

|欄位|說明|
|-|-|
|wafer\_id|晶圓識別碼|
|lot\_id|批次識別碼|
|tool\_id|量測設備識別碼|
|operator\_id|操作員識別碼|
|repeat|重複量測次數|
|recipe|量測配方|
|true\_cd\_nm|模擬資料中的真實 CD|
|measured\_cd\_nm|實際模擬量測值|
|timestamp|量測時間|

其中，true\_cd\_nm 僅用於模擬資料驗證及偏差分析，不納入 Gauge R\&R 變異分量估計。

在真實產線中，通常無法直接取得每片 wafer 的絕對真實 CD，因此不能假設真實製程一定具有此欄位。

\---

#### 4\. Methodology

##### 4.1 Crossed ANOVA Gauge R\&R

本次使用交叉變異數分析（Crossed ANOVA），將量測結果表示為：

\[Y\_{ijk}=\\mu+P\_i+O\_j+(PO)*{ij}+\\epsilon*{ijk}
]

其中：

* (Y\_{ijk})：第 i 片 wafer，由第 j 位操作員進行第 k 次量測的 CD。
* (\\mu)：整體平均值。
* (P\_i)：Wafer（Part）效應。
* (O\_j)：Operator 效應。
* ((PO)\_{ij})：Part × Operator 交互作用。
* (\\epsilon\_{ijk})：Repeatability Error。

Gauge R\&R 的組成：

\[
GRR=Repeatability+Reproducibility
]

其中：

\[
Reproducibility=Operator+Part\\times Operator
]

Repeatability 代表同一量測系統在相同條件下重複量測的隨機變異；Reproducibility 則代表不同操作員等條件造成的再現性差異。

##### 4.2 Variance Components

在平衡交叉設計下，使用 Mean Square（MS）估計各變異來源：

\[
\\sigma^2\_{Repeatability}=MS\_{Repeat}
]

\[
\\sigma^2\_{Interaction}=
\\frac{MS\_{Interaction}-MS\_{Repeat}}{n\_{repeat}}
]

\[
\\sigma^2\_{Operator}=
\\frac{MS\_{Operator}-MS\_{Interaction}}
{n\_{part}\\times n\_{repeat}}
]

\[
\\sigma^2\_{Part}=
\\frac{MS\_{Part}-MS\_{Interaction}}
{n\_{operator}\\times n\_{repeat}}
]

負的變異分量估計值在本專案的報表中以 0 截斷，但原始估計值應另外保留，以供模型診斷。

##### 4.3 GRR Evaluation Metrics

**Percentage Contribution**

\[
%Contribution=
\\frac{\\sigma^2\_{GRR}}{\\sigma^2\_{Total}}\\times100%
]

用於評估 GRR 變異占總變異的比例。

**Percentage Study Variation**

\[
%Study\\ Variation=
\\frac{\\sigma\_{GRR}}{\\sigma\_{Total}}\\times100%
]

本專案使用 6 SD 作為 Study Variation 的表示方式。

**Number of Distinct Categories**

\[ndc=
\\left\\lfloor
1.41\\sqrt{\\frac{\\sigma^2\_{Part}}
{\\sigma^2\_{GRR}}}
\\right\\rfloor
]

ndc 用於評估量測系統區分不同產品或 wafer 變異程度的能力。

##### 4.4 Interaction Pooling

當 Part × Operator 交互作用未達顯著時，可以依照傳統 Gauge R\&R 分析慣例，評估將交互作用合併至誤差項。

合併後：

\[MS\_{Pooled}=
\\frac{SS\_{Interaction}+SS\_{Repeatability}}
{DF\_{Interaction}+DF\_{Repeatability}}
]

再以合併後的 Mean Square 重新估計變異分量，並比較 Pooling 前後的 GRR 結果。

此處將 Pooling 作為模型簡化的比較，不代表不顯著的交互作用已被證明不存在。

##### 4.5 Operator Post-hoc Analysis

當 Operator 主效應達顯著時，進一步比較各操作員的平均量測值。

由於所有操作員均量測相同的 wafer，因此先將同一 wafer、同一操作員的兩次量測取平均，再以 wafer 為配對單位執行 Paired t-test。

比較組合：

* OP01 vs OP02
* OP01 vs OP03
* OP02 vs OP03

使用 Holm correction 修正多重比較造成的第一類錯誤率膨脹。

##### 4.6 Residual Diagnostics

使用以下方法檢查模型殘差：

1. Residual vs Fitted：檢查殘差是否存在趨勢或異質變異。
2. Residual Histogram：檢查殘差分布。
3. Normal Q-Q Plot：比較殘差與理論常態分布。
4. Residual by Operator：檢查不同操作員的殘差模式。
5. Shapiro-Wilk Test：輔助檢查殘差常態性。

\---

#### 5\. Results

##### 5.1 Crossed ANOVA

原始 ANOVA Mean Square：

|Source|SS|DF|MS|
|-|-:|-:|-:|
|Part|30.724879|9|3.413875|
|Operator|0.151964|2|0.075982|
|Part × Operator|0.177320|18|0.009851|
|Repeatability|0.215647|30|0.007188|

針對平衡交叉隨機效應 ANOVA，使用對應的 Mean Square 作為 F 檢定分母：

|Source|F-value|p-value|結果|
|-|-:|-:|-|
|Part|346.55|約 2.68 × 10⁻¹⁸|Significant|
|Operator|7.71|0.00381|Significant|
|Part × Operator|1.37|0.21678|Not Significant|

###### *Interpretation*

**Part Effect**

Part 效應達顯著（p < 0.05），表示不同 wafer 的 CD 量測結果存在顯著差異。

這是合理的結果，代表本次模擬資料具有 wafer-to-wafer variation。

**Operator Effect**

Operator 效應達顯著（p = 0.00381），表示不同操作員的平均量測結果存在統計差異。

因此，操作員是本次量測系統中值得進一步檢查的變異來源。

**Part × Operator Interaction**

交互作用未達顯著（p = 0.21678），目前沒有足夠證據顯示操作員對不同 wafer 的量測差異模式存在顯著交互作用。

這代表在目前樣本與實驗條件下，沒有觀察到明顯的操作員 × wafer 特定效應。

\---

##### 5.2 Gauge R\&R Variance Components

保留交互作用的第一版結果：

|Source|Variance|StdDev|% Contribution|% Study Variation|
|-|-:|-:|-:|-:|
|Repeatability|0.0072|0.0848|1.24%|11.14%|
|Operator|0.0033|0.0575|0.57%|7.56%|
|Part × Operator|0.0013|0.0365|0.23%|4.79%|
|Reproducibility|0.0046|0.0681|0.80%|8.95%|
|Total Gauge R\&R|0.0118|0.1087|2.04%|14.29%|
|Part-to-Part|0.5673|0.7532|97.96%|98.97%|
|Total Variation|0.5792|0.7610|100%|100%|

###### *Interpretation*

* Part-to-Part 變異占總變異約 97.96%。
* Gauge R\&R 占總變異約 2.04%。
* Repeatability 是 GRR 中主要變異來源。
* Reproducibility 則包含操作員與交互作用所造成的變異。

此結果顯示，在本次模擬條件下，wafer 間的 CD 差異遠大於量測系統本身的變異。

\---

##### 5.3 Interaction Pooling

由於 Part × Operator 交互作用未達顯著，因此進一步將其與 Repeatability 合併。

\[MS\_{Pooled}=
\\frac{0.177320+0.215647}{18+30}
\\approx0.008187
]

Pooling 後結果：

|Metric|Value|
|-|-:|
|Repeatability Variance|0.008187|
|Operator Variance|0.003390|
|Reproducibility Variance|0.003390|
|Total GRR Variance|0.011577|
|Part-to-Part Variance|0.567615|
|Total Variance|0.579191|
|GRR Study Variation (6SD)|0.645566|
|Total Study Variation (6SD)|4.566277|
|GRR % Contribution|1.9987%|
|GRR % Study Variation|14.1377%|
|ndc|9|

###### *Comparison*

|Metric|保留交互作用|Pooling 後|
|-|-:|-:|
|GRR % Study Variation|14.29%|14.14%|
|GRR % Contribution|2.04%|2.00%|
|ndc|9|9|

###### *Interpretation*

Pooling 前後的 GRR % Study Variation 僅相差約 0.15 個百分點，ndc 也維持 9。

因此，將交互作用合併後，並未大幅改變本次量測系統的整體評估結果。

本報告後續以 Pooling 後結果作為主要呈現值，同時保留未 Pooling 結果供模型比較。

\---

##### 5.4 Operator Post-hoc Analysis

以每片 wafer 的操作員平均量測值進行配對比較。

|Comparison|Mean Difference (nm)|t-statistic|Raw p-value|Holm p-value|Significant|
|-|-:|-:|-:|-:|-|
|OP01 − OP02|-0.028774|-1.021982|0.333473|0.333473|No|
|OP01 − OP03|+0.089422|3.061943|0.013529|0.029235|Yes|
|OP02 − OP03|+0.118196|3.265979|0.009745|0.029235|Yes|

###### *Interpretation*

**OP01 vs OP02**

兩者平均量測差異約 -0.029 nm，Holm-adjusted p-value = 0.3335，未達顯著。

目前沒有足夠證據認為 OP01 與 OP02 的平均量測值不同。

**OP01 vs OP03**

OP01 平均比 OP03 高約 0.089 nm，Holm-adjusted p-value = 0.0292，達統計顯著。

**OP02 vs OP03**

OP02 平均比 OP03 高約 0.118 nm，Holm-adjusted p-value = 0.0292，達統計顯著。

###### *Engineering Interpretation*

本次分析顯示 OP03 的平均量測值顯著低於 OP01 與 OP02。

這可能代表操作員間存在量測偏移，值得進一步檢查：

1. 量測定位方式是否一致。
2. 對焦及量測區域是否一致。
3. 操作流程與 Recipe 執行方式是否一致。
4. 是否需要重新訓練或建立標準化 SOP。

但統計差異不等於操作錯誤，也不能直接認定 OP03 的結果不正確。

若要判斷誰的量測值較接近真實 CD，仍需使用經校正的標準件或獨立參考量測系統驗證。

\---

#### 6\. Residual Diagnostics（殘差診斷）

##### 6.1 Residual vs Fitted Values

檢查模型殘差與預測值之間是否存在系統性關係。

**觀察結果：**

* 殘差大致分布於 0 附近。
* 未觀察到明顯的曲線型趨勢。
* 未呈現非常明顯的漏斗狀變異模式。
* 少數觀測值的殘差幅度較大，約落在 ±0.15 nm 附近。

**Interpretation：**

目前沒有明顯證據顯示殘差變異會隨 fitted value 系統性增加或減少。

但仍應保留對極端殘差的注意，並於後續分析確認是否為隨機量測波動。

##### 6.2 Normal Q-Q Plot

**觀察結果：**

* 大部分觀測點沿著理論常態分布參考線排列。
* 中間區域與參考線吻合程度良好。
* 分布兩端有輕微偏離。
* 最低與最高尾端觀測值偏離較明顯。

**Interpretation：**

整體殘差分布接近常態分布，但尾端仍存在些微偏離。

目前沒有看到足以直接否定常態性假設的明顯圖形證據。

##### 6.3 Shapiro-Wilk Normality Test

|Metric|Result|
|-|-:|
|W statistic|0.992237|
|p-value|0.968530|
|Significance Level|0.05|

由於 p-value > 0.05，因此未拒絕殘差符合常態分布的虛無假設。

這表示本次檢定沒有發現顯著的常態性違反證據，但不代表已經證明殘差完全服從常態分布。

##### 6.4 Diagnostic Summary

|Diagnostic|Observation|Assessment|
|-|-|-|
|Residual vs Fitted|無明顯曲線或漏斗型趨勢|未見明顯警訊|
|Normal Q-Q Plot|大部分接近參考線，尾端略有偏離|大致合理|
|Shapiro-Wilk|p = 0.96853|未拒絕常態性假設|

**重要方法限制：**

本次殘差來自包含完整 Part × Operator 交互作用的固定效果 OLS 模型。由於每個 wafer × operator 組合均有兩次量測，模型殘差主要反映同一組合內的重複量測誤差。

因此，這些圖主要用於檢查 Repeatability 殘差的分布與模式，不能單獨證明整個隨機效應 Gauge R\&R 模型的所有假設均成立。

\---

#### 7\. Overall Evaluation

##### 7.1 GRR Evaluation

Pooling 後的 GRR % Study Variation 為 14.14%。

常見的 MSA 參考判讀方式：

|GRR % Study Variation|常見參考判讀|
|-|-|
|< 10%|通常可接受|
|10%–30%|依用途、風險與應用條件評估|
|> 30%|通常需要改善|

本次結果落在 10%–30% 區間，代表量測系統有一定的變異，但不宜直接判定為完全合格或不合格。

由於目前尚未設定產品公差或量測用途的允收標準，因此無法計算或判定 %Tolerance。

##### 7.2 Number of Distinct Categories

ndc = 9。

常見參考值：

* ndc < 5：區分能力不足。
* ndc ≥ 5：具備基本區分能力。
* ndc ≥ 10：通常具有較理想的區分能力。

本次 ndc 為 9，代表量測系統具有一定的 wafer 差異辨識能力，但尚未達到常見的 10 以上參考值。

##### 7.3 Key Findings

本次 MSA 最重要的工程發現：

1. Wafer-to-wafer variation 是主要變異來源。
2. 量測系統整體 GRR 變異占比不高。
3. OP03 的平均量測結果顯著低於 OP01 與 OP02。
4. Part × Operator 交互作用沒有顯著證據。
5. Pooling 前後的 GRR 評估結果相當接近。
6. 殘差常態性檢查未發現明顯問題，但尾端仍有少數偏離點。

\---

#### 8\. Improvement Recommendations

依據本次分析結果，建議優先處理操作員差異，再進行更完整的量測系統驗證。

##### 8.1 Operator Standardization

針對 OP03 與其他操作員的量測差異：

* 檢查量測定位與對焦方式。
* 確認量測區域及取樣位置一致。
* 建立統一的量測 SOP。
* 安排操作員間的交叉確認及再訓練。

改善後可重新執行相同的 MSA 設計，比較改善前後 Operator Variance 與 GRR。

##### 8.2 Measurement System Improvement

若後續分析顯示 Repeatability 仍是主要 GRR 來源，可進一步檢查：

* 設備重複量測穩定性。
* 量測環境及設備狀態。
* Recipe 與量測參數設定。
* 量測定位及取樣的一致性。

##### 8.3 Further Validation

目前資料僅包含 10 片 wafer、3 位操作員、1 台設備及 2 次重複量測。

建議後續擴大樣本數，並納入：

* 更多 wafer 與不同 CD 區間。
* 更多操作員或不同操作班別。
* 不同設備間的量測比較。
* 不同時間點的量測穩定性。
* 具備已知參考值的標準件，以驗證 Accuracy 與 Bias。

此外，若要評估量測系統的長期穩定性，可進一步搭配時間序列與 SPC 管制圖。

\---

#### 9\. Limitations

本次結果具有以下限制：

**1. 模擬資料限制**

資料由預先設定的 wafer variation、operator bias 與 repeatability error 產生，並非實際半導體產線資料。

因此，分析成果主要用於展示統計方法與工程分析流程，不能直接推論真實設備的量測效能。

**2. 樣本數限制**

目前只有 10 片 wafer，對變異分量及操作員差異的估計仍有不確定性。

**3. 單一設備限制**

本次 MSA 只有 Tool\_A，無法評估設備間的 Reproducibility 或 Tool-to-Tool Matching。

**4. Accuracy 未納入 GRR**

雖然模擬資料具有 true\_cd\_nm，但本次 Gauge R\&R 僅使用 measured\_cd\_nm 估計變異分量。

量測系統的 Bias、Linearity 與 Accuracy 仍需獨立分析。

**5. 缺乏產品公差**

目前沒有設定實際 CD 規格上下限，因此不能評估 GRR 相對於公差的比例，也不能直接做量測系統的規格允收判定。

**6. 隨機效應假設**

本次變異分量估計採用平衡交叉隨機效應 ANOVA 的方法，假設各變異來源符合模型設定。若操作員僅代表固定的特定人員，而非更大操作員群體的抽樣，實務解釋應相應調整。

\---

#### 10\. Conclusion

本專案透過 Crossed ANOVA Gauge R\&R，建立一套評估 CD 量測系統重複性與再現性的分析流程。

在本次模擬資料中，Pooling 後 GRR % Study Variation 為 14.14%，GRR % Contribution 為 2.00%，ndc 為 9，顯示量測系統具有一定的 wafer 差異辨識能力。

然而，Operator 主效應達統計顯著，配對比較進一步指出 OP03 的平均量測值顯著低於 OP01 與 OP02，顯示操作員間可能存在系統性量測偏移。

此外，Part × Operator 交互作用未達顯著，Pooling 前後的 GRR 結果相近。殘差診斷亦未發現明顯的常態性違反證據。

本模組的主要成果不僅是計算 GRR 指標，更是從量測數據中辨識可能的變異來源，並將統計結果轉化為具體的量測流程改善建議。

此分析流程可作為後續 Tool Matching、SPC 製程監控及異常偵測模組的基礎。

\---

**Project Status：MSA Analysis — Initial Validation Completed**

**Next Module：Tool Matching \& Measurement Consistency**

後續將延伸至不同量測設備間的系統性偏差、一致性與設備匹配分析，建立更完整的 Semiconductor Metrology Data Quality \& Process Monitoring System。

