---
title: lucene&solr
date: 2018-11-22 09:23:07
tags: 全文检索
categories:
- 全文检索
- lucene&solr
---

## 1 需求分析
在一些大型门户网站、电子商务网站等都需要站内搜索功能，使用传统的数据库查询方式实现搜索无法满足一些高级的搜索需求，比如：搜索速度要快、搜索结果按相关度排序、搜索内容格式不固定等，这里就需要使用全文检索技术实现搜索功能。

<!--more-->

### 1.1 实现方法

#### 1.1.1 使用Lucene实现
单独使用Lucene实现站内搜索需要开发的工作量较大，主要表现在：索引维护、索引性能优化、搜索性能优化等，因此不建议采用。

#### 1.1.2 使用solr实现
基于Solr实现站内搜索扩展性较好并且可以减少程序员的工作量，因为Solr提供了较为完备的搜索引擎解决方案，因此在门户、论坛等系统中常用此方案。

### 1.2 什么是solr
Solr 是Apache下的一个顶级开源项目，采用Java开发，它是基于Lucene的全文搜索服务器。Solr提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展，并对索引、搜索性能进行了优化。 
Solr可以独立运行，运行在Jetty、Tomcat等这些Servlet容器中，Solr 索引的实现方法很简单，用 POST 方法向 Solr 服务器发送一个描述 Field 及其内容的 XML 文档，Solr根据xml文档添加、删除、更新索引 。Solr 搜索只需要发送 HTTP GET 请求，然后对 Solr 返回Xml、json等格式的查询结果进行解析，组织页面布局。Solr不提供构建UI的功能，Solr提供了一个管理界面，通过管理界面可以查询Solr的配置和运行情况。

**Solr与Lucene的区别**
Lucene是一个开放源代码的全文检索引擎工具包，它不是一个完整的全文检索引擎，Lucene提供了完整的查询引擎和索引引擎，目的是为软件开发人员提供一个简单易用的工具包，以方便的在目标系统中实现全文检索的功能，或者以Lucene为基础构建全文检索引擎。
 Solr的目标是打造一款企业级的搜索引擎系统，它是一个搜索引擎服务，可以独立运行，通过Solr可以非常快速的构建企业的搜索引擎，通过Solr也可以高效的完成站内搜索功能。
