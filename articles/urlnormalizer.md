##Nutch教程——URLNormalizer源码详解 by 逼格DATA



URL正规化（URLNormalize）对大多数网络爬虫来说是一个非常重要的流程,大部分爬虫的去重机制都是基于URL的去重,在实际中,不同的URL有可能对应同一个网页,例如下面的几个URL：

+ [http://nutcher.org/](http://nutcher.org/)
+ [http://nutcher.org/abc/../](http://nutcher.org/abc/../)
+ [http://nutcher.org/./](http://nutcher.org/./)

这些URL其实都指向[http://nutcher.org/](http://nutcher.org/),在文件路径中,../表示上级目录,
./表示当前目录,所以/abc/../和/./都等价于/.如果爬虫没有URL正规化机制,
爬虫会认为[http://nutcher.org/](http://nutcher.org/)、[http://nutcher.org/abc/../](http://nutcher.org/abc/../)、[http://nutcher.org/./](http://nutcher.org/./)是三个不同的页面,而三个URL实际都指向[http://nutcher.org/](http://nutcher.org/),因此会将[http://nutcher.org/](http://nutcher.org/)爬取三次.

可以看出,URL正规化的主要目的就是防止网页的重复采集.这里我们介绍Nutch的URL正规化组件之一,
urlnormalizer-basic,该组件在nutch中对应的类是[BasicURLNormalizer.java](https://github.com/CrawlScript/nutcher/blob/master/nutch-chinese/apache-nutch-1.9/src/plugin/urlnormalizer-basic/src/java/org/apache/nutch/net/urlnormalizer/basic/BasicURLNormalizer.java)，这里我们用源码注释的方式描述urlnormalizer-basic的工作机制：


```java
package org.apache.nutch.net.urlnormalizer.basic;

import java.net.URL;
import java.net.MalformedURLException;

// Slf4j Logging imports
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

// Nutch imports
import org.apache.nutch.net.URLNormalizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.oro.text.regex.*;

/**
 * Converts URLs to a normal form:
 * <ul>
 * <li>remove dot segments in path: <code>/./</code> or <code>/../</code></li>
 * <li>remove default ports, e.g. 80 for protocol <code>http://</code></li>
 * </ul>
 * 将URL转换为正规的形式
 * 移除URL中类似/./和/../的部分
 * 移除默认端口，例如对于http协议，移除URL中指定的80端口
 */
public class BasicURLNormalizer extends Configured implements URLNormalizer {
    public static final Logger LOG = LoggerFactory.getLogger(BasicURLNormalizer.class);

    /*这里使用Perl规范的正则，借助了org.apache.oro.text.regex.Perl5Compiler类*/
    private Perl5Compiler compiler = new Perl5Compiler();
    private ThreadLocal<Perl5Matcher> matchers = new ThreadLocal<Perl5Matcher>() {
        protected Perl5Matcher initialValue() {
          return new Perl5Matcher();
        }
      };
    private final Rule relativePathRule;
    private final Rule leadingRelativePathRule;
    private final Rule currentPathRule;
    private final Rule adjacentSlashRule;

    private final static java.util.regex.Pattern hasNormalizablePattern = java.util.regex.Pattern.compile("/\\.?\\.?/");

    private Configuration conf;

    public BasicURLNormalizer() {
      try {
        // this pattern tries to find spots like "/xx/../" in the url, which
        // could be replaced by "/" xx consists of chars, different then "/"
        // (slash) and needs to have at least one char different from "."
        // 这里希望从URL中找到匹配/xx/../的部分,替换为/
        // xx中的每个字符不能是'/'，且至少出现一个不为'.'的字符
        relativePathRule = new Rule();
        relativePathRule.pattern = (Perl5Pattern)
          compiler.compile("(/[^/]*[^/.]{1}[^/]*/\\.\\./)",
                           Perl5Compiler.READ_ONLY_MASK);
        relativePathRule.substitution = new Perl5Substitution("/");


        // this pattern tries to find spots like leading "/../" in the url,
        // which could be replaced by "/"
        // 如果URL中的路径，以/../起始,将其替换为/
        leadingRelativePathRule = new Rule();
        leadingRelativePathRule.pattern = (Perl5Pattern)
          compiler.compile("^(/\\.\\./)+", Perl5Compiler.READ_ONLY_MASK);
        leadingRelativePathRule.substitution = new Perl5Substitution("/");


        // this pattern tries to find spots like "/./" in the url,
        // which could be replaced by "/"
        // 这里希望从URL中找到匹配/./的部分,替换为/
        currentPathRule = new Rule();
        currentPathRule.pattern = (Perl5Pattern)
          compiler.compile("(/\\./)", Perl5Compiler.READ_ONLY_MASK);
        currentPathRule.substitution = new Perl5Substitution("/");


        // this pattern tries to find spots like "xx//yy" in the url,
        // which could be replaced by a "/"
        // 这里希望从URL中找到匹配//的部分,替换为/
        adjacentSlashRule = new Rule();
        adjacentSlashRule.pattern = (Perl5Pattern)
          compiler.compile("/{2,}", Perl5Compiler.READ_ONLY_MASK);
        adjacentSlashRule.substitution = new Perl5Substitution("/");

      } catch (MalformedPatternException e) {
        throw new RuntimeException(e);
      }
    }

    public String normalize(String urlString, String scope)
            throws MalformedURLException {
        if ("".equals(urlString))                     // permit empty
            return urlString;

        urlString = urlString.trim();                 // remove extra spaces

        URL url = new URL(urlString);

        String protocol = url.getProtocol();
        String host = url.getHost();
        int port = url.getPort();
        String file = url.getFile();

        boolean changed = false;

        if (!urlString.startsWith(protocol))        // protocol was lowercased
            changed = true;

        if ("http".equals(protocol) || "https".equals(protocol) || "ftp".equals(protocol)) {

            if (host != null) {
                String newHost = host.toLowerCase();    // lowercase host
                if (!host.equals(newHost)) {
                    host = newHost;
                    changed = true;
                }
            }

            if (port == url.getDefaultPort()) {       // uses default port 如果URL中出现端口信息，检查是否为URL对应协议默认端口
                port = -1;                              // so don't specify it
                changed = true;
            }

            if (file == null || "".equals(file)) {    // add a slash
                file = "/";
                changed = true;
            }

            if (url.getRef() != null) {                 // remove the ref
                changed = true;
            }

            // check for unnecessary use of "/../"
            String file2 = substituteUnnecessaryRelativePaths(file);

            if (!file.equals(file2)) {
                changed = true;
                file = file2;
            }

        }

        if (changed)
            urlString = new URL(protocol, host, port, file).toString();

        return urlString;
    }

    private String substituteUnnecessaryRelativePaths(String file) {

    	if (!hasNormalizablePattern.matcher(file).find())
    		return file;

        String fileWorkCopy = file;
        int oldLen = file.length();
        int newLen = oldLen - 1;

        // All substitutions will be done step by step, to ensure that certain
        // constellations will be normalized, too
        //
        // URL正规化会一步一步进行，下面给出一个例子
        //
        // For example: "/aa/bb/../../cc/../foo.html will be normalized in the
        // following manner:
        //   "/aa/bb/../../cc/../foo.html"
        //   "/aa/../cc/../foo.html"
        //   "/cc/../foo.html"
        //   "/foo.html"
        //
        // The normalization also takes care of leading "/../", which will be
        // replaced by "/", because this is a rather a sign of bad webserver
        // configuration than of a wanted link.  For example, urls like
        // "http://www.foo.com/../" should return a http 404 error instead of
        // redirecting to "http://www.foo.com".
        //
        Perl5Matcher matcher = (Perl5Matcher)matchers.get();

        while (oldLen != newLen) {
            // substitue first occurence of "/xx/../" by "/"
            oldLen = fileWorkCopy.length();
            fileWorkCopy = Util.substitute
              (matcher, relativePathRule.pattern,
               relativePathRule.substitution, fileWorkCopy, 1);

            // remove leading "/../"
            fileWorkCopy = Util.substitute
              (matcher, leadingRelativePathRule.pattern,
               leadingRelativePathRule.substitution, fileWorkCopy, 1);

            // remove unnecessary "/./"
            fileWorkCopy = Util.substitute
            (matcher, currentPathRule.pattern,
            		currentPathRule.substitution, fileWorkCopy, 1);


            // collapse adjacent slashes with "/"
            fileWorkCopy = Util.substitute
            (matcher, adjacentSlashRule.pattern,
              adjacentSlashRule.substitution, fileWorkCopy, 1);

            newLen = fileWorkCopy.length();
        }

        return fileWorkCopy;
    }


    /**
     * Class which holds a compiled pattern and its corresponding substition
     * string.
     */
    private static class Rule {
        public Perl5Pattern pattern;
        public Perl5Substitution substitution;
    }


  public void setConf(Configuration conf) {
    this.conf = conf;
  }

  public Configuration getConf() {
    return this.conf;
  }

}
```
    

