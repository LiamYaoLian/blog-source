---
title: Java Web
date: 2022-04-17 19:23:29
tags:
---
## XML
* Extensible Markup Language, has tags, to save and transfer data or config
### markup
* lower case, "-"
* no duplicate names in different layers
* &lt, &gt, &amp, &apos, &quot
### cdata
* element content that parser to interpret as character data, not markup
* `<![CDATA[ content here ]]>`
### Semantic constraint
#### dtd
* <!ELEMENT>: to define what elements and how many are allowed
  - <!ELEMENT hr (employee) >: one "employee" node under the "hr" node
  - <!ELEMENT hr (employee+) >: at least one "employee" node under the "hr" node
  - <!ELEMENT hr (employee*) >: can have 0 to n "employee" nodes under the "hr" node
  - <!ELEMENT hr (employee?) >: at most one "employee" node under the "hr" node
  - <!ELEMENT employee (name, age, salary, department)>: must have four nodes with this order
  - <!ELEMENT name (#PCDATA)>: #PCDATA is text
  - <!ATTLIST employee no CDATA "">: the type of the attribute "no" in "employee" is CDATA and the default value is ""
* `<!DOCTYPE hr SYSTEM "hr.dtd">` in XML to import the dtd file
#### XML schema
```
<?xml version="1.0" encoding="UTF-8"?>
<schema xmlns="http://www.w3.org/2001/XMLSchema">
	<element name="hr">
		<!-- complexType标签含义是复杂节点，包含子节点时必须使用这个标签 -->
		<complexType>
			<sequence>
				<element name="employee" minOccurs="1" maxOccurs="9999">
					<complexType>
						<sequence>
							<element name="name" type="string"></element>
							<element name="age">
								<simpleType>
									<restriction base="integer">
										<minInclusive value="18"></minInclusive>
										<maxInclusive value="60"></maxInclusive>
									</restriction>
								</simpleType>
							</element>
							<element name="salary" type="integer"></element>
							<element name="department">
								<complexType>
									<sequence>
										<element name="dname" type="string"></element>
										<element name="address" type="string"></element>
									</sequence>
								</complexType>
							</element>
						</sequence>
						<attribute name="no" type="string" use="required"></attribute>
					</complexType>
				</element>
			</sequence>
		</complexType>
	</element>
</schema>

```
### DOM
* document object model
* Dom4j
```

import java.util.List;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

public class HrReader {
	public void readXml(){
		String file = "d:/workspace/xml/src/hr.xml";

		SAXReader reader = new SAXReader();
		try {
			Document document = reader.read(file);
			Element root = document.getRootElement();
			List<Element> employees =  root.elements("employee");
			for (Element employee : employees){
				Element name = employee.element("name");
				String empName = name.getText();
				System.out.println(empName);
				System.out.println(employee.elementText("age"));
				System.out.println(employee.elementText("salary"));
				Element department = employee.element("department");
				System.out.println(department.element("dname").getText());
				System.out.println(department.element("address").getText());
				Attribute att = employee.attribute("no");
				System.out.println(att.getText());
			}
		} catch (DocumentException e) {
			e.printStackTrace();
		}
	}
	public static void main(String[] args) {
		HrReader reader = new HrReader();
		reader.readXml();
	}

}

```

```
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.Writer;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

public class HrWriter {
	public void writeXml(){
		String file = "d:/workspace/xml/src/hr.xml";
		SAXReader reader = new SAXReader();
		try {
			Document document = reader.read(file);
			Element root = document.getRootElement();
			Element employee = root.addElement("employee");
			employee.addAttribute("no", "3311");
			Element name = employee.addElement("name");
			name.setText("Liam");
			employee.addElement("age").setText("37");
			employee.addElement("salary").setText("3600");
			Element department = employee.addElement("department");
			department.addElement("dname").setText("IT");
			department.addElement("address").setText("N2J 3C5");
			Writer writer = new OutputStreamWriter(new FileOutputStream(file) , "UTF-8");
			document.write(writer);
			writer.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	public static void main(String[] args) {
		HrWriter hrWriter = new HrWriter();
		hrWriter.writeXml();
	}
}
```
### XPath
* to search data in XML
![Xpath 1](../image/java-web/1.png)
![Xpath 2](../image/java-web/2.png)
![Xpath 2](../image/java-web/3.png)
* Jaxen
  - a Xpath library that supports DOM, XOM, Dom4j, and JDOM
  - Dom4j depends on Jaxen
```
import java.util.List;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;

public class XPathTestor {
	public void xpath(String xpathExp){
		String file = "E:/lianxi/xml/hr.xml";
		SAXReader reader = new SAXReader();
		try {
			Document document = reader.read(file);

			List<Node> nodes = document.selectNodes(xpathExp);
			for(Node node : nodes){
				Element emp = (Element)node;
				System.out.println(emp.attributeValue("no"));
				System.out.println(emp.elementText("name"));
				System.out.println(emp.elementText("age"));
				System.out.println(emp.elementText("salary"));
				System.out.println("==============================");
			}


		} catch (DocumentException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		XPathTestor testor = new XPathTestor();
//		testor.xpath("/hr/employee");
//		testor.xpath("//employee");
//		testor.xpath("//employee[salary<4000]");
//		testor.xpath("//employee[name='������']");
//		testor.xpath("//employee[@no=3304]");
//		testor.xpath("//employee[1]");
//		testor.xpath("//employee[last()]");
		//testor.xpath("//employee[position()<3]");
		testor.xpath("//employee[3] | //employee[8]");

	}
}

```

## Tomcat
* Tomcat: web application server software, a container to run Servlet
* configuration
  - port
  - ContextPath
  - 应用自动重载


## Servlet
* Server Applet, to create dynamic web content
* process: request -> Tomcat -> search in web.xml to find the relevant Servlet -> Servlet runs -> response
* steps
  - extends HttpServlet
  - override service(HttpServletRequest request , HttpServletResponse response)
    ```
    PrintWriter out =  response.getWriter();
    out.println("<a href='http://www.baidu.com'>Baidu</a>");
    ```
  - configure web.xml or use annotation
    - XML
      - <load-on-startup>0~9999</load-on-startup>, 0 means this Servlet has the highest priority
      - default home page
      - status code and page
      - Servlet 通配符映射及初始化参数
      ```
      <?xml version="1.0" encoding="UTF-8"?>
      <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
        <display-name>servlet_advanced</display-name>
        <welcome-file-list>
          <welcome-file>index.html</welcome-file>
          <welcome-file>index.htm</welcome-file>
          <welcome-file>index.jsp</welcome-file>
          <welcome-file>default.html</welcome-file>
          <welcome-file>default.htm</welcome-file>
          <welcome-file>default.jsp</welcome-file>
        </welcome-file-list>
        <servlet>
        	<servlet-name>patternServlet</servlet-name>
        	<servlet-class>com.ll.servlet.pattern.PatternServlet</servlet-class>
        </servlet>
        <servlet-mapping>
        	<servlet-name>patternServlet</servlet-name>
        	<url-pattern>/pattern/*</url-pattern>
        </servlet-mapping>
        <context-param>
        	<param-name>copyright</param-name>
        	<param-value>© 2018 ll.com</param-value>
        </context-param>
        <context-param>
        	<param-name>title</param-name>
        	<param-value>Liam Lian</param-value>
        </context-param>
        <!-- 指定错误页面 -->
        <error-page>
        	<error-code>404</error-code>
        	<location>/error/404.html</location>
        </error-page>
        <error-page>
        	<error-code>500</error-code>
        	<location>/error/500.jsp</location>
        </error-page>
      </web-app>
      ```
    - `@WebServlet`
      ```
      @WebServlet("/anno/*")
      public class AnnotationServlet extends HttpServlet{

        @Override
        protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.getWriter().println("I'm annotation servlet!");
        }

      }
      ```

  - http://ip:port/context-path/url-mapping: context-path is project name by default
* methods
  - request.getParameter() // get one parameter
  - request.getParameterValues() // get multiple parameter values
  - request.setAttribute(属性名，属性值)
  - Object attr = request.getAttribute(属性名)
  - request.getRequestURL()
  - request.getServletContext()
  ```
  ServletContext context = request.getServletContext();
  String copyright = context.getInitParameter("copyright");
  context.setAttribute("copyright", copyright);
  ```
  - request.getSession()
  - request.getHeader("User-Agent");

  - service() // can deal with all requests
  - doGet() // deal with get requests
  - doPost // deal with post requests
    ```
    public void doPost(HttpServletRequest request , HttpServletResponse response) throws IOException{
      String name = request.getParameter("name");
      response.getWriter().println("<h1 style='color:red'>" + name + "</h1>");
    }
    ```


* life cycle
  - load: web.xml
  - constructor
  - init()
  - provide service: service(), etc
  - destroy()
* note: content in WEB-INF can be accessed only by the server, secure

### Forward and Redirect
* request.getRequestDispatcher().forward(): `request.getRequestDispatcher("/direct/index").forward(request, response);`
  - one request
  - url that appears will not change
* response.sendRedirect(): `response.sendRedirect("/servlet_advanced/direct/index");`
  - two requests
#### Cookie
* sent with request to Tomcat
* default
```
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/cookies/index")
public class llIndexServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

    public llIndexServlet() {
        super();
    }

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		Cookie[] cs = request.getCookies();
		if (cs == null) {
			response.getWriter().println("user not login");
			return;
		}
		String user = null;
		for (Cookie c : cs) {
			System.out.println(c.getName() + ":" + c.getValue());
			if (c.getName().equals("user")) {
				user = c.getValue();
				break;
			}
		}

		if (user == null) {
			response.getWriter().println("user not login");
		} else {
			response.getWriter().println("user:" + user);
		}
	}

}

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/cookies/login")
public class llLoginServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

    public llLoginServlet() {
        super();
    }

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		Cookie cookie = new Cookie("user" , "admin");
		cookie.setMaxAge(60 * 60 * 24 * 7);
		response.addCookie(cookie);
		response.getWriter().println("login success");
	}
}
```
#### Session
* data related to browser window, saved in Tomcat server memory
* 通过Cookie的SessionId值提取用户数据
* methods
  - request.getSession()
  - getAttribute()
  - setAttribute()
  - removeAttribute()
  - setMaxInactiveInterval: 设置Session超时时间
#### ServletContext
* one global object
#### Scope
* HttpServletRequest
* HttpSession
* ServletContext: after the application starts and before it closes
#### Char set
* get: before Tomcat 8, server.xml增加URIEncoding="UTF-8"
* post: `request.setCharacterEncoding("UTF-8");`
* response: `response.setContentType("text/html;charset=utf-8");`

### Filter
* 对URL进行统一的拦截处理, 过滤器对象在Web应用启动时被创建且全局唯一, 唯一的过滤器对象在并发环境中采用“多线程”提供服务
过
* implements javax.servlet.Filter
* doFilter()
* web.xml
 ```
  <filter>
    <filter-name>MyFirstFilter</filter-name>
    <filter-class>com.ll.filter.MyFirstFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>MyFirstFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
 ```
 ```
 @WebFilter (filterName="MyAnnoationFilter",urlPatterns="/*")
  public class MyAnnotationFilter implements Filter {
  }
 ```
  - `<init-param>`
    ```
    import java.io.IOException;
    import javax.servlet.Filter;
    import javax.servlet.FilterChain;
    import javax.servlet.FilterConfig;
    import javax.servlet.ServletException;
    import javax.servlet.ServletRequest;
    import javax.servlet.ServletResponse;
    import javax.servlet.annotation.WebFilter;
    import javax.servlet.annotation.WebInitParam;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;

    @WebFilter(filterName="CharacterEncodingFilter",urlPatterns="/*",
    	initParams= {
    			@WebInitParam(name="encoding" , value="GBK"),
    			@WebInitParam(name="p1" , value="v1"),
    			@WebInitParam(name="p2" , value="v2")
    	})
    public class CharacterEncodingFilter implements Filter {
    	private String encoding;
    	@Override
    	public void init(FilterConfig filterConfig) throws ServletException {
    		encoding = filterConfig.getInitParameter("encoding");
    		System.out.println("encoding:"+encoding);
    	}

    	@Override
    	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    			throws IOException, ServletException {
    		HttpServletRequest req = (HttpServletRequest)request;
    		req.setCharacterEncoding(encoding);
    		HttpServletResponse res = (HttpServletResponse)response;
    		res.setContentType("text/html;charset=" + encoding);
    		chain.doFilter(request, response);
    	}

    	@Override
    	public void destroy() {
    	}

    }
    ```
    ```
    <filter>
      <filter-name>CharacterEncodingFilter</filter-name>
      <filter-class>com.ll.filter.CharacterEncodingFilter</filter-class>
      <init-param>
        <param-name>encoding</param-name>
        <param-value>GBK</param-value>
      </init-param>
      <init-param>
        <param-name>p1</param-name>
        <param-value>v1</param-value>
      </init-param>
      <init-param>
        <param-name>p2</param-name>
        <param-value>v2</param-value>
      </init-param>
    </filter>
    ```
#### Lifecycle
* Filter.init()
* Filter.doFilter()
* Filter.destroy()

#### ServletRequest
* ServletRequest是所有请求的顶层接口，代表任何"请求"
* HttpServletRequest是Http协议请求的抽象接口，是J2EE标准
* RequestFacade是HttpServletRequest接口的实现类，由Tomcat创建

#### / 映射的问题
* / 指映射 Web应用根路径，且只对Servlet生效
* 默认首页index.jsp会让 / 失效
* / 与 /* 含义不同，前者指向根路径，后者代表所有

#### chain
* 过滤器的执行顺序以<filter-mapping>为准
* 调用chain.doFilter()将请求向后传递

## JSP
* Java server page, separate Java code and HTML, in nature is Servlet, .jsp
* .jsp -> Servlet source code ->(compiled) byte code
#### JSP code block
* <% java code %>: `<% System.out.println("Hello"); %>`
#### JSP declaration block
* <%! java code %>: `<%! public int add(int a,int b){return a+b;} %>`
#### output
* <%= java code %>: `<%= "<b>" + name + "</b>" %>`
#### JSP commands
* `<%@ page import="java.util.*" %>`
  - <%@ page %> 定义当前JSP页面全局设置: `<%@page import="java.util.*,java.text.*" contentType="text/html;charset=utf-8" isErrorPage="true"%>`
  - <%@ include %> 将其他JSP页面与当前JSP页面合并: `<%@include file="include/header.jsp" %>`
  - <%@ taglib %> 引入JSP标签库
#### comments
* <%-- comments --%>
* // 、/*..*/ 用于注释<%%>java代码
* `<!-- html -->` HTML注释
### JSP objects
* request: HttpServletRequest
* response: HttpServletResponse
* session: HttpSession
* application: ServletContext
* out: PrintWriter
* page: this (this page)
* pageContext: PageContext
* config: ServletConfig
* exception: Throwable
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<%
		String url = request.getRequestURL().toString(); // HttpServletRequest
		response.getWriter().println(url);//HttpServletResponse
	%>
	<% out.println("<br>ABCCC");
		session.setAttribute("user", "Liam");
		out.println((String)session.getAttribute("user"));
	%>
	<%
		String cp = application.getInitParameter("copyright") ; //ServletContext
		out.println("<hr/>");
		out.println(cp);
		pageContext.getRequest();
		pageContext.getResponse();
		pageContext.getSession();
		pageContext.getServletContext();
	%>
</body>
</html>
```
#### example
```
<%@page import="java.util.*,java.text.*" contentType="text/html;charset=utf-8" %>
<%!
	boolean isPrime(int num){
		boolean flag = true;
		for(int j = 2 ; j < num ; j++){
			if(num % j == 0){
				flag = false;
				break;
			}
		}
		return flag;
	}
%>
<%
	List<Integer> primes = new ArrayList();
	for(int i = 2 ; i<=1000 ; i++){
		boolean flag = isPrime(i);

		if(flag == true){
			primes.add(i);
		}
	}
%>
<%
	for(int p : primes){
%>
	<h1 style="color:red;"><%=p %>是质数</h1>
<%
	}
%>

```
### EL
* Expression Language, to simplify JSP
* ${expression}
#### Scope
* pageScope: `{pageScope.student.name}`
* requestScope
* sessionScope
* applicationScope

* if scope is omitted, EL will try to find the value from the smallest scope to the largest scope
* ${param.参数名}

### JSTL
* JSP Standard Tag Library, to simplify JSP
* put jar files in `/WEB-INF/lib`
  - taglibs-standard-spec-1.2.5.jar
  - taglibs-standard-impl-1.2.5.jar
  - taglibs-standard-jstlel-1.2.5.jar: to support EL, optional
  - taglibs-standard-compat-1.2.5.jar: to be compatible with version 1.0, optional
* libraries
  - core
    - <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

    - <c:if>
    - <c:choose>, <c:when>, <c:otherwise>
    - <c:forEach>
      ```
      <c:forEach var="p" items="${persons}" varStatus = "idx">
      No. ${idx.index + 1} <br/>
      name：${p.name} sex: ${p.sex} age:${p.age}
      </c:forEach>
      ```

      ```
      <%@ page language="java" contentType="text/html; charset=UTF-8"
          pageEncoding="UTF-8"%>
      <!-- 在Java或者JSP文件中输入 Alt + / 可出现智能提示 -->
      <%@ taglib  uri = "http://java.sun.com/jsp/jstl/core" prefix = "c" %>
      <!DOCTYPE html>
      <html>
      <head>
      <meta charset="UTF-8">
      <title>Insert title here</title>
      </head>
      <body>
      	<h1>${requestScope.score}</h1>
      	<c:if test = "${score >= 60 }">
      		<h1 style = "color:green">恭喜，你已通过测试</h1>
      	</c:if>
      	<c:if test = "${score < 60 }">
      		<h1 style = "color:red">对不起，再接再厉</h1>
      	</c:if>

      	<!-- choose when otherwise -->
      	${grade }
      	<c:choose>
      		<c:when test="${grade == 'A'}">
      			<h2>你很优秀</h2>
      		</c:when>
      		<c:when test="${grade == 'B' }">
      			<h2>不错呦</h2>
      		</c:when>
      		<c:when test="${grade == 'C' }">
      			<h2>水平一般，需要提高</h2>
      		</c:when>
      		<c:when test = "${grade == 'D'}">
      			<h2>需要努力啦，不要气馁</h2>
      		</c:when>
      		<c:otherwise>
      			<h2>一切随缘吧</h2>
      		</c:otherwise>
      	</c:choose>

      	<!-- forEach标签用于遍历集合
      		List companys = (List)request.getAttribute("companys")
      		for(Company c : companys){
      			out.print("...")
      		}
      		idx = index
      		idx.index属性代表循环的索引值（0开始）
      	-->
      	<c:forEach varStatus="idx" items = "${requestScope.companys }" var = "c">
      		<h2>${idx.index + 1}-${c.cname }-${c.url }</h2>
      	</c:forEach>

      </body>
      </html>
      ```
  - fmt
    - <fmt:formatDate value = "" pattern = "" />
    - <fmt:formatNumber value = "" pattern = ""/>
    ```
    <%@ page language="java" contentType="text/html; charset=UTF-8"
        pageEncoding="UTF-8"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    	<%
    		request.setAttribute("amt", 1987654.326);
    		request.setAttribute("now", new java.util.Date());
    		request.setAttribute("html", "<a href='index.html'>index</a>");
    		request.setAttribute("nothing", null);
    	%>
    	<h2>${now }</h2>
    	<!--
    		formatDate pattern
    		yyyy - 四位年
    		MM - 两位月
    		dd - 两位日
    		HH - 24小时制
    		hh - 12小时制
    		mm - 分钟
    		ss - 秒数
    		SSS - 毫秒
    	 -->
    	<h2>
    		<fmt:formatDate value="${requestScope.now }" pattern="yyyy年MM月dd日 HH时mm分ss秒 SSS毫秒" />
    	</h2>

    	<h2>${amt }</h2>
    	<h2>¥<fmt:formatNumber value = "${amt }" pattern="0,00.00"></fmt:formatNumber>元</h2>
    	<h2>null默认值：<c:out value="${nothing }" default="无"></c:out> </h2>
    	<h2><c:out value="${ html}" escapeXml="true"></c:out></h2>
    </body>
    </html>
    ```
  - sql
  - xml
  - functions

### Listener
* ServletContext - 对全局ServletContext及其属性进行监听
* HttpSession - 对用户会话及其属性操作进行监听
* ServletRequest - 对请求及属性操作进行监听
* ServletContextAttributeListener - 监听全局属性操作
* HttpSessionAttributeListener- 监听用户会话属性操作
* ServletRequestAttributeListener - 监听请求属性操作

#### Steps
* implements XxListener
* <listener> in web.xml or `@WebListener`
  ```
    <listener>
      <listener-class>com.ll.listener.FirstListener</listener-class>
    </listener>
  ```

## REST
* REST: architecture principles, resource's Representational State Transfer. 我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。客户端和服务器之间，传递这种资源的某种表现层。如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。
* RESTful: obeys the principles of REST
* URI: uniform resource identifier
* domain name and version
  - https://api.example.com/v1/
  - 如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下: https://example.org/api/v1/
* naming
  - URI必须具有语义: GET /a/1 -> GET /student/1
  - URI必须使用名词，所用的名词往往与数据库的表格名: POST /createArticle/1 -> POST /article/1
  - URI扁平化，不超两级: GET /articles/author/1 -> GET /articles/author?id=1
  - URI名词区分单复数: DELETE /articles/1 -> GET /articles?au=lily DELETE /article/1
  - filtering
    - ?limit=10：指定返回记录的数量
    - ?offset=10：指定返回记录的开始位置。
    - ?page=2&per_page=100：指定第几页，以及每页的记录数。
    - ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
    - ?animal_type_id=1：指定筛选条件
  - 不在URI中加入版本号，因为不同的版本，可以理解成同一种资源的不同表现形式。版本号可以在HTTP请求头信息的Accept字段中进行区分（参见Versioning REST Services）：`Accept: vnd.example-com.foo+json; version=1.0`
* annotation
  - `@RestController`
  - `@PathVariable`
  - `@GetMapping` and `@PostMapping`
### complex request
* simple request: get, post
* complex request: PUT/PATCH/DELETE、扩展标准请求;需要发送预检请求
GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。

### response
* GET /collection：返回资源对象的列表（数组）
* GET /collection/resource：返回单个资源对象
* POST /collection：返回新生成的资源对象
* PUT /collection/resource：返回完整的资源对象
* PATCH /collection/resource：返回完整的资源对象
* DELETE /collection/resource：返回一个空文档

### Hypermedia API
* 最好做到Hypermedia，即返回结果中提供链接，连向其他API方法
* 比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。
  ```
  {"link": {
    "rel":   "collection https://www.example.com/zoos",
    "href":  "https://api.example.com/zoos",
    "title": "List of zoos",
    "type":  "application/vnd.yourformat+json"
  }}
  ```

## 跨域
* 同源策略阻止从一个域加载的脚本去获取另一个域上的资源
* 只要协议、域名、端口有任何一个不同，都被当作是不同的域
* 浏览器Console看到Access-Control-Allow-Origin就代表跨域了
* HTML中允许跨域的标签: <img> <script> <link>
### Example
* http://ll.com http://abc.ll.com 不能
* http://localhost http://127.0.0.1 不能
### CORS
* CORS是一种机制,使用额外的HTTP头通知浏览器可以访问其他域
* URL响应头包含 Access-Control-*指明请求允许跨域
* `@CrossOrigin`
  ```
  @RestController
  @CrossOrigin(origins = "*",maxAge = 3600) // origin="*"代表允许所有域名都可访问该URL; maxAge=3600代表预检请求的缓存时间,单位:秒
  public class UserController {
  @GetMapping("/find_user")
    public User findUser(@RequestParam(value="id") Integer id){
    // codes
    }
  }
  ```
* <mvc:cors>: Spring MVC全局跨域配置
  ```XML
  <mvc:cors>
    <mvc:mapping path="/api/**"
      allowed-origins="http://domain1.com, http://domain2.com"
      allowed-methods="GET, POST"
      max-age="3600" />
    <mvc:mapping path="/resources/**" allowed-origins="*" />
  </mvc:cors>
  ```

## Freemarker
* ${属性名}
* ${属性名!默认值}
* ${属性名?string}: 格式化输出
* if
  ```
  <#if 条件1>
  条件1成立执行代码
  <#elseif 条件2>
  条件2成立执行代码
  <#elseif 条件3>
  条件3成立执行代码
  <#else>
  其他情况下执行代码

  <#switch value>
    <#case refValue1>
    ...
    <#break>
    <#case refValue2>
    ...
    <#break>
    <#case refValueN>
    ...
    <#break>
    <#default>
    ...
  </#switch>
  ```
* iteration
  ```
  <#list students as stu>
    <li>${stu_index}-${stu.name}</li>
  </#list>

  <#list map?keys as key>
    ${key}:${map[key]}
  </#list>

  <#list 1..20 as x>
    <li>${x}</li>
  </#list>
  ```
* functions
  - substring 截取字符串 "abcdefg"?substring(2,4)
  - cap_first 首字母大写 "jackson"?cap_first
  - index_of 查找字符索引 "abcdef"?index_of("b")
  - length 返回字符串长度 "abcdef"?length
  - round/floor/ceiling 四舍五入/下取整/上取整 pi?floor
  - size 得到集合元素总数 students?size
  - first/last 获取第一个/最后一个元素 student?first
  - sort_by 按某个属性对集合排序 list?sort_by("time")
* 与servlet整合
  - web.xml
    ```
      <servlet>
        <servlet-name>freemarker</servlet-name>
        <servlet-class>freemarker.ext.servlet.FreemarkerServlet</servlet-class>
        <init-param>
          <param-name>TemplatePath</param-name>
          <param-value>/WEB-INF/ftl</param-value>
        </init-param>
      </servlet>
      <servlet-mapping>
        <servlet-name>freemarker</servlet-name>
        <url-pattern>*.ftl</url-pattern>
      </servlet-mapping>
    ```
  - ftl
    ```

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width,initial-scale=1">
        <title></title>
        <link href="css/bootstrap.css" type="text/css" rel="stylesheet"></link>

        <script type="text/javascript" src="js/jquery-1.11.1.min.js"></script>
        <script type="text/javascript" src="js/bootstrap.js"></script>

        <style type="text/css">
            .pagination {
                margin: 0px
            }

            .pagination > li > a, .pagination > li > span {
                margin: 0 5px;
                border: 1px solid #dddddd;
            }

            .glyphicon {
                margin-right: 3px;
            }

            .form-control[readonly] {
                cursor: pointer;
                background-color: white;
            }
            #dlgPhoto .modal-body{
                text-align: center;
            }
            .preview{

                max-width: 500px;
            }
        </style>
    </head>
    <body>

    <div class="container">
        <div class="row">
            <h1 style="text-align: center">llԱ����Ϣ��</h1>
            <div class="panel panel-default">
                <div class="clearfix panel-heading ">
                    <div class="input-group" style="width: 500px;">

                    </div>
                </div>

                <table class="table table-bordered table-hover">
                    <thead>
                    <tr>
                        <th>Index</th>
                        <th>Employee Number</th>
                        <th>Name</th>
                        <th>Department</th>
                        <th>ְJob</th>
                        <th>Salary</th>
                        <th>&nbsp;</th>
                    </tr>
                    </thead>
                    <tbody>
                    <#list employee_list as emp>
                    <tr>
                        <td>${emp_index + 1}</td>
                        <td>${emp.empno?string("0")}</td>
                        <td>${emp.ename}</td>
                        <td>${emp.department}</td>
                        <td>${emp.job}</td>
                        <td style="color: red;font-weight: bold">��${emp.salary?string("0.00")}</td>

                    </tr>
    				</#list>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    </body>
    </html>
    ```
  - servlet
    ```
      import java.io.IOException;
      import java.util.ArrayList;
      import java.util.List;

      import javax.servlet.ServletException;
      import javax.servlet.annotation.WebServlet;
      import javax.servlet.http.HttpServlet;
      import javax.servlet.http.HttpServletRequest;
      import javax.servlet.http.HttpServletResponse;

      /**
       * Servlet implementation class ListServlet
       */
      @WebServlet("/list")
      public class ListServlet extends HttpServlet {
      	private static final long serialVersionUID = 1L;

          /**
           * @see HttpServlet#HttpServlet()
           */
          public ListServlet() {
              super();
              // TODO Auto-generated constructor stub
          }

      	/**
      	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
      	 */
      	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      		List list = new ArrayList();
      		list.add(new Employee(7731,"Liam" , "IT�" , "Engineer" , 8000f));
      		list.add(new Employee(8871,"Alex" , "Sales" , "Sales Representative" , 7000f));
      		request.getServletContext().setAttribute("employee_list", list);
      		request.getRequestDispatcher("/employee.ftl").forward(request, response);
      	}

      }

    ```

## 利用Apache Commons FileUpload组件实现上传功能
* form表单method="post"
* form表单enctype="multipart/form-data"
* form表单持有file类型input进行文件选择
```HTML
<%@page contentType="text/html;charset=utf-8"%>
<!-- 修改油画页面 -->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>作品更新</title>
<link rel="stylesheet" type="text/css" href="css\create.css">
<script type="text/javascript" src="js/jquery-3.4.1.min.js"></script>
<script type="text/javascript" src="js/validation.js"></script>
<script type="text/javascript">
	<!-- 提交前表单校验 -->
	function checkSubmit(){
		var result = true;
		var r1 = checkEmpty("#pname","#errPname");
		var r2 = checkCategory('#category', '#errCategory');
		var r3 = checkPrice('#price', '#errPrice');
		var r4 = checkFile('#painting', '#errPainting');
		var r5 = checkEmpty('#description', '#errDescription');
		if (r1 && r2 && r3 && r4 && r5){
			return true;
		}else{
			return false;
		}
	}
