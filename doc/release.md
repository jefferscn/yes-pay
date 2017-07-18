# 发布结果描述

## 发布结果组成

1. yes-pay.jar

   这个jar包包含了支付服务端的实现代码

2. 第三方依赖库

    * alipaySDK-20160812.jar
    * alipay-sdk-java20161226110022.jar
    * reflections

        包括了reflections-0.9.10.jar,guava-15.0.jar,javaassist-3.19.0-GA.jar,annotation-2.0.1.jar,objenesis-2.2.jar,
        这个库主要用于搜索接口实现

    * httpclient

      包括了httpclient-4.3.4jar,httpcore-4.3.2.jar

    * guice

        一个IOC库，用于创建类实例，提高整体扩展性，包括guice-4.1.0.jar,javax.inject-1.jar,aopaliance-1.0.jar

    * gson

        一个json库，项目中用与json的序列化和反序列化

3. 文档

    * release.md

        发布结果内容描述

    * usage.md

        使用步骤描述

    * interface.md

        项目开发相关的接口描述
        
    * extend.md

        扩展开发涉及的接口描述

