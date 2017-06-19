# SOLID 原則（SOLID） {#solid}

SOLID is a set of principles that you should follow if you want to get pure object oriented code which is easy to test and extend.

這些原則為：

* [單一功能](https://yii2-cookbook.readthedocs.io/solid/#single-responsibility)（Single responsibility）
* [開閉原則](https://yii2-cookbook.readthedocs.io/solid/#open-closed)（Open-closed）
* [里氏替換](https://yii2-cookbook.readthedocs.io/solid/#liskov-substitution)（Liskov substitution）
* [接口隔離](https://yii2-cookbook.readthedocs.io/solid/#interface-segregation)（Interface segregation）
* [依賴反轉](https://yii2-cookbook.readthedocs.io/solid/#dependency-inversion)（Dependency inversion）

我們來看看這些原則的意義。

## 單一功能 {#single-responsibility}

一個類別（class）應該只負責一項任務。

## 開閉原則 {#open-closed}

一個類別或者模組（module，許多相關類別的集合）應該要隱藏實做的細節，但是有一個完整定義的界面，讓其他類別可以透過public 函式使用其功能，並可以透過繼承public 和 protected 函式擴充其功能。

或者換個說法，對於擴展是開放的，但是對於修改是封閉的。

## 里氏替換 {#liskov-substitution}

里氏替換原則（Liskov substitution，LSP），如果以傳統方式定義，是SOLID原則裡面最複雜的一個。不過事實上這原則並沒有這麼複雜。

> 傳統定義：令 Φ\(x\) 為 T 種類的某物件物件 x，其某個為真的性質。那麼對於T的亞種S來說，S種類的某物件y，也應該使Φ\(y\)為真。  
> （Let Φ\(x\) be a property provable about objects x of type T. Then Φ\(y\) should be true for objects y of type S where S is a subtype of T.）

這原則是關於類別的層次（hierarchy）與繼承（inheritance）。當你建立一個類別，這個類別繼承了父類別時。該類別應該與父類別有相同的界面，並且在同樣的狀態下有相同的行為。所以原則上，可以將父類別的物件替換成該類別的，而程式的行為不會有改變。

更多資訊可以參考 [StackOverflow上面的不同回答](http://stackoverflow.com/questions/56860/what-is-the-liskov-substitution-principle)。

## 接口隔離 {#interface-segregation}

介面隔離原則（interface segregation principles， ISP） points that an interface should not define more functionality that is actually used at the same time. It is like single responsibiltiy but for interfaces. In other words: if an interface does too much, break it into multiple more focused interfaces.

## 依賴反轉 {#dependency-inversion}

依賴反轉原則（Dependency inversion principle，DIP）基本上是說類別應該透過界面（interface）取得需要的東西，而不是自己與其他類別接觸。

換個說法，如果原本有一個類別A，需要依賴類別B來實做。則改成界面a，令類別A實做界面a，並

更多資訊可以參考[依賴的教學](/dependencies.md)。