</script>
</head>
<body>
	<div class="container">
		<fieldset>
			<legend>作品名称</legend>
			<form action="[这里写更新URL]" method="post"
				autocomplete="off" enctype="multipart/form-data"
				onsubmit = "return checkSubmit()">
				<ul class="ulform">
					<li>
						<span>油画名称</span>
						<span id="errPname"></span>
						<input id="pname" name="pname" onblur="checkEmpty('#pname','#errPname')"/>
					</li>
					<li>
						<span>油画类型</span>
						<span id="errCategory"></span>
						<select id="category" name="category" onchange="checkCategory('#category','#errCategory')">
							<option value="-1">请选择油画类型</option>
							<option value="1">现实主义</option>
							<option value="2">抽象主义</option>
						</select>
					</li>
					<li>
						<span>油画价格</span>
						<span id="errPrice"></span>
						<input id="price" name="price" onblur="checkPrice('#price','#errPrice')"/>
					</li>
					<li>
						<span>作品预览</span><span id="errPainting"></span><br/>
						<img id="preview" src="/upload/1.jpg" style="width:361px;height:240px"/><br/>
						<input id="painting" name="painting" type="file" style="padding-left:0px;" accept="image/*"/>
					</li>

					<li>
						<span>详细描述</span>
						<span id="errDescription"></span>
						<textarea
							id="description" name="description"
							onblur="checkEmpty('#description','#errDescription')"
							></textarea>
					</li>
					<li style="text-align: center;">
						<button type="submit" class="btn-button">提交表单</button>
					</li>
				</ul>
			</form>
		</fieldset>
	</div>

