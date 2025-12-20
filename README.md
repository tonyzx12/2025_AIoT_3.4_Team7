# Hour 4：Edge AI 實戰 — COCO / YOLO / esp-dl 與模型部署
（課程時間：60 分鐘）

---

## 一、本小時課程目標（Learning Objectives）

完成本小時課程後，學生應能：

- 理解 **Edge AI 與傳統 AI 在部署上的根本差異**
- 清楚分辨「模型檔案」在不同階段的型態與用途
- 看懂 **COCO / YOLO 在 ESP32-P4 上的實際程式碼**
- 理解 **esp-dl 與 espdl 模型格式的角色**
- 能說明 ONNX → espdl 的完整生成流程與指令

---

## 二、Edge AI 不是「把模型丟進來跑」

在 PC / Server 上，AI 流程通常是：

```
Dataset → Training → Model → Inference
```

在 AIoT / Edge AI 上，實際流程是：

```
Dataset
  ↓
Training (PyTorch)
  ↓
ONNX（交換格式）
  ↓
量化 / 編譯（esp-dl）
  ↓
espdl（部署格式）
  ↓
MCU Runtime
```

> **部署流程本身，就是 Edge AI 的一半難度。**

---

## 三、本專案使用的 Edge AI 技術堆疊

### 3.1 AI 模型：YOLO11n（COCO）

- 任務：Object Detection
- 資料集：COCO
- 模型特性：
  - 輕量化
  - 適合量化
  - 可在 MCU 上即時推論
  - 高精度
  <img width="626" height="360" alt="image" src="https://github.com/user-attachments/assets/781daa19-bf48-4587-802e-ae23fc30f052" />
  
---
  <img width="536" height="367" alt="image" src="https://github.com/user-attachments/assets/797ebe38-90c1-4b6b-9d50-0bc5d81d725f" />
  
---
  <img width="833" height="437" alt="image" src="https://github.com/user-attachments/assets/8cfa71f4-33cb-4cd2-87e3-c0402cbaf667" />




---

### 3.2 AI Runtime：esp-dl

- Espressif 官方 Edge AI Runtime
- 提供：
  - 模型載入
  - Preprocess / Postprocess
  - 記憶體管理

---

## 四、COCO Detect 模組程式碼導讀（核心）
- 什麼是微軟 COCO 資料集？
  - 微軟通用物件上下文資料集（COCO）是評估最先進電腦視覺模型效能的黃金標準基準資料集。 COCO 包含超過 33 萬張圖像，其中超過 20 萬張圖像已標註，涵蓋數十個物件類別。 COCO 是一個合作項目，由來自眾多知名機構（包括Google、加州理工學院和喬治亞理工學院）的電腦視覺專家共同維護。
  - COCO 資料集旨在代表我們在日常生活中經常遇到的各種事物，從自行車等交通工具到狗等動物再到人。
  - COCO 資料集包含來自 80 多個「物件」類別和 91 個通用「物品」類別的影像，這意味著該資料集可以比小規模資料集更有效地用於對通用模型進行基準測試。
  - 此外，COCO 資料集還包含：
    - 121,408 張圖片
    - 883,331 物件註釋
    - 80類數據
    - 影像中位數比例為 640 x 480
<img width="756" height="175" alt="image" src="https://github.com/user-attachments/assets/b8e5bb36-4d32-4b4d-8816-03e0f2fa2516" />



### 4.1 模組位置與檔案

**檔案位置：**
- `components/coco_detect/coco_detect.cpp`
- `components/coco_detect/coco_detect.hpp`

**角色定位：**
- 封裝 YOLO11n 模型
- 對 App 提供「簡單 API」

---

### 4.2 COCODetect 類別（封裝層）

```cpp
COCODetect::COCODetect(model_type_t model_type)
```

**說明：**
- App 只需選模型類型
- 不需知道模型檔案路徑
- 不需知道模型存放位置

> 這是「AI Service Layer」的概念。

---

### 4.3 模型載入（dl::Model）

```cpp
m_model = new dl::Model(
    path,
    model_name,
    model_location,
    0,
    dl::MEMORY_MANAGER_GREEDY,
    nullptr,
    param_copy
);
```

