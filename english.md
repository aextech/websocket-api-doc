AEX Websocket API Protocol Documentation  
[Chinese Document](/README.md)

# API Request URL
```
URL: wss://api.aex.zone/ws/v1

Currently the Websocket API is only available for the api.aex.zone domain name.
```

# Websocket connection process
```
With the same account, only one connection can survive at the same time. The connection with successful authentication will disconnect other connections. In the case of unauthentication, the same IP allows more connections.
such as:
  1) IP1 connection on websocket
  2) IP1 initiates an auth request and succeeds
  3) IP2 connection on websocket
  4) IP2 initiates an auth request and succeeds
  5) The successful connection of auth on IP1 will be disconnected actively by the server.
```

# API Request Restrictions
+ Up to 10 Websocket connections on the same IP
+ 1 Websocket connection up to 10 requests per second, recounting requests in the next second
+ 1 Websocket connection up to 10 transaction pairs at the same time

# Table of Contents
+ [Protocol Command Type](#protocol-command-type)
+ [Error Code](#error-code)
+ [Protocol request/response structure (json)](#protocol-request-response-data-structure)
   + [CMD: 1, Order book notification](#cmd-1-order-book-notification-the-server-actively-notifies-the-client)
   + [CMD: 2, Watch/unwatch specified market pairs](#cmd-2-watchunwath-market-pairs-maximum-of-10)
   + [CMD: 4, Authentication](#cmd-4-authentication)
   + [CMD: 5, Get Balance](#cmd-5-get-balance)
   + [CMD: 6, Pending order](#cmd-6-pending-order)
   + [CMD: 7, Withdraw](#cmd-7-withdraw)
   + [CMD: 8, Query order according to the order ID](#cmd-8-query-order-according-to-the-order-id)
   + [CMD: 9, Query trade according to the trade ID](#cmd-9-query-trade-according-to-the-trade-id)
   + [CMD: 10, Query market pair meta data](#cmd-10-query-market-pair-meta-data)
   + [CMD: 11, Query orders according to the order tag](#cmd-11-query-orders-according-to-the-order-tag)
   + [CMD: 12, Query trades according to the order tag](#cmd-12-query-trades-according-to-the-order-tag)
   + [CMD: 13, Query valid market pair list](#cmd-13-query-valid-market-pair-list)
   + [CMD: 14, Query public trades of the specified market pair](#cmd-14-query-public-trades-of-the-specified-market-pair)
   + [CMD: 15, Query ticker of the specified market pair](#cmd-15-query-ticker-of-the-specified-market-pair)
+ [Link orders and trades through tags](#link-orders-and-trades-through-tags)
+ [Implementation of trading strategy](#implementation-of-trading-strategy)


# Protocol Command Type
CMD | Description
----- | ---------
1 | Depth change notification, the server proactively notifies the client, the server only informs the transaction pair that has been paid attention by the 2 command
2 | Add the deal pair you want to follow and delete the deal pair you're already interested in
3 | K-line data (not yet implemented)
4 | Signature Certification
5 | Get all currency balances
6 | Place an order
7 | Cancel order
8 | Query individual order information based on order ID
9 | Search for a single transaction record based on the transaction ID
10 | Root Query Transaction Information
11 | Query multiple order information based on tag
12 | Query multiple transaction records based on tag
13 | Query all valid transaction pairs
14 | Query the transaction record of the specified transaction pair
15 | Query market data for a specified trade pair or specified market


# Error Code
Error code | Description
----- | ---------
0 | normal
Xx+001 | ================ System Error Code ================
1001 | Add a deal pair you want to follow, delete the deal pair you're already interested in
4001 | Invalid request
5001 | Not certified
6001 | Verified
7001 | System busy
8001 | The currency does not exist
9001 | Trading currency does not exist
10001 | Invalid trading area
11001 | The currency is not traded in the current trading zone
12001 | Request frequency limit
Xx+002 | ================ Attention error code ================
1002 | The number of concerns exceeds the limit
Xx+006 | ================ Pending order error code ================
1006 | User ID is invalid
2006 | Invalid transaction type
3006 | Price is invalid
4006 | Invalid
5006 | Quantity points exceed the limit
6006 | Quantity exceeds maximum limit
7006 | Quantity exceeds minimum limit
8006 | Price exceeds maximum limit
9006 | Price point exceeds limit
10006 | Transaction amount is less than the minimum limit
11006 | Insufficient balance
12006 | Quantity accuracy configuration error
13006 | Price accuracy not configured, or configuration error
14006 | Input parameter error
Xx+007 | ================ Cancel order error code ================
1007 | User ID is invalid
2007 | Order does not exist
3007 | Order deletion failed, either the order was merged or has been deleted
4007 | Invalid order ID
Xx+008 | ================ Query order error code ================
1008 | Order does not exist
Xx+009 | ================ Query the transaction record error code ================
1009 | Transaction record does not exist
10080 | transaction uses V1 version
10081 | transaction uses V2 version
1008000 | transaction uses V1 version
1008001 | transaction uses V2 version

# Protocol request response data structure
##### CMD: 1, Order book notification, the server actively notifies the client
```
Request structure:
no

Response structure:
{
  "cmd": {
    "type": 1,     // command type 1 means server sent whole order book to client when order book changed
    "eno": 0       // error code always returns 0
  },
  "market": "cnc", // market currency is cnc
  "coin": "btc",   // trading currency is btc
  "bids": [        // buy order book
    {"price": 32076, "amount": 742.799405},
    {"price": 32000, "amount": 79253.604638},
    ...
  ],
  "asks": [        // sell order book
    {"price": 32077, "amount": 1},
    {"price": 32153, "amount": 2},
    ...
  ]
}
```

##### CMD: 2, Watch/Unwath market pairs (maximum of 10)
```
Request structure:
{
  "cmd": {
    "type": 2         // watch/unwatch the specified market pair
  },
  "type": 1,          // 1=watch the market pair, 2=unwatch the market pair
  "pairs": [{         // market pairs list to follow/unfollow
    "market": "cnc",  // market currency
    "coin": "btc"     // trading currency
  }, {
    "market": "cnc",  // market currency
    "coin": "eos"     // trading currency
  }]
}

Response structure:
{
  "cmd": {
    "type": 2,        // follow/unfollow the market pair
    "eno": 0          // error code, after watch successfully, order book change will be sent to client
  },
  "type": 1,          // 1=watch, 2=unwatch
  "pairs": [{
    "market": "cnc",  // market currency
    "coin": "btc"     // trading currency
  }, {
    "market": "cnc",  // market currency
    "coin": "eos"     // trading currency
  }]
}
```

##### CMD: 4, Authentication
```
Request structure:
{
  "cmd": {
    "type": 4 // authentication
  },
  "key": "xxxxxxxxx",    // public key, obtained from "https://www.aex.plus/page/api_detailed.html"
  "time": 1545727168,    // timestamp, in seconds
  "md5": "xxxxxxxxxxxxx" // md5: md5("{key}_{userID}_{skey}_{time}")
}

Response structure:
{
  "cmd": {
    "type": 4, // authentication
    "eno": 0   // error code
  }
}
```

##### CMD: 5, Get Balance
```
Request structure:
{
  "cmd": {
    "type": 5 // get balance
  }
}

Response structure:
{
  "cmd": {
    "type": 5, // get balance
    "eno": 0   // error code
  },
  "balances": [{ 
    "coin": "bkbt",  // currency name
    "val": 116.99,   // available balance
    "locked": 0      // freeze funds
  }, {
    "coin": "blk",   // currency name
    "val": 0.180533, // available balance
    "locked": 0      // freeze funds
  }, {
    "coin": "btc",   // currency name
    "val": 0,        // balance
    "locked": 0      // freeze funds
  }]
}
```

##### CMD: 6, Pending order
```
Request structure:
{
  "cmd": {
    "type": 6       // pending order
  },
  "market": "gat",  // market currency
  "coin": "bkbt",   // trading currency
  "tag": 2,         // user-defined tag, a non-negative integer used to associate the order with the trade
  "type": 1,        // pending order type: 1=bid, 2=ask
  "price": 12.5,    // price
  "amount": 20      // quantity
}

Response structure:
{
  "cmd": {
    "type": 9, // the transaction record, the transaction record will be returned if the pending order is requested, there may be multiple responses of type 9
    "eno": 0 // error code
  },
  "data": {
    "market": "gat", // trading area, pricing currency
    "coin": "bkbt", // trading currency
    "type": 1, // 1=buy the deal, 2=sell the deal
    "tag": 2, // user-defined tag, a non-negative integer used to associate the order with the transaction record. The tag here is the tag in the request.
    "tradeid": 18, // transaction record ID
    "price": 10.5, // transaction price
    "amount": 57, // the number of deals
    "fee": 0 // handling fee, type=1 is the handling fee, in transaction currency, income = amount-fee; type=2 is the selling fee, in pricing currency, income = price* Amount-fee
    "self":0 //1 is self purchase and self sale 0 no
  }
}
{
  "cmd": {
    "type": 6, // pending order request is not fully filled, the remaining part generates order
    "eno": 0 // error code
  },
  "data": {
    "market": "gat", // trading area, pricing currency
    "coin": "bkbt", // trading currency
    "type": 1, // 1 = pay, 2 = sell order
    "tag": 2, // user-defined tag, an integer used to associate the order with the transaction record. The tag here is the tag in the request.
    "orderid": 17, // order ID
    "price": 12.5, // order price
    "amount": 3 // order quantity
  }
}
```

##### CMD: 7, Withdraw
```
Request structure:
{
  "cmd": {
    "type": 7 // Withdraw
  },
  "market": "gat", // trading area, pricing currency
  "coin": "bkbt", // trading currency
  "orderid": 17 // order ID
}

Response structure:
{
  "cmd": {
    "type": 7, // withdraw the order
    "eno": 0 // error code
  },
  "market": "gat", // trading area, pricing currency
  "coin": "bkbt", // trading currency
  "orderid": 17 // Order ID in the request
}
```

##### CMD: 8, Query order according to the order ID
```
Request structure:
{
  "cmd": {
    "type": 8 // Query order information
  },
  "market": "gat", // trading area, pricing currency
  "coin": "bkbt", // trading currency
  "orderid": 17 // The order ID to prepare for the query
}

Response structure:
{
  "cmd": {
    "type": 8, // Query order information
    "eno": 0 // error code
  },
  "data": {
    "market": "gat", // trading area, pricing currency
    "coin": "bkbt", // trading currency
    "type": 1, // 1=pay, 2=sell
    "tag": 3, // user-defined tag, a non-negative integer used to associate the order with the transaction record. The tag here is the tag in the pending order request.
    "orderid": 17, // order ID
    "price": 12.5, // order price
    "amount": 3 // The remaining quantity of the order
    "time": 1547627993 // pending time, time stamp, unit seconds
  }
}
```

##### CMD: 9, Query trade according to the trade ID
```
Request structure:
{
  "cmd": {
    "type": 9 // Query the transaction record
  },
  "market": "gat", // trading area, pricing currency
  "coin": "bkbt", // trading currency
  "tradeid": 18 // The transaction record ID to be queried
}

Response structure:
{
  "cmd": {
    "type": 9, // returns the transaction record according to the transaction ID. In the case of self-selling, one request will return two responses.
    "eno": 0 // error code
  },
  "data": {
    "market": "gat", // trading area, pricing currency
    "coin": "bkbt", // trading currency
    "type": 1, // 1 = buy order record, 2 = sell order record
    "tag": 3, // user-defined tag, a non-negative integer used to associate the order with the transaction record. The tag here is the tag in the pending order request.
    "tradeid": 18, // transaction record ID
    "price": 10.5, // transaction price
    "amount": 57, // the number of deals
    "fee": 0 // handling fee, type=1 is the handling fee, in transaction currency, income = amount-fee; type=2 is the selling fee, in pricing currency, income = price* Amount-fee
    "time": 1547627993 // transaction time, time stamp, unit seconds
    "self":0 //1 is self purchase and self sale 0 no
  }
}
```
Note: When the user's pending order is filled, if the user does not actively request to query the transaction record, the server will use the response structure of the cmd:9 command to actively notify the user of the transaction record.

##### CMD: 10, Query market pair meta data
```
Request structure:
{
  "cmd": {
    "type": 10 // Query transaction pair information
  },
  "pairs": [{ // Prepare the query for the pair of transactions
    "market": "cnc", // trading area, pricing currency
    "coin": "btc" // trading currency
  }, {
    "market": "usdt", // trading area, pricing currency
    "coin": "btc" // trading currency
  }]
}

Response structure:
{
  "cmd": {
    "type": 10, // Query transaction pair information
    "eno": 0 // error code
  },
  "pairs": [{ // A list of transaction pairs that successfully queryed the data
    "market": "cnc", // trading area, pricing currency
    "coin": "btc", // trading currency
    "precisions": [{ // precision array
      "name": "Price", // pending order price accuracy
      "num": 0 // number of pending precision digits
    }, {
      "name": "Amt", // the quantity accuracy of the pending order
      "num": 6 // number of pending precision digits
    }],
    "valuelimits": [{ // pending order value limit array
      "name": "AmtMax", // the maximum number of pending orders
      "value": "10000000" // The maximum number of pending orders
    }, {
      "name": "AmtMin", // minimum pending order quantity
      "value": "0.001" // minimum pending order quantity
    }, {
      "name": "PriceMax", // maximum pending order price
      "value": "1000000" // maximum pending order price
    }, {
      "name": "MoneyMin", // minimum pending order amount
      "value": "1" // minimum pending order amount
    }]
  }]
}
```

##### CMD: 11, Query orders according to the order tag
```
Request structure:
{
  "cmd": {
    "type": 11 // Query the order according to the tag
  },
  "market": "cnc", // trading area, pricing currency
  "coin": "bitcny", // trading currency
  "tag": 3, // the tag to prepare for the query
  "since_order_id": 1 // starting order ID
}

Response structure:
{
  "cmd": {
    "type": 11, // query the order according to the tag
    "eno": 0 // error code
  },
  "market": "cnc", // trading area, pricing currency
  "coin": "bitcny", // trading currency
  "tag": 3, // the tag to prepare for the query
  "orders": [{ // Order result array, returning up to 10 orders per order
    "type": 1, // 1=pay, 2=sell
    "orderid": 53, // order ID
    "price": 0.997, // order price
    "amount": 20, // the current remaining quantity of the order
    "time": 1547629377 // pending order time, time stamp, unit seconds
  }]
}
```

##### CMD: 12, Query trades according to the order tag
```
Request structure:
{
  "cmd": {
    "type": 12 // Query the transaction record based on the tag
  },
  "market": "cnc", // trading area, pricing currency
  "coin": "bitcny", // trading currency
  "tag": 3, // the tag to prepare for the query
  "since_trade_id": 1 // Start transaction record ID
}

Response structure:
{
  "cmd": {
    "type": 12, // query the transaction record based on the tag
    "eno": 0 // error code
  },
  "market": "cnc", // trading area, pricing currency
  "coin": "bitcny", // trading currency
  "tag": 3, // the tag to prepare for the query
  "trades": [{ // An array of transaction records, up to 10 entries per time
    "type": 1, // 1=pay, 2=sell
    "tradeid": 1, // transaction record ID
    "price": 0.993, // transaction price
    "amount": 10, // the number of deals
    "fee": 0, // // handling fee, type=1 is the handling fee, in transaction currency, income = amount-fee; type=2 is the selling fee, in terms of pricing currency, income =price*amount-fee
    "time": 1547627993 // transaction time, time stamp, unit seconds
    "self":0 //1 is self purchase and self sale 0 no
  }]
}
```

##### CMD: 13, Query valid market pair list
```
Request structure:
{
  "cmd": {
    "type": 13 // Query the list of valid transaction pairs
  }
}

Response structure:
{
  "cmd":{
    "type": 13, // Query the list of valid transaction pairs
    "eno": 0 // error code
  },
  "pairs":[ // transaction pair array
    {
      "market":"cnc", // market
      "coin":"xlm" // coin name
    },
    {
      "market":"cnc", // market
      "coin": "qash" // currency name
    }
  ]
}
```

##### CMD: 14, Query public trades of the specified market pair
```
Request structure:
{
  "cmd":{
    "type":14 // Query the transaction record of the specified transaction pair
  },
  "pair":{
    "market":"cnc", // market
    "coin":"btc" // currency name
  },
  "since_trade_id":1 // Start with the transaction record ID and return up to 10 transaction records at a time, including the since_trade_id itself
}

Response structure:
{
  "cmd": {
    "type": 14, // Query the transaction record of the specified transaction pair
    "eno": 0 // error code
  },
  "pair": {
    "market": "cnc", // market
    "coin": "btc" // currency name
  },
  "trades": [ // transaction record array
    {
      "type": 1, // Buying and selling direction: 1=buy, 2=sell
      "tradeid": 1, // transaction record ID
      "price":73444, // transaction price
      "amount":0.027227, // quantity
      "time":1520329471 // time of sale
    }
  ]
}
```

##### CMD: 15, Query ticker of the specified market pair
```
Request structure:
{
  "cmd":{
    "type": 15 // Query the market data of the specified transaction pair or the specified market
  },
  "pair":{
    "market":"cnc", // ready to query which market data
    "coin": "eth" // Ready to query which currency's market data, ** If coin is an empty string, then query the market data of all normal traded currencies in the market specified by market**
  }
}

Response structure:
{
  "cmd":{
    "type": 15,, // Query the market data of the specified transaction pair or the specified market
    "eno": 0 // error code
  },
  "tickers":[
    {
      "market":"cnc", // market name
      "coin":"eac", // coin name
      "ticker": { // market data
        "high": 0.00098, // the highest price within 24 hours
        "low": 0.00168, // lowest price within 24 hours
        "last":0.00188, // 1 transaction price
        "vol": 1820699513.4921, // 24-hour volume
        "buy":0.00184, // currently buy 1 price
        "sell":0.00194 // Currently selling 1 price
      }
    }
  ]
}
```

# Link orders and trades through tags
+ In the current Aex RESTful interface, there is no correlation between orders and transaction records, which makes it difficult to analyze the status of pending orders and transactions, and also increases the difficulty of implementing trading strategies.
+ The problem of lack of association between order and transaction record in RESTful api is solved by the tag in the pending order in the websocket api. The order and the result of the transaction formed by the same pending order will record the tag in the pending order, so the order and the transaction record can be Associated by tags.


# Implementation of trading strategy
Since the tag in the pending request in the websocket api is completely user-defined, as long as tag > 0, the user can encode the tag to implement a certain custom trading strategy.

