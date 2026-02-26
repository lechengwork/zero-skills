# Error Code Patterns

本文件說明 walletServer 錯誤碼體系與 `WalletError` 的使用方式。

## 錯誤碼範圍

| 範圍 | 類型 | 可重試 | 說明 |
|------|------|--------|------|
| `1xxx` | 業務錯誤 | ❌ | 客戶端錯誤，如餘額不足、參數無效 |
| `2xxx` | 系統錯誤 | ✅ | 服務端錯誤，如超時、DB、Redis 錯誤 |
| `3xxx` | 狀態錯誤 | ❌ | 狀態流轉錯誤，如交易已完成 |
| `9xxx` | 未知錯誤 | ❌ | 未分類錯誤 |

## 常用錯誤碼

### 業務錯誤（1xxx）

| 錯誤碼 | 常量 | 說明 |
|--------|------|------|
| 1001 | `ErrCodeInsufficientBalance` | 餘額不足 |
| 1002 | `ErrCodePlayerNotFound` | 玩家不存在 |
| 1003 | `ErrCodeDuplicateRequest` | 重複請求（冪等成功） |
| 1004 | `ErrCodeTxNotFound` | 交易不存在 |
| 1005 | `ErrCodeInvalidAmount` | 金額無效（≤ 0 或格式錯誤） |
| 1006 | `ErrCodeInvalidParams` | 參數無效 |

### 系統錯誤（2xxx）

| 錯誤碼 | 常量 | 說明 |
|--------|------|------|
| 2001 | `ErrCodeSystemBusy` | 系統繁忙（Redis 鎖衝突） |
| 2002 | `ErrCodeApiTimeout` | 外部 API 超時 |
| 2003 | `ErrCodeApiError` | 外部 API 回傳錯誤 |
| 2004 | `ErrCodeDatabaseError` | DB 查詢或連線失敗 |
| 2005 | `ErrCodeRedisError` | Redis 操作失敗 |

### 狀態錯誤（3xxx）

| 錯誤碼 | 常量 | 說明 |
|--------|------|------|
| 3001 | `ErrCodeInvalidState` | 不允許的狀態流轉 |
| 3002 | `ErrCodeTxAlreadySettled` | 交易已結算（重複 Settle） |
| 3003 | `ErrCodeTxAlreadyReleased` | 交易已退還（重複 Release） |
| 3004 | `ErrCodeReserveNotSuccess` | Reserve 未成功（Settle 時狀態不對） |
| 3005 | `ErrCodeReservePending` | Reserve 仍在處理中 |

## WalletError 使用方式

```go
// 使用預定義錯誤
err := common.ErrInsufficientBalance

// 建立新錯誤
err := common.NewWalletError(
    common.ErrCodeInvalidAmount,
    "金額必須大於 0",
    nil,
)

// 包裝原始錯誤（保留 cause）
err := common.WrapError(
    common.ErrCodeDatabaseError,
    "查詢玩家失敗",
    originalErr,
)
```

## 判斷函數

```go
// 判斷是否可重試（2xxx）
common.IsRetryableError(err)

// 判斷是否為冪等成功（1003 DuplicateRequest、1004 TxNotFound）
common.IsIdempotentSuccess(err)
```

## 在 go-zero 中回傳錯誤

walletServer 為 gRPC 服務，錯誤直接回傳 `WalletError`，呼叫方（gameServer）需處理：

```go
// 呼叫方判斷是否重試
if common.IsRetryableError(err) {
    // 可重試，例如系統繁忙、超時
}

// 判斷是否冪等成功（視為正常）
if common.IsIdempotentSuccess(err) {
    // 重複請求，已處理過，視為成功
}
```

## 相關檔案

- 錯誤碼定義：`services/walletServer/internal/common/`
- 完整錯誤碼指南：`services/walletServer/internal/common/ERROR_CODE_GUIDE.md`
