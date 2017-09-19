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

上記条件を満たしていても特権をもつクライアントは`AccessibleObjct#setAccessible()`メソッドを使用してリフレクションにより`private`コンストラクタを呼び出せる
対して`enum`によるシングルトンはシンプル

	public enum Elvis {
		INSTANCE;

		public void leaveTheBuilding() {...}
	}

## section 4 `private`のコンストラクタでインスタンス化不可能を強調する

例）

	public class UtilityClass {
		private UtilityClass() { throw new AssertionError(); }
		:
	}

## section 5 不必要なオブジェクトの生成を避ける

悪い例）

	String s = new String("aaa");
	Boolean a = new Boolean("true");

良い例） static factory method を使用することで大抵は不必要なオブジェクト生成を避けられる

	String s = "aaa";
	Boolean a = Boolean.valueOf("true");

悪い例２）

	public class Person {
		private final Date birthDate;

		public boolean isBabyBoomer() {
			Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
			Date boomStart = gmtCal.getTime();
			gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
			Date boomEnd = gmtCal.getTime();
			return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
		}
	}

良い例２）

	public class Person {
		private final Date birthDate;

		private static final Date BOOM_START;
		private static final Date BOOM_END;

		static {
			Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_START = gmtCal.getTime();
			gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_END = gmtCal.getTime();
		}

		public boolean isBabyBoomer() {
			return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
		}
	}

## 廃れたオブジェクト参照を取り除く

*メモリリーク!!!*

	public class Stack {
		private Object[] elements;
		:
		public Object pop() {
			if (size == 0) throw new EmptyStackException();
			return elements[--size];
		}
		:
		private void ensureCapacity() {
			if (elements.length == size)
				elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}

上記の`return elements[--size];`は`elements[size]`オブジェクトが廃れた参照のため、メモリリークする。以下コードの通り置き換える：

	Object result = elements[--size];
	elements[size] = null;
	return result;

*クラスが独自のメモリを管理（`Object[] elements`）している時にはプログラマはメモリリークに対して注意を払うべき*

その他のメモリリーク例：

* キャッシュ。オブジェクト参照をキャッシュに入れて忘れてしまう。そのオブジェクト参照がなくなった後でもキャッシュに残したまま。 解決策として`WeakHashMap`を使用するとGCされる
* リスナーやコールバック。コールバックが蓄積されてしまう。同じく`WeakHashMap`で解決可能

## ファイナライザを避ける
