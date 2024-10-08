## 开发环境的搭建
## action VS Action类
1、`action：代表一个struts2的请求
2、`Action类：能够处理Struts2请求的类
- `属性的名字必须遵守与JavaBeans属性名相同的命名规则。
- `属性的类型可以是任意类型，从字符串到非字符串（基本数据库类型）之间的数据转换可以自动发生
- `必须有一个不带参的构造器：必须通过反射创建实例
- `至少有一个供struts在执行这个action时调用的方法
- `同一个Action类可以含多个action方法
- `Struts2会为每一个HTTP，请求创建一个新的Action实例，即Action不是单例的，是线程安全的

## 在Action中访问WEB资源
1、`为什么访问WEB资源
	`B\S的应用的Controller中必然需要访问WEB资源
2、`如何访问？
- `和Servlet API 解耦的方式：只能访问有限的Servlet API对象，且只能访问其有限的方法（读取请求参数，读写域对象的属性）
	- `使用ActionContext
	- `实现XxxxAware接口
- `和Servlet API 耦合的方式：可以访问更多的Servlet API 对象，且可以调用其原生的方法
	- `使用ServletActionContext
	- `实现ServletXxxAware接口
3、`关于Struts2请求的扩展名问题
- `org.apache.struts2包下的default.properties中配置了Struts2应用的一些常量
-  `struts.action.extension定义了当前Struts2应用可以接受的请求的扩展名
- `可以在struts.xml文件中以常量配置的方式修改default.properties所配置的常量。
```XML
<constant name="struts.action.extension" value="action,do,"/>
```
4、`ActionSupport
- `ActionSupport是默认的Action类：若是某个Action节点配置没有class类属性，则ActionSupport即为待执行的Action类，而execute方法即为要默认执行的action方法

```XML
<action name="testActionSupport">
		<result>/testActionSupport.jsp</result>
</action>
```

- `等同于：

```XML
<action name="testActionSupport"
		class="com.opensymphony.xworrk2.ActionSupport" //该方法在的全类名
		method="execute">
		<result>/testActionSupport.jsp</result> //在根目录下的那个文件
</action>
```

- `在手工完成字段验证，显示错误消息，国际化等情况下，推荐使用ActionSupport。

5、`result：
- `result 是 action节点的子节点
- `result 代表action 方法执行后，可能去的一个目的地
- `一个action 节点可以配置多个result子节点
- `result 的 name 属性值对应的 action 方法可能有一个返回值
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
- `resulty一共有2个属性，还有一个是type：表示结果的响应类型
- `result 的 type 属性值在 struts-default 包的 result-type 节点的 name 属性中定义：
	- `常用的有：
		- `dispatcher（默认的）：转发
		- `redirect ：重定向
		- `redirectAction ：重定向到Action
		- `chain: ：转发一个Action
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
`规则：
- `若找到多个匹配，没有通配符的那个将胜出
- `若指定的动作不存在，Struts将会尝试把这个URL与任何一个包含着通配符 * 的动作名及进行匹配
- `被通配符匹配到的URI字符串的子串可以使用{1},{2}来引用。{1}匹配第一个字串，{2}匹配第二个字串....
- `若struts找到的带有通配符的匹配不止一个，则按先后顺序进行匹配
- `* 可以匹配0个或多个字符，但不包括/字符，如果想把/字符包括在内可以使用** ,如果需要对某个字符进行转义，需要使用\

## 动态方法调用
- `动态方法调用：通过url动态调用Action方法
- `URI:
  `/struts-app2/Prodouct!save.action:Struts`调用Product类的save()方法
- `默认情况下，Struts的动态方法调用处于禁用状态
```XML
<!--打开允许动态方法调用的开关,默认是false-->
<constant name="Struts.enable.DynamicMethodInvocation" value="true"></constant>
```


## OGNL值栈
`值栈贯穿整个Action的生命周期，每个Action类的对象实例都拥有一个ValueStacke对象
1. `可以从ActionContext中获取值栈对象
2. `值栈分为两个逻辑部分：
	- `Map栈：实际上是OgnlContext类型，是Map，也是对ActionContext的一个引用。里面保存各种map：requestMap,sessionMap,applicationMap,parametersMap,attr
	- `对象栈：实际上是CompoundRoot类型，是一个使用ArrayList定义的栈。里面保存各种和当前Action实例相关的对象。是一个数据结构意义的栈。

- `在JSP页面上可以利用OGNL访问到值栈里的对象属性
 - `Struts2 利用s : property 标签和OGNL 表达式来读取值栈中的属性值
	 - `值栈中的属性值：
		 - `对于对象栈：对象栈中某一个对象的属性值
		 - `Map 栈：request,session,appliction 的一个属性值 或 一个请求参数的值
	- `读取对象栈中对象的属性：
		- `若想访问Object Stack 里的某个对象的属性。可以使用以下几种形式之一：
		 `object.propertyName object['propertyNmae']  objecy["propertyNmae"]
		- `ObjectStack 里的对象可以通过一个从零开始的下标来引用。ObjectStack 里的栈顶对象可以用 [0] 来引用，它下面的那个对象可以用 [1] 引用。若希望返回栈顶对象的message属性值：【0】.message或【0】【'message'】或 【0】【"message"】
		- `若在指定的对象里面没有找到指定的属性，则到指定对象的下一个对象里面继续搜索，即【n】是的含义是从第n个开始搜索，而不是只搜索第n个对象
		- `若从栈顶对象开始搜索，则可以省略下表部分
		- `结合s : property 标签 ： <s:property value="[0].message"/>

- `使用 OGNL  调用 public 类的 public 类型的静态字段和静态方法
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
`在跳转的页面上写上下面代码片段可显示异常
```JSP  
<s:property value="exceptionStack"/> <!-- 异常的堆栈 -->  
  
<s:property value="exception"/> - ${exception}  
  
<s:property value="exception.message"/> - ${exception.message}  
```

`配置全局异常处理
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
`Struts2 自动把Action 对象放入到值栈中
- `放入时间点为：
	`Struts2 终将调用 Action 类的 Action 方法。
	`但在调用该方法之前：    
	- `先创建一个StrutsActionProxy对象    
	- `在创建StrutsActionProxy之后，对其进行初始化时，把Action对象放入值栈中。
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
	<s:param name="date" value="#session.date"></s:param>
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
${url5} <!-- testUrl?name=atguigu -->




<br>
s:set: 向page, request, session, application 域对象中加入一个属性值
<s:set name="productName" value="productName" scope="request"></s:set>
productName: ${requsetScope.productName} <!-- productName: CPU -->

<s:set name="productName" value="productName" scope="page"></s:set>
productName: ${pageScope.productName} <!-- productName: CPU -->

<br>
s:push : 把一个对象在标签开始后压入到栈中，标签结束时，弹出栈
<%
	Person person = new Person();
	person.setName("QingJiu");
	person.setAge(10);
	request.setAttribute("person",person);
%>
<s:push value="#request.person">
	${name} <!-- s:push : 把一个对象在标签开始后压入到栈中，标签结束时，弹出栈 QingJiu -->
</s:push>

<br>
--${name}-- <!-- ---- -->




<br>
s:if,s:else,s:elseif: 可以直接使用值栈中的属性
<br>
<s:if test="productPrice > 1000">
I7处理器
</s:if>
<s:elseif test="productPrice > 800">
I5处理器
</s:elseif>
<s:else test="productPrice > 800">
I3处理器
</s:else>

<br>
<s:if test="#request.person.age > 10">
	大于10岁
</s:if>
<s:else>
	小于或等于10岁
<s/else>
```

`作用域与隐含对象

1. **`request` 和 `requestScope`**:
    
    - `request` 是一个隐含对象，它表示当前 HTTP 请求对象 (`HttpServletRequest`)。所有与请求相关的数据都可以通过这个对象访问。
    - `requestScope` 是一个 `Map`，用于访问当前请求作用域中的属性。`requestScope` 相当于通过 `request.getAttribute()` 和 `request.setAttribute()` 来获取和设置请求属性。
2. **`page` 和 `pageScope`**:
    
    - `page` 是 JSP 的一个隐含对象，表示当前的 `Servlet` 实例。通常不直接用于存储和管理数据。
    - `pageScope` 是一个 `Map`，用于访问页面作用域中的属性。这个作用域的数据只能在当前 JSP 页面内访问。

 `什么时候使用 `requestScope` 或 `pageScope`？

-` 当你想在 JSP 页面中直接访问某个作用域内的属性时，可以使用 `requestScope` 或 `pageScope`
    - `使用 `requestScope` 可以确保你访问的是请求作用域中的属性，适用于在多个 JSP 页面间传递数据的情况。
    - `使用 `pageScope` 可以确保你访问的是当前页面作用域中的属性，适用于仅在当前页面有效的数据

`iterator标签：用来遍历一个数组，Collection或一个Map，并把这个可遍历对象里的每一个元素依次压入和弹出。
```jsp
s:iterator: 遍历集合。把这个可遍历对象里的每一个元素依次压入和弹出
<br>
<%
	List<Preson> persons = new ArrayList<Preson>();
	persons.add(new Person("AA",10));
	persons.add(new Person("BB",20));
	persons.add(new Person("CC",30));
	persons.add(new Person("DD",40));
	persons.add(new Person("EE",50));
	
	request.setAttribute("persons",persons);
%>

<s:iterator value="#request.persons">
	${name} - ${age}<br> //注意与s:push一样只在标签内部有用，结束后弹出
</s:iterator>


<!-- 在值栈的对象里面有个persons属性 -->
<s:iterator value="persons">
	${name} - ${age} <br>
</s:iterator>

<s:iterator value="#request.persons" status="status">
	index: ${status.index}-count: ${status.count}: ${name} - ${age}<br>
	<!-- index:0-count 1: AA-10 排序下去 -->
</s:iterator>

```

`sort标签：用来对一个可遍历对象里的元素进行排序
```jsp
s:sort 可以对集合中的元素进行排序
<br>
<%
	PersonComparator pc = new PersonComparator();
	request.setAttribute("comparator",pc);
%>

<s:sort comparator="#request.comparator" source="persons" var="persons2">
</s:sort>
<s:iterator value="#attr.persons2">
	${name} - ${age} <br>
</s:iterator>
```

`date标签：date标签用来对Date对象进行排版
```jsp
s:date 可以对Date进行排版
<br>
<s:date name="#session.date" format="yyyy-MM-dd hh:mm:ss" var="date2" />
date:${date2}
```

`a标签：呈现为一个HTML连接。这个标签可以接受HTML语言中的a元素所能接受的所有属性
```jsp
<S:iteartor value="persons">
	<!-- 可以使用%{}把属性包装起来，使其进行强制的OGNL解析-->
	<s:a href="getPerson.action?name=%{name}">${name}</s:a>
</s:iteartor>
```


## 表单标签
`表单标签将在HTML文档里被呈现为一个表单元素`
`使用表单标签的优点：`
	`表单回显`
	`对页面进行布局和排版`
`标签的属性可以被赋值为一个静态的值或一个OGNL表达式。如果在赋值时使用了一个OGNL表达式并把它用%{}括起来，这个表达式将会被求值`

`form标签：`
```jsp
<!-- 
	表单标签：
	1、使用和html的form标签的感觉差不多
	2、Struts2的form标签会生成一个table，以进行自动的排版
	3、可以对表但提交的值进行回显：从栈顶对象开始匹配属性，并把匹配的属性值赋值道对应的标签value中，若栈顶对象没有对应的属性，则依次向下找相对应的属性。
-->
<s:form action="save">
	<s:hidden name="userId"></s:hidden>
	<s:textfield name="userName" label="UserName"></s:textfield>
	<s:password name="passWord" label="PassWord" showPassword="true"></s:password>
	<s:textarea name="desc" label="Desc"></s:textarea>
	<s:submit></submit>
</s:form>
```

`基础方法
```Java
package com.qingjiu.springboot3reactor.Java;  
  
import java.util.Arrays;  
import java.util.List;  
  
public class UserAction {  
  
    String userId;  
    String userName;  
    String passWord;  
    String desc;  
  
    boolean married;  
  
    String gender;  
  
    List<String> city;  
  
    String age;  
  
    public boolean isMarried() {  
        return married;  
    }  
  
    public void setMarried(boolean married) {  
        this.married = married;  
    }  
  
    public String getUserId() {  
        return userId;  
    }  
  
    public void setUserId(String userId) {  
        this.userId = userId;  
    }  
  
    public String getUserName() {  
        return userName;  
    }  
  
    public void setUserName(String userName) {  
        this.userName = userName;  
    }  
  
    public String getPassWord() {  
        return passWord;  
    }  
  
    public void setPassWord(String passWord) {  
        this.passWord = passWord;  
    }  
  
    public String getDesc() {  
        return desc;  
    }  
  
    public void setDesc(String desc) {  
        this.desc = desc;  
    }  
  
    public String getGender() {  
        return gender;  
    }  
  
    public void setGender(String gender) {  
        this.gender = gender;  
    }  
  
    public List<String> getCity() {  
        return city;  
    }  
  
    public void setCity(List<String> city) {  
        this.city = city;  
    }  
  
    public String getAge() {  
        return age;  
    }  
  
    public void setAge(String age) {  
        this.age = age;  
    }  
  
    public String save(){  
        System.out.println(this);  
  
        UserAction ua = new UserAction();  
        ua.setUserId("1001");  
        ua.setUserName("QINGJIU");  
        ua.setPassWord("123456");  
        ua.setDesc("Oracle!");  
  
        /*ActionContext.getContext().getValueStack.push(ua);*/  
  
        return "input";  
    }  
  
    @Override  
    public String toString() {  
        return "UserAction{" +  
                "userId='" + userId + '\'' +  
                ", userName='" + userName + '\'' +  
                ", passWord='" + passWord + '\'' +  
                ", desc='" + desc + '\'' +  
                ", married=" + married +  
                ", gender='" + gender + '\'' +  
                ", city=" + city +  
                ", age='" + age + '\'' +  
                '}';  
    }  
  
}
```

```XML
<action name="save" class="com.qingjiu.springboot3reactor.Java.UserAction"
		method="save">
		<result name="input">/form-tag.jsp</result>
