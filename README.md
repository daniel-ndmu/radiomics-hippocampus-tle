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
- ComBat、ICC 篩選、冗餘移除、標準化、LASSO 全部在交叉驗證的摺內執行；模型 C 的訓練集 CV AUC 以巢狀交叉驗證估計。
- 保留測試集只開啟一次（`open_test_once()` 第二次呼叫會報錯）；開啟前以 Git tag `pre-test-lock` 標記程式與設定版本。

## 分割穩定性與特徵篩選設定

- 分割：IDEAS 公開、經人工檢查的 FreeSurfer 7.3.2 `aseg`（海馬迴 17／53；杏仁核 18／54 為探索性對照）。
- 擾動：每個海馬迴遮罩另做 1 體素侵蝕與 1 體素膨脹（SimpleITK，3D），三版本都萃取特徵；擾動強度（Dice、體積變化 %）輸出於 `perturbation_stats.csv`。
- ICC：ICC(3,1)，在**左右不對稱指標**層級計算，門檻 0.75，只用訓練摺受試者。
- 摺內順序：ComBat（原始版本估計、同參數套用到侵蝕／膨脹版本）→ ICC → 冗餘移除（Spearman |ρ| ≤ 0.90，以 `vol_ai` 為參照）→ 標準化 → LASSO（≤ 5 個）。
- 以上於 2026-11-25、開啟測試集前寫定，見 `docs/計畫書補充說明_20261125.md`。

## 輸出檔（`results/`，執行後產生）

| 檔案 | 內容 |
|---|---|
| `features_AI_v0/v1/v2.csv` | 原始版特徵矩陣（原始／侵蝕／膨脹遮罩的不對稱指標） |
| `features_AI_filtered_train.csv`、`features_AI_filtered_test.csv` | 篩選後版特徵矩陣（ICC＋冗餘移除後） |
| `perturbation_stats.csv` | 遮罩擾動強度 |
| `stability_summary.json`、`icc_train_final.csv` | ICC 與篩選摘要 |
| `nested_cv_C.csv`、`selection_frequency_C.csv` | 模型 C 巢狀 CV 結果、穩定度選擇頻率 |
| `test_results.csv`、`fig_test.png`、`selected_features_C.csv` | 保留測試集結果（只開啟一次） |
| `versions.json` | 套件版本 |

## 可重現性

- OpenNeuro 資料集編號與版本：`ds005602` v1.0.0（2024-10-28，doi:10.18112/openneuro.ds005602.v1.0.0），只含原始 T1w 與 FLAIR。
- 臨床資料表：figshare tables_metadata 的 `Metadata_Release_Anon.csv`（放在 Drive 的 `metadata/`，不進 repo）。原始表有一列完全重複（ID 209），程式去除後為 442 人；原始檔不修改。
- FreeSurfer：figshare freesurfer_zip，每人一個 `<ID>.tar.gz`，下載連結格式 `https://figshare.com/ndownloader/files/<file_id>?private_link=b13b8bb41390d3f7a088`；只取出 `mri/aseg.mgz`、`mri/orig.mgz`、`stats/aseg.stats`。
- FreeSurfer 分割、臨床與人口學資料表、切除遮罩由 IDEAS 團隊放在 figshare，連結見 OpenNeuro 的 README 與 cnnp-lab.com/ideas-data；實際使用的檔案與下載日期於第一次下載後填入。

- 隨機種子 2026；分析設定見 `config/analysis.yaml`，建模開始後不再擴充。
- notebook 自動輸出 `versions.json`（套件版本）；第一次成功執行後，請據此鎖定 `requirements.txt` 的版本。

## 進度

見 `docs/project_plan.xlsx`。期末口頭報告 W17–W18（12/24、12/31）；實作成果 W18（12/31）繳交。
