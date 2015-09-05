Android設計模式源碼解析之工廠方法模式
====================================
> 本文為 [Android 設計模式源碼解析](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis) 中工廠方法模式分析  
> Android系統版本：4.4.4       
> 分析者：[Aige](https://github.com/AigeStudio)，分析狀態：未完成，校對者：[Mr.Simple](https://github.com/bboyfeiyu)，校對狀態：未開始   

## 1. 模式介紹  
 
###  模式的定義
你要什麼工廠造給你就是了，你不用管我是怎麼造的，造好你拿去用就是了，奏是介麼任性。

### 模式的使用場景
任何需要生成對象的情況都可使用工廠方法替代生成。

## 2. UML類圖
![](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis/blob/master/factory-method/AigeStudio/images/factory-method.jpg?raw=true)

### 角色介紹
如圖

## 3. 模式的簡單實現
###  簡單實現的介紹
工廠方法相對來說比較簡單也很常用，如上所說，任何需要生成對象的情況都可使用工廠方法替代生成。我們知道在Java中生成對象一般會使用關鍵字new：

```java
Client client = new Client();
```

如果使用工廠方法來替代，我們則可以先聲明一個工廠類：

```java
/**
 * 工廠類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Factory {
	public static Client getClient() {
		return new Client();
	}
}
```

然後呢就可以使用這個工廠類來生成Client對象：

```java
Client client = Factory.getClient();
```

但是，相信即便是新手也會覺得這套路感覺不對啊是吧，生成個對象居然這麼麻煩我也是醉了。So，我們極少這麼使用，生成某個具體的對象直接new就是了對吧。那好，假定我們這有三個類：香蕉類、黃瓜類、甘蔗類，我們將其統稱為水果類，額某種意義上來說黃瓜也算是水果，為其定義一個抽象父類：

```java
/**
 * 抽象水果類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public abstract class Fruits {
	/**
	 * 水果的顏色
	 */
	public abstract void color();

	/**
	 * 水果的重量
	 */
	public abstract void weight();
}
```

然後呢則是各個具體的水果類：

```java
/**
 * 香蕉類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Banana extends Fruits {
	@Override
	public void color() {
		System.out.println("Banana is red");
	}

	@Override
	public void weight() {
		System.out.println("Weight 0.3kg");
	}
}
```

```java
/**
 * 黃瓜類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Cucumber extends Fruits {
	@Override
	public void color() {
		System.out.println("Cucumber is green");
	}

	@Override
	public void weight() {
		System.out.println("Weight 0.5kg");
	}
}
```

```java
/**
 * 甘蔗類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Sugarcane extends Fruits {
	@Override
	public void color() {
		System.out.println("Sugarcane is purple");
	}

	@Override
	public void weight() {
		System.out.println("Weight 1.3kg");
	}
}
```

最後，上場景：

```java
/**
 * 場景模擬類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Client {
	public static void main(String[] args) {
		Fruits banana = new Banana();
		banana.color();
		banana.weight();
		
		Fruits cucumber = new Cucumber();
		cucumber.color();
		cucumber.weight();
		
		Fruits sugarcane = new Sugarcane();
		sugarcane.color();
		sugarcane.weight();
	}
}
```

具體的運行結果就不貼了，不用運行你也該知道是啥結果 = = 。這樣一段代碼其實已經算過得去了，抽象有了具現也有，也確實沒啥問題對吧，可是仔細想想每個類我們都要通過具體的類去new生成，能不能進一步解耦將生成具體對象的工作交由第三方去做呢？答案是肯定的，我們來定義一個果農類，你要啥水果給果農說一聲，讓它給你就是了：

```java
/**
 * 果農類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Grower {
	public <T extends Fruits> T getFruits(Class<T> clz) {
		try {
			Fruits fruits = (Fruits) Class.forName(clz.getName()).newInstance();
			return (T) fruits;
		} catch (Exception e) {
			return null;
		}
	}
}
```

這樣一來，我們要什麼水果直接跟果農報個名（Class<T> clz），然後果農給你就是了，OK，修改下場景類：

```java
/**
 * 場景模擬類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Client {
	public static void main(String[] args) {
		Grower grower = new Grower();

		Fruits banana = grower.getFruits(Banana.class);
		banana.color();
		banana.weight();

		Fruits cucumber = grower.getFruits(Cucumber.class);
		cucumber.color();
		cucumber.weight();

		Fruits sugarcane = grower.getFruits(Sugarcane.class);
		sugarcane.color();
		sugarcane.weight();
	}
}
```

具體的水果類不用變，運行結果什麼的就不扯了，自己去試。恩，這樣就差不多，更進一步地，我們可以考慮將果農類進一步抽象，在抽象類中定義方法：

```java
/**
 * 抽象果農類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public abstract class AGrower {
	/**
	 * 獲取水果
	 * 
	 * @param clz
	 *            具體水果類型
	 * @return 具體水果的對象
	 */
	public abstract <T extends Fruits> T getFruits(Class<T> clz);
}
```

然後讓Grower extends AGrower就行了其他不變，最終的結構就與上面我們給出的UML類圖一致了，香蕉、甘蔗、黃瓜什麼的為具體的產品類而水果則作為產品類的抽象，Grower和AGrower的關係亦如此。然而很多時候其實我們沒必要多個工廠類，一個足以：

```java
/**
 * 果農類
 * 
 * @author Aige{@link https://github.com/AigeStudio}
 *
 */
public class Grower {
	public static <T extends Fruits> T getFruits(Class<T> clz) {
		try {
			Fruits fruits = (Fruits) Class.forName(clz.getName()).newInstance();
			return (T) fruits;
		} catch (Exception e) {
			return null;
		}
	}
}
```

如代碼所示，去掉了抽象父類的繼承使getFruits變為靜態方法在Client中直接調用不再生成類了。具體代碼就不貼了。

### 實現源碼
`上述案例的源碼實現`


### 總結
`對上述的簡單示例進行總結說明`

  


## Android源碼中的模式實現
`分析源碼中的模式實現，列出相關源碼，以及使用該模式原因等`  


 

## 4. 雜談
該模式的優缺點以及自己的一些感悟，非所有項目必須。  



`寫完相關內容之後到開發群告知管理員，管理員安排相關人員進行審核，審核通過之後即可。`  

