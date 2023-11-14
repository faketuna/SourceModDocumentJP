# 5. ConVarを活用する

今回はConVarの作成、検索、値の取得方法について書いていきたいと思います。

## ConVarとは

ConVarとは`Console Variable`の略で、一般的には変更可能な設定値などを保存しておく用途に使用されています。

また、大抵の場合はConVarではなくcvarと呼ばれることが多いです。 (一例: SourceModのConVarを変更するコマンドが`sm_cvar`として登録されてる)

## Configとは何が違うの？

ConVarはConfigとは違い、ゲーム内で設定値をリアルタイムに変更/反映が可能な点が大きく異なります。 (一応Configでも同じ様なことはやろうと思えばできます。面倒ですが...)

## TL;DR

この記事でわかるもの
* ConVarを作成する方法
* ConVarを検索する方法
* ConVarの値を取得する方法
* ConVarの値を設定する方法
* ConVarの値の変更をHookする方法
* ConVarのFlagの理解

今回解説した内容の実装例は以下になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/convar.sp)からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

ConVar g_cvPluginEnabled;

public Plugin myinfo =
{
    name = "ConVar",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    g_cvPluginEnabled = CreateConVar("sm_plugin_enabled", "1", "Disable/Enable Plugin", FCVAR_NONE, true, 0.0, true, 1.0);
    g_cvPluginEnabled.AddChangeHook(OnCvarChanged);

    RegConsoleCmd("sm_convar", CommandConVarTest, "ConVar test command");
    RegConsoleCmd("sm_setcvar", CommandCvarSet, "ConVar test command");
    RegConsoleCmd("sm_findcvar", CommandFindCvar, "ConVar test command");
}

public Action CommandConVarTest(int client, int args) {
    if(GetConVarBool(g_cvPluginEnabled)) {
        ReplyToCommand(client, "Plugin is currently enabled");
        return Plugin_Handled;
    }

    ReplyToCommand(client, "Plugin is currently disabled");
    return Plugin_Handled;
}

public Action CommandCvarSet(int client, int args) {
    if(args == 0) {
        ReplyToCommand(client, "Not enough arguments!");
        return Plugin_Handled;
    }
    int value;
    
    if(!GetCmdArgIntEx(1, value)) {
        ReplyToCommand(client, "Invalid value!");
        return Plugin_Handled;
    }

    SetConVarInt(g_cvPluginEnabled, value);
    return Plugin_Handled;
}

public Action CommandFindCvar(int client, int args) {
    if(args == 0) {
        ReplyToCommand(client, "Not enough arguments!");
        return Plugin_Handled;
    }
    
    char conVarName[64], conVarValue[32];
    ConVar find;
    
    GetCmdArg(1, conVarName, sizeof(conVarName));

    find = FindConVar(conVarName);

    if(find == INVALID_HANDLE) {
        ReplyToCommand(client, "Specified ConVar not found!");
        return Plugin_Handled;
    }

    GetConVarString(find, conVarValue, sizeof(conVarValue));
    ReplyToCommand(client, "ConVar found: %s\nValue: %s", conVarName, conVarValue);
    return Plugin_Handled;
}

