# 12月27日　　C++テキスト学習
#### ﾀｲﾋﾟﾝｸﾞ練習
- 1回目 4:47
- 5回目 4:37
&nbsp;  
&nbsp;  
## 配列とポインター
**1. ポインターへの暗黙変換**  
配列は、ポインター型への暗黙変換ができる。  
配列にはたくさんの要素があるが、暗黙変換されるときには先頭の要素へのポインターとして変換される。  
```配列の暗黙変換.cpp
#include <iostream>

int main()
{
  int array[] = { 0, 1, 2, 3 };

  std::cout << "先頭のアドレス : " << &array[0] << std::endl;

  int* ptr = array;

  std::cout << "ポインター : " << ptr << std::endl;
  std::cout << "値 : " << *ptr << std::endl;
}
```
↓実行結果
``` 
先頭のアドレス : 00000009D40FF918
ポインター : 00000009D40FF918
値 : 0
```
ポインターが持つアドレスも、間接参照した値も配列の先頭を指していることがわかる。  
文字列リテラルも文字の配列なので、同様にポインターへの暗黙変換が可能。  
ただし、文字列リテラルは変更不可の配列なので、const修飾されたポインターとなる。  
```
const char* string = "string literal"; //文字列リテラルはconstな配列
```
※const : 変数などを変更不可とするための修飾子  
&nbsp;  
**2. 次のアドレス**  
配列をポインターに変換したときは先頭の要素のアドレスとなるが、先頭以外の要素にアクセスするときは配列の時と同じく添字演算子を使う。  
```先頭以外の要素.cpp
#include <iostream>

int main()
{
	int array[] = { 0, 1, 2, 3 };

	int* ptr = array;

	std::cout << ptr[0] << std::endl;
	std::cout << ptr[1] << std::endl;
	std::cout << ptr[2] << std::endl;
	std::cout << ptr[3] << std::endl;
}
```
↓実行結果
```
0
1
2
3
```
添字演算子は直接任意の要素にアクセスするが、ポインターのアドレスを使ってもアクセスできる。  
変数のアドレスは１を足すと次の要素のアドレスを、１を引くと前の要素のアドレスを指す。  
１以上を足しひきするとさらに次や前の要素のアドレスとなり、好きな要素のアドレスを入手できる。  
``` 先頭以外のアドレス.cpp
#include <iostream>

int main()
{
	int array[] = { 0, 1, 2, 3 };

	int* ptr = array;

	ptr += 2; //2番目の要素のアドレス
	std::cout << *ptr << std::endl;

	++ptr; //3番目の要素のアドレス
	std::cout << *ptr << std::endl;

	ptr -= 2; //1番目の要素のアドレス
	std::cout << *ptr << std::endl;

	--ptr; //0番目の要素のアドレス
	std::cout << *ptr << std::endl;
}
```
↓実行結果
```
2
3
1
0
```
&nbsp;  
**3. 配列と引数**  
配列のコピーを作れないのは引数であっても同じ。  
しかし関数の引数として配列を記述することは可能。  
ただし、引数に記述された配列は配列ではなくポインターとして宣言されたものとして扱われる。  
```
void function(int array[5]);
void function(int* array);
//上記の２つのプロトタイプ宣言は同じ意味になる
```
ポインターと同じ意味になるので、配列の長さが違っていても関数呼び出しはできる。
```
void fuction(int array[5]);

int main()
{
	int array[4] = {};
	function(array); //ポインターへの暗黙変換が行われる
}
```
&nbsp;  
**4. 配列の型と別名**  
配列はポインターに暗黙的に変換できるが、配列自体はポインター型ではなく「配列の型」というものを持っている。  
ただし、配列へのポインター型や配列への参照型は、普通のポインター型や参照型の宣言とは表記方法が大きく異なる。  
```配列の型.cpp  
型 [配列の要素数] //配列の型
型 (*)[配列の要素数] //配列へのポインター型
型 (*ポインター名)[配列の要素数] = & 配列名; //配列へのポインターの宣言
型 (&)[配列の要素数] //配列への参照型
型 (& 参照名)[配列の要素数] = 配列名; //配列への参照の宣言
```
()を忘れたり、*や&の位置が異なると別の意味になる。↓
```a.cpp
int* pointer[10] //長さ10のint&型の配列。(int*型の変数が10個並んでいる)
int& (reference)[10]; //長さ10のint&型の配列。ただし参照型の配列は作れないのでエラーとなる。
```
また、長さが異なる配列はそれぞれ別の型を持っており、仮引数のように勝手に無視されたりはしない。↓  
```
int array[] = { 0, 1, 2, 3, 4};

int (*pointer)[10] = &array; //arrayはint[5]型なのでint[10]型へのポインターには代入できないのでエラー。
```
&nbsp;
配列へのポインターや参照は型に配列の長さの情報を持っているので、範囲for文で走査できる。↓
```配列のポインターと参照.cpp
#include <iostream>

int main()
{
	int array[5] = { 0, 1, 2, 3, 4};

	int (*ptr)[5] = &array;  //配列へのポインター

	for (int e : *ptr) //ポインターなので間接参照演算子*が必要
	{
		std::cout << e << std::endl;
	}

	std::cout << std::endl; //空行

	int(&ref)[5] = array; //配列への参照

	for (int e : ref) //参照なので間接参照演算子*は不要
	{
		std::cout << e << std::endl;
	}
}
```
↓実行結果
```
0
1
2
3
4

0
1
2
3
4
```
配列へのポインターや参照は()があり、使いづらいので必要となった場合には型に別名を与えるとよい。↓
```別名.cpp
using int_array = int[5]; //int[5]にint_arrayという別名を与える
int_array array; //arrayは長さ5のint型の配列
int_array* aptr = &array; //aptrは長さ5のint型の配列へのポインター
int_array& aref = array; //arefは長さ5のint型の配列への参照
using int_array_pointer = int (*)[5]; //int(*)[5]にint_array_pointerという別名を与える
int_array_pointer ptr = &array; //ptrは長さ5のint型の配列へのポインター
using int_array_reference = int (&)[5]; //int(&)[5]にint_array_referenceという別名を与える
int_array_reference ref = array; //refは長さ5のint型の配列への参照
```
特に配列へのポインターや参照を返す関数の戻り値の型は、複雑で読みづらくなるので別名を使うとよい。↓
```
//長さ10のint型配列へのポインターを返す関数
int (*function(int a))[10]
{
  .....
}
```
