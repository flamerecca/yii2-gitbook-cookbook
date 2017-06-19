# 實做有型態的集合（Implementing typed collections） {#implementing-typed-collections}

對嚴格的型態提示（type hinting）以及界面來說，實做一個類別集合（typed collection）來儲存模型以及其他相似類別的物件是很有用的。

作為示範，假設我們有個`Post`類別：

```php
class Post
{
    private $title;

    public function __construct($title)
    {
        $this->title = $title;
    }

    public function getTitle()
    {
        return $this->title;
    }
}
```

現在我們實做一個用來儲存`Post`，簡單的不可變型態集合（typed immutable collection）如下：

```php
class PostCollection implements \Countable, \IteratorAggregate
{
    private $data;

    public function __construct(array $data)
    {
        foreach ($data as $item) {
            if (!$item instanceof Post) {
                throw new \InvalidArgumentException('All items should be of Post class.');
            }
        }
        $this->data = $data;
    }

    public function count()
    {
        return count($this->data);
    }

    public function getIterator()
    {
        return new \ArrayIterator($this->data);
    }
}
```

好了，現在我們可以使用`PostCollection`如下：

```php
$data = [new Post('post1'), new Post('post 2')];
$collection = new PostCollection($data);

foreach ($collection as $post) {
    echo $post->getTitle();
}
```

除了建立函式中的類型檢查之外，這種作法的主要好處是在於界面（interface）。使用型態集合，我們可以在界面裡明確的要求一組特定類別：

```php
interface FeedGenerator
{
    public function generateFromPosts(PostsCollection $posts);
}
```

而不是

```php
interface FeedGenerator
{
    public function generateFromPosts(array $posts);
}
```