</action>
```

`checkbox标签：呈现为一个HTML复选框元素，该复选框元素通常用于提交一个布尔值`
```jsp
<s:form action="save">
	<s:hidden name="userId"></s:hidden>
	<s:textfield name="userName" label="UserName"></s:textfield>
	<s:password name="passWord" label="PassWord" showPassword="true"></s:password>
	<s:textarea name="desc" label="Desc"></s:textarea>
	<s:checkbox name="married" label="Married"></s:checkbox>
	<s:submit></submit>
</s:form>

<!--一般的表单-->
<br>
<form action="save" method="post">
	Married:<input type="checkbox" name="married"/>
	<input type="submit" value="Submit"/>
</form>

```

```Java
package com.qingjiu.springboot3reactor.Java;  
  
public class UserAction {  
  
    String userId;  
    String userName;  
    String passWord;  
    String desc;  
    boolean married;  
    public boolean isMarried() {  
        return married;  
    }  
  
    public void setMarried(boolean married) {  
        this.married = married;  
    }  
  
    public String getUserId() {  
        return userId;  
    }  
  
    public void setUserId(String userId) {  
        this.userId = userId;  
    }  
  
    public String getUserName() {  
        return userName;  
    }  
  
    public void setUserName(String userName) {  
        this.userName = userName;  
    }  
  
    public String getPassWord() {  
        return passWord;  
    }  
  
    public void setPassWord(String passWord) {  
        this.passWord = passWord;  
    }  
  
    public String getDesc() {  
        return desc;  
    }  
  
    public void setDesc(String desc) {  
        this.desc = desc;  
    }  
  
    public String save(){  
        System.out.println(this);  
  
        UserAction ua = new UserAction();  
        ua.setUserId("1001");  
        ua.setUserName("QINGJIU");  
        ua.setPassWord("123456");  
        ua.setDesc("Oracle!");  
        ActionContext.getContext().getValueStack.push(ua);  
  
        return "input";  
    }  
  
    @Override  
    public String toString() {  
        return "UserAction{" +  
                "userId='" + userId + '\'' +  
                ", userName='" + userName + '\'' +  
                ", passWord='" + passWord + '\'' +  
                ", desc='" + desc + '\'' +  
                ", married=" + married +  
                '}';  
    }  
  
}
```

`注意的是s:form中的input和form表单里的input有所不同，它是两部分组成`
`当包含着一个复选框的表单被提交时，如果某个复选框被选中了，它的值将为true，这个复选框在HTTP请求里增加一个请求参数。但如果该复选框未被选中，在请求中就不会增加一个请求参数。ckeckbox标签解决了这个局限性，它采取的办法时为单个复选框元素创建一个配对的不可见字段`

```jsp
<input type="checkbox" name="married" value="true" checked="checked" id="save_married"/>
<input type="hidden" id="__checkbox_save_married" name="__checkbox_married" value="true" />
```

`list,listKey和listValue这3个属性对radio,select,checkboxlist等标签非常重要`
```Java
package com.qingjiu.springboot3reactor.Java;  
  
public class City {  
   
    Integer cityId;  
    String cityName;  
  
    public Integer getCityId() {  
        return cityId;  
    }  
  
    public void setCityId(Integer cityId) {  
        this.cityId = cityId;  
    }  
  
    public String getCityName() {  
        return cityName;  
    }  
  
    public void setCityName(String cityName) {  
        this.cityName = cityName;  
    }  
      
    public City(){  
          
    }  
  
    @Override  
    public String toString() {  
        return "City{" +  
                "cityId=" + cityId +  
                ", cityName='" + cityName + '\'' +  
                '}';  
    }  
}
```

```jsp
<!--
	服务端需要使用集合类型，以保证能够被正常回显
-->
<%
	List<City> cities = new ArrayList<City>();
	cities.add(new City(10001,"AA"));
	cities.add(new City(10002,"BB"));
	cities.add(new City(10003,"CC"));
	cities.add(new City(10004,"DD"));
	request.setAttribute("cities",cities);
%>
<s:radio name="gender" list="#{'1':'Male','0':'Female'}" label="Gender"></s:radio>
<s:checkboxlist list="#request.cities" listKey="cityId" listValue="cityName"
				label="City" name="city"></s:checkboxlist>
```

`select标签:
```jsp
<s:select list="{11,12,13,14,15,16,17,18,19,20}"
		  headerKey=""
		  headerValue="请选择"
		  name="age"
		  label="Age">
		  <!-- 
			  s:optgroup 可以用作s:select 的子标签，用于显示更多的下拉框
			  注意：必须有键值对，而不能使用一个集合，让其值即作为键，又作为值
		   -->
		  <s:optgroup label="21-30" list="#{21:21}"></s:optgroup>
		  <s:optgroup label="31-40" list="#{31:31}"></s:optgroup>
</s:select>
```

## 示例代码
```Java
public class Role {

    private Integer roleId;
    private String roleName;

    // Getter for roleId
    public Integer getRoleId() {
        return roleId;
    }

    // Setter for roleId
    public void setRoleId(Integer roleId) {
        this.roleId = roleId;
    }

    // Getter for roleName
    public String getRoleName() {
        return roleName;
    }

    // Setter for roleName
    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }
	
	public Role(){
	
	}
	
	public Role(Integer roleId,String roleName){
		super();
		this.roleId = roleId;
		this.roleNmae = roleName;
	}

    // toString method
    @Override
    public String toString() {
        return "Role{" +
                "roleId=" + roleId +
                ", roleName='" + roleName + '\'' +
                '}';
    }
}

```

```Java
public class Department {
    private Integer deptId;
    private String deptName;

    // 无参构造方法
    public Department() {
    }

    // 有参构造方法
    public Department(Integer deptId, String deptName) {
        this.deptId = deptId;
        this.deptName = deptName;
    }

    // Getter for deptId
    public Integer getDeptId() {
        return deptId;
    }

    // Setter for deptId
    public void setDeptId(Integer deptId) {
        this.deptId = deptId;
    }

    // Getter for deptName
    public String getDeptName() {
        return deptName;
    }

    // Setter for deptName
    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    // toString method
    @Override
    public String toString() {
        return "Department{" +
                "deptId=" + deptId +
                ", deptName='" + deptName + '\'' +
                '}';
    }
}

```

```Java
public class Dao{
	public List<Department> getDepartments(){
		
		List<Department> depts = new ArrayList<>();
		depts.add(new Departemnt(1001,"AAA"));
		depts.add(new Departemnt(1002,"BBB"));
		depts.add(new Departemnt(1003,"CCC"));
		depts.add(new Departemnt(1004,"DDD"));
		depts.add(new Departemnt(1005,"EEE"));
		
		return depts;
	}
	
	public List<Role> getROles(){
		
		List<Role> roles = new ArrayList<>();
		roles.add(new Role(2001,"XXX"));
		roles.add(new Role(2001,"YYY"));
		roles.add(new Role(2001,"ZZZ"));
		
		return null;
	}
}
```

```Java
public class Employee implements RequestAware{

	private Map<String,Object> requestMap = null;
	private Dao dap = new Dao();
	
	private String name;
	private String password;
	
	private String gender;
	private String dept;
	
	private List<String> roles;
	private String desc;
	
	public String save(){
		System.out.println("save：" + this);
		return "save";
	}

	public String input(){
		requestMap.put("depts",dao.getDepartments());
		requestMap.put("roles",dao.getRoles());
		return "input"; //返回input页面
	}
	@Override 
	public void setRequest(Map<String, Object> request) { 
		this.requestMap = request; 
	}
	@Override public String toString() { 
		return "Employee{" + "name='" + name + '\'' + ", password='" + password + '\'' + ", gender='" + gender + '\'' + ", dept='" + dept + '\'' + ", roles=" + roles + ", desc='" + desc + '\'' + '}'; 
		}
}
```

`index.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
</head>
<body>
	<a href="emp-input.action">Emp Input Page</a>
</body>
</html>

```
`struts.xml配置
```xml
<struts>
    <!-- 定义一个名为 'default' 的包 -->
    <package name="default" namespace="/" extends="struts-default">
        
        <!-- 配置访问 index.jsp 的 action -->
        <action name="emp-*" class="com.example.EmployeeAction" method="{1}">
            <result name="{1}">/emp-{1}.jsp</result>
        </action>
    </package>
</struts>

```

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Employee Input Page</title>
</head>
<body>

	<s:form action="emp-save">
		<s:textfield name="name" label="Name"></s:textfield>
		<s:password name="password" label="Password"></s:password>
		
		<s:radio name="gender" list="#{'1':'Male','0':'Female'}" label="Gender"></s:radio>
		<s:select list="#request.depts" listKey="deptId" listValue="deptName" name="dept" label="Department"></s:select>
		<s:checkboxlist list="#request.roles" listKey="roleId" listValue="roleName" name="roles" label="Role"></s:checkboxlist>
		<s:textarea name="desc" label="Desc"></s:textarea>
		<s:submit></s:submit>
	</s:form>

</body>
</html>

```

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Save Result</title>
</head>
<body>
    Name；${name}
    <br>
    Password；${password}
    <br>
    Gender；${gender}
    <br>
    Dept；${dept}
    <br>
    Roles；${roles}
    <br>
    Desc；${desc}
    <br>
</body>
</html>

```

## 主题
`默认情况下，form标签将呈现为一个HTML form元素和一个table元素`
`修改主题：`
- `通过UI标签的theme属性`
- `在一个表单里，若没有给出某个UI标签的theme属性，它将使用这个表单的主题`
- `page,request,session或appliction中添加一个theme属性
- `修改struts.properties文件中的struts.ui.theme属性

## CRUD操作
`在web.xml配置struts2`
```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <!-- Optional init parameters can be configured here -->
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```
`struts2的配置文件`
```xml
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        
    </package>
</struts>

```

`新建index.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <a href="emp-list">List All Employees</a>
    
</body>
</html>

```

```jsp
<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="emp-*" 
	        class="com.QingJiu.struts2.app.EmployeeAction"
	        method="{1}">
		    <result name="{1}">/emp-{1}.jsp</result>
	    </action>
    </package>
</struts>
```

```Java
public class EmployeeAction{
	public String list(){
		return "list";
	}
}
```

```Java
public class Employee {
    private Integer employeeId;
    private String firstName;
    private String lastName;
    private String email;

    // No-argument constructor
    public Employee() {
    }

    // Parameterized constructor
    public Employee(Integer employeeId, String firstName, String lastName, String email) {
	    super();
        this.employeeId = employeeId;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }

    // Getter and Setter for employeeId
    public Integer getEmployeeId() {
        return employeeId;
    }

    public void setEmployeeId(Integer employeeId) {
        this.employeeId = employeeId;
    }

    // Getter and Setter for firstName
    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    // Getter and Setter for lastName
    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    // Getter and Setter for email
    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    // toString method
    @Override
    public String toString() {
        return "Employee{" +
                "employeeId=" + employeeId +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}


```

```Java
public class Dao{

	private static Map<Integer,Employee> emps = new LinkedHashMap<Integer,Employee>();
	
	static{
		emps.put(1001,new Employee(1001,"AA","aa","aa@qq.com"));
		emps.put(1002,new Employee(1002,"BB","bb","bb@qq.com"));
		emps.put(1003,new Employee(1003,"CC","cc","cc@qq.com"));
		emps.put(1004,new Employee(1004,"DD","dd","dd@qq.com"));
		emps.put(1005,new Employee(1005,"EE","ee","ee@qq.com"));
	}
	
	public List<Employee> getEmployees(){
		return new ArrayList<>(emp.values());
	}
	
	public void delete(Integer empId){
		emps.remove(empId);
	}
	
	public void savr(Employee emp){
		long time = System.currentTimeMillis();
		emp.setEmployeeId((int)time);
		emps.put(emp.getEmployeeId(),emp)
	}
	
	public Employee get(Integer empId){
		return emps.get(empId);
	}
	
