---
title: API Docs v1 | bitextbit

language_tabs:
  - python: Python
  - javascript: Node.js

toc_footers:
  - <a href='https://www.bitexbit.com/auth/login'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

# includes:
#   - errors

code_clipboard: true

search: true
---


# Introduction


Welcome to the bitexbit API docs!

Service [bitexbit.com](https://bitexbit.com) provides open API for trading operations and broadcasting of all trading events. 


# Web Sockets API

Currently Web Socket API can be used for getting the freshest info about lats trades, new and cancelled orders as well. Such as using socket connection reduce network overhead it's preferable way to get latest exchange rate and orderbook.

API built on latest version of [socket.io](https://www.npmjs.com/package/socket.io) library for Node.js. So it should be easy to get work with.

## Orderbook events

> There are two types of order book notification messages
> In case new order was created, you'll get information about order, what amount remained after operation and list of trades created after that order.   

```json
{
  "success": true,
  "date": 1485686507.0944622, 
  "oid": 15234,
  "type": "sell", 
  "price": 935.23, 
  "amount": 0.0, 
  "trades": [
    {
      "tid": 14876, 
      "type": "buy",
      "price": "936.00", 
      "amount": "0.1", 
      "o_id": 15201
     }
  ]
}
```

> When order was cancelled by owner 

```json
{
  "success": true,
  "date": 1485686413.5688234,
  "oid": 15235,
  "cancel_order_id": 4535,
  "price": 935.23, 
  "type": "sell", 
  "amount": 0.0123,
  "pair": "BTC_USD",
  "cancelled": true
}
```

All you need to start receiving live information about orderbook is connect to `wss://bitexbit.com` and subscribe to necessary channel.

There are chanel names have pattern `book_<pair_name>` where `<pair_name>` is any pair available in service. For example `book_BTC_USD` or `book_BTC_ETH`. 


# HTTP API (v1)


HTTP API endpoint available on [https://bitexbit.com/api/v1/](https://bitexbit.com/api/v1/).

Some methods requires authorization [read below](#authorization).

<aside class="success">
We have limit in 2 calls per second from single account to authorization required methods and 100 calls per secong from single IP address to public methods.
</aside>

All HTTP methods accept `JSON` formats of requests and responses if it not specified by headers.  

## Authorization

> To generate auth headers, use this code:

```python
import hmac
from time import time
from urllib.parse import urlencode

def get_auth_headers(self, data):
        msg = 'your key' + urlencode(sorted(data.items(), key=lambda val: val[0]))

        sign = hmac.new('your keys secret'.encode(), msg.encode(), digestmod='sha256').hexdigest()

        return {
            'X-KEY': 'your key',
            'X-SIGN': sign,
            'X-NONCE': str(int(time() * 1000)),
        }
```

```javascript
const hmacSha256 = require('crypto-js/hmac-sha256'); // sha256 hash. or use you favorite u like
const request = require('request'); // for http requests. or use you favorite u like

const BASE_URL = 'https://bitexbit.com/api/v1';
const API_KEY = '0000-0000...';
const SECRET = 'Z%2........';

//Serialize for singing only. Can be used in request body if u like urlencoded form format instead of json
function serializePayload(payload) {
  return Object
    .keys(payload) // get keys of payload object
    .sort() // sort keys
    .map((key) => key + "=" + encodeURIComponent(payload[key])) // each value should be url encoded. the most sensitive part for sign checking
    .join('&'); // to sting, separate with ampersand
}

// Generates auth headers
function getAuthHeaders(payload) {
  // get SHA256 of <API_KEY><sorted urlencoded payload string><SECRET>
  const sign = hmacSha256(API_KEY + serializePayload(payload), SECRET).toString();

  return {
    'X-KEY': API_KEY,
    'X-SIGN': sign,
    'X-NONCE': Date.now()
  };
}

function getWallets(callback) {
  payload = {};

  const options = {
    method: 'get',
    url: `${BASE_URL}/wallets/`,
    headers: getAuthHeaders(payload)
  };

  request(options, callback);
}

function createOrder(order, callback) {
  const options = {
    method: 'post',
    url: `${BASE_URL}/order/`,
    headers: getAuthHeaders(order),
    form: order, // API accepts urlencoded form or json. Use appropriate headers!
  };

  request(options, callback);
}

// test
getWallets((error, response, body) => {
  console.log('error', error);
  console.log('body', body);
});

const order = {
  type: 'buy',
  pair: 'BTC_USD',
  amount: '0.0001',
  price: '0.1'
};

createOrder(order, (error, response, body) => {
  // get json, etc
  console.log('error', error);
  console.log('body', body);
});
```

In order for access to private API methods, generate authorization keys [in profile settings](https://bitexbit.com/profile/api).

All request to these methods must contain the following headers:

* X-KEY - your key.
* X-SIGN - query's POST data, sorted by keys and signed by your key's "secret" according to the HMAC-SHA256 method.
* X-NONCE - integer value, must be greater then nonce in previous api call.


# HTTP API (v2)

Read more about API v2 [here](https://bitexbit.github.io/api-docs-v2)


# HTTP API Gateway

Read more about API Gateway [here](https://bitexbit.github.io/api-docs-gateway)
# Success responses

Code | Meaning
-----|--------
200  | Returns is response is success. Body of response usually contains object that affected.

# Errors

We are using standard HTTP statuses in responses.

| Error Code | Meaning                                                                                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400        | If response has http status 400 that means that client should change some input parameters to make it success. What exactly wrong returned in response body.         |
| 401        | In case when HTTP endpoint require authentication but provided authentication headers not valid - response with status 401 will be returned and the details in body. |
| 403        | You do not have access to do the operation. For example when you try to cancel orders that was created by other user.                                                |
| 429        | We have different rate limits to call API methods. If you reached this - response with status 429 will be returned.                                                  |
| 500        | Nobody is safe to make mistakes. This is the case when error happened on serer side. No details but we are see it.                                                   |
| 502        | Looks like we under high load. Try your request later.                                                                                                               |

# Currencies


## List all currencies


```python
import requests

response = requests.get('https://bitexbit.com/api/v1/currencies/')
```

```javascript
var request = require('request');

request.get('https://bitexbit.com/api/v1/currencies/', function (error, response, body) {
    // process response    
});
```

> Sample output

```json
[
  {
    "sign": "Ƀ", 
    "short_name": "BTC"
  }, 
  {
    "sign": "Ξ", 
    "short_name": "ETH"
  } 
]
```

Returns information about all available currencies.

### HTTP Request

`GET https://bitexbit.com/api/v1/pairs/`


# Pairs


## List all pairs


```python
import requests

response = requests.get('https://bitexbit.com/api/v1/pairs/')
```

```javascript
var request = require('request');

request.get('https://bitexbit.com/api/v1/pairs/', function (error, response, body) {
    // process response    
});
```

> Sample output

```json
[
  {
    "currency2": "USD", 
    "maximum_order_size": 100000000.00000000, 
    "minimum_order_size": 0.00000001,
    "currency1": "BTC", 
    "name": "BTC_USD", 
    "price_precision": 3
   }
]
```

Returns information about all available pairs.

### HTTP Request

`GET https://bitexbit.com/api/v1/pairs/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency1 | all | Filter by first currency.
currency2 | all | Filter by second currency.


# Wallets


## List own wallets

```python
import requests

response = requests.get('https://bitexbit.com/api/v1/wallets/', headers = get_auth_headers({}))
```


```javascript
var request = require('request');

request.get(
    'https://bitexbit.com/api/v1/wallets/', 
    {
        headers: getAuthHeaders({})
    }, 
    function (error, response, body) {
        // process response    
    }
);
```

> Sample output

```json
[
  {
    "currency": "BTC", 
    "balance": 0.00000000, 
    "reserve": 0.00000000
   }
]
```

Returns information about all wallets of account.

### HTTP Request

`GET https://bitexbit.com/api/v1/wallets/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency_id | all | Filter by currency.

<aside class="warning">
This method requires authorization.
</aside>


# Orders

## Order statuses

Value | Status name | Description
--------- |  ----------- | -----------
1 | Active | Order in queue for executing
2 | Canceled | Order not active, permanently
3 | Done | Order fully executed


## Get orderbook

```python
import requests

params = { 
    limit_bids: 1,
    limit_asks: 1,
}

response = requests.get('https://bitexbit.com/api/v1/orderbook/BTC_USD/', params=params)
```

```javascript
var request = require('request');

var url = 'https://bitexbit.com/api/v1/orderbook/BTC_USD/';

var params = {
    limit_bids: 1,
    limit_asks: 1
};

request.get({url: url, qs: params}, function (error, response, body) {
        // process response    
    }
);
```

> Sample output when grouping disabled

```json
{
  "sell": [
    {
      "price": 911.519, 
      "id": 44667, 
      "amount": 0.000446, 
      "timestamp": 1485777324.410015
     }
   ],
  "buy": [
    {
      "price": 911.122, 
      "id": 44647, 
      "amount": 0.001233, 
      "timestamp": 1485777124.415542
    }
  ]
}
```

> Sample output when grouping enabled

```json
{
  "sell": [
    {
      "price": 911.519, 
      "amount": 0.000446 
     }
   ],
  "buy": [
    {
      "price": 911.122, 
      "amount": 0.001233 
    }
  ]
}
```

Get full orderbook of **active** orders

### HTTP Request

`GET https://bitexbit.com/api/v1/orderbook/<pair_name>/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit_sell | all | Sell orders limit
limit_buy | all | Buy orders limit
group | 0 | If set 1, then order will grouped by price


## List own orders

```python
import requests

response = requests.get('https://bitexbit.com/api/v1/orders/own/', headers = get_auth_headers({}))
```

```javascript
var request = require('request');

request.get('https://bitexbit.com/api/v1/orders/own/', {headers: getAuthHeaders({})}, function (error, response, body) {
    // process response    
});
```

> Sample output

```json
[
  {
    "amount": 0.500000000, 
    "pair": "ETH_USD", 
    "type": "buy", 
    "status": 3, 
    "price": 0.00113000,
    "id": 11249
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

List all orders created in your account. Can be filtered by query parameters.

### HTTP Request

`GET https://bitexbit.com/api/v1/orders/own/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
type | all | Filter by orders type (sell or buy).
pair | none | Filter by pair.
status | all | Filter by status.
limit | 2000 | Limiting results.

## Retrieve single order


```python
import requests

oid = 53189

response = requests.get('https://bitexbit.com/api/v1/order/{}/'.format(oid))
```

```javascript
var request = require('request');

var oid = 53189;

request.get('https://bitexbit.com/api/v1/order/' + oid + '/', function (error, response, body) {
    // process response    
});
```

> Sample output

```json
{
  "amount": 0.1250000,
  "pair": "BTC_USD",
  "type": "buy",
  "status": "1",
  "price": 870.69000000, 
  "id": 11259
}
```

Get detailed information about order by its id

### HTTP Request

`GET https://bitexbit.com/api/v1/order/<pk>/`


## Create order


```python
import requests

order = {
    type: 'buy',
    pair: 'BTC_USD',
    amount: '1.0',
    price: '870.69'
}

response = requests.post('https://bitexbit.com/api/v1/order/', data = order, headers = get_auth_headers(order))
```

```javascript
var request = require('request');

var order = {
    type: 'buy',
    pair: 'BTC_USD',
    amount: '1.0',
    price: '870.69'    
};

request.post({
                url: 'https://bitexbit.com/api/v1/order/',
                form: order,
                headers: getAuthHeaders(order)
            }, function (error, response, body) {
                // process response
                // save order id, check if it's executed
            }
);
```

> Sample output

```json
{
  "success": true,
  "type": "buy", 
  "date": 1483721079.51632, 
  "oid": 11268, 
  "price": 870.69000000, 
  "amount": 0.00000000,
  "trades": [
    {
      "type": "sell", 
      "price": 870.69000000, 
      "o_id": 11266, 
      "amount": 0.00010000, 
      "tid": 6049
    }
  ]
}
```

<aside class="warning">
This method requires authorization.
</aside>

Create new order that will be automatically executed.

### HTTP Request

`POST https://bitexbit.com/api/v1/order/`

### POST params

Parameter | Description
--------- | -----------
type | Type of order (`sell` or `buy`).
pair | Pair of order
amount | Amount of first currency of pair.
price | Price of order. This param have limited precision (See [pairs](#pairs)).


## Cancel order


```python
import requests

data = {
    order: 63568
}

response = requests.post('https://bitexbit.com/api/v1/order-cancel/', data = data, headers = get_auth_headers(data))
```

```javascript
var request = require('request');

var data = {
    order: 63568
};

request.post({
        url: 'https://bitexbit.com/api/v1/order-cancel/',
        form: data,
        headers: getAuthHeaders(data)
    }, function (error, response, body) {
        // process response    
    }
);
```

> Sample output

```json
{
  "order": 63568
}
```

<aside class="warning">
This method requires authorization.
</aside>

Cancel your **active** order. If order not exists, it's done or cancelled error message will be returned. 

### HTTP Request

`POST https://bitexbit.com/api/v1/order-cancel/`

### POST params

Parameter | Description
--------- | -----------
order | ID of order to cancel


# Exchanges


## List all exchanges


```python
import requests

response = requests.get('https://bitexbit.com/api/v1/exchanges/')
```

```javascript
var request = require('request');

request.get('https://bitexbit.com/api/v1/exchanges/', function (error, response, body) {
    // process response    
});
```

> Sample output

```json
[
  {
    "id": 6030, 
    "price": 839.36000000, 
    "pair": "BTC_USD", 
    "type": "sell", 
    "timestamp": 1483705817.735508,
    "amount": 0.00281167
  }
]
```


### HTTP Request

`GET https://bitexbit.com/api/v1/exchanges/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
pair | all | Filter by pair.
limit | 100 | Limiting results.
offset | 0 | Skip number of records.
ordering | -id | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting.


## List own exchanges

<aside class="warning">
This method requires authorization.
</aside>

List only own exchanges where your account was buyer or seller.

### HTTP Request

`GET https://bitexbit.com/api/v1/exchanges/own/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
type | all | Filter by orders type (sell or buy).
pair | all | Filter by pair.
limit | 100 | Limiting results.
offset | 0 | Skip number of records.
ordering | -id | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting.


# Deposits


## List own deposits

```python
import requests

response = requests.get('https://bitexbit.com/api/v1/deposits/', headers = get_auth_headers({}))
```

```javascript
var request = require('request');

request.get({
                url: 'https://bitexbit.com/api/v1/deposits/',
                headers: getAuthHeaders({})
            }, function (error, response, body) {
                // process response    
            }
);
```

```json
[
  {
    "timestamp": 1485363039.18359, 
    "id": 317, 
    "currency": "BTC", 
    "amount": 530.00000000
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

List your deposits

### HTTP Request

`GET https://bitexbit.com/api/v1/deposits/`


# Withdraws

## Withdraw statuses

Value | Status name | Description
--------- | ----------- | -----------
10 | New | Withdraw created, verification need
20 | Verified | Withdraw verified, waiting for approving 
30 | Approved | Approved by moderator
40 | Refused | Refused by moderator. See your email for more details
50 | Canceled | Cancelled by user

## List own made withdraws

```python
import requests

response = requests.get('https://bitexbit.com/api/v1/withdraws/', headers: get_auth_headers({}))
```

```javascript
var request = require('request');

request.get({
        url: 'https://bitexbit.com/api/v1/withdraws/',
        headers: getAuthHeaders({})
    }, function (error, response, body) {
        // process response    
    }
);
```

> Sample output

```json
[
  {
    "id": 403, 
    "timestamp": 1485363466.868539, 
    "currency": "BTC", 
    "amount": 0.53000000, 
    "status": 20
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

Get list of your withdraws

### HTTP Request

`GET https://bitexbit.com/api/v1/withdraws/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency_id | all | Filter currency.
status | all | Filter by status.

# Charts

```python
import requests

params = {
    'limit': 1
    }

response = requests.get('https://bitexbit.com/api/charts/BTC_USD/60/chart', params = params)
```

```javascript
var request = require('request');

var params = {limit: 1};

request.get({
                url: 'https://bitexbit.com/api/charts/BTC_USD/60/chart',
                qs: params
            }, function (error, response, body) {
                // process response    
            }
);
```

> Sample output

```json
[
  {
    "volume": 0.262929, 
    "high": 912.236, 
    "low": 910.086, 
    "close": 911.915, 
    "time": 1485777600, 
    "open": 910.424
   }
]
```

### Chart types

Type value | Type name
--------- | -------
1 | 1 minute
5 | 5 minutes
15 | 15 minutes
30 | 30 minutes
60 | 1 hour
240 | 4 hours
D | 1 day

### HTTP Request

`GET https://bitexbit.com/api/charts/<pair>/<type>/chart`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit | 720 | Limiting results
since | None | Since Timestamp
until | None | Until Timestamp
