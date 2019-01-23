# 汽车之家口碑频道爬虫(A型,B型,SUV车型)

## 爬虫爬取思路
1. 分析汽车之家口碑频道的PC端网页与移动端网页，发现PC端网页的反爬机制比较多，而移动端网页的反爬机制很少，所以选择从移动端网页入手.
2. 分析移动端网页的评论数据加载方式，发现是通过ajax进行加载，分析访问ajax接口的url，发现是由车型id与评论数量组成，所以只要获得了所有的车型id与当前车型的评论数量
   就可以构造get请求来获取所有评论信息.
3. 分析移动端网页的主界面，发现所有车型信息的加载方式与评论信息的加载方式类似，也是通过ajax进行一次性的加载，可以方便的获取包含所有车型id的json数据.
4. 先访问获取车型id的json数据，提取车型id与评论总数，然后直接利用获得的车型id与评论总数构造get请求，访问获取评论内容的接口，获取json数据.
5. 将获取的json数据使用jsonpath_rw_ext解析，并存入数据库.

## 调度器
使用scrapy_redis进行url的去重及完成断点续爬功能.

## 存储器
使用sqlalchemy连接mysql数据库进行数据去重存储.

## 配置重爬
在爬虫完成后，进入redis数据库，删除db0数据库里的koubei:dupefilter，也就是删除了访问过的url的指纹,让这条url可以再次被访问.之后重新运行爬虫，则爬虫将进行重爬.
同样重爬后，爬虫遍历所有url，每获取一个url，就提取其中的评论信息，根据某个字段来判断这条评论信息是否存在，从而决定是否将这条信息存储进数据库.汽车之家口碑频道下的
(A,B,SUV车型的总共评论数量大概在42W左右)