	public void update(Employee emp){
		emps.put(emp.getEmployeeId(),emp);
	}

}
```

`为EmployeeAction添加`
```Java
public class EmployeeAction implements RequsetAware{
	private Dao dao = new Dao();
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	private Map<String,Object> request;
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
}
```

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Employee List</title>
</head>
<body>
    <!-- Employee list content goes here -->
    <table cellpadding="10" cellspacing="0" border="1">
	    <thead>
		    <tr>
			    <td>ID</td>
			    <td>FirstName</td>
			    <td>ListName</td>
			    <td>Email</td>
			    <td>Edit</td>
			    <td>Delete</td>
		    </tr>
	    </thead>
	    <tbody>
		    <s:iterator value="#request.emps">
			    <tr>
				    <td>${employeeId}</td>
				    <td>${firstName}</td>
				    <td>${lastName}</td>
				    <td>${email}</td>
				    <td><a href="">Edit</a></td>
				    <td><a href="emp-delete?employeeId=${employeeId}">Delete</a></td>
			    </tr>
		    </s:iterator>
	    <tbody>
    </table>
</body>
</html>

```

`完善方法`
```java
public class EmployeeAction implements RequsetAware{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	//需要在当前的EmployeeAction中定义 employeeId属性
	//以接收请求参数
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String delete(){
		dao.delete(employeeId);
		//返回结果的类型应为：redirectAction
		//也可以时chain:实际上chain时没有必要的。因为不需在下一个Action中保留当前Action的状态；还有，若使用chain，则到达目标页面后，地址栏显示的依然时删除的那个链接，刷新时会有重复提交。
		return "delete";
	}
	
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
}
```
`为删除添加精确匹配`
```jsp
<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="emp-*" 
	        class="com.QingJiu.struts2.app.EmployeeAction"
	        method="{1}">
		    <result name="{1}">/emp-{1}.jsp</result>
		    <result name="delete" type="redirectAction">emp-list</result>
	    </action>
    </package>
</struts>
```

`Params拦截器
`Parameters拦截器将把表单字段映射到ValueStack栈的栈顶对象的各个属性中。如果，某个字段在模型里面没有匹配的属性，Param拦截器将尝试ValueStack栈中的下一个对象。

`添加操作
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Employee List</title>
</head>
<body>

	<s:form action="emp-save">
		
		<s:textfield name="firstName" label="FirstName"></s:textfield>
		<s:textfield name="lastName" label="ListName"></s:textfield>
		<s:textfield name="email" label="Email"></s:textfield>
		
		<s:submit></s:submit>
	</s:form>

	<br>
	<hr>
	<br>
    <!-- Employee list content goes here -->
    <table cellpadding="10" cellspacing="0" border="1">
	    <thead>
		    <tr>
			    <td>ID</td>
			    <td>FirstName</td>
			    <td>ListName</td>
			    <td>Email</td>
			    <td>Edit</td>
			    <td>Delete</td>
		    </tr>
	    </thead>
	    <tbody>
		    <s:iterator value="#request.emps">
			    <tr>
				    <td>${employeeId}</td>
				    <td>${firstName}</td>
				    <td>${lastName}</td>
				    <td>${email}</td>
				    <td><a href="emp-edit?employeeId=${employeeId}">Edit</a></td>
				    <td><a href="emp-delete?employeeId=${employeeId}">Delete</a></td>
			    </tr>
		    </s:iterator>
	    <tbody>
    </table>
</body>
</html>
```

```Java
public class EmployeeAction implements RequsetAware{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	//需要在当前的EmployeeAction中定义 employeeId属性
	//以接收请求参数
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		//1、获取请求参数：通过定义对应属性的方式
		//2、调用Dao的save方法
		Employee employee = new Employee(null,firstName,listName,email);
		dao.save(employee);
		//3、通过redirectAction的方式响应结果给emp-list
		//return "save";
		return "success";
	}
	
	public String delete(){
		dao.delete(employeeId);
		//返回结果的类型应为：redirectAction
		//也可以时chain:实际上chain时没有必要的。因为不需在下一个Action中保留当前Action的状态；还有，若使用chain，则到达目标页面后，地址栏显示的依然时删除的那个链接，刷新时会有重复提交。
		//return "delete";
		return "success";
	}
	
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
}
```

```xml
<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="emp-*" 
	        class="com.QingJiu.struts2.app.EmployeeAction"
	        method="{1}">
		    <result name="{1}">/emp-{1}.jsp</result>
		    <!--因为多次使用所以改为success-->
		    <result name="success" type="redirectAction">emp-list</result>
	    </action>
    </package>
</struts>
```

`代码成功实现但太过重复，对其简化`
```java
public class EmployeeAction implements RequsetAware{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	private Employee employee;
	
	//需要在当前的EmployeeAction中定义 employeeId属性
	//以接收请求参数
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		//1、获取请求参数：通过定义对应属性的方式
		//2、调用Dao的save方法
		dao.save(employee);
		//3、通过redirectAction的方式响应结果给emp-list
		//return "save";
		return "success";
	}
	
	public String delete(){
		dao.delete(employee.getEmployeeId());
		//返回结果的类型应为：redirectAction
		//也可以时chain:实际上chain时没有必要的。因为不需在下一个Action中保留当前Action的状态；还有，若使用chain，则到达目标页面后，地址栏显示的依然时删除的那个链接，刷新时会有重复提交。
		//return "delete";
		return "success";
	}
	
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
}
```

`要实现功能则需要，把 Action 和 Model隔开：`
`  在使用Struts作为前端的企业级应用程序时把Action和Model清晰地隔离开是有必要的：有些Action类不代表任何Model对象，它们的功能仅限于提供显示服务。`

`如果Action类实现了ModelDriven接口，该拦截器将把ModelDriven接口的getModel()方法返回的对象治愈栈顶`

`于是对其改进`
```java
public class EmployeeAction implements RequsetAware , ModelDriven<Employee>{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	private Employee employee;
	
	//需要在当前的EmployeeAction中定义 employeeId属性
	//以接收请求参数
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		//1、获取请求参数：通过定义对应属性的方式
		//2、调用Dao的save方法
		dao.save(employee);
		//3、通过redirectAction的方式响应结果给emp-list
		//return "save";
		return "success";
	}
	
	public String delete(){
		dao.delete(employee.getEmployeeId());
		//返回结果的类型应为：redirectAction
		//也可以时chain:实际上chain时没有必要的。因为不需在下一个Action中保留当前Action的状态；还有，若使用chain，则到达目标页面后，地址栏显示的依然时删除的那个链接，刷新时会有重复提交。
		//return "delete";
		return "success";
	}
	
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
	
	@Override 
	public Employee getModel() { 
		employee = new Employee();
		return employee;
	}
}
```

`Action 实现ModelDriven接口后的运行流程
1、先会执行ModelDrivenInterceptor的intercept方法。
```java
package org.apache.struts2.interceptor;

import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.interceptor.AbstractInterceptor;
import com.opensymphony.xwork2.ModelDriven;
import com.opensymphony.xwork2.util.ValueStack;

public class ModelDrivenInterceptor extends AbstractInterceptor {

    @Override
    public String intercept(ActionInvocation invocation) throws Exception {
        // 获取Action对象：例如 EmployeeAction 对象，此时 EmployeeAction 已经实现了 ModelDriven 接口 
        // public class EmployeeAction implements RequestAware, ModelDriven<Employee>
        Object action = invocation.getAction();

        // 判断 action 是否实现了 ModelDriven 接口
        if (action instanceof ModelDriven) {
            // 强制转换为 ModelDriven 类型
            ModelDriven<?> modelDriven = (ModelDriven<?>) action;

			// 获取当前的值栈
	        ValueStack stack = invocation.getStack();
            
            //调用ModelDriven 接口的getModel()方法
            //即调用EmployeeAction的getModel()方法
            /**
	        @Override 
			public Employee getModel() { 
				employee = new Employee();
				return employee;
			}    
            */
            Object model = modelDriven.getModel();

            // 检查模型对象是否为空，避免空指针异常
            if (model != null) {
                // 将模型对象推入值栈的顶部，实际压入的是EmployeeAction的employee成员变量
                stack.push(model);
            } else {
                // 如果模型对象为 null，记录警告或处理异常（根据需要）
                System.out.println("ModelDrivenInterceptor: model is null!");
            }
        }

        // 调用下一个拦截器或目标 Action
        return invocation.invoke();
    }
}


```

2、`执行ParametersInterceptor的intercept方法：把请求参数的值赋给栈顶对象对应的属性。若栈顶对象没有对应的属性，则查询值栈中下一个对象对应的属性....

3、`注意：getModel方法不能提供以下实现.的确会返回一个Employee对象到值栈的栈顶。但当前Action的employee成员变量是null。
```java
@Override 
public Employee getModel() { 
	return new Employee();
}
```

`更改内容：`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>emp-edit</title>
</head>
<body>
    <s:form action="emp-update">
		<s:hidden name="employeeId" label=""></s:hidden>
		<s:textfield name="firstName" label="FirstName"></s:textfield>
		<s:textfield name="lastName" label="ListName"></s:textfield>
		<s:textfield name="email" label="Email"></s:textfield>
		
		<s:submit></s:submit>
	</s:form>
</body>
</html>
```
`
`编写edit的方法：在EmployeeAction中添加`
```Java
public String edit(){
	//1、获取传入的employeeId:employee.getEmployeeId;
	//2、根据employeeId获取Employee对象
	Employee emp = dao.get(employee.getEmployeeId());
	//3、把栈顶对象属性装配好：此时栈顶对象是employee
	//目前的employee对象才new出来，因此目前只有employeeId属性，其他属性为null
	/**
	*struts2表单回显时：从值栈栈顶开始查找匹配的属性，若找到就添加到value属性中。
	*/
	employee.setEmail(emp.getEmail());
	employee.setFirstName(emp.getFirstName());
	employee.setLastName(emp.getLastName());
	
	
	return "edit";
	
}
```

![[Pasted image 20240831100513.png]]

`错误演示：`
```Java
public String edit(){
	//不能够进行表单的回显，因为经过重写赋值的employee对象已经不再是栈顶对象了。
	employee = dao.get(emplotee.getEmployeeId());
	
	return "edit";
	
}
```
`原理解析图`
![[Pasted image 20240831104103.png]]
`方法的改进`
```Java
public String edit(){
//手动的把从数据库中获取的Employee对象放到值栈的栈顶
//但此时值栈栈顶及第二个对象均为Employee对象，不够完美
//ActionContext.getContext().getValueStack().push(dao.get(emplotee.getEmployeeId()));

	return "edit";
}
```
`给EmployeeAction类中的getModle（）放添加判断`
```Java
//为其方便添加一个setEmployeeId方法
public void setEmployeeId(Integer employeeId){

	this.employeeId = employeeId;

}

@Override 
	public Employee getModel() { 
		//判断Create 还是Edit
		//若为Create,则employee = new Employee();
		//若为Edit,则employee = dao.get(employeeId);
		//可以通过判定employeeId是否为空，来决定哪个为Create哪个是Edit;若通过employeeId来判断，则需要在modelDriven拦截器之前先执行一个params拦截器
		//而这可以通过使用paramsParams拦截器栈实现
		//需要在struts.xml文件中配置使用paramsPrepareParams作为默认的拦截器栈。
		//添加是：没有employeeId这个请求参数
		if(employeeId == null || employeeId.equals("")){
			employee = new Employee();
		}else{ //回显页面时：有employeeId这个请求参数
			employee = dao.get(emplotee.getEmployeeId());
		}
		return employee;
	}
```
`在struts的配置文件中添加新的配置：`
```xml
<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
	    <!--配置使用paramsPrepareParams作为默认的拦截器栈-->
	    <default-interceptor-ref name="paramsPrepareParams"></default-interceptor-ref>
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="emp-*" 
	        class="com.QingJiu.struts2.app.EmployeeAction"
	        method="{1}">
		    <result name="{1}">/emp-{1}.jsp</result>
		    <!--因为多次使用所以改为success-->
		    <result name="success" type="redirectAction">emp-list</result>
	    </action>
    </package>
</struts>
```
`EmployeeAction类中添加update方法`
```Java
public String update(){

	return "success";
}
```

`目前EmployeeAction类中的方法如下：`
```Java
public class EmployeeAction implements RequsetAware , ModelDriven<Employee>{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	private Employee employee;

	//为其方便添加一个setEmployeeId方法
	public void setEmployeeId(Integer employeeId){
	
		this.employeeId = employeeId;
	
	}
	
	//需要在当前的EmployeeAction中定义 employeeId属性
	//以接收请求参数
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		System.out.println("employee：" + employee.hashCode());
		System.out.println("值栈栈顶对象：" + ActionContext.getContext().getValueStack().peek().hashCode());
		//1、获取请求参数：通过定义对应属性的方式
		//2、调用Dao的save方法
		dao.save(employee);
		//3、通过redirectAction的方式响应结果给emp-list
		//return "save";
		return "success";
	}
	
	public String delete(){
		dao.delete(employeeId);
		//返回结果的类型应为：redirectAction
		//也可以时chain:实际上chain时没有必要的。因为不需在下一个Action中保留当前Action的状态；还有，若使用chain，则到达目标页面后，地址栏显示的依然时删除的那个链接，刷新时会有重复提交。
		//return "delete";
		return "success";
	}

	public String edit(){
		return "edit";
	}

	public String update(){
		dao.update(employee);
		return "success";
	}
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
	
	@Override 
	public Employee getModel() { 
		//判断Create 还是Edit
		//若为Create,则employee = new Employee();
		//若为Edit,则employee = dao.get(employeeId);
		//可以通过判定employeeId是否为空，来决定哪个为Create哪个是Edit;若通过employeeId来判断，则需要在modelDriven拦截器之前先执行一个params拦截器
		//而这可以通过使用paramsParams拦截器栈实现
		//需要在struts.xml文件中配置使用paramsPrepareParams作为默认的拦截器栈。
		//添加是：没有employeeId这个请求参数
		if(employeeId == null || employeeId.equals("")){
			employee = new Employee();
		}else{ //回显页面时：有employeeId这个请求参数
			employee = dao.get(emplotee.getEmployeeId());
		}
		return employee;
	}
}
```
![[Pasted image 20240831114633.png]]

### `小结`
`使用paramsPrepareParams拦截器栈后的运行流程`
	`1).paramsPrepareParams 和 defaultStack 一样都是拦截器栈，而struts-default 包默认使用的是后者`
	`2).可以在struts配置文件中通过以下方式修改使用的默认的拦截器栈：
	` <default-interceptor-ref name="paramsPrepareParams"></default-interceptor-ref>
	`3).paramsPrepareParams 拦截器在于：
	`params -> modelDriven -> params
	`所以可以先把请求参数赋给Action 对应的属性，再根据赋给Action 的那个属性值决定压到值栈栈顶的对象，最后再为栈顶对象的属性赋值`
	`对edit操作而言：`
		`1.先为EmployeeAction的employeeId赋值`
		`2.根据employeeId从数据库中加载对应的对象，并放入值栈的栈顶`
		`3.再为栈顶对象的employeeId 赋值(实际上此时employeeId 属性值已经存在)`
		`4.把栈顶对象属性回显在表单中。`
	`4).关于回显：Struts2 表单标签会从值栈中获取对应的属性值进行回显。`
	`5).存在的问题：`
		`getModel方法：`
```Java
@Override 
	public Employee getModel() { 
		if(employeeId == null || employeeId.equals("")){
			employee = new Employee();
		}else{ //回显页面时：有employeeId这个请求参数
			employee = dao.get(emplotee.getEmployeeId());
		}
		return employee;
	}
```
		`1、在执行删除的时候，getModel 不为null或空字符串，但 getModel 方法却从数据库加载了一个对象，本不该加载`
		`2、执行查询全部信息时，也 new 了个 Employee()对象。浪费`
	`6)、解决方案：使用PrepareInterceptor 和 Preparable 接口`
```Java
public class EmployeeAction implements RequsetAware , ModelDriven<Employee>,Preparable{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	private Employee employee;


