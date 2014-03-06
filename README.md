#【NEOCrawler 介绍】

NEOCrawler是nodejs、redis、phantomjs实现的爬虫系统。

* nodejs的非阻塞、异步的特点，用于开发IO密集CPU需求不高的系统表现很出色；
* 网址调度中心，可以控制一定时间后更新抓取，可以限制一个时间片内的并发连接数；
* 集成了phantomjs，无界面的浏览器渲染，可以抓取到js执行后输出的内容，可以定义js动作，在页面上模拟用户操作，返回需要的数据；
* 可以预设cookie，解决需要登录后才能抓取到内容的问题；
* 集成了http proxy功能，所有请求经过proxy router转发到proxy上，访问目标网站的是不同的IP地址，可以避免对方网站屏蔽抓取。
* 不仅仅是将页面html源代码下载下来，还支持实时内容解析，用网址规则来分类，设置不同的结构化摘取规则，摘取规则可以是css选择符、正则表达式。支持多层嵌套摘取。
* 定向抓取可以设置感兴趣的网址规则，爬虫只跟踪符合规则的网址，避免全站下载，浪费资源；
* 根据网址规则分类，设定不同类别的调度权重、重新抓取的周期，调度权重决定了每个抓取时间片内各类网址所占的数目；
* 可以设定页面内容校验规则、排除一些异常页面，可以设定摘取校验规则，只将有效数据入库。爬虫重试多次后抓取异常、校验不通过的抓取将记录下来，便于分析原因；
* 以上提到的所有功能及配置都可以通过web界面进行设置，哪怕是爬虫系统在运行中，调度规则、摘取规则修改后立刻被热刷新到各个爬虫上，爬虫会应用最新的设置；
* 允许多个抓取实例共存，这些实例共享了爬虫系统的主干功能，但各自的配置不同。
* 由三个子系统组成：爬虫系统、调度系统、代理系统；调度系统负责任务调度，决定接下来抓取哪些网页，爬虫系统负责页面下载和结构化摘取，爬虫在检测到网址后将其放入网址库让调度器进行调度，代理系统负责将请求映射到若干http代理服务器上下载资源；
* 爬虫系统结构上参考了scrapy，由core、spider、downloader、extractor、pipeline组成，core是各个组件的联合点和事件控制中心，spider负责队列的进出，downloader负责页面的下载，根据配置规则选择使用普通的html源码下载或者下载后用phantomjs浏览器环境渲染执行js/css，extractor根据摘取规则对文档进行结构化数据摘取，pipeline负责将数据持久化或者输出给后续的数据处理系统。这些组件都提供定制化接口，如果通过配置不能满足需求，可以很容易个性化扩展某个组件的功能。

#【运行环境准备】
* 安装好nodejs 环境，从git仓库clone源码到本地，在文件夹位置打开命令提示符，运行“npm install”安装依赖的模块；
* redis server安装。
* hbase环境，抓取到网页、摘取到的数据将存储到hbase，hbase安装完毕后要讲http rest服务开启，后面的配置中会用到，如果要使用其他的数据库存储，可以不安装hbase，下面的章节中将会讲到如何关闭hbase功能以及定制化自己的存储。

#【实例配置】
* 实例在instance目录下，拷贝一份example，重命名其他的实例名，例如：abc，以下说明中均使用该实例名举例。
* 编辑instance/abc/setting.json
```javascript
{
    /*注意：此处用于解释各项配置，真正的setting.json中不能包含注释*/
    
    "driller_info_redis_db":["127.0.0.1",6379,0],/*网址规则配置信息存储位置，最后一个数字表示redis的第几个数据库*/
    "url_info_redis_db":["127.0.0.1",6379,1],/*网址信息存储位置*/
    "url_report_redis_db":["127.0.0.1",6380,2],/*抓取错误信息存储位置*/
    "proxy_info_redis_db":["127.0.0.1",6379,3],/*http代理网址存储位置*/
    "use_proxy":false,/*是否使用代理服务*/
    "proxy_router":"127.0.0.1:2013",/*使用代理服务的情况下，代理服务的路由中心地址*/
    "download_timeout":60,/*下载超时时间，秒，不等同于相应超时*/
    "save_content_to_hbase":false,/*是否将抓取信息存储到hbase*/
    "crawled_hbase_db":["127.0.0.1",8080,"crawled"],/*hbase rest服务地址及存储的表*/
    "statistic_mysql_db":["127.0.0.1",3306,"crawling","crawler","123"],/*用来存储抓取日志分析结果，需要结合flume来实现，一般不使用此项*/
    "check_driller_rules_interval":120,/*多久检测一次网址规则的变化以便热刷新到运行中的爬虫*/
    "spider_concurrency":5,/*爬虫的抓取页面并发请求数*/
    "spider_request_delay":0,/*两个并发请求之间的间隔时间，秒*/
    "schedule_interval":60,/*调度器两次调度的间隔时间*/
    "schedule_quantity_limitation":200/*调度器给爬虫的最大网址待抓取数量*/
}

#【运行】
<未完待续>