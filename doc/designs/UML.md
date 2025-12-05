# UML 設計文件

## 1. 使用案例圖 (Use Case Diagram)

對應 User Stories (US-01 ~ US-06) 與功能需求 (FR-01 ~ FR-17)。

```mermaid
usecaseDiagram
    actor Player as "玩家 (Player)"
    actor System as "系統 (System)"

    package "蜜袋鼯飼養模擬" {
        usecase "建立角色 (FR-01)" as UC1
        usecase "餵食 (FR-02)" as UC2
        usecase "清潔籠子 (FR-03)" as UC3
        usecase "互動/撫摸 (FR-04, FR-15)" as UC4
        usecase "查看日誌 (FR-05)" as UC5
        usecase "購買物品 (FR-12, FR-13)" as UC6
        usecase "處理醫療事件 (FR-14)" as UC7
        usecase "查看回顧日誌 (FR-11)" as UC8
    }

    Player --> UC1
    Player --> UC2
    Player --> UC3
    Player --> UC4
    Player --> UC5
    Player --> UC6
    Player --> UC7
    Player --> UC8

    UC1 ..> System : 生成初始屬性
    UC2 ..> System : 更新飢餓/快樂/健康
    UC3 ..> System : 更新清潔度
    UC4 ..> System : 更新親密度/快樂
    UC6 ..> System : 扣除金錢
    UC7 ..> System : 扣除金錢/恢復健康
    UC8 ..> System : 角色死亡時生成
```

## 2. 類別圖 (Class Diagram)

對應 `SystemOverview.md` 中的 3.1 系統模組圖與 3.2 模組功能說明。

```mermaid
classDiagram
    class GameManager {
        +StartGame()
        +Update()
        +EndGame()
    }

    class StatsManager {
        +float Hunger
        +float Cleanliness
        +float Happiness
        +float Health
        +UpdateStats()
        +ModifyHealth(amount)
    }

    class LifeCycleManager {
        +int AgeDays
        +enum LifeStage
        +UpdateAge()
        +CheckStage()
    }

    class InteractionManager {
        +Feed(FoodType)
        +Clean()
        +Interact(InteractionType)
    }

    class EconomyManager {
        +int Money
        +BuyItem(Item)
        +DeductMoney(amount)
    }

    class EventManager {
        +TriggerEvent(EventType)
        +CheckRandomEvents()
    }

    class UIManager {
        +UpdateDisplay()
        +ShowAlert(message)
    }

    GameManager --> StatsManager
    GameManager --> LifeCycleManager
    GameManager --> InteractionManager
    GameManager --> EconomyManager
    GameManager --> EventManager
    GameManager --> UIManager

    InteractionManager ..> StatsManager : 修改數值
    InteractionManager ..> EconomyManager : 消耗金錢
    LifeCycleManager ..> StatsManager : 影響健康上限
    EventManager ..> StatsManager : 影響健康/情緒
```

## 3. 活動圖 (Activity Diagram)

### 3.1 玩家餵食流程 (FR-02)

```mermaid
flowchart TD
    Start([玩家選擇餵食]) --> SelectFood[選擇食物類型]
    SelectFood --> CheckFood{食物是否適合?}
    
    CheckFood -- 正確食物 --> UpdateHunger[飢餓值增加]
    UpdateHunger --> UpdateHappiness[快樂值增加]
    UpdateHappiness --> UpdateHealth[健康值正常變化]
    UpdateHealth --> DeductMoney[扣除食物成本]
    DeductMoney --> ShowSuccess[顯示餵食成功]
    ShowSuccess --> End([結束])
    
    CheckFood -- 錯誤食物 --> DamageHealth[健康值下降]
    DamageHealth --> ShowWarning[顯示警告訊息]
    ShowWarning --> End
    
    CheckFood -- 過量餵食 --> OverfeedWarning[顯示肥胖風險]
    OverfeedWarning --> UpdateStats[更新數值但健康受影響]
    UpdateStats --> End
```

### 3.2 玩家清潔流程 (FR-03)

```mermaid
flowchart TD
    Start([玩家點擊清潔操作]) --> CheckCleanliness{檢查當前清潔度}
    CheckCleanliness -- 已乾淨 --> ShowMsg[顯示無需清潔]
    ShowMsg --> End([結束])
    
    CheckCleanliness -- 需要清潔 --> PerformClean[執行清潔動作]
    PerformClean --> UpdateCleanStat[清潔度提升]
    UpdateCleanStat --> RemoveSicknessRisk[降低疾病機率]
    RemoveSicknessRisk --> ShowCleanSuccess[顯示清潔完成]
    ShowCleanSuccess --> End
```

