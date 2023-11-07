# sdkhooks

## 概要

sdkhooksはゲーム内の現象(ダメージを受けた時等)をhookすることができます。

hookできる内容は[ここ](https://sm.alliedmods.net/new-api/sdkhooks/SDKHookType)から確認できます。

## 使用方法

今回は落下ダメージをキャンセルするプラグインを書いていきたいと思います。

## TL;DR

今回出来上がるコードは以下の内容になります。 ([リポジトリ](https://github.com/faketuna/sm-Example-Plugins/blob/main/scripting/no-fall-damage.sp)からも確認できます)
```C++
#include <sourcemod>
#include <sdkhooks>

#pragma semicolon 1
#pragma newdecls required

#define PLUGIN_VERSION "0.0.1"

public Plugin myinfo =
{
    name = "No fall damage",
    author = "faketuna",
    description = "",
    version = PLUGIN_VERSION,
    url = ""
}

public void OnPluginStart()
{
}

public void OnClientPutInServer(int client)
{
    SDKHook(client, SDKHook_OnTakeDamage, OnTakeDamage);
}

public void OnClientDisconnect(int client)
{
    SDKHook(client, SDKHook_OnTakeDamage, OnTakeDamage);
}

public Action OnTakeDamage(int client, int &attacker, int &inflictor, float &damage, int &damagetype)
{
    if(damagetype & DMG_FALL)
    {
        PrintToChat(client, "Damage cancelled!");
        return Plugin_Handled;
    }
    return Plugin_Continue;
}
```

## 詳細解説

### SDKHook()

SDKHook関数の使い方は以下のとおりです。([元のドキュメントリンク](https://sm.alliedmods.net/new-api/sdkhooks/SDKHook))
```C++
SDKHook(int entity, SDKHookType type, SDKHookCB callback)
```

第1引数は、エンティティのIndexを入れる必要があります。

第2引数は、SDKHookのTypeを指定する必要があります。一覧は[ここ](https://sm.alliedmods.net/new-api/sdkhooks/SDKHookType)から

第3引数は、Hookした内容をどの関数で処理するかを指定します。 使用する関数は[ここ](https://sm.alliedmods.net/new-api/sdkhooks/SDKHookCB)で対応する記述にする必要があります。

今回はダメージをキャンセルする必要があるため、voidではなくAction型の関数を作成します。

```C++
function Action(int victim, int& attacker, int& inflictor, float& damage, int& damagetype)
```

### SDKUnhook()

バグ回避のためにプレイヤーが切断した際はUnhookしましょう。

記述する内容は`SDKHook()`と同じで構いません。
```C++
SDKUnhook(int entity, SDKHookType type, SDKHookCB callback)
```

### 落下ダメージをキャンセルする

`if(damagetype & DMG_FALL)`でbit演算を使用して、落下ダメージのフラグが含まれているかを確認します。

フラグが含まれていた場合は`return Plugin_Handled`を返しダメージをキャンセルします。

その他の場合は`return Plugin_Continue`を返し通常通りの動作をしてもらいます。