# ssm-crud
---

## 总体的设计思路：

（1）总体功能点：

①分页

②数据校验（jquery前端校验+JSR303后端校验）

③ajax

④Restful的URI：使用HTTP协议请求方式的动词来表示对资源的操作（GET--查询,POST--新增,PUT--修改,DELETE--删除）

（2）技术点：

①基础框架--ssm（SpringMVC+Spring+MyBatis）

②数据库--MySql

③前端框架--Bootstrap

④项目的依赖管理--Maven

⑤分页--pagehelper

⑥逆向工程--MyBatis Generator

## 项目运行说明：（开发环境：MyEclipse2017 + Tomcat8.0）

1、以Maven方式导入项目：Import -> Maven -> Existing maven project，选中项目导入。

2、将dbconfig.properties文件中的password改成自己的Mysql密码，mbg.xml也一样。

3、在本地新建数据库ssm_crud（字符集设为utf8）。

4、参考eclipse中maven项目部署到tomcat：https://www.cnblogs.com/iahui/articles/6099832.html

这里采用的方式是在pom.xml中加入如下配置：

	<build> 		
		<finalName>ssm-crud</finalName>  
		    <plugins>
		    	<plugin>
			      <groupId>org.apache.maven.plugins</groupId>
			      <artifactId>maven-compiler-plugin</artifactId>
			      <configuration>
			        <source>1.8</source>
			        <target>1.8</target>
			      </configuration>
			    </plugin>
			    <plugin>  
			      <groupId>org.apache.tomcat.maven</groupId>
			      <artifactId>tomcat7-maven-plugin</artifactId>
			      <version>2.2</version>
			      <configuration>  
			           <!-- 远程tomcat下manager路径 -->
			        <url>http://localhost:8080/manager/text</url>      
			        <server>tomcat8</server>    
			      </configuration>                  
			    </plugin>
		    </plugins>
	</build> 

5、运行redeploy命令前，要启动tomcat，并能正常访问http://localhost:8080/manager（若eclipse启动tomcat, 但http://localhost:8080无法访问，参考：https://www.cnblogs.com/zhangboy/p/6877764.html）

通过项目右键 run as --> maven build... --> main --> goals 中填入 tomcat8:deploy命令即可部署成功（遇到更多的部署方面的问题，可以参考：http://blog.csdn.net/u012052168/article/details/52448943）

## 开发过程记录：

（1）基础环境搭建：

①创建一个Maven工程:新建Maven Project -> 勾选create a simple project -> 打包方式改为war包，然后finish。建好工程后，右键项目点击Properties，选中Project Facets，取消Dynamic Web Module的勾选，然后Apply，然后再重新勾选，点击下面出现的futher选项，在弹出的界面勾选Generate web.xml并将Content directory改为src/main/webapp即可。

②引入项目依赖的jar包（部分）

· spring

· springmvc

· MyBatis

· 数据库连接池，驱动包

· 其它（jstl， servlet-api， junit）

③引入Bootstrap前端框架（下载用于生产环境的源码），在src/main/webapp目录下新建一个static文件夹，存放刚刚下载的代码文件。在webapp下新建一个index.jsp，引入bootstrap的样式。还要引入Jquery，同理，在static文件夹下新建一个js文件夹，存放jquery.js。

④编写ssm整合的关键配置文件：web.xml、spring,springmvc,mybatis的配置文件。
（下面步骤中属于Spring配置的步骤有：1、2、5，
属于SpringMVC配置的步骤有：3、4，
属于MyBatis配置的步骤有5、6。）

<1> 编写web.xml文件：

	<!--1、启动Spring的容器  -->
	<!-- needed for ContextLoaderListener -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<!-- Bootstraps the root web application context before servlet initialization -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

<2>	新建Spring Bean Configuration File:命名为applicationContext.xml文件 （到此为止，Spring的配置文件已创建完成）

<3>	编写web.xml（配置springmvc和Springmvc完成的字符串编码的过滤器，对应于web.xml的2和3）

	<!--2、springmvc的前端控制器，拦截所有请求  -->
	<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
		<!-- <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>location</param-value>
		</init-param> -->
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
	注意：由于省略了contextConfigLocation参数的配置，因此要在与web.xml
	同级的目录下新建一个名为{servlet-name}-servlet.xml的配置文件（要注意名字格式）

<4>	编写dispatcherServlet-servlet.xml文件（注意use-default-filters="false"，因为不使用默认的filter，而是使用自己设置的include-filter）：

	<!--SpringMVC的配置文件，包含网站跳转逻辑的控制，配置  -->
	<context:component-scan base-package="com.zyuxuan" use-default-filters="false">
		<!--只扫描控制器。  -->
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
	
	<!--配置视图解析器，方便页面返回  -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
	
	<!--两个标准配置  -->
	<!-- 将springmvc不能处理的请求交给tomcat，如动态图片资源等 -->
	<mvc:default-servlet-handler/>
	<!-- 能支持springmvc更高级的一些功能，JSR303校验，快捷的ajax...映射动态请求 -->
	<mvc:annotation-driven/>
	