**關鍵說明：**
- `path`：Flash / Partition / SD
- `param_copy`：是否複製參數到 PSRAM
- 會依 ESP32-P4 記憶體條件動態決定

---

### 4.4 為什麼呼叫 minimize()

```cpp
m_model->minimize();
```

**用途：**
- 釋放初始化用 buffer
- 降低 Runtime 記憶體占用

> 在 Edge AI 中，「不用的記憶體一定要釋放」。

---

## 五、Preprocess 與 Postprocess（AI 成敗關鍵）

### 5.1 Preprocess（影像前處理）

在 `coco_detect.cpp` 中：

```cpp
m_image_preprocessor =
    new dl::image::ImagePreprocessor(
        m_model,
        {0, 0, 0},
        {255, 255, 255}
    );
```

**負責：**
- Resize
- Normalize
- Color format 調整

---

### 5.2 Postprocess（YOLO 特有）

```cpp
m_postprocessor =
    new dl::detect::yolo11PostProcessor(
        m_model,
        0.25,   // confidence threshold
        0.7,    // NMS threshold
        10,     // max boxes
        {...}
    );
```

**說明：**
- Threshold 會直接影響「準不準」
- 不只是模型權重的問題

---

## 六、模型檔案的生命週期（非常重要）

### 6.1 訓練階段模型（PyTorch）

- `.pt`
- 僅用於訓練 / 評估
- **不能部署到 MCU**

---

### 6.2 ONNX（中繼交換格式）

**檔案範例：**
- `yolo11n.onnx`
- `yolo11n_320.onnx`

**用途：**
- 介於 Training 與 Deployment 之間

---

### 6.3 ONNX 生成方式

**工具：**
- PyTorch
- YOLOv11
- 自訂腳本：`export_onnx.py`

**生成指令範例：**

```bash
python export_onnx.py   --weights yolo11n.pt   --imgsz 640   --output yolo11n.onnx
```

---

### 6.4 espdl（最終部署格式）

**檔案範例：**
- `coco_detect_yolo11n_s8_v1.espdl`

**特性：**
- INT8 / Mixed precision
- 記憶體佈局最佳化
- 可直接由 esp-dl 載入
<img width="578" height="245" alt="image" src="https://github.com/user-attachments/assets/d370879b-1aba-4efc-98ff-30f1e5107625" />

<img width="585" height="196" alt="image" src="https://github.com/user-attachments/assets/b9b0d098-723c-4a9a-b2de-8efa2694426a" />

<img width="455" height="194" alt="image" src="https://github.com/user-attachments/assets/f9c1cada-39e8-4b53-a99f-b1b6cee7e661" />

---

## 七、ONNX → espdl 的生成流程

### 7.1 為什麼不能直接用 ONNX？

- Float32
- 記憶體需求過大
- 不適合 MCU

---

### 7.2 espdl 生成工具

- Espressif esp-dl toolchain
- `pack_espdl_models.py`

此步驟通常在 **build 階段自動完成**。

---

## 八、效能與解析度的取捨（640 vs 320）

| 模型 | Input | 推論時間（P4） |
|----|----|----|
| YOLO11n | 640x640 | ~3 ms |
| YOLO11n | 320x320 | ~0.6 ms |

**教學重點：**
- Edge AI 永遠在 trade-off
- 不存在「又快又準又省」

---

## 九、常見錯誤與教學提醒

- 認為模型越大越好
- 忽略 Preprocess / Postprocess
- 不釋放模型初始化資源

---

## 十、課堂練習（10 分鐘）

請學生回答：

1. 為什麼 espdl 才是部署格式？
2. param_copy 對效能有什麼影響？
3. 如果 AI 很慢，應該先檢查哪一段？

---

## 十一、本小時重點整理

- Edge AI 的難點在部署
- 模型有完整生命週期
- esp-dl + espdl 是關鍵
- P4 讓 YOLO 在 MCU 上變得實用

---

## 十二、Next Hour 預告

**Hour 5：UI、事件、決策與儲存 — AIoT 的行為閉環**
