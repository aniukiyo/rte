# rte
# 🌙 STM32 睡眠追蹤與智慧照明控制系統 (Night Guardian)

本專案基於 **STM32 NUCLEO-G474RE** 開發板，結合**多段式 PWM 智慧夜燈控制**、**外部中斷（EXTI）夜間起床監測**與 **UART 串列埠命令列介面 (CLI)**。專為提供舒適的夜間光線調整與自動睡眠起床行為紀錄而設計。

---

## 🌟 核心功能特色 (Features)

1. **💡 四段式 PWM 漸進式調光 (TIM3 PWM)**
   - 透過實體按鈕循環切換光線亮度：
     - **0% (LIGHT_OFF)**：熄滅模式（準備入睡）
     - **20% (LIGHT_LEVEL_1)**：微亮柔光夜燈（夜間摸黑不刺眼）
     - **60% (LIGHT_LEVEL_2)**：中等照明模式
     - **100% (LIGHT_LEVEL_3)**：全亮模式
2. **🛌 自動起床事件紀錄 (GPIO EXTI)**
   - 使用 PC8 外部中斷監測按鈕觸發。
   - 每次夜間按鈕開燈即視為一次「起床事件」，自動累加至 `wake_up_count`。
   - 內建 300ms 軟體時間戳消抖演算法（Debounce），防止按鈕彈跳造成誤計數。
3. **💻 UART 串列埠 CLI 互動介面 (USART2 @ 115200 8N1)**
   - 使用 64-byte 非阻塞式環形接收緩衝區（Ring Buffer）與 RX 中斷接收。
   - 支援下列指令（於 Tera Term 或 Serial Monitor 內輸入）：
     - `report`：輸出當前夜間起床累計次數報告。
     - `new`：清空起床紀錄數值，重新開箱一段新的睡眠統計。

---

## 🛠️ 硬體規格與腳位配置 (Hardware Setup)

- **微控制器**：STMicroelectronics STM32G474RE (Arm® Cortex®-M4)
- **開發板**：STM32 NUCLEO-G474RE

| 週邊 (Peripheral) | 引腳 (Pin) | 模式 / 設定 (Mode / Config) | 功能描述 |
| :--- | :--- | :--- | :--- |
| **PC8** | `PC8` | GPIO EXTI8 (Rising Edge, Pull-up) | 實體開燈/起床觸發按鈕 |
| **TIM3** | `PC6` | PWM Generation CH1 (1kHz, Period=999) | 智慧燈光調光控制 |
| **TIM2** | N/A | Base Interrupt (1s Period) | 系統基礎定時器 |
| **USART2** | `PA2 (TX) / PA3 (RX)` | Baud 115200, 8N1, Interrupt Enable | ST-Link Virtual COM Port 傳輸 |

---

## 💻 CLI 指令說明 (Command Line Interface)

使用 Serial Terminal（如 Tera Term、PuTTY 或 STM32 CubeIDE Terminal）連接可進行互動：

| 指令 (Command) | 說明 (Description) | 系統回應範例 (Response Example) |
| :--- | :--- | :--- |
| `report` | 查詢目前累積的起床次數 | `report() -> Wake Ups: 3 times` |
| `new` | 重置睡眠紀錄與次數 | `New sleep log initialized.` |

> ⚠️ **Terminal 設定說明**：
> - 波特率：`115200` / Data bits: `8` / Stop bits: `1` / Parity: `None`
> - 請在 Terminal 中勾選 **Local Echo（本地回顯）** 方便觀察打字過程。
> - 換行符號（Transmit / Newline）請設定為 `CR` 或 `CR+LF`。

---

## 🚀 快速開始與編譯 (Getting Started)

### 1. 軟體需求
- **STM32CubeIDE** (v1.10.0 以上版本)
- **STM32CubeMX**
- **Tera Term** 或任意 UART 序列埠監控軟體

### 2. 硬體設定注意事項 (CubeMX NVIC)
使用 CubeMX 生成專案時，請務必確認以下中斷已勾選：
1. **Connectivity** -> **USART2** -> **NVIC Settings** -> 勾選 `USART2 global interrupt`。
2. **Core** -> **NVIC** -> 勾選 `EXTI line[9:5] interrupts`（優先權可設為 `2`）。

### 3. 編譯與燒錄
1. 下載本專案原始碼並在 STM32CubeIDE 中開啟。
2. 連接 NUCLEO-G474RE 開發板至電腦。
3. 點擊 **Build** (`Ctrl + B`) 並選擇 **Run / Debug** 燒錄至晶片。
4. 打開 Tera Term 連接對應的 COM Port，按下 Reset 鍵看到 `STM32 UART2 Ready!` 即可開始體驗！

---

## 📁 檔案結構 (Project Structure)

```
.
├── Core
│   ├── Inc
│   │   ├── main.h
│   │   ├── stm32g4xx_hal_conf.h
│   │   └── stm32g4xx_it.h
│   └── Src
│       ├── main.c           # 主程式邏輯（PWM調光、CLI解析與中斷回調）
│       ├── stm32g4xx_hal_msp.c
│       └── stm32g4xx_it.c
├── README.md                # 專案說明文件
└── stm32_sleep_tracker_report.html # 完整專題報告 (PDF排版)
```

---

## 📄 授權條款 (License)

本專案基於 MIT License 開源，歡迎自由修改與延伸開發！

