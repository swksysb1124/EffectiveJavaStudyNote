# 第二條： 當面對多個構建參數情況時，考慮使用構建器(Builder)


傳統做法上，當碰到有多個建構參數的物件生成，會採取一種叫**伸縮式建構函式 (Telescoping Constructor)模式**。

```java
public class NutritonFact {
	private final int servingSize; 	// (mL) 			required
	private final int servings;		// (per container)	required

	private final int calories;		// 					optional
	private final int fat;			// (g)				optional
	private final int sodium;		// (mg)				optional
	private final int carbohydrate;	// (g)				optional

	public NutirtionFact(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}

	public NutirtionFact(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}

	public NutirtionFact(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}

	public NutirtionFact(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, sodium, 0);
	}

	public NutirtionFact(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}
```
當你要調用時
```java
NutritionFact cocaCola = 
	= new NutritionFact(240, 8, 100, 0, 35, 27);
```

這種創建物件的方式，即時很多參數你不想設定，你還是必須要設定。當創建參數越多，這種情況越嚴重。

所以基本上**伸縮建構函式可以使用，但當參數過多時，會變得很難寫也會造成客戶端程式閱讀困難**。當客戶端程式編寫時，需要很仔細的計算建構參數的個數避免填錯，這樣很容易出錯。另外，由於編譯階段可能無法報錯，造成問題到在執行階段才發現。

另一種方法，是使用**JavaBean模式**，就是建構子本身不帶參數，所以資料欄位都有設定預設值，之後再由`setter`帶入參數改變資料欄位

```java
public class NutritionFact {

	// Parameter initialized to default value (if any)
	private int servingSize = -1; 	// required: no default value
	private int servings = -1; 		// required: no default value
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFact() { }

	// Setters
	public void setServingSize(int val) { servingSize = val; }
	public void setServings(int val) { servings = val; }
	public void setCalories(int val) { calories = val; }
	public void setFat(int val) { fat = val; } 
	public void setSodium(int val) { sodium = val; } 
	public void setCarbohydrate(int val) { carbohydrate = val; } 
}
```

但這個 JavaBean 模式有兩個缺點，主要是它使建構物件的過程切分成很多階段，這有可能造成建構過程物件狀態不一致的情形。

另外，JavaBean模式，由於每個建構參數都是生成物件後，才去修改，這排除了物件不可變的可能性。

這時候我們就建議使用 **構建器(Builder)** 模式。

我們不直接創建物件，而是先取得一個構建器物件，輸入創建參數給這個構建器，最後透過這個構建器生成我們需要的物件。

```java
// Builder Pattern
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder {
		// Required parameters
		private final int servingSize;
		private final int servings;

		// Optional parameters - initialized to default values
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val) {
			calories = val;
			return this;
		}

		public Builder fat(int val) {
			this.fat = val;
			return this;
		}

		public Builder sodium(int val) {
			this.sodium = val;
			return this;
		}

		public Builder carbohydrate(int val) {
			this.carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}

	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}
}
```

需要生成物件時
```java
NutritionFacts cocoCola 
	= new NutritionFacts.Builder(240, 3)
		.calories(100)
		.sodium(35)
		.carbohydrate(27)
		.build();
```

可以看到，相較於伸縮式，需要**輸入的參數有名稱的說明，更一目瞭然**。
另外，他可以**使創建狀態一致**，也能**保留物件不可變的可能性**。

Builder 可以是一個虛擬工廠模式的實現

```java
// A builder for objects of type T
public interface Builder<T> {
	public T build();
}
```

```java
NutritionFacts cocoCola 
	= new Builder<NutritionFacts>(240, 3)
		.calories(100)
		.sodium(35)
		.carbohydrate(27)
		.build();
```

我們可以使用 **限制通配符類型(bounded wildcard type)** 限定由Builder生成物件。

例如
```java
Tree buildTree(Builder<? extends Node> nodeBuilder) { ... }
```

當然構建器模式也有缺點，首先定義類別時，必須也同時定義他的構建器。雖然在實際上創建構建器的成本不太可能明顯，但在某些性能關鍵的情況下它可能是一個問題。

構建器也叫伸縮建構子要來的冗長，所以一般是**建議當建構參數有四個以上**時，才來考慮構建器模式。但是要留心，在未來也許會想加入新的參數。

總結，當我們有多的建構參數(尤其多數是選擇性的)的時候，構建器模式是個不錯的選擇。使用構造器模式，客戶端程式也更容易閱讀跟編寫。
