#**消息聚合推送平台**

##**1. 业务需求背景**
   目前司内KM、WiKi、Mail三个平台中信息多且更新快，而这三者中不乏有一些优秀文章、工程资源等。但司内员工的工作强度大，
如非刻意寻求资源，可能不会太刻意去这三个平台查阅技术文章或工程资源等，这很可能会造成优秀文章/工程被忽略，而有心者因为种种原因而错过这些资源。
所以可以为内部提供一个消息主动推送的服务，这样能让所有人看到三大平台的更新动态，有利于去筛选出自己感兴趣的文章/工程。


##**2. 技术方案**
###**2.1 整体框架图**
![消息聚合平台整体框架图](https://msgplat.oss-cn-beijing.aliyuncs.com/%E6%B6%88%E6%81%AF%E8%81%9A%E5%90%88%E6%8E%A8%E9%80%81%E5%B9%B3%E5%8F%B0%E6%96%B9%E6%A1%88.png?Expires=1567616048&OSSAccessKeyId=TMP.hXmVyGfsaMx7ksv8Qe4ecxEmFAeMkiqv2NDsUqfpr4iPkmXFs2KxC3kewfeoGDZ34thUx4M4m3GZmWXSFDyazKVKWKuDLqzMYFjxcdfzWkTkj2NrJVRjvPSo9Zez1T.tmp&Signature=Z8S5Snw72jOBz1qSxbsoN75Ldr8%3D "消息聚合推送平台整体框架图")

###**2.2 用户流程图**
![用户请求路程图](https://msgplat.oss-cn-beijing.aliyuncs.com/%E7%94%A8%E6%88%B7%E8%AF%B7%E6%B1%82%E6%B5%81%E7%A8%8B%E5%9B%BE.png?Expires=1567616096&OSSAccessKeyId=TMP.hXmVyGfsaMx7ksv8Qe4ecxEmFAeMkiqv2NDsUqfpr4iPkmXFs2KxC3kewfeoGDZ34thUx4M4m3GZmWXSFDyazKVKWKuDLqzMYFjxcdfzWkTkj2NrJVRjvPSo9Zez1T.tmp&Signature=QyPUJIFFgyPtmAmqXINJiZSy25I%3D
 "消息聚合推送平台整体框架图")

###**2.3 技术支持**
Web: LayUI框架 + JS + PHP<br> 
Svr: golang + MySQL<br>

###**2.4 编码策略**
####**2.4.1 推送策略**
(定时+定量)推送：<br>
标定KM、WiKi、Mail的阈值分别为100,200,100(假定);<br>
三个 goroutine##**2. 技术方案**(GeXxxInfoHandler)以2s/次的频率轮询接口，当累计超过阈值，马上推送；
当10小时(推送间隔)内未超过阈值，而又获取到数据，也进行推送;

注：三个阈值和推送间隔需要根据实际测试来做一个均衡，再看是否需要动态修改阈值和推送间隔的策略;
比如，在流量高峰期间, 在一定的时间内连续n次发生超过阈值的情况，则对阈值*2进行扩容；
在流量低峰期间,就以推送间隔进行推送即可。

####**2.4.2 数据格式**
1)企业微信推送信息中附加的url:<br>
[KM](ttp://127.0.0.1:10000/km_request)<br>
[WiKi](ttp://127.0.0.1:10000/wiki_request)<br>
[Mail](ttp://127.0.0.1:10000/mail_request)<br>
注:上述三个链接后期是服务器链接<br>

2)JSON:<br>
KM/WiKi <===> 后台Svr<br>
后台Svr  <===> Web前端<br>
```
type Interface struct {
    Id     int      `json:"id"`
    Author string   `json:"author"`
    Time   string   `json:"time"`
    Title  string   `json:"title"`
    Url    string   `json:"url"`
} 
```
注：其中 Id 根据拿去到的文章/工程数量做好偏移计算, 并不是直接取用 KM/WiKi 返回的 Id<br>
3)MIME/TEXT/HTML:<br>
Mail <===> 后台Svr <br>
```
此处需要再确定
```
####**2.4.3 数据库接口**
DB字段设计
```
Create TABLE `t_km_table` (
    `f_id` int(11) NOT NULL,
    `f_author` varchar(32) NOT NULL,
    `f_title` varchar(64) NOT NULL,
    `f_url` varchar(256) NOT NULL,
    `f_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATA CURRENT_TIMESTAMP COMMENT '更新时间'
    `f_create_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '创建时间',
    PRIMARY KEY (`f_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

Create TABLE `t_wiki_table` (
    `f_id` int(11) NOT NULL,
    `f_author` varchar(32) NOT NULL,
    `f_title` varchar(64) NOT NULL,
    `f_url` varchar(256) NOT NULL,
    `f_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATA CURRENT_TIMESTAMP COMMENT '更新时间'
    `f_create_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '创建时间',
    PRIMARY KEY (`f_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

====== t_mail_table待定 ======
```
##**3 工程量排期**
09.05 确定邮件解码和网页渲染的方案,并用代码验证<br>
09.06 前端三个接口的 Php 实现, 前端页面实现<br>
后面进度再排,预计时间09.13号之前弄完
