# 3. 初めてのコマンド!

ほとんどのプラグインでは必要となるであろうコマンドの作り方です。

### TL;DR

今回作る物

* 通常のConsoleコマンド
* Admin専用コマンド

今回出来上がるコードは以下の内容になります。 ([リポジトリ]()からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "Command",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    RegConsoleCmd("sm_test", CommandTest, "Test command");
    RegConsoleCmd("cmd_test", CommandTest, "Test command");

    RegAdminCmd("sm_admtest", CommandAdminTest, ADMFLAG_BAN, "Admin test command");
}

public Action CommandTest(int client, int args) {
    ReplyToCommand("Test command!");
    return Plugin_Handled;
}

public Action CommandAdminTest(int client, int args) {
    ReplyToCommand("Test admin command!");
    return Plugin_Handled;
}
```

## 通常のコマンドを作成する

### コマンドを受け付けるCallback関数を作成する

コマンドをRegisterするにはCallback関数が必須です。 今回はわかりやすさ重視で行きたいので最初に作成します。

#### 1. Callback関数を定義する

Callback関数の定義は以下の形で定義を行います。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/ConCmd)

```C++
public Action FunctionName(int client, int args)
```

#### 2. 明示的なreturnを行う

次に、Action型は明示的なreturnを要求するため、コマンドが正常に終わったことを伝える `return Plugin_Handled` を返します。

```C++
return Plugin_Handled;
```

#### 3. できたもの

これらを組み合わせると以下のような形になるはずです。(関数名は自由)

```C++
public Action CommandTest(int client, int args) {
    return Plugin_Handled;
}
```

#### 4. コマンドに対して返信を行う

これでCallback関数は完成ですが、実行した際にちゃんと動作しているかわからないためコマンドに対して返信を行ってみましょう。

今回は`ReplyToCommand`関数を使用してコマンド使用者に`Test command!`というメッセージを送信したいと思います。

`ReplyToCommand`関数は以下のような形で使用できます。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/ReplyToCommand)

```C++
ReplyToCommand(int client, const char[] format, any... ...);
```

上記の内容を踏まえて作成すると、完成形は以下のような形になるはずです。 これでCallback関数の作成は完了しました。

```C++
public Action CommandTest(int client, int args) {
    ReplyToCommand(client, "Test command!");
    return Plugin_Handled;
}
```

### コマンドをregisterする

Callback関数が完成したらあとはプラグインのロード時にregisterすればゲーム内で利用可能になります。 

以下の形でregisterを行うことができます。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/RegConsoleCmd)

```C++
RegConsoleCmd(const char[] cmd, ConCmd callback)
```

```C++
RegConsoleCmd(const char[] cmd, ConCmd callback, const char[] description)
```

今回の場合は以下のような形になるはずです

```C++
RegConsoleCmd("sm_test", CommandTest, "Test command")
```

上の`RegConsoleCmd`をOnPluginStart()関数に入れてあげるとコマンドをregisterすることができます。

> Tips
>
> コマンド名は`sm_`で始めると、(例: `sm_test`) チャットでコマンドを打つ際に`!test`として実行が可能になります。
>
> コマンド名を`sm_`以外で始めると、 (例: `cmd_test`) チャットでコマンドを打つ際に `!cmd_test`のように全体を打つ必要が出てきます。
> 
> コンソールからの実行はこれの影響を受けず、どちらともフルで入れる必要があります。(例: `sm_test`, `cmd_test`)

```C++
public void OnPluginStart()
{
    RegConsoleCmd("sm_test", CommandTest, "Test command");
    RegConsoleCmd("cmd_test", CommandTest, "Test command");
}
```

これでゲーム内で`!test`コマンドが使用できるようになりました。

## Admin専用コマンドを作成する

### コマンドのCallback関数を作成する {id="callback_admin"}

Adminコマンドを作る際も通常のコマンドを作る方法と全く同じでCallback関数を作成するため、ここではスキップします。

今回使用するCallback関数は以下のとおりです。

```C++
public Action CommandAdminTest(int client, int args) {
    ReplyToCommand(client, "Test admin command!");
    return Plugin_Handled;
}
```

### Adminコマンドをregisterする

Adminコマンドのregisterは`RegAdminCmd`関数を使用してregisterします。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/console/RegAdminCmd)

```C++
RegAdminCmd(const char[] cmd, ConCmd callback, int adminflags, const char[] description, const char[] group, int flags)
```

この関数はほとんど`RegConsoleCmd`と同じなのでそこまで難しくありません。 

#### 使用したいAdminフラグを探す {id="find_admin_flags"}

Adminのflagは[ここ](https://github.com/alliedmodders/sourcemod/blob/7747beb4cffa90bf42d40e9ca27de6e85258ee55/plugins/include/admin.inc#L71-L91)で見つけられます。

今回はBanする権限を持っている人のみコマンドを使用できるようにしたいため、`ADMFLAG_BAN`を指定します。

> tips
> 
> 複数のフラグを指定したい場合は 
> 
> `ADMFLAG_BAN | ADMFLAG_RCON`
> 
> のように `|`で区切ってあげると複数のフラグを指定することができます。

結果としてRegAdminCmdの内容は以下のようになるはずです。

```C++
RegAdminCmd("sm_admtest", CommandAdminTest, ADMFLAG_BAN, "Admin test command");
```

あとは`OnPluginStart`関数に入れてあげれば、Admin専用コマンドが使えるようになります。

