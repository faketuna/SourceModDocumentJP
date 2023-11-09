# 4. コマンドの引数

さて、前回の内容ではコマンドの登録方法を記載しましたが、それだけではあまり高度なことはできません。

今回は、コマンドの引数の扱い方を書いていきたいと思います。

### TL;DR

この記事でわかるもの
* 引数の内容を取得する方法
* テキストのフォーマット方法


今回出来上がるコードは以下の内容になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/command-arguments.sp)からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "Command arguments",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    RegConsoleCmd("sm_argtest", CommandTest, "Arguments test command");
}

public Action CommandTest(int client, int args) {
    if(args == 0) {
        ReplyToCommand(client, "Not enough arguments!");
        return Plugin_Handled;
    }
    else {
        char buff[32];
        for(int i = 1; i <= args; i++) {
            GetCmdArg(i, buff, sizeof(buff));
            ReplyToCommand(client, "Argument %d: %s", i, buff);
        }
    }
    return Plugin_Handled;
}
```

## コマンドを作成する

前回解説した、[コマンド作成の方法](3-first-command.md)に従ってCallback関数を定義しコマンドのregisterを済ませた前提で話を進めます。

## 引数の数に応じて処理を変える

引数の数はCallback関数の第二引数(ここでは`args`)から取得できます。

### 引数がない場合の処理を書く

引数がない場合は`args`は0となるので、引数がない場合の処理を記述することができます。

まずは引数がない場合にメッセージを表示するようにしてみましょう。

以下のコードは引数がない場合に`Not enough arguments!`というメッセージを表示しその時点でコマンドの実行を終了します。

```C++
if(args == 0) {
    ReplyToCommand(client, "Not enough arguments!");
    return Plugin_Handled;
}
```

## 引数の内容を取得する

* 文字列として引数の内容を取得するには、`GetCmdArg`関数を使用します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/GetCmdArg)

* Intとして引数の内容を取得する場合は、`GetCmdArgInt`関数を使用します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/GetCmdArgInt)

* Floatとして引数の内容を取得する場合は、 `GetCmdArgFloat`関数を使用します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/GetCmdArgFloat)

今回は文字列として引数の内容を取得したいため、`GetCmdArg`関数を使用していきます。

#### 1. 文字列を格納するバッファを作成する

引数の文字列を利用するには、文字列を一時的に格納するバッファが必要になるので作成します。

```C++
char buff[32]
```

#### 2. 引数の内容をバッファに格納する

`GetCmdArg`関数を使用して引数の内容をバッファに格納します。

```C++
GetCmdArg(int argnum, char[] buffer, int maxlength)
```

第1引数は、引数の位置を入れる必要があります。 (0はコマンド自体, 1が1個目の引数, 2が二個目の...)

第2引数は、文字列を格納するバッファを指定します。

第3引数は、文字列の格納するバッファのサイズを指定します。


実際に使用する際は以下のようなコードになります。

```C++
char buff[32]
GetCmdArg(1, buff, sizeof(buff))
```

もし、`!argtest arg1text`と実行した場合、上記のコードであれば`buff`には`arg1text`が格納されることになります。

### 引数の内容を表示する

引数の内容を処理の分岐点として使うのはもちろんですが、ユーザーに対してメッセージを送信する際にも使用することがあると思います。(例: `The number <ここに引数の内容> is out of range!`)

今回は引数のIndexと内容をそのまま返信する機能を実装したいと思います。

#### 1. for文を作る

一般的なfor文の作り方と全く一緒です。 今回考慮する内容は

1. コマンドの引数は1から始まる
2. 引数分だけ処理する

```C++
for(int i = 1; i <= args; i++)
```

#### 2. メッセージを送信する {id=send_msg}

`ReplyToCommand`関数を使用してIndexと内容をフォーマットして返信します。

```C++
char buff[32];
for(int i = 1; i <= args; i++) {
    GetCmdArg(i, buff, sizeof(buff));
    ReplyToCommand(client, "Argument %d: %s", i, buff);
}
```

CやC++を触ってきた方はすでに理解できていると思いますが超簡易的な説明を挟みます。

```C++
ReplyToCommand(client, "Argument %d: %s", i, buff);
```

`%d`は`i`すなわちforのloop indexと対応しています。

`%s`は`buff`すなわちコマンドの引数の内容と対応しています。

細かい内容は[ここ](formatting.md)を確認してください。

#### 3. 作ったやつを結合する

作ったコードを結合するとこの様になると思います。

registerは済ませてあると思うので、ゲーム内で実際に実行してみましょう。

```C++
public Action CommandTest(int client, int args) {
    if(args == 0) {
        ReplyToCommand(client, "Not enough arguments!");
        return Plugin_Handled;
    }
    else {
        char buff[32];
        for(int i = 1; i <= args; i++) {
            GetCmdArg(i, buff, sizeof(buff));
            ReplyToCommand(client, "Argument %d: %s", i, buff);
        }
    }
    return Plugin_Handled;
}
```

#### 4. コマンドを実行してみる {id=exec_cmd}

`!argtest this is a test`を実行すると出力結果は以下のようになると思います。

```
Argument 1: this
Argument 2: is
Argument 3: a
Argument 4: test
```

問題がなければこれで引数の数に応じて処理の分岐を行う方法と、引数の内容を取得する方法を覚えることができました。