	public void setEmployeeId(Integer employeeId){
	
		this.employeeId = employeeId;
	
	}
	
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		System.out.println("employee：" + employee.hashCode());
		System.out.println("值栈栈顶对象：" + ActionContext.getContext().getValueStack().peek().hashCode());
		dao.save(employee);
		return "success";
	}
	
	public String delete(){
		dao.delete(employeeId);
		return "success";
	}

	public String edit(){
		return "edit";
	}

	public String update(){
		dao.update(employee);
		return "success";
	}
	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
	
	@Override 
	public Employee getModel() { 
		if(employeeId == null || employeeId.equals("")){
			employee = new Employee();
		}else{ //回显页面时：有employeeId这个请求参数
			employee = dao.get(emplotee.getEmployeeId());
		}
		return employee;
	}

	/**
	*Preparable 方法的主要作用：为 getModel() 方法准备model的
	*/
	@Override 
	public void prepare() throws Exception { 
		if (employeeId != null) {
		 
			employee = dao.get(employeeId); // 根据 employeeId 从数据库获取 Employee 对象 
		} else { 
			employee = dao.get(emplotee.getEmployeeId());
		} 
	}
}
```
	`7).关于PrepareInterceptor`
```Java
public String doIntercept(ActionInvocation invocation) throws Exception {
    // 判断 Action 实例
    Object action = invocation.getAction();

    // 判断 Action 是否实现了 Preparable 接口
    if (action instanceof Preparable) {
        
        try {
            // 在 try 块中声明 prefixes 数组
            String[] prefixes;

            // 根据当前拦截器的 firstCallPrepareDo(默认false) 属性确定  prefixes
            if (firstCallPrepareDo) {
              prefixes = new String[] {ALT_PREPAPRE_PREFIX,PREPARE_PREFIX};
            } else {
                // 如果没有自定义的 prepare 方法，则调用默认的 prepare 方法
               prefixes= new String[] {PREPARE_PREFIX,ALT_PREPAPRE_PREFIX};
            }
            //若false,则prefixes：prepare prepareDo
            //调用前缀方法
            prefixMethodInvocationUtil.invokePrepareMethod(invocation,prefixes);
        } catch (InvocationTargetException e) {
            // 捕获并处理异常
            Throwable cause = e.getCause()；
            if(cause instanceof Exception){
	            throw (Exception) cause;
            }else if(cause instanceof Error){
	            throw (Error) cause;
            }else{
	            throw e;
            }
        }
		
        //根据当前拦截器的alwaysInvokePrepare(默认时true)决定是否调用Action 的 prepare 方法
        if(alwaysInvokePrepare){
	        ((Preparable) action).prepare();
        }
    }

    // 继续执行下一个拦截器或目标 Action
    return invocation.invoke();
}

```
```Java
public static void invokePrepareMethod(Object action, String[] prefixes) throws Exception{
	
	//获取Action实例
	Object action = invocation.getAction();
	//获取要调用的Action方法的名字(update)
	String methodName = actionInvocation.getProxy().getMethod();
	
	if(methodName == null){
		methodName = DEFAULT_INVOCATION_METHODNAME;
	}
	
	//获取前缀方法
	MethodName = getPrefixedMethod(prefixex,methodName,action);
	
	//若方法不为null，则通过反射调用前缀方法
	if(method != null){
		method.invoke(action,new Object[0]);
	}

}
```
	`1、若Action实现了Prepareble接口，则Struts 将会尝试执行prepare[ActionMethodName]方法，若prepare[ActionMethodName]不存在，则尝试执行prepareDo[ActionMethodName]方法，若都不存在，就都不执行。`
	`2、若PrepareInterceptor 的alwaysInvokePrepare属性为 false，则Struts2 将不会调用实现了Preparable 接口的Action 的 prepare()方法`
	可以为每一个ActionMethod 准备 prepare[ActionMethdName] 方法，而抛弃掉原来的prepare()方法，将PrepareInterceptor 的alwaysInvokePrepare属性置为 false，以避免Struts2框架再调用prepare()方法。
	`最后的改进：`
```Java
public class EmployeeAction implements RequsetAware , ModelDriven<Employee>,Preparable{
	private Dao dao = new Dao();
	
	private Integer employeeId;
	
	private Employee employee;


	public void setEmployeeId(Integer employeeId){
	
		this.employeeId = employeeId;
	
	}
	
	public void setEmployeeId(Integer employeeId) { 
		this.employeeId = employeeId; 
	}
	
	public String save(){
		dao.save(employee);
		return "success";
	}
	
	//私人定制
	public void prepareSave(){
		employee = new Employee();
	}
	
	public String delete(){
		dao.delete(employeeId);
		return "success";
	}

	public String edit(){
		return "edit";
	}

	public void prepareEdit(){
		employee = dao.get(employeeId);
	}

	public String update(){
		dao.update(employee);
		return "success";
	}

	public String prepareUpdate(){
		employee = new Employee();
	}

	public String list(){
		request.put("emps",dao.getEmployees());
		return "list";
	}
	
	private Map<String,Object> request;
	
	@Override 
	public void setRequest(Map<String, Object> arg0) { 
		this.request = arg0; 
	}
	
	@Override 
	public Employee getModel() { 
		if(employeeId == null || employeeId.equals("")){
			employee = new Employee();
		}else{ //回显页面时：有employeeId这个请求参数
			employee = dao.get(emplotee.getEmployeeId());
		}
		return employee;
	}

	/**
	*Preparable 方法的主要作用：为 getModel() 方法准备model的
	*/
	@Override 
	public void prepare() throws Exception { 
		System.out.println("prepare...")
	}
}
```
	`为其添加配置：`
```xml
<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
		<!--修改PrepareInterceptor 拦截器的alwaysInvokePrepare 属性值为false -->
		<interceptors>
			<interceptor-stack name="QingJiu">
				<interceptors-ref name="paramsPrepareParams">
					<param name="prepare.alwaysInvokePrepare">false</param>
				</interceptors-ref>
			</interceptor-stack>
		</interceptors>
	    <!--配置使用paramsPrepareParams作为默认的拦截器栈-->
	    <default-interceptor-ref name="QingJiu"></default-interceptor-ref>
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="emp-*" 
	        class="com.QingJiu.struts2.app.EmployeeAction"
	        method="{1}">
		    <result name="{1}">/emp-{1}.jsp</result>
		    <!--因为多次使用所以改为success-->
		    <result name="success" type="redirectAction">emp-list</result>
	    </action>
    </package>
</struts>
```


## 类型转换

`概述：`
- `从一个HTML表单到一个Action对象，类型转换是从字符串到非字符串。
	- `HTTP没有“类型”的概念。每一项表单输入只可能是一个字符串或一个字符串数字。在服务器端，必须把String转为特定的数据类型。`
 - `在struts2中，把请求参数映射到action属性的工作由Parameters拦截器负责，它是默认的defaultStack拦截器中的一员。Parameters拦截器可以自动完成字符串和基本数据类型之间的转换。`

`类型转换错误：`
- `若Action类没有实现ValidationAware接口：Struts在遇到类型转换错误时仍会继续调用其Action方法，就好像什么都没发生一样
- `若Action类实现ValidationAware接口:Struts在遇到类型转换错误时将不会继续调用其Action方法：Struts将检查相关action元素的声明是否包含着一个name=input的result。如果有，Struts将会把控制权转交给那个result元素：若没有input结果，Struts将把控制权转交给那个result元素；若没有input结果，Struts将抛出一个异常

`实现`
`web.xml`
```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <!-- Optional init parameters can be configured here -->
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

`struts2配置文件`
```xml
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="testConversion" class="com.QingJiu.struts2.ConversionAction">
	        <result> /success.jsp </result>
	        <result name="input"> /index.jsp </result>
        </action>
    </package>
</struts>

```

`index.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <!--
	问题一：如何覆盖默认的错误消息？
		对应的Action类所在的包中新建
		ActionClassName.properties 文件，ClassName即为包含着输入字段的Action类的类名
		在属性文件中添加如下键值对：
		invalid.fieldvalue.fieldName=XXX
	问题二：如果是simple主题，还会自动显示错误消息吗？如果不会显示，怎么办？
		通过debug标签，可知若转换出错，则在值栈的Action（实现了ValidationAware接口）对象中有一个fieldErrors属性。该属性的类型为Map<String,List<String>> 键：字段（属性名），值：错误消息组成的List。所以可以使用LE或OGNL的方式来显示错误消息${fieldErros.age[0]}
		还可以通过使用s:fielderror标签来显示。可以通过fieldName属性显示指定字段的错误
	问题三：若是simple主题，且使用s:fielderror filedName="filedname"></s:fielderror>来显示错误消息，则该消息在一个ul,li,span中，如何去除ul,li,span呢？
		在template.simple下面的filederror.ftl定义了simple主题下，s:fielderror标签显示错误消息的样式。所以修改还配置文件即可。在src下新建template.simple包，新建filederror.ftl文件，把原生的filederror.ftl中的内容复制到新建的filederror.ftl中，然后剔除ul,li,span部分即可
		
    -->

	 <s:form action="testConversion" theme="simple">
	     Age:<s:testfield name="age" label="Age"></s:testfield>
	     ${fieldErros.age[0]}
	     <s:fielderror fieldName="age"></s:fielderror>
	     <br>
	    <s:submit></s:submit>
    </s:form>

    <s:form action="testConversion">
	    <s:testfield name="age" label="Age"></s:testfield>
	    <s:submit></s:submit>
    </s:form>
</body>
</html>


```

`ConversionAction.java`
```java
public class ConversionAction {
    // 私有字段 age
    private int age;

    // 获取 age 的方法
    public int getAge() {
        return age;
    }

    // 设置 age 的方法
    public void setAge(int age) {
        this.age = age;
    }

	public String execute(){

		System.out.println("age：" + age);
		return "success";

	}
}

```

`sucess.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>success Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <h4>Success</h4>
</body>
</html>
```

`为了使其出错使可以抛出异常，需要继承ActionSupport`
```java
public class ConversionAction extends ActionSupport {

	private static final long serialVersionUID = 1L;

    // 私有字段 age
    private int age;

    // 获取 age 的方法
    public int getAge() {
        return age;
    }

    // 设置 age 的方法
    public void setAge(int age) {
        this.age = age;
    }

	public String execute(){

		System.out.println("age：" + age);
		return "success";

	}
}
```

`类型转换错误消息的定制`
- `作为默认的default拦截器的一员，ConversionError拦截器负责添加与类型转换有关的错误消息（前提:Action类必须实现了ValidationAware接口）和保存个请求参数的原始值。`
- `若字段标签使用的不是simple主题，则非法输入字段将导致一条有以下格式的出错消息：Invalid field value for field fieldName.`
- `覆盖默认的出错消息
	- `在对应的Action类所在的包中新建ActionClassName.properties 文件，ClassName即为包含着输入字段的Action类的类名`
	- `在属性文件中添加如下键值对：invalid.fieldvalue.fieldName=Custom error message`
- `定制出错消息的样式：
	- `每一条出错消息都被打包在一个HTML span元素里，可以通过覆盖其行标为errorMessage的那个css样式来改变出错消息的格式。`
- `显示错误消息：如果是simple主题，可以通过<s:fielderror filedName="filedname"></s:fielderror>标签显示错误消息`

`ConversionAction.properties`
```properties
invalid.fieldvalue.age=\u9519\u8bef\u7684\u5e74\u9f84\u683c\u5f0f.
```


## 定制类型转换器
`自定义类型转换器必须实现ongl.TypeConverter接口或对这个接口的某种具体实现做扩展。`
![[Pasted image 20240906004235.png]]
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>

	<!--
	如何自定义类型转换器？
	1、为什么需要自定义类型转换器？
		因为Struts 不能自动完成字符串引用类型的转换。
	2、如何自定义类型转化器？
		1.开发类型转换器的类：
			扩展StrutsTypeConverter类
		2.配置类型转换器：
			有两种方式：
			·基于字段的配置:
				>在字段所在的model(可能是Acition，可能是一个JavaBean)的包下，新建一个ModelClassName-conversion.properties文件
				>在该文件中输入键值对：fieldName=类型转换器的全类名.
				>第一次使用该转换器时创建实例。
				>该类型转换器是单实例的
			·基于类型的配置：
				>在src下新建xword-conversion.properties
				>键入：待转换的类型=类型转换其的全类名。
				>在当前struts2应用被加载时创建实例。
	-->
	 <s:form action="testConversion" theme="simple">
	     Age:<s:testfield name="age" label="Age"></s:testfield>
	     ${fieldErros.age[0]}
	     <s:fielderror fieldName="age"></s:fielderror>
	     <br>


		Birth：<s:textfield filedName="birth"></s:textfield>
		<s:fielderror fieldName="birth"></s:fielderror>
		<br>
	    <s:submit></s:submit>
    </s:form>

    <s:form action="testConversion">
	    <s:testfield name="age" label="Age"></s:testfield>
	    <s:submit></s:submit>
    </s:form>
</body>
</html>

```

`==方法一：基于字段==`
```java
public class ConversionAction extends ActionSupport {

	private static final long serialVersionUID = 1L;

    // 私有字段 age
    private int age;

    // 获取 age 的方法
    public int getAge() {
        return age;
    }

    // 设置 age 的方法
    public void setAge(int age) {
        this.age = age;
    }

	// 私有字段 birth
    private Date birth;

    // 获取 birth 的方法
    public Date getBirth() {
        return birth;
    }

    // 设置 birth 的方法
    public void setBirth(Date birth) {
        this.birth = birth;
    }
	

	public String execute(){

		System.out.println("age：" + age);
		return "success";

	}
}
```

`基于字段的配置`
```Java
import org.apache.struts2.util.StrutsTypeConverter;
import java.util.Map;

public class DateConverter extends StrutsTypeConverter {

	private DateFormat dateFormat;

	public DateConverter(){
		dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
	}

    @Override
    public Object convertFromString(Map context, String[] values, Class toClass) {
        // 调用父类方法，不进行任何重写
        if(toClass == Date.class){
	        if(values != null && values.length > 0){
		        String value = values[0];
		        try{
			        return dateFormat.parseObject(value);
		        }catch(ParseException e){
			        e.printStackTrace();
		        }
		        
	        }
        }
        return values;
    }

    @Override
    public String convertToString(Map context, Object o) {
        // 调用父类方法，不进行任何重写
	    if(o instanceof Date){
		    Date date = (Date) o;
		    return dateFormat.format(date);
	    }
        return null;
    }
}

```

`ConversionAction-conversion.properties`
```.properties
birth=DateConverter的全类名
```

