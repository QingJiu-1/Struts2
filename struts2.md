## 开发环境的搭建
## action VS Action类
1、action：代表一个struts2的请求
2、Action类：能够处理Struts2请求的类
- 属性的名字必须遵守与JavaBeans属性名相同的命名规则。
- 属性的类型可以是任意类型，从字符串到非字符串（基本数据库类型）之间的数据转换可以自动发生
- 必须有一个不带参的构造器：必须通过反射创建实例
- 至少有一个供struts在执行这个action时调用的方法
- 同一个Action类可以含多个action方法
- Struts2会为每一个HTTP，请求创建一个新的Action实例，即Action不是单例的，是线程安全的

## 在Action中访问WEB资源
1、为什么访问WEB资源
B\S的应用的Controller中必然需要访问WEB资源
2、如何访问？
- 和Servlet API 解耦的方式：只能访问有限的Servlet API对象，且只能访问其有限的方法（读取请求参数，读写域对象的属性）
	- 使用ActionContext
	- 实现XxxxAware接口
- 和Servlet API 耦合的方式：可以访问更多的Servlet API 对象，且可以调用其原生的方法
	- 使用ServletActionContext
	- 实现ServletXxxAware接口
3、关于Struts2请求的扩展名问题
- org.apache.struts2包下的default.properties中配置了Struts2应用的一些常量
-  struts.action.extension定义了当前Struts2应用可以接受的请求的扩展名
- 可以在struts.xml文件中以常量配置的方式修改default.properties所配置的常量。
```XML
<constant name="struts.action.extension" value="action,do,"/>
```
4、ActionSupport
- ActionSupport是默认的Action类：若是某个Action节点配置没有class类属性，则ActionSupport即为待执行的Action类，而execute方法即为要默认执行的action方法

```XML
<action name="testActionSupport">
		<result>/testActionSupport.jsp</result>
</action>
```

- 等同于：

```XML
<action name="testActionSupport"
		class="com.opensymphony.xworrk2.ActionSupport" //该方法在的全类名
		method="execute">
		<result>/testActionSupport.jsp</result> //在根目录下的那个文件
</action>
```

- 在手工完成字段验证，显示错误消息，国际化等情况下，推荐使用ActionSupport。

5、result：
- result 是 action节点的子节点
- result 代表action 方法执行后，可能去的一个目的地
- 一个action 节点可以配置多个result子节点
- result 的 name 属性值对应的 action 方法可能有一个返回值
```XML
<action name="testActionSupport"
		class="com.opensymphony.xworrk2.ActionSupport" //该方法在的全类名
		method="execute">
		<result name="success">/success.jsp</result> //在根目录下的那个文件
		<result name="login">/login.jsp</result> //name中的值与.jsp文件名没有关系
		<result name="index">/index.jsp</result>
		<result name="test">/abcd.jsp</result>
</action>
```
- resulty一共有2个属性，还有一个是type：表示结果的响应类型
- result 的 type 属性值在 struts-default 包的 result-type 节点的 name 属性中定义：
	- 常用的有：
		- dispatcher（默认的）：转发
		- redirect ：重定向
		- redirectAction ：重定向到Action
		- chain: ：转发一个Action
```XML

<action name="testActionSupport"
		class="com.opensymphony.xworrk2.ActionSupport" //该方法在的全类名
		method="execute">
		
		<result name="success" type="dispatcher">/success.jsp</result> //在根目录下的那个文件
		
		<result name="login" type="redirectAction">/login.jsp</result> //name中的值与.jsp文件名没有关系
		
		<!-- 重定向到Action 第一种方法 -->
		<result name="index" type=“redirect”>
			<param name="actionName">testAction</param>
			<param name="namespace">/atguigu</param>
		</result>
		<!-- 重定向到Action 第二种方法 ：通过redirect 的响应类型也可以便捷的实现redirceAction的功能-->
		<result name="index" type=“redirect” >/atguig/testAction.do</result>
		
		<!-- chain不能通过type=dispatcher 的方式转发到一个Acation-->
		<result name="test" type="chain">
			<param name="actionName">testAction</param>
			<param name="namespace">/atguigu</param>
		</result>
</action>

<package name="testParam" namespace="/atguigu" extends="struts-default">
	<action name="testAction" class="该类所在的全类名路径">
		<result>/success.jsp</result>
	</action>
</package>
```


