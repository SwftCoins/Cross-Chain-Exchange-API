This document describes how to integrate SWFT Blockchain's cross-chain exchange API to a wallet or any crypto applications.
Please refer to the link in the description for details.

## 1.查询币种列表
提供币种列表展示给用户，告诉用户哪些币可以进行兑换 

 详细参数参见API接口文档 查询货币清单接口 api/v1/queryCoinList  
 https://docs.google.com/spreadsheets/d/1LgjKhNGQRL1_TJmzrZfLjqW-FGWcJDI6uuzjuB7aBRM/edit#gid=1924547912
 
 | 入参字段   | 说明   |
| --------   | -----:  |
| equipmentNo      | 环境编号 |

#### 返回结果

```
{
      "data": [
           {
            "coinAllCode": "SwftCoin",
            "coinCode": "SWFTC",
            "coinImageUrl": "",
            "coinName": "速币",
            "contact": "0x0bb217E40F8a5Cb79Adf04E1aAb60E5abd0dfC1e",
            "isSupportAdvanced": "Y",
            "mainNetwork": "ETH",
            "noSupportCoin": "BCC,SAN,ICX,EET,ETDM,BCC,GZRO,DTO,UCTT"
        },
        {
            "coinAllCode": "Bitcoin",
            "coinCode": "BTC",
            "coinImageUrl": "/static/image/coins/bitcoin.png",
            "coinName": "比特币",
            "contact": "",
            "isSupportAdvanced": "Y",
            "mainNetwork": "",
            "noSupportCoin": "BCC,SAN,ICX,EET,ETDM,BCC,GZRO,DTO,UCTT"
        },
          ...
      ],
      "resCode": "800",
      "resMsg": "成功"
  }
```


##### 注意事项
1.目前对外开放的接口主要是高级兑换，所以在请求时，强烈建议在入参中supportType 传指定值advanced 则请求返回结果全部为支持高级兑换的币种
2.该接口请求方式和其余接口不一样，请求方式为GET/POST application/x-www-form-urlencoded



 
## 2.获取兑换汇率基本信息接口

### 接口汇率更新频率
 提供两个币种之间兑换的汇率，汇率更新频率为：4~6s   

### 接口调用
详细参数参见API接口文档 获取基本信息接口 api/v1/getBaseInfo，请求方式为POST application/json   

### 传入参数

 | 入参字段   | 说明   |
| --------   | -----:  |
| minerFee      | 该值为存入币种使用SWFTC作为手续费时的汇率值，SWFTC手续费 = 存币数量 * minerFee |
| receiveCoinFee      | 该值为兑换成功后发币时扣取的网络手续费数量，单位为接收币币种，可用于提前计算用户大致接收到的币的数量，或用于显示用户即将扣除发币网络手续费的数量 |

### 注意事项：
因为高级兑换，用户的手续费方式为原币，非SWFTC，因此可以忽略该字段，手续费固定收取存入原币的千分之二（即：存入0.1btc，实际会扣取0.0002btc作为兑换的手续费，实际兑换时，只拿0.0998btc去做兑换）

### 计算用户兑换实际到账数量

实际到账数量 = （用户存币数量 - 兑换手续费数量）* 汇率 -  链上发币手续费
receiveCoinAmt = （depositCoinAmt - depositCoinAmt * 兑换手续费率） *  instantRate - receiveCoinFee

#### 返回结果
```
{
    "data": {
        "depositMax": "1692690",
        "depositMin": "169269",
        "instantRate": "0.000000538438",
        "minerFee": "0.00004368",
        "receiveCoinFee": "0.009328"
    },
    "resCode": "800",
    "resMsg": "成功"
}
```

## 3.创建订单

### 1. 调用频率限制
创建订单有ip频率限制，2s内下单数量不能超过5个

### 2. 接口调用
详细参数参见API接口文档 高级兑换接口 api/v2/accountExchange，请求方式为POST application/json

### 3. 入参注意事项
| 入参字段   | 说明   |
| --------   | -----:  |
| depositCoinAmt      | 用户创建订单后，需向订单地址上充值depositCoinAmt  |
| receiveCoinAmt      | receiveCoinAmt = depositCoinAmt * instantRate 汇率是通过getBaseInfo接口获得，该值用于记录当时下单时的市场价格，精度保留6位小数 |
| equipmentNo      |环境编号，这个可用于查询属于该编号的所有订单信息，请勿泄露|
| sourceFlag      |用于标识属于哪个平台的订单，该字段项目方产品上线后务必传与SWFT约定的标识来源|
| developerId      |用于记录项目方的订单关联数据，项目方可用该字段来表示该订单归属于自己的某个用户或用于记录自己系统内的订单编号，或其他编号；在订单创建完成后，会回传该字段值（SWFT不支持通过该字段查询寻订单信息）|

