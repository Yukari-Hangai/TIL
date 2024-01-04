# 1月4日　　C++テキスト学習
#### ﾀｲﾋﾟﾝｸﾞ練習
- 1回目 4:38
- 5回目 4:25
&nbsp;  
&nbsp;  
## 参照を返す関数
**1. グローバル変数の参照を返す関数**
　関数の引数としてオブジェクトの参照を受け渡しできたように、関数からオブジェクトの参照を返すこともできる。  
``` グローバル変数を参照で返す.cpp
#include <iostream>

int x;

int& get_x() //参照を返す関数
{
    return x;
}

int main()
{
    x = 10;
    int& y = get_x(); //返された参照xをそのまま参照として受け取りyに代入
    y = 100;
    std::cout << x << std::endl;
}
```
↓実行結果
```
100
```
　普通に値を返すようにグローバル変数の参照を返している。戻り値として受け取る方も参照として受け取る。  
参照なので、変数の値を書き換えると元の変数の値も書き換わる。  
**ここで、注意**  
　上記の例ではグローバル変数の参照を返したので、変数の生存期間については特に気にする必要はなかったが、  
例えば関数の中で宣言されたローカル変数の参照を返した場合はダングリングリファレンス（無効な変数への参照を返す）となる。  
&nbsp;  
**2. メンバー変数の参照を返す関数**  
　パフォーマンスの問題などから、メンバー関数がメンバー変数を参照で返すということもよく行われる。  
特に、コピーできなかったり、コピーの処理が非常に大きなメンバー変数を返したりするとき。  
　返されたメンバー変数を受け取る側がただメンバー変数の中を見たり、直接変更したい場合には参照で受け取り、オブジェクトに影響を与えないようコピーを作って別の用途に使うなど、呼び出し側にコピーするのかしないのかの選択肢を与えることができる。  
 ```メンバー変数を参照で返す.cpp
#include <iosutream>
#include <string>

calss Object
{
    std::string name;

public:
    Object(std::stirng name);
    const std::string& get_name() const;
};

Object::Object(std::string name) : name{name}//文字列nameをメンバー変数のnameにコピー
{
}

const std:: string& Object::get_name() const
{
    return name; //nameを参照で返す
}

int main()
{
    Object obj{"とても大きなオブジェクト"};
    const std::string& name = obj.get_name(); //メンバー変数への参照を取得。コピーが起きないので高速
    std::cout << name << std::endl;
}
```
↓実行結果
```
とても大きなオブジェクト
```
　こうすることでオブジェクトのコピーを避けつつ、受け取る側は欲しい情報を得ることができる。  
constな参照としているので、クラスの外部からメンバー変数を変更することはできない。  
&nbsp;  
**3. 参照を返した場合の型推論**  
　関数が参照を返す問、戻り値を受け取る変数を型推論に任せるとどのような型になるのか。
 ```参照を返す関数の型推論.cpp
#include <iostream>

int& id(int& i)
{
    return i;
}

int main()
{
    int i = 42;
    auto j = id(i);//戻り値を受け取る変数の型をautoにする。jは参照？値？
    j = 0; //jに0を代入する。型が参照であればiが0になるはず。。。
    std::cout << i << std::endl;
}
```
↓実行結果
```
42
```
　42が出力されたということは、jの型は参照ではないということを示す。つまり、autoは参照ではなく値になるように型推論しているということになる。  
 　では、autoを使って参照として型推論したい場合はどうするのか。
```
auto& 参照名 = 式;
```
```参照の型推論.cpp
#include <iostream>

int& id(int& i)
{
    return i;
}

int main()
{
    int i = 42;
    auto& j = id(i);//auto&で参照の型推論
    j = 0;//jは参照なのでiが変わる
    std::cout << i << std::endl;
}
```
↓実行結果
```
0
```
&nbsp;  
&nbsp;  
**練習問題**
1.次のプログラムには問題があります。どのような問題か説明して下さい。
```
int& function()
{
    int value = 0;
    return value;
}

int main()
{
    int& value = function();
    value = 10;
}
```
↓解答
```
int& value = function();がダングリングリファレンスとなっている
```
&nbsp;  
2.getter/setterではなく、メンバー変数への参照を直接返すメンバー関数を定義してください。  
そしてそれをconstメンバー関数でもオーバーロードしてください。  
↓解答
```
class Human
{
	std::string name;

public:
	std::string& name()
	{
		return name;
	}

	const std::string& n name() const
	{
		return name;
	}
};
```
