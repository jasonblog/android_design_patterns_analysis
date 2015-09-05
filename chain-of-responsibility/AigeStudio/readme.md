Android設計模式源碼解析之責任鏈模式 
====================================
> 本文為 [Android 設計模式源碼解析](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis) 中責任鏈模式分析  
> Android系統版本： 4.4.4        
> 分析者：[Aige](https://github.com/AigeStudio)，分析狀態：完成，校對者：[SM哥](https://github.com/bboyfeiyu)，校對狀態：撒丫校對中  
 
## 1. 模式介紹  
 
###  模式的定義
一個請求沿著一條“鏈”傳遞，直到該“鏈”上的某個處理者處理它為止。


### 模式的使用場景
 一個請求可以被多個處理者處理或處理者未明確指定時。
 

## 2. UML類圖
![UML](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis/blob/master/chain-of-responsibility/AigeStudio/images/chain-of-responsibility.jpg?raw=true)

### 角色介紹
Client：客戶端

Handler：抽象處理者

ConcreteHandler：具體處理者




## 3. 模式的簡單實現
###  簡單實現的介紹
責任鏈模式非常簡單異常好理解，相信我它比單例模式還簡單易懂，其應用也幾乎無所不在，甚至可以這麼說……從你敲代碼的第一天起你就不知不覺用過了它最原始的裸體結構：分支語句：
```java
public class SimpleResponsibility {
	public static void main(String[] args) {
		int request = (int) (Math.random() * 3);
		switch (request) {
		case 0:
			System.out.println("SMBother handle it: " + request);
			break;
		case 1:
			System.out.println("Aige handle it: " + request);
			break;
		case 2:
			System.out.println("7Bother handle it: " + request);
			break;
		default:
			break;
		}
	}
}
```
誰敢說沒用過上面這種結構體的站出來我保證不打屎他，沒用過swith至少if-else用過吧，if-else都沒用過你怎麼知道github的……上面的這段代碼其實就是一種最最簡單的責任鏈模式，其根據request的值進行不同的處理。當然這只是個不恰當的例子來讓大家儘快對責任鏈模式有個簡單的理解，因為可能很多童鞋第一次聽說這個模式，而人對未知事物總是恐懼的，為了消除大家的這種恐懼，我將大家最常見的code搬出來相信熟悉的代碼對大家來說有一種親切的感覺，當然我們實際應用中的責任鏈模式絕逼不是這麼Mr.Simple，但是也不會複雜不到哪去。責任鏈模式，顧名思義，必定與責任Responsibility相關，其實質呢就像上面定義中說的那樣一個請求（比如上面代碼中的request值）沿著一條“鏈”（比如上面代碼中我們的switch分支語句）傳遞，當某個處於“鏈”上的處理者（case定義的條件）處理它時完成處理。其實現實生活中關於責任者模式的例子數不勝數，最常見的就是工作中上下級之間的責任請求關係了。比如：
>程序猿狗屎運被派出去異國出差一週，這時候就要去申請一定的差旅費了，你心裡小算一筆加上各種車馬費估計大概要個兩三萬，於是先向小組長彙報申請，可是大於一千塊小組長沒權利批覆，於是只好去找項目主管，項目主管一看媽蛋這麼狠要這麼多我只能批小於五千塊的，於是你只能再跑去找部門經理，部門經理看了下一陣淫笑後說沒法批我只能批小於一萬的，於是你只能狗血地去跪求老總，老總一看喲！小夥子心忒黑啊！老總話雖如此但還是把錢批給你了畢竟是給公司辦事，到此申請處理完畢，你也可以屁顛屁顛地滾了。

如果把上面的場景應用到責任鏈模式，那麼我們的request請求就是申請經費，組長主管經理老總們就是一個個具體的責任人他們可以對請求做出處理但是他們只能在自己的責任範圍內處理該處理的請求，而程序猿只是個底層狗請求者向責任人們發起請求…………苦逼的猿。

### 實現源碼
上面的場景我們可以使用使用如下的代碼來模擬實現：

首先定義一個程序員類：
```Java
/**
 * 程序猿類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class ProgramApe {
	private int expenses;// 聲明整型成員變量表示出差費用
	private String apply = "爹要點錢出差";// 聲明字符串型成員變量表示差旅申請

	/*
	 * 含參構造方法
	 */
	public ProgramApe(int expenses) {
		this.expenses = expenses;
	}

	/*
	 * 獲取程序員具體的差旅費用
	 */
	public int getExpenses() {
		return expenses;
	}

	/*
	 * 獲取差旅費申請
	 */
	public String getApply() {
		return apply;
	}
}
```

然後依次是各個大爺類：

```Java
/**
 * 小組長類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class GroupLeader {

	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的猿
	 */
	public void handleRequest(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("GroupLeader: Of course Yes!");
	}
}
```

```Java
/**
 * 項目主管類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Director {
	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的猿
	 */
	public void handleRequest(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Director: Of course Yes!");
	}
}
```

```Java
/**
 * 部門經理類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Manager {
	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的猿
	 */
	public void handleRequest(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Manager: Of course Yes!");
	}
}
```

```Java
/**
 * 老總類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Boss {
	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的猿
	 */
	public void handleRequest(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Boss: Of course Yes!");
	}
}
```

好了，萬事俱備只欠場景，現在我們模擬一下整個場景過程：

```Java
/**
 * 場景模擬類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Client {
	public static void main(String[] args) {
		/*
		 * 先來一個程序猿 這裡給他一個三萬以內的隨機值表示需要申請的差旅費
		 */
		ProgramApe ape = new ProgramApe((int) (Math.random() * 30000));

		/*
		 * 再來四個老大
		 */
		GroupLeader leader = new GroupLeader();
		Director director = new Director();
		Manager manager = new Manager();
		Boss boss = new Boss();

		/*
		 * 處理申請
		 */
		if (ape.getExpenses() <= 1000) {
			leader.handleRequest(ape);
		} else if (ape.getExpenses() <= 5000) {
			director.handleRequest(ape);
		} else if (ape.getExpenses() <= 10000) {
			manager.handleRequest(ape);
		} else {
			boss.handleRequest(ape);
		}
	}
}
```

運行一下，我的結果輸出如下（注：由於隨機值的原因你的結果也許與我不一樣）：

>爹要點錢出差
>
>Manager: Of course Yes!

是不是感覺有點懂了？當然上面的代碼雖然在一定程度上體現了責任鏈模式的思想，但是確是非常terrible的。作為一個code新手可以原諒，但是對有一定經驗的code+來說就不可饒恕了，很明顯所有的老大都有共同的handleRequest方法而程序猿也有不同類型的，比如一個公司的php、c/c++、Android、IOS等等，所有的這些共性我們都可以將其抽象為一個抽象類或接口，比如我們的程序猿抽象父類：

```java
/**
 * 程序猿抽象接口
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public abstract class ProgramApes {
	/**
	 * 獲取程序員具體的差旅費用
	 * 
	 * @return 要多少錢
	 */
	public abstract int getExpenses();

	/**
	 * 獲取差旅費申請
	 * 
	 * @return Just a request
	 */
	public abstract String getApply();
}
```

這時我們就可以實現該接口使用呆毛具現化一個具體的程序猿，比如Android猿：

```java
/**
 * Android程序猿類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class AndroidApe extends ProgramApes {
	private int expenses;// 聲明整型成員變量表示出差費用
	private String apply = "爹要點錢出差";// 聲明字符串型成員變量表示差旅申請

	/*
	 * 含參構造方法
	 */
	public AndroidApe(int expenses) {
		this.expenses = expenses;
	}

	@Override
	public int getExpenses() {
		return expenses;
	}

	@Override
	public String getApply() {
		return apply;
	}
}
```
同樣的，所有的老大都有一個批覆經費申請的權利，我們把這個權利抽象為一個IPower接口：

```java
/**
 * 老大們的權利接口
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public interface IPower {
	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的猿
	 */
	public void handleRequest(ProgramApe ape);
}
```

然後讓所有的老大們實現該接口即可其它不變，而場景類Client中也只是修改各個老大的引用類型為IPower而已，具體代碼就不貼了，運行效果也類似。

然而上面的代碼依然問題重重，為什麼呢？大家想想，當程序猿發出一個申請時卻是在場景類中做出判斷決定的……然而這個職責事實上應該由老大們來承擔並作出決定，上面的代碼搞反了……既然知道了錯誤，那麼我們就來再次重構一下代碼：

把所有老大抽象為一個leader抽象類，在該抽象類中實現處理邏輯：

```java
/**
 * 領導人抽象類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public abstract class Leader {
	private int expenses;// 當前領導能批覆的金額
	private Leader mSuperiorLeader;// 上級領導

	/**
	 * 含參構造方法
	 * 
	 * @param expenses
	 *            當前領導能批覆的金額
	 */
	public Leader(int expenses) {
		this.expenses = expenses;
	}

	/**
	 * 迴應程序猿
	 * 
	 * @param ape
	 *            具體的程序猿
	 */
	protected abstract void reply(ProgramApe ape);

	/**
	 * 處理請求
	 * 
	 * @param ape
	 *            具體的程序猿
	 */
	public void handleRequest(ProgramApe ape) {
		/*
		 * 如果說程序猿申請的money在當前領導的批覆範圍內
		 */
		if (ape.getExpenses() <= expenses) {
			// 那麼就由當前領導批覆即可
			reply(ape);
		} else {
			/*
			 * 否則看看當前領導有木有上級
			 */
			if (null != mSuperiorLeader) {
				// 有的話簡單撒直接扔給上級處理即可
				mSuperiorLeader.handleRequest(ape);
			} else {
				// 沒有上級的話就批覆不了老……不過在這個場景中總會有領導批覆的淡定
				System.out.println("Goodbye my money......");
			}
		}
	}

	/**
	 * 為當前領導設置一個上級領導
	 * 
	 * @param superiorLeader
	 *            上級領導
	 */
	public void setLeader(Leader superiorLeader) {
		this.mSuperiorLeader = superiorLeader;
	}
}
```

這麼一來，我們的領導老大們就有了實實在在的權利職責去處理底層苦逼程序猿的請求。OK，接下來要做的事就是讓所有的領導繼承該類：

```Java
/**
 * 小組長類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class GroupLeader extends Leader {

	public GroupLeader() {
		super(1000);
	}

	@Override
	protected void reply(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("GroupLeader: Of course Yes!");
	}
}
```

```java
/**
 * 項目主管類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Director extends Leader{
	public Director() {
		super(5000);
	}

	@Override
	protected void reply(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Director: Of course Yes!");		
	}
}
```

```java
/**
 * 部門經理類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Manager extends Leader {
	public Manager() {
		super(10000);
	}

	@Override
	protected void reply(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Manager: Of course Yes!");
	}
}
```

```java
/**
 * 老總類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Boss extends Leader {
	public Boss() {
		super(40000);
	}

	@Override
	protected void reply(ProgramApe ape) {
		System.out.println(ape.getApply());
		System.out.println("Boss: Of course Yes!");
	}
}
```

最後，更新我們的場景類，將其從責任人的角色中解放出來：

```java
/**
 * 場景模擬類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Client {
	public static void main(String[] args) {
		/*
		 * 先來一個程序猿 這裡給他一個三萬以內的隨機值表示需要申請的差旅費
		 */
		ProgramApe ape = new ProgramApe((int) (Math.random() * 30000));

		/*
		 * 再來四個老大
		 */
		Leader leader = new GroupLeader();
		Leader director = new Director();
		Leader manager = new Manager();
		Leader boss = new Boss();

		/*
		 * 設置老大的上一個老大
		 */
		leader.setLeader(director);
		director.setLeader(manager);
		manager.setLeader(boss);

		// 處理申請
		leader.handleRequest(ape);
	}
}
```

運行三次，下面是三次運行的結果（注：由於隨機值的原因你的結果也許與我不一樣）：

>爹要點錢出差
>
>Boss: Of course Yes!
***
>爹要點錢出差
>
>Director: Of course Yes!
***
>爹要點錢出差
>
>Boss: Of course Yes!

### 總結

OK，這樣我們就將請求和處理分離開來，對於程序猿來說，不需要知道是誰給他批覆的錢，而對於領導們來說，也不需要確切地知道是批給哪個程序猿，只要根據自己的責任做出處理即可，由此將兩者優雅地解耦。

## Android源碼中的模式實現
Android中關於責任鏈模式比較明顯的體現就是在事件分發過程中對事件的投遞，其實嚴格來說，事件投遞的模式並不是嚴格的責任鏈模式，但是其是責任鏈模式的一種變種體現，在ViewGroup中對事件處理者的查找方式如下：

```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 省略兩行代碼…………

    boolean handled = false;
    if (onFilterTouchEventForSecurity(ev)) {

        // 省略N行代碼…………

        /*
         * 如果事件未被取消並未被攔截
         */
        if (!canceled && !intercepted) {
        	/*
         	 * 如果事件為起始事件
         	 */
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {

                // 省掉部分邏輯…………

                final int childrenCount = mChildrenCount;

                /*
         		 * 如果TouchTarget為空並且子元素不為0
         		 */
                if (newTouchTarget == null && childrenCount != 0) {
                    final float x = ev.getX(actionIndex);
                    final float y = ev.getY(actionIndex);

                    final View[] children = mChildren;

                    final boolean customOrder = isChildrenDrawingOrderEnabled();

                   /*
         		 	* 遍歷子元素
         		 	*/
                    for (int i = childrenCount - 1; i >= 0; i--) {
                        final int childIndex = customOrder ?
                                getChildDrawingOrder(childrenCount, i) : i;
                        final View child = children[childIndex];

                       /*
         		 		* 如果這個子元素無法接收Pointer Event或這個事件點壓根就沒有落在子元素的邊界範圍內
         		 		*/
                        if (!canViewReceivePointerEvents(child)
                                || !isTransformedTouchPointInView(x, y, child, null)) {
                            // 那麼就跳出該次循環繼續遍歷
                            continue;
                        }

                        // 找到Event該由哪個子元素持有
                        newTouchTarget = getTouchTarget(child);


                        if (newTouchTarget != null) {
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;
                        }

                        resetCancelNextUpFlag(child);

                       /*
         		 		* 投遞事件執行觸摸操作
         		 		* 如果子元素還是一個ViewGroup則遞歸調用重複此過程
         		 		* 如果子元素是一個View那麼則會調用View的dispatchTouchEvent並最終由onTouchEvent處理
         		 		*/
                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                            mLastTouchDownTime = ev.getDownTime();
                            mLastTouchDownIndex = childIndex;
                            mLastTouchDownX = ev.getX();
                            mLastTouchDownY = ev.getY();
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            break;
                        }
                    }
                }

               /*
 		 		* 如果發現沒有子元素可以持有該次事件
 		 		*/
                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                }
            }
        }

        // 省去不必要代碼……
    }

    // 省去一行代碼……

    return handled;
}
```

再來看看dispatchTransformedTouchEvent方法是如何調度子元素dispatchTouchEvent方法的：

```java
private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
        View child, int desiredPointerIdBits) {
    final boolean handled;

    final int oldAction = event.getAction();

    /*
     * 如果事件被取消
     */
    if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        event.setAction(MotionEvent.ACTION_CANCEL);

       /*
     	* 如果沒有子元素
     	*/
        if (child == null) {
        	// 那麼就直接調用父類的dispatchTouchEvent注意這裡的父類終會為View類
            handled = super.dispatchTouchEvent(event);
        } else {
        	// 如果有子元素則傳遞cancle事件
            handled = child.dispatchTouchEvent(event);
        }
        event.setAction(oldAction);
        return handled;
    }

    /*
     * 計算即將被傳遞的點的數量
     */
    final int oldPointerIdBits = event.getPointerIdBits();
    final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

    /*
     * 如果事件木有相應的點那麼就丟棄該次事件
     */
    if (newPointerIdBits == 0) {
        return false;
    }

    // 聲明臨時變量保存座標轉換後的MotionEvent
    final MotionEvent transformedEvent;

    /*
     * 如果事件點的數量一致
     */
    if (newPointerIdBits == oldPointerIdBits) {
        /*
    	 * 子元素為空或子元素有一個單位矩陣
    	 */
        if (child == null || child.hasIdentityMatrix()) {
            /*
    		 * 再次區分子元素為空的情況
    		 */
            if (child == null) {
            	// 為空則調用父類dispatchTouchEvent
                handled = super.dispatchTouchEvent(event);
            } else {
            	// 否則嘗試獲取xy方向上的偏移量（如果通過scrollTo或scrollBy對子視圖進行滾動的話）
                final float offsetX = mScrollX - child.mLeft;
                final float offsetY = mScrollY - child.mTop;

                // 將MotionEvent進行座標變換
                event.offsetLocation(offsetX, offsetY);

                // 再將變換後的MotionEvent傳遞給子元素
                handled = child.dispatchTouchEvent(event);

                // 復位MotionEvent以便之後再次使用
                event.offsetLocation(-offsetX, -offsetY);
            }

            // 如果通過以上的邏輯判斷當前事件被持有則可以直接返回
            return handled;
        }
        transformedEvent = MotionEvent.obtain(event);
    } else {
        transformedEvent = event.split(newPointerIdBits);
    }

    /*
     * 下述雷同不再累贅
     */
    if (child == null) {
        handled = super.dispatchTouchEvent(transformedEvent);
    } else {
        final float offsetX = mScrollX - child.mLeft;
        final float offsetY = mScrollY - child.mTop;
        transformedEvent.offsetLocation(offsetX, offsetY);
        if (! child.hasIdentityMatrix()) {
            transformedEvent.transform(child.getInverseMatrix());
        }

        handled = child.dispatchTouchEvent(transformedEvent);
    }

    transformedEvent.recycle();
    return handled;
}
```

ViewGroup事件投遞的遞歸調用就類似於一條責任鏈，一旦其尋找到責任者，那麼將由責任者持有並消費掉該次事件，具體的體現在View的onTouchEvent方法中返回值的設置（這裡介於篇幅就不具體介紹ViewGroup對事件的處理了），如果onTouchEvent返回false那麼意味著當前View不會是該次事件的責任人將不會對其持有，如果為true則相反，此時View會持有該事件並不再向外傳遞。

## 4. 雜談
世界不是完美的，所以不會有完美的事物存在。就像所有的設計模式一樣，  有優點優缺點，但是總的來說優點必定大於缺點或者說缺點相對於優點來說更可控。責任鏈模式也一樣，有點顯而易見，可以對請求者和處理者關係的解耦提高代碼的靈活性，比如上面我們的例子中如果在主管和經理之間多了一個總監，那麼總監可以批覆小於7500的經費，這時候根據我們上面重構的模式，僅需新建一個總監類繼承Leader即可其它所有的存在類都可保持不變。責任鏈模式的最大缺點是對鏈中責任人的遍歷，如果責任人太多那麼遍歷必定會影響性能，特別是在一些遞歸調用中，要慎重。

