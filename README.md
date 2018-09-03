# UDEX HTTP REST API Specification v0

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
| 404  | Not found                               |
| 429  | Too many requests - Rate limit exceeded |
| 500  | Internal Server Error                   |
| 501  | Not Implemented                         |


### Misc.

- All requests and responses should be of **application/json** content type
- All token amounts are sent in amounts of the smallest level of precision (base units). (e.g if a token has 18 decimal places, selling 1 token would show up as selling `'1000000000000000000'` units by this API).
- All addresses are sent as lower-case (non-checksummed) Ethereum addresses with the `0x` prefix.

## REST API

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
		"baseTokenVolume": "0",
		"decimals": 18,
		"miniAmount": 159.3266666666667,
		"fee": 0.001
	}
],
....
```

- `address` - address of the token
- `minAmount` - the minimum trade amount the relayer will accept
- `maxAmount` - the maximum trade amount the relayer will accept
- `precision` - the desired price precision a relayer would like to support within their orderbook

### GET /v0/orders

Retrieves a list of orders given query parameters. This endpoint should be paginated. For querying an entire orderbook snapshot, the [orderbook endpoint](#get-v0orderbook) is recommended.

#### Parameters

* exchangeContractAddress [string]: returns orders created for this exchange address
* tokenAddress [string]: returns orders where makerTokenAddress or takerTokenAddress is token address
* makerTokenAddress [string]: returns orders with specified makerTokenAddress
* takerTokenAddress [string]: returns orders with specified makerTokenAddress
* maker [string]: returns orders where maker is maker address
* taker [string]: returns orders where taker is taker address
* trader [string]: returns orders where maker or taker is trader address
* feeRecipient [string]: returns orders where feeRecipient is feeRecipient address

All parameters are optional.

If both makerTokenAddress and takerTokenAddress are specified, returned orders will be sorted by price determined by (takerTokenAmount/makerTokenAmount) in ascending order. By default, orders returned by this endpoint are unsorted.

#### Response

[See response schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/signed_orders_schema.ts#L1)

```
[
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
    },
    ...
]
```

### GET /v0/order/[orderHash]

Retrieves a specific order by orderHash.

#### Response

[See response schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/order_schemas.ts#L24)


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

Returns HTTP 404 if no order with specified orderHash was found.

### GET /v0/orderbook

Retrieves the orderbook for a given token pair.

#### Parameters

* baseTokenAddress [string]: address of token designated as the baseToken in the [currency pair calculation](https://en.wikipedia.org/wiki/Currency_pair) of price (required)
* quoteTokenAddress [string]: address of token designated as the quoteToken in the currency pair calculation of price (required)

#### Response

[See response schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/relayer_api_orderbook_response_schema.ts#L1)

```
{
    "bids": [
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
        },
        ...
    ],
    "asks": [
        {
            "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
            "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
            "taker": "0xa2b31dacf30a9c50ca473337c01d8a201ae33e32",
            "makerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
            "takerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
            "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
            "makerTokenAmount": "22000000000000000",
            "takerTokenAmount": "10000000000000000",
            "makerFee": "100000000000000",
            "takerFee": "200000000000000",
            "expirationUnixTimestampSec": "632",
            "salt": "54515451557974875123697849345751275676157243756715784155226239582178",
            "ecSignature": {
                "v": 27,
                "r": "0x61a3ed31b43c8780e905a260a35faefcc527be7516aa11c0256729b5b351bc33",
                "s": "0x40349190569279751135161d22529dc25add4f6069af05be04cacbda2ace2254"
            }
        },
        ...
    ]
}
```

- `bids` - array of signed orders where `takerTokenAddress` is equal to `baseTokenAddress` 
- `asks` - array of signed orders where `makerTokenAddress` is equal to `baseTokenAddress`

Bids will be sorted in descending order by price, and asks will be sorted in ascending order by price. Within the price sorted orders, the orders are further sorted first by total fees, then by expiration in ascending order.

### POST /v0/fees

Given an unsigned order without the fee-related properties, returns the required `feeRecipient`, `makerFee`, and `takerFee` of that order.

#### Payload

[See payload schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/relayer_api_fees_payload_schema.ts#L1)

```
{
    "exchangeContractAddress": "0x12459c951127e0c374ff9105dda097662a027093",
    "maker": "0x9e56625509c2f60af937f23b7b532600390e8c8b",
    "taker": "0x0000000000000000000000000000000000000000",
    "makerTokenAddress": "0x323b5d4c32345ced77393b3530b1eed0f346429d",
    "takerTokenAddress": "0xef7fff64389b814a946f3e92105513705ca6b990",
    "makerTokenAmount": "10000000000000000",
    "takerTokenAmount": "20000000000000000",
    "expirationUnixTimestampSec": "42",
    "salt": "67006738228878699843088602623665307406148487219438534730168799356281242528500"
}
```

#### Response

[See response schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/relayer_api_fees_response_schema.ts#L1)

```
{
    "feeRecipient": "0xb046140686d052fff581f63f8136cce132e857da",
    "makerFee": "100000000000000",
    "takerFee": "200000000000000"
}
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

[See error response schema](https://github.com/0xProject/0x.js/blob/v1-protocol/packages/json-schemas/schemas/relayer_api_error_response_schema.ts#L1)

```
{
    "code": 101,
    "reason": "Validation failed",
    "validationErrors": [
        {
            "field": "maker",
            "code": 1002,
            "reason": "Invalid address"
        }
    ]
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
