# Host-Device Control Engineering Methodology

## 一套從實務經驗整理而來的實驗性工程方法論

這是一個實驗性的 Side Project。

它的起點，不是想再發明一套新的通訊協定，也不是想建立一個適用所有產品、所有平台與所有產業的萬用框架。

這個專案真正想做的事情，是把我多年參與韌體、電腦程式、硬體整合與產品開發過程中，逐漸累積下來的「隱性工程經驗」，透過與 AI 持續討論、整理、質疑與修正，轉換成一套比較有系統的工程方法論。

很多工程經驗，平常只存在於工程師的腦中。

例如：

- 一個系統在開始大量寫程式之前，應該先定義哪些事情？
- PC、SoC、MCU 與各個 Device Module 之間的責任應該如何劃分？
- 命令、回覆、狀態與即時資料應該如何區分？
- 發生逾時、斷線、資料錯誤、模組失效或裝置重啟時，系統應該如何處理？
- 哪些資訊應該成為所有程式共同遵循的唯一來源？
- 如何避免 PC、SoC 與 MCU 各自維護不同版本的通訊規格？
- 如何隔離不同 RTOS、MCU 平台與外部 IC 的差異，並評估後續抽換的可行性？
- 如何設計 Logging、Testing、Concurrency 與錯誤處理？
- 哪些規則可以透過工具自動檢查？
- 哪些判斷仍然必須由工程師負責？
- 如何讓 AI 產生的程式碼，不只是可以編譯，而是更接近真正可維護、可審查、可測試與可驗證的工程實作？

這些事情很難只靠一份程式碼說清楚，也很難從單一產品或單一技術平台直接歸納出來。

因此，這個專案嘗試把這些經驗整理成一套可閱讀、可討論、可實作、可檢查，也可以持續修正的工程架構。

---

# 文件定位與權威邊界

本儲存庫是整套 Host-Device Control 工程方法論的白話介紹與導覽入口。

它的用途是協助一般讀者、工程師及 AI 快速理解這套方法論的背景、範圍、儲存庫分工與概念驗證方向。

本儲存庫本身：

- 不建立新的正式工程規則
- 不修改或取代 Framework 中的權威文件
- 不取代個別專案的需求、風險控制、硬體規格或 Project Protocol
- 不代表所有相關文件都已經完成、定版或通過實際產品驗證
- 不應被單獨視為產品設計、測試證據、法規符合性或發布核准依據

