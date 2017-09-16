# chapter 2 オブジェクトの生成と消滅

## section 1 コンストラクタの代わりに`static` ファクトリーメソッドを検討する

* objectの生成コストが高い場合、不必要なインスタンス化をしないように実装できる
* method名が何をするのかを明示できる　

例）
	public static Boolean valueOf(boolean b) return b ? Boolean.TRUE : Boolean.FALSE;
