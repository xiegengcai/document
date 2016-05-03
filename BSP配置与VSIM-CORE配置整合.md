# BSP配置与VSIM-CORE配置整合

1. 问题
> 由于BSP使用统一配置，而VSIM-CORE使用apache-configuration配置，故使用VSIM-CORE接入层客户端的模块无法直接从统一配置上读取到VSIM-CORE需要且各个环境需要的配置信息。

1. 解决思路及方法
> 因为各个环境这些配置值都需要不一样，最理想是在统一配置中配置，那就需要把这些统一配置上读取到的值在加载VSIM-CORE相关spring文件时加到apache-configuration配置中，从而解决apache-configuration不能读取统一配置的值的问题。
> 解放方法是在加载完配置文件后，加载VSIM-CORE相关spring文件之前下手，把统一配置中的一下VSIM-CORE客户端需要的配置值增加到apache-configuration中。

1. 配置指导

> * config.xml

> ~~~
> <?xml version="1.0" encoding="UTF-8"?>
> <!-- see http://commons.apache.org/proper/commons-configuration/userguide_v1.10/user_guide.html -->
> <!-- see http://commons.apache.org/proper/commons-configuration/userguide_v1.10/howto_configurationbuilder.html#Using_DefaultConfigurationBuilder -->
> <configuration>
>     <header>
>         <!-- <fileSystem config-class="org.apache.commons.configuration.VFSFileSystem" /> -->
>     </header>
>     <!--不再加载跟环境（IP、PORT）相关的properties文件，移到统一配置中配置-->
>     <properties fileName="conf/config-default.properties"/>
>     <properties fileName="innerConnectorPool.properties" />
>     <!--<properties fileName="innerDbConnectorPool.properties" />-->
> 
> </configuration>
~~~
> * applicationContext.xml

> ~~~
> <?xml version="1.0" encoding="UTF-8" ?>
> <beans xmlns="http://www.springframework.org/schema/beans"
> xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
> xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
> http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
> 
>     <!-- 使用disconf必须添加以下配置 -->
>     <bean id="disconfMgrBean" class="com.baidu.disconf.client.DisconfMgrBean" destroy-method="destory">
>         <property name="scanPackage" value="com.skyroam.ota,com.simo.vsim.mediation"/>
>     </bean>
>     <bean id="disconfMgrBean2" class="com.baidu.disconf.client.DisconfMgrBeanSecond" init-method="init" destroy-method="destory"/>
> 
>     <!-- 使用托管方式的disconf配置(无代码侵入, 配置更改不会自动reload)-->
>     <bean id="loadProperties" class="com.baidu.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
>         <property name="locations">
>             <list>
>                 <value>bsp_common_config.properties</value>
>                 <value>bsp_ota_mediation_config.properties</value>
>             </list>
>         </property>
>     </bean>
> 
> 
>     <bean name="configuration" class="com.simo.vsim.base.config.Loader" factory-method="loadConfig">
>         <constructor-arg value="conf/config.xml" />
>     </bean>
> 
>     <bean name="commonsConfigurationFactoryBean" class="org.springmodules.commons.configuration.CommonsConfigurationFactoryBean">
>         <constructor-arg ref="configuration" />
>     </bean>
> 
>     <!--com.baidu.disconf.client.addons.properties.ReloadingPropertyPlaceholderConfigurer com.skyroam.bsp.common.config.CustomizedPropertyConfigurer-->
>     <bean id="propertyConfigurerForProject" class="com.skyroam.bsp.common.config.CustomizedPropertyConfigurer">
>         <property name="ignoreResourceNotFound" value="true"/>
>         <property name="ignoreUnresolvablePlaceholders" value="true"/>
>         <property name="propertiesArray">
>             <list>
>                 <ref bean="loadProperties"/>
>                 <ref bean="commonsConfigurationFactoryBean" />
>             </list>
>         </property>
>     </bean>
> 
>     <!-- 将vsim-core需要的属性从统一配置写进configuration中 -->
>     <bean class="com.skyroam.bsp.common.config.VsimPropertiesConfig">
>         <!-- org.apache.commons.configuration.Configuration bean名称 -->
>         <property name="configuration" ref="configuration" />
>         <!-- 需要增加到configuration的配置属性key -->
>         <property name="propertyNames">
>             <list>
>                 <value>server.id</value>
>                 <value>innerDbPool.ip</value>
>                 <value>innerDbPool.port</value>
>                 <value>innerDbPool.connect.count</value>
>                 <value>innerDbPool.connect.timeout</value>
>             </list>
>         </property>
>     </bean>
> 
>     <!-- log4j 关闭 -->
>     <bean id="log4j" class="com.simo.vsim.base.config.Log4j" destroy-method="shutdown"/>
> 
>     <context:component-scan base-package="com.skyroam.bsp.ota" />
> 
>    <import resource="classpath*:bsp-ota-*.xml"/>
>    <import resource="./spring-redis.xml" />
>    <import resource="classpath:appContext.xml" />
> </beans>
> ~~~

> * config.xml

> ~~~
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
>    xmlns:tx="http://www.springframework.org/schema/tx"
>    xsi:schemaLocation="http://www.springframework.org/schema/beans
>     http://www.springframework.org/schema/beans/spring-beans.xsd
>     http://www.springframework.org/schema/context
>     http://www.springframework.org/schema/context/spring-context.xsd
>     http://www.springframework.org/schema/tx
>     http://www.springframework.org/schema/tx/spring-tx.xsd
>    ">
>    <context:component-scan base-package="com.simo.vsim" />
>    <tx:annotation-driven proxy-target-class="true" />
> 
>    !--已整合，不再需要->
>    <!--<import resource="loadConfig.xml" />-->
> 
>    <!-- application -->
>    <import resource="innerConnectorPool.xml" />
>    <bean name="innerConnectorPool" factory-bean="innerConnectorPoolBuilder"
>       factory-method="getInnerConnectorPool" destroy-method="shutdown">
>    </bean>
>    <!-- db -->
>    <import resource="innerDbConnectorPool.xml" />
>    <import resource="accessConnectorPoolInit.xml" />
>    <!-- handler配置，放在pool后，否则会有循环引用问题 -->
>    <bean name="mediationIoHandler" factory-bean="innerConnector"
>       factory-method="getHandler">
>       <property name="handlerMapping">
>          <map>
>             <entry key="7601" value-ref="vsimOtaMediationHandler" />
>             <entry key="7602" value-ref="otaConfirmHandler" />
>          </map>
>       </property>
>    </bean>
> </beans>
> ~~~