`Customer.java`
```Java
public class Custormer{

	// 私有字段 age
    private int age;

    // 获取 age 的方法
    public int getAge() {
        return age;
    }

    // 设置 age 的方法
    public void setAge(int age) {
        this.age = age;
    }

	// 私有字段 birth
    private Date birth;

    // 获取 birth 的方法
    public Date getBirth() {
        return birth;
    }

    // 设置 birth 的方法
    public void setBirth(Date birth) {
        this.birth = birth;
    }

	@Override 
	public String toString() { 
		
		return "Customer{birth='" + birth + "', age=" + age + "}"; 
		
	}
	


}
```
`重写ConversionAction方法使其继承ModelDriven<Customer>`
```Java
public class ConversionAction extends ActionSupport implements ModelDriven<Custormer> {

	private static final long serialVersionUID = 1L;
	
	private Customer model;

	@Override 
	public Customer getModel() { 
		
		model = new Custormer();
		return model; 
		
	}
	
	public String execute(){

		System.out.println("model：" + model);
		return "success";

	}

	
}
```
`添加属性文件Custormer-conversion.properties`
```.properties
birth=DateConverter的全类名
```

`==方法二：基于类型==`
`xwork-conversion.properties`
```.properties
java.util.Data=DateConverter的全类名
```


`实例：创建时机的不同会导致无法对其转换格式`
`在web.xml中添加param`
```XML

<context-param>
	<param-name>pattern</param-name>
	<param-value>yyyy-MM-dd hh:mm:ss</paaram-value>
</context-param>

<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <!-- Optional init parameters can be configured here -->
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

```Java
import org.apache.struts2.util.StrutsTypeConverter;
import java.util.Map;

public class DateConverter extends StrutsTypeConverter {

	private DateFormat dateFormat;

	public DateConverter(){
		//获取当前WEB应用的初始化参数pattern
		ServletContext servletContext ServletContext.getServletContext();
		String pattern = servletContext.getInitParameter("pattern");
		dateFormat = new SimpleDateFormat(pattern);
	}

    @Override
    public Object convertFromString(Map context, String[] values, Class toClass) {
        // 调用父类方法，不进行任何重写
        if(toClass == Date.class){
	        if(values != null && values.length > 0){
		        String value = values[0];
		        try{
			        return dateFormat.parseObject(value);
		        }catch(ParseException e){
			        e.printStackTrace();
		        }
		        
	        }
        }
        return values;
    }

    @Override
    public String convertToString(Map context, Object o) {
        // 调用父类方法，不进行任何重写
	    if(o instanceof Date){
		    Date date = (Date) o;
		    return dateFormat.format(date);
	    }
        return null;
    }
}
```

`上面实例代码有个问题，就是要注意创建时机，下面代码就是对其的优化：`
```Java
import org.apache.struts2.util.StrutsTypeConverter;
import java.util.Map;

public class DateConverter extends StrutsTypeConverter {

	private DateFormat dateFormat;

	public DateConverter(){
		System.out.println("DateConverter.....");
	}

	public DateFormat getDateFormat(){

		if( dateFormat == null){
			//获取当前WEB应用的初始化参数pattern
			ServletContext servletContext ServletContext.getServletContext();
			String pattern = servletContext.getInitParameter("pattern");
			dateFormat = new SimpleDateFormat(pattern);
		}

		return dateFormat;

	}

    @Override
    public Object convertFromString(Map context, String[] values, Class toClass) {
        // 调用父类方法，不进行任何重写
        if(toClass == Date.class){
	        if(values != null && values.length > 0){
		        String value = values[0];
		        try{
			        return dateFormat.parseObject(value);
		        }catch(ParseException e){
			        e.printStackTrace();
		        }
		        
	        }
        }
        return values;
    }

    @Override
    public String convertToString(Map context, Object o) {
        // 调用父类方法，不进行任何重写
	    if(o instanceof Date){
		    Date date = (Date) o;
		    return dateFormat.format(date);
	    }
        return null;
    }
}
```

## `类型转换与复杂数据配合使用`

`创建两个类：`
`Manager.java`
```Java
public class Manager{

	private String name;

	private Date birth;


	// Getter for name 
	public String getName() { 
		return name; 
	} 
	
	// Setter for name 
	public void setName(String name) { 
		this.name = name; 
	} 
	
	// Getter for birth 
	public Date getBirth() { 
		return birth; 
	} 
	
	// Setter for birth 
	public void setBirth(Date birth) { 
		this.birth = birth; 
	}

	@Override 
	public String toString() { 
		return "Manager{name='" + name + "', birth=" + birth + "}"; 
	}

}
```

`Department.java`
```Java
public class Department {

	/**
		1、Department 是模型，实际录入的Department.deptName 可以直接写到s:textfield的name属性中。那么 mgr 属性如何处理？
		Struts2 表单标签的name 值可以被复位属性的属性：name=mgr.name ,name=mgr.birth

		2、mgr 中有一个Date类型的 birth 属性， struts2 可以完成自动的类型转换吗？
		全局的类型转换器可以正常工作


		
	*/

    private Integer id;
    private String deptName;
    private Manager mgr;

    // Getter for id
    public Integer getId() {
        return id;
    }

    // Setter for id
    public void setId(Integer id) {
        this.id = id;
    }

    // Getter for deptName
    public String getDeptName() {
        return deptName;
    }

    // Setter for deptName
    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    // Getter for manager
    public Manager getMgr() {
        return mgr;
    }

    // Setter for manager
    public void setMgr(Manager mgr) {
        this.mgr = mgr;
    }

    // Override toString method
    @Override
    public String toString() {
        return "Department{id=" + id + 
               ", deptName='" + deptName + "'" + 
               ", manager=" + (mgr != null ? mgr.toString() : "null") + "}";
    }
}

```

`complext-property.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testComplextProperty">
	    <s:textfield name="deptName" label="DeptName"></s:textfield>
	    <!--映射属性的属性的属性也是可以使用全局类型转换器 -->
	    <s:textfield name="mgr.naem" label="MgrName"></s:textfield>
	    <s:textfield name="mgr.birth" label="MgrBirth"></s:textfield>
	    <s:submit></s:submit>
    </s:form>
</body>
</html>
```
`TestComlextPropertyAction.java`
```Java
public class TestComlextPropertyAction extends ActionSupport implements ModelDriven<Department>{

	private static final long serialVersionUID = 1L; //固定版本的id

	private Department department;

	@Override 
	public Department getModel() { 
		department = new Department;
		return department; 
	}

    @Override
    public String execute() throws Exception {
        return super.execute();
    }
}

```

`sturts.xml的配置文件`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="testConversion" class="com.QingJiu.struts2.ConversionAction">
	        <result> /success.jsp </result>
	        <result name="input"> /index.jsp </result>
        </action>
        <action name="testComplextProperty" calss="TestComlextPropertyAction全类名">
	        <result>/success.jsp</result>
        </action>
    </package>
</struts>
```

`Struts还允许填充Conllection里的对象，这常见于需要快速录入数据的场景`

```Java

public class TestCollectionAction extends ActionSupport {

    private static final long serialVersionUID = 1L;

    private Collection<Manager> mgrs = null;

    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }

    // Getter 方法
    public Collection<Manager> getMgrs() {
        return mgrs;
    }

    // Setter 方法
    public void setMgrs(Collection<Manager> mgrs) {
        this.mgrs = mgrs;
    }
}

```

`为其添加配置`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
        <!-- 这里可以定义 actions, interceptors 等 -->
        <action name="testConversion" class="com.QingJiu.struts2.ConversionAction">
	        <result> /success.jsp </result>
	        <result name="input"> /index.jsp </result>
        </action>
        <action name="testComplextProperty" calss="TestComlextPropertyAction全类名">
	        <result>/success.jsp</result>
        </action>
        <action name="testConversion2" calss="TestCollectionAction全类名">
	        <result>/success.jsp</result>
        </action>
        
    </package>
</struts>
```


## `消息处理与国际化`
`概述：`
- `在程序设计领域，把在无需改写源代码即可让开发出来的应用程序能够支持多种语言和数据格式的技术称为国际化.
- `与国际化对应的时本地化，指让一个具备国际化支持的应用程序支持某个特定的地区.`
- `Struts2国际化是建立在Java国际化基础上的：`
	- `与不同国家/语言提供对应的消息资源文件`
	- `Struts2矿建会根据请求中包含的Locale加载对应的资源文件`
	- `通过程序代码取得该资源文件中指定key对应的消息`

`创建动态的java工程`
`1、国际化的目标`
	>`如何配置国际化资源文件`
		`Action范围资源文件：在Action类文件所在的路径建立名为Action_语言_国家.properties的文件`
		`包范围资源文件：在包的根路径下建立文件名为package_语言_国家.properties的属性文件，一旦建立，处于该包下的所以Action都可以访问该资源文件。注意：包范围资源文件的bassName就是package，不是Action所在的包名。`
		`全局资源文件`
			`命名方式：basename_语言_国家.properties`
			`struts.xml：<constant name="struts.custom.i18n.resources" value="baseName"/>`
			`struts.properties：struts.custom.i18n.resources=baseName`
		`临时指定资源文件：<s:i18n.../>标签的name属性指定临时的国际化资源文件`
		`国际化资源文件加载的顺序如何？ 离当前Action较近的将被优先加载`
			``
	>`如何在页面上和Action类中访问国际化资源文件的value值`
		`在Action类中.若Action实现了TextProvider 接口，则可以调用其getText()方法获取value值。`
			`通过继承ActionSupport的方式。`
		`页面上可以使用s:text标签；对于表单标签可以使用表单标签的key属性值`
			`若有占位符，则可以使用s:text标签的s:param子标签来添充占位符`
			`可以利用标签和OGNL表达式直接访问值栈中的属性值(对象栈和Map栈)`
	>`实现通过超链接切换语言`
	

`全局资源文件：`
`i18n.properties`
```.properties
username=UserName
password=Password
submit=Submit
time=Time:{0}
time2=Time:${date}
```
`i18n_en_US.properties`
```.properties
username=UserName
password=Password
submit=Submit
time2=Time:${date}
```
`i18n_zh_CN.properties`
```.properties
username=\u7528\u6237\u540D
password=\u5BC6\u7801
submit=\u63D0\u4EA4
time=\u65F6\u95F4:{0}
time2=\u65F6\u95F4:${date}
```
`web.xml`
```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <!-- Optional init parameters can be configured here -->
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

`struts.xml`
```xml
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
       
    </package>
</struts>
```

`i18n.jsp`
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>i18n Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="">
	    <!--label的方式是把label写死在标签里-->
	    <s:textfield name="username" label="UserName"></s:textfield>
	    <!-- 
	    若label标签使用 %{getText('username')} 的方式就也可以从国际化资源文件中获取value值了，因为此时在对象栈中有DefaultTextProvider的一个实例，该对象中提供了访问国际化资源文件的getText()方法，同时还需要通知struts2框架label中放入的不再是一个普通的字符串，而是一个OGNL表达式，所以使用%{}把getText()包装起来，以强制进行OGNL解析。
	     -->
	    <s:textfield name="username" label="%{getText('username')}"></s:textfield>
	    

		<!--key的方式是直接上资源文件中获取value值-->
	    <s:textfield name="username" key="username"></s:textfield>
		<s:possword name="password" key="password"></s:possword>
		<s:submit key="submit"></s:submit>
		
    </s:form>

	<s:text name="time">
		<s:param value="date"></s:param>
	</s:text>
	<br>
	<s:text name="time2"></s:text>
	<br>

	<s:form action="" theme="simple">
	   <!-- 
		   页面上可以直接使用 <s:text name=""/> 标签来访问国际资源文法里的value值。
		 -->
	    <s:text name="username"/>:<s:textfield name="username" label="%{getText('username')}"></s:textfield>
	    
	    <s:text name="username"/>:<s:textfield name="username" key="username"></s:textfield>
		<s:text name="password"/>:<s:possword name="password" key="password"></s:possword>
		<s:submit key="submit" value="%{getText('submit')}"></s:submit>
    </s:form>
</body>
</html>
```

`TestI18nAction.java`
```Java
public class TestI18nAction extends ActionSupport{

	private static final long serialVersionUID = 1L;

	private Date date = null;

	public Date getDate(){
		return date;
	}

	public void setDate(Date date){
		this.date = date;
	}


	@Override
	public String execute() throws Exception{

		date = new Date();

		//1、在Action中访问国际化资源文件的value值
		String username = getText("username");
		System.out.println(username);

		

		//2、带占位符的
		String time = getText("time",Arrays.asList(date)));
		System.out.println(time);
		return success;

	}

}
```
`配置struts.xml`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
       <action name="testI18n" calss="TestI18nAction的全类名">
	       <result>/i18n.jsp</result>
       </action>
    </package>
</struts>
```
`index.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <a href="testI18n"> Test I18n </a>
</body>
</html>
```


## `利用超链接实现动态加载国际化资源文件`
`1、关键之处在于知道Struts2框架是如何确定Local对象的；`
`2、可以通过阅读I18N 拦截器知道。`
![[Pasted image 20240910205536.png]]

`i18n.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>i18n Page</title>
</head>
<body>

	<a href="testI18n.action?request_Locale=en_US">English</a>
	<a href="testI18n.action?request_Locale=zh_CN">中文</a>
	<br>

	<a href="index.jsp"> Index Page </a>

	<s:text name="time">
		<s:param value="date"></s:param>
	</s:text>
	<br>
	<s:text name="time2"></s:text>
	<br>

</body>
</html>
```
`3、具体确定Locale对象的进程`
	>`Struts2使用i18n拦截器处理国际化，并且将其注册在默认的拦截器栈中`
	>`i18n拦截器在执行Action方法前，自动查找请求中一个名为request_locale的参数`
		`如果该参数存在，拦截器将其作为参数，转换成Locale对象，并将其设置为用户默认的Locale(代表国家/语言环境)。并把其设置为session的WW_TRANS_I18N_LOCALE属性。`
	>`若request 没有名为request_locale的参数，则i18n拦截器会从Session中获取WW_TRANS_I18N_LOCALE的属性值。若该值不为空，则将该属性值为浏览器的默认Locale`
	>`若session中的WW_TRANS_I18N_LOCALE的属性值为空，则从ActionContext中获取Locale对象`
`4、具体实现：只需要在超链接的后面附着 request_locale的请求参数，值是语言国家代码。`
	`<a href="testI18n.action?request_Locale=en_US">English</a>
	`<a href="testI18n.action?request_Locale=zh_CN">中文</a>`
	`注意：超链接必须是一个Struts2的请求，即使i18n拦截器工作！`


## `Struts2运行流程分析`
`运行流程图：`
![[Pasted image 20240915100741.png]]



