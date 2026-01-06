# 燒錄Factory範例


## 複製範例
使用 Git 複製範例
```bash
git clone --recursive https://github.com/espressif/esp-dev-kits.git
```

進入 factory_demo 專案目錄
```bash
cd C:\workspase\Python\ESP\esp-dev-kits\examples\esp32-p4-eye\examples\factory_demo
```

在專案目錄執行
```bash
#設定晶片為 ESP32-P4
idf.py set-target esp32p4

idf.py menuconfig
```
![image](https://hackmd.io/_uploads/rJJRAXK4Zl.png)

## 專案編譯（Build）
```
idf.py build
```
終端機出現類似訊息即成功：
```
Project build complete.
```
![image](https://hackmd.io/_uploads/rkHfuNFEZg.png)



## 燒錄至開發板（Flash）

1. 確認 ESP32 已透過 USB 連接到 Windows 11
2. 於「裝置管理員 → 連接埠 (COM & LPT)」確認 COM Port
![image](https://hackmd.io/_uploads/SyM7GEFNZg.png)

3. 燒錄
```bash
idf.py -p COM4 flash
```
![image](https://hackmd.io/_uploads/HyC3_NFNbx.png)
![S__41615390](https://hackmd.io/_uploads/HyerK4t4bg.jpg)
