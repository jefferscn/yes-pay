# 支付相关接口描述

>不管哪种支付方式，在支付完成之后都回使用通过异步回调的方式通知支付的结果。这个异步通知
结果，开发组已经进行了包装，对于项目的开发人员，只需要开发一个实现即可。

## 接口描述
>接口的定义位于yes-pay.jar中，在发布目录的lib子目录下，定义如下
```java
public interface IPaymentCallback {
	public void doCallback(DefaultContext context,PayNotifyData data);
}
```

### 参数

* context

    >这个参数提供了用户操作数据库的句柄，但是这个参数并不是一个完整的Context,并没有用户的相关信息，也没有Document对象
    ，如果用户需要操作Document则需要自己更具第二个参数中的相关内容自己加载进来。

* data

    >这是一个PayNotifyData对象,PayNotifyData类是一个纯粹的数据类，包含支付结果信息。

    ```java
    private String payType;
	private boolean result;
	private Date payDate;
	private int amount;//分计
	private String businessTicketNO;//业务单号
	private String payTicketNO;//支付单号
	private String error_code;
	private String error_msg;
	private String payerId;//支付帐号
	private Map<String,String> data;
    ```

    * payType

        支付类型(wechat|alipay)
    * result

        支付结果
    * payDate
        
        支付日期
    * amount

        支付金额(分计)
    * businessTicketNO

        业务单据编号，这个编号是发起支付的时候调用pay函数的第一个参数
    * payTicketNO

        支付单据编号，支付平台上的支付号
    * error_code,error_msg

        支付发生错误的时候返回的错误信息，只在result=false的时候才有
    * payerId

        支付人帐号
    * data

        从支付平台返回的所有数据集合，每个支付平台都是不同的,
        微信的数据描述请参考[这个][1]地址,
        支付宝的数据表述请参考[这个][2]地址。
      
### 注册方式
>开发项目的回调实现的时候，只需要在实现类上增加@PaymentCallbackAnnotation这个标注就可以
并且，系统支持多个回调实现，相互之间并没有影响，例子如下

```java
@PaymentCallbackAnnotation
public class TestCallbackImplement implements IPaymentCallback {

	@Override
	public void doCallback(DefaultContext context,PayNotifyData data) {
		// TODO Auto-generated method stub
		System.console().printf("Received %s callback", data.getPayType());
	}

}
```

>这个接口的实现建议单独打包，这样将来如果需要进行替换只需要把包移除即可

  [1]: https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_7&index=3#       "wechat"
  [2]: https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.9nanoc&treeId=59&articleId=103666&docType=1
  "alipay"