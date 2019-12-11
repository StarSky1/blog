[pixiv: 024]: # 'https://cdn.jsdelivr.net/gh/starsky1/poi/2019/24.png'
# 什么是tk-mybatis
通用 Mapper4（tk-mybatis） 是一个可以实现任意 MyBatis 通用方法的框架，项目提供了常规的增删改查操作以及Example 相关的单表操作。通用 Mapper 是为了解决 MyBatis 使用中 90% 的基本操作，使用它可以很方便的进行开发，可以节省开发人员大量的时间。

项目WiKi地址：https://gitee.com/free/Mapper/wikis/Home

# 如何在Maven项目中使用tk-mybatis
## 第一步，添加依赖
首先创建一个Maven项目，然后在`pom.xml`文件中加入以下依赖项：
1. 添加mybatis3.x依赖和tk-mybatis4.x依赖以及mysql数据库连接驱动
```xml
	<!-- 添加tk-mybatis数据库工具 -->
      <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis</artifactId>
          <version>3.4.6</version>
      </dependency>

      <!-- https://mvnrepository.com/artifact/tk.mybatis/mapper -->
      <dependency>
          <groupId>tk.mybatis</groupId>
          <artifactId>mapper</artifactId>
          <version>4.1.5</version>
      </dependency>
	  
	  <!-- mysql数据库连接驱动 -->
	  <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.44</version>
        </dependency>
```
2. 添加dbcp数据库连接池（后面创建数据源会用到）
```xml
		<!--注意：在linux环境下，commons-logging1.2好像无法找到里面的/LogFactory，但commons-logging1.1.3可以-->
        <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.1.3</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-dbcp2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-dbcp2</artifactId>
            <version>2.5.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.4.2</version>
        </dependency>
```
3. 添加log4j2日志工具（也可使用其他日志框架，不强求）
```xml
		<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.11.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.11.1</version>
        </dependency>
```
## 第二步，创建mybatis-config.xml文件
1. 在项目的`src/main/resources`路径下，新建一个`mybatis-config.xml`文件，内容如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 引入外部资源文件 -->
    <!--<properties resource="jdbc.properties"/>-->

    <settings>
        <!-- log4j.properties的配置，用于打印SQL语句及结果到控制台，便于调试 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
        <setting name="callSettersOnNulls" value="true"/>
        <!--将数据库字段字段映射成java对象的驼峰命名-->
        <setting name= "mapUnderscoreToCamelCase" value="true" />
    </settings>
    <typeAliases>
    </typeAliases>

    <!-- 配置环境：可以配置多个环境，default：配置某一个环境的唯一标识，表示默认使用哪个环境 -->
    <environments default="phm">
        <environment id="phm">
            <transactionManager type="JDBC"/>
            <dataSource type="com.yj.config.DBCPDataSourceFactory">
                <!-- 配置连接信息 -->
                <!--<property name="driver" value="${jdbc.driverClass}"/>-->
                <!--<property name="url" value="${jdbc.connectionURL}"/>-->
                <!--<property name="username" value="${jdbc.username}"/>-->
                <!--<property name="password" value="${jdbc.password}"/>-->
            </dataSource>
        </environment>
    </environments>
    <!-- 映射map -->
    <mappers>
        <!--mapper的方式是配置mapper.xml文件-->
        <mapper resource="mapper/DeviceInfoMapper.xml"/>

        <!--package是执行mapper接口的包的路径-->
        <!--<package name="com.mlamp.htcrrc.mapper"/>-->
    </mappers>
</configuration>
```
`mybatis-config.xml`文件是mybatis的核心配置文件，它用来设置mybatis的日志输出、数据源信息、实体类别名、数据表映射文件的位置等，这些设置项都是初始化mybatis的必备条件。

2. 创建`DBCPDataSourceFactory` 类，加载dbcp连接池：
```java
public class DBCPDataSourceFactory extends UnpooledDataSourceFactory {