正式的方法論文件、文件版本、文件狀態及權威主題，應以 [`host-device-control-framework`](https://github.com/jkman357/host-device-control-framework) 的 GitHub `main`、其中的 [`authority-registry.yaml`](https://github.com/jkman357/host-device-control-framework/blob/main/authority-registry.yaml)，以及其所指向的權威文件為準。

實際專案中的產品需求、風險控制、硬體規格、Project Protocol、測試證據與核准紀錄，則仍應以該專案所指定及核准的來源為準。

若本 Overview 與上述權威來源不一致，應以上述權威來源為準。

---

# 名詞說明：Host／Device 與 Coordinator／Node

為了讓一般讀者容易理解，本文件主要使用 Host 與 Device。

在正式 Framework 文件中，則使用 Coordinator 與 Node 表示通訊關係中的系統角色。

一般情況下：

- Host 通常扮演 Coordinator
- Device 通常扮演 Node

但 Coordinator 與 Node 是「關係角色」，不是固定綁定某一種硬體、作業系統或程式語言。

例如：

- PC 可以是 MCU 的 Coordinator
- MCU 可以是 PC 的 Node
- SoC 可以是上層 PC 的 Node
- 同一顆 SoC 也可以同時是下層 MCU 的 Coordinator
- 一顆 Main MCU 可以是多顆 Sub MCU 的 Coordinator

因此，Coordinator 不一定是 PC，Node 也不一定是 MCU。

實際角色應依特定通訊關係、控制權、責任邊界與 Protocol 定義判定。

---

# 這套方法論的定位

這是一套以 Host-Device Control System 為主要實驗場景，涵蓋軟體工程規則、架構模式、並行處理、Logging、Testing、單一資訊來源治理，以及 RTOS 抽象化與 Device 模組化後續實作方向的實驗性工程方法論。

Host 與 Device 之間的控制及資料交換，是目前最主要的概念驗證場景，但不是這套方法論唯一關注的內容。

通訊協議只是整個系統中的一部分。

一個真正可以長期維護、持續擴充、跨平台移植並被不同工程師接手的系統，還需要同時處理：

- Software Engineering Rules
- Architecture Patterns
- Embedded C Coding Rules
- C# Coding Rules
- Concurrency Guide
- Logging Guide
- Testing Guide
- Protocol Definition
- Single Source of Truth
- Operating System Abstraction
- Hardware Abstraction
- Device Abstraction
- Error Handling
- Version and Change Management
- Automated Validation and CI
- Human and AI Responsibility Boundaries

因此，這套方法論比較接近一組可組合、可延伸的工程指引，而不只是一份 PC 與 MCU 的通訊規格。

---

# 這套方法論想解決什麼問題？

Host、SoC、MCU 與外部裝置的整合，表面上看起來只是「把各個模組接起來」。

但真正進入專案後，通常會同時遇到許多問題：

- PC 與 MCU 對同一個命令的理解不同
- 文件寫的內容與實際程式碼不一致
- 通訊格式只存在於某一端的程式碼中
- 不同工程師各自保留一份規格
- UI、通訊、Protocol 與產品邏輯全部混在一起
- MCU 的中斷、通訊解析與功能執行互相耦合
- 應用程式直接依賴特定 RTOS API
- 更換 RTOS 時需要修改大量應用層程式
- 更換 Sensor、Charger IC 或 Smart Battery 時影響整個系統
- 裝置模組無法被模擬或替換，導致測試困難
- 命令送出後，沒有定義多久算逾時
- 裝置忙碌時，不知道應該拒絕、排隊還是覆蓋命令
- 發生斷線後，系統不知道如何恢復
- 即時資料與控制命令共用相同流程，互相影響
- 多執行緒或多 Task 共享資料，但沒有清楚同步規則
- 系統發生問題時，Log 無法還原事件經過
- 測試只驗證正常流程，沒有覆蓋錯誤與邊界條件
- 程式雖然可以執行，但難以測試、維護與移植
- AI 可以快速生成大量程式碼，卻缺乏共同規則可以審查

這套方法論希望把這些問題往前移。

在大量寫程式之前，先定義：

- 系統架構
- 模組責任
- 公開介面
- 通訊協議
- 單一資訊來源
- Coding Rules
- Concurrency Rules
- Logging Strategy
- Testing Strategy
- RTOS 與硬體抽象方向
- 錯誤處理
- 版本與變更規則
- 自動檢查方式
- 人與 AI 的責任邊界

這樣做不一定能消除所有問題，但可以讓問題更早被看見、更容易被討論，也更容易留下可以追蹤的工程紀錄。

---

# Software Engineering Rules

這套方法論會整理軟體開發中應共同遵循的工程規則。

內容不只包含程式碼的命名、縮排或排版，也包含：

- 模組責任應如何劃分
- 公開介面與內部實作應如何分離
- 資料由哪一個模組擁有
- 模組之間是否可以直接存取彼此內部資料
- 錯誤是否可以被忽略
- 相依關係應如何控制
- 平台相關程式碼應放在哪一層
- 哪些內容可以共用
- 哪些內容必須依專案實際需求實作
- 哪些內容可以由 AI 協助產生
- 哪些內容必須由工程師確認
- 如何避免程式雖然可以執行，卻無法測試、擴充與維護

這些規則會分別落實到 Embedded C、C# 及其它可能使用的程式語言中。

不同語言會有不同的實作方式，但仍然應遵守共同的系統原則。

---

# Architecture Patterns

這套方法論也會探討可重複使用的軟體架構模式。

例如：

- Layered Architecture
- Event-driven Architecture
- State Machine
- Message Queue
- Command Dispatcher
- Publish-Subscribe
- Dependency Inversion
- Hardware Abstraction
- Operating System Abstraction
- Device Abstraction
- Adapter Pattern
- Interface-based Design

這些架構模式不是為了讓程式看起來更複雜，而是希望解決實際專案中常見的問題：

- UI 與通訊程式綁得太緊
- 通訊解析直接控制硬體
- MCU Driver 與產品邏輯混在一起
- 更換 RTOS 時需要重寫大量應用程式
- 更換 Sensor、Charger IC 或 Smart Battery 時影響整個系統
- 測試時無法替換真實硬體
- 模組之間互相依賴，修改一處便影響多處
- 新增功能時只能持續堆疊條件判斷
- 系統責任隨著專案成長而逐漸失去邊界

架構模式的目的，是讓系統中的變動被限制在合理範圍內。

---

# Concurrency Guide

Host、SoC 與 MCU 系統通常同時執行多項工作。

例如：

- 接收通訊資料
- 傳送命令
- 更新畫面
- 控制馬達
- 讀取感測器
- 管理充電器
- 讀取 Smart Battery
- 寫入紀錄
- 執行逾時判斷
- 處理錯誤
- 執行背景測試

因此，這套方法論也包含 Concurrency Guide，用來整理並行、非同步與多執行緒程式的設計原則。

在 MCU 端，可能涉及：

- Interrupt
- Main Loop
- Event Queue
- RTOS Task
- Message Queue
- Semaphore
- Mutex
- Shared Resource
- Task Priority
- Timeout
- Race Condition
- Priority Inversion

在 PC 端，可能涉及：

- UI Thread
- Background Thread
- `async` 與 `await`
- `Task`
- `CancellationToken`
- Event Callback
- Thread-safe Collection
- Synchronization Context
- Connection Lifecycle

Concurrency Guide 的目的，不是要求所有功能都拆成獨立執行緒或 RTOS Task。

真正的重點是清楚回答：

- 哪一個執行環境負責哪一項工作？
- 哪些資料可能被同時存取？
- 哪些操作可以阻塞？
- 哪些操作必須可以取消？
- 發生逾時後如何恢復？
- 系統關閉或斷線時，背景工作如何停止？
- 如何避免 Race Condition、Deadlock 與資料不同步？
- 如何避免不必要的 Task、Thread 與同步機制增加系統複雜度？

---

# Logging Guide

當系統發生問題時，只知道「不能動」、「斷線」或「程式當掉」通常不足以找到真正原因。

因此，這套方法論也會整理 Logging Guide。

Logging 不只是輸出一些除錯文字，而是要考慮：

- 哪些事件需要留下紀錄
- Log Level 如何分類
- 時間戳如何產生
- Host 與 Device 的時間如何對應
- Command 與 Response 如何關聯
- 錯誤碼與系統狀態如何記錄
- 高頻資料是否應全部保存
- Log 是否會影響即時效能
- 儲存空間不足時如何處理
- 敏感資訊是否需要隱藏
- 如何讓測試人員與工程師可以重建問題發生過程

PC、SoC 與 MCU 的資源條件不同，因此 Logging 的實作方式也會不同。

但各端仍然需要使用一致的事件名稱、錯誤定義與系統語意，否則各端的紀錄仍然無法互相對照。

---

# Testing Guide

這套方法論不把測試視為程式完成後才開始進行的工作。

Testing Guide 會從設計階段開始考慮：

- 模組是否可以獨立測試
- Hardware-dependent Code 是否可以替換
- Protocol 是否可以使用模擬器驗證
- 正常流程與異常流程是否都有測試
- Timeout、斷線與重新連線如何測試
- 不同版本的相容性如何測試
- PC 與 MCU 是否能進行自動化整合測試
- 長時間運作是否穩定
- 高頻資料是否會遺失
- 系統忙碌時是否仍能正確回應
- 更換 RTOS、MCU 或 Device Module 後，需要重新驗證哪些內容

測試可能包含：

- Unit Test
- Module Test
- Protocol Test
- Integration Test
- Hardware-in-the-loop Test
- System Test
- Stress Test
- Long-duration Test
- Fault Injection
- Regression Test

這些測試不一定都能在每個概念驗證中完整實作，但應該在架構中預留可測試性。

---

# OSHAL：後續跨執行環境實作與驗證方向

Firmware 端可能使用不同的執行環境。

例如：

- Bare-metal
- FreeRTOS
- Azure RTOS ThreadX
- Zephyr RTOS
- TI-RTOS
- 其它 RTOS

目前 Framework 已經整理執行模型、模組責任、Concurrency、Driver、HAL、BSP 及平台相依性隔離等基礎原則。

不過，這不代表目前已經完成一套可以直接支援所有 Bare-metal 與 RTOS 的統一 OSHAL，也不代表更換 RTOS 時可以完全不修改應用程式。

OSHAL 是這套方法論後續預計透過實際專案驗證的方向之一。

如果應用模組直接呼叫特定 RTOS 的 API，例如：

- FreeRTOS Queue
- ThreadX Message Queue
- Zephyr Message Queue
- 特定 RTOS 的 Mutex、Thread、Event 或 Timer API

RTOS 的差異便可能直接擴散到應用層。

一種可能的隔離方向如下：

```text
Application Modules
        │
        ▼
       OSHAL
        │
        ├──── Bare-metal Adapter
        ├──── FreeRTOS Adapter
        ├──── ThreadX Adapter
        ├──── Zephyr Adapter
        └──── Other RTOS Adapter
```

未來可由實際專案評估 OSHAL 是否需要提供下列共同能力：

- Task 或 Thread
- Queue
- Mutex
- Semaphore
- Event
- Timer
- Delay
- Time
- Critical Section

但實際介面、能力範圍、生命週期、錯誤語意、資源限制及不同 RTOS Adapter，仍需要由後續專案定義、實作與測試。

不同 RTOS 在排程、記憶體、時間精度、同步行為、中斷整合與錯誤模式上可能存在實質差異。

因此，OSHAL 的目標是降低平台差異直接擴散到應用層，而不是保證 RTOS 可以無成本、無風險或無須重新驗證地抽換。

---

# Device 模組化：後續實作與驗證方向

目前 Framework 已提供分層、Adapter、Driver、HAL、BSP、相依方向及平台隔離等基礎原則。

這些原則可以作為 Sensor、Charger IC、Smart Battery、Motor Driver 及其它外部裝置模組化的設計基礎。

不過，目前並未宣稱已經完成一套適用所有專案的統一：

- Sensor Interface
- Charger Interface
- Smart Battery Interface
- Motor Driver Interface
- Capability Model
- Mock Device Standard
- Driver Replacement Compatibility Rule

具體 Device Interface 仍然屬於各專案需要依實際硬體能力、產品需求、錯誤模式與測試策略建立及驗證的實作內容。

實際產品可能需要更換：

- Sensor
- Charger IC
- Smart Battery
- Motor Driver
- ADC
- EEPROM
- Flash
- Display
- Communication Transceiver
- 其它外部 IC 或模組

如果產品邏輯直接依賴特定 IC 的 Register 與 Driver，硬體變更便可能影響大量程式。

因此，可由專案評估透過共同介面與 Adapter，將應用功能和特定元件實作分離。

概念上可以表示為：

```text
Power Management Application
            │
            ▼
     Charger Interface
            │
      ┌─────┴─────┐
      │           │
Charger Driver A  Charger Driver B
```

Sensor 也可以採用類似方向：

```text
Sensing Application
          │
          ▼
    Sensor Interface
          │
    ┌─────┼─────┐
    │     │     │
Sensor A Sensor B Mock Sensor
```

以上圖示只是架構概念，不代表目前 Framework 已經提供可直接使用的標準介面或完整實作。

這類模組化方向希望達成：

- 限制特定 IC 對上層應用的影響
- 支援不同供應商或不同產品型號
- 讓測試程式可以使用 Mock 或替代實作
- 讓新舊模組可以逐步轉移
- 讓不同專案在條件允許時重用共同應用邏輯
- 將硬體差異限制在 Driver、HAL 與 Adapter 層

不過，可抽換不代表所有 Device 都具備相同能力，也不代表任何兩顆 IC 都能直接互換。

不同元件的功能、限制、初始化流程、時序、精度、安全機制與錯誤模式可能不同。

因此，共同介面只能定義真正共通的能力；無法共通的差異，必須被明確保留、建模及驗證，而不能為了表面一致而被隱藏。

---

# 可擴充與可延伸

這套方法論的目標，不是預先猜出所有未來需求。

它比較重視的是，當系統需要增加新功能時，是否能在不破壞既有責任邊界的情況下擴充。

例如：

- 增加新的 Command
- 增加新的 Telemetry
- 增加第二個 MCU
- 從一對一擴充為一對多
- 增加新的 Sensor
- 更換 Charger IC
- 更換 Smart Battery
- 支援另一套 RTOS
- 增加新的 PC UI
- 增加 CLI 或 Python 測試工具
- 增加 Ethernet 或 CAN Bus 通訊介面
- 增加 Logging Backend
- 增加新的自動化測試

理想的擴充方式，不是直接修改所有既有模組，而是：

- 新增明確的介面
- 新增可替換的實作
- 保持既有模組責任
- 更新單一資訊來源
- 補充測試與驗證
- 清楚記錄相容性與限制

因此，「可擴充」不是單純代表可以一直增加程式碼。

它真正代表的是：

> 當需求、平台、RTOS、通訊介面或硬體元件改變時，系統仍然有清楚的位置可以容納這些變動。

---

# 適用的 Host-Device 架構

這套方法論以 Host-Device Control System 作為主要應用與驗證場景。

這裡的 Host，不一定只是一般桌上型電腦。

它可能是：

- Windows 電腦
- Linux 電腦
- 工業電腦
- 嵌入式電腦
- 單板電腦
- SoC 系統
- 閘道器
- 控制器
- 另一顆 MCU

Device 也不一定只是單一 MCU。

它可能是：

- 馬達控制器
- 感測器模組
- 電源控制板
- Charger Module
- Smart Battery
- 遠端 I/O 模組
- 機械設備
- 醫療設備中的子系統
- 其它具有通訊能力的嵌入式裝置

實際架構可能是：

## 一對一

一台 Host 控制一個 Device。

```text
PC ───── MCU
```

## 一對多

一台 Host 控制多個 Device。

```text
              ┌──── MCU A
PC 或 SoC ────┼──── MCU B
              └──── MCU C
```

## 多層式架構

PC 控制 SoC，再由 SoC 控制一個或多個 MCU。

```text
PC ───── SoC ───── MCU
                  ├──── Motor Controller
                  ├──── Sensor Controller
                  └──── Power Controller
```

## 裝置對裝置

MCU 與 MCU 之間交換命令、狀態或資料。

```text
Main MCU ───── Sub MCU
```

## 分散式架構

多個控制節點共同完成系統功能。

```text
Host
  │
  ├──── Control Node A
  ├──── Control Node B
  └──── Control Node C
```

這些架構的技術形式不同，但在工程上仍然會遇到許多共同問題，例如：

- 誰可以發出命令？
- 誰擁有最終控制權？
- 一個命令是否需要回覆？
- 多個裝置的狀態如何同步？
- 哪一端負責逾時判斷？
- 裝置離線後，Host 應該如何處理？
- 多個控制節點同時要求動作時，誰有較高權限？
- 資料是否需要版本管理？
- 發生異常後，系統應該如何恢復？

Host-Device Control 是這套方法論的主要應用背景，但不是它唯一關注的內容。

---

# 不綁定特定通訊介面

Host 與 Device 之間，可以使用很多不同的通訊方式。

例如：

- USB
- UART
- RS-232
- RS-422
- RS-485
- CAN Bus
- Ethernet
- EtherCAT
- Wi-Fi
- Bluetooth Low Energy
- 其它有線或無線通訊介面

不同介面在速度、距離、成本、即時性、抗干擾能力與複雜度上各有差異。

不過，在通訊介面的上層，它們仍然有許多共同問題：

- 一個命令代表什麼？
- 命令由誰發出？
- 接收端如何回覆？
- 是否需要確認訊息？
- 發生逾時時如何處理？
- 資料格式如何版本化？
- 新舊版本能否相容？
- 系統重新連線後如何恢復？
- 哪些資料是即時資料？
- 哪些資料是裝置狀態？
- 哪些資料允許遺失？
- 哪些命令必須保證執行一次？
- 哪些操作具有風險，必須增加保護條件？

因此，這套方法論不把自己限制在單一實體介面，而是嘗試整理這些介面之上的共同工程原則。

---

# 不綁定特定電腦程式語言

Host 端程式可以使用不同語言與框架開發，例如：

- Java
- C++
- Qt
- C#
- WPF
- Python
- 其它桌面、伺服器或嵌入式應用技術

不同產品會因為既有團隊能力、作業系統、效能需求、授權模式、維護年限與開發工具，而選擇不同技術。

這套方法論不試圖決定所有專案都應該使用哪一種程式語言。

它比較關心的是：

- Host 端應該負責什麼？
- UI、通訊層、Protocol 層與產品邏輯是否分離？
- 裝置狀態如何管理？
- 命令送出後如何等待與處理結果？
- 通訊錯誤是否會直接影響使用者介面？
- 測試工具能否獨立於正式應用程式？
- 未來更換 UI 技術時，通訊核心是否可以保留？
- PC 端是否依照共同 Protocol 實作，而不是自行定義？

也就是說，技術選擇可以改變，但工程責任與設計邊界應該盡量保持清楚。

---

# 不綁定特定 MCU 平台

Device 端可能使用不同的微控制器或處理器平台，例如：

- Microchip
- Texas Instruments
- STM32
- NXP
- Renesas
- Nordic Semiconductor
- Espressif
- 其它 MCU 或 SoC 平台

每個平台都有自己的周邊、SDK、驅動程式、開發工具與限制。

但在通訊與系統架構上，仍然會遇到許多相似問題：

- 接收到命令後要如何解析？
- 是否可以直接在中斷中處理？
- 命令要交給哪一個模組？
- 執行中的命令能否被取消？
- 多個命令同時到達時如何排序？
- 系統忙碌時如何回應？
- 裝置錯誤如何回報？
- 通訊資料是否會影響即時控制工作？
- 如何避免通訊程式與產品功能過度耦合？
- 如何避免硬體驅動與通訊協議綁死？
- 如何讓同一套應用邏輯可以在不同 MCU 或 RTOS 上重用？

因此，這套方法論希望把平台相關的程式碼，與較高層的通訊規則、控制流程及系統責任區分開來。

---

# Bare-metal 與 RTOS 都可以使用

Device 韌體可能採用不同的執行架構。

## Bare-metal

對於規模較小、資源有限或控制流程明確的系統，可以採用 Bare-metal 架構，搭配：

- Event-driven
- State Machine
- 非阻塞式處理
- 週期性排程
- 中斷與主迴圈分工

Bare-metal 並不代表程式只能全部寫在一個無限迴圈中。

透過事件驅動與狀態機，同樣可以建立清楚、可測試且具備擴充性的系統。

```text
Interrupt / Driver
        │
        ▼
     Event Queue
        │
        ▼
   State Machine
        │
        ▼
  Application Module
```

這樣可以避免：

- 在中斷中執行複雜工作
- 在主迴圈中大量使用阻塞式等待
- 通訊解析直接控制硬體
- 所有功能互相呼叫，難以追蹤

## RTOS

規模較大的系統，則可能採用即時作業系統，例如：

- FreeRTOS
- Azure RTOS ThreadX
- Zephyr RTOS
- TI-RTOS
- 其它 RTOS

使用 RTOS 之後，仍然需要清楚定義：

- 哪些功能需要獨立 Task
- Task 之間如何傳遞資料
- 哪些資料需要同步保護
- 通訊接收與命令執行如何分離
- 即時控制與非即時工作如何隔離
- Queue、Semaphore 與 Mutex 的責任
- Task 阻塞或資源不足時如何處理
- 優先權反轉與競態條件如何避免

RTOS 可以提供工具，但不會自動帶來良好的系統架構。

因此，這套方法論同時關注 Bare-metal 與 RTOS，並透過 OSHAL 嘗試保留兩者共通的應用層設計概念。

---

# Coding Rules：建立共同的工程基礎

除了系統架構與通訊協議之外，這套方法論也包含程式撰寫規則，也就是 Coding Rules。

Coding Rules 並不只是要求大家使用相同的縮排、括號位置或命名方式。

它更重要的目的，是降低程式碼中模糊、不一致、難以審查與難以維護的部分。

這樣可以讓：

- 不同工程師撰寫的程式
- 不同專案衍生出的程式
- AI 協助產生的程式
- 後續接手人員維護的程式

至少具備共同的工程基礎。

目前主要涵蓋：

- Embedded C Coding Rules
- C# Coding Rules

---

## Embedded C Coding Rules

Embedded C Coding Rules 主要應用在：

- MCU 韌體
- Bare-metal 專案
- RTOS 專案
- 驅動程式
- 通訊模組
- 即時控制程式

規則關注的內容包括：

- 資料型別與數值範圍應明確
- 明確區分 signed 與 unsigned 資料
- 避免隱含型別轉換造成錯誤
- 避免未受控制的動態記憶體配置
- 避免沒有名稱與說明的 Magic Number
- 避免陣列越界
- 避免整數溢位
- 避免未定義行為
- 函式應清楚說明用途、輸入、輸出與限制
- 模組公開介面與內部實作應分離
- 中斷服務程式應保持簡短
- 中斷中不應執行長時間或阻塞式工作
- 通訊接收與命令執行應適當分離
- 狀態切換應透過事件與狀態機管理
- 模組之間的責任及資料擁有權應清楚
- 錯誤不能只被忽略，應有明確處理方式
- 無限迴圈、等待條件及逾時行為應明確
- Hardware Abstraction Layer 與產品邏輯應適當分離
- RTOS Task、Queue、Mutex 與共享資源應有明確規則
- Protocol 定義不應散落在多個 `.c` 與 `.h` 檔案中

這些規則並不要求所有 MCU 都使用完全相同的程式架構。

Microchip、TI、STM32、NXP 或其它 MCU 平台，仍然可以使用各自的驅動程式、SDK 與開發工具。

Coding Rules 所規範的，是平台之上的共同工程原則。

---

## C# Coding Rules

C# Coding Rules 主要應用在：

- PC 應用程式
- WPF 應用程式
- 測試工具
- 通訊工具
- 模擬器
- Protocol 驗證程式
- 自動化測試程式

目前的概念驗證使用 C# 與 WPF，但這些規則不應只侷限於畫面程式。

規則關注的內容包括：

- 命名方式應保持一致
- Class、Interface、Method、Property 與 Event 的責任應清楚
- UI、通訊、Protocol 與應用邏輯應適當分層
- View 不應直接包含大量通訊與裝置控制邏輯
- 通訊層不應依賴特定畫面元件
- Protocol Model 不應與 UI Model 混在一起
- 非同步操作應正確使用 `async` 與 `await`
- 避免使用 `.Wait()` 或 `.Result` 造成執行緒阻塞或死鎖
- 長時間工作不應直接阻塞 UI Thread
- `CancellationToken` 應用於可取消的非同步工作
- 連線、斷線、逾時及重新連線應有明確狀態
- Exception 不應被空的 `catch` 區塊直接忽略
- Exception 應在適當層級被記錄、轉換或回報
- `IDisposable` 資源應被正確釋放
- Serial Port、Stream、Socket、Timer 與背景工作應有明確生命週期
- Event 訂閱與解除訂閱應避免造成記憶體洩漏
- 多執行緒共享資料應有明確同步機制
- UI Binding 更新應在正確執行緒執行
- Nullable Reference Types 應被合理使用
- 公開 API、資料模型及錯誤結果應有清楚定義
- Magic Number、Command ID 與 Error Code 不應散落在程式各處
- Log 應區分 Debug、Information、Warning 與 Error
- 測試工具與正式應用程式應盡量共用相同的 Protocol 定義與核心通訊元件

C# Coding Rules 的目的，不只是讓畫面程式看起來整齊。

更重要的是避免 PC 端程式隨著功能增加，逐漸變成：

- 所有功能都寫在 `MainWindow`
- UI Event Handler 直接控制裝置
- 通訊程式與畫面無法分離
- Serial Port 斷線後無法恢復
- 背景執行緒直接修改畫面
- 多個地方各自解析相同封包
- 測試工具與正式程式使用不同 Protocol
- 發生錯誤時只顯示模糊訊息或直接當機

因此，C# Coding Rules 也會關注：

- 分層
- 非同步
- 多執行緒
- 資源管理
- 錯誤處理
- 可測試性
- 可維護性
- Protocol 一致性

---

## 不同語言，不代表不同的系統事實

Embedded C 與 C# 的語言特性不同，Coding Rules 也不可能完全相同。

例如：

- Embedded C 關注記憶體配置、中斷、數值範圍與即時性
- C# 關注物件生命週期、非同步、執行緒、事件與資源釋放

但是，兩端仍然應該遵守共同的系統原則，例如：

- 模組責任應清楚
- 不應隱藏錯誤
- 不應任意複製共同定義
- 通訊規格應來自單一資訊來源
- 實作應能被測試
- 異常路徑應事先定義
- 修改應留下版本與原因
- AI 產生的程式碼仍需由工程師審查

換句話說：

> C 與 C# 可以使用不同的程式寫法，但不能各自建立不同的系統事實。

例如：

- Command ID
- Message Length
- Byte Order
- Timeout
- Error Code
- Protocol Version
- Telemetry Format

不應分別由 C# 與 Embedded C 程式自行定義。

它們應該共同依據同一份 Protocol 規格。

---

# 單一資訊來源

Host-Device 系統通常同時包含多個程式與多個儲存庫。

例如：

- PC 應用程式
- MCU 韌體
- SoC 軟體
- 測試工具
- 通訊模擬器
- 自動化測試程式
- Protocol 文件

如果每一端都自行保存一份通訊協議，時間久了便很容易出現不同版本。

例如：

- PC 認為 Command ID `0x10` 是啟動命令
- MCU 認為 Command ID `0x10` 是停止命令
- 文件寫資料長度為 16 Bytes
- 實際程式卻傳送 20 Bytes
- 一端已經加入新的錯誤碼
- 另一端仍然使用舊版本
- Excel、Word、程式碼註解及實際實作彼此不同
- 測試程式又維護了第四份定義

這類問題往往要到系統整合或測試階段才會被發現。

因此，這套方法論強調建立 Single Source of Truth，也就是「單一資訊來源」。

---

## 以 Protocol 為例

在目前的概念驗證中：

### [`host-device-control-poc-system`](https://github.com/jkman357/host-device-control-poc-system)

這個儲存庫中的通訊協議文件，是 PC 與 MCU 通訊定義的主要資訊來源。

它應明確定義：

- Protocol Version
- Message Format
- Command ID
- Request 與 Response
- 資料型別
- 資料長度
- Byte Order
- Telemetry
- Stream
- Error Code
- Timeout
- Update Rate
- Compatibility Rule
- Device Capability

PC 端與 MCU 端都必須依照這份定義實作，而不是各自在自己的儲存庫中重新發明一套規格。

```text
                 單一 Protocol 定義
                         │
            ┌────────────┴────────────┐
            │                         │
        PC App 實作                MCU FW 實作
            │                         │
            └────────────互相通訊─────┘
```

這樣可以讓：

- PC 工程師知道應該依據哪一份規格
- MCU 工程師知道應該依據哪一份規格
- 測試人員知道預期結果來自哪裡
- AI 知道產生程式碼時應該遵循哪一份定義
- CI 工具知道應該檢查哪些內容

---

## Protocol 變更流程

當通訊協議需要修改時，原則上應依照以下順序進行：

1. 先修改共同的 Protocol 定義
2. 說明修改原因
3. 評估影響範圍
4. 更新 Protocol Version
5. 完成必要的 Review
6. 更新 PC 端實作
7. 更新 MCU 端實作
8. 更新模擬器與測試工具
9. 執行相容性與整合測試
10. 確認雙方仍能正確通訊

不應先修改 PC 或 MCU 程式碼，再回頭要求另一端配合。

---

## 衍生檔案不等於新的權威來源

實際開發中，可能會從 Protocol 定義產生：

- C Header
- C Enum
- C# Class
- C# Enum
- Message Parser
- Serializer
- Deserializer
- 測試資料
- 文件表格
- API 文件
- 模擬器資料模型

這些檔案可以由工具自動產生，也可以由工程師依照規格實作。

但它們都屬於衍生內容，不應反過來取代原始 Protocol 定義。

當衍生檔案與共同規格不一致時，應先確認共同規格是否需要正式修改，而不是直接把某一端目前的程式碼視為正確答案。

---

## 單一資訊來源不只適用於 Protocol

相同概念也可以使用在其它工程內容，例如：

- Command 定義
- Error Code
- 裝置能力清單
- 系統狀態
- 設定參數
- 版本相容性
- 硬體介面定義
- Coding Rules
- Architecture Patterns
- Concurrency Rules
- Logging Rules
- Testing Rules
- 專案自行建立並指定為權威來源的 OSHAL Interface
- 專案自行建立並指定為權威來源的 Device Interface
- 架構責任邊界
- 測試預期結果
- Release 條件

但單一資訊來源不代表所有資料都必須放在同一個巨大檔案中。

真正的重點是：

> 每一項重要工程資訊，都應該有一個明確且具權威性的來源。

其它文件、程式碼或測試內容可以引用它、實作它或由它產生，但不應存在多份彼此競爭，卻沒有人知道哪一份才正確的定義。

---

## 單一資訊來源不代表內容一定正確

單一資訊來源只能解決「大家應該看哪一份」的問題，不能保證那份內容本身一定正確。

Protocol、Coding Rules 或 Architecture Guide 仍然可能存在：

- 定義錯誤
- 未考慮的使用情境
- 不合理的逾時設定
- 不完整的錯誤處理
- 不適合實際硬體的資料頻率
- 新舊版本相容性問題
- 不清楚的責任邊界
- 過度抽象或抽象不足

因此，共同規格仍然需要由工程師進行：

- 需求確認
- 架構審查
- 風險評估
- 實作驗證
- 硬體測試
- 版本管理
- 變更核准

單一資訊來源的目的，是讓問題能在同一個地方被看見、討論與修正，而不是讓錯誤分散在不同程式與文件之中。

---

# 自動檢查與人工審查

部分 Coding Rules、Protocol 規則與儲存庫規則，可以透過工具或 CI 自動檢查。

Embedded C 可以搭配：

- Compiler Warning
- Static Analysis
- Coding Rule Validator
- Unit Test
- Integration Test
- Hardware Test

C# 可以搭配：

- .NET Compiler Warning
- Roslyn Analyzer
- `.editorconfig`
- StyleCop 或其它 Analyzer
- Unit Test
- Integration Test
- CI Build
- Protocol Compatibility Test

Protocol 與文件可以檢查：

- Message ID 是否重複
- 必要欄位是否缺少
- 資料長度是否一致
- Request 與 Response 是否配對
- Telemetry 與 Stream 定義是否完整
- Schema 是否符合規則
- 文件是否引用正確來源
- 儲存庫結構是否符合要求
- 不同模組是否違反責任邊界
- 若專案已正式定義 OSHAL 或 Device Interface，可依該專案的介面契約與檢查規則確認必要定義是否完整

不過，自動檢查只能找出已經明確定義的問題。

它無法完整判斷：

- 需求是否合理
- 架構是否適合產品
- 抽象層是否過度設計
- 通訊頻率是否超出硬體能力
- UI 行為是否符合實際使用需求
- 多執行緒設計是否涵蓋所有使用情境
- 系統是否能長時間穩定運作
- 發生硬體異常時是否安全
- 產品是否已經達到可以發布的程度

因此，Coding Rules、CI 與自動檢查是工程品質的基本防線，而不是品質保證。

最終仍然需要工程師進行：

- 設計審查
- 程式碼審查
- 整合測試
- 實際硬體驗證
- Release 判斷

---

# 如何使用 Framework、Project Template 與 AI

[`host-device-control-framework`](https://github.com/jkman357/host-device-control-framework) 與 [`host-device-control-project-template`](https://github.com/jkman357/host-device-control-project-template) 的主要用途之一，是提供 AI 可閱讀、可引用並可持續遵循的工程上下文。

這兩個儲存庫不是要讓 AI 取代工程師，也不是要讓每一個聊天室各自建立一套工程規則。

它們的目的，是讓不同專案與不同 AI 聊天室，可以從清楚且可追蹤的共同來源開始工作。

兩者的分工如下：

- `host-device-control-framework`：提供跨專案的方法論、工程規則、權威文件索引與設計約束
- `host-device-control-project-template`：提供建立新專案儲存庫時使用的骨架、AI 入口、專案輸入欄位與紀錄結構
- 複製後的專案儲存庫：保存該專案的實際需求、限制、Framework 基線、Protocol、設計、決策、風險、測試計畫與紀錄
- AI 專案與聊天室：讀取同一個專案儲存庫，並在被指定的工作範圍內協助分析、草擬、實作與檢查

原始 Project Template 是建立專案時使用的來源。

專案建立後，AI 的主要工作入口應該是**複製後的專案儲存庫**，而不是持續把原始 Template 當成該專案的事實來源。

---

## 建議使用流程

當要開始一個新的 Host-Device Control 專案時，可以依照以下方式進行：

1. 使用 `host-device-control-project-template` 建立新的專案儲存庫。
2. 檢查新專案儲存庫中的 `LICENSE` 與 `NOTICE.md`，確認所有權、公開方式與散布條件。
3. 由授權人員填寫 `FRAMEWORK_REFERENCE.md` 與 `PROJECT_INPUT.md`。
4. 對尚未確認的必要項目，明確使用 `TBD`、`Unknown`、`None` 或 `N/A`，不要留白，也不要要求 AI 自行猜測。
5. 在提供內容給 AI 之前，確認資料可以被揭露，並移除或保護機密、個人、受管制、出口管制、憑證及未獲授權的第三方內容。
6. 為該設計案建立獨立的 AI 專案或工作區。
7. 依工作範圍建立聊天室，例如：
   - 系統架構
   - Coordinator／PC App
   - Node／MCU Firmware
   - Hardware／Datasheet Analysis
   - Protocol／Integration
   - Verification and Validation
8. 將**新建立的專案儲存庫**連結提供給 AI。
9. 要求每一個聊天室先閱讀專案中的 `START_HERE.md`，並只處理該聊天室被指定的工作範圍。
10. AI 依 `START_HERE.md` 的順序，讀取：
    - `FRAMEWORK_REFERENCE.md`
    - `PROJECT_INPUT.md`
    - `docs/Approval_Records.md`
    - `docs/Decision_Log.md`
    - 與本次工作範圍相關的專案文件
    - 適用的上游 Framework 文件
11. 後續所有聊天室都以同一個專案儲存庫為共同來源，並將需要保留的需求、決策、風險、文件與變更寫回專案中。
12. 所有 AI 產出都必須由適當的人員 Review；AI 不得自行建立核准、風險接受、測試通過、Evidence 接受、Framework Conformance 或 Release 結論。

概念上可以表示為：

```text
host-device-control-framework
    跨專案工程方法、規則與權威文件
                    │
                    │ 由 FRAMEWORK_REFERENCE.md 指定來源與適用性
                    ▼
由 Project Template 建立的專案儲存庫
    專案事實、需求、Protocol、設計、決策、
    Framework 基線、風險、測試與紀錄
                    │
                    │ 每個聊天室先讀 START_HERE.md
                    ▼
AI Project / Chats
    只處理被指定的工作範圍
```

---

## Framework 版本與基線控制

Framework 的 GitHub `main` 可以作為最新內容的檢視來源，但「最新版本」不應自動取代專案目前採用的 Framework 基線。

專案應透過 `FRAMEWORK_REFERENCE.md` 記錄：

- 上游 Framework 儲存庫
- 供參考檢視的 Branch
- 擬採用或已採用的完整 Commit SHA
- 存取日期
- Framework 各領域的適用性
- 前一版 Framework 基線
- 基線或適用性變更的影響評估來源

在 Framework 尚未 Pin 到完整 Commit SHA、尚未完成適用性判定，或尚未由有效的外部核准紀錄確認前：

- AI 可以將目前的 `main` 作為諮詢性參考
- AI 必須明確說明來源尚未成為有效的專案 Framework 基線
- 不應宣稱專案已符合 Framework
- 不應以未鎖定的來源建立正式核准、Deviation 或 Conformance Claim

當 Framework 或 Project Template 更新時，可以要求 AI 閱讀最新版並進行：

- 差異分析
- 適用性分析
- 對需求、架構、Protocol、實作、風險、測試、Evidence、Release 與既有核准的影響分析

但是：

> 最新版不得自動取代專案目前記錄及核准的 Framework 基線。

是否採用新版，仍需由授權人員決定。

確認採用後，才可以：

1. 更新 `FRAMEWORK_REFERENCE.md`
2. 建立或更新變更影響評估
3. 更新受影響的專案文件與實作
4. 建立新的內容基線
5. 完成必要的 Review 與核准紀錄
6. 重新確認受影響的測試、Evidence、Claim 與 Release 狀態

Framework 基線或適用性變更後，既有的下游核准不應被預設為仍然有效。

受影響項目需要新的基線與紀錄，或由適當授權人員留下「不受影響」的明確結論與理由。

---

## 建議起始提示詞

建立專案儲存庫後，可以在新的 AI 聊天室使用以下提示詞。

```text
以下是本專案的 GitHub 儲存庫：

<PROJECT_REPOSITORY_URL>

接下來的討論將以這個專案儲存庫所指定的專案事實、
Framework 基線、權威來源與文件責任為基礎。

請先依照下列順序閱讀：

1. START_HERE.md
2. FRAMEWORK_REFERENCE.md
3. PROJECT_INPUT.md
4. docs/Approval_Records.md
5. docs/Decision_Log.md
6. 與本次工作範圍相關的專案文件
7. 依 FRAMEWORK_REFERENCE.md 指向及判定適用的上游 Framework 文件

本次工作範圍是：

<REQUESTED_SCOPE>

請遵守下列要求：

1. 先說明你實際讀取的來源、目前 Framework 是否已 Pin、
   Framework 適用性狀態，以及任何存取限制。
2. 區分下列內容：
   - 外部強制義務
   - 已核准的專案需求、限制與決策
   - 有效的 Framework 基線與適用規則
   - Project Template 的骨架或提示內容
   - 專案 Draft
   - AI 建議
   - 尚待人員決定的項目
3. 不要自行補造尚未提供的產品需求、硬體能力、Protocol、
   Authority、Approval、Risk Acceptance、測試結果或 Evidence。
4. 對未知或尚未決定的項目，明確使用 Unknown、TBD、None 或 N/A。
5. 只處理本次指定的工作範圍，不要自行生成整個系統。
6. 提出設計、文件、程式碼或測試建議時，說明依據的來源、
   Framework 規則、專案事實、假設與限制。
7. 若專案目前只以 Framework main 作為諮詢性參考，
   請明確說明它尚未成為有效的 pinned project baseline。
8. 不要自動把較新的 Framework 或 Template 套用到本專案。
   若發現新版，先提供差異與影響分析，等待授權人員決定。
9. 保留由適當人員執行需求確認、架構決策、風險判斷、
   Review、實體測試、Evidence 接受、核准與 Release 的責任。

完成閱讀後，先摘要：

- 目前專案狀態
- 使用的來源與基線
- 本次適用的權威文件
- 重要未知事項、衝突與 Authority 缺口
- 本次工作範圍內的下一個具體行動

不要在第一次回覆中自行產生整個系統。
```

---

## 後續討論的責任邊界

AI 在後續討論中可以協助：

- 整理需求、限制與未知事項
- 依專案文件結構草擬內容
- 依適用的 Framework 規則提出架構與模組建議
- 草擬 Protocol、程式碼與測試
- 檢查文件、程式碼、Protocol 與規則的一致性
- 找出可能的缺口、衝突、風險與未處理情境
- 比較新版 Framework 或 Template 對既有專案的可能影響

但 AI 不應：

- 把本 Overview 當成正式權威來源
- 把原始 Project Template 當成已建立專案的事實來源
- 把範例、概念圖或 Draft 誤認為已核准的專案設計
- 自行創造產品需求、硬體能力、安全需求或 Authority
- 自行把 Framework `main` 宣稱為已核准的專案基線
- 自動將專案切換到較新的 Framework 或 Template
- 宣稱尚未執行的測試已經通過
- 宣稱未被接受的 Evidence 已經有效
- 取代工程師的風險判斷、設計審查、硬體驗證、核准或發布決策

Framework 提供的是跨專案工程方法與規則。

Project Template 提供的是建立新專案儲存庫的骨架。

複製後的專案儲存庫，才是 AI 進行後續專案工作的主要入口與專案資訊來源。

真正的產品需求、Project Protocol、硬體限制、風險控制、實作決策、客觀測試證據、核准與 Release 紀錄，仍然必須在個別專案中由適當人員建立、確認及維護。

---

# 目前的儲存庫

整個專案目前分成：

1. 方法論與樣板
2. 概念驗證

---

# 方法論與樣板

## [`host-device-control-framework`](https://github.com/jkman357/host-device-control-framework)

這是整套方法論的源頭儲存庫。

內容主要包括：

- Software Engineering Rules
- Architecture Patterns
- Concurrency Guide
- Logging Guide
- Testing Guide
- 架構設計指引
- 通訊協議定義方式
- Host 與 Device 的責任邊界
- 訊息與命令設計規則
- 錯誤處理原則
- Embedded C Coding Rules
- C# Coding Rules
- 與 OSHAL 相關的執行環境及平台隔離原則
- Device 模組化與抽象化的基礎原則
- 文件範本
- 自動檢查工具
- AI 輔助工程的使用原則
- 單一資訊來源的治理原則

這個儲存庫不直接代表某一項產品的完整程式碼。

它比較像是一套工程規則、設計依據、權威文件索引與共同語言，用來協助後續專案建立一致的基礎。

這個儲存庫的主要讀者之一是 AI。AI 應先讀取 GitHub `main`、`authority-registry.yaml` 及其所指向的適用文件，再依文件狀態與權威範圍協助後續專案工作。

---

## [`host-device-control-project-template`](https://github.com/jkman357/host-device-control-project-template)

這是依照 Framework 建立的專案樣板。

它提供一個可以開始填寫與實作的基本結構，讓新專案不需要每次都從空白資料夾開始。

樣板中的內容需要由實際專案的工程師依需求填寫。

如果某個項目不適用，也應該明確標示為：

- None
- Not applicable
- 無
- 不適用

重點不是把所有表格填滿，而是避免重要工程判斷只存在於少數人的腦中，最後沒有留下可追蹤的設計依據。

這個儲存庫也是提供給 AI 使用的專案起始骨架。AI 可以依 Framework 規則協助建立及填寫專案內容，但不得自行把未知需求、硬體能力、風險控制或測試結果補成既定事實。

---

# 概念驗證

為了確認這套方法不只停留在文件層級，目前也建立了一組最小可運作的概念驗證專案。

---

## [`host-device-control-poc-system`](https://github.com/jkman357/host-device-control-poc-system)

這個儲存庫負責定義 PC 與 MCU 之間的通訊協議。

它是雙方共同遵循的單一資訊來源。

內容可能包括：

- 訊息格式
- Command ID
- Request 與 Response
- Telemetry 資料
- Stream 資料
- 錯誤碼
- 資料型別
- 更新頻率
- 逾時條件
- 版本資訊
- 相容性規則

這樣可以避免 PC 程式與 MCU 韌體各自解讀、各自實作，最後才發現雙方定義不同。

---

## [`host-device-control-poc-pc-app`](https://github.com/jkman357/host-device-control-poc-pc-app)

這是概念驗證中的 PC 端應用程式。

目前使用：

- C#
- WPF

它負責：

- 與 MCU 建立通訊
- 發送控制命令
- 接收裝置資料
- 顯示即時資訊
- 驗證通訊協議是否能被實際使用

這個專案的目的，不是展示最華麗的使用者介面。

它主要用來驗證：

- C# 程式的分層方式
- UI 與通訊是否分離
- Protocol 是否能被正確實作
- 非同步通訊是否穩定
- 斷線與錯誤處理是否合理
- C# Coding Rules 是否能落地使用
- Logging 與 Testing 是否能支援實際除錯與驗證

---

## [`host-device-control-poc-stm32f446re-fw`](https://github.com/jkman357/host-device-control-poc-stm32f446re-fw)

這是概念驗證中的 MCU 韌體。

目前使用：

- STM32F446RE
- Bare-metal
- Event-driven
- State Machine

它負責：

- 接收 PC 命令
- 解析通訊訊息
- 執行對應操作
- 回傳結果與狀態
- 週期性傳送測試資料

這個專案主要用來驗證：

- Embedded C Coding Rules
- Event-driven 架構
- State Machine 架構
- 通訊接收與命令執行分離
- PC 與 MCU 是否能依照同一份 Protocol 實作
- 未來 OSHAL 與 Device Module 抽象是否能逐步落地

目前的概念驗證刻意使用相對單純的硬體與功能，讓重點集中在架構、通訊流程與工程規則，而不是特定產品的複雜功能。

---

# AI 在這個專案中的角色

這套方法論是透過人與 AI 多次討論後逐步整理出來的。

AI 在其中可以協助：

- 將零散經驗整理成文件
- 找出規則中的矛盾
- 檢查責任邊界是否清楚
- 產生初步程式碼
- 建立測試案例
- 檢查文件與程式碼的一致性
- 找出可能被忽略的錯誤路徑
- 協助比較不同設計方案
- 根據 Protocol 產生 C 或 C# 初步實作
- 協助建立 CI 與自動檢查工具
- 協助檢查 OSHAL、Device Interface 與模組邊界

但 AI 並不是最後的工程決策者。

實際專案仍然需要由工程師負責：

- 定義需求
- 確認風險
- 選擇技術
- 審查設計
- 驗證程式碼
- 進行硬體測試
- 判斷抽象層是否合理
- 判斷產品是否可以發布
- 承擔最終工程責任

AI 可以加快整理、生成與檢查的速度，但不能取代真實世界中的工程判斷與驗證。

---

# 目前的限制

這套方法論目前仍處於實驗階段。

雖然已經建立或開始建立：

- Software Engineering Rules
- Architecture Patterns
- Concurrency Guide
- Logging Guide
- Testing Guide
- Coding Rules
- 專案樣板
- 自動檢查工具
- Protocol 定義
- PC 端概念驗證
- MCU 端概念驗證
- OSHAL 與 Device 模組化方向

但這並不代表它已經適用於所有產品或所有產業。

一套工程方法是否成熟，不能只看文件是否完整，也不能只看一個 Demo 是否能執行。

它還需要經過不同類型的實際專案驗證，例如：

- 不同 MCU 平台
- 不同 RTOS
- 不同電腦程式語言
- 不同通訊介面
- 一對多裝置架構
- 高速資料傳輸
- 即時控制
- 長時間運作
- 斷線與重新連線
- 韌體升級
- 多版本相容
- Sensor、Charger、Smart Battery 等 Device 替換
- 功能安全
- 資訊安全
- 工業產品
- 醫療產品
- 法規與品質系統要求

這些驗證需要大量：

- 時間
- 設備
- 人力
- 實際專案
- 測試資源
- 資金

因此，目前比較適合把這個專案視為：

> 一套正在形成中的工程方法論，以及一組用來驗證這套方法是否可行的公開實驗。

它不是產品認證，也不是安全保證，更不是任何專案可以直接套用而不需審查的標準答案。

---

# 為什麼仍然值得做？

即使它還不成熟，將這些經驗整理出來仍然有價值。

因為許多工程專案失敗，未必是工程師不會寫程式，而是：

- 一開始沒有說清楚雙方責任
- 通訊協議只存在於程式碼中
- PC 與 MCU 各自維護不同規格
- 文件與實作互相矛盾
- RTOS 與應用邏輯過度綁定
- Device Driver 與產品功能混在一起
- 更換 Sensor、Charger 或 Smart Battery 時需要大幅重寫
- 錯誤情境沒有事先定義
- Logging 無法重建問題
- 測試只驗證正常流程
- 更換人員後，原本的設計理由消失
- Coding Rules 只有口頭要求
- AI 產生大量程式碼，卻沒有人知道應該如何審查
- 專案直到整合階段，才開始處理架構問題

這個專案希望做的，是把這些問題往前移。

在真正開始大量寫程式之前，先把重要的問題提出來、寫下來、討論清楚，再逐步進入實作與驗證。

即使最後證明某些規則不夠好，或某些設計不適合實際產品，這些結果本身也會成為方法論持續修正的一部分。

---

# Copyright 與使用邊界

Copyright © 2026 Ray Yang. All rights reserved.

## No License Granted

除非特定檔案另有明確聲明，或著作權人另以書面授權，否則本儲存庫不授予重製、修改、散布、發布、再授權、銷售、商業使用或將原始內容納入其它專案的權利。

內容公開放置於 GitHub，不代表授予使用、修改、散布或商業化權限。

## Personal Engineering Project Disclaimer

本儲存庫是個人工程研究、方法論整理與概念驗證介紹。

內容不能取代：

- 產品需求
- 專業工程判斷
- 合格人員審查
- 實體硬體驗證
- 資訊安全評估
- 法律意見
- 法規核准
- 產品認證
- 正式第三方標準

## No Employer or Company Representation

本儲存庫不代表任何現任或過往雇主、客戶、合作夥伴、供應商或其它公司與組織的正式政策、規格、設計、Coding Standards、意見、核准、採用、認證或背書。

不應僅因作者或貢獻者與某組織具有現在或過去的關係，而推定該組織已採用、核准、贊助、認證或支持本儲存庫。

## AI Assistance and Human Responsibility

生成式 AI 可能被用於文件草擬、重整、翻譯、一致性檢查、程式實作支援、測試準備與產出物生成。

AI 協助不會轉移工程權限或責任。

需求確認、來源查證、工程判斷、Review、實體測試、客觀驗證、核准與最終責任，仍然必須由適當的人員承擔。

---

# 結語

這不是一套已經完成的標準。

它比較像是一場長期的工程實驗。

目前整套內容可以概括為：

1. Software Engineering Rules
2. Architecture Patterns
3. Embedded C 與 C# Coding Rules
4. Concurrency、Logging 與 Testing Guides
5. 單一且具權威性的資訊來源
6. OSHAL 與跨 RTOS 抽象的後續實作與驗證方向
7. Device 模組化與可抽換設計的後續實作與驗證方向
8. 可擴充、可延伸的系統架構
9. 實際可執行的 Host-Device 概念驗證

這場實驗試著回答一個問題：

> 能不能把工程師多年累積、難以言傳的實務經驗，整理成一套人類與 AI 都能理解、使用、檢查、實作並持續改善的工程方法？

目前還沒有最終答案。

但透過文件、樣板、Coding Rules、Architecture Patterns、OSHAL 與 Device 模組化方向、單一資訊來源、概念驗證與實際程式碼，至少可以一步一步把原本只存在腦中的經驗，轉換成可以被閱讀、被質疑、被實作，也能被後續工程專案檢驗的內容。
