# 海馬迴 MRI 影像組學輔助內側顳葉癲癇側化

課程「人工智慧於醫學影像組學的分析與應用」學期專題（115 學年度第 1 學期）。

**研究問題**：在術前 MRI 上，雙側海馬迴的影像組學不對稱特徵，能否在「海馬迴體積不對稱＋年齡組＋性別」基準模型之外，提升藥物難治型內側顳葉癲癇（mTLE）左右側化的鑑別力？

**主要假說**：在事先切出、只開啟一次的保留測試集上，模型 C 的 AUC 比基準模型 A 高 ≥ 0.05，且模型 C 的測試 AUC 與訓練集交叉驗證 AUC 相差 < 0.05。

## 資料

- 來源：IDEAS（Imaging Database for Epilepsy And Surgery），公開於 OpenNeuro；資料連結見 cnnp-lab.com/ideas-data。
- 引用：Taylor PN, et al. *Epilepsia*. 2025;66(2):471–481. doi:10.1111/epi.18192
- **本 repo 不包含任何影像**。執行 notebook 時會從 OpenNeuro 下載所需檔案（T1、FLAIR、FreeSurfer `aseg`、`participants.tsv`）。

## 結構

```
notebooks/   Colab notebook（M0–M5，先切分 → 摺內建模 → 測試集只開啟一次）
config/      PyRadiomics 參數檔與事先寫定的分析設定
docs/        計畫書、進度試算表
results/     特徵矩陣、模型結果、圖表（執行後產生）
```

## 在 Colab 執行

1. 在 Colab 以「從 GitHub 開啟」載入 `notebooks/hippocampal_radiomics_TLE_colab_v2.ipynb`。
2. 示範模式（預設 `DEMO_MODE = True`）：驗證 PyRadiomics 萃取，並以模擬資料演練完整流程；示範數字不是研究結果。
3. 真實資料：設定 `DEMO_MODE = False`，填入 IDEAS 的 OpenNeuro 編號 `DATASET_ID`，並依 `participants.tsv` 的實際欄位修改 `COLS`。
4. 中間結果存到 Google Drive，可中斷後續跑。

## 防洩漏設計（對應課程「洩漏自我稽核五問」）

- 切分單位是病人；左右海馬迴合併為同一人的不對稱特徵。
- 切分前只做逐一受試者的運算（前處理、萃取、不對稱指標）。
- ComBat、ICC 篩選、冗餘移除、標準化、LASSO 全部在交叉驗證的摺內執行。
- 保留測試集只開啟一次（`open_test_once()` 第二次呼叫會報錯）。

## 可重現性

- 隨機種子 2026；分析設定見 `config/analysis.yaml`，建模開始後不再擴充。
- notebook 自動輸出 `versions.json`（套件版本）；第一次成功執行後，請據此鎖定 `requirements.txt` 的版本。

## 進度

見 `docs/project_plan.xlsx`。期末口頭報告 W17–W18（12/24、12/31）；實作成果 W18（12/31）繳交。
