# Database Driven Design

## 概述
是一種很常見、也最直覺的開發方式，它將資料庫作為設計的中心，所有的設計都圍繞著資料庫進行

開發者的流程通常是
```
先根據需求畫出 ERD (Entity Relationship Diagram)
↓
定義資料表與欄位
↓
建立 Eloquent Model 對應資料表
↓
然後直接在 Controller 或 Service 裡操作 Model 進行 CRUD
```

聽起來很自然，但這其實是一種 「由資料結構主導」 的設計方式，也就是說：「資料表怎麼長，程式就怎麼寫」。

## 核心原則
- **資料庫優先**：先設計資料表結構，再設計應用程式邏輯
- **資料表驅動**：程式邏輯直接對應資料表欄位和關聯
- **CRUD 導向**：主要操作圍繞著資料的增刪改查

## 範例
以 `訂單系統` 為例，資料庫驅動設計大概會長這樣：

### 1. 建立資料表
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    total_price DECIMAL(10, 2),
    status VARCHAR(20),
    created_at DATETIME,
    updated_at DATETIME
);
```

### 2. 建立 Eloquent Model
```php
class Order extends Model {
    protected $fillable = ['user_id', 'total_price', 'status'];
}
```

### 3. 在 Controller 或 Service 裡操作 Model 進行 CRUD
```php
class OrderService
{
    public function createOrder(array $data): Order
    {
        // 驗證商業邏輯
        if ($data['total'] < 0) {
            throw new Exception('Invalid total');
        }

        // 直接操作 Eloquent（依賴資料庫實作）
        return Order::create([
            'user_id' => $data['user_id'],
            'total' => $data['total'],
            'status' => 'pending',
        ]);
    }
}
```

表面上這樣開發很快，但潛藏著一個問題：
- 所有商業邏輯都 `黏在資料結構上`
- 程式的設計被資料庫綁死，Service 改不了資料來源（MySQL → Redis 就要重寫）

## 主要問題
| 問題        | 說明                           | 典型現象                   |
| --------- | ---------------------------- | ---------------------- |
| 🧱 高耦合    | 程式直接依賴資料表欄位、命名、型別            | 改欄位名稱整個 Controller 跑不動 |
| 🌀 商業邏輯分散 | 商業規則被寫在 Controller 或 Model 內 | `if` / `else` 到處都是     |
| 🚫 難測試    | 單元測試必須連資料庫才能跑                | 測試慢又不穩定                |
| 🔄 難擴充    | 想改用 Redis 或外部 API，幾乎得重寫      | Repository 無法抽換        |
| ❌ 缺乏抽象    | 無法在系統層面清楚表達「商業規則」            | 系統只會 CRUD，沒有「行為」       |

總結: Database Driven Design 的 **Service** 是業務邏輯寄生在資料結構上，無法抽換資料來源，也無法在系統層面清楚表達「商業規則」

因此在 Clean Architecture 中，更強調的設計方式應該是 [Domain Driven Design](./03-DomainDrivenDesign.md)
