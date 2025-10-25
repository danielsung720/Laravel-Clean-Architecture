# Domain Driven Design

## 概述
將業務邏輯作為設計的中心，所有的設計都圍繞著業務邏輯進行。資料只是「被儲存的狀態」，不是邏輯的中心，也不應該決定程式結構

亦可稱作: **Use-Case-Driven Design**

## 範例
以 [02-DatabaseDrivenDesign.md](./02-DatabaseDrivenDesign.md) 的訂單系統為例，Domain Driven Design 的流程會變成：

### 1. Domain Layer
```php
class Order
{
    public function __construct(
        private int $userId,
        private float $totalPrice,
        private string $status = 'pending'
    ) {}

    public function confirm(): void
    {
        if ($this->status !== 'pending') {
            throw new DomainException('Order already confirmed.');
        }
        $this->status = 'confirmed';
    }
}
```

### 2. Repository Interface
```php
interface OrderRepositoryInterface
{
    public function save(Order $order): void;
    public function findById(int $id): ?Order;
}
```

### 3. Application Layer
```php
class CreateOrderUseCase
{
    public function __construct(private OrderRepositoryInterface $repository) {}

    public function execute(CreateOrderInput $input): Order
    {
        $order = new Order($input->userId, $input->totalPrice);
        $this->repository->save($order);
        return $order;
    }
}
```

### 4. Infrastructure Layer
負責資料庫實作
```php
class EloquentOrderRepository implements OrderRepository
{
    public function save(Order $order): void
    {
        OrderModel::updateOrCreate(
            [
                'id' => $order->getId()
            ],
            [
                'user_id' => $order->getUserId(),
                'total_price' => $order->getTotalPrice()
            ]
        );
    }

    public function findById(int $id): ?Order
    {
        $model = OrderModel::find($id);

        return $model
            ? new Order(
                $model->user_id,
                $model->total_price,
                $model->status
            ) : null;
    }
}
```

### 5. Presentation Layer
```php
class OrderController
{
    public function __construct(private CreateOrderUseCase $useCase) {}

    public function createOrder(CreateOrderRequest $request)
    {
        $input = new CreateOrderInput($request->userId, $request->totalPrice);
        $order = $this->useCase->execute($input);
        return new OrderResource($order);
    }
}
```

### 特性
- Domain Layer 不知道資料庫的存在
- Infrastructure Layer 才知道如何與資料庫互動
- Presentation Layer 只負責調用 UseCase，不直接操作資料表

## 與 Database Driven Design 的差異
| 層級    | 資料庫驅動設計                              | Clean Architecture                               |
| ----- | ------------------------------------ | ------------------------------------------------ |
| 出發點  | 資料表怎麼設計 → Service 寫業務              | 業務行為是什麼 → 資料要怎麼支援                     |
| 重心    | CRUD、欄位、關聯         | 規則、流程、行為                     |
| 依賴方向 | Service 依賴 Eloquent / DB 結構          | Repository 介面依賴 Domain 規格             |
| 實作方式 | Service 裡面直接呼叫 Model 或 Query Builder | Service (或 UseCase) 呼叫 Repository 介面，Domain 決定資料需求 |
| 耦合度  | 高 (Model 綁 DB)     | 低 (抽象介面)                      |
| 擴充性  | 差 (改資料庫即重寫)         | 強 (可換 Repository 實作)          |

## 結論
只要從開始設計的時候，不以資料庫為優先處理，而是以商業邏輯的角度出發撰寫程式，自然就會先產生抽象介面，而不是先實作資料庫