## `输入验证`
`概述：`
- `一个健壮的web应用程序必须确保用户输入是合法、有效的。`
- `Struts2的输入验证：`
	- `基于XWork Validation Framework 的声明验证：Struts2提供了一些基于XWork Validation Framework的内建验证程序。使用这些验证程序不需要编程，只要一个XML文件里对验证程序应该如何工作作出声明就可以了，需要声明的内容包括：`
		- `哪些字段需要进行验证`
		- `使用什么验证规则`
		- `在验证失败说应该把什么样的出错消息发送到浏览器端`
	- `编程验证：通过编写代码来验证用户输入`


`声明式验证：`
- `声明式验证程序可以分为两类：`
	`字段验证：判断某个字段属性的输入是否有效`
	`非字段验证：不只针对某个字段，而是针对多个字段的输入值之间的逻辑关系进行校验。`
- `使用一个声明式验证程序需要3个步骤：`
	`1、确定哪些Action字段需要验证`
	`2、编写一个验证程序配置文件.它的文件名必须是以下两种格式之一：`
		`若一个Action类的多个action使用同样的验证规则： ActionName.validation.xml`
		`若一个Action类的多个action使用不同的验证规则：ActionClass-alias-validation.xml`
	`3、确定验证失败时的响应页面：在struts.xml文件中定义一个<result name="input">的元素`
`TestValidtionAction.java`
```Java
import com.opensymphony.xwork2.ActionSupport;

public class TestValidtionAction extends ActionSupport {

    private int age;

    // Getter method for 'age'
    public int getAge() {
        return age;
    }

    // Setter method for 'age'
    public void setAge(int age) {
        this.age = age;
    }

    
    @Override
    public String execute() throws Exception {
       return SUCCESS;
    }
}

```

`validtion.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testValidation">
	    <s:textfiled name="age" label="Age"></s:testfile>
	    <s:submit></s:submit>
    </s:form>
</body>
</html>
```

`struts.xml`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
       <action name="testValidation" calss="testValidation的全类名">
	       <result>/success.jsp</result>
	       <!--若验证失败转向的input-->
	       <result name="input">/validation.jsp</result>
       </action>
    </package>
</struts>
```

`success.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <h4>Success Page</h4>
</body>
</html>
```

`TestValidtionAction-validation.xml`
```XML
<!DOCTYPE validators PUBLIC
        "-//Apache Struts//XWork Validator 1.0.2//EN"
        "http://struts.apache.org/dtds/xwork-validator-1.0.2.dtd">

<validators>
    <field name="username">
        <field-validator type="requiredstring">
            <message key="requiredstring"/>
        </field-validator>
    </field>
    <field name="password">
        <field-validator type="requiredstring">
            <message key="requiredstring"/>
        </field-validator>
    </field>
</validators>

```

``
```XML
<!DOCTYPE validators PUBLIC
		"-//Apache Struts//XWork Validator 1.0.2//EN"
		"E:\Struts2\struts-2.3.20-all\struts-2.3.20\src\xwork-core\src\main\resources\xwork-validator-1.0.3.dtd">


<validators>
	<!--针对age属性进行验证。基于字段的验证-->
	<field name="age">
		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message>Age needs to be between ${min} and ${max}</message>
		</field-validator>
	</field>
</validators>
```

`1、先明确哪一个Action的哪一个字段进行验证：age`
`2、编写配置文件：`
	`把E:\Struts2\struts-2.3.20-all\struts-2.3.20\apps\struts2-blank\WEB-INF\classes\example下的Login-validation.xml文件复制到当前Action所在包下。`
	`把该配置文件改为: 把Login 改为Action的名字`
	`在配置文件中可以定义错误消息：`
```
<field name="age">
		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message>Age needs to be between ${min} and ${max}</message>
		</field-validator>
	</field>
```
	`错误消息可以国际化吗？`
	`在资源文件中写好对应的信息`
```
<message key="error.int"></message>
```
`3、若验证失败，则转向input 的result.所以需要name=input 的result`
	`<result name="input">/validation.jsp</result>`
`4、如何显示错误消息？`
	`若使用的是非simple主题，则自动显示错误消息`
	`若使用的是simple主题，则需要自己打印`
``` 
	
	<s:form action="testValidation" theme="simple">
	    Age: <s:textfiled name="age" label="Age"></s:testfile>
	    <s:fielderror fieldName="age" ></s:fielderror>
	    ${fieldError.age[0]}
	    <s:submit></s:submit>
    </s:form>
```
	`注意：若一个Action类可以应答多个action请求，多个action请求使用不同的验证规则，怎么办？`
		`为每一个不同的action请求定义其对应的验证文件：ActionClassName-AliasName-validation.xml`
		`不带别名的配置文件：ActionClassName-Alias-validation.xml中的验证规则依然会发生作用。可以把各个action公有的验证规则配置在其中。但需要注意的是，只适用于某一个action的请求的原则规则就不要在这里配置了。`
	`声明式验证框架的原理:`
		`struts2默认的拦截器栈中提供了一个validation拦截器`
		`每个具体的验证规则都会对应具体的一个验证其。有一个配置文件把验证规则名称和验证器关联起来。而实际上验证的是那个验证器。`

`验证程序的配置`
![[Pasted image 20240915165718.png]]

## `Struts2内建的验证程序`
- `required：确保某个字段的值不是空值null`
- `requiredstring：确保某给定字段的值既不是空值null，也不是空白。`
	- `trim参数。默认为true，表示struts在验证该字段值之前剔除前后空格`
- `stringlength：验证一个非空的字段值是不是有足够的长度`
- `date：确保某给定日期字段的值落在一个给定的范围内`
- `email：检查给定string值是否是一个合法的email`
- `url：检查给定string值是否是一个合法url`
- `regex：检查某给定字段的值是否一个给定的正则表达式模式相匹配：`
	- `expression*：用来匹配的正则表达式`
	- `caseSensitive：是否区分字母大小写.默认为true`
	- `trim：是否去除前后空格.默认true`
- `int：检查给定整数字段值是否在某一个范围内`
- `conversion：检查对给定Action属性进行的类型转化是否会导致一个转换错误。该验证程序还可以在默认的类型转换消息的基础上添加一条自定义的消息`
```XML
<field name="age">
		<!-- 设置短路验证：当前验证没有通过，则不再进行下面的验证 -->
		<field-validator type="conversion" short-circuit="true">
			<message> Conversion Error Occurred</message>
		</field-validator>

		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message>Age needs to be between ${min} and ${max}</message>
		</field-validator>
	</field>
```
`短路验证器：`
- `<field-validator/>元素和<validator/>元素可以指定一个可选的shor-circuit属性，该属性指定该验证器是否是短路验证器，默认值为fales.`
- `对同一个字段内的多个验证器，如果是一个短路验证器验证失败，则其他验证器不会继续校验`

`为了实现类型转换失败将其停下不继续往下走：对其com.opensymphony.xwork2.interceptor 原代码进行修改。com.opensymphony.xwork2.interceptor创建这个包下在新建这个类ConversionErrorInterceptor`
```Java

package com.opensymphony.xwork2.interceptor;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.ValidationAware;
import com.opensymphony.xwork2.conversion.impl.XWorkConverter;
import com.opensymphony.xwork2.util.ValueStack;
import org.apache.commons.lang3.StringEscapeUtils;

import java.util.HashMap;
import java.util.Map;


public class ConversionErrorInterceptor extends AbstractInterceptor {

    public static final String ORIGINAL_PROPERTY_OVERRIDE = "original.property.override";

    protected Object getOverrideExpr(ActionInvocation invocation, Object value) {
        return escape(value);
    }

    protected String escape(Object value) {
        return "\"" + StringEscapeUtils.escapeJava(String.valueOf(value)) + "\"";
    }

    @Override
    public String intercept(ActionInvocation invocation) throws Exception {

        ActionContext invocationContext = invocation.getInvocationContext();
        Map<String, Object> conversionErrors = invocationContext.getConversionErrors();
        ValueStack stack = invocationContext.getValueStack();

        HashMap<Object, Object> fakie = null;

        for (Map.Entry<String, Object> entry : conversionErrors.entrySet()) {
            String propertyName = entry.getKey();
            Object value = entry.getValue();

            if (shouldAddError(propertyName, value)) {
                String message = XWorkConverter.getConversionErrorMessage(propertyName, stack);

                Object action = invocation.getAction();
                if (action instanceof ValidationAware) {
                    ValidationAware va = (ValidationAware) action;
                    va.addFieldError(propertyName, message);
                }

                if (fakie == null) {
                    fakie = new HashMap<Object, Object>();
                }

                fakie.put(propertyName, getOverrideExpr(invocation, value));
            }
        }

        if (fakie != null) {
            // if there were some errors, put the original (fake) values in place right before the result
            stack.getContext().put(ORIGINAL_PROPERTY_OVERRIDE, fakie);
            invocation.addPreResultListener(new PreResultListener() {
                public void beforeResult(ActionInvocation invocation, String resultCode) {
                    Map<Object, Object> fakie = (Map<Object, Object>) invocation.getInvocationContext().get(ORIGINAL_PROPERTY_OVERRIDE);

                    if (fakie != null) {
                        invocation.getStack().setExprOverrides(fakie);
                    }
                }
            });
        }

		//当验证通过了在往后走
		Object action = invocation.getAction();
		if (action instanceof ValidationAware) {
			ValidationAware va = (ValidationAware) action;

			if(va.hasFieldErrors() || va.hasActionErrors()){
				return "input";
			}
			
		}

        return invocation.invoke();
    }

    protected boolean shouldAddError(String propertyName, Object value) {
        return true;
    }
}

```

`短路验证：若对一个字段使用多个验证器，默认情况下会执行所有的验证器。若希望前面的验证没有通过，后面的就不要再验证，可以使用短路验证。`

`若类型转换失败，默认情况下还会执行后面的拦截器，还会进行验证。可以通过修改ConversionErrorInterceptor 源代码的方式使当类型转换失败是，不再执行后续的验证拦截器，而直接返回input的result`


- `expression和fieldexpression：用来验证字段是否满足一个OGNL表达式`
	- `前者是一个非字段验证程序，后者是一个字段验证程序`
	- `前者在验证失败时将生产一个action错误，而后者在验证失败时会生成一个字段错误`
	- `expression*：用来进行验证的OGNL表达式`

`非字段验证：不是针对某一个字段的验证。 显示非字段验证的错误消息，要使用s:actionerror标签`
``
`非字段验证示例：`
```XML
<field name="age">
		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message>Age needs to be between ${min} and ${max}</message>
		</field-validator>
</field>

<validator type="expression">
	<param name="expression"> ![CDATA[password==password2]] </param>
	<message> password is not equals to password2 </message>
</validator>

```


```jsp
	<s:actionerror/>
	<s:form action="testValidation" theme="simple">
	    Age: <s:textfiled name="age" label="Age"></s:testfile>
	    <s:fielderror fieldName="age" ></s:fielderror>
	    ${fieldError.age[0]}


		Password: <s:password name="password"></s:password>
		Password2: <s:password name="password2"></s:password>
	    <s:submit></s:submit>
    </s:form>
```

`在TestValidtionAction其中加上对应的字段`
```Java
import com.opensymphony.xwork2.ActionSupport;

public class TestValidationAction extends ActionSupport {

    private int age;
    private String password;
    private String password2;

    // Getter method for 'age'
    public int getAge() {
        return age;
    }

    // Setter method for 'age'
    public void setAge(int age) {
        this.age = age;
    }

    // Getter method for 'password'
    public String getPassword() {
        return password;
    }

    // Setter method for 'password'
    public void setPassword(String password) {
        this.password = password;
    }

    // Getter method for 'password2'
    public String getPassword2() {
        return password2;
    }

    // Setter method for 'password2'
    public void setPassword2(String password2) {
        this.password2 = password2;
    }

    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }
}

```

`字段验证vs非字段验证：`
- `字段验证字段优先，可以为一个字段配置多个验证规则`
- `非字段验证规则优先`
- `大部分验证规则支持两种验证器，但个别的验证规则只能使用非字段验证，例如表达式验证`


## `错误消息的重用性`
`不同的字段使用同样的验证规则，而且使用同样的响应消息`
```jsp
	<s:actionerror/>
	<s:form action="testValidation" theme="simple">
	    Age: <s:textfiled name="age" label="Age"></s:testfile>
	    <s:fielderror fieldName="age" ></s:fielderror>
	    ${fieldError.age[0]}


		Password: <s:password name="password"></s:password>
		Password2: <s:password name="password2"></s:password>

		Count：<s:textfield name="count"></s:textfield>
		<s:fielderror fieldName="count"></s:fielderror>
	    <s:submit></s:submit>
    </s:form>
```

`添加count字段`
```Java
import com.opensymphony.xwork2.ActionSupport;

public class TestValidationAction extends ActionSupport {

    private int age;
    private String password;
    private String password2;
    private int count;

    // Getter method for 'age'
    public int getAge() {
        return age;
    }

    // Setter method for 'age'
    public void setAge(int age) {
        this.age = age;
    }

    // Getter method for 'password'
    public String getPassword() {
        return password;
    }

    // Setter method for 'password'
    public void setPassword(String password) {
        this.password = password;
    }

    // Getter method for 'password2'
    public String getPassword2() {
        return password2;
    }

    // Setter method for 'password2'
    public void setPassword2(String password2) {
        this.password2 = password2;
    }

    // Getter method for 'count'
    public int getCount() {
        return count;
    }

    // Setter method for 'count'
    public void setCount(int count) {
        this.count = count;
    }

    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }
}

```

`补上其对应的规则：`
```XML
<field name="age">
		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message key="error.int"></message>
		</field-validator>
</field>

<field name="count">
		<field-validator type="int">
		<param name="min">1</param>
		<param name="max">10</param>
		<message key="error.int"></message>
		</field-validator>
</field>

<validator type="expression">
	<param name="expression"> ![CDATA[password==password2]] </param>
	<message> password is not equals to password2 </message>
</validator>
```

`i18n.properties`
```properties
error.int=${filedName} needs to be between ${min} and ${max}
```

`也可使实现国际化：`
```properties
error.int=${getText(filedName)} needs to be between ${min} and ${max}

age=\u5E74\u9F84
count=\u6570\u91CF

