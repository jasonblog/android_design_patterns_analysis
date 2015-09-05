Android設計模式源碼解析之橋接模式 
====================================
> 本文為 [Android 設計模式源碼解析](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis) 中 橋接模式 分析  
> Android系統版本： 4.2         
> 分析者：[shen0834](https://github.com/shen0834)，分析狀態：未完成，校對者：[Mr.Simple](https://github.com/bboyfeiyu)，校對狀態：完成   

## 模式介紹

### 模式的定義

將抽象部分與實現部分分離，使它們都可以獨立的變化。

### 模式的使用場景
* 如果一個系統需要在構件的抽象化角色和具體化角色之間添加更多的靈活性，避免在兩個層次之間建立靜態的聯繫。
* 設計要求實現化角色的任何改變不應當影響客戶端，或者實現化角色的改變對客戶端是完全透明的。
* 需要跨越多個平臺的圖形和窗口系統上。
* 一個類存在兩個獨立變化的維度，且兩個維度都需要進行擴展。

### UML類圖

![uml](http://img.blog.csdn.net/20150322120730408)

### 角色介紹

* 抽象化(Abstraction)角色：抽象化給出的定義，並保存一個對實現化對象的引用。
修正抽象化(Refined Abstraction)角色：擴展抽象化角色，改變和修正父類對抽象化的定義。
* 實現化(Implementor)角色：這個角色給出實現化角色的接口，但不給出具體的實現。必須指出的是，這個接 口不一定和抽象化角色的接口定義相同，實際上，這兩個接口可以非常不一樣。實現化角色應當只給出底層操作，而抽象化角色應當只給出基於底層操作的更高一層的操作。
* 具體實現化(ConcreteImplementor)角色：這個角色給出實現化角色接口的具體實現。

## 模式的簡單實現

### 介紹

其實Java的虛擬機就是一個很好的例子，在不同平臺平臺上，用不同的虛擬機進行實現，這樣只需把Java程序編譯成符合虛擬機規範的文件，且只用編譯一次，便在不同平臺上都能工作。 但是這樣說比較抽象，用一個簡單的例子來實現bridge模式。

 編寫一個程序，使用兩個繪圖的程序的其中一個來繪製矩形或者原型，同時，在實例化矩形的時候，它要知道使用繪圖程序1（DP1）還是繪圖程序2（DP2）。

(ps:假設dp1和dp2的繪製方式不一樣，它們是用不同方式進行繪製，示例代碼，不討論過多細節)

### 實現源碼

```java
    首先是兩個繪圖程序dp1,dp2
//具體的繪圖程序類dp1
public class DP1 {
	
	public void draw_1_Rantanle(){
		System.out.println("使用DP1的程序畫矩形");
	}
	
	public void draw_1_Circle(){
		System.out.println("使用DP1的程序畫圓形");
	}
}
//具體的繪圖程序類dp2
public class DP2 {
  
	public void drawRantanle(){
		System.out.println("使用DP2的程序畫矩形");
	}
	
	public void drawCircle(){
		System.out.println("使用DP2的程序畫圓形");
	}
	
}
接著​抽象的形狀Shape和兩個派生類：矩形Rantanle和圓形Circle
//抽象化角色Abstraction
abstract class Shape {
	//持有實現的角色Implementor(作圖類)
	protected Drawing myDrawing;
	
	public Shape(Drawing drawing) {
		this.myDrawing = drawing;
	}
	
	abstract public void draw();

	//保護方法drawRectangle
	protected void drawRectangle(){
       //this.impl.implmentation()
		myDrawing.drawRantangle();
	}

	//保護方法drawCircle
	protected void drawCircle(){
        //this.impl.implmentation()
		myDrawing.drawCircle();
	}
}
//修正抽象化角色Refined Abstraction(矩形)
public class Rantangle extends Shape{
	public Rantangle(Drawing drawing) {
		super(drawing);
	}
     
	@Override
	public void draw() {
		drawRectangle();
	}
}
//修正抽象化角色Refined Abstraction(圓形)
public class Circle extends Shape {
	public Circle(Drawing drawing) {
		super(drawing);
	}
	@Override
	public void draw() {
		drawCircle();
	}
}
最後，我們的實現繪圖的Drawing和分別實現dp1的V1Drawing和dp2的V2Drawing
//實現化角色Implementor
//implmentation兩個方法，畫圓和畫矩形
public interface Drawing {
	public void drawRantangle();
	public void drawCircle();
}
//具體實現化邏輯ConcreteImplementor
//實現了接口方法，使用DP1進行繪圖
public class V1Drawing implements Drawing{
 
	DP1 dp1;
	
	public V1Drawing() {
		dp1 = new DP1();
	}
	@Override
	public void drawRantangle() {
		dp1.draw_1_Rantanle();
	}
	@Override
	public void drawCircle() {
		dp1.draw_1_Circle();
	}			
}
//具體實現化邏輯ConcreteImplementor
//實現了接口方法，使用DP2進行繪圖
public class V2Drawing implements Drawing{
	DP2 dp2;
	
	public V2Drawing() {
		dp2 = new DP2();
	}
	
	@Override
	public void drawRantangle() {
		dp2.drawRantanle();
	}
	@Override
	public void drawCircle() {
		dp2.drawCircle();
	}
}
```

   ​在這個示例中，圖形Shape類有兩種類型，圓形和矩形，為了使用不同的繪圖程序繪製圖形，把實現的部分進行了分離，構成了Drawing類層次結構，包括V1Drawing和V2Drawing。在具體實現類中，V1Drawing控制著DP1程序進行繪圖，V2Drawing控制著DP2程序進行繪圖，以及保護的方法drawRantangle,drawCircle(Shape類中) 。

## Android源碼中的模式實現

在Android中也運用到了Bridge模式，我們使用很多的ListView和BaseAdpater其實就是Bridge模式的運行，很多人會問這個不是Adapter模式，接下來根據源碼來分析。

首先ListAdapter.java：

```java
public interface ListAdapter extends Adapter{
    //繼承自Adapter，擴展了自己的兩個實現方法
      public boolean areAllItemsEnabled();
      boolean isEnabled(int position);
}  
```

這裡先來看一下父類AdapterView。
   
```java   
public abstract class AdapterView<T extends Adapter> extends ViewGroup {  
    //這裡需要一個泛型的Adapter
      public abstract T getAdapter();
     public abstract void setAdapter(T adapter);  
}  
```

接著來看ListView的父類AbsListView，繼承自AdapterView

```java
public abstract class AbsListView extends AdapterView<ListAdapter>   
    //繼承自AdapterView,並且指明瞭T為ListAdapter
    /**
     * The adapter containing the data to be displayed by this view
     */
    ListAdapter mAdapter;  
    //代碼省略
    //這裡實現了setAdapter的方法，實例了對實現化對象的引用
    public void setAdapter(ListAdapter adapter) {
        //這的adapter是從子類傳入上來，也就是listview，拿到了具體實現化的對象
        if (adapter != null) {
            mAdapterHasStableIds = mAdapter.hasStableIds();
            if (mChoiceMode != CHOICE_MODE_NONE && mAdapterHasStableIds &&
                    mCheckedIdStates == null) {
                mCheckedIdStates = new LongSparseArray<Integer>();
            }
        }
        if (mCheckStates != null) {
            mCheckStates.clear();
        }
        if (mCheckedIdStates != null) {
            mCheckedIdStates.clear();
        }
    }
```

大家都知道，構建一個listview，adapter中最重要的兩個方法，getCount()告知數量，getview()告知具體的view類型，接下來看看AbsListView作為一個視圖的集合是如何來根據實現化對象adapter來實現的具體的view呢？

```java
   protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        
            //省略代碼，
            //這裡在加入window的時候，getCount()確定了集合的個數
            mDataChanged = true;
            mOldItemCount = mItemCount;
            mItemCount = mAdapter.getCount();
        }
    }
```
    
接著來看

```java
 View obtainView(int position, boolean[] isScrap) {
        //代碼省略
      ​//這裡根據位置顯示具體的view,return的child是從持有的實現對象mAdapter裡面的具體實現的
      ​//方法getview來得到的。
        final View child = mAdapter.getView(position, scrapView, this);
        //代碼省略
        return child;
    }  
```

   接下來在ListView中，onMeasure調用了obtainView來確定寬高，在擴展自己的方法來排列這些view。知道了

這些以後，我們來畫一個簡易的UML圖來看下:

![uml](http://img.blog.csdn.net/20150322120809221)

對比下GOF的上圖，是不是發現很像呢？實際上最開始研究Adapter模式的時候,越看越不對啊，於是整理結構，畫了UML發現這更像是一個bridge模式，那時候對設計模式也是模模糊糊的，於是靜下來研究。抽象化的角色一個視圖的集合AdapterView，它擴展了AbsListView，AbsSpinner，接下來他們分別擴展了ListView，GridView，Spinner,Gallery，用不同方式來展現這些ItemViews，我們繼續擴展類似ListView的PulltoRefreshView等等。而實現化角色Adapter擴展了ListAdpater,SpinnerAdapter，接著具體的實現化角色BaseAdapter實現了他們，我們通過繼承BaseAdapter又實現了我們各式各樣的ItemView。


## 雜談

這裡就是Android工程師的牛X之處了，用一個bridge和adapter來解決了一個大的難題。試想一下，視圖的排列方式是無窮盡，是人們每個人開發的視圖也是無窮盡的。如果你正常開發，你需要多少類來完成呢？而Android把最常用用的展現方式全部都封裝了出來，而在實現角色通過Adapter模式來應變無窮無盡的視圖需要。抽象化了一個容器使用適配器來給容器裡面添加視圖，容器的形狀(或理解為展現的方式)以及怎麼樣來繪製容器內的視圖，你都可以獨自的變化，雙雙不會干擾，真正的脫耦，就要最開始說的那樣：“將抽象部分與實現部分分離，使它們都可以獨立的變化。”

從上面的兩個案例，我們可以看出，我們在兩個解決方案中都用到bridge和adapter模式，那是因為我們必須使用給定的繪圖程序(adapter適配器)，繪圖程序(adapter適配器)有已經存在的接口必須要遵循，因此需要使用Adapter進行適配，然後才能用同樣的方式處理他們,他們經常一起使用，並且相似，但是Adapter並不是Bridge的一部分。

### 優點與缺點
實現與使用實現的對象解耦，提供了可擴展性，客戶對象無需擔心操作的實現問題。  如果你採用了bridge模式，在處理新的實現將會非常容易。你只需定義一個新的具體實現類，並且實現它就好了，不需要修改任何其他的東西。但是如果你出現了一個新的具體情況，需要對實現進行修改時，就得先修改抽象的接口，再對其派生類進行修改，但是這種修改只會存在於局部,並且這種修改將變化的英雄控制在局部，並且降低了出現副作用的風險，而且類之間的關係十分清晰，如何實現一目瞭然。
