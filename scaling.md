# Configuring a Yii2 Application for an Multiple Servers Stack {#configuring-a-yii2-application-for-an-multiple-servers-stack}

這邊的教學集中在如何讓  Yii2 建立的程式能夠無狀態（stateless ）化。 無狀態的意思是每一個主機不儲存狀態資料，每台主機的內容幾乎都是一樣的。程式不會在主機實體上面儲存資料。 這種架構擴展的彈性度，比起傳統的架構更高。目前這邊所說的架構比較簡單，更加複雜的架構會在未來的篇章說明。

在網頁開發上，一般來說擴展代表的是水平擴展（horizontal scaling），也就是加入更多的伺服器來處理更多流量。可以手動增添伺服器，或者，再大多數平台，可以自動增添伺服器。在自動增減（autoscaling）的環境之下，平台可以自動偵測變多的流量，並主動增添伺服器。也可以自動偵測流量變少，主動減少伺服器，以節省不必要的開銷。

要建立一個可擴展的應用，該應用必須要是無狀態的，換句話說，沒有資料應該直接寫入主機，所以沒有本地端的 session 或 cache 。所有的 session，cache，以及資料庫必須放在獨立的伺服器上面。

設置一個能自動擴展的 Yii2 服務相當直覺。下面我們來說明怎麼做。

## 先決條件 {#prerequisites}

* 任何完善的平台即服務（Platform as a Service，PaaS）系統

  * 一個完整的 PaaS 系統會支援自動縮放（autoscaling）、負載平衡（load balancing）以及 SQL 資料庫等功能。像是 Google 雲端平台，就有自己的 Instance Group 和 Load Balancer。Amazon Web Services，或者 AWS， 也有AutoScaling Group 和 Elastic Load Balancer。

