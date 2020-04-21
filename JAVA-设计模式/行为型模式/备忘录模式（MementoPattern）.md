# 一. 定义
保存一个对象的某个状态，以便在适当的时候恢复对象。该模式又称快照模式。

# 二. 优缺点
## 优点：
1. 为用户提供一种可恢复的机制，方便用户操作历史回滚。

2. 封装存档的信息，用户不需知道信息的保存细节。

## 缺点：
因为需要保存信息，所以会导致资源占用，特别是保存大对象的时候，有可能出现oom。

# 三. 使用场景
1. 提供一个可回滚的操作。

2. 需要保存/恢复数据的相关状态场景，譬如文字或图像的编辑器，Windows系统下使用ctrl+z回退上一步。
# 四. 实例
* 发起人（Originator）角色： 记录当前时刻的内部状态信息，提供创建备忘录和恢复备忘录数据的功能，实现其他业务功能，它可以访问备忘录里的所有信息。

* 备忘录（Memento）角色： 负责存储发起人的内部状态，在需要的时候提供这些内部状态给发起人。

* 管理者（Caretaker）角色： 对备忘录进行管理，提供保存与获取备忘录的功能，但其不能对备忘录的内容进行访问与修改。

![b19ebb76423400c443c2d138cfc4a782](备忘录模式（MementoPattern）.resources/947677AC-7064-434E-8F1F-A36A24E5C1F8.png)

## 示例：模拟编辑器，具有撤销功能，相当于Windows系统的 ctrl+z 。
Article（Originator角色）：
```java
public class Article {
    private String title;
    private String content;
    private String img;

    public ArticleMemento saveArticle(){
        return new ArticleMemento(this.title,this.content,this.img);
    }

    public ArticleMemento undoArticle(ArticleMemento articleMemento){
        return articleMemento;
    }

    //省略get、set方法
}
```
ArticleMemento（Memento角色）：

```java
public class ArticleMemento {

    private String title;
    private String content;
    private String img;

    public ArticleMemento(String title, String content, String img) {
        this.title = title;
        this.content = content;
        this.img = img;
    }

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }

    public String getImg() {
        return img;
    }
}
```
ArticleCaretaker（Caretaker角色）：
```java
public class ArticleCaretaker {
    private Stack<ArticleMemento> stack = new Stack();

    public ArticleMemento get() throws Exception {
        if (stack.size() == 0) {
            throw new Exception("暂存区没有数据...");
        }
        ArticleMemento articleMemento = stack.get(stack.size() - 1);
        return articleMemento;
    }

    public void saveArticle(ArticleMemento articleMemento) {
        stack.push(articleMemento);
    }

    //撤销操作
    public ArticleMemento undoArticle() {
        if (!stack.empty()) {
            stack.pop();
            try {
                return get();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```
Test:测试类
```java
public class Test {
    public static void main(String[] args) throws Exception {
        ArticleCaretaker ac  = new ArticleCaretaker();
        Article article = new Article();

        article.setTitle("设计模式之备忘录模式");
        article.setContent("定义：保存一个对象的某个状态，以便在适当的时候恢复对象。");
        article.setImg("MementoUML.png");
        ac.saveArticle(article.saveArticle());
        System.out.println(ac.get().getTitle());

        article.setTitle("设计模式之单例模式");
        article.setContent("定义：保证一个类只有一个实例，并提供全局访问点。");
        article.setImg("SingletonUML.png");
        ac.saveArticle(article.saveArticle());
        System.out.println(ac.get().getTitle());

        article.setTitle("设计模式之享元模式");
        article.setContent("定义：使用共享的方式高效的支持大量的细粒度的对象。\n");
        article.setImg("FlyweightUML.png");
        ac.saveArticle(article.saveArticle());
        System.out.println(ac.get().getTitle());

        System.out.println("------------------- 执行撤销操作  -------------------");

        ArticleMemento articleMemento1 = article.undoArticle(ac.undoArticle());
        System.out.println(articleMemento1.getTitle());

        ArticleMemento articleMemento2 = article.undoArticle(ac.undoArticle());
        System.out.println(articleMemento2.getTitle());

        ArticleMemento articleMemento3 = article.undoArticle(ac.undoArticle());
        System.out.println(articleMemento3.getTitle());
    }
}
```
输出结果：
```java
设计模式之备忘录模式
设计模式之单例模式
设计模式之享元模式
------------------- 执行撤销操作  -------------------
设计模式之单例模式
设计模式之备忘录模式
java.lang.Exception: 暂存区没有数据...
    at Behavioral.MementoPattern.ArticleCaretaker.get(ArticleCaretaker.java:14)
    at Behavioral.MementoPattern.ArticleCaretaker.undoArticle(ArticleCaretaker.java:29)
    at Behavioral.MementoPattern.Test.main(Test.java:38)
Exception in thread "main" java.lang.NullPointerException
    at Behavioral.MementoPattern.Test.main(Test.java:39)
```
# 五. 总结
在设计备忘录模式的时候，如果不是将备份数据持久化则需要考虑备份数据的上限、摧毁，否则很容易出现oom。