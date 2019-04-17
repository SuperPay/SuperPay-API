
### SuperPay API接口文档

SuperPay API提供了一种简单易用、功能强大和安全的在线支付方法，来接受比特币、USDT、以太坊和其他加密货币。[了解更多](https://superpay.one)

* [English Version](./README.md)
* [PHP库](https://github.com/SuperPayNet/deepay-php/blob/master/README-CN.md)


## 概览
1. 调用创建订单接口。
2. SuperPay会验证请求是否合法。
    * 如果请求合法，SuperPay响应`HTTP 200`状态码和返回订单数据。在接收到正确的响应后，您应该把用户重定向到支付页(payment_url)。
    * 如果请求不合法，SuperPay会响应`HTTP 200`状态码和返回错误信息。
3. 如果用户成功支付，SuperPay会发送通知到`notify_url`。在订单状态改变时，SuperPay同样会发出通知。
4. 您也可以通过订单号异步查询订单支付状态。


## 获取授权
你需要获取商户ID和API密钥以访问SuperPay API.[马上注册](https://superpay.one) 

## 换算汇率

获取系统支持的加密货币与法币的汇率。此接口是公共的，不需要身份验证。

```
POST https://superpay.one/api/demo/coin2cny
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|coin|string|否|货币符号。例如：BTC, ETH, EOS。如果不传递则返回全部系统支持的加密货币与法币的汇率|


#### 响应示例

```json
{
    "status":1,
    "msg":"",
    "data":[
        {
            "coin":"btc",
            "value":"35163.97"
        },
        {
            "coin":"eth",
            "value":"1138.35"
        }
    ]
}
```

## 创建订单
创建订单纪录并返回订单数据。

```
POST https://superpay.one/api/demo/order
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|mch_id|integer|是|SuperPay分配的商户号|
|out_trade_no|string|是|商户系统内部订单号，在同一个商户号下唯一|
|coin|string|是|加密货币。USDT, ETH, BTC, EOS...|
|amount|double|是|法币金额，即商品金额。例如: 100.2|
|info|string|否|备注内容|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|

#### 响应示例

```json
{
    "status":1,
    "msg":"下单成功",
    "data":{
        "trade_no":"190413152811ethDemorKU5hd5eSD",
        "out_trade_no":"a0015482ebcddd",
        "address":"0xe33b5c1bc3b4c10967bb57b40b9574ea1e40c529",
        "coin":"eth",
        "num":"0.00087893",
        "fee":0,
        "cny":1,
        "info":"",
        "status":0,
        "addtime":1555140491
    }
}
```

## 查询订单状态
通过提交订单时获取到的订单号trade_no, 异步查询订单支付状态

```
POST https://superpay.one/api/demo/trade_status
```

#### 请求参数
|字段名|类型|必填|描述|
|----|---|---|---|
|mch_id|integer|是|SuperPay分配的商户号|
|trade_no|string|是|平台下单成功生成的订单号|
|coin|string|是|加密货币。USDT, ETH, BTC, EOS...|
|sign|string|是|详见<a href="#签名算法">签名算法</a>|


#### 响应示例

```json
{
    "status":1,
    "trade_no":"190413153712usdtDemofExlkITPBK",
    "coin":"eos",
    "num":1.0000,
    "deal":1.0000,
    "addtime":1555140491,
    "updatetime":1555140521,
    "msg":"实际支付1 EOS"
}
```

## <a name="签名算法">签名算法</a>

商家的后端程序和SuperPay基于相同的密钥和算法创建相同的签名，以验证彼此的身份。


#### 签名生成的通用步骤如下：

1. 设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

    特别注意以下重要规则：

    * 参数名ASCII码从小到大排序（字典序）；
    * 如果参数的值为空不参与签名；
    * 参数名区分大小写；
    * 验证调用返回或SuperPay主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。

2. 在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。


#### 举例：

假设传送的参数如下：


```
mch_id: 10001 
trade_no: 190413153712usdtDemofExlkITPBK
info: abcd
```

第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：

```
stringA="info=abcd&mch_id=10001&trade_no=190413153712usdtDemofExlkITPBK";
```

第二步：拼接API密钥：

```
stringSignTemp="stringA&key=spay20191245785412gegscseook24" 
sign=MD5(stringSignTemp)
```