* 專用的 Redis 或者 Memcached 伺服器

  * 使用 [Bitnami Cloud](https://bitnami.com/cloud) 可以簡單的發布在多數主流的PaaS服務上，Redis 一般來說運作的比 Memcached 要好，所以這邊只講使用 Redis 的作法。

* 專用的資料庫

  * 大多 PaaS 服務都可以簡單的使用資料庫，比方說 Google SQL 或者 AWS Relational Database Service。

## 使應用無狀態（stateless）化 {#making-your-application-stateless}

使用 Yii2 支援的 Session Storage/Caching/Logging 服務，像是 Redis。更多細節可以參考下面的資料：

* [Yii2 類別 yii\redis\Session](http://www.yiiframework.com/doc-2.0/yii-redis-session.html)
* [Yii2 類別 yii\redis\Cache](http://www.yiiframework.com/doc-2.0/yii-redis-cache.html)
* [Yii2 Redis Logging 元件](https://github.com/JackyChan/yii2-redis-log)

當我們使用 PaaS 平台，要注意是使用 Redis 伺服器的內部 IP 而不是外部 IP。這對應用的速度非常重要。

redis 伺服器主要是在 RAM 上面運作，不需要很多硬碟空間。這邊建議先至少使用1GB RAM，然後在有需要的時候做垂直擴展，或者說，換成有更大RAM的機器。我們可以透過 SSH 連線進 Redis 主機，然後運作`top`指令，來度量我們使用的 RAM 大小。

另外，將應用設置成不是連進本機的資料庫，而是使用外部的SQL主機，比方說是Google SQL。

## 配置堆疊 {#configuring-the-stack}

以下說明可能會有改變和改進。

---

首先，設置一個臨時的服務器，以用來配置我們的應用程序。

將應用的程式碼放置在 Git 上面，這樣的話，每個主機佈署時都會是最新的程式碼。建議遵守以下步驟：

* 將應用`git clone`到網頁根目錄，可能是`www`資料夾。 根據平台不同，git clone 的資料夾路徑也可能不同。
* 設置一個 cron job，每分鐘執行`git pull`
* 設置一個 cron job，每分鐘在該資料夾執行`composer install`

當應用在我們的臨時伺服器上面運行成功之後，建立一個快照（snapshot），然後用這個快照來建立我們的伺服器集合。

多數 PaaS 平台，像是 Google Cloud Managed Instance Groups 和 Amazon Elastic Beanstalk，可以設置「起始」命令。我們的起始命令應該設置為安裝並更新應用，根據你的伺服器快照是否已經包含應用的 git 使用`git clone`或`git pull`，以及補充使用一個`composer install`指令，來安裝所需要的 composer 套件。

當伺服器集合能成功的使用快照來建立新主機時，我們就可以移除臨時伺服器。

伺服器集合已經成功設置之後，在PaaS 平台上設置一個 load balancer，比方說 Google 上的 Load Balancer，並在 DNS 上面設置網域的 A 或者 CNAME 紀錄，連接到 load balancer 的靜態 IP。

---

上述的機制很單純，但是足以運作一個可自動增減的應用。The mechanism described above is really simple yet sufficient enough to have your application up and running. There can be another process incorporated in the mechanism such as deployment failure prevention, version rollback, zero downtime, etc. 這些會在其他的地方進行介紹。

Google Cloud 和 Amazon Web Services 等不同平台還有提供一些其他的佈署方案。這些佈署方案也會在其他的地方介紹。

## 資源管理 {#assets-management}

Yii 預設是程式運作時產生資源（asset），並儲存在`web/assets`資料夾裡面，以產生的時間命名。如果使用多個伺服器， 每個伺服器設置資源的時間可能會有所不同。不同的原因可能是程式儲存該資源的時間延遲不同，或者因為在自動增減環境下，新的伺服器可能會在其他主機設置一段時間過後才自動加入，導致資源出現的時間有所差異。

這樣的話，同一個資源可能會被不同的伺服器放置在不一樣的路徑，這可能會導致使用者無法存取到該資源。在 load balancer 裡面設定伺服器關係（server affinity），透過讓每個使用者與第一次使用的主機產生關聯，之後的需求都會由同一台主機處理，可以解決這個問題。但是這個作法不太好，因為，如果運氣很不好，大多數動作集中在單一個伺服器上面，可能會造成我們的 cache 集中儲存在同一個地方。而且，在自動增減的環境下，伺服器可能隨時被自動程序關閉，導致使用者的後續動作必須由不同伺服器負責，因此還是產生找不到資源的錯誤。

更安全的方式，是在 config 的 [AssetManager](http://www.yiiframework.com/doc-2.0/yii-web-assetmanager.html#%24hashCallback-detail) 裡面設置`hashCallback`，這樣資源就不會以時間命名，而是依據另一個決定性的函式來命名。

舉例來說，如果你希望讓每個伺服器的資源路徑相同，我們可以修改 `hashCallback`如下：

```php
$config = [
    'components' => [
       'assetManager' => [
           'hashCallback' => function ($path) {
               return hash('md4', $path);
            }    
       ]
    ]
```

如果我們是在 Apache 或者 NGINX 裡面，使用 HTTP caching 機制來處理 JS、CSS、JPG……等資源，我們可以設置[`appendTimestamp`](http://www.yiiframework.com/doc-2.0/yii-web-assetmanager.html#%24appendTimestamp-detail)為真，這樣的話，當資源更新的時候，舊的資源就會被無效化。

```php
$config = [
    'components' => [
       'assetManager' => [
           'appendTimestamp' => true,
           'hashCallback' => function ($path) {
               return hash('md4', $path);
            }    
       ]
    ]
];
```

沒有伺服器關係（server affinity）的負載平衡（Load balancing）可以提高擴展彈性，但是除了資源共用的問題，又多了一個情況： 對所有伺服器的資源可用性問題。

假設使用者對頁面的請求 **1**，傳遞到了伺服器 **A**。伺服器 **A **依據該請求，產生對應的資源，並儲存在本地資料夾裡面。Then the HTML output returned to the browser which then generates request **2 **for the asset. 如果我們設置了伺服器關係，請求 **2** 應該會到達If you configure the server affinity, this request will hit server **A **given the server is still available and the server will return the requested asset. 但是，這個請求實際上不一定會到達伺服器 **A**。而是可能到達尚未產生資源的伺服器 **B**，這樣的話就會產生`404 Not Found`錯誤。雖然最終伺服器 **B **還是會產生這些資源，但是仍舊有產生錯誤的機會。伺服器集合有越多伺服器的話，資源同步的時間就會越長，這種資源同步誤差產生的`404 Not Found`錯誤機會就會越高。

避免這個問題最好的作法是使用 [Asset Combination and Compression](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html#combining-compressing-assets). 。如上所述，when you deploy your application, generally deployment platform can be set to execute hook such as`composer install`. Here you can also execute asset combination and compression using`yii asset assets.php config/assets-prod.php`. Just remember everytime you add new asset in your application, you need to add that asset in the`asset.php`configuration.

