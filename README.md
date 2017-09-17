# chapter 2 オブジェクトの生成と消滅

## section 1 コンストラクタの代わりに`static` ファクトリーメソッドを検討する

例）
	public static Boolean valueOf(boolean b) return b ? Boolean.TRUE : Boolean.FALSE;

* objectの生成コストが高い場合、不必要なインスタンス化をしないように実装できる
* constractorと異なりmethod名が何をするのかを明示できる
* 不変クラスに2つの同じインスタンスが存在しないことを保証すれば `a.equals(b)`の代わりに`==`演算子を使用できる（パフォーマンスの向上）
* constractorと異なりメソッドの戻り値型の任意のサブタイプオブジェクトでも返すことができることです



