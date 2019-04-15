#### 关于spring多模块
dependencyManagement 里面的依赖不会真实的去下载jar包，如果子模块中的jar没有指定<version>，就会到父类的<dependencyManagement>中寻找对应的version，如果没有报错！

spring-boot-dependencies与spring-cloud-dependencies就相当于boot与cloud的不同版本，里面集合了很多配套的jar包

```java
<dependencyManagement>
<dependencies>  
	<!-- 这里是spring boot 的版本，里面有很多很多jar包 -->  
	<dependency>  
	    <groupId>org.springframework.boot</groupId>  
	    <artifactId>spring-boot-dependencies</artifactId>  
	    <version>2.0.8.RELEASE</version>  
	    <type>pom</type>  
	    <scope>import</scope>  
	</dependency>  
	  
	<!-- 这里是spring cloud 的版本，里面有很多很多jar包 -->  
	<dependency>  
	    <groupId>org.springframework.cloud</groupId>  
	    <artifactId>spring-cloud-dependencies</artifactId>  
	    <version>${spring-cloud.version}</version>  
	    <type>pom</type>  
	    <scope>import</scope>  
	</dependency>
</dependencies>  
</dependencyManagement>

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMzNDE4ODg2NV19
-->