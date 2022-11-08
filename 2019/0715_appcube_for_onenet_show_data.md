# 使用ONENET应用开发环境（华为appcube）实现树莓派数据报表展示

ONENET新增应用开发环境后，APP的开发功能丰富了很多。但是这套环境使用有些门槛，特别是概念比较多（Flow、Script、Data Access等），一不小心就会被绕晕。本次是在应用开发环境中开发一个报表。

# 设备脚本配置
具体操作如下：首先需要在ONENET设备管理中创建一个设备，然后使用“上报数据点”的API实现设备数据上报，脚本参见文章：[使用中国移动ONENET平台实现树莓派信息自动上报](2018/0725_upload_raspberry_data_to_onenet.md)。
>注意：最近ONENET的API调整过，现在改为json中的一个顶级key对应一个消息流的名称，因此为了合并到一个消息流，本次修改了json消息的嵌套格式，消息流名称改为pi_info，消息体是：

```
{
   "pi_info": {
        "CPUUsage": 99,
        "IntranetIP": "127.0.0.1",
        "MemoryFree": 9.99,
        "MemoryTotal": 9.99,
        "MemoryUsage": 99
    }
}
```

ONENET设备管理的上报数据点API文档参见：[上报数据点API](https://open.iot.10086.cn/doc/book/application-develop/api/TCP/15.%E4%B8%8A%E4%BC%A0%E6%95%B0%E6%8D%AE%E7%82%B9.html)

本次简化起见，仍旧采用HTTP协议。

根据协议，在postman模拟消息，可以正常工作：
>本次postman模拟的消息格式需要完全参考ONENET的API定义。在header中还需要配置api-key，这里略。

![image.PNG](2019/0725/refer1.webp)

postman发送成功后，在ONENET中可以查到数据，如下：
![image.PNG](2019/0725/refer2.webp)

# 应用开发环境开发
然后可以在ONENET应用开发环境进行开发了。

## 创建项目
开发过程：首先点击创建项目，按钮，创建一个APP项目，这里随便起一个名字即可：
![image.PNG](2019/0725/refer3.webp)

## 进入项目开发视图
点击进去后，按照下图12345的顺序，分别创建一个元素（这个系统中管这类东东叫资产），创建顺序是：
1. 事件（Event）
2. 数据接入（Data Access）
3. 对象（Object）
4. 流（Flow）
5. 标准页面（Standard Page）

![image.png](https://upload-images.jianshu.io/upload_images/7311839-b4a378bfc56b1f46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 创建事件
这个场景下，事件是数据接入（Data Access）到流（Flow）的纽带，必须要先创建事件，后续才能成功创建数据接入。创建事件后只需要填几个自定义参数即可，比较简单，详见下图：
![image.png](https://upload-images.jianshu.io/upload_images/7311839-838f0bd74d70b2c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 开发数据接入
数据接入简单来说就是把开发环境外的数据接入进来。开发环境中默认有4个接入源，这里使用ONENET接入源。
从整个流程看来，应该是数据接入后，需要通过KAFKA进行缓冲，估计这么设计是为了解决信令风暴问题，不过感觉对于开发者而言拆得过细了。
![image.png](https://upload-images.jianshu.io/upload_images/7311839-03130a7f84757364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击编辑ONENET输入源，这里可以配置基本信息，比较重要的就是协议和TOKEN了，这里的TOKEN跟ONENET平台约定就好。本次填写1234567890abcdef。
>这里需要注意的是，只有ONENET应用开发环境配置好这个输入源后，才可以在ONENET环境中配置全局推送，否则会报TOKEN INVALID错误！

![image.png](https://upload-images.jianshu.io/upload_images/7311839-ace68cdef00e122f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后就是配置数据源的输入消息体了，这里要注意的是，ONENET的数据流起名为pi_info，但是这里需要以value开头（好多坑。。。）
![image.png](https://upload-images.jianshu.io/upload_images/7311839-fac208cccfb3c841.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来配置channel（这里略，我选择了一个内存通道，没有啥特别注意的）

最后配置输出，这里可以看到这个Data Access的输出是输出到一个事件，所以其实要先把事件配置好，不然这里就没有办法选择了，在事件属性中配置好输入源的各个值（因为输入源的值有嵌套，所以本次都有value_前缀）跟事件属性（之前创建事件时创建的自定义属性）的映射关系。
![](https://upload-images.jianshu.io/upload_images/7311839-74a12ca7fd15883f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 创建对象
接下来创建对象，对象可以理解为数据库表，新建一些字段保存即可。
![image.png](https://upload-images.jianshu.io/upload_images/7311839-b43d68f3c88573ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 创建流
接下来是创建流，本次流做一个小逻辑：
1. 等待事件触发
2. 触发后，如果是失败的，发送告警email
3. 触发成功后，创建记录。

这里需要用到4种assignment，分别是：
1. 开始（start）
2. 等待（wait）
3. 邮件告警（alarm email）
4. 记录保存（record Create）
![image.png](https://upload-images.jianshu.io/upload_images/7311839-ec04d543e231177c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后运行object中的record视图，并点击预览，就可以看到有数据了：
![image.png](https://upload-images.jianshu.io/upload_images/7311839-1fad1139d7c88a37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后再用标准页面做一个仪表盘展示界面即可，界面直接读刚才新建的object中数据即可。
