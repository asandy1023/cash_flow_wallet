# wallet

# 錢包系統

## 錢包系統流程設計

1. 呼叫餘額充值服務
1. 生成交易訂單表
1. 呼叫支付系統接口
1. 跳躍至使用者用戶端
1. 支付成功結果通知
1. 更新交易訂單表狀態
1. 觸發錢包餘額表增加操作並記錄交易明細表


(1)應用App呼叫餘額充值服務,電子錢包微服務先生成餘額交易訂單表,然後呼叫「支付微服務」介面發起針對該交易的支付請求。

(2)之後的流程將脫離電子錢包微服務流程,由應用App與「支付微服務」直接進行支付行為。

(3)在使用者支付成功後,支付系統以訊息的方式將支付結果通知給錢包系統微服務。

(4)錢包系統微服務在收到支付系統的通知後,更新交易訂單表狀態,並操作帳戶增加餘額的邏輯。

## 錢包系統測試

**使用postman呼叫介面**

1. 建立錢包帳戶


post http://127.0.0.1:9090/account/openAcc

```
{
  "userId":10001,
  "accType":0,
  "currency":"TWD"
}
```

**回傳開戶成功**
```
{
  "code": 0,
  "message": "成功",
  "data": {
    "userId": 10001,
    "accNo": "202206173748256109816748254",
    "accType": "0",
    "currency": "TWD"
  }
}
```
2. 充值系統

post http://127.0.0.1:9090/account/chargeOrder
```
{
  "userId": 10001,
  "amount": 1000,
  "currency": "TWD",
  "paymentType": 1,
  "isRenew": 0
}
```
**回傳充值傳送訊息成功**
```
{
  "code": 0,
  "message": "成功",
  "data": {
    "userId": 10001,
    "amount": 1000,
    "currency": "TWD",
    "orderId": "25381",
    "tradeNo": "25381",
    "extraInfo": ""
  }
}
由於電腦網頁支付此時請求並未真正發送至第三方支付渠道,因此這裡的TradeNo用接入方orderId填充
```
3. 充值支付回復錢包系統

支付系統會 post http://127.0.0.1:9090/account/payNotify
```
{
  "orderId": "25381",
  "amount": 1000,
  "currency": "TWD",
  "payStatus": "2"
}
payStatus: 支付訂單狀態,0-待支付;1-支付中;2-支付成功;3-支付失敗
```
**回傳充值成功訊息給使用者**


```
{
  "code": 0,
  "message": "成功",
  "data": {
    "result": "success"
  }
}
```
