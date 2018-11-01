# 第一條： 考慮使用靜態工廠方法(static factory method)，替代建構函式(constructor)

### 所謂的**靜態工廠方法 (static factory method)**，就是指一個類別提供一個靜態方法來返回這個類別的實例(物件)。

比如說 ，像是`Boolean`類別的提供的
```java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

## 使用靜態工廠方法的有以下優勢：

### 靜態工廠方法有名字

建構函式一般來說很難說明回傳的物件跟輸入的參數的關係。透過定義良好的靜態工廠方法名稱，使它容易使用且客戶端程式也容易閱讀。


### 靜態工廠方法不受建構子單一簽名的限制

Java的方法(Method)有單一簽名的限制，當然建構函式也不意外。透過特製化各個靜態工廠方法名稱，即使輸入參數一樣，還是可以生成物件。

>簽名 = 方法名稱 + 輸入參數


### 靜態工廠方法可以不用每次都返回新的物件

當我們可以減少創建物件的花費，效能就可以提昇。


### 靜態工廠方法可以自由的回傳子類別的實體

這提供我們彈性來選擇回傳的實體。


### 靜態工廠方法可以減少創建參數型別的實體的冗長敘述

比如說當我們需要一個`Map`的結構
```java
// 實體化一個 Map 結構
Map<String, List<String>> m = 
	new HashMap<String, List<String>>();
```
可以看到 `new HashMap<String, List<String>>()` 這段很冗長，如果今天輸入參數不同，變成`Integer`變成`Long`型別，就會變成`new HashMap<Integer, List<Integer>>()`&`new HashMap<Long, List<Long>>()`，變得更冗長。


改成**靜態工廠方法**

```java
class MapUtil{
	public static <K, V> HashMap<K, V> newInstance() {
		return new HashMap<K, V>();
	}
}

Map<String, List<String>> m = MapUtil.newInstance();
```

## 使用靜態工廠方法的缺點：

### 當只允許靜態工廠方法生成實例時，會使此類別無法被繼承

不過這也變現鼓勵開發者**使用組成(composition)替代繼承**。(第16條)


### 靜態工廠方法其實不是這麼容易跟其他靜態方法做區別

我們可以用一些常見**方法命名方式**來命名靜態工廠方法

- `valueOf` ： 回傳的物件跟傳入參數有相同值
- `of` : 更精簡表示 valueOf
- `getInstance`
- `newInstance`
- `getType`: 
- `newType`


>### 備註：

#### 服務提供者框架 (Service Provider Framework)

這是靜態工廠方法的一種實踐，這個系統由**服務提供者實現要提供的服務**，讓這系統來**提供服務給客戶端**

客戶端如果想要使用特定服務，可以透過這個框架，
選擇特定的(或是預設的)服務提供者，再由這個框架取得所需要的服務。


這個系統由三種組件組成

#### 服務介面 (Service Interface)

服務實體


#### 服務提供者介面 (Service Provider Interface)

服務提供者，由提供者來實現服務


#### 提供者註冊 (Provider Registration)

使系統能將實現的服務註冊，使客戶能夠使用


#### 服務存取 (Service Access)

讓客戶能夠取得服務實體




以JDBC為例

 類別       	                   | 角色
-------------------------------|---------------
 `Connectoin` 				   | Service Interface
 `DrvierManger.registerDriver` | Provider Registration
 `DriverManger.getConnection`  | Service Access
 `Driver` 					   | Service Provider Interface


```java
// 服務提供者概略設定

// 服務介面 (Service interface)
public interface Service {
	... // 定義特定於服務相關商業邏輯的方法
}

// 服務提供者介面 (Service provider interface)
public interface Provider {
	Service new Service(); 
}

// Noninstantiable class for service 
public class Services {
	private Services() {} // 避免實體化 (第四條)

	private static final Map<String, Provider> providers 
		= new ConcurrentHashMap<String, Provider>();

	public static final String DEFAULT_PROVIDER_NAME = "<def>";

	// 提供者註冊 API (Provider registration API)
	public static void registerDefaultProvider(Provider p) {
		registerProvider(DEFAULT_PROVIDER_NAME, p);
	}

	public static void registerProvider(String name, Provider p) {
		providers.put(name, p);
	}

	// 服務存取 API (Service access API)
	public static Service newInstance() {
		return newInstance(DEFAULT_PROVIDER_NAME); 
	}

	public static Service newInstance(String name) {
		Provider p = providers.get(name);
		if(p == null) {
			throw new IllegalArgumentException("No provider registered with name: " + name);
		}
		return p.newService();
	}
}
```