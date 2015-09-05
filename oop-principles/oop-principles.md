公共技術點之面向對象六大原則
---------

## 概述
在工作初期，我們可能會經常會有這樣的感覺，自己的代碼接口設計混亂、代碼耦合較為嚴重、一個類的代碼過多等等，自己回頭看的時候都覺得汗顏。再看那些知名的開源庫，它們大多有著整潔的代碼、清晰簡單的接口、職責單一的類，這個時候我們通常會捶胸頓足而感嘆：什麼時候老夫才能寫出這樣的代碼！     

在做開發的這些年中，我漸漸的感覺到，其實國內的一些初、中級工程師寫的東西不規範或者說不夠清晰的原因是缺乏一些指導原則。他們手中揮舞著面向對象的大旗，寫出來的東西卻充斥著面向過程的氣味。也許是他們不知道有這些原則，也許是他們知道但是不能很好運用到實際代碼中，亦或是他們沒有在實戰項目中體會到這些原則能夠帶來的優點，以至於他們對這些原則並沒有足夠的重視。     

今天，我們就是以剖析優秀的Android網絡框架Volley為例來學習這六大面向對象的基本原則，體會它們帶來的強大能量。         

在此之前，有一點需要大家知道，熟悉這些原則不會讓你寫出優秀的代碼，只是為你的優秀代碼之路鋪上了一層柵欄，在這些原則的指導下你才能避免陷入一些常見的代碼泥沼，從而讓你專心寫出優秀的東西。另外，我是個新人，以下只是我個人的觀點。如果你覺得還行，可以頂個帖支持一下；如果你覺得它爛透了，那請分享你的經驗。         


## 一、單一職責原則 （ Single Responsibility   Principle ）

### 1.1 簡述
單一職責原則的英文名稱是Single Responsibility Principle，簡稱是SRP，簡單來說一個類只做一件事。這個設計原則備受爭議卻又及其重要的原則。只要你想和別人爭執、慪氣或者是吵架，這個原則是屢試不爽的。因為單一職責的劃分界限並不是如馬路上的行車道那麼清晰，很多時候都是需要靠個人經驗來界定。當然最大的問題就是對職責的定義，什麼是類的職責，以及怎麼劃分類的職責。      
     
試想一下，如果你遵守了這個原則，那麼你的類就會劃分得很細，每個類都有自己的職責。恩，這不就是高內聚、低耦合麼！ 當然，如何界定類的職責這需要你的個人經驗了。      


### 1.2 示例

在Volley中，我覺得很能夠體現SRP原則的就是HttpStack這個類族了。HttpStack定義了一個執行網絡請求的接口，代碼如下 : 

```java
/**
 * An HTTP stack abstraction.
 */
public interface HttpStack {
    /**
     * 執行Http請求,並且返回一個HttpResponse
     */ 
    public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
        throws IOException, AuthFailureError;

}
```    
可以看到，HttpStack只有一個函數，清晰明瞭，它的職責就是執行網絡請求並且返回一個Response。它的職責很單一，這樣在需要修改執行網絡請求的相關代碼時我們只需要修改實現HttpStack接口的類，而不會影響其它的類的代碼。如果某個類的職責包含有執行網絡請求、解析網絡請求、進行gzip壓縮、封裝請求參數等等，那麼在你修改某處代碼時你就必須謹慎，以免修改的代碼影響了其它的功能。但是當職責單一的時候，你修改的代碼能夠基本上不影響其它的功能。這就在一定程度上保證了代碼的可維護性。**注意，單一職責原則並不是說一個類只有一個函數，而是說這個類中的函數所做的工作必須要是高度相關的，也就是高內聚**。HttpStack抽象了執行網絡請求的具體過程，接口簡單清晰，也便於擴展。     
     
我們知道，Api 9以下使用HttpClient執行網絡請求會好一些，api 9及其以上則建議使用HttpURLConnection。這就需要執行網絡請求的具體實現能夠可擴展、可替換，因此我們對於執行網絡請求這個功能必須要抽象出來，HttpStack就是這個職責的抽象。


### 1.3 優點
*    類的複雜性降低，實現什麼職責都有清晰明確的定義； 
*    可讀性提高，複雜性降低，那當然可讀性提高了； 
*    可維護性提高，可讀性提高，那當然更容易維護了； 
*    變更引起的風險降低，變更是必不可少的，如果接口的單一職責做得好，一個接口修改只對相應的實現類有影響，對其他的接口無影響，這對系統的擴展性、維護性都有非常大的幫助。




## 里氏替換原則 （ Liskov Substitution Principle）