<5>	编写applicationContext.xml（重点），并在类路径下创建mybatis-config.xml文件:

	<context:component-scan base-package="com.zyuxuan">
		<context:exclude-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>

	<!-- Spring的配置文件，这里主要配置和业务逻辑有关的 -->
	<!--=================== 数据源，事务控制，xxx ================-->
	<context:property-placeholder location="classpath:dbconfig.properties" />
	<bean id="pooledDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
		<property name="driverClass" value="${jdbc.driverClass}"></property>
		<property name="user" value="${jdbc.user}"></property>
		<property name="password" value="${jdbc.password}"></property>
	</bean>

	<!--================== 配置和MyBatis的整合=============== -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 指定mybatis全局配置文件的位置 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"></property>
		<property name="dataSource" ref="pooledDataSource"></property>
		<!-- 指定mybatis，mapper文件的位置 -->
		<property name="mapperLocations" value="classpath:mapper/*.xml"></property>
	</bean>
	
	<!-- 配置扫描器，将mybatis接口的实现加入到ioc容器中 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!--扫描所有dao接口的实现，加入到ioc容器中 -->
		<property name="basePackage" value="com.zyuxuan.crud.dao"></property>
	</bean>
	
	<!-- ===============事务控制的配置 ================-->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<!--控制住数据源  -->
		<property name="dataSource" ref="pooledDataSource"></property>
	</bean>
	<!--开启基于注解的事务，使用xml配置形式的事务（比较主要的都是使用配置式）  -->
	<aop:config>
		<!-- 切入点表达式 -->
		<aop:pointcut expression="execution(* com.zyuxuan.crud.service..*(..))" id="txPoint"/>
		<!-- 配置事务增强 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
	</aop:config>
	
	<!--配置事务增强，事务如何切入  -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!-- 所有方法都是事务方法 -->
			<tx:method name="*"/>
			<!--以get开始的所有方法  -->
			<tx:method name="get*" read-only="true"/>
		</tx:attributes>
	</tx:advice>
	
	注意：<context:property-placeholder>用来引入外部文件配置项;
	<aop:advisor>的advice-ref与<tx:advice>的id一致；
	<aop:advisor>的pointcut-ref与<aop:pointcut>的id一致。
	
<6>	编写mybatis-config.xml(参考官方文档)：

	<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
	<configuration>
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	</settings>
	
	<typeAliases>
		<package name="com.zyuxuan.crud.bean"/>
	</typeAliases>
	
	<plugins>
		<plugin interceptor="com.github.pagehelper.PageInterceptor">
			<!--分页参数合理化  -->
			<property name="reasonable" value="true"/>
		</plugin>
	</plugins>

	</configuration>
	
⑤为使用Rest风格的URI,在web.xml中配置filter（对应于web.xml中的4）

============截止目前为止，SSM基本整合完成（后面还需要mybatis的逆向工程等步骤）=====================

（2）在MySql的ssm_crud数据库建表(大致语句如下，使用navicat修改)：

①建tbl_emp表（含外键）：

方法一（直接）：create table tbl_emp(emp_id int(11) primary key auto_increment,emp_name varchar(255) not null,gender char(1),email varchar(255),d_id int(11),foreign key(d_id) references tbl_dept(dept_id));

方法二（创建外键约束关系）：

create table tbl_emp(emp_id int(11) primary key auto_increment,emp_name varchar(255) not null,gender char(1),email varchar(255),d_id int(11));

alter table tbl_emp add constraint fk_emp_dept foreign key(d_id) references tbl_dept (dept_id);

②建tbl_dept表

tbl_dept表的建立类似（见sql文件）

（3）使用mybatis的逆向工程（mybatis generator）生成对应的bean以及mapper：

①在pom.xml引入mybatis generator的jar包

②新建mbg.xml文件并编写（参考官方文档：http://www.mybatis.org/generator/quickstart.html）

③运行逆向工程（参考官方文档Running Mybatis Generator With Java:http://www.mybatis.org/generator/running/runningWithJava.html）:在crud.test包中编写MBGTest类。

