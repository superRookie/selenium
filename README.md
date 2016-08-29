#PatatiumWebUi
<h1>简介</h1>
 **这是一个webui自动化测试框架，由<a>webdriver中文社区</a>创办人土豆所创建故取名patatiumWebui,该web自动化测试框架是用java语言编写的，基于selenium webdriver 的开源自动化测试框架，该框架结合了testng,selenium,webdriver，jxl，jodd-http 等工具。该框架基于页面对象模型（POM）模型架构，实现了关键字驱动技术，数据驱动,无需掌握多少编程知识即可编写脚本，同时实现了数据与代码分离的功能：1、元素定位信息保存在对象库文件中 2、测试用例数据可以存储在excel中。从而实现，页面元素位置变化，无需改动脚本，只需修改对应的元素定位信息即可。
目前框架还不是特别完善，还需要写一些脚本实现自动化；学习该框架需要熟悉一定的HTML 和java基础，后续可以考虑自动编码的实现。**

<h1>Demo演示</h1>
<h2>1、对象库文件编写(文件名定义为UILibrary.xml)</h2>
```
<?xml version="1.0" encoding="UTF-8"?>
<!--整个对象库文件的根目录，管理整个项目的对象-->
<map>
    <!--管理一个页面的元素（webelement：input,select,textare,a,li等标签），一个page包含多个locator对象
    Pagename:page对象名字，格式：net.hk515.PageObject.xxxPage;最后面那位才是真正的页面名字，前面的是java对象库路径；
    另外注意，页面名字是头个单词大写；例如主页：名字定义为 net.hk515.PageObject.HomePage
    Value：页面对象的URL，可不填。
    Desc:页面对象中文描述-->
    <page pagename="org.webdriver.patatiumwebui.pageObject.LoginPage" value="" desc="京东登录页面">
        <!--管理一个页面的元素（webelement：input,select,textare,a,li等标签），一个page包含多个locator对象
        Type：定位方式，包含id,name,class,linktext,xpath,css等，定位元素的时候灵活使用，一般可以统一用xpath
        代替id,name,class，linktext的定位方式。
        Timeout：元素加载时间，有些页面元素，可能要等待一段时间才能加载过来，为了查找元素的稳定性，需加等待时间。
        Value:元素定位信息，如果是id,name,class，linktext直接把网页元素对应的这些属性值写上即可，如果是xpath定位方式，
        需要填写正确的xpath语法格式。
        Desc:元素的描述，元素的中文描述信息-->
		<locator type="xpath" timeout="3" value="//input[@id='loginname']"  desc="用户名">用户名输入框</locator>
		<locator type="id" timeout="3" value="nloginpwd"  desc="密码">密码输入框</locator>
		<locator type="id" timeout="3" value="loginsubmit"  desc="登录">登录按钮</locator>
	</page>
</map>
```
对象库文件编写后，运行/src/main/java/org/webdriver/patatiumwebui/PageObjectConfig/PageObjectAutoCode.java 文件生成对象库java代码
<h2>2、公共action封装实例（业务操作）</h2>
```
package org.webdriver.patatiumwebui.action;

import org.webdriver.patatiumwebui.pageObject.LoginPage;
import org.webdriver.patatiumwebui.utils.ElementAction;
import org.webdriver.patatiumwebui.utils.TestBaseCase;

import java.io.IOException;

/**
 * Created by zhengshuheng on 2016/8/29 0029.
 */
public class LoginAction extends TestBaseCase{
    public LoginAction(String Url,String UserName,String PassWord) throws IOException
    {
        //此driver变量继承自TestBase变量
        LoginPage loginPage=new LoginPage();
        loginPage.open(Url);
        System.out.println(driver.getCurrentUrl());
        ElementAction action=new ElementAction();
        action.clear(loginPage.密码输入框());
        action.type(loginPage.用户名输入框(),UserName);
        action.clear(loginPage.密码输入框());
        action.type(loginPage.密码输入框(),PassWord);
        action.click(loginPage.登录按钮());
    }
}

```
公共Action代码放在src/main/java/org/webdriver/patatiumwebui/Action 包下
<h2>3、驱动数据来源实例</h2>
1、在src/main/resources/data下创建loginData.xml文件
编写如下内容
![输入图片说明](http://git.oschina.net/uploads/images/2016/0829/123627_cb6607c8_482055.png "在这里输入图片标题")
<h2>4、测试用例编写</h2>
普通测试用例：
```
@Test(description="登录成功测试")
	@Parameters({"BaseUrl"})//读取testng.xml参数
	public void login(String BaseUrl) throws IOException
	{
		//调用登录方法，输入正确的用户名和密码
		LoginAction loginAction=new LoginAction(BaseUrl+"/new/login.aspx","13026696420","zheng15970066750");
		action.sleep(2);
		//设置检查点
		Assertion.VerityTextPresentPrecision("jd_8456195","输入正确的用户名和密码，验证是否成功进入主页");
		//设置用例断言，判断用例是否失败
		Assertion.VerityError();
	}
```
数据驱动测试用例：
```
//数据驱动案例--start
	@DataProvider(name="longinData")
	public Object[][] loginData()
	{
		//读取登录用例测试数据
		String filePath="src/main/resources/data/loginData.xls";
		//读取第一个sheet，第2行到第5行-第2到第4列之间的数据
		return ExcelReadUtil.case_data_excel(0, 1, 4, 1, 3,filePath);
	}
	@Test(description="登录失败用例",dataProvider = "longinData")
	public void loginFail (String userName,String password,String message) throws IOException, DocumentException {
		//代替testng参数化的方法
		String BaseUrl= XmlReadUtil.getTestngParametersValue("testng.xml","BaseUrl");
		//调用登录方法
		LoginAction loginAction=new LoginAction(BaseUrl+"/new/login.aspx",userName,password);
		action.sleep(1);
		//设置检查点
		Assertion.VerityTextPresent(message,"验证是否出现预期的错误提示信息:"+message);
		//设置断言
		Assertion.VerityError();
	}
	//数据驱动案例--end
```
测试用例代码放在src/test/java 包下
<h2>5、testng.xml配置</h2>
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" >
	<parameter name="driver" value="FirefoxDriver" /> <!--测试浏览器：支持火狐，谷歌，IE-->
	<parameter name="nodeURL" value="" /> <!--selenium grid分布式运行node节点url，如不用分布式运行，则留空-->
	<parameter name="BaseUrl" value="https://passport.jd.com" />  <!-- 测试系统基础Url-->
	<parameter name="UserName" value="" /> <!-- 系统登录用户名-->
	<parameter name="PassWord" value="" />  <!-- 系统登录密码-->
    <parameter name="smtpUserName" value="zhengshuheng@hk515.com" />  <!-- 测试报告邮件发送：smtp身份证验证-->
    <parameter name="smtpPassWord" value="zheng@159791" />  <!-- 测试报告邮件发送：smtp身份证验证-->
    <parameter name="smtpHost" value="smtp.hk515.com" />  <!-- 测试报告邮件发送：smtp主机地址-->
    <parameter name="smtpPort" value="25" />  <!-- 测试报告邮件发送：smtp主机端口-->
    <parameter name="mailTitle" value="Webdriver中文社区-自动化测试报告" />  <!-- 测试报告邮件发送：邮件标题-->
    <parameter name="logUrl" value="" />  <!-- 测试报告邮件发送：用例运行日志url-->
    <parameter name="reportUrl" value="" />  <!-- 测试报告邮件发送：完整测试报告url-->
	<parameter name="recipients" value="609958331@qq.com" /> <!-- 测试报告邮件发送：收件人，多个用,号隔开-->
    <parameter name="reportTitle" value="Webdriver中文社区-自动化测试报告" />  <!--测试报告标题-->
	<listeners><!-- 监听器设置-->
        <listener class-name="org.webdriver.patatiumwebui.utils.TestListener"></listener>
        <listener class-name="org.webdriver.patatiumwebui.utils.TestReport"></listener>
    </listeners>
     <test name="登录失败测试用例：数据驱动"> <!-- 测试用例描述-->
    <classes>
      <class name="LoginTest">
      	     <methods >
                   <include name="loginFail" />
             </methods>
       </class>
    </classes>
  </test> <!-- Test -->
    <test name="登录成功测试用例">
        <classes>
            <class name="LoginTest">
                <methods >
                    <include name="login" />
                </methods>
            </class>
        </classes>
    </test> <!-- Test -->
</suite> <!-- Suite -->

```
testng.xml放在项目根目录下面。
<h2>6、执行用例</h2>
IDE：在IDE集成开发环境下右键testng.xml使用testng运行
Maven:执行mvn clean ;mvn test 命令
Jenkins：1、checkout 项目代码 2、指定pom.xml文件  3、执行mvn clean ;mvn test 命令
<h2>7、查看测试报告及日志文件</h2>
用例执行完毕，会自动发送邮件报告及生成测试报告文件
测试报告文件生成在项目根目录下test-out目录下report.html文件
报告展示如下：
![输入图片说明](http://git.oschina.net/uploads/images/2016/0829/135306_b9ddfe80_482055.jpeg "在这里输入图片标题")
邮件展示如下：
![输入图片说明](http://git.oschina.net/uploads/images/2016/0829/135522_d205f0dd_482055.png "在这里输入图片标题")
日志文件展示如下：
![输入图片说明](http://git.oschina.net/uploads/images/2016/0829/135713_8d369238_482055.jpeg "在这里输入图片标题")

下面给大家简单讲解下，该框架的使用。（使用该框架之前首先要做的是环境搭建，环境搭建比较简单，在此就不介绍了）

第一步：创建XML对象库（编写xml对象库文件）
<?xml version="1.0" encoding="UTF-8"?>
<map>	
	<page pagename="net.hk515.PageObject.LoginPage"value="http://192.168.0.21:8086/User/Login" desc="华康运营后台登录页面">
		<locator type="id" timeout="3" value="userName"  desc="用户名">userName</locator>
		<locator type="id" timeout="3" value="password"  desc="密码">password</locator>
		<locator type="id" timeout="3" value="loginButton"  desc="登录">loginButton</locator>
	</page>
</map>
说明：
1、<map>标签是整个对象库文件的根目录，管理整个项目的对象。
2、<page>标签管理一个页面的元素（webelement：input,select,textare,a,li等标签）。一个page包含多个locator对象
3、<locator> 标签管理一个元素对象的信息。一个locator对象包含 定位方式(type),元素加载等待时间（timeout,秒），元素定位信息（value）,元素对象描述（desc）,元素对象名称（locator标签的文本值）
4、<locator>标签对象属性详解：
 Type：定位方式，包含id,name,class,linktext,xpath,css等，定位元素的时候灵活使用，一般可以统一用xpath 代替id,name,class，linktext的定位方式
 Timeout：元素加载时间，有些页面元素，可能要等待一段时间才能加载过来，为了查找元素的稳定性，需加等待时间。
 Value:元素定位信息，如果是id,name,class，linktext直接把网页元素对应的这些属性值写上即可，如果是xpath定位方式，需要填写正确的xpath语法格式(后续给大家详解)
 Desc:元素的描述，元素的中文描述信息
 5、<page>标签对象属性详解：
Pagename:page对象名字，格式：net.hk515.PageObject.xxxPage;最后面那位才是真正的页面名字，前面的是java对象库路径；另外注意，页面名字是头个单词大写；例如主页：名字定义为 net.hk515.PageObject.HomePage
Value：页面对象的URL，可不填
Desc:页面对象中文描述


第二步：运行PageObjectAutoCode类，把xml对象库转化为java文件对象库（自动按页面生成页面对象类文件存放在PageObject包下）


第三步：编写测试脚本。具体步骤如下
  1、在net.hk515.Test包目录下，创建测试类XXXTest，一般以模块划分，比如：登录测试：LoginTest.java
主页测试：HomePageTest.java  账户管理测试：AccountMangerTest.java
  2、编写测试代码
 
结构如下：
   Public HomePageTest extends TestBaseCase  ---测试类名
   {
      HomePage  homePage =new HomePage();---创建HomPage对象，用于后面调用HomePage对象库的元素信息
      ElementAction action =new ElmentAction();---创建操作页面元素的对象，用于后面调用操作元素的方法
      @BeforeClass()---运行本类用例之前必须执行的方法
	  @Parameters({"Base_Url","UserName","PassWord"})---读取testng.xml配置的项目地址，用户名和密码
	  public void beforetest(String Base_Url,String UserName,String PassWord)
	 {
		log.info("输入用户名密码进入后台主页");	---控制台日志输出	
		CommonAction.Login(Base_Url+"/login/login", UserName, PassWord);---调用公共登录方法
		action.sleep(5);---等待暂停操作3秒
	 }
    @Test(description="主页--预约挂号菜单",priority=1)---用例描述
	public void checkAppiont_memue()---用例方法名
	{
		action.click(homePage.appiont_memue());---单击主页面的预约挂号菜单（appiont_memue）
		action.sleep(3);---等待暂停操作3秒
		Assertion.VerityTextPresent("按医院挂号","点击预约挂号菜单是否成功进入预约挂号页面");---设置检查点，检查单击预约挂号菜单后，页面是否出现按医院挂号文字。如果页面报错，无法访问，那么就找不到该文字，用例就会标记为失败(failed)
		Assertion.VerityError();---判断用例是否有错误的检查点，包含1个以上的错误检查点，用例就标记为失败(failed)
		
	}
   }

第四步：配置用例执行顺序testng.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="none">
	<parameter name="driver" value="FirefoxDriver"></parameter>---设置浏览器类型
	<parameter name="nodeURL" value=""></parameter>---selenium grid分布式运行用例节点ip地址，如果不采用grid分布式运行，此参数设置为空
         //项目URL
	<parameter name="Base_Url" value="http://shenzhen.call.hk515.com/login/login"></parameter>
	<parameter name="UserName" value="hljadmin"></parameter>--项目登录的用户名
	<parameter name="PassWord" value="111111"></parameter>---项目登录的密码
	<listeners>---监听器，固定填写
        <listener class-name="net.hk515.utils.TestListener"></listener>
        <listener class-name="net.hk515.utils.TestReport"></listener>
    </listeners>
   <test name="账号注册/查询/编辑/注销模块类">--测试集名字
    <classes>
      <class name="net.hk515.Test.AppointmentTest">---测试类名
      	     <methods preserve-order="true">
      	  <include name="checkAppiont_memue" desc="检查预约挂号菜单"/>--测试类方法，检查预约挂号菜单
            </methods>
       </class>
    </classes>
  </test> <!-- Test -->  
</suite> <!-- Suite -->

第五步：运行testng.xml,执行用例

第六步：查看运行结果


用例运行错误自动截图




Xpath 详解：
注：可通过火狐浏览器安装,firebug,firepath插件校验xpath的正确性
先举个xpah例子://div[@id=’abc’]/form/div/input/span
//：从匹配选择的当前节点，选择文档中的节点，不考虑它的具体位置，例如：//div[@name=‘abc’]
查找页面中name属性为abc的div标签
/：从根节点选取元素，例如：/html/body/div[@id='myModalex'] 
可以是文档最根节点开始查找元素，也可以是配陪得节点为根节点往下找
例如：//*[@id='loginForm']/div[1]/label
@：@表示属性 属性可以用and,or运算符
例如：//label[@class='col-sm-2 control-label' and @for='userName'] 在定位中，如果一个属性还不能精确定位某个元素那么则可以再组合增加一个元素，使定位达到唯一性
Text（）：通过元素的文本值查找元素，例：//h2[text()='华康移动医疗客服后台']
Contains();//input[contains(@id,'nt')] 模糊匹配，查找id包含nt的input标签
//h2[contains(text(),'华康移动医疗客服后台')] 查找文本值包含华康移动医疗客服后台的元素
//灵活使用案例：
查找元素<span class=”cde”>八佰伴</span>
<span class=”cde”>嘎嘎嘎</span>
<div id=”abc”>
   <form>
            <div>
                   <input>
                      <span class=”cde”>八佰伴</span>
                   </input>
            </div>
   </form>
<div>
分析：该元素，没有唯一性的id，name等标签，并且层级多，上一级也没有唯一性的东西，只能从上上上级开始查找元素。但是从上上级查找元素，xpath的层级多，定位信息复杂，那么有没有办法优化精简呢？答案是肯定的，利用//可以大幅优化精简xpath表达式
方案一：//div[@id=’abc’]/form/div/input/span
方案二：//*[@id=’abc’]/form/div/input/span[@class=’cde’]
方案三：//span[@class=’cde’][2]
方案四：//div[@id=’abc’]//span[@class=’cde’]--此方法最简洁，结构也最清晰，也最稳定

综上xpath定位原则，元素id,name属性优先使用，其次是class等其他，1、在当前节点没有id,name等属性确定元素唯一性的时候，往上找，通过当前节点父亲，祖父，祖父的父亲，祖父的祖父等节点查找当前元素。2、一个元素属性不足够定位当前元素的时候，可以通过and运算符，组合属性来定位使之达到唯一性，尽可能的缩短xpath层级，使xpath定位更稳定。

Firebug使用：
例：
定位用户名输入框可以用//*[@id=’userName’]表示查找当前页面下，id属性为’userName’的所有元素，相当于id定位方式。*代表所有元素。