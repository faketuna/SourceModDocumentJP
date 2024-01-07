# 7. 多言語対応


## 多言語対応



## TL;DR

この記事でわかるもの
* 翻訳ファイルの作り方
* 翻訳ファイルを使用する
* 翻訳内容に値を代入する

今回解説した内容の実装例は以下になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/translations.sp)からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "Translation",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    LoadTranslations("test_trans.phrases");
    RegConsoleCmd("sm_trans", CommandTranslation, "Translation test command");
    RegConsoleCmd("sm_thw", CommandTranslatedHelloWorld, "Send translated hello world");
}

public Action CommandTranslation(int client, int args) {
    if(client == 0) {
        return Plugin_Handled;
    }

    SetGlobalTransTarget(client);
    if(args == 0) {
        ReplyToCommand(client, "%t", "not enough arguments");
        return Plugin_Handled;
    }
    else if(args > 1) {
        ReplyToCommand(client, "%t", "too many arguments", args);
        return Plugin_Handled;
    }

    char cmdArg1[32], cmdName[32], playerName[64];
    GetCmdArg(0, cmdName, sizeof(cmdName));
    GetCmdArg(1, cmdArg1, sizeof(cmdArg1));
    GetClientName(client, playerName, sizeof(playerName));
    PrintToChatAll("%t", "notify", playerName, cmdName, cmdArg1);
    return Plugin_Handled;
}

