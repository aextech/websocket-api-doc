AEX Websocket API 协议说明文档   
---
[英文文档](/english.md)

# API 请求 URL
```
URL: wss://api.aex.zone/ws/v1

目前 Websocket API 只针对 api.aex.zone 域名提供。
```

# Websocket 连接处理过程
```
同一个账号，同一时间只能有一个连接存活，后面认证成功的连接会把其他连接断开。在未认证的情况下，同一个IP允许多有个连接，
比如：
  1）IP1 连接上websocket
  2）IP1 发起auth请求且成功
  3）IP2 连接上websocket
  4）IP2 发起auth请求且成功
  5）IP1 上已auth成功的连接会被服务端主动断开连接
```

# API 请求 限制
+ 同一个IP最多10个Websocket连接
+ 1个Websocket连接每秒最多10个请求，下一秒重新对请求计数
+ 1个Websocket连接最多同时关注10个交易对

# 目录
+ [协议命令字](#协议命令字)
+ [错误码](#错误码)
+ [协议请求/应答数据结构(json)](#协议请求应答数据结构)
   + [CMD: 1, 深度变化通知, 服务器主动通知客户端](#cmd-1-深度变化通知-服务器主动通知客户端)
   + [CMD: 2, 关注交易对(最多关注10个)](#cmd-2-关注交易对最多关注10个)
   + [CMD: 4, 签名认证](#cmd-4-签名认证)
   + [CMD: 5, 获取余额](#cmd-5-获取余额)
   + [CMD: 6, 挂单](#cmd-6-挂单)
   + [CMD: 7, 撤单](#cmd-7-撤单)
   + [CMD: 8, 根据订单ID查询订单信息](#cmd-8-根据订单id查询订单信息)
   + [CMD: 9, 根据成交记录ID查询成交记录](#cmd-9-根据成交记录id查询成交记录)
   + [CMD: 10, 查询交易对信息](#cmd-10-查询交易对信息)
   + [CMD: 11, 根据tag查询订单](#cmd-11-根据tag查询订单)
   + [CMD: 12, 根据tag查询成交记录](#cmd-12-根据tag查询成交记录)
   + [CMD: 13, 查询有效交易对列表](#cmd-13-查询有效交易对列表)
   + [CMD: 14, 查询指定交易对的成交记录](#cmd-14-查询指定交易对的成交记录)
   + [CMD: 15, 查询指定交易对或者指定市场的行情数据](#cmd-15-查询指定交易对或者指定市场的行情数据)
+ [订单和成交记录的关联](#订单和成交记录的关联)
+ [交易策略的实现](#交易策略的实现)


# 协议命令字
CMD   | 说明
----- | ---------
1     | 深度变化通知，服务器主动通知客户端，服务器只通知已经通过2命令关注的交易对
2     | 添加想要关注的交易对，删除已经关注的交易对
3     | K线数据（尚未实现）
4     | 签名认证
5     | 获取所有币种余额
6     | 下订单
7     | 撤销订单
8     | 根据订单ID查询单个订单信息
9     | 根据成交ID查询单个成交记录
10    | 根查询交易对信息
11    | 根据tag查询多个订单信息
12    | 根据tag查询多个成交记录信息
13    | 查询所有有效的交易对
14    | 查询指定交易对的成交记录
15    | 查询指定交易对或者指定市场的行情数据


# 错误码
错误码 | 说明
-----  | ---------
0      | 正常
xx+001 | ================ 系统错误码 ================
1001   | 系统错误
4001   | 无效请求，可能是请求里把整数当做字符串处理了
5001   | 未认证
6001   | 已认证
7001   | 系统繁忙
8001   | 计价币不存在
9001   | 交易币不存在
10001  | 无效交易区
11001  | 该币在当前交易区不可交易
12001  | 请求频率限制
xx+002 | ================ 关注错误码 ================
1002   | 关注数量超过限制    
xx+006 | ================ 挂单错误码 ================
1006   | 用户ID无效
2006   | 无效交易类型
3006   | 价格无效
4006   | 数量无效
5006   | 数量点位超过限制
6006   | 数量超过最大限制
7006   | 数量超过最小限制
8006   | 价格超过最大限制
9006   | 价格点位超过限制
10006  | 交易金额小于最小限制
11006  | 余额不足
12006  | 数量精度配置错误
13006  | 价格精度未配置,或者配置错误
14006  | 输入参数错误
xx+007 | ================ 撤销订单错误码 ================
1007   | 用户ID无效
2007   | 订单不存在
3007   | 订单删除失败, 可能是订单被撮合, 或者已经被删除
4007   | 订单ID无效
xx+008 | ================ 查询订单错误码 ================
1008   | 订单不存在
xx+009 | ================ 查询成交记录错误码 ================
1009   | 成交记录不存在
10080 | 交易使用v1版本
10081 | 交易使用v2版本
1008000 | 交易使用v1版本
1008001 | 交易使用v2版本


# 协议请求应答数据结构
##### CMD: 1, 深度变化通知, 服务器主动通知客户端
```
请求结构: 
无

应答结构:
{
  "cmd": {
    "type": 1, // 命令类型为1
    "eno":  0  // 错误码始终返回0
  },
  "market": "cnc", // 交易区, 定价币
  "coin":   "btc", // 交易币
  "bids": [ // 买单深度
    {"price": 32076, "amount": 742.799405}, 
    {"price": 32000, "amount": 79253.604638},
    ...
  ],
  "asks": [// 卖单深度
    {"price": 32077, "amount": 1}, 
    {"price": 32153, "amount": 2},
    ...
  ]
}
```

##### CMD: 2, 关注交易对(最多关注10个)
```
请求结构: 
{
  "cmd": {
    "type": 2 // 关注交易对，类型：整数
  },
  "type": 1,  // 关注类型: 1=添加关注的交易对, 2=删除关注的交易对，类型：整数
  "pairs": [{ // 准备关注的交易对数组(可1个以上)
    "market": "cnc",  // 类型：字符串
    "coin": "btc"     // 类型：字符串
  }, {
    "market": "cnc",  // 类型：字符串
    "coin": "eos"     // 类型：字符串
  }]
}

应答结构:
{
  "cmd": {
    "type": 2, // 关注交易对
    "eno": 0   // 错误码
  },
  "type": 1,  // 关注类型: 1=添加关注的交易对, 2=删除关注的交易对 
  "pairs": [{ // type=1是关注成功的交易对数组（可以为空）, type=2是删除成功的交易对数组(可以为空)
    "market": "cnc",
    "coin": "btc"
  }, {
    "market": "cnc",
    "coin": "eos"
  }]
}
```

##### CMD: 4, 签名认证
```
请求结构: 
{
  "cmd": {
    "type": 4 // 签名认证, 类型：整数
  },
  "key": "xxxxxxxxx",    // 公钥，类型：字符串
  "time": 1545727168,    // 时间戳,单位秒,类型：整数
  "md5": "xxxxxxxxxxxxx" // md5： md5(key_用戶ID_skey_time)，順序不可顛倒，类型：字符串
}

应答结构:
{
  "cmd": {
    "type": 4, // 签名认证
    "eno": 0   // 错误码
  }
}
```

##### CMD: 5, 获取余额
```
请求结构: 
{
  "cmd": {
    "type": 5 // 获取余额，类型：整数
  }
}

应答结构:
{
  "cmd": {
    "type": 5, // 获取余额
    "eno": 0   // ENO_*
  },
  "balances": [{ // 余额数组
    "coin": "bkbt", // 币名
    "val": 116.99,  // 余额
    "locked": 0     // 冻结资金
  }, {
    "coin": "blk",   // 币名
    "val": 0.180533, // 余额
    "locked": 0      // 冻结资金
  }, {
    "coin": "btc",   // 币名
    "val": 0,        // 余额
    "locked": 0      // 冻结资金
  }]
}
```

##### CMD: 6, 挂单
```
请求结构: 
{
  "cmd": {
    "type": 6 // 挂单，类型：整数
  },
  "market": "gat", // 交易区, 定价币，类型：字符串
  "coin": "bkbt",  // 交易币，类型：字符串
  "tag": 2,        // 用户自定义tag, 一个非负整数, 用来把订单和成交记录关联起来，类型：整数
  "type": 1,       // 挂单类型: 1=买单, 2=卖单，类型：整数
  "price": 12.5,   // 价格，类型：浮点数
  "amount": 20     // 数量，类型：浮点数
}

应答结构:
{
  "cmd": {
    "type": 9, // 成交记录, 挂单请求成交的情况下会返回成交记录, 可能会有多个类型为9的应答
    "eno": 0   // 错误码
  },
  "data": {
    "market": "gat", // 交易区，定价币
    "coin": "bkbt",  // 交易币
    "type": 1,       // 1=买单成交,2=卖单成交
    "tag": 2,        // 用户自定义tag, 一个非负整数, 用来把订单和成交记录关联起来, 这里的tag就是请求中的tag
    "tradeid": 18,   // 成交记录ID
    "price": 10.5,   // 成交价格
    "amount": 57,    // 成交数量
    "fee": 0         // 手续费, type=1时是买单手续费, 以交易币为单位,收入= amount-fee; type=2时是卖单手续费,以定价币为单位,收入=price*amount-fee
  }
}
{
  "cmd": {
    "type": 6, // 挂单请求没有完全成交，剩下的部分生成订单
    "eno": 0   // 错误码
  },
  "data": {
    "market": "gat", // 交易区，定价币
    "coin": "bkbt",  // 交易币
    "type": 1,       // 1=买单，2=卖单
    "tag": 2,        // 用户自定义tag, 一个整数, 用来把订单和成交记录关联起来, 这里的tag就是请求中的tag
    "orderid": 17,   // 订单ID
    "price": 12.5,   // 订单价格
    "amount": 3      // 订单数量
  }
}
```

##### CMD: 7, 撤单
```
请求结构: 
{
  "cmd": {
    "type": 7 // 撤单，类型：整数
  },
  "market": "gat", // 交易区，定价币，类型：字符串
  "coin": "bkbt",  // 交易币，类型：字符串
  "orderid": 17    // 订单ID，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 7, // 撤单
    "eno": 0   // 错误码
  },
  "market": "gat",  // 交易区，定价币
  "coin": "bkbt",   // 交易币
  "orderid": 17     // 请求里的订单ID
}
```

##### CMD: 8, 根据订单ID查询订单信息
```
请求结构: 
{
  "cmd": {
    "type": 8 //  查询订单信息，类型：整数
  },
  "market": "gat", // 交易区，定价币，类型：字符串
  "coin": "bkbt",  // 交易币，类型：字符串
  "orderid": 17    // 准备查询的订单ID，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 8, // 查询订单信息
    "eno": 0   // 错误码
  },
  "data": {
    "market": "gat",   // 交易区，定价币
    "coin": "bkbt",    // 交易币
    "type": 1,         // 1=买单, 2=卖单
    "tag": 3,          // 用户自定义tag, 一个非负整数, 用来把订单和成交记录关联起来, 这里的tag就是挂单请求中的tag
    "orderid": 17,     // 订单ID
    "price": 12.5,     // 订单价格
    "amount": 3        // 订单剩余数量
    "time": 1547627993 // 挂单时间，时间戳，单位秒
  }
}
```

##### CMD: 9, 根据成交记录ID查询成交记录
```
请求结构: 
{
  "cmd": {
    "type": 9 // 查询成交记录，类型：整数
  },
  "market": "gat", // 交易区，定价币，类型：字符串
  "coin": "bkbt",  // 交易币，类型：字符串
  "tradeid": 18    // 准备查询的成交记录ID，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 9, // 根据成交ID返回成交记录，在自买自卖的情况下，一个请求会返回两个应答
    "eno": 0   // 错误码
  },
  "data": {
    "market": "gat",   // 交易区，定价币
    "coin": "bkbt",    // 交易币
    "type": 1,         // 1=买单成交记录, 2=卖单成交记录
    "tag": 3,          // 用户自定义tag, 一个非负整数, 用来把订单和成交记录关联起来, 这里的tag就是挂单请求中的tag
    "tradeid": 18,     // 成交记录ID
    "price": 10.5,     // 成交价格
    "amount": 57,      // 成交数量
    "fee": 0           // 手续费, type=1时是买单手续费, 以交易币为单位,收入= amount-fee; type=2时是卖单手续费,以定价币为单位,收入=price*amount-fee
    "time": 1547627993 // 成交时间，时间戳，单位秒
  }
}
```
说明：当用户的挂单成交的时候，在用户没有主动发请求查询成交记录的情况下，服务器也会使用cmd:9命令的应答结构主动通知用户成交记录。

##### CMD: 10, 查询交易对信息
```
请求结构: 
{
  "cmd": {
    "type": 10 // 查询交易对信息，类型：整数
  },
  "pairs": [{ // 准备查询的交易对数组
    "market": "cnc", // 交易区，定价币，类型：字符串
    "coin": "btc"    // 交易币，类型：字符串
  }, {
    "market": "usdt", // 交易区，定价币，类型：字符串
    "coin": "btc"     // 交易币，类型：字符串
  }]
}

应答结构:
{
  "cmd": {
    "type": 10, // 查询交易对信息
    "eno": 0    // 错误码
  },
  "pairs": [{ // 成功查询到数据的交易对数组
    "market": "cnc", // 交易区，定价币
    "coin": "btc",   // 交易币
    "precisions": [{ // 精度数组
      "name": "Price", // 挂单价格精度
      "num": 0         // 挂单精度位数
    }, {
      "name": "Amt", // 挂单数量精度
      "num": 6       // 挂单精度位数
    }],
    "valuelimits": [{ // 挂单数值限制数组
      "name": "AmtMax",   // 最大挂单下单数量
      "value": "10000000" //  最大挂单下单数量
    }, {
      "name": "AmtMin", // 最小挂单下单数量
      "value": "0.001"  // 最小挂单下单数量
    }, {
      "name": "PriceMax", // 最大挂单价格
      "value": "1000000"  // 最大挂单价格
    }, {
      "name": "MoneyMin", // 最小挂单金额
      "value": "1"        // 最小挂单金额
    }]
  }]
}
```

##### CMD: 11, 根据tag查询订单
```
请求结构: 
{
  "cmd": {
    "type": 11 // 根据tag查询订单，类型：整数
  },
  "market": "cnc",    // 交易区，定价币，类型：字符串
  "coin": "bitcny",   // 交易币，类型：字符串
  "tag": 3,           // 准备查询的tag，类型：整数
  "since_order_id": 1 // 起始订单ID，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 11, // 根据tag查询订单
    "eno": 0    // 错误码
  },
  "market": "cnc",  // 交易区，定价币
  "coin": "bitcny", // 交易币
  "tag": 3,         // 准备查询的tag
  "orders": [{ // 订单结果数组,1次最多返回10个订单
    "type": 1,         // 1=买单, 2=卖单
    "orderid": 53,     // 订单ID
    "price": 0.997,    // 订单价格
    "amount": 20,      // 订单当前剩余数量
    "time": 1547629377 // 挂单时间,时间戳,单位秒
  }]
}
```

##### CMD: 12, 根据tag查询成交记录
```
请求结构: 
{
  "cmd": {
    "type": 12 // 根据tag查询成交记录，类型：整数
  },
  "market": "cnc",    // 交易区，定价币，类型：字符串
  "coin": "bitcny",   // 交易币，类型：字符串
  "tag": 3,           // 准备查询的tag，类型：整数
  "since_trade_id": 1 // 起始成交记录ID，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 12, // 根据tag查询成交记录
    "eno": 0    // 错误码
  },
  "market": "cnc",   // 交易区，定价币
  "coin": "bitcny",  // 交易币
  "tag": 3,          // 准备查询的tag
  "trades": [{ // 成交记录数组，1次最多返回10条
    "type": 1,         // 1=买单, 2=卖单
    "tradeid": 1,      // 成交记录ID
    "price": 0.993,    // 成交价格
    "amount": 10,      // 成交数量
    "fee": 0,          // // 手续费, type=1时是买单手续费, 以交易币为单位,收入= amount-fee; type=2时是卖单手续费,以定价币为单位,收入=price*amount-fee
    "time": 1547627993 // 成交时间，时间戳，单位秒
  }]
}
```

##### CMD: 13, 查询有效交易对列表
```
请求结构: 
{
  "cmd": {
    "type": 13 // 查询有效交易对列表，类型：整数
  }
}

应答结构:
{
  "cmd":{
    "type":13, // 查询有效交易对列表
    "eno":0    // 错误码
  },
  "pairs":[  // 交易对数组
    {
      "market":"cnc", // 市场
      "coin":"xlm"    // 币名
    },
    {
      "market":"cnc", // 市场
      "coin":"qash"   // 币名
    }
  ]
}
```

##### CMD: 14, 查询指定交易对的成交记录
```
请求结构: 
{
  "cmd":{
    "type":14 // 查询指定交易对的成交记录，类型：整数
  },
  "pair":{
    "market":"cnc", // 市场，类型：字符串
    "coin":"btc"    // 币名，类型：字符串
  },
  "since_trade_id":1 // 从哪个成交记录ID开始查询，一次最多返回10条成交记录, 包含since_trade_id本身，类型：整数
}

应答结构:
{
  "cmd": {
    "type": 14, // 查询指定交易对的成交记录
    "eno": 0    // 错误码
  },
  "pair": {
    "market": "cnc", // 市场
    "coin": "btc"    // 币名
  },
  "trades": [ // 交易记录数组
    {
      "type":1,          // 买卖方向: 1=买, 2=卖
      "tradeid":1,       // 成交记录ID
      "price":73444,     // 成交价格
      "amount":0.027227, // 成交数量
      "time":1520329471  // 成交时间
    }  
  ]
}
```

##### CMD: 15, 查询指定交易对或者指定市场的行情数据
```
请求结构: 
{
  "cmd":{
    "type":15 // 查询指定交易对或者指定市场的行情数据，类型：整数
  },
  "pair":{
    "market":"cnc", // 准备查询哪个市场的行情数据，类型：字符串
    "coin":"eth"    // 准备查询哪个币的行情数据，**如果coin是空字符串，则查询market指定的市场下所有正常交易的币的行情数据**，类型：字符串
  }
}

应答结构:
{
  "cmd":{
    "type":15, // 查询指定交易对或者指定市场的行情数据
    "eno":0    // 错误码
  },
  "tickers":[
    {
      "market":"cnc", // 市场名
      "coin":"eac",   // 币名
      "ticker":{ // 行情数据
        "high":0.00198,        // 24小时内的最高价
        "low":0.00168,         // 24小时内的最低价
        "last":0.00188,        // 最近1次成交价
        "vol":1820699513.4921, // 24小时成交量
        "buy":0.00184,         // 当前买1价
        "sell":0.00194         // 当前卖1价
      }
    }
  ]
}
```

# 订单和成交记录的关联
+ 在当前 Aex 的 RESTful 接口中，订单和成交记录之间是没有任何关联的，这导致很难对挂单和成交的情况进行分析，也增加了交易策略的实现难度。
+ RESTful api 中订单和成交记录缺乏关联的问题，在 websocket api 中通过挂单请求中的tag来解决，由同一挂单请求形成的订单和成交结果都会记录挂单请求中的tag，因此订单和成交记录可以通过tag关联起来。


# 交易策略的实现
由于 websocket api 中挂单请求里的tag完全是用户自定义的，只要 tag > 0 就行，用户可以自行对tag进行编码，实现某种自定义的交易策略。
