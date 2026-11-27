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
2. **M0-1**：自動 clone 本 repo 並安裝 `requirements-colab.txt`（PyRadiomics、neuroCombat、SimpleITK 固定版本與 nibabel）。`REPO_REF` 平時用 `"main"`；期末重現改成定版的 tag（例如 `pre-test-lock`），即可取得與報告數字相同的程式版本。斷線或重新啟動後，從這一格重跑。
3. **M0-2**：選擇模式。`DEMO_MODE = True`（預設）以 PyRadiomics 示範影像與模擬特徵演練完整流程，示範數字不是研究結果；真實資料設 `DEMO_MODE = False`，試跑再設 `PILOT_N = 3`。事先寫定的參數全部讀自 `config/analysis.yaml`，並與計畫書寫定值逐項核對，不符即停止。
4. **M0-3**：掛載 Google Drive（失敗會自動重試一次並列出處理方式），在 `ideas_radiomics/` 做寫入測試，環境紀錄寫入 `versions.json`（套件版本、repo commit、執行時間、與 `requirements.txt` 不同的套件）。
5. 真實資料：把 figshare tables_metadata 的三個檔放到 Drive 的 `ideas_radiomics/metadata/`，再依 notebook 末尾的執行順序與檢查清單進行。中間結果都在 Drive，M2a、M3 可續跑。

## 防洩漏設計（對應課程「洩漏自我稽核五問」）

- 切分單位是病人；左右海馬迴合併為同一人的不對稱特徵。
- 切分前只做逐一受試者的運算（前處理、萃取、不對稱指標）。
- ComBat、ICC 篩選、冗餘移除、標準化、LASSO 全部在交叉驗證的摺內執行；模型 C 的訓練集 CV AUC 以巢狀交叉驗證估計。
- 保留測試集只開啟一次（`open_test_once()` 第二次呼叫會報錯）；開啟前以 Git tag `pre-test-lock` 標記程式與設定版本。

## 真實資料的下載與存放

- 執行順序：M1a 世代篩選 → M2a 逐人下載 → M2b 對齊檢查與分割品管 → M1b 品管後切分 → M3 萃取 → M4 → M5。
- `config/figshare_freesurfer_files.csv`：figshare freesurfer_zip 的 542 個檔案清單（file_id、檔名、大小、MD5），2026-11-27 取得；下載時逐檔核對 MD5。
- Drive 上每位受試者一個資料夾 `subjects/sub-<ID>/`：`T1w.nii.gz`、`FLAIR.nii.gz`、JSON、`aseg.nii.gz`、`orig.nii.gz`、`aseg.stats`、`hbt_lh.nii.gz`、`hbt_rh.nii.gz`、`flair_to_t1.tfm`、`features.json`。影像只存在 Drive，不進 repo。
- 品管：M2b 產生 `qc/qc_page_*.png`（不顯示側別與病理）與 `qc/qc_decisions_template.csv`；判讀結果存為 `qc_decisions.csv`（收／退）。

## 分割穩定性與特徵篩選設定

- 分割：IDEAS 公開、經人工檢查的 FreeSurfer 7.3.2 `aseg`（海馬迴 17／53；杏仁核 18／54 為探索性對照）。
- 兩套分割（B11）：FreeSurfer `aseg`（主要）與 FreeSurfer 海馬迴亞區模組的 HBT（`[lr]h.hippoAmygLabels.HBT.FSvoxelSpace.mgz`，取標籤 226／231／232）。兩者的 Dice 輸出於 `volumes_alignment.csv`。
- ICC：ICC(3,1)，以 aseg 與 HBT 兩套分割、在**左右不對稱指標**層級計算，門檻 0.75，只用訓練摺受試者。
- 擾動（探索性）：aseg 另做 1 體素侵蝕與膨脹（SimpleITK，3D）並萃取；擾動強度輸出於 `perturbation_stats.csv`，擾動 ICC 輸出於 `icc_train_final.csv`，只報告、不用於篩選。
- 摺內順序：ComBat（aseg 版本估計、同參數套用到 HBT 版本）→ ICC → 冗餘移除（Spearman |ρ| ≤ 0.90，以 `vol_ai` 為參照）→ 標準化 → LASSO（≤ 5 個）。
- 以上於 2026-11-25、開啟測試集前寫定，見 `docs/計畫書補充說明_20261125.md`。

## 輸出檔（`results/`，執行後產生）

| 檔案 | 內容 |
|---|---|
| `features_AI_v0/hbt/ero/dil.csv` | 原始版特徵矩陣（aseg／HBT／侵蝕／膨脹的不對稱指標） |
| `features_AI_filtered_train.csv`、`features_AI_filtered_test.csv` | 篩選後版特徵矩陣（ICC＋冗餘移除後） |
| `perturbation_stats.csv` | 遮罩擾動強度 |
| `stability_summary.json`、`icc_train_final.csv` | ICC 與篩選摘要 |
| `nested_cv_C.csv`、`selection_frequency_C.csv` | 模型 C 巢狀 CV 結果、穩定度選擇頻率 |
| `test_results.csv`、`fig_test.png`、`selected_features_C.csv` | 保留測試集結果（只開啟一次） |
| `versions.json` | 套件版本 |

## 可重現性

- OpenNeuro 資料集編號與版本：`ds005602` v1.0.0（2024-10-28，doi:10.18112/openneuro.ds005602.v1.0.0），只含原始 T1w 與 FLAIR。
- 臨床資料表：figshare tables_metadata 的 `Metadata_Release_Anon.csv`（放在 Drive 的 `metadata/`，不進 repo）。原始表有一列完全重複（ID 209），程式去除後為 442 人；原始檔不修改。
- FreeSurfer：figshare freesurfer_zip，每人一個 `<ID>.tar.gz`，下載連結格式 `https://ndownloader.figshare.com/files/<file_id>?private_link=b13b8bb41390d3f7a088`（2026-11-27 實測：`figshare.com/ndownloader/...` 對程式下載回傳驗證頁，須用 `ndownloader.figshare.com` 網域）；只取出 `mri/aseg.mgz`、`mri/orig.mgz`、`stats/aseg.stats`。
- FreeSurfer 分割、臨床與人口學資料表、切除遮罩由 IDEAS 團隊放在 figshare，連結見 OpenNeuro 的 README 與 cnnp-lab.com/ideas-data；實際使用的檔案與下載日期於第一次下載後填入。

- 隨機種子 2026；分析設定只寫在 `config/analysis.yaml`（PyRadiomics 參數只寫在 `config/params.yaml`），建模開始後不再擴充。
- `requirements-colab.txt`：notebook 實際安裝的套件；`requirements.txt`：完整環境（Colab 內建套件只核對、不強制改版，差異寫入 `versions.json`）。

## 進度

見 `docs/project_plan.xlsx`。期末口頭報告 W17–W18（12/24、12/31）；實作成果 W18（12/31）繳交。
