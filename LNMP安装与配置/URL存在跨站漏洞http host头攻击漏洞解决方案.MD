#### 一 跨网站脚本 
跨网站脚本（Cross-site scripting，通常简称为XSS或跨站脚本或跨站脚本攻击）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。这类攻击通常包含了HTML以及用户端脚本语言。 

XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java， VBScript， ActiveX， Flash 或者甚至是普通的HTML。攻击成功后，攻击者可能得到包括但不限于更高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。 防止XSS攻击简单的预防就是对Request请求中的一些参数去掉一些比较敏感的脚本命令。

#### 二 解决方案  
方式一：【使用Nginx的修复方案】
~~~
server {
   listen 8888 default;
   server_name _;
   location / {
        return 403;
   }
}
~~~
添加一个默认 server，当 host 头被修改匹配不到 server 时会跳到该默认 server，该默认 server 直接返回 403 错误。

重启 nginx 即可。

除了这种做法，也可以在目标 server 添加检测规则。比如下面的 if 判断配置。

~~~
server {
  server_name  192.168.0.171;
  listen       8888;
  if ($http_Host !~*^192.168.0.171:8888$){
    return 403;
  }
  include /etc/nginx/default.d/*.conf;
  location / {
    root /www/dvwa;
    index index.php index.html index.htm;
  }
}
~~~
方式二：【基于tocmat的修复方案】
经测试，最低支持Tomcat6.0.x以上版本的修复。

打开tomcat的conf目录中的server.xml文件，将Host节点做如下配置：

 

<Host name="www.baidu.com" appBase="webapps"
unpackWARs="true" autoDeploy="true"
xmlValidation="false" xmlNamespaceAware="false"><!--本机对外域名-->
<Alias>172.19.43.28</Alias><!--本机所支持的所有IP-->
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
prefix="localhost_access_log." suffix=".txt" resolveHosts="false"
pattern="%a %A %b %B %h %H %l %m %p %s %S %t %u %U %v %D %T" />
说白了，这个漏洞是因为你使用了 Host 而没验证它。

String path = request.getContextPath();
String basePath = request.getScheme() + "://"
  + request.getServerName()
  + ":" + request.getServerPort()
  + path + "/";
方式三：【基于Filter的修复方案】
在工程的web.xml中配置下面代码中的拦截器，注意该拦截器一定要放在第一个执行。

最低支持Tomcat7.0.x以上版本的修复

1、首先配置web.xml，添加如下配置信息：

~~
<!-- xSS跨站漏洞filter --> 
  <filter>
    <filter-name>xSSFilter</filter-name>
    <filter-class>com.founder.mrp.web.filter.XSSFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>xSSFilter</filter-name>
    <url-pattern>/*</url-pattern>
 </filter-mapping>
~~~
2、编写过滤器：

~~~
package com.founder.mrp.web.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

public class XSSFilter implements Filter {

    @Override
    public void destroy() {
        // TODO Auto-generated method stub

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException {
        //自定义request包装类,并把它传入过滤器链
        XssHttpServletRequestWrapper xssRequest = new XssHttpServletRequestWrapper((HttpServletRequest)request);
        chain.doFilter(xssRequest , response);

    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {
        // TODO Auto-generated method stub
    }

}
~~~
3、包装类

主要是覆盖实现了getParameter，getParameterValues，getHeader这几个方法，然后对获取的value值进行XSS处理。

~~~
package com.founder.mrp.web.filter;

import java.util.regex.Pattern;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {
    
HttpServletRequest orgRequest = null;
    
    public XssHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
        orgRequest = request;
    }
 
    /**
     * 覆盖getParameter方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getParameterValues(name)来获取<br/>
     * getParameterNames,getParameterValues和getParameterMap也可能需要覆盖
     */
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(xssEncode(name));
        if (value != null) {
            value = xssEncode(value);
        }
        return value;
    }
    
    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(xssEncode(name));
         if(values != null && values.length > 0){
             for(int i =0; i< values.length ;i++){
                 values[i] = xssEncode(values[i]);
             }
         }
        return values;
    }

    /**
     * 覆盖getHeader方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getHeaders(name)来获取<br/>
     * getHeaderNames 也可能需要覆盖
     */
    @Override
    public String getHeader(String name) {
 
        String value = super.getHeader(xssEncode(name));
        if (value != null) {
            value = xssEncode(value);
        }
        return value;
    }
 
    /**
     * 将容易引起xss & sql漏洞的半角字符直接替换成全角字符
     * 
     * @param s
     * @return
     */
    private static String xssEncode(String s) {
        if (s == null || s.isEmpty()) {
            return s;
        }else{
            s = stripXSSAndSql(s);
        }
        StringBuilder sb = new StringBuilder(s.length());
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
            case '<':
                sb.append("＜");
                break;
            case '>':
                sb.append("＞");
                break;            
            case '(':
                sb.append("（");
                break;
            case ')':
                sb.append("）");
                break;
            case '&':
                sb.append("＆");
                break;
            case '|':  
                sb.append("｜");
                break;
            case '+':
                sb.append("＋");
                break;
            case '%':
                sb.append("％");
                break;
            case '@':  
                sb.append("＠");
                break;    
            case '$':
                sb.append("＄");
                break;
            case '#':
                sb.append("＃");
                break;    
            case '\'':
                sb.append("＇");// 转义单引号
                break;
            case '\"':
                sb.append("＂");// 转义双引号
                break;
            case '\\':
                sb.append("＼");//全角斜线
                break;
            default:
                sb.append(c);
                break;
            }
        }
        return sb.toString();
    }
 
    /**
     * 获取最原始的request
     * 
     * @return
     */
    public HttpServletRequest getOrgRequest() {
        return orgRequest;
    }
 
    /**
     * 获取最原始的request的静态方法
     * 
     * @return
     */
    public static HttpServletRequest getOrgRequest(HttpServletRequest req) {
        if (req instanceof XssHttpServletRequestWrapper) {
            return ((XssHttpServletRequestWrapper) req).getOrgRequest();
        }
 
        return req;
    }
 
    /**
     * 
     * 防止xss跨脚本攻击（替换，根据实际情况调整）
     */
 
    public static String stripXSSAndSql(String value) {
        if (value != null) {
            // NOTE: It's highly recommended to use the ESAPI library and
            // uncomment the following line to
            // avoid encoded attacks.
            // value = ESAPI.encoder().canonicalize(value);
            // Avoid null characters
            value = value.replaceAll("", "");
            // Avoid anything between script tags
            Pattern scriptPattern = Pattern.compile("<[\r\n| | ]*script[\r\n| | ]*>(.*?)</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid anything in a src="..." type of e-xpression
            scriptPattern = Pattern.compile("src[\r\n| | ]*=[\r\n| | ]*[\\\"|\\\'](.*?)[\\\"|\\\']", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Remove any lonesome </script> tag
            scriptPattern = Pattern.compile("</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Remove any lonesome <script ...> tag
            scriptPattern = Pattern.compile("<[\r\n| | ]*script(.*?)>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid eval(...) expressions
            scriptPattern = Pattern.compile("eval\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid e-xpression(...) expressions
            scriptPattern = Pattern.compile("e-xpression\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid javascript:... expressions
            scriptPattern = Pattern.compile("javascript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid vbscript:... expressions
            scriptPattern = Pattern.compile("vbscript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid onload= expressions
            scriptPattern = Pattern.compile("onload(.*?)=", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Remove any lonesome <script ...> tag
            scriptPattern = Pattern.compile("<iframe>(.*?)</iframe>",Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            scriptPattern = Pattern.compile("</iframe>",Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            scriptPattern = Pattern.compile("<iframe(.*?)>",Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
        }
        return value;
    }

}
~~~