```


## `自定义验证器`

`定义一个验证器的类`
	>`自定义的验证器都需要实现 Validator
	>`可以选择继承ValidatorSupport 或 FieldValidatorSupport
	>`若希望实现一个一般的验证器，则可以继承ValidatorSupport
	>`若希望实现一个字段验证器，则可以继承FieldValidatorSupport
	>`具体实现可以参考目前已有的验证器
	>`若验证程序需要接受一个输入参数，需要为这个参数增加一个相应的属性
`在配置文件中配置验证器`
	>`默认的情况下mStruts2会在类的路径的根目录下加载validators.xml文件。在该文件中加载验证器。该文件的定义方式同默认的验证器的那个配置文件：位于E:\Struts2\struts-2.3.20-all\struts-2.3.20\src\xwork-core\src\main\resources\com\opensymphony\xwork2\validator\validators的default.xml
	>`若类路径下没有指定的验证器，则从com\opensymphony\xwork2\validator\validators下的default.xml 中的default.xml中的验证器加载。
`使用：和目前的验证器一样`
`示例代码：自定义一个18位身份证验证器`
`创建验证器的类：IDCardValidator`
```Java
import com.opensymphony.xwork2.validator.ValidationException;
import com.opensymphony.xwork2.validator.validators.FieldValidatorSupport;

public class IDCardValidator extends FieldValidatorSupport {

    @Override
    public void validate(Object object) throws ValidationException {
       //1、获取字段的名字和值
		String fieldName = getFieldName();
        Object value = this.getFieldValue(fieldName, object);
		//2、验证
		IDCard idCard = new IDCard();
		boolean result = idCard.Verify((String)value);
		//3、若验证失败，则
		if(!result){
			addFieldError(fieldName, object);
		}
    }
}

```

`在根目录下定义一个validators.xml文件`
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE validators PUBLIC
        "-//Apache Struts//XWork Validator Definition 1.0//EN"
        "http://struts.apache.org/dtds/xwork-validator-definition-1.0.dtd">

<!-- START SNIPPET: validators-default -->
<validators>
    <validator name="idcard" class="IDCardValidator的全类名"/>
</validators>
<!--  END SNIPPET: validators-default -->

```

`在validation.jsp中添加字段`
```JSP
	<s:actionerror/>
	<s:form action="testValidation" theme="simple">
	    Age: <s:textfiled name="age" label="Age"></s:testfile>
	    <s:fielderror fieldName="age" ></s:fielderror>
	    ${fieldError.age[0]}


		Password: <s:password name="password"></s:password>
		Password2: <s:password name="password2"></s:password>

		Count：<s:textfield name="count"></s:textfield>
		<s:fielderror fieldName="count"></s:fielderror>

		idCard：<s:textfield name="idCard"></s:textfield>
		<s:fielderror fieldName="idCard"></s:fielderror>

	    <s:submit></s:submit>
    </s:form>
```

`在TestValidation中添加idCard的属性`
```JAVA
import com.opensymphony.xwork2.ActionSupport;

public class TestValidationAction extends ActionSupport {

    private int age;
    private String password;
    private String password2;
    private int count;
    private String idCard;

    // Getter method for 'age'
    public int getAge() {
        return age;
    }

    // Setter method for 'age'
    public void setAge(int age) {
        this.age = age;
    }

    // Getter method for 'password'
    public String getPassword() {
        return password;
    }

    // Setter method for 'password'
    public void setPassword(String password) {
        this.password = password;
    }

    // Getter method for 'password2'
    public String getPassword2() {
        return password2;
    }

    // Setter method for 'password2'
    public void setPassword2(String password2) {
        this.password2 = password2;
    }

    // Getter method for 'count'
    public int getCount() {
        return count;
    }

    // Setter method for 'count'
    public void setCount(int count) {
        this.count = count;
    }

    // Getter method for 'idCard'
    public String getIdCard() {
        return idCard;
    }

    // Setter method for 'idCard'
    public void setIdCard(String idCard) {
        this.idCard = idCard;
    }

    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }
}

```

`在配置文件中声明使用这个验证器：`
```XML
<field name="age">
		<field-validator type="int">
		<param name="min">20</param>
		<param name="max">50</param>
		<message key="error.int"></message>
		</field-validator>
</field>

<field name="count">
		<field-validator type="int">
		<param name="min">1</param>
		<param name="max">10</param>
		<message key="error.int"></message>
		</field-validator>
</field>

<field name="idCard">
		<field-validator type="idcard">
		<message>It is not a idCard</message>
		</field-validator>
</field>

<validator type="expression">
	<param name="expression"> ![CDATA[password==password2]] </param>
	<message> password is not equals to password2 </message>
</validator>
```



## `文件的上传下载`
`表单的准备：`
- `要想使用HTML表单上传一个或多个文件`
	- `须把HTML表单的enctype属性设置为multipart/form-data`
	- `须把HTML表单的method属性设置为post`
	- `需添加<input type="file">字段`

`Struts2对文件上传的支持`
- `在Struts应用程序里,FileUpload拦截器和Jakarta Commons FileUpload组件可以完成文件的上传`

`创建所需的struts配置文件、web配置文件、i18n的资源化文`
`struts.xml`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
       
    </package>
</struts>
```

`新建表单：upload.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testUpload" method="post" enctype="multipart/form-data">
	    <s:file name="ppt" label="PPTFile"></s:file>
	    <s:textfield name="pptDesc" label="PPTDesc"></s:textfield>

		<s:submit></s:submit>
    </s:form>
</body>
</html>
```

`创建UploadAction继承父类ActionSupport`
```Java
package com.example.action;

import com.opensymphony.xwork2.ActionSupport;
import org.apache.commons.io.FileUtils;

import java.io.File;

public class UploadAction extends ActionSupport {
    private static final long serialVersionUID = 1L;

    // 上传文件的临时文件
    private File ppt;
    private String pptFileName;
    private String pptContentType;
    
    // 文件描述
    private String pptDesc;

    // Getter 和 Setter for ppt, pptFileName, pptContentType
    public File getPpt() {
        return ppt;
    }

    public void setPpt(File ppt) {
        this.ppt = ppt;
    }

    public String getPptFileName() {
        return pptFileName;
    }

    public void setPptFileName(String pptFileName) {
        this.pptFileName = pptFileName;
    }

    public String getPptContentType() {
        return pptContentType;
    }

    public void setPptContentType(String pptContentType) {
        this.pptContentType = pptContentType;
    }

    // Getter 和 Setter for pptDesc
    public String getPptDesc() {
        return pptDesc;
    }

    public void setPptDesc(String pptDesc) {
        this.pptDesc = pptDesc;
    }

    @Override
    public String execute() throws Exception {
        ServletContext servletContext = ServletActionContext.getServletContext();
        String dir = servletContext.getRealPath("/files/"+pptFileName);

		FileOutputStream out = new FileOutputStream(dir);
		FileInputStream in = new FileInputStream(ppt);

		byte [] buffer = new byte[1024];
		int len = 0;

		while((len = in.read(buffer)) != -1){
			out.write(buffer,0,len);
		}

		out.close();
		in.close();

        return super.execute;
    
    }
}

```

`添加UploadAction所需的配置信息`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">
       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>
    </package>
</struts>
```

`创建success.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <h4>Success Page</h4>
</body>
</html>
```

- `struts2的文件上传实际上使用的是Commons FileUpload组件，所以需要导入commons-fileupload-1.3.1.jar commons-io-2.2.jar`
- `Struts2进行文件上传需要使用FileUpload拦截器`
- `基本的文件上传：：直接在Action中定义如下3个属性，并提供对应的getter 和setter方法
	`//文件对应的File对象`
	`private File ppt;
	`//文件类型`
	`private String pptFileName;
	`//文件名`
	`private String pptContentType;
- `使用IO流进行文件上传即可`
```Java
@Override
    public String execute() throws Exception {
        ServletContext servletContext = ServletActionContext.getServletContext();
        String dir = servletContext.getRealPath("/files/"+pptFileName);

		FileOutputStream out = new FileOutputStream(dir);
		FileInputStream in = new FileInputStream(ppt);

		byte [] buffer = new byte[1024];
		int len = 0;

		while((len = in.read(buffer)) != -1){
			out.write(buffer,0,len);
		}

		out.close();
		in.close();

        return super.execute;
    
    }
```
- `一次性传多个文件怎么办？`
	- `若传递多个文件，则上述的3个属性，可改为list类型。多个文件域的name属性值需要一致。`
```XML
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testUpload" method="post" enctype="multipart/form-data">
	    <s:file name="ppt" label="PPTFile"></s:file>
	    <s:textfield name="pptDesc[0]" label="PPTDesc"></s:textfield>
	    
		<s:file name="ppt" label="PPTFile"></s:file>
	    <s:textfield name="pptDesc[1]" label="PPTDesc"></s:textfield>
	    
	    <s:file name="ppt" label="PPTFile"></s:file>
	    <s:textfield name="pptDesc[2]" label="PPTDesc"></s:textfield>
		<s:submit></s:submit>
    </s:form>
</body>
</html>
```

```Java
package com.example.action;

import com.opensymphony.xwork2.ActionSupport;
import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;

import javax.servlet.ServletContext;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.List;

public class UploadAction extends ActionSupport {
    private static final long serialVersionUID = 1L;

    // 上传文件的临时文件
    private List<File> ppt;
    private List<String> pptFileName;
    private List<String> pptContentType;
    
    // 文件描述
    private List<String> pptDesc;

    // Getter 和 Setter for ppt, pptFileName, pptContentType
    public List<File> getPpt() {
        return ppt;
    }

    public void setPpt(List<File> ppt) {
        this.ppt = ppt;
    }

    public List<String> getPptFileName() {
        return pptFileName;
    }

    public void setPptFileName(List<String> pptFileName) {
        this.pptFileName = pptFileName;
    }

    public List<String> getPptContentType() {
        return pptContentType;
    }

    public void setPptContentType(List<String> pptContentType) {
        this.pptContentType = pptContentType;
    }

    // Getter 和 Setter for pptDesc
    public List<String> getPptDesc() {
        return pptDesc;
    }

    public void setPptDesc(List<String> pptDesc) {
        this.pptDesc = pptDesc;
    }

    @Override
    public String execute() throws Exception {
        ServletContext servletContext = ServletActionContext.getServletContext();
        
        // 遍历每个上传的文件
        for (int i = 0; i < ppt.size(); i++) {
            String dir = servletContext.getRealPath("/files/" + pptFileName.get(i));
            File fileToSave = new File(dir);

            try (FileInputStream in = new FileInputStream(ppt.get(i));
                 FileOutputStream out = new FileOutputStream(fileToSave)) {
                 
                byte[] buffer = new byte[1024];
                int len;

                while ((len = in.read(buffer)) != -1) {
                    out.write(buffer, 0, len);
                }
            } catch (Exception e) {
                e.printStackTrace();
                return ERROR;  // 如果出错，返回 ERROR 状态
            }
        }

        return SUCCESS;  // 返回 SUCCESS 状态
    }
}

```
- `可以对上传的文件进行限制吗？例如扩展名，内容类型，上传文件的大小？若可以，若出错，显示什么错误消息？消息可以定制嘛？`
	- `可以的`
	- `可以通过配置FileUpoadInterceptor拦截器的参数方式来进行限制`
		- `maximumSize(optional) - 默认的最大值为2M。上传的单个文件的最大值。`
		- `allowedTypes(optional) - 允许的上传文件的类型。多个使用逗号，分隔`
		- `allowedExtensions(optional) - 允许的上传文件的扩展名。多个使用逗号，分隔`
	- `定制错误消息。可以在国际化资源文件中定义如下的消息：`
		- `struts.message.error.uploading - 文件上传出错的消息`
		- `struts.message.error.file.too.large - 文件超过最大值的消息`
		- `struts.message.error.content.type.not.allowed - 文件内容类型不合法的消息`
		- `struts.message.error.file.extension.not.allowed - 文件扩展名不合法的消息`

```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">

	<interceptors>
		<interceptor-stack name="QingJiu">
			<interceptors-ref name="defaultStack">
				<param name="fileUpload.maximumSize">2000</param>
				<param name="fileUpload.allowedTypes">text/html,text/xml</param>
				<param name="fileUpload.allowedExtensions">html,dtd,xml</param>
			</interceptors-ref>
		</interceptors-stacke>
	</interceptors>

       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>
    </package>
</struts>
```

	`注意：在struts-2.3.20\src\core\src\main\resources\org\apache\struts2下的default.properties中有对上传的文件总大小的限制。可以使用常量的方式来进行修改`
	 `struts.multipart.maxSize=2097152`

`国际化资源文件`
```properties
struts.message.error.uploading = \u6587\u4EF6\u4E0A\u4F20\u51FA\u9519
struts.message.error.file.too.large = \u6587\u4EF6\u8D85\u8FC7\u6700\u5927\u503C
struts.message.error.content.type.not.allowed = \u6587\u4EF6\u5185\u5BB9\u7C7B\u578B\u4E0D\u5408\u6CD5
struts.message.error.file.extension.not.allowed = \u6587\u4EF6\u6269\u5C55\u540D\u4E0D\u5408\u6CD5
```
`此种消息定制并不完善。可以参考struts-2.3.20\src\core\src\main\resources\org\apache\struts2下的struts-messages.properties`
```properties\
struts.messages.error.uploading=Error uploading: {0}
struts.messages.error.file.too.large=File {0} is too large to be uploaded. Maximum allowed size is {4} bytes!
struts.messages.error.content.type.not.allowed=Content-Type not allowed: {0} "{1}" "{2}" {3}
struts.messages.error.file.extension.not.allowed=File extension not allowed: {0} "{1}" "{2}" {3}

