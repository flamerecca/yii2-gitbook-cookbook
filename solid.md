# SOLID 原則（SOLID） {#solid}

如果你希望你的程式能盡可能物件導向，讓程式的測試跟擴展比較簡單的話，建議能在設計時盡量讓程式遵守 SOLID 原則。

這些原則為：

* [單一功能](https://yii2-cookbook.readthedocs.io/solid/#single-responsibility)（Single responsibility）
* [開閉原則](https://yii2-cookbook.readthedocs.io/solid/#open-closed)（Open-closed）
* [里氏替換](https://yii2-cookbook.readthedocs.io/solid/#liskov-substitution)（Liskov substitution）
* [介面隔離](https://yii2-cookbook.readthedocs.io/solid/#interface-segregation)（Interface segregation）
* [依賴反轉](https://yii2-cookbook.readthedocs.io/solid/#dependency-inversion)（Dependency inversion）

我們來看看這些原則的意義。

## 單一功能 {#single-responsibility}

一個類別（class）應該只負責一項任務。

## 開閉原則 {#open-closed}

一個類別或者模組（module，許多相關類別的集合）應該要隱藏實做的細節，但是有一個完整定義的界面，讓其他類別可以透過public 函式使用其功能，並可以透過繼承public 和 protected 函式擴充其功能。

或者換個說法，對於擴展是開放的，但是對於修改是封閉的。

## 里氏替換 {#liskov-substitution}

里氏替換原則（Liskov substitution，LSP），如果以傳統方式定義，是SOLID原則裡面最複雜的一個。不過事實上這原則並沒有這麼複雜。

這原則是關於類別的層次（hierarchy）與繼承（inheritance）。當你建立一個類別，這個類別繼承了父類別時。該類別應該與父類別有相同的界面，並且在同樣的狀態下有相同的行為。所以原則上，可以將父類別的物件替換成該類別的，而程式的行為不會有改變。

> 傳統定義：令 Φ\(x\) 為某個性質，遇到 T 種類的某物件 x 其值為真。那麼對於 T 的亞種 S 來說，S 種類的某物件 y，也應該會使Φ\(y\) 為真。  
> （Let Φ\(x\) be a property provable about objects x of type T. Then Φ\(y\) should be true for objects y of type S where S is a subtype of T.）

更多資訊可以參考 [StackOverflow上面的不同回答](http://stackoverflow.com/questions/56860/what-is-the-liskov-substitution-principle)。

## 介面隔離 {#interface-segregation}

介面隔離原則（interface segregation principles， ISP） points that an interface should not define more functionality that is actually used at the same time. 這有點像是給介面的單一功能原則。

換個說法，如果一個介面有很多任務，應該拆解成多個更簡潔的小介面。

## 依賴反轉 {#dependency-inversion}

依賴反轉原則（Dependency inversion principle，DIP）基本上是說類別應該透過界面（interface）取得需要的東西，而不是自己與其他類別接觸。

換個說法，如果原本有一個類別A，需要在程式碼內使用類別B，這會讓A類別無法在不改程式碼的狀況下離開類別B。也就是說，類別A依賴B。這樣的

應該加入界面a，令類別A依賴介面a，類別B則是介面a的實做。這樣我們可以在程式運行時選擇是否要使用類別B。這樣程式的擴充彈性就變大了。

另外，這樣會反轉原本高階類別一向需要低階類別才能實做的關係，變成低階類別需要實做高階介面，所以稱為依賴反轉。

更多資訊可以參考[依賴的教學](/dependencies.md)。

