# 前端和评测

整个 ***Saiblo*** 网站包括三个部分，即**前端**、**后端**与**评测端**。这个文档主要讲的是**前端**与**评测端**的交互。这个交互主要在**房间系统**上进行。

## 交互示意图

![用户与房间列表与评测端的交互](./svgs/roomlist2judger.svg)

![用户与房间与评测端的交互](./svgs/room2judger.svg)

图中如[①]的标号为协议编号。

## 交互过程与协议

以下对交互过程中的每一个**过程**与**协议**进行说明。

**所有的协议均基于 `Websocket` 通信，并且均采用 `JSON` 格式**。

**[①]** 1.从后端获取所有服务器的地址；2.选择某一个评测端服务器；3.使用`Websocket`与评测端建立连接。

**[②]** 1.确认已经连接到某一个评测端服务器；2.用户点击创建房间，一并将房间信息发送给评测端。

```json
{
    'type': 'create_room', //指令类型
    'user': user_object, //创建房间的用户信息，与后端user结构相同
    'game': game_object, //创建房间的游戏信息，与后端game结构相同
    'private': Boolean //房间是否为私密房间
}
```

**[③]** 1.由于其他用户的操作使得房间列表发生变化时，评测端将会推送列表信息的更新给前端。

```json
{
    'type': 'push_room_list', //指令类型
    'room': room_list //房间列表
}
```

`room_list`中每一个元素结构如下。

```json
{
    'id': room_id, //房间id
    'host': host_username, //房主的用户名
    'users': user_object_list, //房间内用户列表，每一个元素均为user_object，与后端user结构相同
    'game': game_id, //游戏id
    'private': Boolean //是否私密
}
```

**[④]** 1.确认已经连接到某一个评测端服务器；2.用户点击加入某一个房间，将这一信息发送给评测端。

```json
{
    'type': 'add_room', //指令类型
    'user': user_object //加入房间的用户信息，与后端user结构相同
}
```

**[⑤]** 1.用户向评测端发送了创建房间的请求；2.评测端创建好了房间，告知前端已创建房间，令前端自动跳转到房间页面。

```json
{
    'type': 'room_created', //指令类型
    'user': username, //创建房间的用户的用户名
    'room': room_id //房间id，用以根据id进入到房间页面
}
```

**[⑥]** 1.用户通过`Websocket`与评测端建立连接。`Websocket`地址的计算方式为`JUDGE_IP + ":" + JUDGE_PORT + "/" + ROOM_ID + "/" + USER_NAME`。

**[⑦]** 1.确认已经连接到评测端；2.用户点击房间中的某一个座位作为人类玩家加入游戏或者退出该座位，将这一信息发送给评测端。

```json
{
    'type': 'ready_human', //加入人类玩家
    'user': username, //加入座位的用户的用户名
    'seat': seat //加入的座位编号
}

{
    'type': 'cancel_human', //人类玩家退出座位
    'user': username, //退出座位的用户的用户名
    'seat': seat //退出的座位编号
}
```

**[⑧]** 1.确认已经连接到评测端；2.用户点击房间中的某一个座位添加一个AI加入游戏或者是踢出这个座位上的AI，将这一信息发送给评测端。

```json
{
    'type': 'ready_ai', //加入AI
    'seat': seat, //座位编号
    'ai': ai_object //AI的信息
}

{
    'type': 'cancel_ai', //踢出AI
    'seat': seat //踢出的座位编号
}
```

`ai`结构为后端的`entity`，再附加上`code`作为`ai.code`。

**[⑨]** 1.当房间信息有变动时，评测端将会推送新的房间信息给用户。

```json
{
    'type': 'status', //指令类型
    'users': user_object_list, //房间内用户列表
    'players': player_list, //房间内玩家列表(指在座位上的玩家)
    'game': game_id, //游戏id
    'game_status': game_status, // 游戏是否开始
    'configs': { //房间的配置信息
        'attr1': { 'name': String, 'value': defaultvalue1, 'hint': String }, //配置名、默认值、配置显示的内容
        'attr2': { 'name': String, 'value': defaultvalue2, 'hint': String },
        ...
    }
}
```

**[⑩]** 1.当游戏开始时，评测端会推送一次房间信息，其中`game_status`为`true`。

本质上是房间信息的更新，通过判断房间信息中的`game_status`来判断是否已经开始游戏。

**[⑪]** 1.确认已经连接到评测端；2.房主点击某个座位踢出该座位上的玩家，将这一信息发送给评测端。

实际上就是玩家退出座位的命令。只不过房主可以令任何人退出座位。

**[⑫]** 1.确认已经连接到评测端；2.房主修改房间内的一些信息，例如房间人数，游戏设置等等，将这一信息发送给评测端。

```json
{
    'type': 'edit_game_configs', //指令类型
    'configs': { //配置的内容
        'attr1': value1,
        'attr2': value2,
        ...
    }
}
```

**[⑬]** 1.确认已经连接到评测端；2.确认所有玩家均已准备；3.房主点击开始游戏。

```json
{
    'type': 'start' //指令类型
}
```

**[⑭]** 1.确认已经连接到评测端；2.玩家点击切换网页播放器/客户端播放器；3.若切换为网页播放器，前端向网页播放器发送链接评测端的信息；若切换为客户端播放器，则断开网页播放器，等待客户端播放器的链接。
