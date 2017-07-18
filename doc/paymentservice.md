# PaymentService的相关函数描述

## prepay

    预支付接口

### 参数

* paytype(String)

  支付方式，这个传入的值必须和paylist中返回的支付类型保持一致

* ticketNO(String)

  业务单据号，这个单据号在支付的时候必须是唯一的，如果已经支付过的单据会报错

* desc(String)
  
  单据的支付信息，可以任意填写，最好是能够描述本张单据的特征的

* amount(String,long,BigDecimal)

  支付金额，使用元为单位

### 返回值

    返回值是一个String,但是内容对于不同的支付有所不同，比如微信返回的是一个xml,
    支付宝返回的是一个QueryString

## refund

    退款接口，这个函数要能够执行后台必须已经设置好了证书文件的路径

### 参数

* ticketNO(String)
  
  业务单据号，这个单据号必须和支付时候的保持一致

* totalAmount(String,BigDecimal,long)

  单据总金额，这个属性在微信支付时需要，但是在支付宝时是不需要的，统一就都加上了

* amount
 
  退款金额

### 返回值

    一个boolean值，标识是否退款成功，这里返回成功，金额可能不会马上到账，但是只
    要这里返回成功，就一定会到账

## paylist

    返回当前服务器支持的所有支付方式

### 参数

   无

### 返回值

>JSON形式的一个String,内容形式如下

```javascript
    {"paylist":[{"type":"wechat","appid":'xxxxxxx'}]}
```

>当前支持三种支付方式

* wechat

    微信App支付

* wechat_qr

    微信扫码支付

* alipay

    支付宝

## query

>查询一个单据的支付情况，这里的返回值是依据后台回调的结果，所以可能客户端已经完成支付，但是这个接口任然返回false的情况

### 参数

* ticketNO(String)

    必须域支付的时候的单据号保持一致

### 返回值

    一个int值，0-支付失败，1-支付成功,-1-未收到回调