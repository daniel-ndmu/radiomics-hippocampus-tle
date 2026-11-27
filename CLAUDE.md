# CLAUDE.md — Radiomics 學期專題

## 專案背景
- 課程：人工智慧於醫學影像組學的分析與應用（林彥聖老師），115-1。
- 題目：海馬迴 MRI 影像組學輔助內側顳葉癲癇側化；資料為 IDEAS（OpenNeuro）。
- 繳交：期末口頭報告 W17–W18（12/24、12/31，8 分鐘 + 4 分鐘問答）；實作成果 W18（12/31）：完整程式碼、特徵矩陣、模型結果、可重現說明文件。
- 課程要求分析在 Google Colab 執行。這個 repo 在本機維護，推上 GitHub 後於 Colab 開啟執行。

## 必須遵守的規則
1. **不得提交任何影像資料**（.nii、.nii.gz、.mgz、raw/、derivatives/）。特徵矩陣與結果表可以提交。
2. **先切分**：病人為單位、依側別分層、test_size 0.30、seed 2026。切分前只允許逐一受試者的運算。
3. **摺內處理**：ComBat、ICC 篩選、冗餘移除、標準化、特徵選擇、超參數搜尋，全部只用訓練摺資料。
4. **保留測試集只開啟一次**。不要為了改善測試結果回頭修改模型；發現問題記為限制。
5. **事先寫定的設定**在 `config/analysis.yaml`，不得擴充超參數範圍或更改主要假說（ΔAUC ≥ 0.05）。
6. 主要比較只有一組：測試集上模型 C vs 模型 A；其他分析一律標示為探索性。
7. 文件與註解使用繁體中文；程式識別字使用英文。
8. 引用文獻必須能在 PubMed 檢索；不確定的書目不要寫入。

## 目前狀態
- `notebooks/hippocampal_radiomics_TLE_colab_v2.ipynb`：2026-11-25 已在 Colab 以示範模式完整跑通 M0–M5（M4 巢狀 CV 約 13 分鐘）；尚未下載真實資料。
- 2026-11-25（開啟保留測試集前）寫定計畫書補充說明 B1–B4，見 `docs/計畫書補充說明_20261125.md`；設定同步於 `config/analysis.yaml`（標註 B1–B4），notebook M4–M5 已依此修正：
  - B1 冗餘移除以 `vol_ai` 為已保留的參照（與其 |ρ| > 0.90 的特徵移除）。
  - B2 模型 C 的訓練集 CV AUC 以巢狀 CV 估計（外層 5 摺 × 5 次，內層 5 摺選 C）；B、C-RF 維持非巢狀並標示探索性。
  - B3 摺內順序為 ComBat → ICC（ComBat 以原始遮罩版本估計，同參數套用到侵蝕／膨脹版本）。
  - B4 ICC(3,1) 在左右不對稱指標層級計算，門檻 0.75。
- 同日追加（皆在開啟測試集前）：
  - B6 主要假說三級判定：ΔAUC ≥ 0.05 且 DeLong P < 0.05＝支持；P < 0.05 且 0 < ΔAUC < 0.05＝有增量但未達預設門檻；P ≥ 0.05＝本樣本無法證明增量。
  - B7 假說第二條為單向：巢狀 CV AUC − 測試 AUC < 0.05；測試高於 CV 視為抽樣變異。
  - neuroCombat 0.2.12 的 `neuroCombatFromTraining` 與 NumPy 2.x 不相容，改用 notebook 內 `combat_from_training`（計算相同）；三個遮罩版本一律用同一函式調和。
  - `requirements.txt` 鎖定 Colab 實測版本；PyRadiomics 固定於 GitHub commit 8ed5793（PyPI 版在 Python 3.13 無法編譯）。
- 2026-11-27 追加（尚未切分、未看影像特徵）：B8 年齡級距全納入；B9 世代＝Op_Type T Lx 且 Pathology ∈ {HS, OTHER}；B10 無 3D FLAIR 者排除。依臨床資料表試算：316 → 257 → 244 人（左 135、右 109；HS 205、非 HS 39），尚未扣除分割品管退件。
- 資料來源：OpenNeuro ds005602 v1.0.0（原始 T1w、FLAIR）；FreeSurfer 與臨床資料表在 figshare（見 README）。
- 補充說明不改主要假說、不擴充超參數範圍、不新增模型；之後不得再以補充說明為由修改這些項目。
- 計畫書中標示「待確認」的人數（排除流程、HS 與 ILAE 事件數）需下載資料後填入並凍結。

## 下一步
- 改寫 M2：逐人下載 OpenNeuro T1／FLAIR 與 figshare FreeSurfer（只取 aseg.mgz、orig.mgz、aseg.stats），產生品管疊圖與 qc_decisions.csv；品管完成後才切分。
- notebook 目前在程式內寫定參數（M0、M3）；`config/params.yaml` 與 `config/analysis.yaml` 是同一組數值的文件版本。之後可改為由 notebook 直接讀取 config，避免兩處不一致。