![](https://i.imgur.com/wZClJjQ.jpg)

## 2 solr安装及配置

### 2.1 solr的下载
从[Solr官方网站](http://lucene.apache.org/solr/)下载Solr4.10.3，根据Solr的运行环境，Linux下需要下载lucene-4.10.3.tgz，windows下需要下载lucene-4.10.3.zip。[Solr使用指南可参考](https://wiki.apache.org/solr/FrontPage)

### 2.2 solr的文件结构
下载的Solr4.10.3.zip解压之后如下图
![](https://i.imgur.com/ljBd2hB.jpg)

``` bash
bin：solr的运行脚本
contrib：solr的一些贡献软件/插件，用于增强solr的功能。
dist：该目录包含build过程中产生的war和jar文件，以及相关的依赖文件。
docs：solr的API文档
example：solr工程的例子目录：
--	example/solr：
	该目录是一个包含了默认配置信息的Solr的Core目录。
--	example/multicore：
	该目录包含了在Solr的multicore中设置的多个Core目录。 
--	example/webapps：
    该目录中包括一个solr.war，该war可作为solr的运行实例工程。
licenses：solr相关的一些许可信息
```

### 2.3 solr的运行环境
solr 需要运行在一个Servlet容器中，Solr4.10.3要求jdk使用1.7以上，Solr默认提供Jetty（java写的Servlet容器），本教程使用Tocmat作为Servlet容器，环境如下：

Solr：Solr4.10.3
Jdk：jdk1.8
Tomcat：apache-tomcat-9.0.12

### 2.4 solr整合Tomcat

#### 2.4.1 solrHome与solrCore
创建一个Solr home目录，SolrHome是Solr运行的主目录，目录中包括了运行Solr实例所有的配置文件和数据文件，Solr实例就是SolrCore，一个SolrHome可以包括多个SolrCore（Solr实例），每个SolrCore提供单独的搜索和索引服务。
example\solr是一个solr home目录结构，如下：
![](https://i.imgur.com/nrFgHM5.png)
上图中“collection1”是一个SolrCore（Solr实例）目录 ，目录内容如下所示：
![](https://i.imgur.com/QrtAmyQ.png)
``` bash
说明：
collection1：叫做一个Solr运行实例SolrCore，SolrCore名称不固定，一个solr运行实例对外单独提供索引和搜索接口。
solrHome中可以创建多个solr运行实例SolrCore。
一个solr的运行实例对应一个索引目录。
conf是SolrCore的配置文件目录 。
data目录存放索引文件需要创建
```

#### 2.4.2 整合步骤
第一步：安装tomcat。F:\Server\apache-tomcat-9.0.12
第二步：把solr的war包复制到tomcat 的webapp目录下。
把\solr-4.10.3\dist\solr-4.10.3.war复制到F:\Server\apache-tomcat-9.0.12\webapps下。
改名为solr.war
第三步：solr.war解压。使用压缩工具解压或者启动tomcat自动解压。解压之后删除solr.war
第四步：把\solr-4.10.3\example\lib\ext目录下的所有的jar包添加到solr工程中
第五步：配置solrHome和solrCore。
1）创建一个solrhome（存放solr所有配置文件的一个文件夹）。\solr-4.10.3\example\solr目录就是一个标准的solrhome。
2）把\solr-4.10.3\example\solr文件夹复制到F:\Server路径下，改名为solrhome，改名不是必须的，是为了便于理解。
3）在solrhome下有一个文件夹叫做collection1这就是一个solrcore。就是一个solr的实例。一个solrcore相当于mysql中一个数据库。Solrcore之间是相互隔离。
第六步：告诉solr服务器配置文件也就是solrHome的位置。修改web.xml使用jndi的方式告诉solr服务器。
![](https://i.imgur.com/pMG1Dzh.png)
第七步：启动tomcat
第八步：访问http://localhost:8080/solr/

### 2.5 配置中文分词器

#### 2.5.1 Schema.xml
schema.xml，在SolrCore的conf目录下，它是Solr数据表配置文件，它定义了加入索引的数据的数据类型的。主要包括FieldTypes、Fields和其他的一些缺省设置。
![](https://i.imgur.com/FMcTcsv.png)

#### 2.5.2 域的分类
**普通域：**String，Long等。例如：<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
**动态域：**起到模糊匹配的效果，可以模糊匹配没有定义的域名。例如：<dynamicField name="*_is" type="int"    indexed="true"  stored="true"  multiValued="true"/>
**主键域：** <uniqueKey>id</uniqueKey>一般就使用这个，不需要修改或者添加
**复制域：**用于查询的时候从多个域进行查询，可以将多个域复制到某一个统一域中，然后搜索的时候可以直接从这个统一的域中查询，相当于从多个域中查询。例如：<copyField source="title" dest="text"/>     <copyField source="author" dest="text"/>

#### 2.5.3 安装中文分词器
使用IKAnalyzer中文分析器。

第一步：把IKAnalyzer2012FF_u1.jar添加到solr/WEB-INF/lib目录下。
第二步：复制IKAnalyzer的配置文件和自定义词典和停用词词典到solr的classpath下。
也就是在apache-tomcat-9.0.12\webapps\solr\WEB-INF目录下新建classes目录,将配置文件和词典放进去.
第三步：在schema.xml中添加一个自定义的fieldType，使用中文分析器。
``` bash
<!-- IKAnalyzer-->
    <fieldType name="text_ik" class="solr.TextField">
      <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
    </fieldType>

```
第四步：定义field，指定field的type属性为text_ik
``` bash
<!--IKAnalyzer Field-->
   <field name="title_ik" type="text_ik" indexed="true" stored="true" />
   <field name="content_ik" type="text_ik" indexed="true" stored="false" multiValued="true"/>

```
第四步：重启tomcat
测试：
![](https://i.imgur.com/0IW7wFa.png)

## 3 管理索引库

### 3.1 维护索引

#### 3.1.1 添加/更新文档

**添加单个文档**
![](https://i.imgur.com/np8FGXA.png)

**批量导入数据**
使用dataimport插件批量导入数据。
第一步：把dataimport插件依赖的jar包添加到solrcore（collection1\lib）中，还需要mysql的数据库驱动。
第二步：配置solrconfig.mxl文件，添加一个requestHandler。
![](https://i.imgur.com/hLIiici.png)
``` bash
<requestHandler name="/dataimport" 
class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
     </lst>
  </requestHandler> 
```
第三步：创建一个data-config.xml，保存到collection1\conf\目录下
``` bash
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource type="JdbcDataSource"   
		  driver="com.mysql.jdbc.Driver"   
		  url="jdbc:mysql://localhost:3306/lucene"   
		  user="root"   
		  password="root"/>   
<document>   
	<entity name="product" query="SELECT pid,name,catalog_name,price,description,picture FROM products ">
		 <field column="pid" name="id"/> 
		 <field column="name" name="product_name"/> 
		 <field column="catalog_name" name="product_catalog_name"/> 
		 <field column="price" name="product_price"/> 
		 <field column="description" name="product_description"/> 
		 <field column="picture" name="product_picture"/> 
	</entity>   
</document>   
</dataConfig>
```
第四步：重启tomcat，如图所示
![](https://i.imgur.com/MzczPMt.png)
第五步：点击“execute”按钮导入数据，导入数据前会先清空索引库，然后再导入。

#### 3.1.2 删除文档
删除索引格式如下：
``` bash
1） 删除制定ID的索引 
<delete>
	<id>8</id>
</delete>

2） 删除查询到的索引数据 
<delete>
	<query>product_catalog_name:幽默杂货</query>
</delete>
3） 删除所有索引数据
 <delete>
	<query>*:*</query>
</delete>
```

### 3.2 查询索引
通过/select搜索索引，Solr制定一些参数完成不同需求的搜索：

1. q - 查询字符串，必须的，如果查询所有使用*:*。
2. fq - （filter query）过虑查询，作用：在q查询符合结果中同时是fq查询符合的，例如：过滤查询价格从1到20的记录。也可以在“q”查询条件中使用product_price:[1 TO 20]。
 也可以使用“*”表示无限，例如：
20以上：product_price:[20 TO *]
20以下：product_price:[* TO 20]
3. sort - 排序，格式：sort=<field name>+<desc|asc>[,<field name>+<desc|asc>]… 。示例：product_price desc，按价格降序
4. hl 是否高亮 ,设置高亮Field，设置格式前缀和后缀。例如：  
``` bash
hl.fl:profuct_name 
hl.simple.pre:<span color='red'>
hl.simple.post</span>
```

## 4 使用SolrJ管理索引库

### 4.1 什么是SolrJ
solrj是访问Solr服务的java客户端，提供索引和搜索的请求方法，SolrJ通常在嵌入在业务系统中，通过SolrJ的API接口操作Solr服务，如下图：
![](https://i.imgur.com/LL1vqKz.png)

### 4.2 依赖的jar包
包括solrJ的jar包。还需要solr-4.10.3/example/lib/ext下的jar包
![](https://i.imgur.com/LH39toS.png)

### 4.3 添加文档
第一步：和Solr服务器建立连接。HttpSolrServer对象建立连接。
第二步：创建一个SolrInputDocument对象，然后添加域。
第三步：将SolrInputDocument添加到索引库。
第四步：提交。
``` bash
// 创建solr文档对象
SolrInputDocument document = new SolrInputDocument();
// 域要先定义后使用，注意必须要有id主域
// solr中没有专用的修改方法，会自动根据id进行查找，如果找到了则删除原来的，将新来的加入，如果没找到就直接增加
document.addField("id","a001");
document.addField("product_name","台灯");
document.addField("product_price",149.9);
// 将文档加入solr域中
solrServer.add(document);
// 提交
solrServer.commit();
```

### 4.4 删除文档
根据ID删除文档的代码很简单
``` bash
// 根据主键ID进行删除
solrServer.deleteById("a001");
// 删除所有
solrServer.deleteByQuery("*:*");
// 提交
solrServer.commit();
```

### 4.5 修改文档
在solrJ中修改没有对应的update方法，只有add方法，只需要添加一条新的文档，和被修改的文档id一致就，可以修改了。本质上就是先删除后添加。

### 4.6 查询文档

#### 4.6.1 简单查询
根据索引查询
``` bash
	// 链接solr服务器
    private SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
	// 创建solr查询对象
	SolrQuery solrQuery = new SolrQuery();
	// 查询所有
	solrQuery.setQuery("*:*");
	// 查询，并获取查询的响应
	QueryResponse queryResponse = solrServer.query(solrQuery);
	// 从查询的响应中获取结果集对象
	SolrDocumentList documentList = queryResponse.getResults();
	// 打印一共产生多少条记录
	System.out.println("=========count=======" + documentList.getNumFound());
	for (SolrDocument documents : documentList){
	    System.out.println("============" + documents.get("id"));
	    System.out.println("============" + documents.get("product_name"));
	    System.out.println("============" + documents.get("product_price"));
    System.out.println("=========这是分割线=============");
        }
```
#### 4.6.1 复杂查询
其中包含查询、过滤、分页、排序、高亮显示等处理。
``` bash
// 链接solr服务器
    private SolrServer solrServer = new HttpSolrServer("http://localhost:8080/solr");
	SolrQuery solrQuery = new SolrQuery();
    //根据条件查询
    solrQuery.setQuery("台灯");
    // 设置默认搜索域
    solrQuery.set("df","product_keywords");
    // 设置过滤器
    solrQuery.addFilterQuery("product_price : [1 TO 100]");
    // 设置排序
    solrQuery.setSort("product_price", SolrQuery.ORDER.desc);
    // 设置分页
    solrQuery.setStart(0);
    // 设置查询多少条
    solrQuery.setRows(50);

    // 设置高亮显示
    // 高亮显示默认是关闭的
    solrQuery.setHighlight(true);
    // 设置高亮显示的区域
    solrQuery.addHighlightField("product_name");
    // 设置高亮显示的前缀
    solrQuery.setHighlightSimplePre("<span style=\"color:red\">");
    // 设置高亮显示的后缀
    solrQuery.setHighlightSimplePost("</span>");

    // 查询并获取查询响应
    QueryResponse queryResponse = solrServer.query(solrQuery);
    // 将响应放入查询结果集
    SolrDocumentList documentList = queryResponse.getResults();
    // 显示出查询到多少条信息
    System.out.println("==========count======= " + documentList.getNumFound());
    for (SolrDocument document : documentList){
        Map<String, Map<String,List<String>>> highlighting = queryResponse.getHighlighting();
        List<String> list = highlighting.get(document.get("id")).get("product_name");
        if (list != null & list.size() > 0)
            System.out.println("======high ligthing========" + list.get(0));
        System.out.println(document.get("id"));
        System.out.println(document.get("product_name"));
        System.out.println(document.get("product_price"));
        System.out.println("============分割线===========");
    }

}

```

	本文作者：MinheZ
	版权声明：转载请注明出处！