### 2.1 簡述
面向對象的語言的三大特點是繼承、封裝、多態，里氏替換原則就是依賴於繼承、多態這兩大特性。里氏替換原則簡單來說就是所有引用基類的地方必須能透明地使用其子類的對象。通俗點講，只要父類能出現的地方子類就可以出現，而且替換為子類也不會產生任何錯誤或異常，使用者可能根本就不需要知道是父類還是子類。但是，反過來就不行了，有子類出現的地方，父類未必就能適應。     


### 2.2 示例
還是以HttpStack為例，Volley定義了HttpStack來表示執行網絡請求這個抽象概念。在執行網絡請求時，我們只需要定義一個HttpStack對象，然後調用performRequest即可。至於HttpStack的具體實現由更高層的調用者給出。示例如下 :       

```java

    public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

        String userAgent = "volley/0";
  		// 代碼省略
		// 1、構造HttpStack對象
        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                stack = new HurlStack();
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
            }
        }
		// 2、將HttpStack對象傳遞給Network對象
        Network network = new BasicNetwork(stack);
		// 3、將network對象傳遞給網絡請求隊列
        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();

        return queue;
    }
```   

BasicNetwork的代碼如下:      

```java
/**
 * A network performing Volley requests over an {@link HttpStack}.
 */
public class BasicNetwork implements Network {
	// HttpStack抽象對象
    protected final HttpStack mHttpStack;

    protected final ByteArrayPool mPool;

    public BasicNetwork(HttpStack httpStack) {

        this(httpStack, new ByteArrayPool(DEFAULT_POOL_SIZE));
    }
    
    
    public BasicNetwork(HttpStack httpStack, ByteArrayPool pool) {
        mHttpStack = httpStack;
        mPool = pool;
    }
}
```       

上述代碼中，BasicNetwork構造函數依賴的是HttpStack抽象接口，任何實現了HttpStack接口的類型都可以作為參數傳遞給BasicNetwork用以執行網絡請求。這就是所謂的里氏替換原則，任何父類出現的地方子類都可以出現，這不就保證了可擴展性嗎？！ 此時，用手撐著你的小腦門作思考狀...  任何實現HttpStack接口的類的對象都可以傳遞給BasicNetwork實現網絡請求的功能，這樣Volley執行網絡請求的方法就有很多種可能性，而不是隻有HttpClient和HttpURLConnection。       
  
喔，原來是這樣！此時我們放下裝逼的模樣，呈現一副若有所得的樣子。細想一下，框架可不就是這樣嗎？ 框架不就是定義一系列相關的邏輯骨架與抽象，使得用戶可以將自己的實現傳遞進給框架，從而實現變化萬千的功能。    

### 2.3 優點
*    代碼共享，減少創建類的工作量，每個子類都擁有父類的方法和屬性；
*    提高代碼的重用性； 
*    提高代碼的可擴展性，實現父類的方法就可以“為所欲為”了，很多開源框架的擴展接口都是通過繼承父類來完成的； 
*    提高產品或項目的開放性。

### 2.4 缺點
*    繼承是侵入性的。只要繼承，就必須擁有父類的所有屬性和方法；
*    降低代碼的靈活性。子類必須擁有父類的屬性和方法，讓子類自由的世界中多了些約束；
*    增強了耦合性。當父類的常量、變量和方法被修改時，必需要考慮子類的修改，而且在缺乏規範的環境下，這種修改可能帶來非常糟糕的結果——大片的代碼需要重構。


##  依賴倒置原則 （Dependence Inversion Principle）

### 3.1 簡述
依賴倒置原則這個名字看著有點不好理解，“依賴”還要“倒置”，這到底是什麼意思？       
依賴倒置原則的幾個關鍵點如下:      

*    高層模塊不應該依賴低層模塊，兩者都應該依賴其抽象；
*    抽象不應該依賴細節； 
*    細節應該依賴抽象。       

在Java語言中，抽象就是指接口或抽象類，兩者都是不能直接被實例化的；細節就是實現類，實現接口或繼承抽象類而產生的類就是細節，其特點就是可以直接被實例化，也就是可以加上一個關鍵字 new 產生一個對象。依賴倒置原則在 Java 語言中的表現就是：**模塊間的依賴通過抽象發生，實現類之間不發生直接的依賴關係，其依賴關係是通過接口或抽象類產生的。**軟件先驅們總是喜歡將一些理論定義得很抽象，弄得神不楞登的，其實就是一句話:面向接口編程，或者說是面向抽象編程，這裡的抽象指的是接口或者抽象類。面向接口編程是面向對象精髓之一。