    public DBCPDataSourceFactory() {
        try {
            InputStreamReader isr = new InputStreamReader(DBCPDataSourceFactory.class.getResourceAsStream("/jdbc.properties"), "UTF-8");
            Properties prop = new Properties();
            prop.load(new BufferedReader(isr));
            this.dataSource = BasicDataSourceFactory.createDataSource(prop);
            LOGINFO.info("初始化数据库连接池成功");
        } catch (Exception e) {
            e.printStackTrace();
            LOGERROR.error(e.getLocalizedMessage());
            LOGINFO.info(e.getLocalizedMessage());
            //如果这里连不上数据库就退出程序
            System.exit(0);
        }
    }
```
这个类，就是上面`mybatis-config.xml`配置文件中datasource标签使用的数据库连接池创建类。

3. 在`src/main/resources`路径下创建`jdbc.properties`文件，用来配置数据库连接的相关信息：
```shell
#数据库的连接参数
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/demo?allowMultiQueries=true&useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC
username=root
password=123456
#初始资源
initialSize=5
#活动资源
maxActive=10
#最大空闲资源
maxIdle=6
#最小空闲资源
minIdle=2
#从数据库连接池获得一个连接时，最大的等待时间，单位为毫秒
maxWait=60000
characterEncoding=UTF-8
#自动提交事务，这个必须设置，否则setAutoCommit无效
defaultAutoCommit=true

#设定数据库连接池可回收利用
#标记是否删除超过removeAbandonedTimout所指定时间的被遗弃的连接
removeAbandoned=true
#一个被抛弃连接可以被移除的超时时间，单位为秒
removeAbandonedTimeout=120
#标志是否为应用程序中遗弃语句或连接的代码开启日志堆栈追踪
logAbandoned=true
#对connection进行有效性检测
testOnBorrow=true
#测试连接可用性
validationQuery=select 1
```
## 第三步，初始化mybatis，集成tk-mybatis
1. 使用`MybatisConfig`类初始化mybatis，加载`mybatis-config.xml`配置文件：
```java
/**
 * Mybati配置类，获取sqlSessionFactory
 */
public class MybatisConfig {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }
}
```
sqlSessionFactory应该使用单例模式，作用域为应用区域，即在整个程序运行期间，它都应该存在，且只存在一个。

2. 创建`SqlSessionUtil`类，用于创建`SqlSession`，并集成`tk-mybatis`：
```java
public class SqlSessionUtil {

    public static SqlSession openSqlSession() {
        //从刚刚创建的 sqlSessionFactory 中获取 session
        SqlSession sqlSession = MybatisConfig.getSqlSessionFactory().openSession();
        //创建一个MapperHelper
        MapperHelper mapperHelper = new MapperHelper();
        //特殊配置
        Config config = new Config();
        //主键自增回写方法,默认值MYSQL,详细说明请看文档
        config.setIDENTITY("MYSQL");
        //支持getter和setter方法上的注解
        config.setEnableMethodAnnotation(true);
        //设置 insert 和 update 中，是否判断字符串类型!=''
        config.setNotEmpty(true);
        //校验Example中的类型和最终调用时Mapper的泛型是否一致
        config.setCheckExampleEntityClass(true);
        //启用简单类型
        config.setUseSimpleType(true);
        //枚举按简单类型处理
        config.setEnumAsSimpleType(true);
        //自动处理关键字 - mysql
        config.setWrapKeyword("`{0}`");
        //设置配置
        mapperHelper.setConfig(config);
        //注册通用接口，和其他集成方式中的 mappers 参数作用相同
        //4.0 之后的版本，如果类似 Mapper.class 这样的基础接口带有 @RegisterMapper 注解，就不必在这里注册
        mapperHelper.registerMapper(MyMapper.class);
        //配置 mapperHelper 后，执行下面的操作
        mapperHelper.processConfiguration(sqlSession.getConfiguration());
        return sqlSession;
    }

    public static void closeSqlSession(SqlSession sqlSession){
        sqlSession.commit();
        sqlSession.close();
        sqlSession=null;
    }
```
3. 在`src/main/java`路径下加入package `tk.mybatis.mapper`，并在该包下创建`MyMapper`类：
```java
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;

public interface MyMapper<T> extends Mapper<T>, MySqlMapper<T> {
}
```
## 第四步，开始使用tk-mybatis操作数据库
以下是使用通用Mapper（tk-mybatis）查询数据的推荐写法：
```java
public class DeviceInfoDao {

