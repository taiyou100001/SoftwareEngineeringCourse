# 細部系統架構

- 健康系統
	- HP
	- 疾病
	- 營養
- 情緒系統
	- 快樂
	- 壓力
- 互動系統
	- 摸摸
	- 玩耍
- 時間系統
	- 晝夜
	- 年齡轉換
- 事件系統
	- 生病
	- 逃跑
	- 過度壓力

## 遊戲邏輯流程
1. 餵食 → 更新健康值 → 更新情緒 → 回饋動畫
2. 清潔 → 更新衛生狀態 → 減少疾病機率

## 架構圖（Mermaid）
```mermaid
graph TD
		A[健康系統] --> A1(HP)
		A --> A2(疾病)
		A --> A3(營養)
		B[情緒系統] --> B1(快樂)
		B --> B2(壓力)
		C[互動系統] --> C1(摸摸)
		C --> C2(玩耍)
		D[時間系統] --> D1(晝夜)
		D --> D2(年齡轉換)
		E[事件系統] --> E1(生病)
		E --> E2(逃跑)
		E --> E3(過度壓力)
		F[遊戲邏輯流程] --> F1(餵食→健康→情緒→動畫)
		F --> F2(清潔→衛生→疾病機率)
```

## 資料結構設計
- pet_status (健康、飢餓、清潔、情緒、年齡)
- inventory (食物、玩具)
- event_log (事件紀錄)

## 資料結構圖（Mermaid ERD）
```mermaid
erDiagram
		PET_STATUS {
				int HP
				int Hunger
				int Cleanliness
				int Emotion
				int Age
		}
		INVENTORY {
				string Food
				string Toy
		}
		EVENT_LOG {
				string Event
				datetime Time
		}
		PET_STATUS ||--o{ EVENT_LOG : has
		PET_STATUS ||--o{ INVENTORY : owns
```