### 4.返回结果示例
```
{
      "data": {
          "changeType": "advanced",
          "choiseFeeType": "3",
          "depositCoinAmt": "0.01",
          "depositCoinCode": "BTC",
          "depositCoinFeeAmt": "0.00001",
          "depositCoinFeeRate": "0.001",
          "depositCoinState": "wait_send",
          "destinationAddr": "0x364397e2fc9929f11ba0c03826ef282dd64a829f",
          "detailState": "wait_deposit_send",
          "developerId": "",
          "orderId": "60a31323-0507-446b-866c-973a91856e8c",
          "orderState": "wait_deposits",
          "platformAddr": "3HKL2rwot5YezHGdc5yAabZo74TJeTkVqb",
          "receiveCoinAmt": "0.33642",
          "receiveCoinCode": "ETH",
          "receiveSwftAmt": "18.06",
          "refundAddr": "18orDLFMp3fGoy5Uk93LDGTGbxWEm7b7FY",
          "refundCoinAmt": "",
          "refundCoinMinerFee": "",
          "refundDepositTxid": "",
          "refundSwftAmt": "",
          "swftCoinFeeRate": "0.0005",
          "swftCoinState": "",
          "swftReceiveAddr": "",
          "swftRefundAddr": "",
          "transactionId": ""
      },
      "resCode": "800",
      "resMsg": "成功"
  }
```

## 4.查询订单状态

### 1. 接口调用
详细参数参见API接口文档 查询订单状态接口 api/v2/queryOrderState，请求方式为POST application/json

### 2.入参注意事项
| 入参字段   | 说明   |
| --------   | -----:  |
| minerFee      | 该值为存入币种使用SWFTC作为手续费时的汇率值，SWFTC手续费 = 存币数量 * minerFee |

## 5.批量获取兑换汇率基本信息接口

### 1.接口汇率更新频率
批量获取兑换汇率基本信息接口，汇率更新频率为：4~6s      

### 2.接口调用
详细参数参见API接口文档 批量获取兑换汇率基本信息接口 api/v1/getInfo，请求方式为POST application/json  

### 3. 入参注意事项
| 入参字段   | 说明   |
| --------   | -----:  |
|  minerFee      | 该值为存入币种使用SWFTC作为手续费时的汇率值，SWFTC手续费 = 存币数量 * minerFee  |
|receiveCoinFee |该值为兑换成功后发币时扣取的网络手续费数量，单位为接收币币种，可用于提前计算用户大致接收到的币的数量，或用于显示用户即将扣除发币网络手续费的数量|

### 4.返回结果示例
```
{
    "data": {
        "SWFTCtoBTC": {
            "depositMax": "100000",
            "depositMin": "10",
            "instantRate": "0.00000048597",
            "minerFee": "0.00100000",
            "receiveCoinFee": "0.000059"
        },
        "SWFTCtoETH": {
            "depositMax": "100000",
            "depositMin": "10",
            "instantRate": "0.000016298659",
            "minerFee": "0.00100000",
            "receiveCoinFee": "0.001794"
        }
    },
    "resCode": "800",
    "resMsg": "成功"
}
```
### 5.注意事项：
因为高级兑换，用户的手续费方式为原币，非SWFTC，因此可以忽略该字段，手续费固定收取存入原币的千分之二（即：存入0.1btc，实际会扣取0.0002btc作为兑换的手续费，实际兑换时，只拿0.0998btc去做兑换）
计算用户兑换实际到账数量
实际到账数量 = （用户存币数量 - 兑换手续费数量）* 汇率 -  链上发币手续费
receiveCoinAmt = （depositCoinAmt - depositCoinAmt * 兑换手续费率） *  instantRate - receiveCoinFee

## 5.移除订单

### 1. 接口调用：
 https://{host}/api/v2/removeOrder

### 2. 请求参数

| 入参字段   | 说明   |
| --------   | -----:  |
|  orderId      | 订单Id,如果是多笔订单,用","分割,正在进行中的订单无法被删除  |

### 3.请求参数示例
```
{
	"orderId":"3a380d57-fe56-41df-a6ba-fda7eac998a3"
}
```
### 4.返回结果示例
```
{
     "data":null,
     "resCode":"800",
     "resMsg":"成功"
}
```
  
