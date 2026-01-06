# 完成Terminal 範例 Markdown 操作手冊 PR

## 一、開發環境

- 作業系統：Windows 11
- 開發框架：ESP-IDF
- 操作終端：ESP-IDF Command Prompt
- 範例專案：hello_world（ESP-IDF 內建範例）

首先下載並安裝windows專用ESP-IDF開發工具
https://docs.espressif.com/projects/esp-idf/zh_CN/stable/esp32/get-started/windows-setup.html

![image](https://hackmd.io/_uploads/SJu-rWYNWg.png)



## 二、建立專案（Project Building）

1. 開啟 **ESP-IDF CMD**
2. 切換至工作目錄

```bash
cd C:\esp
```
3.使用 Git 遞歸克隆 esp-idf 專案的 Git 儲存庫：
```bash
git clone --recursive https://github.com/espressif/esp-idf.git
```

4.複製 HelloWorld至工作目錄
```
esp-idf\examples\get-started\hello_world
```
5.進入 HelloWorld 專案目錄
```bash
cd C:\workspase\Python\ESP\hello_world
```


## 三、專案設定（menuconfig）

在專案根目錄執行(c:\esp\2025_AIoT_1\Project_Building\hello_world)：
```bash
#設定晶片為 ESP32-P4
idf.py set-target esp32p4

idf.py menuconfig
```
![image](https://hackmd.io/_uploads/rJJRAXK4Zl.png)

建議檢查項目：

- **Serial flasher config**
  - Flash baud rate（可先維持預設）
- **Serial Monitor**
  - 預設 UART / 監看埠（依實際板子連線情況）
- 其餘維持預設即可

完成後按提示儲存並離開。

---

## 四、專案編譯（Build）
```
idf.py build
```
終端機出現類似訊息即成功：
```
Project build complete.
```
---

## 五、燒錄至開發板（Flash）

1. 確認 ESP32 已透過 USB 連接到 Windows 11
2. 於「裝置管理員 → 連接埠 (COM & LPT)」確認 COM Port
![image](https://hackmd.io/_uploads/SyM7GEFNZg.png)

3. 燒錄
```bash
idf.py -p COM4 flash
```
---

## 六、序列監看（Monitor）

查看DEBUG輸出
```bash
idf.py -p COM4 monitor
```

看到類似輸出即成功：
```
Hello world!
This is esp32 chip with 2 CPU cores
ESP-IDF version: x.x.x
```
![image](https://hackmd.io/_uploads/SyKnmVtVbe.png)


離開監看模式快捷鍵：

    Ctrl + T、Ctrl + X

---

## 七、程式碼位置與說明

HelloWorld 主程式位於：
```
hello_world\main\hello_world_main.c
```
ESP-IDF 專案的進入點為 `app_main()`，核心示意如下：

```c
void app_main(void)
{
    printf("Hello world!\n");

    /* Print chip information */
    esp_chip_info_t chip_info;
    uint32_t flash_size;
    esp_chip_info(&chip_info);
    printf("This is %s chip with %d CPU core(s), %s%s%s%s, ",
           CONFIG_IDF_TARGET,
           chip_info.cores,
           (chip_info.features & CHIP_FEATURE_WIFI_BGN) ? "WiFi/" : "",
           (chip_info.features & CHIP_FEATURE_BT) ? "BT" : "",
           (chip_info.features & CHIP_FEATURE_BLE) ? "BLE" : "",
           (chip_info.features & CHIP_FEATURE_IEEE802154) ? ", 802.15.4 (Zigbee/Thread)" : "");

    unsigned major_rev = chip_info.revision / 100;
    unsigned minor_rev = chip_info.revision % 100;
    printf("silicon revision v%d.%d, ", major_rev, minor_rev);
    if(esp_flash_get_size(NULL, &flash_size) != ESP_OK) {
        printf("Get flash size failed");
        return;
    }

    printf("%" PRIu32 "MB %s flash\n", flash_size / (uint32_t)(1024 * 1024),
           (chip_info.features & CHIP_FEATURE_EMB_FLASH) ? "embedded" : "external");

    printf("Minimum free heap size: %" PRIu32 " bytes\n", esp_get_minimum_free_heap_size());

    for (int i = 10; i >= 0; i--) {
        printf("Restarting in %d seconds...\n", i);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
    printf("Restarting now.\n");
    fflush(stdout);
    esp_restart();
}
```


## 八、實作成果

- 成功使用 Git clone 取得課程專案
- 成功進入 HelloWorld 專案目錄並完成 `menuconfig`
- 成功完成 `idf.py build`
- 成功完成 `idf.py flash`
- 成功在 `idf.py monitor` 看到 HelloWorld 輸出
