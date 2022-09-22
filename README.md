# BigFileTransfer
利用Netty进行大文件传输(zero-copy)

使用Netty自带的ChunkedWriteHandler在客户端和服务器之间传输超大文件

write()方法，将数据包装成PendingWrite塞入发送队列

flush()方法，从发送队列取出数据（如果是ChunkedInput类型的数据，同时根据isEndOfInput()方法，判断stream中是否还有数据），
将其发送到对应的channel中，该过程如果发生阻塞，通过注册监听以及channelWritabilityChanged()方法在通道可写的情况下继续发送。

|>|ByteBuf|>|inbound|>|FileReceiveHandler|>|FileSendHandler|>|FilePacketHandler|
|----|----|----|----|----|----|----|----|----|----|
| | | | | |如果是文件内容直接读取| |使用FileRegion直接发送文件内容| |获取接收文件的属性|

>该项目操作流程：
1. 开启服务端(Server)
2. 开启客户端(Client)，完成与服务端连接
3. 输入文件路径，完成从client->server/server->client的文件发送

有问题可以联系zhangjimic@outlook.com交流学习


这个文件发送的例子测试下来一切正常

![1663854690708](https://user-images.githubusercontent.com/43486326/191765355-b8c3d359-87ec-45c2-a08f-5fbbc45df5f2.png)

		bootstrap.group(boss, worker)
				.channel(NioServerSocketChannel.class)
				.option(ChannelOption.SO_BACKLOG, 1024)
				//.option(ChannelOption.TCP_NODELAY, true)
				.childOption(ChannelOption.TCP_NODELAY, true)
				.childHandler(new ChannelInitializer<NioSocketChannel>() {
					@Override
					protected void initChannel(NioSocketChannel channel) throws Exception {
						ChannelPipeline pipeline = channel.pipeline();
						pipeline.addLast(new FileReceiveServerHandler());
						pipeline.addLast(new FileSendServerHandler());
						pipeline.addLast(new DecodeHandler());
						pipeline.addLast(new EncodeHandler());
						pipeline.addLast(new JoinClusterRequestHandler());
						pipeline.addLast(new FilePacketServerHandler());

						// pipeline.addLast("handler", new MyServerHandler());
						pipeline.addLast(new ExceptionCaughtHandler());
					}
				});

定义一个异常处理类，放在pipeline的最后
在流水线的末尾添加捕获异常的处理函数pipeline.addLast(new ExceptionCaughtHandler());

package com.hhu.server.handler;


import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

public class ExceptionCaughtHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // ..

        //if (cause instanceof BusinessException) {
            System.out.println("BusinessException");
        //}
    }
}
