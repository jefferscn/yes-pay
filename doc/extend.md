# 扩展开发
>当前的支付模块中使用了reflections库和guice库进行类的反射操作

## 可扩展的接口

### IParaService
>这个接口是用来获取配置参数的，支付时候使用到的所有参数都是通过这个接口得到的，
系统中已经内置了一个实现YESSysParaService,这个实现从系统的Sys_Para表中读取配置信息
，当然如果项目上有要求必须从其他地方获取参数，可以实现这个接口

```java
public interface IParaService {
	String getPara(String key);
}
```

>如果需要替换系统默认的实现需要在自己的实现上增加一个标记LevelAnnotation,其中有一个属性level,系统中level最大的实现
将被使用。

```java
@LevelAnnotation(level=100)
public class TestParaService implements IParaService {
	private Map<String,String> data = new HashMap<String,String>();
	public TestParaService() {
		super();
		// TODO Auto-generated constructor stub
		data.put(WeChatPayment.CONST_SYSPARA_WECHAT_APIKEY, "xxx");
		data.put(WeChatPayment.CONST_SYSPARA_WECHAT_APPID, "xxx");
		data.put(WeChatPayment.CONST_SYSPARA_WECHAT_PARTERID, "xxx");
		data.put(WeChatPayment.CONST_SYSPARA_WECHAT_CERT_PATH_, "C:\\workroom\\projects\\yes-pay\\apiclient_cert.p12");
		data.put(WeChatPayment.CONST_SYSPARA_WECHAT_URLSCHEMA, "");
		data.put(AlipayPayment.CONST_SYSPARA_ALIPAY_APPID, "xxx");
		data.put(AlipayPayment.CONST_SYSPARA_ALIPAY_PRIVATEKEY, "xxx");
		data.put(AlipayPayment.CONST_SYSPARA_ALIPAY_URLSCHEMA, "");
		data.put(AlipayPayment.CONST_SYSPARA_ALIPAY_ALIPUBLICKEY,"xxx");
	}

	@Override
	public String getPara(String key) {
		// TODO Auto-generated method stub
		return this.data.get(key);
	}

}
```

### IPayment
>这个是支付的实现接口，yes中的函数pay和refund最终都回调到这个接口的一个实现上

```java
public interface IPayment {
	public String prepay(String ticketNO,String desc,String amount);
	public boolean refund(String ticketNO,String totalAmount,String refundAmount) throws RefundException;
	public String payconfig();
	public boolean isSupport();
}
```
#### 函数
* prepay
    >这个是实现预支付的函数，一般在支付的时候，由预支付过程产生一个交互用的字符串，这个字符串
    一般都会包含一些配置信息，wechat的实现是调用了wechat的统一下单接口，返回的结果是在客户端发起
    支付的参数。支付宝这步则不需要到用接口，只是产生了一个调用的服务器地址包含了所有需要的参数。
* refund
    >退款接口
* payconfig
    >这个接口不用于支付，只是返回一个json的字符串，包含了本支付实现的描述，其中必须包含type这个属性，
    微信对应wechat,支付宝对应alipay
* isSupport
    >返回当前系统环境是否支持本实现对应的支付方式

#### 注册方式
>完成了一个新的支付方式的实现之后，需要在系统中进行注册，才能正式支持这种支付方式，那就需要进行注册，
注册的过程只需要在实现类上增加一个Annotation,@PaymentType,如下

```java
@PaymentType(name="wechat")
public class WeChatPayment implements IPayment {
...
}
```

>如果IPayment的实现需要使用IParaService接口，则可以定义一个构建函数，定义一个IParaService的接口即可，
系统使用ioc进行注入，如
```java
	@Inject
	public AlipayPayment(IParaService paraService) {
		super();
		this.paraService = paraService;
	}
```

### IPayNotifyProcess
>这个是异步支付回调的处理实现，每个支付方式都要一个对应的实现，主要的所用是完成支付方式的签名验证，返回数据
的统一化

```java
public interface IPayNotifyProcess {
	public PayNotifyData process(HttpServletRequest request);
	public boolean check(HttpServletRequest request);
	public void success(HttpServletResponse response);
	public void fail(HttpServletResponse response,String msg);
}
```

####　函数

* process

    >这个函数根据调用的Request进行签名验证，然后填充一个PayNotifyData的对象返回。

* check

    >这个函数根据Request进行判断是否是实现对应的支付方式的回调，由于所有的支付回调的地址都是统一的，
    所以需要用这个函数来区分不同的支付方式。

* success

    >所有处理成功之后http的返回内容在这个函数中写入

* fail

    >如果处理发生问题，会调用这个函数，在response中写入执行失败之后的内容，这个函数一般只在process函数发生异常的
    时候才会调用，也就是一般只在签名验证失败的情况下才会调用。

#### 注册方式

>这个实现不需要额外的注入过程，只是需要注意如果需要IParaService对象的时候，可以同IPayment一样处理。

### IPaymentResultPersistService
>这个接口用于定义支付结果的保存，系统默认了一个保存到数据库中的实现，数据库的表名为SYS_PAYMENTINFO

```java
public interface IPaymentResultPersistService {
	public void save(String key,PayNotifyData result);
	public PayNotifyData fetch(String key);
	public void remove(String key);
}
```

#### 函数
* save

    保存支付结果，参数中的key是支付的业务单据号

* fetch

    这个是根据key(业务单据号)得到PayNotifyData对象的函数

* remove

    删除一个业务单据号相关的支付结果

#### 注册方式

>使用@LevelAnnotation进行注册，LevelAnnotation中level属性最大的实现会被启用

### IPrepayProcess
>这个接口在执行实际的支付之前调用，可以让通过这个接口对支付之后的参数进行干预，特别是在一张单据多次支付的情况下，可以通过这个接口重置支付的业务单据号

```java
public interface IPrepayProcess {
	PayData process(DefaultContext context,PayData data);
}
```

#### 函数
* process

	处理函数，第二个参数是支付信息，里面包含了ticketNO,desc,amount,这里amount是以分为单位的一个long型数字,需要返回一个PayData