④对EmployeeMapper.xml等文件进行一些修改或者补充（若补充了新的配置，则相应的要在EmployeeMapper.java中新建方法）：

	1）在EmployeeMapper.java补充方法：
	
	List<Employee> selectByExampleWithDept(EmployeeExample example);

    Employee selectByPrimaryKeyWithDept(Integer empId);

	2）在Employee.java中添加属性Department以及getter、setter方法：
	
    private Department department;//希望查询员工的同时部门信息也是查询好的
	
	3）在EmployeeMapper.xml中补充配置（重点，包含多对一配置等多个知识点）：
	
	<sql id="WithDept_Column_List">
  		e.emp_id, e.emp_name, e.gender, e.email, e.d_id,d.dept_id,d.dept_name
  	</sql>
	
	<!-- 查询员工同时带部门信息 -->
  	<select id="selectByExampleWithDept" resultMap="WithDeptResultMap">
	   	select
	   <if test="distinct">
	     distinct
	   </if>
	   <include refid="WithDept_Column_List" />
		FROM tbl_emp e
		left join tbl_dept d on e.`d_id`=d.`dept_id`
	   <if test="_parameter != null">
	     <include refid="Example_Where_Clause" />
	   </if>
	   <if test="orderByClause != null">
	     order by ${orderByClause}
	   </if>
	 </select>
	 <select id="selectByPrimaryKeyWithDept" resultMap="WithDeptResultMap">
	   select 
	   <include refid="WithDept_Column_List" />
	    FROM tbl_emp e
		left join tbl_dept d on e.`d_id`=d.`dept_id`
	    where emp_id = #{empId,jdbcType=INTEGER}
	 </select>
	 
	 <resultMap type="com.zyuxuan.crud.bean.Employee" id="WithDeptResultMap">
	  	<id column="emp_id" jdbcType="INTEGER" property="empId" />
	    <result column="emp_name" jdbcType="VARCHAR" property="empName" />
	    <result column="gender" jdbcType="CHAR" property="gender" />
	    <result column="email" jdbcType="VARCHAR" property="email" />
	    <result column="d_id" jdbcType="INTEGER" property="dId" />
	    <!-- 指定联合查询出的部门字段的封装 -->
	    <association property="department" javaType="com.zyuxuan.crud.bean.Department">
	    	<id column="dept_id" property="deptId"/>
	    	<result column="dept_name" property="deptName"/>
	    </association>
	  </resultMap>
	  
（4）测试配置完上面的东西之后dao层是否已经成功工作（即测试mapper），需要搭建Spring单元测试环境：

在crud.test包建立一个MapperTest.java用于测试，内容见文件，推荐Spring的项目使用Spring的单元测试，可以自动注入我们需要的组件，搭建步骤如下：
 
①导入SpringTest模块
 
②@ContextConfiguration指定Spring配置文件的位置

③直接autowired要使用的组件即可

完成上述步骤后编写测试代码，运行，由于下面代码起作用（更多测试代码看文件），数据库中的表自动创建数据：

departmentMapper.insertSelective(new Department(null, "开发部"));

departmentMapper.insertSelective(new Department(null, "测试部"));

注意：在测试代码中若要批量插入多个员工，可以执行批量操作的sqlSession，方法如下：

①先在application.xml中配置一个可以执行批量的sqlSession：

	<!-- 配置一个可以执行批量的sqlSession -->
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
		<constructor-arg name="executorType" value="BATCH"></constructor-arg>
	</bean>

②测试代码中注入sqlSession，并批量插入数据（重点）：

	@Autowired
	SqlSession sqlSession;

	EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
	for(int i = 0;i<1000;i++){
		String uid = UUID.randomUUID().toString().substring(0,5)+i;
		mapper.insertSelective(new Employee(null,uid, "M", uid+"@zyuxuan.com", 1));
	}
	System.out.println("批量完成");
	
================截至目前为止，dao层基本完成====================

（5）首页要能够展示数据，做法如下：

①访问index-2.jsp

②index-2.jsp页面发送出查询员工列表请求：

<jsp:forward page="/emps"></jsp:forward>

③EmployeeController来接受请求，查处员工数据
（相应的，在WEB-INF/views下创建一个list.jsp显示数据;还要创建EmployeeService类并编写相应的方法；
里面还用到了PageHelper分页插件（要引入jar包，并在mybatis-config.xml中配置plugin节点））：

@Controller
public class EmployeeController {

	@Autowired
	EmployeeService employeeService;

	@RequestMapping("/emps")
	public String getEmps(
			@RequestParam(value = "pn", defaultValue = "1") Integer pn,
			Model model) {
		// 这不是一个分页查询；
		// 引入PageHelper分页插件
		// 在查询之前只需要调用，传入页码，以及每页的大小
		PageHelper.startPage(pn, 5);
		// startPage后面紧跟的这个查询就是一个分页查询
		List<Employee> emps = employeeService.getAll();
		// 使用pageInfo包装查询后的结果，只需要将pageInfo交给页面就行了。
		// 封装了详细的分页信息,包括有我们查询出来的数据，传入连续显示的页数(这里是5)
		PageInfo page = new PageInfo(emps, 5);
		model.addAttribute("pageInfo", page);

		return "list";
	}
	
写完上面的控制器代码之后不必急于显示到页面上看效果，而是应该编写Spring单元测试（crud.test包下的MvcTest类，注意Spring单元测试需要servlet3.0的jar包）：

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "classpath:applicationContext.xml",
		"file:src/main/webapp/WEB-INF/dispatcherServlet-servlet.xml" })