</body>
</html>

```



```Java
package com.ll.mgallery.controller;

import java.io.File;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Date;
import java.util.List;
import java.util.UUID;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileItemFactory;
import org.apache.commons.fileupload.FileUploadException;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.apache.commons.fileupload.servlet.ServletFileUpload;

import com.ll.mgallery.entity.Painting;
import com.ll.mgallery.service.PaintingService;
import com.ll.mgallery.utils.PageModel;


@WebServlet("/management")
public class ManagementController extends HttpServlet {
	private PaintingService paintingService = new PaintingService();
	private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public ManagementController() {
        super();
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html;charset=utf-8");
		//通过method参数区分不同的操作
		String method = request.getParameter("method");
		if(method.equals("list")) {//分页查询列表
			this.list(request,response);
		} else if(method.equals("delete")) {
			this.delete(request,response);
		} else if(method.equals("show_create")) {//显示新增页面
			this.showCreatePage(request,response);
		} else if(method.equals("create")) {
			this.create(request, response);
		} else if(method.equals("show_update")) {
			this.showUpdatePage(request,response);
		} else if(method.equals("update")) {
			this.update(request,response);
		}
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}
	//分页查询列表
	private void list(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String p = request.getParameter("p");
		String r = request.getParameter("r");
		if(p == null) {
			p = "1";
		}
		if(r == null) {
			r = "6";
		}
		PageModel pageModel = paintingService.pagination(Integer.parseInt(p),Integer.parseInt(r));
		request.setAttribute("pageModel", pageModel);
		request.getRequestDispatcher("/WEB-INF/jsp/list.jsp").forward(request, response);
	}
	//显示新增页面
	private void showCreatePage(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		request.getRequestDispatcher("/WEB-INF/jsp/create.jsp").forward(request, response);		
	}
	//新增油画方法
	private void create(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//文件上传时的数据处理与标准表单完全不同
		/*
		String pname = request.getParameter("pname");
		System.out.println(pname);
		*/
		//1. 初始化FileUpload组件
		FileItemFactory factory = new DiskFileItemFactory();
		/**
		 * FileItemFactory 用于将前端表单的数据转换为一个个FileItem对象
		 * ServletFileUpload 则是为FileUpload组件提供Java web的Http请求解析
		 */
		ServletFileUpload sf = new ServletFileUpload(factory);
		//2. 遍历所有FileItem
		try {
			List<FileItem> formData = sf.parseRequest(request);
			Painting painting = new Painting();
			for(FileItem fi : formData) {
				if (fi.isFormField()) {
					System.out.println("普通输入项:" + fi.getFieldName() + ":" + fi.getString("UTF-8"));
					switch (fi.getFieldName()) {
					case "pname":
						painting.setPname(fi.getString("UTF-8"));
						break;
					case "category":
						painting.setCategory(Integer.parseInt(fi.getString("UTF-8")));
						break;
					case "price":
						painting.setPrice(Integer.parseInt(fi.getString("UTF-8")));
						break;
					case "description":
						painting.setDescription(fi.getString("UTF-8"));
						break;
					default:
						break;
					}
				} else {
					System.out.println("文件上传项:" + fi.getFieldName());
					//3.文件保存到服务器目录
					String path = request.getServletContext().getRealPath("/upload");
					System.out.println("上传文件目录:" + path);
					//String fileName = "test.jpg";
					String fileName = UUID.randomUUID().toString();
					//fi.getName()得到原始文件名,截取最后一个.后所有字符串,例如:wxml.jpg->.jpg
					String suffix = fi.getName().substring(fi.getName().lastIndexOf("."));
					//fi.write()写入目标文件
					fi.write(new File(path, fileName + suffix));
					painting.setPreview("/upload/" + fileName + suffix);
				}
			}
			paintingService.create(painting);//新增功能
			response.sendRedirect("/management?method=list");//返回列表页
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 显示更新页面
	 * @param request
	 * @param response
	 * @throws ServletException
	 * @throws IOException
	 */
	private void showUpdatePage(HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {
		String id = request.getParameter("id");
		Painting painting = paintingService.findById(Integer.parseInt(id));
		request.setAttribute("painting", painting);
		request.getRequestDispatcher("/WEB-INF/jsp/update.jsp").forward(request, response);
	}
	/**
	 * 实现油画更新
	 * @param request
	 * @param response
	 * @throws ServletException
	 * @throws IOException
	 */
	private void update(HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {
		int isPreviewModified = 0;
		//文件上传时的数据处理与标准表单完全不同
		/*
		String pname = request.getParameter("pname");
		System.out.println(pname);
		*/
		//1. 初始化FileUpload组件
		FileItemFactory factory = new DiskFileItemFactory();
		/**
		 * FileItemFactory 用于将前端表单的数据转换为一个个FileItem对象
		 * ServletFileUpload 则是为FileUpload组件提供Java web的Http请求解析
		 */
		ServletFileUpload sf = new ServletFileUpload(factory);
		//2. 遍历所有FileItem
		try {
			List<FileItem> formData = sf.parseRequest(request);
			Painting painting = new Painting();
			for(FileItem fi:formData) {
				if(fi.isFormField()) {
					System.out.println("普通输入项:" + fi.getFieldName() + ":" + fi.getString("UTF-8"));
					switch (fi.getFieldName()) {
					case "pname":
						painting.setPname(fi.getString("UTF-8"));
						break;
					case "category":
						painting.setCategory(Integer.parseInt(fi.getString("UTF-8")));
						break;
					case "price":
						painting.setPrice(Integer.parseInt(fi.getString("UTF-8")));
						break;
					case "description":
						painting.setDescription(fi.getString("UTF-8"));
						break;
					case "id":
						painting.setId(Integer.parseInt(fi.getString("UTF-8")));
						break;
					case "isPreviewModified":
						isPreviewModified = Integer.parseInt(fi.getString("UTF-8"));
						break;
					default:
						break;
					}
				}else {
					if(isPreviewModified == 1) {
						System.out.println("文件上传项:" + fi.getFieldName());
						//3.文件保存到服务器目录
						String path = request.getServletContext().getRealPath("/upload");
						System.out.println("上传文件目录:" + path);
						//String fileName = "test.jpg";
						String fileName = UUID.randomUUID().toString();
						//fi.getName()得到原始文件名,截取最后一个.后所有字符串,例如:wxml.jpg->.jpg
						String suffix = fi.getName().substring(fi.getName().lastIndexOf("."));
						//fi.write()写入目标文件
						fi.write(new File(path,fileName + suffix));
						painting.setPreview("/upload/" + fileName + suffix);
					}
				}
			}
			//更新数据的核心方法
			paintingService.update(painting, isPreviewModified);
			response.sendRedirect("/management?method=list");//返回列表页
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	/**
	 * 客户端采用Ajax方式提交Http请求
	 * Controller方法处理后不再跳转任何jsp,而是通过响应输出JSON格式字符串
	 * Tips:作为Ajax与服务器交互后,得到的不是整页HTML,而是服务器处理后的数据
	 * @throws IOException
	 */

	public void delete(HttpServletRequest request, HttpServletResponse response) throws IOException {
		String id = request.getParameter("id");
		PrintWriter out = response.getWriter();
		try {
			paintingService.delete(Integer.parseInt(id));
			//{"result":"ok"}
			out.println("{\"result\":\"ok\"}");
		}catch(Exception e) {
			e.printStackTrace();
			out.println("{\"result\":\"" + e.getMessage() + "\"}");
		}

	}

}

```



## Publish
  * export WAR
  * put WAR in `{TOMCAT_HOME}/webapps`
  * start: run bin/startup.bat
  * `{TOMCAT_HOME}/conf/server.xml`
  ```
  <Connector port="8080" protocol="HTTP/1.1"
             connectionTimeout="20000"
             redirectPort="8443" />
  <Context path="/"> <!-- so that we don't need to have the project name in the url -->
  ```

## Distributed
* 利用物理架构形成多个自治的处理元素，不共享主内存，但是通过发送信息合作。一个业务分拆多个子业务，部署在不同的服务器上。微服务是架构设计方式，分布式是系统部署方式。
### 单体应用问题
* 代码耦合严重
* 可用性低
### CAP
Consistency：读操作是否总能读到前一个写操作的结果
Availability：非故障节点应该在合理的时间内作出合理的响应（不是错误或超时的响应），但是可能不是最新的数据
Partition tolerance: 当出现网络分区现象后，系统能够继续“履行职责”，尽管任意数量的消息被节点间的网络丢失（或延迟），系统仍继续运行。
