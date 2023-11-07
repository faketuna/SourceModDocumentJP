# Pragma

`#pragma`はコンパイラに特定の情報を渡すためのコンパイラ司令と呼ばれる物です。
pragmaを使用することによってコンパイル内容を詳細に制御できます。

### SourceModのPragmaで指定できる内容のリスト

#### `#pragma ctrlchar`
通常のエスケープ文字は`\\`ですが、`#pragma ctrlchar`を使用してエスケープ文字を置き換えることができます。

例:
```C++
#pragma ctrlchar `^`

PrintToServer("Line one ^Line 2");
```

#### `#pragma deprecated`
プログラム内の特定の関数を非推奨に変更します。
結果として、コンパイル時に関数が非推奨である旨の警告が表示されるようになります。

例:
```C++
#pragma deprecated This function is deprecated
void Test()
{
    PrintToServer("Test!");
}
```

#### `#pragma dynamic`
スタックとヒープのメモリブロックのサイズをセル単位で指定することができます。

通常は設定する必要は一切ありません。 プラグイン実行中にStack Errorが発生した場合等に設定してください。

例:
```C++
#pragma dynamic 131072
```

#### `#pragma newdecls`
SourcePawnの新しい記法を強制するかどうかを指定できます。
新しい記法はより明示的にコードを書くことができるため、個人的には入れるのを推奨します。

変数の例:
```C++
#pragma newdecls required

// Error
new g_iTest;
new Handle:g_hTest;

// OK
int g_iTest;
Handle g_hTest;
```

関数の例:
```C++
#pragma newdecls required

// Error
testFunction(){}

// OK
void testFunction(){}

```

関数の例2:
```C++
#pragma newdecls required

// Error
testFunction(client, args){}

// OK
void testFunction(int client, int args){}
```


#### `#pragma semicolon`
行末に`;`を強制するかどうかを指定できます。

例:
```C++
#pragma semicolon 1

// Error
PrintToServer("Test!")

// OK
PrintToServer("Test!");
```

#### `#pragma tabsize`
調べたのですが情報見つからず...

#### `#pragma unused`
プログラム内で使用されていない関数/変数のコンパイル時の警告を消すことができます。

ただ、これを使用するのであれば大人しく使用してない関数/変数を削除するほうが良いと思います。

例:
```C++
#pragma unused g_iUnUsedVariable
int g_iUnUsedVariable;
```