### 3.2 示例
採用依賴倒置原則可以減少類間的耦合性，提高系統的穩定性，降低並行開發引起的風險，提高代碼的可讀性和可維護性。      
第二章節中的BasicNetwork實現類依賴於HttpStack接口( 抽象 )，而不依賴於HttpClientStack與HurlStack實現類 ( 細節 )，這就是典型的依賴倒置原則的體現。假如BasicNetwork直接依賴了HttpClientStack，那麼HurlStack就不能傳遞給了，除非HurlStack繼承自HttpClientStack。但這麼設計明顯不符合邏輯，HurlStack與HttpClientStack並沒有任何的is-a的關係，而且即使有也不能這麼設計，因為HttpClientStack是一個具體類而不是抽象，如果HttpClientStack作為BasicNetwork構造函數的參數，那麼以為這後續的擴展都需要繼承自HttpClientStack。這簡直是一件不可忍受的事了！        


### 3.3 優點
*   可擴展性好；
*   耦合度低；



## 開閉原則 （ Open-Close Principle ）

### 4.1 簡述
開閉原則是 Java 世界裡最基礎的設計原則，它指導我們如何建立一個穩定的、靈活的系統。開閉原則的定義是 : 一個軟件實體如類、模塊和函數應該對擴展開放，對修改關閉。在軟件的生命週期內，因為變化、升級和維護等原因需要對軟件原有代碼進行修改時，可能會給舊代碼中引入錯誤。因此當軟件需要變化時，我們應該儘量通過擴展的方式來實現變化，而不是通過修改已有的代碼來實現。    


### 4.2 示例

在軟件開發過程中，永遠不變的就是變化。開閉原則是使我們的軟件系統擁抱變化的核心原則之一。對擴展可放，對修改關閉給出了高層次的概括，即在需要對軟件進行升級、變化時應該通過擴展的形式來實現，而非修改原有代碼。當然這只是一種比較理想的狀態，是通過擴展還是通過修改舊代碼需要根據代碼自身來定。      

在Volley中，開閉原則體現得比較好的是Request類族的設計。我們知道，在開發C/S應用時，服務器返回的數據格式多種多樣，有字符串類型、xml、json等。而解析服務器返回的Response的原始數據類型則是通過Request類來實現的，這樣就使得Request類對於服務器返回的數據格式有良好的擴展性，即Request的可變性太大。

例如我們返回的數據格式是Json，那麼我們使用JsonObjectRequest請求來獲取數據，它會將結果轉成JsonObject對象，我們看看JsonObjectRequest的核心實現。     

```java
/**
 * A request for retrieving a {@link JSONObject} response body at a given URL, allowing for an
 * optional {@link JSONObject} to be passed in as part of the request body.
 */
public class JsonObjectRequest extends JsonRequest<JSONObject> {

   // 代碼省略
    @Override
    protected Response<JSONObject> parseNetworkResponse(NetworkResponse response) {
        try {
            String jsonString =
                new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            return Response.success(new JSONObject(jsonString),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JSONException je) {
            return Response.error(new ParseError(je));
        }
    }
}
```       
JsonObjectRequest通過實現Request抽象類的parseNetworkResponse解析服務器返回的結果，這裡將結果轉換為JSONObject，並且封裝到Response類中。     

例如Volley添加對圖片請求的支持，即ImageLoader( 已內置 )。這個時候我的請求返回的數據是Bitmap圖片。因此我需要在該類型的Request得到的結果是Request，但支持一種數據格式不能通過修改源碼的形式，這樣可能會為舊代碼引入錯誤。但是你又需要支持新的數據格式，此時我們的開閉原則就很重要了，對擴展開放，對修改關閉。我們看看Volley是如何做的。        

