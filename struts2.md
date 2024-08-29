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