## 通配符映射
```XML
<action name="UserAction-save" class="该类的全类名"
		method="save">
		<result name="save-success">/success.jsp</result>
</action>

<action name="UserAction-update" class="该类的全类名"
		method="update">
		<result name="update-success">/success.jsp</result>
</action>

<action name="UserAction-delete" class="该类的全类名"
		method="delete">
		<result name="delete-success">/success.jsp</result>
</action>

<action name="UserAction-query" class="该类的全类名"
		method="query">
		<result name="query-success">/success.jsp</result>
</action>
```


```XML
<action name="UserAction-*" class="该类的全类名"
		method="{1}">
		<result name="{1}-success">/success.jsp</result>
</action>
```
规则：
- 若找到多个匹配，没有通配符的那个将胜出
- 若指定的动作不存在，Struts将会尝试把这个URL与任何一个包含着通配符 * 的动作名及进行匹配
- 被通配符匹配到的URI字符串的子串可以使用{1},{2}来引用。{1}匹配第一个字串，{2}匹配第二个字串....
- 若struts找到的带有通配符的匹配不止一个，则按先后顺序进行匹配
- * 可以匹配0个或多个字符，但不包括/字符，如果想把/字符包括在内可以使用** ,如果需要对某个字符进行转义，需要使用\

## 动态方法调用
- 动态方法调用：通过url动态调用Action方法
- URI:
  `/struts-app2/Prodouct!save.action:Struts`调用Product类的save()方法
- 默认情况下，Struts的动态方法调用处于禁用状态
```XML
<!--打开允许动态方法调用的开关,默认是false-->
<constant name="Struts.enable.DynamicMethodInvocation" value="true"></constant>
```


## OGNL值栈
值栈贯穿整个Action的生命周期，每个Action类的对象实例都拥有一个ValueStacke对象
1. 可以从ActionContext中获取值栈对象
2. 值栈分为两个逻辑部分：
	- Map栈：实际上是OgnlContext类型，是Map，也是对ActionContext的一个引用。里面保存各种map：requestMap,sessionMap,applicationMap,parametersMap,attr
	- 对象栈：实际上是CompoundRoot类型，是一个使用ArrayList定义的栈。里面保存各种和当前Action实例相关的对象。是一个数据结构意义的栈。

- 在JSP页面上可以利用OGNL访问到值栈里的对象属性
 - Struts2 利用s : property 标签和OGNL 表达式来读取值栈中的属性值
	 - 值栈中的属性值：
		 - 对于对象栈：对象栈中某一个对象的属性值
		 - Map 栈：request,session,appliction 的一个属性值 或 一个请求参数的值
	- 读取对象栈中对象的属性：
		- 若想访问Object Stack 里的某个对象的属性。可以使用以下几种形式之一：
		 object.propertyName object['propertyNmae']  objecy["propertyNmae"]
		- ObjectStack 里的对象可以通过一个从零开始的下标来引用。ObjectStack 里的栈顶对象可以用 [0] 来引用，它下面的那个对象可以用 [1] 引用。若希望返回栈顶对象的message属性值：【0】.message或【0】【'message'】或 【0】【"message"】
		- 若在指定的对象里面没有找到指定的属性，则到指定对象的下一个对象里面继续搜索，即【n】是的含义是从第n个开始搜索，而不是只搜索第n个对象
		- 若从栈顶对象开始搜索，则可以省略下表部分
		- 结合s : property 标签 ： <s:property value="[0].message"/>

