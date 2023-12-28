# 12月28日　　C++テキスト学習
#### ﾀｲﾋﾟﾝｸﾞ練習
- 1回目 4:16
- 5回目 4:03
&nbsp;  
&nbsp;  
## オブジェクトの配列
**1. オブジェクトの配列と初期化**
C++ではクラスも組み込み型（charとかintとか基本的な型）と同じように配列を作ることができる。  
組み込み型の配列では、初期値を渡さなければ初期化されず、配列の長さと同じ数だけ初期値を渡すことで、それぞれが初期化されていた。  
しかし、そのクラスがコンストラクターを定義していた場合、どうやってコンストラクターに引数を渡せばよいか。  
（関数形式の）明示的な型変換と呼ばれるコンストラクター呼び出しを行ってインスタンスを返す記法がある。  
これは都有の変数を作ることなく、関数呼び出しの実引数に使うインスタンスを作ったりする際に使う。  
```
型（引数）
型｛引数｝

void foo(A{"name", 42}); //明示的な変換を使ってインスタンスを作り関数の引数として渡す
```
配列内の初期化する各要素について明示的な型変換を並べていくと、配列の各要素を目的通りに初期化できる。  
```３つの三角形の初期化と面積の表示.cpp
//３つの三角形の初期化と面積の表示

#include <iostream>

class Triangle
{
	int m_height; //高さ
	int m_base_length; //底辺の長さ

public:
	explicit Triangle(int height, int base_length);//explicit:暗黙の型変換の禁止

	int height() const;//const関数heightの宣言
	int base_length() const;// const関数base_lengthの宣言
};

Triangle::Triangle(int height, int base_length)//コンストラクター
	: m_height(height), m_base_length(base_length)//コンストラクターのメンバー初期化リスト
{
}

int Triangle::height() const
{
	return m_height;
}

int Triangle::base_length() const
{
	return m_base_length;
}

int main()
{
	Triangle triangles[] = //配列のインスタンスを作成
	{
		Triangle{10, 20}, //各要素のコンストラクタにそれぞれの引数を渡す
		Triangle{20, 30},
		Triangle{40, 50},
	};

	for (auto& tri : triangles)
	{
		std::cout << "面積 : " << (tri.base_length() * tri.height() / 2) << std::endl;
	}
}
```   
↓実行結果
```
面積 : 100
面積 : 300
面積 : 1000
```
このようにクラスであっても必要な数だけ配列にして走査することができる。  
コンストラクターをオーバーロードしていても呼び出したいコンストラクターに合った引数を渡せばOK。
コンストラクター呼び出しの数よりも、配列の数の方が長い場合、先頭から順番に渡した引数で初期化され、足りない分はデフォルトコンストラクターが呼び出される。  
そのためデフォルトコンストラクターがない場合はエラーとなる。
```
#include <iostream>

class A
{
	std::string m_name;
	int m_value;

public:
	explicit A(std::string name, int value);
	explicit A(std::string name);
	A(); //デフォルトコンストラクター
	void show() const;
};

A::A(std::string name, int value) : m_name(name), m_value(value)
{
}

A::A(std::string name) : A(name, -1)
{
}

A::A() : A("default")
{
}

void A::show() const
{
	std::cout << m_name << " " << m_value << std::endl;
}

int main()
{
	A a[4] =
	{
		A{"first", 42}, //1つ目のコンストラクター呼び出し
		A{"second"}, //2つ目のコンストラクター呼び出し
		//3つ目以降はデフォルトコンストラクターが自動で呼び出しされる
	};

	a[0].show();
	a[1].show();
	a[2].show();
	a[3].show();
}
```
↓実行結果
```
first 42
second -1
default -1
default -1
```
配列とはいえオブジェクトなのでデストラクターの呼び出しが必ず行われる。  
そのため、デストラクターで多くの処理をするようなクラスを不必要に大きな配列として作ると、要素の数だけデストラクターの処理が走り処理速度に影響を与える。  
デストラクターで必要以上の処理をしないことと、配列も必要な長さにとどめておく。
また、配列は危険な操作ができてしまうことが多く、標準ライブラリが提供しているコンテナクラスを使った方が安全である。  
&nbsp;  
**2. 動的配列**  
配列というのは長さが決まっていて実行中にその長さを変えることができない。このような配列を固定長配列と呼ぶ。  
しかしプログラムを書いたとき配列の長さが決まっていないということの方が実際は多く、後から増えたり減ったりということは通常の配列では行うことができない。  
std::vectorは固定長配列と同じように扱えるが、長さは後から自由に変えることができる。このように長さが変わる配列を動的配列という。  
```
#include <vector> //std::vectorを使うときにヘッダーをincludeする

class A
{
   ....
}

std::vector<int> int_vector; //int型の動的配列
std::vector<A> A_vector; //クラスAの動的配列
```
<int>や<A>はテンプレート引数というもの。
std::vectorはテンプレートという機能を使っており、テンプレート引数に好きな型を核とその型用のstd::vectorをコンパイラーが自動に生成する。  
上記例ではint型用のstd::vectorの変数（int_vector）と、クラスA用のstd::vectorの変数（A_vector）を宣言している。  
動的配列は宣言しただけだと、中身は空の配列となる。通常の配列と同じく、{} を使って初期値を与えることもできる。  
また、size()メンバー関数を使うと動的配列の現在の長さがわかる。
```
std::size_t size() const;
```
↓動的配列
```動的配列.cpp
#include <vector>
#include <iostream>

int main()
{
	std::vector<int> empty;
	std::cout << "empty.size() : " << empty.size() << std::endl;

	std::vector<int> array = { 10, 20, 30, 40, 50 }; //{}を使って初期化
	std::cout << "array.size() : " << array.size() << std::endl;

	for (int e : array)//範囲for文で走査もできる
	{
		std::cout << e << std::endl;
	}
}
```
↓実行結果
```
empty.size() : 0
array.size() : 5
10
20
30
40
50
```
動的配列は通常の配列と同じように添字演算子を使って各要素にアクセスできる。  
この時の添字演算子で使うインデックスは０オリジンになる。  
&nbsp; 
通常の配列ではできない、後から追加や削除をする操作についてはいくつか種類がある。  
「最後に追加する」push_back()メンバー関数と、「最後を削除する」pop_back()関数メンバーについて
```
void push_back(const T& value);
void pop_back();
```
push_back()のTはテンプレート引数で渡した型と同じものになる。  
pop_back()は何も戻り値を返さないので、削除した要素が必要であればあらかじめコピーしておく必要がある。
```
#include <vector>
#include <iostream>

int main()
{
	std::vector<int> list;

	list.push_back(42);
	list.push_back(0);

	for (int e : list)
	{
		std::cout << e << std::endl;
	}

	std::cout << std::endl;

	list.pop_back();
	list.push_back(-10);

	for (int e : list)
	{
		std::cout << e << std::endl;
	}
}
```
↓実行結果
```
42
0

42
-10
```
&nbsp;  
&nbsp;  
**練習問題**
1. 次のプログラムがコンパイルできるよう修正して下さい。  

