本文為 Android設計模式源碼中 組合模式 分析  
Android系統版本： 5.0  
分析者：Tiny-Times，分析狀態：未完成，校對者：無，校對狀態：未開始

## 1\. 模式介紹

### 模式的定義

        組合模式(Composite Pattern)又叫作部分-整體模式，它使我們樹型結構的問題中，模糊了簡單元素和複雜元素的概念，客戶程序可以向處理簡單元素一樣來處理複雜元素,從而使得客戶程序與複雜元素的內部結構解耦。 GoF在《設計模式》一書中這樣定義組合模式：將對象組合成樹形結構以表示“部分-整體”的層次結構。使得用戶對單個對象和組合對象的使用具有一致性。

### 模式的使用場景

*   表示對象的部分-整體層次結構。
*   從一個整體中能夠獨立出部分模塊或功能的場景。

## 2\. UML類圖

* * *

![組合模式通用類圖][1]

### 角色分析

*   Component抽象構件角色 ：定義參加組合對象的共有方法和屬性，可以定義一些默認的行為或屬性。
*   Leaf葉子構件 : 葉子對象，其下再也沒有其他的分支。
*   Composite樹枝構件 ：樹枝對象，它的作用是組合樹枝節點和葉子節點形成一個樹形結構。

## 3\. 該模式的實現實例

* * *

抽象構件 Component.java：

     public abstract class Component {
        //個體和整體都具有的共享
       public void doSomething(){
        //業務邏輯
       }
     }
    

樹枝構件 Composite.java

    public class Composite extends Component {
       //構件容器
       private ArrayList<Component> componentArrayList = new ArrayList<Component>();
       //增加一個葉子構件或樹枝構件
       public void add(Component component){
          this.componentArrayList.add(component);
       }
       //刪除一個葉子構件或樹枝構件
       public void remove(Component component){
          this.componentArrayList.remove(component);
       }
       //獲得分支下的所有葉子構件和樹枝構件
       public ArrayList<Component> getChildren(){
         return this.componentArrayList;
       }
     }
    

樹葉構件 Leaf.java

    public class Leaf extends Component {
    
       //可以覆寫父類方法
       public void doSomething(){    
       }
    
    }
    

場景類 Client.java

    public class Client {
        public static void main(String[] args) {
           //創建一個根節點
          Composite root = new Composite();
          root.doSomething();
          //創建一個樹枝構件
          Composite branch = new Composite();
          //創建一個葉子節點
          Leaf leaf = new Leaf();
          //建立整體
          root.add(branch);
          branch.add(leaf);
       }
       //通過遞歸遍歷樹
       public static void display(Composite root){
          for(Component c:root.getChildren()){
             if(c instanceof Leaf){ //葉子節點
                 c.doSomething();
             }else{ //樹枝節點
                 display((Composite)c);
                }
            }
       }
    }
    

### 組合模式在Android源碼中的應用

* * *

**Adnroid系統中採用組合模式的組合視圖類圖：**

![enter image description here][2]

**具體實現代碼**

View.java

    public class View ....{
     //此處省略無關代碼...
    }
    

ViewGroup.java

    public abstract class ViewGroup extends View ...{
    
         //增加子節點
        public void addView(View child, int index) { 
    
        }
        //刪除子節點
        public void removeView(View view) {
    
        }
         //查找子節點
        public View getChildAt(int index) {
          try {
                return mChildren[index];
              } catch (IndexOutOfBoundsException ex) {
                return null;
              } 
       }
    }
    

## 4\. 注意事項

* * *

使用組合模式組織起來的對象具有出色的層次結構，每當對頂層組合對象執行一個操作的時候，實際上是在對整個結構進行深度優先的節點搜索。但是這些優點都是用操作的代價換取的，比如頂級每執行一次 store.show 實際的操作就是整一顆樹形結構的節點均遍歷執行一次。

## 5\. 雜談

* * *

**優點**

*   不破壞封裝，整體類與局部類之間鬆耦合，彼此相對獨立 。
*   具有較好的可擴展性。
*   支持動態組合。在運行時，整體對象可以選擇不同類型的局部對象。
*   整體類可以對局部類進行包裝，封裝局部類的接口，提供新的接口。

**缺點**

*   整體類不能自動獲得和局部類同樣的接口。
*   創建整體類的對象時，需要創建所有局部類的對象 。

 [1]: http://belial.me/wp-content/uploads/2015/03/QQ截圖20150318225518.png
 [2]: http://belial.me/wp-content/uploads/2015/03/QQ截圖20150315115212.png