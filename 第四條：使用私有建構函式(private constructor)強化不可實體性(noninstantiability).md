# 第四條： 使用私有建構函式(private constructor)強化不可實體性(noninstantiability)

有的時候我們會想要創建一個類別來包含我們常用的靜態常數或是靜態方法，
像這樣工具類型的類別，並不需要實體化，我們可以定義私有的建構函式來避免他們被實體化。

```java
// Noninstantiable utility class 
public class UtilityClass {

	// Suppress default constructor for noninstantiability
	private UtilityClass() {
		throw new AssertionError();
	}

	// ...
}
```

確定地將建構函式私有化會有一個問題，就是他會導致這個類別不能被繼承。
所有繼承的類別的建構函式都會呼叫他的父類別的建構函示，當父類別的建構函式是私有的，子類別無法調用就無法完成建構實體。