# HTTP REST API Specification v0

## Table of Contents

* [General](#general)
    * [Errors](#errors)
    * [Misc](#misc)
* [REST API](#rest-api)
    * [GET token_pairs](#get-v0token_pairs)
    * [GET orders](#get-v0orders)
    * [GET order](#get-v0orderorderhash)
    * [GET orderbook](#get-v0orderbook)
    * [POST fees](#post-v0fees)
    * [POST order](#post-v0order)

## General

### Errors

Unless the spec defines otherwise, errors to bad requests should respond with HTTP 4xx or status codes.

#### Common error codes

| Code | Reason
| ---- | --------------------------------------- |
| 400  | Bad Request – Invalid request format    |
| 401  | Unauthorized – Unauthorized request     |
| 404  | Not found                               |
| 429  | Too many requests - Rate limit exceeded |
| 500  | Internal Server Error                   |
| 501  | Not Implemented                         |


### Misc.

- All requests and responses should be of **application/json** content type
- All addresses are sent as lower-case (non-checksummed) Ethereum addresses with the `0x` prefix.

## Public Rest API

### GET /v0/markets

Retrieves a list of available token pairs and the information required to trade them. 

#### Response

```
[
	{
		"id": 2,
		"marketId": "ZRX-WETH",
		"name": "0xProject",
		"url": "https://0xproject.com/",
		"baseToken": "0xc778417e063141139fce010982780140aa0cd5ab",
		"tradeToken": "0xa8e9fa8f91e5ae138c74648c9c304f1c75003a8d",
		"price24h": 0.003,
		"price": 0.0012,
		"tradeTokenVolume": "0",
		"baseTokenVolume": "0.000044859458000000",
		"decimals": 18,
		"miniAmount": 159.3266666666667,
		"fee": 0.001
	}
],
....
```

- `marketId` - symbol of the token
- `baseToken` - the base token address of the market
- `tradeToken` - the trade token address of the market
- `price24h` - the price 24 hours ago of the market
- `price` - current price of the market
- `miniAmount` - mini amount of the market
- `fee` - fee of the market

### GET /v0/orderbook

Retrieves the orderbook for a given token pair.

#### Parameters

* pair [string]: address of token designated as the baseToken in the [currency pair calculation](https://en.wikipedia.org/wiki/Currency_pair) of price (required)


#### Response

```
{
	"bids": [{
		"id": 452,
		"maker": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"taker": "0x0000000000000000000000000000000000000000",
		"makerTokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
		"takerTokenAddress": "0xa8e9fa8f91e5ae138c74648c9c304f1c75003a8d",
		"feeRecipient": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"exchangeContractAddress": "0x479cc461fecd078f766ecc58533d6f69580cf3ac",
		"salt": "64714771787346576693970527227544016460454079355363503584618354068699042527224",
		"makerTokenAmount": "5729184000000",
		"takerTokenAmount": "13921902000000000",
		"makerFee": "0",
		"takerFee": "0",
		"expirationUnixTimestampSec": 1535963954,
		"takerTokenAmountFilled": "0",
		"takerTokenAmountCancelled": "0",
		"status": "activity",
		"makerTokenRemaining": "5729184000000",
		"takerTokenRemaining": "13921902000000000",
		"makerLockedAmount": "0",
		"takerLockedAmount": "0",
		"price": 0.000411112,
		"pair": "ZRX-WETH",
		"side": "buy",
		"orderHash": "e441cc9e0f567d2521d6dfa1d3f12faac65ab26cebce3c5449e4cce5fa33e4b3",
		"createdTime": "2018-09-02T16:39:17.285983+08:00",
		"updatedTime": "2018-09-02T16:39:17.285984+08:00"
	},
	...
	],
	"asks": [{
		"id": 436,
		"maker": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"taker": "0x0000000000000000000000000000000000000000",
		"makerTokenAddress": "0xa8e9fa8f91e5ae138c74648c9c304f1c75003a8d",
		"takerTokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
		"feeRecipient": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"exchangeContractAddress": "0x479cc461fecd078f766ecc58533d6f69580cf3ac",
		"salt": "60115282420328746583803936021604412873917235397941994748653467998912648672037",
		"makerTokenAmount": "13921902000000000",
		"takerTokenAmount": "18080374000000",
		"makerFee": "0",
		"takerFee": "0",
		"expirationUnixTimestampSec": 1535962538,
		"takerTokenAmountFilled": "0",
		"takerTokenAmountCancelled": "0",
		"status": "activity",
		"makerTokenRemaining": "13921902000000000",
		"takerTokenRemaining": "18080374000000",
		"makerLockedAmount": "0",
		"takerLockedAmount": "0",
		"price": 0.0013,
		"pair": "ZRX-WETH",
		"side": "sell",
		"orderHash": "40fedfcb41885d6f1b7ceb1c90d0d3adaa036a2b8dfb23aad0112b7c034220da",
		"createdTime": "2018-09-02T16:15:40.891439+08:00",
		"updatedTime": "2018-09-02T16:15:40.891441+08:00"
	},
	...
	]
}
```

- `bids` - array of signed orders where `takerTokenAddress` is equal to `baseTokenAddress` 
- `asks` - array of signed orders where `makerTokenAddress` is equal to `baseTokenAddress`

Bids will be sorted in descending order by price, and asks will be sorted in ascending order by price. Within the price sorted orders, the orders are further sorted first by total fees, then by expiration in ascending order.

## Private Rest API

###  POST /v0/auth

Before you can get your own open order list or cancel an order, you need to get authentication token.

#### Using web3.js
```
async sign(address, data) {
	const hash = this.web3.sha3(data);
	const sig = await new Promise(function(resolve, reject) {
		web3.eth.sign(address, hash, function(error, result) {
			if (error) {
				reject(error);
			} else {
				resolve(result);
			}
		})
	});
	return {
		sign: sig,
		data: hash
	};
}

async verify() {
	const data = "Sign this message to verify your authority of current address !";
	const sigData = await this.sign(this.currentAddress, data);
	const authForm = {
		address: this.currentAddress,
		data: sigData.data,
		signature: sigData.sign
	};
	return DoPost(authForm);
}
```
#### Payload
```
{
	"address": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
	"data": "0x1bf59099f1a36e1df4d8afc601951ecfd07a6cc8639bc016eb627b33eeec9198",
	"signature": "0x20a17d52961bfff8a2348a143274711ff01c239bf955de5568015d1d3aa500ca195061f9c969e1e4723d4432be295fae4bbc5ddb32707d3088219a53996b96131c"
}
```
- `address` - ethereum address for signing
- `data` - hash result of the message
- `signature` - signature result of data

###### Response
```
{
	"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MzY1NjQ4NzYsImlzcyI6IjB4MWJiZmQ0MDA5ZGRlNzM1OTViNmZmNGFiYWM5ZGQxNDk0NDM4YWFhZSIsIm5iZiI6MTUzNTk2MDA3Nn0.OScS5DJA1N-eAer2ymhnJQl9koS2QSZbgBCOQ_0h6Fw"
}
```

### GET /v0/orderHistory

Retrieves a list of orders given query parameters. This endpoint should be paginated. For querying an entire orderbook snapshot, the [orderbook endpoint](#get-v0orderbook) is recommended.

#### Headers
- `Content-Type` - application/json
- `Authorization` - authentication token 

#### Parameters

* pair [string]: returns orders created for this exchange address
* tokenAddress [string]: returns orders where makerTokenAddress or takerTokenAddress is token address
* makerTokenAddress [string]: returns orders with specified makerTokenAddress

Returns HTTP 400 if no pair was found.

#### Response

```
[
	{
		"id": 1,
		"maker": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"taker": "0x0000000000000000000000000000000000000000",
		"makerTokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
		"takerTokenAddress": "0xa8e9fa8f91e5ae138c74648c9c304f1c75003a8d",
		"feeRecipient": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"exchangeContractAddress": "0x479cc461fecd078f766ecc58533d6f69580cf3ac",
		"salt": "55830723742902371711551913246819251315650256325984569206718629062992028427256",
		"makerTokenAmount": "4180747000000",
		"takerTokenAmount": "3480475000000000",
		"makerFee": "0",
		"takerFee": "0",
		"expirationUnixTimestampSec": 1535963543,
		"takerTokenAmountFilled": "3480475000000000",
		"takerTokenAmountCancelled": "0",
		"status": "filled",
		"makerTokenRemaining": "0",
		"takerTokenRemaining": "0",
		"makerLockedAmount": "0",
		"takerLockedAmount": "0",
		"price": 0.0012,
		"pair": "ZRX-WETH",
		"side": "buy",
		"orderHash": "c30eb1c828671b7ba3a454a765c9eeabecd61b58f9e2280a24d4a5f0bf3c8eaa",
		"createdTime": "2018-09-02T16:32:28.871066+08:00",
		"updatedTime": "2018-09-02T16:32:31.883558+08:00"
	},
    ...
]
```

### GET /v0/openOrders

Retrieves a list of orders given query parameters. This endpoint should be paginated. For querying an entire orderbook snapshot, the [orderbook endpoint](#get-v0orderbook) is recommended.

#### Headers
- `Content-Type` - application/json
- `Authorization` - authentication token 

#### Parameters

* pair [string]: returns orders created for this exchange address
* tokenAddress [string]: returns orders where makerTokenAddress or takerTokenAddress is token address
* makerTokenAddress [string]: returns orders with specified makerTokenAddress

Returns HTTP 400 if no pair was found.

#### Response

```
[
	{
		"id": 452,
		"maker": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"taker": "0x0000000000000000000000000000000000000000",
		"makerTokenAddress": "0xc778417e063141139fce010982780140aa0cd5ab",
		"takerTokenAddress": "0xa8e9fa8f91e5ae138c74648c9c304f1c75003a8d",
		"feeRecipient": "0x1bbfd4009dde73595b6ff4abac9dd1494438aaae",
		"exchangeContractAddress": "0x479cc461fecd078f766ecc58533d6f69580cf3ac",
		"salt": "64714771787346576693970527227544016460454079355363503584618354068699042527224",
		"makerTokenAmount": "5729184000000",
		"takerTokenAmount": "13921902000000000",
		"makerFee": "0",
		"takerFee": "0",
		"expirationUnixTimestampSec": 1535963954,
		"takerTokenAmountFilled": "0",
		"takerTokenAmountCancelled": "0",
		"status": "activity",
		"makerTokenRemaining": "5729184000000",
		"takerTokenRemaining": "13921902000000000",
		"makerLockedAmount": "0",
		"takerLockedAmount": "0",
		"price": 0.000411112,
		"pair": "ZRX-WETH",
		"side": "buy",
		"orderHash": "e441cc9e0f567d2521d6dfa1d3f12faac65ab26cebce3c5449e4cce5fa33e4b3",
		"createdTime": "2018-09-02T16:39:17.285983+08:00",
		"updatedTime": "2018-09-02T16:39:17.285984+08:00",
		"txId": "",
		"txAmount": ""
	},
    ...
]
```

### POST /v0/order

Submit a signed order to the relayer.

#### Payload

[See payload schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/order_schemas.ts#L24)

```
{
    "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
    "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
    "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
    "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
    "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
    "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
    "makerTokenAmount": "10000000000000000",
    "takerTokenAmount": "20000000000000000",
    "makerFee": "100000000000000",
    "takerFee": "200000000000000",
    "expirationUnixTimestampSec": "42",
    "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500",
    "ecSignature": {
        "v": 27,
        "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
        "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
    }
}
```

#### Response

###### Success Response

Returns HTTP 201 upon success.

###### Error Response

Error response will be sent with a non-2xx HTTP status code

```
{
	"error": {
		"code": 106,
		"reason": "Signature Verify Failed."
	}
}
```

General error codes:

```
100 - Validation Failed
101 - Malformed JSON
102 - Order submission disabled
103 - Throttled
```

Validation error codes:

```
1000 - Required field
1001 - Incorrect format
1002 - Invalid address
1003 - Address not supported
1004 - Value out of range
1005 - Invalid ECDSA or Hash
1006 - Unsupported option
```
### DELETE /v0/orders/:orderHash

Delete an order by orderHash.

#### Headers
- `Content-Type` - application/json
- `Authorization` - authentication token 

#### Parameters

* orderHash [string]: The hash result of order you wish to cancel.

###### Response

Returns HTTP 200 upon success.


