# chapter 2 オブジェクトの生成と消滅

## section 1 コンストラクタの代わりに`static` ファクトリーメソッドを検討する

例）

	public static Boolean valueOf(boolean b) return b ? Boolean.TRUE : Boolean.FALSE;

* objectの生成コストが高い場合、不必要なインスタンス化をしないように実装できる
* constractorと異なりmethod名が何をするのかを明示できる
* 不変クラスに2つの同じインスタンスが存在しないことを保証すれば `a.equals(b)`の代わりに`==`演算子を使用できる（パフォーマンスの向上）
* constractorと異なりメソッドの戻り値型の任意のサブタイプオブジェクトでも返すことができることです

## section 2 コンストラクタパラメタが多い場合はビルダーを使う

例）

	public class NutritionFacts {
		private final int servingSize;
		private final int servings;
		private final int calories;
		:
		public static class Builder {
			// 必須パラメタ
			private final int servingSize;
			private final int servings;
			// オプションパラメタ
			private int calories = 0;
			:
			public Builder(int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			}
			
			public Builder calories(int val) {
				calories = val;
				return this;
			}
			:
			public NutritionFacts build() {
				return new NutritionFacts(this);
			}
		}
		// private コンストラクタ
		private NutritionFacts(Builder builder) {
			servingSize = builder.servingSize;
			servings = builder.servings;
			calories = builder.calories;
			:
		}
	}
	
* 欠点は`Builder`オブジェクトの生成が必要なためコストがかかる

## section 3 `private`のコンストラクタか`enum`型でシングルトン特性を強調する

`public final` のフィールドによるシングルトンや`static`ファクトリーメソッドによるシングルトンでは
シリアライズの際に以下の手順を追加する必要がある

1. すべてのインスタンスフィールドを`transient`宣言する
2. `readResolve`メソッドを提供する（デシリアライズするごとに新たなインスタンスが生成されることを防ぐ）

例）

	private Object readResolve() throws ObjectStreamException {
		// 本物のオブジェクトを返して偽物をGCに始末させる
		return INSTANCE;
	}

対して`enum`によるシングルトンはシンプル

	public enum Elvis {
		INSTANCE;
		
		public void leaveTheBuilding() {...}
	}
	


