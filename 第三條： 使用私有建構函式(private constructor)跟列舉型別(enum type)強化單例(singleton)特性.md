# 第三條： 使用私有建構函式(private constructor)跟列舉型別(enum type)強化單例(singleton)屬性


單例模式就是確保一個類別只能有一個物件實體。在系統中，像是Window manager跟File System都是單例實體。
對於客戶端來說，單利模式很難被測試。為什麼呢？因為他必須要先實現一個這個類別的介面，不然無法利用仿真物件特換掉單例物件。


有幾種方法來實現單例模式

### 公開靜態實體方法

```java
/**
 * 	Singleton with public static final field
 */
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding() { ... }
}
```

### 靜態工廠方法

```java
/**
 *	Singleton with static factory
 */
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding() { ... }
}
```


為了避免反序列化後，單例物件生成一個一上的實體，定義`readResolve`方法來保留單例特性：

```java
private Object readResolve() {
	return INSTANCE;
}
```


實際上還有一種更好的方法，就是**採用列舉型別來實現單例模式**

```java
public enum Elvis {
	INSTANCE;

	public void leaveTheBuilding() { ... }
}
```

其實由列舉型別實現單例模式，功能上等同於公開靜態實體實現。但他更簡潔，也免費提供了序列化下保證單例特性的機制。
即使在複雜的序列化及反射攻擊下，不會生成多的實體的情況。

雖然列舉法目前並沒有廣泛使用，但**單一元素的列舉型別是實現單例模式最好的方法**。


>## 補充

### 列舉型別 (enum type)

列舉型別定義一個變數型別跟他對應一組常數數值，這個型別的變數之值必須是這些常數數值的其中一個。
當我們有需要定義一組固定的常數數值，比如說**特定的日期**、**系統的狀態**等，列舉就很適合。


定義一個Day的列舉型別

```java
public enum Day {
	SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
	THURDAY, FRIDAY, SATURDAY
}
```

取得列舉的值

```java
for(Day day: Day.values()) {
	System.out.println("名稱: "+day.name());
	// System.out.println("NAME: "+day.toString());
	System.out.println("順序: "+day.ordinal());
}
```

列舉搭配`switch`

```java
/**
 * 賦予一個Day列舉一個數值，
 *
 * 比如說
 * 		Day day = Day.SUNDAY;
 */
switch(day) {

	case MONDAY:
		System.out.println("憂鬱的星期一");
		break

	case TUESDAY:
		System.out.println("忙碌的星期二");
		break;
	
	default:
		System.out.println("其他天普普");
		break;
}
```


進階設定

列舉不只能設定變數型別對應單一種常數數值組合，它可以對應一種常數資料結構。
我們稱它為**列舉類別 (enum class)**。

### 列舉類別 (enum class)

```java
public enum Plant {

	public static final double G = 6.67300E-11;

	MERCURY	(3.303e+23, 2.4397e6),
	VENUS	(4.869e+24, 6.0518e6),
	EARTH	(5.976e+24, 6.37814e6),
	MARS	(6.421e+23, 3.3972e6),
	JUPITER	(1.9e+27, 	7.1492e7),
	SATURN	(5.288e+26, 6.0268e7),
	URANUS	(8.686e+25,	2.5559e7),
	NEPTUNE	(1.024e+26,	2.4746e7);

	// 定義兩種常數資料類別
	private final double mass; 		// 單位為公斤
	private final double radius; 	// 單位為公尺

	Plant(double mass, double radius) {
		this.mass = mass;
		this radius = radius;
	}

	/**
	 *	--------  自訂列舉類別方法  --------
	 */

	private double mass() {	return mass; }

	private double radius() { return radius; }

	double surfaceGravity() { return G * mass / (radius * radius); }

	double surfaceWeight(double otherMass) { return otherMass * surfaceGravity(); }

}
```

實際上調用

```java
double mass = earthWeight / Plant.EARTH.surfaceGravity();

for(Plant p: Plant.values()) {
	System.out.println("你在%s的體重為%f\n",p.name(), p.surfaceWeight(mass))
}
```

