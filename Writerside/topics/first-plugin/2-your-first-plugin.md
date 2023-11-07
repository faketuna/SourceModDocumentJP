# 2. 最初のプラグイン!

最初のプラグインはベタですが起動時に"Hello World!"をサーバーコンソールに出力する物を作っていきます。

### TL;DR

今回出来上がるコードは以下の内容になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/hello-world.sp)からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "First Plugin",
    author = "Author name",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    PrintToServer("Hello World!");
}
```

## 詳細解説

### #include \<sourcemod\>
JavaやPython等を使用してきた方は一瞬戸惑うかもしれません。 厳密には違いますが、Java等のimportと同じ様な存在と捉えて構いません。

ですので、今回はプラグインを作るのに必要な機能が入っている`sourcemod.inc`を使用したいため一番上に`#include <sourcemod>`を記述します。

### #pragma
`#pragma`はコンパイラに特定の情報を渡すためのコンパイラ司令と呼ばれる物です。
pragmaを使用することによってコンパイル内容を詳細に制御できます。

今回使用している内容は以下の二種類です。

#### semicolon 1
`#pragma semicolon 1`は行末に`;`を入れることを強制します。

#### newdecls required
`#pragma newdecls required`はSourcePawnの新しい記法を強制します。 新しい記法は明示的でコードが読みやすいため、特段理由がなければ強制するのを推奨します。

他のpragmaの種類を知りたい場合は[こちら](pragma.md)を参照してください

### #define
`#define`は定数を宣言するためのものであり、Java等におけるfinal staticを付けた定数と思っていただいて良いと思います。

プラグインのコードが増えてもバージョンを簡単に変更できるようにするために私はこの方法を使いますが、`version = "0.0.1"`としても問題はありません。

### public Plugin myinfo
以下の情報をプラグインに関連付けることができます。

* name - プラグインの名前
* author -  制作者の名前
* description - 詳細
* version - バージョン
* url - URL

特に入れる情報がない場合は`""`を指定してください。

### public void OnPluginStart(){}
プラグインの開始時に呼び出される関数です。
基本的にはプラグインに必要な初期セットアップをここで行います。

### PrintToServer()
名前のままですが、サーバーのコンソールにメッセージを表示します。

## プラグインのコンパイル
scriptingフォルダに作成したプラグインのファイルを設置し以下の方法でコンパイルを行えます。

* .spファイルを`compile.exe`もしくは`compile`にドラッグアンドドロップ
* Windowsのコマンドプロンプトであれば`compile <filename>.sp`
* linuxであれば`./compile <filename>.sp`

エラーが特に無ければ`scripting/compiled/`ディレクトリにコンパイルされた`.smx`ファイルが生成されるため他のプラグインと同様にロードしてあげればokです。


## 最後に
SourcePawnはC++に近いためJavaやPython等を書いていた方は少し戸惑ったり不便を感じたりするかもしれませんがやってれば慣れるので頑張ってください。

応用編の解説は少しづつ書いていきます。