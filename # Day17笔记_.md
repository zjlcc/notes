# Day17笔记

# mediator.net

### 扫描注册


```cs notranslate position-relative overflow-auto
var mediaBuilder = new MediatorBuilder();
var mediator = mediaBuilder.RegisterHandlers(typeof(this).Assembly).Build();
```

### 使用管道

有 5 种不同类型的管道可供使用 [![图像](https://cloud.githubusercontent.com/assets/3387099/21959127/9a065420-db09-11e6-8dbc-ca0069894e1c.png)](https://cloud.githubusercontent.com/assets/3387099/21959127/9a065420-db09-11e6-8dbc-ca0069894e1c.png)

#### 全局接收管道


在消息到达下一个管道和处理程序之前，只要有消息被发送、发布或请求，此管道就会被触发

#### 命令接收管道



该管道将​​在到达命令处理程序之后和之前触发`GlobalReceivePipeline`，该管道将仅用于`ICommand`

#### 事件接收管道



该管道将​​在到达其事件处理程序之后和之前触发`GlobalReceivePipeline`，该管道将仅用于`IEvent`

#### 请求接收管道


该管道将​​在到达其请求处理程序之后和之前触发`GlobalReceivePipeline`，该管道将仅用于`IRequest`

#### 发布管道


当处理程序内部发布时，此管道将被触发`IEvent`，此管道仅用于`IEvent`并且通常用作传出拦截器

