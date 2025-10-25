# Dependency Injection

## 概述

想像你開了一家咖啡店。每天你都需要咖啡豆，但如果你每天都得自己去買豆、自己烘焙、自己研磨，那時間就被綁死在這個流程裡。這樣的設計，就像程式中直接在類別內 `new` 出 `依賴物件`，
同時這也違反了 [DIP](../chapter-02-DependencyInversionPrinciple/02-DependencyInversionPrinciple.md) 原則。

而所謂的 **依賴注入(Dependency Injection, DI)**，就像你請了一家供應商——他每天把烘好的豆子送來，你只要「要求」這個供應商提供產品，不需要知道背後怎麼烘的。換句話說：

> **你不再自己建立依賴，而是把依賴交由外部提供。**

在程式設計中，這個外部供應商可以是：

* Laravel 的 `Service Container`
* DI Framework (如 `Spring`、`Guice`)
* 或手動在程式中傳遞物件 (`Constructor Injection`)

### 範例：未使用 DI 的寫法

```php
class CoffeeMaker
{
    private $beanProvider;

    public function __construct()
    {
        // 直接在內部 new 出依賴
        $this->beanProvider = new BeanProvider();
    }

    public function brew(): string
    {
        return 'Brewing coffee using ' . $this->beanProvider->getBeans();
    }
}
```

**問題：**

* `CoffeeMaker` 依賴於具體的 `BeanProvider`
* 無法替換為不同的供應商 (例如 `Mock`、`Fake` 或 `其他實作方式`)
* 難以測試，也難以擴充

### 範例：使用 DI 的寫法

```php
interface BeanProviderInterface
{
    public function getBeans(): string;
}

class PremiumBeanProvider implements BeanProviderInterface
{
    public function getBeans(): string
    {
        return 'premium beans';
    }
}

class CoffeeMaker
{
    public function __construct(
        private BeanProviderInterface $provider
    ) {}

    public function brew(): string
    {
        return 'Brewing coffee using ' . $this->provider->getBeans();
    }
}

// 綁定關係
$provider = new PremiumBeanProvider();
$coffeeMaker = new CoffeeMaker($provider);

echo $coffeeMaker->brew(); // Brewing coffee using premium beans
```

這樣一來，`CoffeeMaker` 不再關心「豆子哪裡來」，只關心「如何使用豆子」。

在 Laravel 裡，這種綁定通常由 `Service Provider` 自動完成：

```php
$this->app->bind(BeanProviderInterface::class, PremiumBeanProvider::class);
```

---

## 優點

### **降低耦合 (Loose Coupling)**

程式模組之間只依賴 `抽象介面`，不依賴 `具體實作`，任何一方變動時，影響範圍小

> ✅ CoffeeMaker 只依賴 BeanProviderInterface，而不是 PremiumBeanProvider

### **提高可測試性 (Testability)**

可以輕鬆替換為假的依賴 (`Fake` 或 `Mock`) 來進行單元測試

```php
class FakeBeanProvider implements BeanProviderInterface
{
    public function getBeans(): string
    {
        return 'fake beans';
    }
}

$coffeeMaker = new CoffeeMaker(new FakeBeanProvider());
$this->assertEquals('Brewing coffee using fake beans', $coffeeMaker->brew());
```

### **提升擴充性與彈性**

想換供應商？只需改綁定，不用改內部程式。

```php
$this->app->bind(BeanProviderInterface::class, LocalFarmBeanProvider::class);
```

### **支援多環境配置**

在不同環境（dev/test/prod）可注入不同實作，例如：

* 本地開發 → 使用 Fake Service
* 正式環境 → 使用真實外部 API Client

### **落實依賴反轉原則 (Dependency Inversion Principle)**

DI 是 **DIP 原則的實踐手段**。

> 高層模組 `CoffeeMaker` 不應依賴低層模組 `BeanProvider`；兩者都應依賴抽象 `BeanProviderInterface`

---

## 結語

依賴注入不是多此一舉的技巧，而是一種**設計上的哲學**：

> 「讓類別專注於自己的行為，而不是外部資源的建立。」

無論是開咖啡店還是開發後端系統，只要懂得讓外部提供資源、讓 `類別` 只專注在 `行為` 上，就能寫出更具彈性、可維護的程式架構
