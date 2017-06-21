# 依賴（Dependencies） {#dependencies}

當類別A裡面使用到類別B，像是A裡面的某個函式必須用到類別B時，我們說類別 A 依賴類別 B。

## 好的與壞的依賴關係 {#bad-and-good-dependencies}

依賴關係有兩種度量方向：

* 內聚（cohesion）
* 耦合（coupling）

簡單的說，內聚代表功能相關的類別互相依賴的程度。

耦合與內聚的觀念是相對的，代表功能並不相關的類別或模組之間互相依賴的程度。

在軟體開發上，為了程式的可讀性、擴展性……等，會建議程式碼有高的內聚度，以及低的耦合度。所以我們應該盡可能把相關的功能放進一個模組（這邊所說的模組不專指 Yii 模組，也不是某個類別，而是一種邏輯概念）。在這個模組裡面，可以不用過度抽象化，直接使用相依的程式類別運作。

而被這個模組所使用，但是並不屬於該模組的其他類別，則不應該直接使用。而應該透過介面（interface）以降低耦合度。

透過介面，模組可以得到自己運作所需要的東西，並且不和模組外的類別互相依賴。

## 達成低耦合 {#achieving-low-coupling}

我們不可能完全消除程式之間的依賴，但是我們可以盡可能使程式碼的彈性提高。

## 控制反轉 {#inversion-of-control}

控制反轉（inversion of control，IoC）的原則，是用來降低程式之間的耦合度。

## 依賴注入 {#dependency-injection}

依賴注入（Dependency injection，DI）

## 依賴注入容器 {#dependency-container}

Injecting basic dependencies is simple and easy. You're choosing a place where you don't care about dependencies, which is usually controller which you aren't going to unit-test ever, create instances of dependencies needed and pass these to dependent classes.

It works well when there aren't many dependencies overall and when there are no nested dependencies. When there are many and each dependency has dependencies itself, instantiating the whole hierarchy becomes tedious process which requires lots of code and may lead to hard to debug mistakes.

Additionally, lots of dependencies, such as certain third party API wrapper, are the same for any class using it. So it makes sense to:

* Define how to instantiate such API wrapper once.
* Instantiate it when required and only once per request.

That's what dependency containers are for.

See [官方教學](http://www.yiiframework.com/doc-2.0/guide-concept-di-container.html) for more information about Yii's dependency container.