    public List<DeviceInfo> getAllDeviceInfoList(){
        List<DeviceInfo> list=null;
        try {
            SqlSession sqlSession=SqlSessionUtil.openSqlSession();
            DeviceInfoMapper mapper=sqlSession.getMapper(DeviceInfoMapper.class);
            list=mapper.selectAll();
            SqlSessionUtil.closeSqlSession(sqlSession);
        }catch (Exception e){
            LOGINFO.error("查询错误：",e);
            LOGERROR.error("查询错误：",e);
        }
        return list;
    }
}
```
注意，从`openSqlSession`那一行到`closeSqlSession`结束，应该放入`try-catch`语句中，并且每次使用mapper操作数据库时，都应该是先`openSqlSession`，再`closeSqlSession`。重要！使用完后，一定要调用`closeSqlSession`。

# 使用mybatis代码自动生成插件，生成实体类、mapper接口、映射文件
## 第一步，在pom.xml文件中加入mybatis-generator插件
```xml
	<properties>
		<mysql.version>5.1.44</mysql.version>
        <tk-mybatis-mapper.version>4.1.5</tk-mybatis-mapper.version>
        <!-- Maven Settings -->
        <mybatis-generator-maven-plugin.version>1.3.7</mybatis-generator-maven-plugin.version>
	</properties>

<build>
	<plugins>
	<!--mybatis代码自动生成插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>${mybatis-generator-maven-plugin.version}</version>
                <configuration>
                    <!--配置文件的位置-->
                    <configurationFile>
                        ${basedir}/src/main/resources/generator/generatorConfig.xml
                    </configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${mysql.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper</artifactId>
                        <version>${tk-mybatis-mapper.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
	</plugins>
	
	<resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
        </resources>
</build>
```
## 第二步，创建generatorConfig.xml配置文件
1. 在`src/main/resources`下创建generator文件夹，在generator下创建`generatorConfig.xml`，内容如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 引入数据库连接配置 -->
    <properties resource="generator/jdbc.properties"/>

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>

        <!-- 自带插件，实体类序列化 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 配置 tk.mybatis 插件 -->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.MyMapper"/>
            <property name="caseSensitive" value="true"/>
            <!--<property name="lombok" value="Data"/>-->
            <!--<property name="swagger" value="true"/>-->
        </plugin>

        <!-- 配置数据库连接 -->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.username}"
                        password="${jdbc.password}">
            <!--解决mysql驱动升级到8.0后不生成指定数据库代码的问题-->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>

        <!-- 配置实体类存放路径 -->
        <javaModelGenerator targetPackage="com.yj.bean" targetProject="src/main/java"/>

        <!-- 配置 XML 存放路径 -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources"/>

        <!-- 配置 DAO 存放路径 -->
        <javaClientGenerator targetPackage="com.yj.dao.mapper"
                             targetProject="src/main/java"
                             type="XMLMAPPER"/>

        <!-- 配置需要指定生成的数据表，% 代表所有表 -->
        <table tableName="%">
            <!-- mysql 配置 -->
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```
2. 创建`jdbc.properties`数据库连接配置文件：
```shell
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/demo?nullCatalogMeansCurrent=true
jdbc.username=root
jdbc.password=123456
```
## 第三步，使用mybatis-generator插件生成代码
在idea的maven project面板中，找到`mybatis-generator`插件，点击`mybatis-generator:generate`即可自动生成实体类、dao层的mapper接口、数据表映射文件，如下图所示：
![使用方法](https://cdn.jsdelivr.net/gh/starsky1/poi/2019/2401.png)
# 参考资料
- [Mybatis3.x中文官方文档](https://mybatis.org/mybatis-3/zh/getting-started.html)
- [tk-mybatis官方文档](https://gitee.com/free/Mapper/wikis/Home)
- [MyBatis 单独使用步骤（不结合Spring）](https://www.jianshu.com/p/188918c8c445)