### 3.3 互動流程 (FR-04)

```mermaid
flowchart TD
    Start([玩家選擇互動]) --> SelectInteraction{選擇互動方式}
    SelectInteraction -- 撫摸 --> TouchAction[執行撫摸]
    SelectInteraction -- 遊戲 --> PlayAction[執行遊戲]
    SelectInteraction -- 聲音互動 --> VoiceAction[執行聲音互動]
    
    TouchAction --> IncreaseHappiness[快樂值上升]
    PlayAction --> IncreaseHappiness
    VoiceAction --> IncreaseHappiness
    
    IncreaseHappiness --> UpdateIntimacy[更新親密度]
    UpdateIntimacy --> ShowReaction[顯示角色反應動畫/音效]
    ShowReaction --> RecordDaily[記錄每日互動次數]
    RecordDaily --> End([結束])
```

## 4. 循序圖 (Sequence Diagram)

### 4.1 互動與回饋流程 (FR-15, FR-16)

```mermaid
sequenceDiagram
    participant Player as 玩家
    participant UI as 使用者介面
    participant InteractMgr as 互動行為模組
    participant StatsMgr as 狀態管理模組
    participant AIPet as 蜜袋鼯 AI
    participant EconomyMgr as 經濟系統模組
    
    Player->>UI: 選擇餵食操作
    UI->>InteractMgr: 傳送餵食請求(食物類型)
    InteractMgr->>InteractMgr: 驗證食物是否適合
    
    alt 正確食物
        InteractMgr->>StatsMgr: 更新飢餓值(+20)
        InteractMgr->>StatsMgr: 更新快樂值(+10)
        InteractMgr->>StatsMgr: 更新健康值(+5)
        StatsMgr->>AIPet: 通知狀態變化
        AIPet->>UI: 返回正向回饋(動畫/音效)
        InteractMgr->>EconomyMgr: 扣除食物成本
        EconomyMgr->>UI: 更新餘額顯示
        UI->>Player: 顯示餵食成功
    else 錯誤食物
        InteractMgr->>StatsMgr: 健康值下降(-15)
        StatsMgr->>AIPet: 通知狀態惡化
        AIPet->>UI: 返回生病/拒絕反應
        UI->>Player: 顯示警告訊息
    end
```

## 5. 狀態圖 (State Machine Diagram)

### 5.1 生命週期狀態 (FR-07, FR-08, FR-10)

```mermaid
stateDiagram-v2
    [*] --> 幼年階段
    
    幼年階段: 0-30 天
    幼年階段: 食量小
    幼年階段: 互動需求高
    
    幼年階段 --> 青年階段: 達到30天
    
    青年階段: 31-90 天
    青年階段: 食量增加
    青年階段: 活力充沛
    
    青年階段 --> 成年階段: 達到90天
    
    成年階段: 91-300 天
    成年階段: 食量大
    成年階段: 行為穩定
    
    成年階段 --> 老年階段: 達到300天
    
    老年階段: 301+ 天
    老年階段: 食量減少
    老年階段: 容易生病
    
    老年階段 --> 死亡: 健康值歸零或自然壽終
    
    死亡 --> [*]
```

### 5.2 健康狀態變化 (FR-06, FR-14, FR-17)

```mermaid
stateDiagram-v2
    [*] --> 健康狀態
    
    健康狀態: 健康值 80-100
    健康狀態: 正常活動
    
    健康狀態 --> 亞健康狀態: 飢餓/清潔度低/缺乏互動
    健康狀態 --> 健康狀態: 正常照顧
    
    亞健康狀態: 健康值 50-79
    亞健康狀態: 輕微症狀
    
    亞健康狀態 --> 健康狀態: 恢復正常照顧
    亞健康狀態 --> 生病狀態: 持續忽略/餵錯食物
    
    生病狀態: 健康值 20-49
    生病狀態: 需要醫療
    
    生病狀態 --> 亞健康狀態: 醫療處理+改善照顧
    生病狀態 --> 危急狀態: 未及時醫療
    
    危急狀態: 健康值 1-19
    危急狀態: 生命危險
    
    危急狀態 --> 生病狀態: 緊急醫療成功
    危急狀態 --> 死亡: 健康值歸零
    
    死亡: 健康值 = 0
    死亡 --> [*]
```