```java 

/**
 * A canned request for getting an image at a given URL and calling
 * back with a decoded Bitmap.
 */
public class ImageRequest extends Request<Bitmap> {
	// 代碼省略

	// 將結果解析成Bitmap，並且封裝套Response對象中
    @Override
    protected Response<Bitmap> parseNetworkResponse(NetworkResponse response) {
        // Serialize all decode on a global lock to reduce concurrent heap usage.
        synchronized (sDecodeLock) {
            try {
                return doParse(response);
            } catch (OutOfMemoryError e) {
                VolleyLog.e("Caught OOM for %d byte image, url=%s", response.data.length, getUrl());
                return Response.error(new ParseError(e));
            }
        }
    }

    /**
     * The real guts of parseNetworkResponse. Broken out for readability.
     */
    private Response<Bitmap> doParse(NetworkResponse response) {
        byte[] data = response.data;
        BitmapFactory.Options decodeOptions = new BitmapFactory.Options();
        Bitmap bitmap = null;
        if (mMaxWidth == 0 && mMaxHeight == 0) {
            decodeOptions.inPreferredConfig = mDecodeConfig;
            bitmap = BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
        } else {
            // If we have to resize this image, first get the natural bounds.
            decodeOptions.inJustDecodeBounds = true;
            BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
            int actualWidth = decodeOptions.outWidth;
            int actualHeight = decodeOptions.outHeight;

            // Then compute the dimensions we would ideally like to decode to.
            int desiredWidth = getResizedDimension(mMaxWidth, mMaxHeight,
                    actualWidth, actualHeight);
            int desiredHeight = getResizedDimension(mMaxHeight, mMaxWidth,
                    actualHeight, actualWidth);

            // Decode to the nearest power of two scaling factor.
            decodeOptions.inJustDecodeBounds = false;
            // TODO(ficus): Do we need this or is it okay since API 8 doesn't support it?
            // decodeOptions.inPreferQualityOverSpeed = PREFER_QUALITY_OVER_SPEED;
            decodeOptions.inSampleSize =
                findBestSampleSize(actualWidth, actualHeight, desiredWidth, desiredHeight);
            Bitmap tempBitmap =
                BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);

            // If necessary, scale down to the maximal acceptable size.
            if (tempBitmap != null && (tempBitmap.getWidth() > desiredWidth ||
                    tempBitmap.getHeight() > desiredHeight)) {
                bitmap = Bitmap.createScaledBitmap(tempBitmap,
                        desiredWidth, desiredHeight, true);
                tempBitmap.recycle();
            } else {
                bitmap = tempBitmap;
            }
        }

        if (bitmap == null) {
            return Response.error(new ParseError(response));
        } else {
            return Response.success(bitmap, HttpHeaderParser.parseCacheHeaders(response));
        }
    }
}
```    
需要添加某種數據格式的Request時，只需要繼承自Request類，並且實現相應的方法即可。這樣通過擴展的形式來應對軟件的變化或者說用戶需求的多樣性，即避免了破壞原有系統，又保證了軟件系統的可擴展性。     

### 4.3 優點
*   增加穩定性；
*   可擴展性高。


## 接口隔離原則 （Interface Segregation Principle）

### 5.1 簡述
客戶端不應該依賴它不需要的接口；一個類對另一個類的依賴應該建立在最小的接口上。根據接口隔離原則，當一個接口太大時，我們需要將它分割成一些更細小的接口，使用該接口的客戶端僅需知道與之相關的方法即可。    

可能描述起來不是很好理解，我們還是以示例來加強理解吧。       


### 5.2 示例
我們知道，在Volley的網絡隊列中是會對請求進行排序的。Volley內部使用PriorityBlockingQueue來維護網絡請求隊列，PriorityBlockingQueue需要調用Request類的compareTo函數來進行排序。試想一下，PriorityBlockingQueue其實只需要調用Request類的排序方法就可以了，其他的接口它根本不需要，即PriorityBlockingQueue只需要compareTo這個接口，而這個compareTo方法就是我們上述所說的最小接口。當然compareTo這個方法並不是Volley本身定義的接口方法，而是Java中的Comparable<T>接口，但我們這裡只是為了學習本身，至於哪裡定義的無關緊要。

```java
public abstract class Request<T> implements Comparable<Request<T>> {

    /**
     * 排序方法,PriorityBlockingQueue只需要調用元素的compareTo即可進行排序
     */
    @Override
    public int compareTo(Request<T> other) {
        Priority left = this.getPriority();
        Priority right = other.getPriority();

        // High-priority requests are "lesser" so they are sorted to the front.
        // Equal priorities are sorted by sequence number to provide FIFO ordering.
        return left == right ?
                this.mSequence - other.mSequence :
                right.ordinal() - left.ordinal();
    }
}
```    

PriorityBlockingQueue類相關代碼 :     

