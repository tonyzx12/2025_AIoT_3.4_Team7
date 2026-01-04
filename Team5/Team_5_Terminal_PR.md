# 2025_AIoT_Team5_Terminal_PR
AIoT：Terminal Hello World（ESP-IDF / ESP32-P4）

## 目標
* 使用 Terminal 完成 ESP-IDF `hello_world` 範例的「編譯」與「燒錄」
* 能在序列埠終端（minicom）看到 `Hello world!`

## 前置需求
* 一台可上網的開發環境（建議：AWS SageMaker Notebook 的 Terminal / Amazon Linux 2023）
* 本機電腦（MacOS / Windows / Linux 皆可，以下以 MacOS 示範燒錄與 minicom）
* ESP32-P4 開發板（例如 ESP32-P4-EYE）與 USB Type‑C 線

> 注意：本文件採「雲端編譯 + 本機燒錄」流程。你也可以全程在本機做，但環境安裝會較久。

---

## AWS Linux 範例（SageMaker Terminal）
請使用這一區：在 AWS 上完成編譯，最後把 build 產物下載到本機。

### 1) 建立工作目錄
```bash
cd SageMaker
mkdir -p esp
cd esp
```

### 2) 下載 ESP-IDF
```bash
git clone --recursive https://github.com/espressif/esp-idf.git
```

### 3) 安裝工具鏈與啟用環境
```bash
cd esp-idf
./install.sh all
source ./export.sh
```

> 每次「重新開機 / 重新開新的 Terminal」都要重新做一次：
```bash
cd ~/SageMaker/esp/esp-idf
./install.sh all
source ./export.sh
```

### 4) 複製 hello_world 範例
```bash
cd ~/SageMaker/esp
cp -r $IDF_PATH/examples/get-started/hello_world .
cd hello_world
```

### 5) 指定目標晶片 + 編譯
以 ESP32‑P4 為例：
```bash
idf.py set-target esp32p4
idf.py menuconfig
idf.py fullclean
idf.py build
```

#### menuconfig 必改設定（ESP32-P4-EYE）
在 `idf.py menuconfig` 內，請確認以下路徑並把最小版本調成 `0.0`：

* `Component config -> Hardware settings -> Chip revision -> Minimum supported revision = 0.0`

> 說明：若未調整，可能會因晶片版本門檻導致無法正常執行。

成功後，會在 `build/` 目錄看到編譯結果。

---

## 下載 build 產物（從 AWS 下載到本機）
請把以下 3 個檔案下載到本機同一個資料夾（例如 `~/Downloads/hello_world_bins/`）：

* `build/bootloader/bootloader.bin`
* `build/partition_table/partition-table.bin`
* `build/hello_world.bin`

---

## 本機燒錄（MacOS 範例）
以下步驟在你的 MacOS Terminal 執行。

### 1) 安裝 esptool
若你沒有安裝過：
```bash
python3 -m pip install --user esptool
```

### 2) 找出序列埠裝置名稱
把板子用 USB 連上電腦後，查詢：
```bash
ls /dev/cu.*
```
常見會看到像：
* `/dev/cu.usbmodem1101`（數字可能不同）

接下來所有指令的 `-p` 請換成你自己的裝置。

### 3) 進入存放 .bin 的資料夾
假設你把檔案放在 `~/Downloads/hello_world_bins`：
```bash
cd ~/Downloads/hello_world_bins
```

### 4) 燒錄 Hello World
```bash
python3 -m esptool -p /dev/cu.usbmodem1101 --chip esp32p4 -b 115200 \
 --no-stub --before default_reset --after hard_reset write_flash --flash_mode dio \
 --flash_size 4MB --flash_freq 80m 0x2000 ./bootloader.bin 0x8000 ./partition-table.bin 0x10000 \
  ./hello_world.bin
```

如果燒錄不穩（或線材品質不好），把 `-b 460800` 改成 `-b 115200`。

---

## 序列埠監看（minicom）
### 1) 連線（波特率 115200）
```bash
python3 -m serial.tools.miniterm /dev/cu.usbmodem1101 115200
```

重新上電或按 Reset 後，應該能在輸出中看到：
* `Hello world!`

### 退出 minicom
* 按 `Ctrl-A`，再按 `Z`，再按 `X` 離開

---

## 常見問題（Troubleshooting）
* 找不到序列埠：確認 USB 線有資料傳輸功能、重新插拔、再跑一次 `ls /dev/cu.*`
* 燒錄失敗：把 baud rate 調低（`460800 -> 115200`），或先 `erase_flash` 再燒