↓問題
```問題.cpp
#include <string>

class product
{
	int id;
	std::string name;
	int price;

public:
	explicit product(int id, std::string name, int orice)
		: id(id), name(name), price(price) {}
};

int main()
{
	product p[4] =
	{
		product{1, "smart phone", 60000},
		product{2, "tablet", 35000},
	};
}
```
↓解答（自分なりの解説付き）
```
#include <string>

class product
{
	int id = 0; //NSDMIでデフォルト値を指定
	std::string name = " ";
	int price = 0;

public:
	product() {} //デフォルトコンストラクタの宣言・定義、NSDMIで指定された値で初期化
	explicit product(int id, std::string name, int orice)
		: id(id), name(name), price(price) {}
};

int main()
{
	product p[4] =
	{
		product{1, "smart phone", 60000},
		product{2, "tablet", 35000},
	};
}
```
&nbsp;  
2. 1.のプログラムをstd::vectorを使って書き直してください。その際、要素を追加して４要素になるようにしてください。  
↓解答
```
#include <string>
#include <vector>

class product
{
	int id = 0;
	std::string name = " ";
	int price = 0;

public:
	product() {}
	explicit product(int id, std::string name, int orice)
		: id(id), name(name), price(price) {}
};

int main()
{
	std::vector<product> p =
	{
		product{1, "smart phone", 60000},
		product{2, "tablet", 35000},
	};

	p.push_back(product{}); //push_backで要素（デフォルトコンストラクタ）を追加
	p.push_back(product{});
}
```
