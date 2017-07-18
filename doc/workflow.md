# 支付流程描述

## 扫码支付

1. 执行pay函数

2. 显示PayActivity

    1. 调用PaymentService的paylist方法，返回当前支持的所有支付方式

    2. 根据1的结果修改界面支付方式的可见性

3. 用户点击扫码支付

4. 调用PaymentService的prepay函数，这个函数的返回值参考paymentservice相关文档

5. 返回值是一个xml形式的String，格式如下

```xml
<xml>
   <return_code><![CDATA[SUCCESS]]></return_code>
   <return_msg><![CDATA[OK]]></return_msg>
   <appid><![CDATA[wx2421b1c4370ec43b]]></appid>
   <mch_id><![CDATA[10000100]]></mch_id>
   <nonce_str><![CDATA[IITRi8Iabbblz1Jc]]></nonce_str>
   <openid><![CDATA[oUpF8uMuAJO_M2pxb1Q9zNjWeS6o]]></openid>
   <sign><![CDATA[7921E432F65EB8ED0CE9755F0E86D72F]]></sign>
   <result_code><![CDATA[SUCCESS]]></result_code>
   <code_url><![CDATA[weixin：//wxpay/s/An4baqw]]></code_url>
   <err_code></err_code>
   <prepay_id><![CDATA[wx201411101639507cbf6ffd8b0779950874]]></prepay_id>
   <trade_type><![CDATA[JSAPI]]></trade_type>
</xml>
```
    1. result_code==FAIL

        这种情况下需要根据,err_code判断是什么错误,当然如果结果中包含err_code_desc可以
        直接使用这个属性提示用户，如果没有，则需要自己进行翻译，然后弹出提示，结束支付流程

    2. result_code==SUCCESS

        1. 读取code_url,以dialog的形式弹出由code_url生成的二维码

        2. 开始一个轮询线程，调用PaymentService的query方法

        3. 当query返回1的时候，提示用户支付成功，关闭所有界面，结束支付流程
            当query返回0的时候，提示用户支付失败，关闭所有界面，结束支付流程
            当前query返回-1的时候，继续轮询

        4. 当用户手动关闭二维码界面，关闭所有界面，结束支付流程

        