- 使用 OGNL  调用 public 类的 public 类型的静态字段和静态方法
```jsp  
<s:property value="@java.lang.Math@PI"/>  
  
<!-- 注意的是在没有打开允许静态方法的调用的时候是无法使用的 -->  
<s:property value="@java.lang.Math@cos(0)"/>  
  
<!-- 调用对象栈的方法为一个属性赋值 -->  
<s:property value="setProductName('qingjiu')"/>  
  
<s:property value="productName"/>  
  
<!-- 调用数组对象的属性 -->  
<%  
    String [] names = new String[]{"aa","bb","cc","dd"};  
    request.setAttribute("names",names);  
%>  
  
length: <s:property value="#attr.names.lenth"/>  
  
names[2]: <s:property value="#attr.names[2]"/>  
  
<%  
    Map<String,String> letters = new HashMap<String,String>();  
    request.setAttribute("letters",letters);  
     
    letters.put("AA","a");  
    letters.put("BB","b");  
    letters.put("CC","c");  
%>  
<!-- 使用OGNL 访问 Map -->  
<s:property value="#attr.letters.size"/>  
  
AA: <s:property value="#atter.letters['AA']"/>  
```  
  
```XML  
<!-- 在struts2的配置文件中添加 -->  
<constant name="struts.ognl.allowStaticMethoAccess" value="true"></constant>  
```  
  
## 声明式异常处理  
```XML  
<package>  
    <action name="product-save"  
         class="com.atguigu.struts.valuestack.Product"  
         method="save">  
         <exception-mapping result="input" exception="java.lang.ArithmeticException"></exception-mapping>  
  
    <result name="input">/input.jsp</result>  
    </action>   
</package>  
```  
在跳转的页面上写上下面代码片段可显示异常
```JSP  
<s:property value="exceptionStack"/> <!-- 异常的堆栈 -->  
  
<s:property value="exception"/> - ${exception}  
  
<s:property value="exception.message"/> - ${exception.message}  
```

配置全局异常处理
```XML  
<global-results>  
    <result name="input">/input.jsp</result>  
</global-results>  
<global-exception-message>  
    <exception-message result="input" exception=java.lang.ArithmeticException">  
       </exception-mapping>  
</global-exception-message>  
```  
  
  
## 通用标签
Struts2 自动把Action 对象放入到值栈中
- 放入时间点为：
	Struts2 终将调用 Action 类的 Action 方法。
	但在调用该方法之前：    
	- 先创建一个StrutsActionProxy对象    
	- 在创建StrutsActionProxy之后，对其进行初始化时，把Action对象放入值栈中。
```Jsp
s:property: 打印值栈中的属性值的： 对于对象栈，打印值栈中对应的属性值
<br>
<s:property value="productName"/>
<br>
对于Map栈，打印request,session,application的某个属性值或某个请求参数的值
<s:property value="#session.date"/>
<br>
<s:property value="#parameters.name[0]"/>


<!--url标签用来动态创建一个URL -->
s:url: 创建一个URL 字符串的
<s:url value="/testUrl" var="url"></s:url>
${url} <!-- /struts2-4/testUrl -->

<br>
<s:url value="/getProduct" var="url">
	<s:param name="productId" value="1001"></s:param>
	<!-- 指定url包含的请求参数，2002不可能是一个属性名，struts2把2002直接作为属性值 -->
	<s:param name="productId" value="2002"></s:param>
</s:url>
${url} <!-- /struts2-4/getProduct?productId=1001 -->

<br>
<!-- 希望value的值来自值栈 -->
<s:url value="/getProduct" var="url2">
	<!-- 对于value 值会自动的进行OGNL 解析 -->
	<s:param name="productId" value="productId"></s:param>
</s:url>
${url2} <!-- /struts2-4/getProduct?productId=1001 -->

<br>
<s:url value="/getProduct" var="url3">
	<!-- 对于value 值会自动的进行OGNL 解析，若不希望进行OGNL 解析，则使用单引号引起来-->
	<s:param name="productId" value="'abcdef'"></s:param>
</s:url>
${url3} <!-- /struts2-4/getProduct?productId=abcdef -->

<br>
<!--构建一个请求action 的地址-->
<s:url action="testAction" namespace="/helloWorld" method="save" var="url4">
</s:url> <!-- /struts2-4/helloWorld/testAction!save.action -->
${url4}

<br>
<s:url value="testUrl" var="url5" includeParams="get"></s:url>
${url5<!-- testUrl?name=atguigu -->}
```

