Android設計模式源碼解析之適配器(Adapter)模式 
====================================
> 本文為 [Android 設計模式源碼解析](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis) 中 適配器模式 分析  
> Android系統版本： 2.3        
> 分析者：[Mr.Simple](https://github.com/bboyfeiyu)，分析狀態：完成，校對者：[Mr.Simple](https://github.com/bboyfeiyu)，校對狀態：完成 



## 1. 模式介紹
### 模式的定義
適配器模式把一個類的接口變換成客戶端所期待的另一種接口，從而使原本因接口不匹配而無法在一起工作的兩個類能夠在一起工作。

### 使用場景
　　用電源接口做例子，筆記本電腦的電源一般都是接受5V的電壓，但是我們生活中的電線電壓一般都是220V的輸出。這個時候就出現了不匹配的狀況，在軟件開發中我們稱之為接口不兼容，此時就需要適配器來進行一個接口轉換。在軟件開發中有一句話正好體現了這點：任何問題都可以加一箇中間層來解決。這個層我們可以理解為這裡的Adapter層，通過這層來進行一個接口轉換就達到了兼容的目的。            

## 2. UML類圖 
　　
　　適配器模式也分兩種，即類適配器模式、對象適配器模式，結構圖如圖1、圖2。
　　<table>
    <tr>
        <td><img src="http://img.blog.csdn.net/20150220212211272"></td>
        <td> <img src="http://img.blog.csdn.net/20150220212230909"></td>
    </tr>
      <tr>
        <td>圖1</td>
        <td>圖2</td>
    </tr>
</table>

　　 如圖所示，類適配器是通過實現Target接口以及繼承Adaptee類來實現接口轉換，而對象適配器模式則是通過實現Target接口和代理Adaptee的某個方法來實現。結構上略有不同。           

### 角色介紹　　 
*   目標(Target)角色：這就是所期待得到的接口。注意：由於這裡討論的是類適配器模式，因此目標不可以是類。
*     源(Adapee)角色：現在需要適配的接口。
*     適配器(Adaper)角色：適配器類是本模式的核心。適配器把源接口轉換成目標接口。顯然，這一角色不可以是接口，而必須是具體類。      


## 3. 模式的簡單實現

  在上述電源接口這個示例中，5V電壓就是Target接口，220v電壓就是Adaptee類，而將電壓從220V轉換到5V就是Adapter。 
  
### 類適配器模式

```java
/**
 * Target角色
 */
public interface FiveVolt {
    public int getVolt5();
}

/**
 * Adaptee角色,需要被轉換的對象
 */
public class Volt220 {
    public int getVolt220() {
        return 220;
    }
}

// adapter角色
public class ClassAdapter extends Volt220 implements FiveVolt {

    @Override
    public int getVolt5() {
        return 5;
    }

}
```
　　Target角色給出了需要的目標接口，而Adaptee類則是需要被轉換的對象。Adapter則是將Volt220轉換成Target的接口。對應的是Target的目標是要獲取5V的輸出電壓，而Adaptee即正常輸出電壓是220V，此時我們就需要電源適配器類將220V的電壓轉換為5V電壓，解決接口不兼容的問題。     

```java
public class Test {
    public static void main(String[] args) {
        ClassAdapter adapter = new ClassAdapter();
        System.out.println("輸出電壓 : " + adapter.getVolt5());
    }
}
```       
　

### 對象適配器模式
　　與類的適配器模式一樣，對象的適配器模式把被適配的類的API轉換成為目標類的API，與類的適配器模式不同的是，對象的適配器模式不是使用繼承關係連接到Adaptee類，而是使用代理關係連接到Adaptee類。     
　　
　　從圖2可以看出，Adaptee類 ( Volt220 ) 並沒有getVolt5()方法，而客戶端則期待這個方法。為使客戶端能夠使用Adaptee類，需要提供一個包裝類Adapter。這個包裝類包裝了一個Adaptee的實例，從而此包裝類能夠把Adaptee的API與Target類的API銜接起來。Adapter與Adaptee是委派關係，這決定了適配器模式是對象的。

示例代碼如下 : 

```java
/**
 * Target角色
 */
public interface FiveVolt {
    public int getVolt5();
}

/**
 * Adaptee角色,需要被轉換的對象
 */
public class Volt220 {
    public int getVolt220() {
        return 220;
    }
}

// 對象適配器模式
public class ObjectAdapter implements FiveVolt {

    Volt220 mVolt220;

    public ObjectAdapter(Volt220 adaptee) {
        mVolt220 = adaptee;
    }

    public int getVolt220() {
        return mVolt220.getVolt220();
    }

    @Override
    public int getVolt5() {
        return 5;
    }

}
```     
注意，這裡為了節省代碼，我們並沒有遵循一些面向對象的基本原則。       

使用示例 :    

```java
public class Test {
    public static void main(String[] args) {
        ObjectAdapter adapter = new ObjectAdapter(new Volt220());
        System.out.println("輸出電壓 : " + adapter.getVolt5());
    }
}
```  

### 類適配器和對象適配器的權衡

　　*　　類適配器使用對象繼承的方式，是靜態的定義方式；而對象適配器使用對象組合的方式，是動態組合的方式。
 
　　*　　對於類適配器，由於適配器直接繼承了Adaptee，使得適配器不能和Adaptee的子類一起工作，因為繼承是靜態的關係，當適配器繼承了Adaptee後，就不可能再去處理Adaptee的子類了。對於對象適配器，一個適配器可以把多種不同的源適配到同一個目標。換言之，同一個適配器可以把源類和它的子類都適配到目標接口。因為對象適配器採用的是對象組合的關係，只要對象類型正確，是不是子類都無所謂。

　　*　  對於類適配器，適配器可以重定義Adaptee的部分行為，相當於子類覆蓋父類的部分實現方法。對於對象適配器，要重定義Adaptee的行為比較困難，這種情況下，需要定義Adaptee的子類來實現重定義，然後讓適配器組合子類。雖然重定義Adaptee的行為比較困難，但是想要增加一些新的行為則方便的很，而且新增加的行為可同時適用於所有的源。

　　*　　對於類適配器，僅僅引入了一個對象，並不需要額外的引用來間接得到Adaptee。對於對象適配器，需要額外的引用來間接得到Adaptee。

　　建議儘量使用對象適配器的實現方式，多用合成/聚合、少用繼承。當然，具體問題具體分析，根據需要來選用實現方式，最適合的才是最好的。
　　

## Android ListView中的Adapter模式
在開發過程中,ListView的Adapter是我們最為常見的類型之一。一般的用法大致如下: 

```java
// 代碼省略
 ListView myListView = (ListView)findViewById(listview_id);
 // 設置適配器
 myListView.setAdapter(new MyAdapter(context, myDatas));

// 適配器
public class MyAdapter extends BaseAdapter{
 
        private LayoutInflater mInflater;
        List<String> mDatas ; 
         
        public MyAdapter(Context context, List<String> datas){
            this.mInflater = LayoutInflater.from(context);
            mDatas = datas ;
        }
        @Override
        public int getCount() {
            return mDatas.size();
        }
 
        @Override
        public String getItem(int pos) {
            return mDatas.get(pos);
        }
 
        @Override
        public long getItemId(int pos) {
            return pos;
        }
 
 		// 解析、設置、緩存convertView以及相關內容
        @Override
        public View getView(int position, View convertView, ViewGroup parent) { 
            ViewHolder holder = null;
            // Item View的複用
            if (convertView == null) {
                holder = new ViewHolder();  
                convertView = mInflater.inflate(R.layout.my_listview_item, null);
              	// 獲取title
                holder.title = (TextView)convertView.findViewById(R.id.title);
                convertView.setTag(holder);
            } else {
                holder = (ViewHolder)convertView.getTag();
            }
            holder.title.setText(mDatas.get(position));
            return convertView;
        }
         
    }
```
這看起來似乎還挺麻煩的，看到這裡我們不禁要問，ListView為什麼要使用Adapter模式呢？    
我們知道，作為最重要的View，ListView需要能夠顯示各式各樣的視圖，每個人需要的顯示效果各不相同，顯示的數據類型、數量等也千變萬化。那麼如何隔離這種變化尤為重要。     

Android的做法是增加一個Adapter層來應對變化，將ListView需要的接口抽象到Adapter對象中，這樣只要用戶實現了Adapter的接口，ListView就可以按照用戶設定的顯示效果、數量、數據來顯示特定的Item View。     
通過代理數據集來告知ListView數據的個數( getCount函數 )以及每個數據的類型( getItem函數 )，最重要的是要解決Item View的輸出。Item View千變萬化，但終究它都是View類型，Adapter統一將Item View輸出為View ( getView函數 )，這樣就很好的應對了Item View的可變性。     

那麼ListView是如何通過Adapter模式 ( 不止Adapter模式 )來運作的呢 ？我們一起來看一看。   
ListView繼承自AbsListView，Adapter定義在AbsListView中，我們看一看這個類。     
 
```java
public abstract class AbsListView extends AdapterView<ListAdapter> implements TextWatcher,
        ViewTreeObserver.OnGlobalLayoutListener, Filter.FilterListener,
        ViewTreeObserver.OnTouchModeChangeListener,
        RemoteViewsAdapter.RemoteAdapterConnectionCallback {

    ListAdapter mAdapter ;
        
    // 關聯到Window時調用的函數
    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        // 代碼省略
		// 給適配器註冊一個觀察者,該模式下一篇介紹。
        if (mAdapter != null && mDataSetObserver == null) {
            mDataSetObserver = new AdapterDataSetObserver();
            mAdapter.registerDataSetObserver(mDataSetObserver);

            // Data may have changed while we were detached. Refresh.
            mDataChanged = true;
            mOldItemCount = mItemCount
            // 獲取Item的數量,調用的是mAdapter的getCount方法
            mItemCount = mAdapter.getCount();
        }
        mIsAttached = true;
    }

  /**
     * 子類需要覆寫layoutChildren()函數來佈局child view,也就是Item View
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        mInLayout = true;
        if (changed) {
            int childCount = getChildCount();
            for (int i = 0; i < childCount; i++) {
                getChildAt(i).forceLayout();
            }
            mRecycler.markChildrenDirty();
        }
        
        if (mFastScroller != null && mItemCount != mOldItemCount) {
            mFastScroller.onItemCountChanged(mOldItemCount, mItemCount);
        }
		// 佈局Child View
        layoutChildren();
        mInLayout = false;

        mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;
    }

	// 獲取一個Item View
	View obtainView(int position, boolean[] isScrap) {
        isScrap[0] = false;
        View scrapView;
		// 從緩存的Item View中獲取,ListView的複用機制就在這裡
        scrapView = mRecycler.getScrapView(position);

        View child;
        if (scrapView != null) {
            // 代碼省略
            child = mAdapter.getView(position, scrapView, this);
            // 代碼省略
        } else {
            child = mAdapter.getView(position, null, this);
            // 代碼省略
        }

        return child;
    }
    }
```    
AbsListView定義了集合視圖的框架，比如Adapter模式的應用、複用Item View的邏輯、佈局Item View的邏輯等。子類只需要覆寫特定的方法即可實現集合視圖的功能，例如ListView。

ListView中的相關方法。    

```java
    @Override
    protected void layoutChildren() {
        // 代碼省略

        try {
            super.layoutChildren();
            invalidate();
			// 代碼省略
			// 根據佈局模式來佈局Item View
            switch (mLayoutMode) {
            case LAYOUT_SET_SELECTION:
                if (newSel != null) {
                    sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
                } else {
                    sel = fillFromMiddle(childrenTop, childrenBottom);
                }
                break;
            case LAYOUT_SYNC:
                sel = fillSpecific(mSyncPosition, mSpecificTop);
                break;
            case LAYOUT_FORCE_BOTTOM:
                sel = fillUp(mItemCount - 1, childrenBottom);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_FORCE_TOP:
                mFirstPosition = 0;
                sel = fillFromTop(childrenTop);
                adjustViewsUpOrDown();
                break;
            case LAYOUT_SPECIFIC:
                sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
                break;
            case LAYOUT_MOVE_SELECTION:
                sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
                break;
            default:
                // 代碼省略
                break;
            }
        
    }
    
    // 從上到下填充Item View  [ 只是其中一種填充方式 ]
    private View fillDown(int pos, int nextTop) {
        View selectedView = null;

        int end = (mBottom - mTop);
        if ((mGroupFlags & CLIP_TO_PADDING_MASK) == CLIP_TO_PADDING_MASK) {
            end -= mListPadding.bottom;
        }

        while (nextTop < end && pos < mItemCount) {
            // is this the selected item?
            boolean selected = pos == mSelectedPosition;
            View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);

            nextTop = child.getBottom() + mDividerHeight;
            if (selected) {
                selectedView = child;
            }
            pos++;
        }

        return selectedView;
    }
    
    // 添加Item View
    private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
            boolean selected) {
        View child;

		// 代碼省略 
        // Make a new view for this position, or convert an unused view if possible
        child = obtainView(position, mIsScrap);

        // This needs to be positioned and measured
        setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);

        return child;
    }
    
```   
ListView覆寫了AbsListView中的layoutChilden函數，在該函數中根據佈局模式來佈局Item View。Item View的個數、樣式都通過Adapter對應的方法來獲取，獲取個數、Item View之後，將這些Item View佈局到ListView對應的座標上，再加上Item View的複用機制，整個ListView就基本運轉起來了。     

當然這裡的Adapter並不是經典的適配器模式，但是卻是對象適配器模式的優秀示例，也很好的體現了面向對象的一些基本原則。這裡的Target角色和Adapter角色融合在一起，Adapter中的方法就是目標方法；而Adaptee角色就是ListView的數據集與Item View，Adapter代理數據集，從而獲取到數據集的個數、元素。     

通過增加Adapter一層來將Item View的操作抽象起來，ListView等集合視圖通過Adapter對象獲得Item的個數、數據元素、Item View等，從而達到適配各種數據、各種Item視圖的效果。因為Item View和數據類型千變萬化，Android的架構師們將這些變化的部分交給用戶來處理，通過getCount、getItem、getView等幾個方法抽象出來，也就是將Item View的構造過程交給用戶來處理，靈活地運用了適配器模式，達到了無限適配、擁抱變化的目的。       

## 雜談
### 優點
* 更好的複用性     
　　系統需要使用現有的類，而此類的接口不符合系統的需要。那麼通過適配器模式就可以讓這些功能得到更好的複用。

* 更好的擴展性     
　　在實現適配器功能的時候，可以調用自己開發的功能，從而自然地擴展系統的功能。

### 缺點
* 過多的使用適配器，會讓系統非常零亂，不易整體進行把握。比如，明明看到調用的是A接口，其實內部被適配成了B接口的實現，一個系統如果太多出現這種情況，無異於一場災難。因此如果不是很有必要，可以不使用適配器，而是直接對系統進行重構。
