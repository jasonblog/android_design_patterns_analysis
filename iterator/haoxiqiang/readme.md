Android設計模式源碼解析之迭代器(Iterator)模式 
========================================
> 本文為 [Android 設計模式源碼解析](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis) 中 迭代器模式 分析  
> Android系統版本： 5.0        
> 分析者： [haoxiqiang](https://github.com/Haoxiqiang)，分析狀態：完成，校對者：，校對狀態：未完成



## 1. 模式介紹
### 模式的定義
迭代器（Iterator）模式，又叫做遊標（Cursor）模式。GOF給出的定義為：提供一種方法訪問一個容器（container）對象中各個元素，而又不需暴露該對象的內部細節。

### 模式的使用場景
　　Java JDK 1.2 版開始支持迭代器。每一個迭代器提供next()以及hasNext()方法，同時也支持remove()(1.8的時候remove已經成為default throw new UnsupportedOperationException("remove"))。對Android來說,集合Collection實現了Iterable接口,就是說,無論是List的一大家子還是Map的一大家子,我們都可以使用Iterator來遍歷裡面的元素,[可以使用Iterator的集合](http://docs.oracle.com/javase/8/docs/api/java/util/package-tree.html)      

## 2. UML類圖
　　　
　![iterator](images/Iterator_UML_class_diagram.svg.png)   

### 角色介紹　　 
* 迭代器接口Iterator：該接口必須定義實現迭代功能的最小定義方法集比如提供hasNext()和next()方法。
* 迭代器實現類：迭代器接口Iterator的實現類。可以根據具體情況加以實現。
* 容器接口：定義基本功能以及提供類似Iterator iterator()的方法。
* 容器實現類：容器接口的實現類。必須實現Iterator iterator()方法。


## 3. 模式的簡單實現
###  簡單實現的介紹
我們有一個數組,對其遍歷的過程我們希望使用者像ArrayList一樣的使用,我們就可以用過iterator來實現.

### 實現源碼 
下面我們自己實現一個Iterator的集合

```
...
public Iterator<Mileage> iterator() {
    return new ArrayIterator();
}

private class ArrayIterator implements Iterator<Mileage> {
/**
 * Number of elements remaining in this iteration
 */
private int remaining = size;

/**
 * Index of element that remove() would remove, or -1 if no such elt
 */
private int removalIndex = -1;

@Override
public boolean hasNext() {
    return remaining != 0;
}

@Override
public Mileage next() {
    Mileage mileage = new Mileage();
    removalIndex = size-remaining;
    mileage.name = String.valueOf(versionCodes[removalIndex]);
    mileage.value = versionMileages[removalIndex];
    remaining-=1;
    return mileage;
}

@Override
public void remove() {
    versionCodes[removalIndex]=-1;
    versionMileages[removalIndex]="It was set null";
}
}
...
```
使用的過程如下,我們特意使用了remove方法,注意這個只是一個示例,和大多數的集合相比,該實現並不嚴謹

```
AndroidMileage androidMileage = new AndroidMileage();
Iterator<AndroidMileage.Mileage> iterator =androidMileage.iterator();
while (iterator.hasNext()){
    AndroidMileage.Mileage mileage = iterator.next();
    if(mileage.name.equalsIgnoreCase("16")){
    	//remove掉的是當前的這個,暫時只是置空,並未真的移掉
        iterator.remove();
    }
    Log.e("mileage",mileage.toString());
}
```

下面直接寫出幾種集合的遍歷方式,大家可以對比一下
  
*  HashMap的遍歷
```
HashMap<String, String> colorMap=new HashMap<>();
colorMap.put("Color1","Red");
colorMap.put("Color2","Blue");
Iterator iterator = colorMap.keySet().iterator();
while( iterator. hasNext() ){
    String key = (String) iterator.next();
    String value = colorMap.get(key);
}
```       
* JSONObject的遍歷
```
String paramString = "{menu:{\"1\":\"sql\", \"2\":\"android\", \"3\":\"mvc\"}}";
JSONObject menuJSONObj  = new JSONObject(paramString);
JSONObject  menuObj = menuJSONObj.getJSONObject("menu");
Iterator iter = menuObj.keys();
while(iter.hasNext()){
    String key = (String)iter.next();
    String value = menuObj.getString(key);
}
```      
就目前而言，各種高級語言都有對迭代器的基本實現，沒必要自己實現迭代器，使用內置的迭代器即可滿足日常需求。         

## Android源碼中的模式實現
一個集合想要實現Iterator很是很簡單的,需要注意的是每次需要重新生成一個Iterator來進行遍歷.且遍歷過程是單方向的,HashMap是通過一個類似HashIterator來實現的,我們為了解釋簡單,這裡只是研究ArrayList(此處以Android L源碼為例,其他版本略有不同)

```
@Override public Iterator<E> iterator() {
    return new ArrayListIterator();
}

private class ArrayListIterator implements Iterator<E> {
    /** Number of elements remaining in this iteration */
    private int remaining = size;

    /** Index of element that remove() would remove, or -1 if no such elt */
    private int removalIndex = -1;

    /** The expected modCount value */
    private int expectedModCount = modCount;

    public boolean hasNext() {
        return remaining != 0;
    }

    @SuppressWarnings("unchecked") public E next() {
        ArrayList<E> ourList = ArrayList.this;
        int rem = remaining;
        if (ourList.modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        if (rem == 0) {
            throw new NoSuchElementException();
        }
        remaining = rem - 1;
        return (E) ourList.array[removalIndex = ourList.size - rem];
    }

    public void remove() {
        Object[] a = array;
        int removalIdx = removalIndex;
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        if (removalIdx < 0) {
            throw new IllegalStateException();
        }
        System.arraycopy(a, removalIdx + 1, a, removalIdx, remaining);
        a[--size] = null;  // Prevent memory leak
        removalIndex = -1;
        expectedModCount = ++modCount;
    }
}
```

* java中的寫法一般都是通過iterator()來生成Iterator,保證iterator()每次生成新的實例
* remaining初始化使用整個list的size大小,removalIndex表示remove掉的位置,modCount在集合大小發生變化的時候後都會進行一次modCount++操作,避免數據不一致,前面我寫的例子這方面沒有寫,請務必注意這點
* hasNext方法中,因為remaining是一個size->0的變化過程,這樣只需要判斷非0就可以得知當前遍歷的是否還有未完成的元素
* next,第一次調用的時候返回array[0]的元素,這個過程中removalIndex會被設置成當前array的index
* remove的實現是直接操作的內存中的數據,是能夠直接刪掉元素的,不展開了



## 4. 雜談
### 優點與缺點
#### 優點  
* 面向對象設計原則中的單一職責原則，對於不同的功能,我們要儘可能的把這個功能分解出單一的職責，不同的類去承擔不同的職責。Iterator模式就是分離了集合對象的遍歷行為，抽象出一個迭代器類來負責，這樣不暴露集合的內部結構，又可讓外部代碼透明的訪問集合內部的數據。

#### 缺點 
* 會產生多餘的對象，消耗內存；
* 遍歷過程是一個單向且不可逆的遍歷
* 如果你在遍歷的過程中,集合發生改變,變多變少,內容變化都是算,就會拋出來ConcurrentModificationException異常.


