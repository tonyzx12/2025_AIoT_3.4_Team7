# 2025_AIoT_Team5：完成自訓練偵測模式（YOLO → ONNX → ESPDL）

## 1. 作業目標

- 使用「李珠珢」圖片資料，自行完成 **Object Detection** 訓練流程
- 產出可部署於 ESP32-P4（esp-dl runtime）可用的 **`.espdl`** 模型
- 在 PC 端先完成推論驗證（確保模型是真的有學到）

本 PR 只聚焦在「資料 → 訓練 → 匯出 → 量化」；
至於如何整合進 ESP32-P4 factory_demo、menuconfig、PSRAM/flash 設定，會在 `Team5/Team_5_Update_Model_Operation_PR.md`說明。

---

## 2. Dataset 準備

### 2.1 影像來源

- 來源：李珠珢 Facebook 圖片
- 數量：240 張
- 內容：正臉 / 側臉 / 不同光源與背景

### 2.2 標註工具：Roboflow

- Project type：Object Detection
- Label class：

```text
face
```

- 每張圖片只標註「李珠珢的臉」

### 2.3 匯出格式

- Format：YOLOv8
- Resize：`320 x 320`
- Split：train / valid / test（由 Roboflow 自動切分）

Roboflow 產生的 `data.yaml` 參考：

```yaml
train: ../train/images
val: ../valid/images
test: ../test/images

nc: 1
names: ['face']
```

---

## 3. YOLO 模型訓練

### 3.1 為什麼需要 ReLU 版本

ESP32-P4 端的 runtime 對某些 activation（例如 SiLU/Swish）支援度有限。
為了避免後續在 esp-dl 端遇到 runtime error，本專案採用 **ReLU 版本 YOLO**。

### 3.2 訓練模型設定

- 模型設定檔：`yolo11n_relu.yaml`
- activation：ReLU
- imgsz：320

### 3.3 訓練指令（範例）

> 這裡以 macOS MPS 為例；若你使用 CUDA 或 CPU，請自行調整 `device=`。

```bash
yolo detect train \
  model=yolo11n_relu.yaml \
  data=data.yaml \
  imgsz=320 \
  epochs=100 \
  batch=8 \
  device=mps
```

### 3.4 訓練結果（PC 端驗證）

- 使用 test 圖片做推論
- 觀察 confidence 約可到 0.85 左右（依資料集而定）
- 確認模型能在不同角度/光源下抓到臉

---

## 4. 匯出 ONNX（ESP 專用）

### 4.1 為什麼不直接用 Ultralytics 預設 export

Ultralytics 預設 export 可能包含：
- activation / graph 形式不利於 ESP 端支援
- Detect head 輸出格式與後端工具鏈不相容

因此本專案使用自製 `export_onnx.py` 來輸出更穩定的 ONNX。

### 4.2 `export_onnx.py` 的重點（摘要）

- 將 SiLU 等可能不相容的算子替代為可支援的組合
- 拆分 Detect head（輸出更可控）
- opset：11
-（可選）simplify

### 4.3 匯出指令

```bash
python export_onnx.py \
  --weights best.pt \
  --imgsz 320 \
  --output best_split.onnx \
  --simplify
```

---

## 5. ONNX → ESPDL 量化

### 5.1 為什麼要量化

- ESP32-P4 端推論以 INT8 為主
- 模型體積更小、推論更快

### 5.2 量化設定重點（本專案）

- Target：`esp32p4`
- Bits：8
- Calibration images：`valid/images`
-（如果有）避免不穩定 fusion（依你的量化工具腳本設定）

### 5.3 量化流程

> 量化程式/指令請以你目前專案實際使用的腳本為準

### 5.4 輸出模型

```text
leejueun_detect_yolo11n_320_s8_v1.espdl
```

