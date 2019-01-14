创1、思考？

单机版项目项目构建过程

一般就是分层，分层大概的业务层，服务层，数据层。

对应的包结构如下：

![](/assets/fbs-tradition.png)

在分布式当中我们应该怎么进行业务拆分，进行项目分割？

![](/assets/fbs1.png)

说明一下：这里只是简单的根据业务层，服务层，数据层，进行分割，抽离出网关进行提供服务，service层由业务层（网关）调用，

调用这里我们使用的dubbo的注解（RPC协议Remote Procedure Call Protocol ， 远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议）。最终的提供服务放在service-provider中。

补充：其中的一些包会被其他项目所使用的，也可以单独抽离出来作为一个项目，方便外部调用

实战搭建：基于dubbo+zookepper的框架。

**新建一个项目fbs-demo-entity**

pom.xml文件

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>fbs</artifactId>
        <groupId>com.zachary.demo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>fbs-demo-entity</artifactId>

    <name>fbs-demo-entity</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <!--这是hibernate 的JPA 包-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <!--这是JSON格式转换需要的包-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.5.0</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

项目结构：

![](/assets/i1.png)

User类

```
package com.zachary.demo.entity;

import javax.persistence.*;
import java.io.Serializable;//序列化，原因是需要通过网络传输，序列化传输，反序列化还原成对象

/**
 * @Title:
 * @Author:Zachary
 * @Desc:
 * @Date:2019/1/14
 **/

@Entity
@Table(name = "user")
public class User implements Serializable {
    private int id;
    private String userName;
    private String password;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @Column(name = "user_name")
    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    @Column(name = "password")
    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

就这样就可以了

**创建fbs-demo-service**

pom.xml，需要引入fbs-demo-entity包

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>fbs</artifactId>
        <groupId>com.zachary.demo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>fbs-demo-service</artifactId>

    <name>fbs-demo-service</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.zachary.demo</groupId>
            <artifactId>fbs-demo-entity</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

项目结构

![](/assets/2.png)

IBaseService.java 抽离出来的数据库操作共同方法，使用泛型

```
package com.zachary.demo.service;

public interface IBaseService<T> {
    T save(T t);

    T update(T t);
}
```

IUserService.java

```
package com.zachary.demo.service;

import com.zachary.demo.entity.User;

/**
 * @Title:
 * @Author:Zachary
 * @Desc:
 * @Date:2019/1/14
 **/
public interface IUserService extends IBaseService<User> {

}
```

**创建fbs-demo-service-provider**

pom文件

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>fbs</artifactId>
        <groupId>com.zachary.demo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>fbs-demo-service-provider</artifactId>

    <name>fbs-demo-service-provider</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.zachary.demo</groupId>
            <artifactId>fbs-demo-service</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--数据库连接-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.35</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.5</version>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>1.7.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>proxool</groupId>
            <artifactId>proxool</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>proxool</groupId>
            <artifactId>proxool-cglib</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.netty</groupId>
            <artifactId>netty</artifactId>
            <version>3.2.9.Final</version>
        </dependency>
        <!--Apache Commons包含了很多开源的工具，用于解决平时编程经常会遇到的问题，减少重复劳动-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.0</version>
        </dependency>
        <!--日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-nop</artifactId>
            <version>1.7.7</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-jar-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
                <plugin>
                    <artifactId>maven-site-plugin</artifactId>
                    <version>3.7.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-project-info-reports-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

项目结构

![](/assets/3.png)

配置一览

applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/data/jpa
           http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
    <!--导入config文件 使用 context:property-placeholder  -->
    <context:property-placeholder location="classpath:config.properties"/>
    <!--引入dubbo.xml 文件-->
    <import resource="dubbo-provider.xml"/>

    <!--配置数据库-->
    <bean id="entityManagerFactory"
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="packagesToScan" value="com.zachary"/>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
        <property name="dataSource">
            <bean class="org.logicalcobwebs.proxool.ProxoolDataSource">
                <property name="driver" value="${db.driver}"/>
                <property name="driverUrl" value="${db.url}"/>
                <property name="user" value="${db.username}"/>
                <property name="password" value="${db.password}"/>
                <!-- 最小连接数（默认5） -->
                <property name="minimumConnectionCount" value="${proxool.minimumConnectionCount}"/>
                <!-- 最大连接数（默认15） -->
                <property name="maximumConnectionCount" value="${proxool.maximumConnectionCount}"/>
                <!-- 检测器休眠时间间隔，单位毫秒（默认30000）。每隔配置时间间隔后检测所有连接状态，空闲回收，超时销毁 -->
                <property name="houseKeepingSleepTime" value="${proxool.houseKeepingSleepTime}"/>
                <!-- 最小可用连接数（默认0） -->
                <property name="prototypeCount" value="${proxool.prototypeCount}"/>

                <property name="simultaneousBuildThrottle" value="${proxool.prototypeCount}"/>
                <property name="houseKeepingTestSql" value="SELECT 1"/>
                <property name="testBeforeUse" value="true"/>
            </bean>
        </property>
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">${hibernate.dialect}</prop>
                <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
                <prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
                <prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
            </props>
        </property>
    </bean>
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!--支持事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <jpa:repositories base-package="com.zachary"
                      entity-manager-factory-ref="entityManagerFactory"
                      transaction-manager-ref="transactionManager"/>

</beans>
```

