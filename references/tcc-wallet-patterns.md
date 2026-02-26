# TCC Wallet Patterns

本文件說明 walletServer TCC 分布式事務的設計規範，適用於 Reserve / Settle / Release 相關開發。

## 概覽

walletServer 採用 TCC（Try-Confirm-Cancel）模式處理遊戲錢包交易：

| 操作 | 時機 | 說明 |
|------|------|------|
| **Reserve** | 遊戲開始 | 預扣玩家餘額（Try） |
| **Settle** | 遊戲結束 | 確認最終輸贏金額（Confirm） |
| **Release** | 遊戲取消 | 退還預扣金額（Cancel） |

## 錢包策略模式

根據設定檔中的 `WalletType` 決定實作策略：

```
WalletType=1 → NormalWalletStrategy  → 本地 DB 事務
WalletType=2 → CompanyWalletStrategy → 外部 API Platform RPC
```

兩種策略實作相同的 interface，邏輯層不感知差異。

## Reserve 流程

```
1. 取得分布式鎖（lock:wallet:{plyGuid}，TTL 5 秒）
2. 冪等性檢查（roundID + plyGuid + tradeType 是否已存在）
3. 查詢玩家餘額，檢查是否足夠
4. 扣款 + 建立交易紀錄（狀態：PENDING）

NormalWallet：  
  └── 本地 DB 事務，一次性完成，狀態直接設為 SUCCESS

CompanyWallet：
  ├── 本地 DB 建立 PENDING 交易
  ├── 建立 Outbox 任務（呼叫外部 API）
  └── Outbox Worker 非同步執行，成功後狀態更新為 SUCCESS
```

## Settle 流程

```
1. 取得分布式鎖
2. 查詢對應的 Reserve 交易
3. 驗證 Reserve 狀態必須為 SUCCESS
4. 計算淨輸贏 = Settle 金額 - Reserve 金額
5. 更新玩家餘額 + 建立 Settle 交易紀錄
6. 清除 Redis round_data 緩存
```

## Release 流程

```
1. 取得分布式鎖
2. 查詢對應的 Reserve 交易
3. 驗證 Reserve 尚未結算或退還
4. 退還預扣金額至玩家帳戶
5. 更新交易狀態為 RELEASED
6. 清除 Redis round_data 緩存
```

## Outbox Pattern（CompanyWallet 專用）

確保本地 DB 寫入與外部 API 呼叫的最終一致性。

```
本地事務：寫入 PENDING 交易 + 寫入 Outbox 任務
Outbox Worker（背景）：
  ├── 批量輪詢待處理任務（500 個/批次，10 個 Worker 并發）
  ├── 呼叫外部 API Platform
  ├── 成功 → 更新交易狀態為 SUCCESS，Outbox 標記 COMPLETED
  └── 失敗 → 指數退避重試（2^n 分鐘，最多 5 次）
              超過上限 → 進入死信隊列（Dead Letter Queue）
```

## 分布式鎖

保護同一玩家的並發交易：

```go
lockKey := fmt.Sprintf("lock:wallet:%s", plyGuid)
// TTL: 5 秒
// 基於 Redis SETNX 實作
```

## 冪等性設計

| 操作 | 冪等鍵 | 重複請求回傳 |
|------|--------|------------|
| Reserve | `roundID + plyGuid + tradeType` | `ErrCodeDuplicateRequest`（1003） |
| Settle | 對應 Reserve 交易 ID | `ErrCodeTxAlreadySettled`（3002） |
| Release | 對應 Reserve 交易 ID | `ErrCodeTxAlreadyReleased`（3003） |

## 狀態流轉

### 交易狀態（DW_Status）
```
PENDING → SUCCESS
        → FAIL
```

### 完整事務狀態（TxStatus，CompanyWallet）
```
INIT → LOCAL_RESERVED → API_PENDING → API_SUCCESS → COMMITTED
                                     → API_FAILURE → COMPENSATING
```

## Recovery Job

每 10 分鐘執行，處理異常情況：
- 掃描超過 1 小時仍為 PENDING 的交易
- 向外部 API 查詢實際狀態並同步

## 相關檔案

- 流程架構圖：`docs/WALLET_SERVER_ARCHITECTURE.md`
- 業務邏輯：`services/walletServer/internal/logic/`
- 策略實作：`services/walletServer/internal/logic/*_strategy_*.go`
- Outbox Worker：`services/walletServer/task/`
- 錯誤碼定義：`error-code-patterns.md`
