---
title: xss & sql漏洞解决
date: 2021-04-15 00:30:30
tags: 
  - xss
categories: 
  - Java
description: xss & sql漏洞解决
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img18.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img18.jpg
---

### web.xml文件配置
```
    <!-- 解决xss & sql漏洞 -->
    <filter>
        <filter-name>xssAndSqlFilter</filter-name>
        <filter-class>com.guochuang.common.filter.XssAndSqlFilter</filter-class>
    </filter>
    <!-- 解决xss & sql漏洞 -->
    <filter-mapping>
        <filter-name>xssAndSqlFilter</filter-name>
        <url-pattern>*</url-pattern>
    </filter-mapping>
```

### 编写过滤器
```
import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

/**
 * 
 * @author BridgeBai
 * extends OncePerRequestFilter
 */
public class XssAndSqlFilter implements Filter  {

    @Override
    public void destroy() {
        // TODO Auto-generated method stub

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        XssAndSqlHttpServletRequestWrapper xssRequest = new XssAndSqlHttpServletRequestWrapper((HttpServletRequest) request);
        chain.doFilter(xssRequest, response);
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {
        // TODO Auto-generated method stub

    }
//   下面是springboot filter配置
//	 //TODO 从配置文件读取
//    private String exclusions = "*.js,*.gif,*.jpg,*.png,*.css,*.ico";
//    private Set<String> excludesPattern;
//    private PatternMatcher pathMatcher = new ServletPathMatcher();
//
//    public XssAndSqlFilter(){
//            excludesPattern = new HashSet<>(Arrays.asList(exclusions.split("\\s*,\\s*")));
//    }
//
//    @Override
//    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
//        String requestURI = request.getRequestURI();
//        String contextPath = request.getContextPath();
//        return isExclusion(requestURI,contextPath);
//    }
//
//    @Override
//    public void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
//        XssAndSqlHttpServletRequestWrapper xssRequest = new XssAndSqlHttpServletRequestWrapper(request);
//        chain.doFilter(xssRequest, response);
//    }
//
//    private boolean isExclusion(String requestURI,String contextPath) {
//        if (requestURI == null) {
//            return false;
//        }
//
//        if (contextPath != null && requestURI.startsWith(contextPath)) {
//            requestURI = requestURI.substring(contextPath.length());
//        }
//
//        for (String pattern : excludesPattern) {
//            if (pathMatcher.matches(pattern, requestURI)) {
//                return true;
//            }
//        }
//        return false;
//    }
}
```

### 包装器
XssAndSqlHttpServletRequestWrapper.java
```
/**
 *
 */
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;


/**
 * 
 * @author BridgeBai
 *
 */
public class XssAndSqlHttpServletRequestWrapper extends HttpServletRequestWrapper {

    public XssAndSqlHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    /**
     * 覆盖getParameter方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getParameterValues(name)来获取<br/>
     * getParameterNames,getParameterValues和getParameterMap也可能需要覆盖
     */
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(HtmlHelper.xssEncode(name));
        if (value != null) {
            value = HtmlHelper.cleanSQLInject(HtmlHelper.xssEncode(value));
        }
        return value;
    }

    public String[] getParameterValues(String s) {
        String str[] = super.getParameterValues(s);
        if (str == null) {
            return null;
        }
        int i = str.length;
        String as1[] = new String[i];
        for (int j = 0; j < i; j++) {
            as1[j] = HtmlHelper.cleanSQLInject(HtmlHelper.xssEncode(str[j]));
        }

        return as1;
    }

    /**
     * 覆盖getHeader方法，将参数名和参数值都做xss & sql过滤。<br/>
     * 如果需要获得原始的值，则通过super.getHeaders(name)来获取<br/>
     * getHeaderNames 也可能需要覆盖
     */
    @Override
    public String getHeader(String name) {

        String value = super.getHeader(HtmlHelper.xssEncode(name));
        if (value != null) {
            value = HtmlHelper.cleanSQLInject(HtmlHelper.xssEncode(value));
        }
        return value;
    }

}
```

HtmlHelper.java
```
import java.util.regex.Pattern;

public class HtmlHelper {
	
	/**
	 * 将容易引起xss & sql漏洞的半角字符直接替换成全角字符
	 * 
	 * @param s
	 * @return
	 */
	public static String xssEncode(String s) {
		if (s == null || s.isEmpty()) {
			return s;
		} else {
			s = stripXSSAndSql(s);
		}
		/*StringBuilder sb = new StringBuilder(s.length() + 16);
		for (int i = 0; i < s.length(); i++) {
			char c = s.charAt(i);
			switch (c) {
			case '>':
				sb.append("＞");// 转义大于号
				break;
			case '<':
				sb.append("＜");// 转义小于号
				break;
			case '\'':
				sb.append("＇");// 转义单引号
				break;
			case '\"':
				sb.append("＂");// 转义双引号
				break;
			case '&':
				sb.append("＆");// 转义&
				break;
			case '#':
				sb.append("＃");// 转义#
				break;
			default:
				sb.append(c);
				break;
			}
		}
		return sb.toString();*/
		return s;
	}

	
	 /**
     * 防止xss跨脚本攻击（替换，根据实际情况调整）
     */

    public static String stripXSSAndSql(String value) {
    	if (value != null) {
            // NOTE: It's highly recommended to use the ESAPI library and
            // uncomment the following line to
            // avoid encoded attacks.
            // value = ESAPI.encoder().canonicalize(value);
            // Avoid null characters
            /** value = value.replaceAll("", ""); ***/
            // Avoid anything between script tags
            Pattern scriptPattern = Pattern.compile(
                    "<[\r\n| | ]*script[\r\n| | ]*>(.*?)</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid anything in a
            // src="http://www.yihaomen.com/article/java/..." type of
            // e-xpression
            scriptPattern = Pattern.compile("src[\r\n| | ]*=[\r\n| | ]*[\\\"|\\\'](.*?)[\\\"|\\\']",
                    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Remove any lonesome </script> tag
            scriptPattern = Pattern.compile("</[\r\n| | ]*script[\r\n| | ]*>", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Remove any lonesome <script ...> tag
            scriptPattern = Pattern.compile("<[\r\n| | ]*script(.*?)>",
                    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid eval(...) expressions
            scriptPattern = Pattern.compile("eval\\((.*?)\\)",
                    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid e-xpression(...) expressions
            scriptPattern = Pattern.compile("e-xpression\\((.*?)\\)",
                    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid javascript:... expressions
            scriptPattern = Pattern.compile("javascript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid vbscript:... expressions
            scriptPattern = Pattern.compile("vbscript[\r\n| | ]*:[\r\n| | ]*", Pattern.CASE_INSENSITIVE);
            value = scriptPattern.matcher(value).replaceAll("");
            // Avoid onload= expressions
            scriptPattern = Pattern.compile("onload(.*?)=",
                    Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL);
            value = scriptPattern.matcher(value).replaceAll("");
        }
        return value;
    }
    
    public static String cleanSQLInject(String src) {
        String temp = src;
        src = src.replaceAll(" (?i)insert ", " forbidI ")
                .replaceAll(" (?i)select ", " forbidS ")
                .replaceAll(" (?i)update ", " forbidU ")
                .replaceAll(" (?i)delete ", " forbidD ")
                .replaceAll(" (?i)and ", " forbidA ")
                .replaceAll(" (?i)or ", " forbidO ");
        if (!temp.equals(src)) {
            System.out.println("输入信息存在SQL攻击！");
            System.out.println("原始输入信息-->" + temp);
            System.out.println("处理后信息-->" + src);
        }
        return src;
    }

}
```