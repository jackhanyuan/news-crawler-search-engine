# News Web Crawler And Search Enging

11-14
爬虫搜索初步版本完成

11-16
网页快照不能分段，图片不能在正确位置显示

重新解析html代码，分标签返回内容，保留标签，遇到图片修改src为本地对应图片路径

11-17
某些网页图片不显示

分析发现，图片保存名称中含有特殊字符，而url会把特殊字符修改为十六进制，所以匹配不上

保存图片时去除名称中特殊字符

11-18
某些网页图片不显示

分析发现，网页图片没有存储格式，图片被过滤掉没有下载

去除过滤图片格式的代码

11-19
部分页面图文混乱在一起

图片标记前后增加<br>

11-20
重构输出格式，使每条爬取都能显示爬取状态
    获取文章是否正常
    文章筛选是否正常
    图片下载是否正常
    url标记是否正常

11-21
爬虫爬取速度过快，5分钟ip被封

添加防反爬虫机制

    请求头增加随机user-agent

    由于没有代理ip，改回单线程

    重构爬取方案：
        打乱mysql返回的url
        每条爬取增加随机延时0-2s
        每5条爬取延时12s
        每50条爬取延时15分钟
以上方案基本能保持一直爬取，虽然半小时左右ip会被封禁，但是后面又会自动解封，
实测6小时共爬去500条内容

11-22
部署到腾讯云服务器

11-23
返回数据量大时mongo排序内存溢出，mongo默认支持100M以内数据排序，实测返回超500条数据服务器卡死

改用python自带排序，初步解决

11-24
问题：返回数据更大时服务器卡死，实测返回超1000条数据服务器卡死

分析：服务器内存耗尽，所有数据在查询瞬间返回，超出内存

想法：重构mongo数据库集合结构，分离数据和图片，只有查询时快照时再返回图片数据和html数据

解决：mongoDB支持定义返回的字段，故返回字段中去除images，查询语句添加{"article.images": 0}

11-25
问题：返回更多数据时服务器又卡死，实测返回超2000条数据服务器卡死

想法：第一次只返回结果_id，其它数据分页返回，查询哪一页数据返回哪一页数据

解决：想法实现，返回一条_id数据只占用24字节内存，1G内存，支持一次返回180万数据id
每页显示10条，实际每页浏览时，查询只返回10条完整数据

11-26
bug：每一页查询都调用了mongo_search函数来返回id，其实第一次时id已经全部返回
解决：创建全局变量存储第一次返回的id列表，加入if判断语句，只在第一次查询时调用mongo_search，其它每页数据请求时不再调用mongo_search


11-27
问题：返回更多数据时服务器又卡死，实测返回超2500条数据服务器卡死

分析：虽然每次只返回_id字段，但每次查询时，实际集合中所有数据都进了内存

解决：集合拆分，分离数据和图片，图片和html存入snapshots集合，其他正文数据保留在articles集合，每次请求网页快照时再依据url查询

实测：2000条数据，snapshots占了600M，正文数据只有6M，此次拆分合理，未拆分前2000条数据查询耗时8s，修改后耗时1s


11-28
本地搜索图片缓存清理问题

增加函数del_file("./static/snapshots")
每次请求快照前先清理本地图片缓存 

将user-agent下到本地文件，解决每次报错问题


----------------------------------------------------
待优化：
爬虫效率低
    及时更换代理ip，就可以多线程爬取 --》 Tor方法 付费代理池
    替换outline方法，自己解析每个连接正文源码

url爬取很多无用链接
    优化过滤链接方法
    如链接含有comment photos多数不是目标链接 --》是否可以采用机器学习方法
    链接过短多数不是目标链接

mongo查询性能优化  ---》 11-27 数据库重构，彻底解决
    内存不足 mongo内存占用巨大 
    排序优化
    返回数据优化 ---》 11-25彻底解决

不支持中文全文检索
    升级为ElasticSearch方案

scrapy某些网页爬不到
    重构scrapy代码，学习scrapy框架
    报错 反爬虫
    深度遍历和广度遍历
    
网页显示效果优化

手机端显示适配