public class MvcTest {
	// 传入Springmvc的ioc
	@Autowired
	WebApplicationContext context;
	// 虚拟mvc请求，获取到处理结果。
	MockMvc mockMvc;

	@Before
	public void initMokcMvc() {
		mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
	}

	@Test
	public void testPage() throws Exception {
		//模拟请求拿到返回值
		MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get("/emps").param("pn", "5"))
				.andReturn();
		
		//请求成功以后，请求域中会有pageInfo；我们可以取出pageInfo进行验证
		MockHttpServletRequest request = result.getRequest();
		PageInfo pi = (PageInfo) request.getAttribute("pageInfo");
		System.out.println("当前页码："+pi.getPageNum());
		System.out.println("总页码："+pi.getPages());
		System.out.println("总记录数："+pi.getTotal());
		System.out.println("在页面需要连续显示的页码");
		int[] nums = pi.getNavigatepageNums();
		for (int i : nums) {
			System.out.print(" "+i);
		}
		
		//获取员工数据
		List<Employee> list = pi.getList();
		for (Employee employee : list) {
			System.out.println("ID："+employee.getEmpId()+"==>Name:"+employee.getEmpName());
		}
		
	}

}

④跳转到了list.jsp页面进行展示：

首先要注意web路径的写法（重点）：

不以/开始的相对路径，找资源，以当前资源的路径为基准，经常容易出问题。

以/开始的相对路径，找资源，以服务器的路径为标准(http://localhost:8080)；需要加上项目名
		http://localhost:8080/crud
		
应用如下（APP_PATH的值是 /ssm_crud）：

<%
	pageContext.setAttribute("APP_PATH", request.getContextPath());
%>

<script type="text/javascript" src="${APP_PATH }/static/js/jquery-1.12.4.min.js"></script>

详细代码见list.jsp，注意的问题：

1>	用taglib引入jstl的core（用到<c:forEach>和<c:if>）

2>	用到了PageInfo对象里的多个属性（如pages,hasNextPage等）

（6）由于步骤（5）的做法只适用于B/S模型，若安卓或iOS发送请求也返回一个页面
是不合适的，应改成返回json格式的数据，同时发送请求的方式改成ajax，步骤如下：

①index.jsp页面（上一种查询方法是用index-2.jsp页面）直接发送ajax请求进行
员工分页数据的查询

②服务器将查出的数据，以json字符串的形式返回给浏览器（或者其它安卓等客户端，
使用json可以实现客户端的无关性）

③浏览器收到js字符串，可以使用js对json进行解析，并使用js通过dom增删改
改变页面。

具体实现过程如下：

①修改EmployeeController类（使用@ResponseBody可以把方法返回的对象转成json字符串（要导入jackson包）），
里面使用到了crud.bean包里的Msg类（注意add方法的写法是便于Msg的链式操作）。

②编写index.jsp（发送ajax请求，解析返回的json数据，这里的重点是JQuery代码）

（7）在index.jsp完成新增员工功能,步骤：

①点击“新增”，弹出新增对话框（使用bootstrap的button的data-toggle属性和data-target属性以及bootstrap对应的模态框）
（要保证字段的name属性名与JavaBean的属性名一一对应，SpringMVC才会把用户提交的数据自动封装成Employee对象）

②去数据库查询部门列表，显示在对话框（应该在弹出模态框之前就要发送ajax请求，这里就要编写处理ajax请求的DepartmentController等类）

③用户输入数据完成保存（模态框中的数据要提交到服务器保存（保存之前还要校验），编写EmployeeController的方法）

注意URI的Rest风格：

/emp/{id}	GET		查询员工

/emp		POST	保存员工

/emp/{id}	PUT		修改员工

/emp/{id}	DELETE	删除员工

补充：校验的方法如下（每个步骤都有很多细节）：

①前端校验：正则表达式

②姓名查重：在EmployeeController中使用EmployeeExample和Criteria
等类

③后台JSR303校验：

1>	导入Hibernate-Validator包（注意tomcat6以及以下的服务器要额外给服务器的lib包中替换支持el的新标准的包）

2>	在Employee类的属性上添加注解（如@Pattern，@Email等）

3>	在EmployeeController的保存方法中添加参数：@Valid Employee employee和BindingResult result

④数据库唯一约束（数据库创建约束使用户名不能重复）











  	