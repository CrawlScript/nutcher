#Nutch教程——导入Nutch工程，执行完整爬取 by 逼格DATA 

在使用本教程之前，需要满足条件：

1）有一台Linux或Linux虚拟机

2）安装JDK（推荐1.7）

3）安装Apache Ant

下载Nutch源码：

推荐使用Nutch 1.9,官方下载地址：[http://mirrors.hust.edu.cn/apache/nutch/1.9/apache-nutch-1.9-src.zip](http://mirrors.hust.edu.cn/apache/nutch/1.9/apache-nutch-1.9-src.zip)


安装IDE：
推荐使用Intellij或者Netbeans，如果用eclipse也可以，不推荐。
Intellij官方下载地址：[http://www.jetbrains.com/idea/download/](http://www.jetbrains.com/idea/download/)

转换：
Nutch源码是用ant进行构建的，需要转换成eclipse工程才可以导入IDE正确使用,Intellij和Netbeans都可以支持ecilpse工程。
解压下载的apache-nutch-1.9-src.zip，得到文件夹apache-nutch-1.9。
在执行转换之前，我们先修改一下ivy中的一个源，将它改为开源中国的镜像，否则转换的过程会非常缓慢。(ant源码中并没有附带依赖jar包，ivy负责从网上自动下载jar包）。
修改apache-nutch-1.9文件夹中的ivy/ivysettings.xml：

![Nutch教程](http://img.blog.csdn.net/20150209115149896?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

找到：


	 <property name="repo.maven.org"  
	  value="http://repo1.maven.org/maven2/"  
	  override="false"/>  

![Nutch教程](http://img.blog.csdn.net/20150209115455396?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


将value修改为http://maven.oschina.net/content/groups/public/  ,修改后：


	<property name="repo.maven.org"  
	      value="http://maven.oschina.net/content/groups/public/"  
	      override="false"/>  

![Nutch教程](http://img.blog.csdn.net/20150209120024335?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



保存并退出，保证当前目录为apache-nutch-1.9，执行命令：

    ant eclipse -verbose  

然后耐心等待，这个过程ant会根据ivy从中心仓库下载各种依赖jar包，可能要十几分钟。

![Nutch教程](http://img.blog.csdn.net/20150209121445849?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



-verbose参数加上之后可以看到ant过程的详细信息。

10分钟左右，转换成功：

![Nutch教程](http://img.blog.csdn.net/20150209122041824?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



打开Intellij, File -> Import Project ->选择apache-nutch-1.9文件夹，确定后选择Import project from external model(Eclipse)


![Nutch教程](http://img.blog.csdn.net/20150209122723777?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

一直点击next到结束。成功将项目导入Intellij:

![Nutch教程](http://img.blog.csdn.net/20150209123250753?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



源码导入工程后，并不能执行完整的爬取。Nutch将爬取的流程切分成很多阶段，每个阶段分别封装在一个类的main函数中。在外面通过Linux Shell调用这些main函数，来完整爬取的流程。我们在后续教程中会对流程调度做一个详细的说明。
下面我们来运行Nutch中最简单的流程：Inject。我们知道爬虫在初始阶段，是需要人工给出一个或多个url，作为起始点（广度遍历树的树根）。Inject的作用，就是把用户写在文件里的种子(一行一个url，是TextInputFormat)，插入到爬虫的URL管理文件(crawldb，是SequenceFile)中。
从src文件夹中找到org.apache.nutch.crawl.Injector类：

![Nutch教程](http://img.blog.csdn.net/20150209125028901?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



在阅读Nutch源码的过程中，最重要的就是找到每个类的main函数：

![Nutch教程](http://img.blog.csdn.net/20150209125336416?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



可以看到,main函数其实是利用ToolRunner，执行了run(String[] args)。这里ToolRunner.run会从第二个参数(new Injector())这个对象中，找到run(String[] args)这个方法执行。
从run方法中可以看出来，String[] args需要有2个参数，第一个参数表示爬虫的URL管理文件夹(输出），第二个参数表示种子文件夹（输入）。对hadoop中的map reduce程序来说，输入文件夹是必须存在的，输出文件夹应该不存在。我们创建一个文件夹 /tmp/urls，来存放种子文件（作为输入）。

![Nutch教程](http://img.blog.csdn.net/20150209125951812?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


在seed.txt中加入一个种子URL



    http://www.cnbeta.com/  


指定一个文件夹/tmp/crawldb来作为URL管理文件夹（输出）
有一种简单的方法来指定args，直接在main函数下加一行：

![Nutch教程](http://img.blog.csdn.net/20150209130345142?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




    args=new String[]{"/tmp/crawldb","/tmp/urls"};  




运行这个类，我们会发现报错了（下面只给了错误的一部分）：


    Caused by: java.lang.RuntimeException: x point org.apache.nutch.net.URLNormalizer not found.  
        at org.apache.nutch.net.URLNormalizers.<init>(URLNormalizers.java:123)  
        at org.apache.nutch.crawl.Injector$InjectMapper.configure(Injector.java:84)  
        ... 23 more  


这是因为用这种方式执行，按照Nutch默认的配置，不能正确地加载插件。我们需要修改Nutch的配置文件，为插件文件夹指定一个绝对路径，修改conf/nutch-default.xml文件，找到：


    <property>  
            <name>plugin.folders</name>  
            <value>plugins</value>  
            <description>Directories where nutch plugins are located.  Each  
                element may be a relative or absolute path.  If absolute, it is used  
                as is.  If relative, it is searched for on the classpath.</description>  
        </property>  

将value修改为绝对路径  apache-nutch-1.9所在文件夹+"/src/plugin"，比如我的配置：


    <property>  
      <name>plugin.folders</name>  
      <value>/home/hu/apache/apache-nutch-1.9/src/plugin</value>  
      <description>Directories where nutch plugins are located.  Each  
      element may be a relative or absolute path.  If absolute, it is used  
      as is.  If relative, it is searched for on the classpath.</description>  
    </property>  


建议在修改nutch-default.xml时，将原来的配置注释，并复制一份新的修改，方便还原：

现在再运行Injector.java,看到结果：

![Nutch教程](http://img.blog.csdn.net/20150209132132046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



运行成功。


读取爬虫文件：

我们查看程序的输出 tree /tmp/crawldb   ,如果没有tree命令，就直接用资源管理器之类的查看吧：

![Nutch教程](http://img.blog.csdn.net/20150209132344739?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



查看里面的data文件：

    vim /tmp/crawldb/current/part-00000/data  


![Nutch教程](http://img.blog.csdn.net/20150209132537837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkdfREFUQQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



这是一个SequenceFile，Nutch中除了Inject的输入（种子）之外，其他文件 全部以SequenceFile的形式存储。SequenceFile的结构如下：

    key0 value0  
    key1 value1  
    key2  value2  
    ......  
    keyn  valuen  

以key value的形式，将对象序列（key value序列）存储到文件中。我们从SequenceFile头部可以看出来key value的类型。

上面的SequenceFile中，可以看出来，key的类型是org.apache.hadoop.io.Text，value的类型是org.apache.nutch.crawl.CrawlDatum。
下面教程给出如何读取SequenceFile的代码。

新建一个类org.apache.nutch.example.InjectorReader


    package org.apache.nutch.example;  
      
    import org.apache.hadoop.conf.Configuration;  
    import org.apache.hadoop.fs.FileSystem;  
    import org.apache.hadoop.fs.Path;  
    import org.apache.hadoop.io.SequenceFile;  
    import org.apache.hadoop.io.Text;  
    import org.apache.nutch.crawl.CrawlDatum;  
      
    import java.io.IOException;  
      
      
    /** 
     * Created by hu on 15-2-9. 
     */  
    public class InjectorReader {  
        public static void main(String[] args) throws IOException {  
            Configuration conf=new Configuration();  
            Path dataPath=new Path("/tmp/crawldb/current/part-00000/data");  
            FileSystem fs=dataPath.getFileSystem(conf);  
            SequenceFile.Reader reader=new SequenceFile.Reader(fs,dataPath,conf);  
            Text key=new Text();  
            CrawlDatum value=new CrawlDatum();  
            while(reader.next(key,value)){  
                System.out.println("key:"+key);  
                System.out.println("value:"+value);  
            }  
            reader.close();  
        }  
    }  


运行结果：

    key:http://www.cnbeta.com/  
    value:Version: 7  
    Status: 1 (db_unfetched)  
    Fetch time: Mon Feb 09 13:20:36 CST 2015  
    Modified time: Thu Jan 01 08:00:00 CST 1970  
    Retries since fetch: 0  
    Retry interval: 2592000 seconds (30 days)  
    Score: 1.0  
    Signature: null  
    Metadata:   
        _maxdepth_=1000  
        _depth_=1  


我们可以看到，程序读出了刚才Inject到crawldb的url，key是url，value是一个CrawlDatum对象，这个对象用来维护爬虫的URL管理信息，我们可以看到一行：

    Status: 1 (db_unfetched)  

表示当前url为未爬取状态，在后续流程中，爬虫会从crawldb取未爬取的url进行爬取。

完整爬取：

下面给出的是各位最期待的代码，就是如何用Nutch完成一次完整的爬取。官方代码在1.7之前（包括1.7），包含一个Crawl.java，这个代码的main函数可以执行一次完整的爬取，但是从1.7之后就取消了。只保留了使用Linux Shell来调用每个流程，来完成爬取的方法。但是好在取消的Crawl.java修改一下，还是可以使用的。

在爬取之前，我们先修改一下conf/nutch-default.xml中的一个地方，找到：

    <property>  
      <name>http.agent.name</name>  
      <value></value>  
      <description>HTTP 'User-Agent' request header. MUST NOT be empty -   
      please set this to a single word uniquely related to your organization.  
      
      NOTE: You should also check other related properties:  
      
        http.robots.agents  
        http.agent.description  
        http.agent.url  
        http.agent.email  
        http.agent.version  
      
      and set their values appropriately.  
      
      </description>  
    </property>  


在<value></value>中随意添加一个值，修改为：

    <property>  
      <name>http.agent.name</name>  
      <value>test</value>  
      <description>HTTP 'User-Agent' request header. MUST NOT be empty -   
      please set this to a single word uniquely related to your organization.  
      
      NOTE: You should also check other related properties:  
      
        http.robots.agents  
        http.agent.description  
        http.agent.url  
        http.agent.email  
        http.agent.version  
      
      and set their values appropriately.  
      
      </description>  
    </property>  


这个值会在发送http请求时，作为User-Agent字段。

下面给出代码：

    package org.apache.nutch.crawl;  
      
    import java.util.*;  
    import java.text.*;  
      
    // Commons Logging imports  
    import org.apache.commons.lang.StringUtils;  
    import org.slf4j.Logger;  
    import org.slf4j.LoggerFactory;  
      
    import org.apache.hadoop.fs.*;  
    import org.apache.hadoop.conf.*;  
    import org.apache.hadoop.mapred.*;  
    import org.apache.hadoop.util.Tool;  
    import org.apache.hadoop.util.ToolRunner;  
    import org.apache.nutch.parse.ParseSegment;  
    import org.apache.nutch.indexer.IndexingJob;  
    //import org.apache.nutch.indexer.solr.SolrDeleteDuplicates;  
    import org.apache.nutch.util.HadoopFSUtil;  
    import org.apache.nutch.util.NutchConfiguration;  
    import org.apache.nutch.util.NutchJob;  
      
    import org.apache.nutch.fetcher.Fetcher;  
      
    public class Crawl extends Configured implements Tool {  
        public static final Logger LOG = LoggerFactory.getLogger(Crawl.class);  
      
        private static String getDate() {  
            return new SimpleDateFormat("yyyyMMddHHmmss").format  
                    (new Date(System.currentTimeMillis()));  
        }  
      
      
        /* Perform complete crawling and indexing (to Solr) given a set of root urls and the -solr 
           parameter respectively. More information and Usage parameters can be found below. */  
        public static void main(String args[]) throws Exception {  
            Configuration conf = NutchConfiguration.create();  
            int res = ToolRunner.run(conf, new Crawl(), args);  
            System.exit(res);  
        }  
      
        @Override  
        public int run(String[] args) throws Exception {  
      
            /*种子所在文件夹*/  
            Path rootUrlDir = new Path("/tmp/urls");  
            /*存储爬取信息的文件夹*/  
            Path dir = new Path("/tmp","crawl-" + getDate());  
            int threads = 50;  
            /*广度遍历时爬取的深度，即广度遍历树的层数*/  
            int depth = 2;  
            long topN = 10;  
      
            JobConf job = new NutchJob(getConf());  
            FileSystem fs = FileSystem.get(job);  
      
            if (LOG.isInfoEnabled()) {  
                LOG.info("crawl started in: " + dir);  
                LOG.info("rootUrlDir = " + rootUrlDir);  
                LOG.info("threads = " + threads);  
                LOG.info("depth = " + depth);  
                if (topN != Long.MAX_VALUE)  
                    LOG.info("topN = " + topN);  
            }  
      
            Path crawlDb = new Path(dir + "/crawldb");  
            Path linkDb = new Path(dir + "/linkdb");  
            Path segments = new Path(dir + "/segments");  
            Path indexes = new Path(dir + "/indexes");  
            Path index = new Path(dir + "/index");  
      
            Path tmpDir = job.getLocalPath("crawl"+Path.SEPARATOR+getDate());  
            Injector injector = new Injector(getConf());  
            Generator generator = new Generator(getConf());  
            Fetcher fetcher = new Fetcher(getConf());  
            ParseSegment parseSegment = new ParseSegment(getConf());  
            CrawlDb crawlDbTool = new CrawlDb(getConf());  
            LinkDb linkDbTool = new LinkDb(getConf());  
      
            // initialize crawlDb  
            injector.inject(crawlDb, rootUrlDir);  
            int i;  
            for (i = 0; i < depth; i++) {             // generate new segment  
                Path[] segs = generator.generate(crawlDb, segments, -1, topN, System  
                        .currentTimeMillis());  
                if (segs == null) {  
                    LOG.info("Stopping at depth=" + i + " - no more URLs to fetch.");  
                    break;  
                }  
                fetcher.fetch(segs[0], threads);  // fetch it  
                if (!Fetcher.isParsing(job)) {  
                    parseSegment.parse(segs[0]);    // parse it, if needed  
                }  
                crawlDbTool.update(crawlDb, segs, true, true); // update crawldb  
            }  
            /* 
            if (i > 0) { 
                linkDbTool.invert(linkDb, segments, true, true, false); // invert links 
     
                if (solrUrl != null) { 
                    // index, dedup & merge 
                    FileStatus[] fstats = fs.listStatus(segments, HadoopFSUtil.getPassDirectoriesFilter(fs)); 
     
                    IndexingJob indexer = new IndexingJob(getConf()); 
                    indexer.index(crawlDb, linkDb, 
                            Arrays.asList(HadoopFSUtil.getPaths(fstats))); 
     
                    SolrDeleteDuplicates dedup = new SolrDeleteDuplicates(); 
                    dedup.setConf(getConf()); 
                    dedup.dedup(solrUrl); 
                } 
     
            } else { 
                LOG.warn("No URLs to fetch - check your seed list and URL filters."); 
            } 
            */  
            if (LOG.isInfoEnabled()) { LOG.info("crawl finished: " + dir); }  
            return 0;  
        }  
      
      
    }  

运行成功，对网站进行了一个2层的爬取，爬取信息都保存在/tmp/crawl+时间的文件夹中。

    2015-02-09 14:23:17,171 INFO  crawl.CrawlDb (CrawlDb.java:update(115)) - CrawlDb update: finished at 2015-02-09 14:23:17, elapsed: 00:00:01  
    2015-02-09 14:23:17,171 INFO  crawl.Crawl (Crawl.java:run(117)) - crawl finished: /tmp/crawl-20150209142212  


有些时候爬虫爬一层就停止了，有几种原因：

+ 1）种子对应的页面大小超过配置的上限，页面被忽略。
+ 2）nutch默认遵循robots协议，有可能robots协议禁止了爬取，不过出现这种情况日志会给出相关信息。
+ 3）网页没有被正确爬取（这种情况少）。

爬很多门户网站时容易出现第一种情况，这种情况只需要找到conf/nutch-default.xml中的：

    <property>  
      <name>http.content.limit</name>  
      <value>65536</value>  
      <description>The length limit for downloaded content using the http://  
      protocol, in bytes. If this value is nonnegative (>=0), content longer  
      than it will be truncated; otherwise, no truncation at all. Do not  
      confuse this setting with the file.content.limit setting.  
      </description>  
    </property>  


将value设置为-1即可

    <property>  
      <name>http.content.limit</name>  
      <value>-1</value>  
      <description>The length limit for downloaded content using the http://  
      protocol, in bytes. If this value is nonnegative (>=0), content longer  
      than it will be truncated; otherwise, no truncation at all. Do not  
      confuse this setting with the file.content.limit setting.  
      </description>  
    </property>  


如果看到日志中有说被robots协议阻拦，修改Fetcher.java的源码，找到：

    if (!rules.isAllowed(fit.u.toString())) {  
                    // unblock  
                    fetchQueues.finishFetchItem(fit, true);  
                    if (LOG.isDebugEnabled()) {  
                      LOG.debug("Denied by robots.txt: " + fit.url);  
                    }  
                    output(fit.url, fit.datum, null, ProtocolStatus.STATUS_ROBOTS_DENIED, CrawlDatum.STATUS_FETCH_GONE);  
                    reporter.incrCounter("FetcherStatus", "robots_denied", 1);  
                    continue;  
                  }  


将整段代码注释即可。
教程持续更新中。。。。