```java 

public class PriorityBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {
    
    // 代碼省略
    
    	// 添加元素的時候進行排序
        public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        final ReentrantLock lock = this.lock;
        lock.lock();
        int n, cap;
        Object[] array;
        while ((n = size) >= (cap = (array = queue).length))
            tryGrow(array, cap);
        try {
            Comparator<? super E> cmp = comparator;
            // 沒有設置Comparator則使用元素本身的compareTo方法進行排序
            if (cmp == null)
                siftUpComparable(n, e, array);
            else
                siftUpUsingComparator(n, e, array, cmp);
            size = n + 1;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
        return true;
    }
    
    private static <T> void siftUpComparable(int k, T x, Object[] array) {
        Comparable<? super T> key = (Comparable<? super T>) x;
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = array[parent];
            // 調用元素的compareTo方法進行排序
            if (key.compareTo((T) e) >= 0)
                break;
            array[k] = e;
            k = parent;
        }
        array[k] = key;
    }
 }
```     
從PriorityBlockingQueue的代碼可知,在元素排序時，PriorityBlockingQueue只需要知道元素是個Comparable對象即可，不需要知道這個對象是不是Request類以及這個類的其他接口。它只需要排序，因此我只要知道它是實現了Comparable接口的對象即可，Comparable就是它的最小接口，也是通過Comparable隔離了PriorityBlockingQueue類對Request類的其他方法的可見性。妙哉，妙哉！      

### 5.3 優點
*   降低耦合性；
*   提升代碼的可讀性；
*   隱藏實現細節。


## 迪米特原則 （ Law of Demeter ）
### 6.1 簡述
迪米特法則也稱為最少知識原則（Least Knowledge Principle），雖然名字不同，但描述的是同一個原則：一個對象應該對其他對象有最少的瞭解。通俗地講，一個類應該對自己需要耦合或調用的類知道得最少，這有點類似接口隔離原則中的最小接口的概念。類的內部如何實現、如何複雜都與調用者或者依賴者沒關係，調用者或者依賴者只需要知道他需要的方法即可，其他的我一概不關心。類與類之間的關係越密切，耦合度越大，當一個類發生改變時，對另一個類的影響也越大。            

迪米特法則還有一個英文解釋是: Only talk to your immedate friends（ 只與直接的朋友通信。）什麼叫做直接的朋友呢？每個對象都必然會與其他對象有耦合關係，兩個對象之間的耦合就成為朋友關係，這種關係的類型有很多，例如組合、聚合、依賴等。


### 6.2 示例
例如，Volley中的Response緩存接口的設計。

```java
/**
 * An interface for a cache keyed by a String with a byte array as data.
 */
public interface Cache {
    /**
     * 獲取緩存
     */
    public Entry get(String key);

    /**
     * 添加一個緩存元素
     */
    public void put(String key, Entry entry);

    /**
     * 初始化緩存
     */
    public void initialize();

    /**
     * 標識某個緩存過期
     */
    public void invalidate(String key, boolean fullExpire);

    /**
     * 移除緩存
     */
    public void remove(String key);

    /**
     * 清空緩存
     */
    public void clear();

}
```    
Cache接口定義了緩存類需要實現的最小接口，依賴緩存類的對象只需要知道這些接口即可。例如緩存的具體實現類DiskBasedCache，該緩存類將Response序列化到本地,這就需要操作File以及相關的類。代碼如下 :       

```java

public class DiskBasedCache implements Cache {

    /** Map of the Key, CacheHeader pairs */
    private final Map<String, CacheHeader> mEntries =
            new LinkedHashMap<String, CacheHeader>(16, .75f, true);

    /** The root directory to use for the cache. */
    private final File mRootDirectory;

    public DiskBasedCache(File rootDirectory, int maxCacheSizeInBytes) {
        mRootDirectory = rootDirectory;
        mMaxCacheSizeInBytes = maxCacheSizeInBytes;
    }

    public DiskBasedCache(File rootDirectory) {
        this(rootDirectory, DEFAULT_DISK_USAGE_BYTES);
    }

	// 代碼省略
}
```     
在這裡，Volley的直接朋友就是DiskBasedCache，間接朋友就是mRootDirectory、mEntries等。Volley只需要直接和Cache類交互即可，並不需要知道File、mEntries等對象的存在。這就是迪米特原則，儘量少的知道對象的信息，只與直接的朋友交互。       
     

### 6.3 優點
*   降低複雜度；
*   降低耦合度；
*   增加穩定性。


## 雜談 
面向對象六大原則在開發過程中極為重要，如果能夠很好地將這些原則運用到項目中，再在一些合適的場景運用一些前人驗證過的模式，那麼開發出來的軟件在一定程度上能夠得到質量保證。其實稍微一想，這幾大原則最終就化為這麼幾個關鍵詞: 抽象、單一職責、最小化。那麼在實際開發過程中如何權衡、實踐這些原則，筆者也在不斷地學習、摸索。我想學習任何的事物莫過於實踐、經驗與領悟，在這個過程中希望能夠與大家分享知識、共同進步。