public void OnCvarChanged(ConVar convar, const char[] oldValue, const char[] newValue) {
    char cvarName[64];
    convar.GetName(cvarName, sizeof(cvarName));
    PrintToChatAll("ConVar: %s value changed to %s!", cvarName, newValue);
}
```

## ConVarの作成 {id=create_cvar}

ConVarを作成するには以下の手順で作成を行います。

### 1. ConVarの情報を格納する変数を作成する

今回は`ConVar g_cvPluginEnabled`を上の方に記述し、グローバル変数を宣言します。

> Tips
> 
> グローバル変数は必然的にスコープが長くなるので、わかりやすいようにprefixを付けるのをおすすめします。
> 
> 名付けるならこれ！みたいなものはないんですが、自分の場合は以下のルールで変数名のprefixを決めています。
> 
> グローバルであることを明示(`g_`),値の型(`cv`), 変数名(`PluginEnabled`)
> 
> 上のルールで行くとint型のグローバル変数であれば `g_iVariableName`となります。

### 2. ConVarを作る

`CreateConVar`関数を使用してConVarを作成します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/convars/CreateConVar)

```C++
CreateConVar(const char[] name, const char[] defaultValue, const char[] description, int flags, bool hasMin, float min, bool hasMax, float max)
```

今回の場合は以下の様に作成します。
```C++
CreateConVar("sm_plugin_enabled", "1", "Disable/Enable Plugin", FCVAR_NONE, true, 0.0, true, 1.0);
```


<procedure title="name" id="val_name">
    <p>sm_cvarやサーバーコンソールから指定する際に必要なConVarの名前です。</p>
    <p>今回はプラグインの有効無効状態を保持したいので、<code>sm_plugin_enabled</code>とします。</p>
</procedure>

<procedure title="defaultValue" id="val_defaultValue">
    <p>ConVarを作成する際の既定の値です。</p>
    <p>今回はデフォルトでは有効にしたいため <code>1</code>とします。</p>
</procedure>

<procedure title="description" id="val_description">
    <p>コンソールかsm_rcon等からConVarの名前だけを入れて実行した際に出てくる説明文です。</p>
    <p>今回はPluginの有効無効を切り替えるConVarであるのを明記するために<code>"Disable/Enable Plugin"</code>と記載しています。</p>
</procedure>

<procedure title="flags" id="val_flags">
    <p>ConVarに関する様々なフラグを設定することができます。 </p>
    <p>詳細は記事下部で解説しています。</p>
</procedure>

<procedure title="hasMin, min" id="val_min">
    <p>値の最小範囲の有効無効, 値を設定します。</p>
    <p>今回は、0か1だけを入れてほしいため <code>hasMin</code>を<code>true</code> <code>min</code>を<code>0</code>とします。</p>
</procedure>


<procedure title="hasMin, min" id="val_max">
    <p>値の最大範囲の有効無効, 値を設定します。</p>
    <p>今回は、0か1だけを入れてほしいため <code>hasMax</code>を<code>true</code> <code>max</code>を<code>1</code>とします。</p>
</procedure>

### 3. ConVarのHandleを変数に入れる

`CreateConVar`関数はConVarのHandleを返してくれるため以下のような形で、変数にConVarのHandleを入れることができます。

```C++
g_cvPluginEnabled = CreateConVar("sm_plugin_enabled", "1", "Disable/Enable Plugin", FCVAR_NONE, true, 0.0, true, 1.0);
```

### 4. 完成

これでConVarが出来上がりました。

## ConVarの検索 {id=find_cvar}

ConVarを検索することによって、後からでも特定のConVarのHandleを取得することができます。

今回はコマンドからConVarの内容を取得するコマンドを作成してみましょう。

### 1. コマンドを作る/登録する

前回と前々回に解説したためコマンドのCallback関数の作成と、Registerは飛ばします。 (忘れてしまった方は[ここを見てください](3-first-command.md))

### 2. ConVarを検索する {id=search_cvar}

ConVarを検索するには`FindConVar`関数を使用します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/convars/FindConVar)

```C++
FindConVar(const char[] name)
```

`FindConVar`関数は、見つかった場合は該当するConVarのHandleを返し、見つからなかった場合は`INVALID_HANDLE`を返します。

> Tips
> 
> コード内の処理でConVarが見つからなかった場合の切り分けを行いたい場合は
> 
> `if(cvar == INVALID_HANDLE)`
> 
> を使用して見つからなかった場合の処理を行うことができます。

## ConVarから値を取得する {id=get_cvar_value}

ConVarから値を取得するには、以下のいずれかの関数を使用して取得できます。

* [`GetConVarBool`](https://sm.alliedmods.net/new-api/convars/GetConVarBool)
* [`GetConVarFloat`](https://sm.alliedmods.net/new-api/convars/GetConVarFloat)
* [`GetConVarInt`](https://sm.alliedmods.net/new-api/convars/GetConVarInt)
* [`GetConVarString`](https://sm.alliedmods.net/new-api/convars/GetConVarString)

<procedure title="Bool, Float, Intの取得" id="cvar_get_value_primitive">
    <p>Bool, Float, Intの様なプリミティブ型でConVarの内容を取得したい際は</p>
    <step>プリミティブ型の変数を宣言 (例: <code>int value</code>)</step>
    <step><code>GetConVar**</code>関数を使用し返り値を格納する</step>
    <p>例: <code>value = GetConVarInt(g_cvPluginEnabled)</code></p>
</procedure>

<procedure title="文字列の取得" id="cvar_get_value_string">
    <p>文字列としてConVarの値を取得したい場合は</p>
    <step>文字列を格納する変数の宣言 (例: <code>char value[32]</code>)</step>
    <step><code>GetConVarString</code>関数を使用し文字列の変数に格納してもらう</step>
    <p>例: <code>GetConVarString(g_cvPluginEnabled, value, sizeof(value))</code></p>
</procedure>

## ConVarの値を設定する {id=set_cvar_value}

ConVarの値を設定するには、以下のいずれかの関数を使用して設定することができます。

* [`SetConVarBool`](https://sm.alliedmods.net/new-api/convars/SetConVarBool)
* [`SetConVarFloat`](https://sm.alliedmods.net/new-api/convars/SetConVarFloat)
* [`SetConVarInt`](https://sm.alliedmods.net/new-api/convars/SetConVarInt)
* [`SetConVarString`](https://sm.alliedmods.net/new-api/convars/SetConVarString)

<procedure title="ConVarを設定する例" id="cvar_set_value_primitive">
    <p>ConVarの内容を設定するには以下の方法を使用します。</p>
    <step><code>SetConVar**(ConVar, Value)</code></step>
    <p>例: <code>SetConVarInt(g_cvPluginEnabled, 1)</code></p>
    <p>例2: <code>SetConVarString(g_cvPluginEnabled, "Value!")</code></p>
</procedure>

## ConVarの変更をHookする {id=hook_cvar_change}

ConVarの内容が変更された際、変更後の値を他のConVarにSyncしたい時や、変更後の値を別の変数に入れておきたい事があると思います。

その様な場合は、ConVarをHookすると便利です。

ConVarをHookするには[`HookConVarChange`](https://sm.alliedmods.net/new-api/convars/HookConVarChange)関数を使用するか、[`AddChangeHook`](https://sm.alliedmods.net/new-api/convars/ConVar/AddChangeHook)メソッドを使用します。

### CallBack関数を作成する {id=hook_cvar_callback}

CallBack関数は以下のような形で定義します。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/convars/ConVarChanged)

```C++
void(ConVar convar, const char[] oldValue, const char[] newValue)
```

この様に実装することができます。

```C++
public void OnCvarChanged(ConVar convar, const char[] oldValue, const char[] newValue)
```

### ConVarをHookする

> Note
>
> どちらも使い方に大差ありませんが、`AddChangeHook`のほうが短くかつ簡易的に記述できるためこちらをおすすめします。
{style="note"}

<procedure title="HookConVarChange関数を使用したHook" id="hook_cvar_use_func">
    <p>以下のような形で使用できます。</p>
    <p><code>HookConVarChange(g_cvPluginEnabled, OnCvarChanged)</code></p>
</procedure>

<procedure title="AddChangeHook" id="hook_cvar_use_method">
    <p>以下のような形で使用できます。</p>
    <p><code>g_cvPluginEnabled.AddChangeHook(OnCvarChanged)</code></p>
</procedure>

## ConVarのFlagについて

Flagの一覧は[`console.inc`](https://github.com/alliedmodders/sourcemod/blob/11c8084ccd530290df459e12b50e5852f777ddee/plugins/include/console.inc#L68-L103)にありますが、今回は比較的使う物のみ紹介します。

* `FCVAR_NONE` 名前の通り何もないフラグです。特に付けるフラグはないけどConVarの最小値と最大値を決めたい時にflagを指定しないといけない時などに使います。(一応最小値とかは直接指定もできるので、使わなくても実は行ける)
* `FCVAR_NOTIFY` ConVarが変更された時にプレイヤーに通知します。 よく見る `サーバーの CVAR 値 'sv_cheats' を 1 に変更しました`の様なメッセージを出すのに使います。
* `FCVAR_DONTRECORD` `AutoExecConfig`を使用した際にConVarの情報が保存されなくなります、プラグインのバージョンを表示するConVarを作成した際等に役立ちます。 また、DemoファイルにConVarの変更が保存されなくなります。
* `FCVAR_PROTECTED` クライアントがConVarの情報を問い合わせても正しい情報を返さないようにします、これによりConVarにパスワードを保存する等ができるようになります。