```
`配置后使用拦截器栈`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">

		<interceptors>
			<interceptor-stack name="QingJiu">
				<interceptors-ref name="defaultStack">
					<param name="fileUpload.maximumSize">2000</param>
					<param name="fileUpload.allowedTypes">text/html,text/xml</param>
					<param name="fileUpload.allowedExtensions">html,dtd,xml</param>
				</interceptors-ref>
			</interceptors-stacke>
		</interceptors>

		<default-interceptor -ref name="QingJiu"></default-interceptor -ref>

       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>
    </package>
</struts>
```
`对页面内容做出修改：`
```XML
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testUpload" method="post" enctype="multipart/form-data" theme="simple">
	    <s:filederror name="ppt"></s:filederror>
	    <s:actionerror/>

	    PPTFile:<s:file name="ppt" label="PPTFile"></s:file>
	    PPTDesc:<s:textfield name="pptDesc[0]" label="PPTDesc"></s:textfield>
		<br>

		PPTFile:<s:file name="ppt" label="PPTFile"></s:file>
	    PPTDesc:<s:textfield name="pptDesc[1]" label="PPTDesc"></s:textfield>
		<br>

	    PPTFile:<s:file name="ppt" label="PPTFile"></s:file>
	    PPTDesc:<s:textfield name="pptDesc[2]" label="PPTDesc"></s:textfield>
		<br>
		<s:submit></s:submit>
    </s:form>
</body>
</html>
```

## `文件下载概述`
`在某些应用程序里，可能需要动态地把一个文件发送到用户的浏览中，而这个文件的名字和存放位置在编程时无法预知的。 `

`Stream结果类型`
- `Struts专门为文件下载提供了一种Stream结果类型。在使用一个Stream结果时，不必准备一个JSP页面`

`文件下载：`
- `Struts2 中使用type="stream" 的result进行下载即可`
- `具体细节参看E:/Struts2/struts-2.3.20-all/struts-2.3.20/docs/docs/stream-result.html这个文档`
- `可以为stream的result设置如下参数：`
	- `contentType - 结果类型 (default = `text/plain`).`
	- `contentLength - 下载文件的长度 (the browser displays a progress bar).
	- `contentDisposition - 设定content-Disposition响应头。该响应头只当接应是一个文件下载类型，一般取值为：attachment；filename (default = `inline`, values are typically _attachment;filename="document.pdf"_).
	- `inputName - 只当文件输入留的getter定义的那个属性的名字。默认值为inputStream (default = inputStream).
	- `bufferSize - 缓存的大小。默认值为1024 (default = 1024).
	- `allowCaching - 是否允许使用缓存 (default = true)
	- `contentCharSet - 指定下载的字符集
- `以上参数可以在Action中以getter方法的方式提供`

`dowlnoad.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <a href="testDownload">Down Load</a>
</body>
</html>
```

`DownloadAction`
```Java
import java.io.InputStream;

public class DownloadAction extends ActionSupport {

    private static final long serialVersionUID = 1L;

    private String contentType;
    private long contentLength;
    private String contentDisposition;
    private InputStream inputStream; 

    // Getter 和 Setter for contentType
    public String getContentType() {
        return contentType;
    }

    public void setContentType(String contentType) {
        this.contentType = contentType;
    }

    // Getter 和 Setter for contentLength
    public long getContentLength() {
        return contentLength;
    }

    public void setContentLength(long contentLength) {
        this.contentLength = contentLength;
    }

    // Getter 和 Setter for contentDisposition
    public String getContentDisposition() {
        return contentDisposition;
    }

    public void setContentDisposition(String contentDisposition) {
        this.contentDisposition = contentDisposition;
    }

    // Getter 和 Setter for inputStream
    public InputStream getInputStream() {
        return inputStream;
    }

    public void setInputStream(InputStream inputStream) {
        this.inputStream = inputStream;
    }

    @Override
    public String execute() throws Exception {
        //确定各个方法的值
        contentType = "text/html";
        contentDisposition="attachment;filename=hidden.html";

		//确定文件目录，方便读取
		ServletContext servletContext = ServletActionContext.getServletContext();
		String fileName = servletContext.getRealPath("/files/hidden.html");
		inputStream = new FileInputStream(fileName);
		contentLength = inputStream.available();


        return SUCCESS;
    }
}


```


`Struts.xml`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">

		<interceptors>
			<interceptor-stack name="QingJiu">
				<interceptors-ref name="defaultStack">
					<param name="fileUpload.maximumSize">2000</param>
					<param name="fileUpload.allowedTypes">text/html,text/xml</param>
					<param name="fileUpload.allowedExtensions">html,dtd,xml</param>
				</interceptors-ref>
			</interceptors-stacke>
		</interceptors>

		<default-interceptor -ref name="QingJiu"></default-interceptor -ref>

       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>

		<action name="testDownload" class="DownloadAction的全类名">
			<result type="stream">
				<param name="bufferSize">2048</param>
			</>
		</action>
    </package>
</struts>
```


## `防止表单重复提交`

- `表单的重复提交：`
	- `若刷新表单页面，再提交表单不算重复提交。`
	- `在不刷新表单页面的前提下：`
		- `多次点击提交按钮`
		- `已经提交成功，按"回退"之后，再点击"提交按钮"`
		- `在控制器响应页面的形式为转发情况下，若已经提交成功，然后点击"刷新"`
	- `重复提交的缺点：`
		- `加重了服务器的负担`
		- `可能导致错误操作`
- `注意的是：若使用的是redirect的响应类型，已经提交成功后，再点击"刷新"，不是表单的重复提交`
`新建页面token.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testToken">
	    <s:testfield name="username" label="Username"></s:testfield>
	     <s:submit></s:submit>
    </s:form> 
    
</body>
</html>
```

`新建一个TokenAction.java`
```Java
public class TokenAction extends ActionSupport {

    private static final long serialVersionUID = 1L;

    private String username;

    // Getter for username
    public String getUsername() {
        return username;
    }

    // Setter for username
    public void setUsername(String username) {
        this.username = username;
    }

    @Override
    public String execute() throws Exception {

		Thread.slepp(2000);

		System.out.println(username);
        return SUCCESS;
    }
}

```

`在struts.xml配置文件中添加配置`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">

		<interceptors>
			<interceptor-stack name="QingJiu">
				<interceptors-ref name="defaultStack">
					<param name="fileUpload.maximumSize">2000</param>
					<param name="fileUpload.allowedTypes">text/html,text/xml</param>
					<param name="fileUpload.allowedExtensions">html,dtd,xml</param>
				</interceptors-ref>
			</interceptors-stacke>
		</interceptors>

		<default-interceptor -ref name="QingJiu"></default-interceptor -ref>

       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>

		<action name="testDownload" class="DownloadAction的全类名">
			<result type="stream">
				<param name="bufferSize">2048</param>
			</result>
		</action>

		<action name="testToken" class="TokenAction的全类名">
			<result>/success.jsp</result>
		</action>
    </package>
</struts>
```

`如何解决重复提交问题？`
- `在token.jsp里添加token标签`
	- `生成一个隐藏域`
	- `在session添加一个属性值`
	- `隐藏域的值和session的属性值是一致的`
```xml
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <s:form action="testToken">
	    <s:token></s:token>
	    <s:testfield name="username" label="Username"></s:testfield>
	    <s:submit></s:submit>
    </s:form>
    
</body>
</html>
```

- `使用Token 或 TokenSession 拦截器`
	- `这两个拦截器均不在默认的拦截器栈中，所以需要手工配置一下`
	- `若使用Token拦截器，则需要配置一个token.valid的result`
	- `若使用TokenSession拦截器，则不需要配置任何其他的result`

- `Token VS TokenSession`
	- `都是解决表单重复提交问题的`
	- `使用token拦截器回转到token.valid这个result`
	- `使用tokenSession拦截器则还会响应那个目标页面，但不会执行tokenSession的后续拦截器。就像什么都没发生过一样`

`在struts.xml配置文件中，添加配置信息`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">

		<interceptors>
			<interceptor-stack name="QingJiu">
				<interceptors-ref name="defaultStack">
					<param name="fileUpload.maximumSize">2000</param>
					<param name="fileUpload.allowedTypes">text/html,text/xml</param>
					<param name="fileUpload.allowedExtensions">html,dtd,xml</param>
				</interceptors-ref>
			</interceptors-stacke>
		</interceptors>

		<default-interceptor -ref name="QingJiu"></default-interceptor -ref>

       <action name="testUpload" class="UploadAction的全类名">
	       <result>/success.jsp</result>
       </action>

		<action name="testDownload" class="DownloadAction的全类名">
			<result type="stream">
				<param name="bufferSize">2048</param>
			</result>
		</action>

		<action name="testToken" class="TokenAction的全类名">
			<interceptor-ref name="token"></interceptor-ref>
			<interceptor-ref name="defaultStack"></interceptor-ref>
			<result>/success.jsp</result>
			<result name="invalid.token">token-error.jsp</result>
		</action>
    </package>
</struts>
```

`创建token-error.jsp`
```JSP
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" contentType="text/html; charset=UTF-8" charset="UTF-8">
    <title>Index Page</title>
</head>
<body>
    <!-- 页面内容 -->
    <h4> Token Error Page </h4>
    <s:actionerror/>
</body>
</html>
```

`对错误消息进行修改，通过修改国际化配置资源文件:`
- `示例位置在：struts-2.3.20\src\core\src\main\resources\org\apache\struts2 下的struts-messages.properties 文件中。`
- `struts.messages.invalid.token=The form has already been processed or no token was supplied, please try again.`



## `自定义拦截器`
![[Pasted image 20240915100741.png]]

`Struts2拦截器`
- `拦截器是Struts2(Interceptor)的核心组成部分。`
- `Struts2很多功能都是构建在拦截器基础上的。如文件的上传和下载、国际化、数据类型转换等等`
- `Struts2拦截器在访问某个Action方法之前或之后实施拦截`
- `Struts2拦截器是可插拔的，拦截器是AOP(面向切面编程)的一种实现`
- `拦截器栈(Interceptor Stack)：将拦截器按一定顺序联结成一条链。在访问被拦截的方法是，Struts2拦截器链中的拦截器就会按其之前定义顺序被依次调用。`

`Interceptor接口`
- `每个拦截器都是实现了Interceptor接口的Java类`
	- `init：该方法将在拦截器被创建后立即被调用，它在拦截器的生命周期内只被调用一次。可以在该方法中对相关资源进行必要的初始化。`
	- `interecept：每拦截一个请求，该方法就会被调用一次`
	- `destroy：该方法将在拦截器被销毁之前被调用，他在拦截器的生命周期内也只被调用一次。`
- `Struts会依次调用为某个Action而注册的每一个拦截器的interecept方法`
- `每次调用interecept方法时,Struts会传递一个ActionInvocation接口的实例。`
- `ActionInvocation：代表一个给定Action的执行状态，拦截器可以从该类的对象里获得与该Action相关的Action对象和Result对象。在完成拦截器自己的任务之后，拦截器将调用ActionInvocation对象的invoke方法前进到Action处理提供了一个空白的实现。`
- `AbstractInterceptor类实现了Interceptor接口。并为init,destroy提供了一个空白的实现。`
```Java
package com.opensymphony.xwork2.interceptor;

import com.opensymphony.xwork2.ActionInvocation;

/**
 * Provides default implementations of optional lifecycle methods
 */
public abstract class AbstractInterceptor implements Interceptor {

    /**
     * Does nothing
     */
    public void init() {
    }
    
    /**
     * Does nothing
     */
    public void destroy() {
    }


    /**
     * Override to handle interception
     */
    public abstract String intercept(ActionInvocation invocation) throws Exception;
}

```

- `自定义拦截器`
	- `具体步骤`
		- `定义一个拦截器类`
			- `可以实现Interceptor接口`
			- `继承AbstractInterceptor抽象方法`
		- `在struts.xml文件配置`
		- `注意：在自定义的拦截器中可以选择不调用ActionInvocation的invoke()方法。那么后续的拦截器和Action方法将不会被调用Struts会渲染自定义拦截器intercept方法返回值对应的result。`
```Java
import com.opensymphony.xwork2.interceptor.AbstractInterceptor;
import com.opensymphony.xwork2.ActionInvocation;

public class MyInterceptor extends AbstractInterceptor {

    @Override
    public String intercept(ActionInvocation invocation) throws Exception {

		private static final long serialVersionUID = 1L;

		System.out.println("before invocation.invoke");

		String result = invocation.invoke();

		System.out.println("after invocation.invoke");

        return result;
    }
}

```
- `在struts.xml配置文件中添加拦截器`
```XML
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
	<!--配置国际化资源管理-->
	<constant name="struts.custom.i18n.resources" value="i18n"></constant>
    <!-- 包配置 -->
    <package name="default" namespace="/" extends="struts-default">


		<interceptors>
			<interceptor name="hello" class="MyInterceptor的全类名">
			</interceptor>
		</interceptors>

		<action name="testToken" class="TokenAction的全类名">
			<interceptor-ref name="hello"></interceptor-ref>
			<interceptor-ref name="token"></interceptor-ref>
			<interceptor-ref name="defaultStack"></interceptor-ref>
			<result>/success.jsp</result>
			<result name="invalid.token">token-error.jsp</result>
		</action>
    </package>
</struts>
```

`实例：`
`登录验证拦截器：`
```Java
import com.opensymphony.xwork2.interceptor.AbstractInterceptor;
import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;
import java.util.Map;

public class LoginInterceptor extends AbstractInterceptor {

    @Override
    public String intercept(ActionInvocation invocation) throws Exception {
        // 获取 ActionContext
        ActionContext context = invocation.getInvocationContext();
        // 获取 session
        Map<String, Object> session = context.getSession();
        
        // 检查用户是否登录（假设用户登录后在 session 中存储 "user" 对象）
        Object user = session.get("user");
        
        if (user == null) {
            // 用户未登录，跳转到登录页面
            return ActionSupport.LOGIN;
        }
        
        // 用户已登录，继续执行后续的拦截器或目标 Action
        return invocation.invoke();
    }
}
```
`struts.xml配置文件：`
```XML
<interceptors>
    <interceptor name="loginInterceptor" class="com.example.interceptor.LoginInterceptor" />
    <interceptor-stack name="defaultStack">
        <interceptor-ref name="loginInterceptor"/>
        <interceptor-ref name="defaultStack"/>
    </interceptor-stack>
</interceptors>

<action name="secureAction" class="com.example.action.SecureAction">
    <interceptor-ref name="defaultStack"/>
    <result name="success">/securePage.jsp</result>
    <result name="login">/login.jsp</result>
</action>

```
- `session检查：拦截器通过ActionContext 获取session并检查是否存在user对象`
- `未登录跳转：如果用户未登录，拦截器返回ActionSupprt.LOGIN，这会让Struts2跳转到登录页面`
- `已登录继续执行：如果用户已经登录，调用invocation.invoke()继续执行后续的拦截器或目标Action`
