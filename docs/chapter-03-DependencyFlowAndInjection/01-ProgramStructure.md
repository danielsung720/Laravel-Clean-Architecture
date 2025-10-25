# Program Structure

## 概述

指的是整個程式碼的組織與分層方式，包括 `目錄架構`、`檔案配置`、`類別職責分配`、以及各層之間的 `依賴方向`<br>
良好的結構能提高維護性、可測試性與擴充性

---

## 常見的程式結構類型

### 傳統三層架構 - Layered Architecture

適合中小型專案或業務流程明確的系統。

#### 目錄結構範例

```bash
app/
  Http/
    Controllers/
      UserController.php
  Services/
    UserService.php
  Repositories/
    UserRepository.php   # 直接具體類別
  Models/
    User.php             # Eloquent Model
```

#### 特徵

* **Controller**：處理 HTTP Request / Response
* **Service**：執行商業邏輯與流程控制
* **Repository**：封裝資料存取邏輯
* **Model**：通常為 ORM (如 Eloquent) 對應的資料表物件

#### ✅ 優點

* 易於理解與快速開發
* 與 Laravel 框架緊密整合 (Eloquent、DI 支援完善)

#### ❌ 缺點

* Service 容易變成 **God Class**
* Repository 容易變成單純的資料夾分類，而非具領域語意的模組
* 商業邏輯分散、難以重用

---

### Use Case / Application Layer 架構

適合中大型產品開發，強調每個 `用例 (Use Case)` 對應一個明確的業務動作

#### 目錄結構範例

```bash
app/
  Http/
    Controllers/
      UserController.php
    Requests/
      RegisterRequest.php
    Resources/
      UserResource.php
  Application/
    Users/
      UseCases/
        RegisterUserUseCase.php
      DTO/
        RegisterUserRequestDTO.php
        RegisterUserResponseDTO.php
  Domain/
    Users/
      Entities/
        User.php                    # 純 Domain 物件（可選）
      Repositories/
        UserRepository.php          # 介面（Port）★ 重點
      Services/
        PasswordPolicy.php          # 可選：Domain Service
      ValueObjects/
        Email.php                   # 可選：Value Object
  Infrastructure/
    Persistence/
      Eloquent/
        UserRepository.php          # 實作（Adapter）
        Models/
          User.php                  # Eloquent Model
  Providers/
    RepositoryServiceProvider.php   # 綁定介面 → 實作
```

#### 特徵

* 每個 `Use Case` 對應一個具體業務動作 (例如：`CreateOrder`、`CancelOrder`)
* 將商業邏輯封裝成 `應用流程`，避免集中於 Service
* Domain 層僅關注 `業務規則`，不處理流程控制

#### ✅ 優點

* 高可測試性
* 明確區分 **業務邏輯** 與 **應用流程**

#### 💡 補充

此架構通常作為從三層架構過渡到 Clean Architecture 的中間型態，開發者能逐步抽離邏輯、建立更明確的邊界

---

### Domain-Driven Design（DDD）

常見於大型企業或長期維護的產品 (例如: 金融系統、SaaS、交易系統)

#### 目錄結構範例

```bash
app/
  Domain/
    Orders/
      Aggregates/
        Order.php
      Repositories/
        OrderRepository.php       # 介面（屬於 Domain）
      Services/
        DomainServiceXxx.php
      ValueObjects/
        OrderId.php
  Application/
    Orders/
      UseCases/
        CreateOrderUseCase.php
      DTO/
        CreateOrderCommand.php
  Infrastructure/
    Persistence/
      Eloquent/
        Orders/
          EloquentOrderRepository.php   # 實作（Adapter）
          Models/
            OrderModel.php
```

#### 特徵

* Repository 介面屬於 Domain 層的一部分
* 商業規則主要封裝在 **Entities** 或 **Domain Services** 中
* Application 層 `Use Cases` 只負責組裝與協調流程

#### ✅ 優點

* 高維護性與擴充彈性
* 天然支援 Clean Architecture 的 `內向依賴`

#### ❌ 缺點

* 初期設計成本高 (需建立大量抽象介面)
* 適合業務複雜、領域穩定的長期產品

---

## 總結比較

| 架構類型 | 特徵摘要 | 優點 | 缺點 | 適用場景 |
| --- | --- | --- | --- | --- |
| **三層架構** | Controller → Service → Repository | 簡單易懂、開發快 | 容易耦合、邏輯分散 | 中小型專案 |
| **Use Case 架構** | 每個 `UseCase` 對應一個業務動作 | 可測試性高、邏輯清晰 | 類別數量多 | 中大型系統 |
| **DDD 架構** | 以 Domain 為核心、明確邊界 | 高維護性、支援 Clean 原則 | 初期成本高 | 複雜長期產品 |

---

## 結論

程式結構不是一成不變的規則，而是一種「分工哲學」

* **三層架構**：講求開發速度與實用性
* **Use Case 架構**：注重應用層邏輯的明確邊界
* **DDD 架構**：追求長期演化與可維護性

> 最佳實踐不是立即套用最高階架構，而是根據專案規模與團隊成熟度，循序演進。

> 但我在職涯工作以來還未曾見過 Use Case 跟 DDD 架構專案...🤔