public Action CommandTranslatedHelloWorld(int client, int args) {
    SetGlobalTransTarget(client);
    ReplyToCommand(client, "%t", "hello world");
    return Plugin_Handled;
}
```

作成した翻訳ファイルは以下になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/translations/test_trans.phrases.txt)からも確認できます)
```
"Phrases"
{
    "hello world"
    {
        "en"    "Hello World!"
        "jp"    "はろーわーるど！"
    }
    "not enough arguments"
    {
        "en"    "Not enough arguments!"
        "jp"    "引数が足りません！"
    }
    "too many arguments"
    {
        "#format"   "{1:d}"
        "en"    "Too many arguments! args: {1}"
        "jp"    "引数が多すぎます！ 引数の数: {1}"
    }
    "notify"
    {
        "#format"   "{1:s},{2:s},{3:s}"
        "en"        "{1} has specified {3} as command {2}'s first argument"
        "jp"        "{1}はコマンド{2}の最初の引数として{3}を指定しました"
    }
}
```

## 翻訳ファイルって? {id=what_is_trans_file}

翻訳ファイルとは、プラグインの中で多言語対応を行う際に必要になるファイルです。

翻訳ファイルの内容は例として以下のような形になります。

```
"Phrases"
{
    "translation key"
    {
        "en"    "Translated text"
        "jp"    "翻訳された文字列"
    }
}
```

<procedure title="translation key" id="val_trans_file_trans_key">
    <p>翻訳を実際に使用する際、特定の翻訳を指定するために必要なキーです。</p>
    <p>複数の翻訳ファイルを同時に読み込まなければ命名は特に気にする必要はありません。自分がわかりやすい名前を指定しましょう。</p>
    <p>Tips</p>
</procedure>

<procedure title="en/jp" id="val_trans_file_lang_key">
    <p>言語のキーとそれに対応する翻訳結果を記載します。</p>
    <p>言語は国コード<a href="https://w.wiki/3EdJ">ISO 639-1</a>に準拠した名前に...一部がなってません。</p>
    <p>そうです、なんか知らんけど日本語がjaでもjpnでもなくjpとして使われてるんですねぇ...</p>
    <p>SourceModにデフォルトで入ってる言語のリストは<a href="https://github.com/alliedmodders/sourcemod/blob/master/configs/languages.cfg">ここ</a>を確認してください</p>
</procedure>

> Tips
>
> 翻訳キーは大文字小文字が区別されるため、基本的には小文字か大文字どちらかで統一することをおすすめします。
> 
> 翻訳キーにスペースが入っていても問題なく使用できます。 心配であれば、`trans_key`のようにアンダースコアで繋げても問題ありません。
> 
> 
{style="note"}

## 翻訳ファイルを作る {id=make_trans}

翻訳ファイルは、2通りの作り方が存在します。

1. 一つのファイルに全てをまとめて記載する方法
2. 言語ごとのファイル/ディレクトリに分割して記載する方法

翻訳ファイルの命名は先人に習って`<Plugin名>.phrases.txt`とすると良いでしょう

### まず雛形を作る {id=make_trans_base}

雛形と行っても記載する内容はいたってシンプルで以下の内容入れるだけです。

```
"Phrases"
{

}
```

### 1の方法で行う場合 {id=make_trans_choice_1}

一つのファイルに全てをまとめて記載する方法は最初に作成した雛形に以下のような形で記載していく方法になります。

```
"Phrases"
{
    "translation key"
    {
        "en"    "Translated text"
        "jp"    "翻訳された文字列"
    }
}
```

### 2の方法で行う場合 {id=make_trans_choice_2}

複数のファイルに分割して記載する方法は1の方法で作成した物を言語ごとに分割する形で作ることが出来ます。

この方法を行う際、ファイル名は同一である必要があります。

#### en/<Plugin名>.phrases.txt

```
"Phrases"
{
    "translation key"
    {
        "en"    "Translated text"
    }
}
```

#### jp/<Plugin名>.phrases.txt

```
"Phrases"
{
    "translation key"
    {
        "jp"    "翻訳された文字列"
    }
}
```

## 結局どっちがいいの? {id=make_trans_which_is_best}

#### 結論: どっちでも良い

正直どっちでも良いと思ってます。個人で管理しやすい方を選んでもらって構いません。

対応言語が増えてくるとフォルダごとに分けたほうが良いとは思いますが、そこまでグローバルに翻訳されるようなプラグインはそうそうないと思うので...

## 翻訳ファイルをロードする {id=load_translation_file}

翻訳ファイルをロードするには、`LoadTranslations()`関数を使用します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/lang/LoadTranslations)

```C++
void LoadTranslations(const char[] file)
```

<procedure title="file" id="val_loadtrans_file">
    <p>読み込むファイル名を指定します。</p>
    <p>拡張子を指定しなかった場合は自動的に入れた内容+.txtのように解釈されます。</p>
    <p>例: <code>test_trans.phrases.txt</code> は <code>test_trans.phrases.txt</code></p>
    <p>例: <code>test_trans.phrases</code> も <code>test_trans.phrases.txt</code></p>
</procedure>

ロードするタイミングは恐らくどのタイミングでも行えるとは思いますが、基本的には`OnPluginStart`関数内で行います。

```C++
public void OnPluginStart()
{
    LoadTranslations("test_trans.phrases");
}
```

## 翻訳された文字列を入手する

翻訳された文字列を入手するには、`Format`関数を使用するか他の`ReplyToCommand`のようなFormatをサポートしてる関数を使用します。

`Format`関数を使用する場合は以下のように`%t`指定子を使用して翻訳された文字列を代入することが出来ます。

```C++
char buff[128];
Format(buff, sizeof(buff), "%t", "translation key");
```

それ以外の`ReplyToCommand`のようなFormatをサポートしている関数も同様に`%t`指定子を使用して翻訳された文字列を代入できます。

```C++
ReplyToCommand(client, "%t", "translation key");
```

## 翻訳内容に値を代入する

ちゃんと、翻訳内容に値を代入することも出来ます。

値を代入できるようにするには少し下準備が必要になります。

### 翻訳ファイルを編集する

値を代入させたい翻訳内容に対して、`"#format" ""`を追加します。

```
"Phrases"
{
    "translation key"
    {
        "#format"   ""
        "en"        "Translated text"
        "jp"        "翻訳された文字列"
    }
}
```

次に、`"#format" ""`にどの様な値が入るかを記載します。

```
"#format"   "{値の番号:型}"
```

型にはFormatで使用できる指定子の中から以下を使用できます。

* d/i - 数値/整数
* x - 16進数
* f - float
* s - String
* c - Character (UTF-8対応)
* t - 他の翻訳を埋め込む

値の番号は複数ある際、また代入する場所を指定するために必要です。

上記を踏まえて、文字列を代入させたい場合は以下のように指定します。

```
"#format"   "{1:s}"
```

指定し終えたら、以下のようにして代入させたい位置に`{数値}`を入れてあげると値がそこに代入されるようになります。

```
"Phrases"
{
    "translation key"
    {
        "#format"   "{1:s}"
        "en"        "{1} Translated text"
        "jp"        "{1} 翻訳された文字列"
    }
}
```

### コードを編集する

コード内で使用する際は、代入したい値をFormatと同じ様な形で引数に追加していきます。

よって以下のような形になります。

```C++
ReplyToCommand(client, "%t", "translation key", "[TEST]");
// もしも日本語にしていたら以下の内容が出る。
// [TEST] 翻訳された文字列
```

## 翻訳関連のエラー {id=error_trans}

以下の状況

* 翻訳ファイルが存在しない
* コード内で使用されている翻訳キーが翻訳ファイルに記載無し/スペルミス

の様な場合は、ロード時点でエラーは吐かず実際に使用する際に以下のようなエラーを発生させます。

```
L 01/06/2024 - 18:18:19: [SM] Exception reported: Language phrase "Not enough arguments" not found (arg 4)
L 01/06/2024 - 18:18:19: [SM] Blaming: translations.smx
L 01/06/2024 - 18:18:19: [SM] Call stack trace:
L 01/06/2024 - 18:18:19: [SM]   [0] ReplyToCommand
L 01/06/2024 - 18:18:19: [SM]   [1] Line 30, translations.sp::CommandTranslation
```