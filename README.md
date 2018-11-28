# DOS_MasterWorkerClient
目前用到的一个message的样式【后续根据需求可以更改】
syntax = "proto2";
package msg;
message Request_all{
	required string myname=1;
	required string srcip=2;
	required string srcport=3;
	required int32 operationtype=4;
	required int32 token=5;
}

myname：独一无二，相当于id，使client和worker都便于被master记录

srcip/srcport并不真正指发送和接收的端口，而是发送请求者想让接收请求者知道的ip和port。

operationtype：请求的类型
operationtype=0:请求注册
operationtype=1:master请求worker回复balance
operationtype=3:client请求master分配一个worker
.etc

token：对方的token，相当于一把钥匙，有钥匙的人才能使用对应接收者的某些服务。默认master的token为0;

example：注册请求(operationtype=0)
worker把自己绑定好的用于监听client请求的ip/port放在srcip/srcport，附上自己的myname="xxxxx"，master的token=0，以及operationtype=0里发给master.
Master收到后首先判断对方有没有权限（token对不对）使用自己的服务，如果没有，则丢弃这个包不管。
如果这是一个合法的请求，则检查请求的类型是否是注册请求，如果是，则把相关信息放进master维护的一个map里。

Tips:
1. 一对bind和connect用的是同一个ip和port（刚刚发的文件两个int main调用worker/master的时候port手滑写得不一样4000!=4003。实际connect的port要随bind的port。）
2. msg传的大小在father.hpp里用#define定义，之后要改大小就用这个数字。比如register请求：
#define SIZE_REG_MSG 256。
其他类似的要在几个class间公用的值、函数等等，最好都写在father.hpp里。但最好用define的形式（常量），全局变量不要太多。
3. 自己的部分写得差不多的时候如果还有不能解决的问题我们就聚在一起写，互相debug。
4. 记得关socket和context

#任务分配：
一共有4对socket
master-worker（2对,socket1,socket3）
master-client(1对，socket2)
worker-client(1对，socket4)

PX：[socket1:registerworker() register()] [socket3:master与worker长期通讯的一对。SendInstruction() ReceiveInstruction()]

LYT:[socket2:client向master发送“我想要计算，请给我分配”。master通过socket3查看负载均衡后把worker信息（ip,port,token等）回复给client,client接收这信息并放进一个数据结构中存储]

QYT:[socket4:client把待计算数据发给worker，worker接收并计算。得出一个计算结果返回给client]


