# GORM Gen Patterns

本文件說明使用 GORM Gen 工具生成 Model / Query 層的規範，適用於含複合主鍵資料表的專案。

## 為何使用 GORM Gen

`goctl model` 假設每張表有單一主鍵，遇到複合主鍵會產生錯誤的生成結果。
GORM Gen 直接從資料庫 schema 反推，能正確處理複合主鍵、decimal 型別映射等情況。

## 生成工具

工具位於 `common/models/tool/gen_gorm.go`：

```bash
# 修改 DSN 後執行（在 common/models/tool/ 目錄下）
cd common/models/tool && go run gen_gorm.go
```

生成結果：
- `common/models/model/*.gen.go` — 純資料結構（struct）
- `common/models/query/*.gen.go` — 型別安全的 CRUD 查詢方法

### 新增資料表

在 `gen_gorm.go` 的 `g.ApplyBasic(...)` 中加入後重新執行：

```go
g.ApplyBasic(
    g.GenerateModel("existing_table"),
    g.GenerateModel("new_table"),   // 新增這行
)
```

## Query 介面使用方式

### 初始化

```go
import "domino/common/models/query"

// 初始化（通常在 ServiceContext）
q := query.Use(db)
```

### 單筆查詢

```go
// 依主鍵查詢
record, err := q.GrCardgame.Where(q.GrCardgame.GRSEQ.Eq(seq)).First()

// 依條件查詢
player, err := q.MemPlayer.Where(q.MemPlayer.PLYGUID.Eq(guid)).First()
```

### 多筆查詢

```go
// 查詢多筆
records, err := q.GrCardgame.
    Where(q.GrCardgame.PLYGUID.Eq(guid)).
    Order(q.GrCardgame.GRCreateDateTime.Desc()).
    Limit(100).
    Find()

// 帶分頁
records, total, err := q.GrCardgame.
    Where(q.GrCardgame.PLYGUID.Eq(guid)).
    FindByPage(offset, limit)
```

### 新增

```go
err := q.GrCardgame.Create(&model.GrCardgame{
    PLYGUID:    guid,
    GMGameCode: gameCode,
    GRBets:     betAmount,
})
```

### 更新

```go
// 更新指定欄位
_, err := q.MemPlayer.
    Where(q.MemPlayer.PLYGUID.Eq(guid)).
    Update(q.MemPlayer.PLYPoint, newBalance)

// 更新多個欄位
_, err := q.MemPlayer.
    Where(q.MemPlayer.PLYGUID.Eq(guid)).
    Updates(map[string]interface{}{
        "PLY_Point":    newBalance,
        "PLY_UpdateDT": time.Now(),
    })
```

### 事務

```go
err := q.Transaction(func(tx *query.Query) error {
    // 扣款
    _, err := tx.MemPlayer.
        Where(tx.MemPlayer.PLYGUID.Eq(guid)).
        Update(tx.MemPlayer.PLYPoint, newBalance)
    if err != nil {
        return err
    }

    // 建立交易紀錄
    return tx.DtWallettrade.Create(&model.DtWallettrade{
        PLYGUID:   guid,
        DWAmount:  amount,
    })
})
```

## 目前已生成的資料表

| Query 物件 | 資料表 | 說明 |
|-----------|--------|------|
| `q.GrCardgame` | `gr_cardgame` | 遊戲記錄 |
| `q.GsPlayerProfile` | `gs_player_profile` | 玩家個人資訊 |
| `q.GsEnvRtp` | `gs_env_rtp` | 環境 RTP |
| `q.DtWallettrade` | `dt_wallettrade` | 資金流水 |
| `q.MemPlayer` | `mem_player` | 玩家主表 |
| `q.SysGameImgSetting` | `sys_game_img_setting` | 遊戲圖片設定 |
| `q.SysAgentImgSetting` | `sys_agent_img_setting` | 代理圖片設定 |

## 相關檔案

- 生成工具：`common/models/tool/gen_gorm.go`
- Model 定義：`common/models/model/*.gen.go`
- Query 方法：`common/models/query/*.gen.go`