dubbo-provider.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="${dubbo.provider.name}" />

    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry protocol="zookeeper" address="${dubbo.zookeeper.address}" />

    <!-- 用dubbo协议将服务暴露在指定端口 -->
    <dubbo:protocol name="dubbo" port="${dubbo.provider.port}" threads="${dubbo.threads}" />

    <!-- 扫描注解，暴露服务 -->
    <dubbo:annotation package="${dubbo.service.scan}" />

    <!-- 监控中心配置 -->
    <!-- 监控中心协议，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心 -->
    <!--<dubbo:monitor protocol="registry"/>-->
</beans>
```

config.properties

```
#全局配置变量
# dubbo 配置
#取得名称，消费者名称与之不同即可
dubbo.provider.name=com.zachary.demo
#我们的zookeeper服务器提供的服务地址和端口
dubbo.zookeeper.address=127.0.0.1\:2181
#dubbo 手动暴露的端口与消费者端口一致即可
dubbo.provider.port=21881
#扫描dubbo下面的服务，进行注册
dubbo.service.scan=com.zachary.demo.service.impl
#线程数500
dubbo.threads=500
#数据库配置
db.driver=com.mysql.jdbc.Driver
db.url=jdbc\:mysql\://localhost\:3306/fbs?useUnicode=true&characterEncoding=UTF-8
db.username=root
db.password=zhangqi
proxool.minimumConnectionCount=5
proxool.maximumConnectionCount=50
proxool.houseKeepingSleepTime=60000
proxool.prototypeCount=5
hibernate.dialect=org.hibernate.dialect.MySQLDialect
hibernate.hbm2ddl.auto=update
hibernate.show_sql=true
hibernate.format_sql=true
```

log4j.properties

```
log4j.rootCategory=INFO, CONSOLE, LOGGER
#log4j.rootCategory=INFO, CONSOLE

log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.Threshold=INFO
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d - %p - %F - %m%n

log4j.appender.LOGGER=org.apache.log4j.DailyRollingFileAppender
log4j.appender.LOGGER.Threshold=INFO
log4j.appender.LOGGER.File=./logs/${project.build.finalName}.log
log4j.appender.LOGGER.DatePattern='.'yyyy-MM-dd
log4j.appender.LOGGER.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGGER.layout.ConversionPattern=%d - %p - %F - %m%n
```

UserService.java

```
package com.zachary.demo.service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import com.zachary.demo.dao.UserDao;
import com.zachary.demo.entity.User;
import com.zachary.demo.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.transaction.annotation.Transactional;

/**
 * @Title:
 * @Author:Zachary
 * @Desc:
 * @Date:2019/1/14
 **/

//注意这里使用的service是dubbo的com.alibaba.dubbo.config.annotation.Service;
@Service
public class UserService implements IUserService {
    @Autowired
    private UserDao userDao;

    //启事务，默认捕捉异常RunTimeException。进行回滚，其他异常一概不回滚，除非制定了rollback
    @Transactional
    @Override
    public User save(User user) throws Exception {
        try {
            userDao.save(user);
        } catch (Exception e) {
            throw new Exception();
        }
        return null;
    }

    @Override
    public User update(User user) {
        return null;
    }
}
```

UserDao.java

```
package com.zachary.demo.dao;

import com.zachary.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserDao extends JpaRepository<User, Integer>, UserDaoCustom {
}
```

UserDaoCustom.java

```
package com.zachary.demo.dao;

import com.zachary.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserDaoCustom {
}
```

UserDaoImpl.java

```
package com.zachary.demo.dao.impl;

import com.zachary.demo.dao.UserDaoCustom;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

/**
 * @Title:
 * @Author:Zachary
 * @Desc:
 * @Date:2019/1/14
 **/
public class UserDaoImpl implements UserDaoCustom {
    @PersistenceContext
    private EntityManager entityManager;
}
```

手动启动Main.java

```
package com.zachary.demo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @Title:
 * @Author:Zachary
 * @Desc:
 * @Date:2019/1/14
 **/
public class Main {
    private static final long BLOCK_TIME = 1000 * 60;

    public static void main(String[] args) {
//        手动从ClassPathXmlApplicationContext 加载 applicationContext文件

        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        context.start();

        System.out.println("===============");
        String[] names = context.getBeanDefinitionNames();
        for (String name : names) {
            System.out.println(name);
        }
        System.out.println("===============");

        while (true) {
            try {
                Thread.sleep(BLOCK_TIME);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



