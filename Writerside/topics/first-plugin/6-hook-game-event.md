# 6. ゲームイベントをHookする

## ゲームイベントって何？

ゲームイベントは、ゲームごとに用意されている様々な事柄が起きた際に呼び出されるイベントです。

CS:GOを例に出すと以下のようなイベントがあります

* `player_spawned` | プレイヤーがスポーンした時のイベント
* `player_ping` | プレイヤーがplayer_pingコマンドを使用した時のイベント
* `bomb_planted` | 爆弾が設置された時のイベント
* `round_start` | ラウンドが始まったときのイベント

上で挙げたのはごく一部ですが、ゲームイベントは色々な用途に使うことが出来ます。

## TL;DR

この記事でわかるもの
* ゲームイベントをHookする方法
* ゲームイベントをキャンセルする方法
* ゲームイベントから値を取得する方法

今回解説した内容の実装例は以下になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/hook-game-event.sp)からも確認できます)
```C++
#include <sourcemod>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "[CS:GO] Hook game event",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
    // キャンセルしないHook
    HookEvent("player_spawned", OnPlayerSpawned, EventHookMode_Post);
    // キャンセルするHook
    HookEvent("player_ping", OnPlayerPing, EventHookMode_Pre);  
}

public void OnPlayerSpawned(Event event, const char[] name, bool dontBroadcast) {
    int client = GetClientOfUserId(GetEventInt(event, "userid"));
    PrintToChatAll("%N spawned!", client);
}

public Action OnPlayerPing(Event event, const char[] name, bool dontBroadcast) {
    bool isUrgent = GetEventBool(event, "urgent", false);
    if(isUrgent) {
        return Plugin_Handled;
    }
    return Plugin_Continue;
}
```

## イベントをHookする

`HookEvent`関数を使用してイベントをHookすることが出来ます。 [(公式ドキュメント)](https://sm.alliedmods.net/new-api/events/HookEvent)

```C++
HookEvent(const char[] name, EventHook callback, EventHookMode mode)
```

<procedure title="name" id="val_evt_name">
    <p>イベントの名称を指定します。</p>
    <p>基本的にはスペースを<code>_</code>で置き換え、アルファベットを全て小文字にします。</p>
</procedure>

<procedure title="callback" id="val_evt_callback">
    <p>イベントのコールバック関数を指定します。</p>
    <p>このイベントのコールバックには引数が同じの<code>Action</code>型と<code>void</code>型の二種類の関数が存在します。</p>
    <p><code>Action(Event event, const char[] name, bool dontBroadcast)</code></p>
    <p><code>void(Event event, const char[] name, bool dontBroadcast)</code></p>
</procedure>

<procedure title="EventHookMode" id="val_evt_hookmode">
    <p>イベントのHookモードを指定します。</p>
    <p>Hookモードには以下の三種類があり以下のような特徴があります。</p>
    <list>
    <li>
    <p>[型:Action] <code>EventHookMode_Pre</code>: このモードでHookしたイベントはAction型の関数を使用してキャンセルすることが出来ます。</p>
    </li>
    <li>
    <p>[型:void] <code>EventHookMode_Post</code>: このモードでHookしたイベントはキャンセルすることは出来ませんが、Eventから特定の値を取得できます。</p>
    </li>
    <li>
    <p>[型:void] <code>EventHookMode_PostNoCopy</code>: このモードでHookしたイベントはキャンセルすることもeventの値がnullのためイベントの値を取得することも出来ません。故に一番最速です。</p>
    </li>
    </list>
</procedure>

ゲームイベントはゲームごとに違うものが用意されていますが、今回はCS:GOのイベントをHookしてみようと思います。 [(ゲームイベント一覧はここから確認できます。)](https://wiki.alliedmods.net/Game_Events_(Source))

## キャンセルしないHook {id=non_cancel_hook}

まずはキャンセルしない(出来ない)Hookの方を作っていきましょう。

プレイヤーがスポーンした際にプレイヤーの名前を全体チャットでお知らせする機能を試しに作ります。

### 1. イベントをHookする

以下の様な形でイベントをHookします。 [(player_spawned)](https://wiki.alliedmods.net/Counter-Strike:_Global_Offensive_Events#player_spawned)

```C++
HookEvent("player_spawned", OnPlayerSpawned, EventHookMode_Post)
```

Callback関数は以下のようにします。

```C++
public void OnPlayerSpawned(Event event, const char[] name, bool dontBroadcast)
```

### 2. イベントから値を取る

[player_spawned](https://wiki.alliedmods.net/Counter-Strike:_Global_Offensive_Events#player_spawned)イベントには`userid`と`inrestart`がイベントの値として存在しています。

プレイヤーのuseridを取得したいため、[GetEventInt](https://sm.alliedmods.net/new-api/events/GetEventInt)関数を使用して`userid`を取得します。

```C++
GetEventInt(event, "userid")
```

### 3. useridからユーザーのindexを取得する

useridはSource Engine内部で使用されているidのためSourceModで使用できるPlayer indexに変換する必要があります。

useridからPlayer indexを取得するには[GetClientOfUserId](https://sm.alliedmods.net/new-api/clients/GetClientOfUserId)関数を使用します。

```C++
int client = GetClientOfUserId(GetEventInt(event, "userid"))
```

### 4. チャットにユーザー名を出す。

%N指定子を使用してプレイヤー名をPrintChatToAllに入れてあげます。 詳細は[ここから](formatting.md)

```C++
PrintToChatAll("%N spawned!", client)
```

## キャンセルするhook {id=cancellable_hook}

キャンセルする方のHookを作っていきます。

player_pingの緊急pingだけキャンセルする機能を作ってみます。

### 1. イベントをHookする {id=hook_event_ping}

以下の様な形でイベントをHookします。[(player_ping)](https://wiki.alliedmods.net/Counter-Strike:_Global_Offensive_Events#player_ping)

```C++
HookEvent("player_ping", OnPlayerPing, EventHookMode_Pre)
```

Callback関数は以下のようにします。

```C++
public Action OnPlayerPing(Event event, const char[] name, bool dontBroadcast)
```

### 2. イベントから値を取る {id=get_event_value_ping}

[(player_ping)](https://wiki.alliedmods.net/Counter-Strike:_Global_Offensive_Events#player_ping)イベントにはかなりのイベント値がありますが、今回は`urgent`を取得して緊急pingかどうかを判定したいと思います。

今回は値の型がBoolのため、 [GetEventBool](https://sm.alliedmods.net/new-api/events/GetEventBool)関数を使用して値を取得します。

```C++
bool isUrgent = GetEventBool(event, "urgent", false)
```

### 3. イベントをキャンセルする {id=cancel_event_ping}

`isUrgent`がtrueの際はPlugin_Handledを返してイベントをキャンセルします。

```C++
if(isUrgent) {
    return Plugin_Handled;
}
```