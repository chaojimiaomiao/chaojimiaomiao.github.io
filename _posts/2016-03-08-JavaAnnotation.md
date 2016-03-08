---
layout: post
title:  "自己动手写个api annotation库"
modified:   2016-03-08
tags: [android, java]
comments: true
---

自己动手写个api annotation库  

1. 目前封装的okhttp api库  
2. retrofit 示例与缺点  
3. 期望的代码  
4. annotation分类与实现  
5. 原理解析  

<!--more-->

### 目前封装的okhttp api库

<br/>

最近在整理公司新开的一个 android 项目代码时，发现了一个不大不小的问题。在API 那层的package 底下有很多重复的代码，随便举个 class 的例子。

目前我们用的是OKhttp的库，封装了一层 `XApi` ：

{% highlight Java %}
public class ApiAddress {
    public static Call<Address> createAddress(String regionId, String detailAddress, String mobile, String name) {
        return XClient.getClient()
                .url("Address.create/")
                .addQueryParameter("regionId", regionId)
                .addQueryParameter("detailAddress", detailAddress)
                .addQueryParameter("mobile", mobile)
                .addQueryParameter("name", name)
                .addQueryParameter("postcode", "")
                .get()
                .makeCall(new TypeToken<Address>() {
                }.getType());
    }
    public static Call<Address> updateAddress(String id, String regionId, String detailAddress, String mobile, String name) {
        return XClient.getClient()
                .url("Address.update/")
                .addQueryParameter("addressId", id)
                .addQueryParameter("regionId", regionId)
                .addQueryParameter("detailAddress", detailAddress)
                .addQueryParameter("mobile", mobile)
                .addQueryParameter("name", name)
                .addQueryParameter("postcode", "")
                .get()
                .makeCall(new TypeToken<Address>() {
                }.getType());
    }
    public static Call<String> deleteAddress(String id) {
        return XClient.getClient()
                .url("Address.delete/")
                .addQueryParameter("addressId", id)
                .get()
                .makeCall(String.class);
    }
}
{% endhighlight %} 

<br/>

#### 这么做的缺点是：  
- 大量的重复代码 （比如addQueryParameter等）  
- 人容易出错

<br/>

### retrofit示例与缺点

假如使用 Retrofit （一个不错的网络请求库）

定义一个新的接口：  
GitHubService.java  
{% highlight Java %}
public interface GitHubService {
  @GET("/users/{user}/repos")
  List<Repo> listRepos(@Path("user") String user);
}
{% endhighlight %} 

GitHubApi.java  
{% highlight Java %}
public static List<Repo> getListRepos() {
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
return service.listRepos("octocat");
}
{% endhighlight %} 

调用：  
`List<Repo> repos = GitHubApi.getListRepos();`

<br/>

##### 缺点是：  
- 额外定义接口文件，但还是要封装 api class。

<br/>

### 期望的代码

<br/>

期望的代码，去除所有的冗余

GET 请求举例：  
{% highlight Java %}
public abstract class AddressApi {
    @GET
    @Path("Address.create/")
    @BXClient
    @BXFullUrlClient
    public abstract Call<Address> createAddress(String regionId, String detailAddress, String mobile, String name);
    @GET
    @Path("Address.update/")
    public abstract Call<Address> updateAddress(String id, String regionId, String detailAddress, String mobile, String name);
    @GET
    @Path("Address.delete/")
    public abstract Call<Address> deleteAddress(String id);
}
{% endhighlight %} 

<br/>

#### 如何实现？

<br/>

一种是使用 [android annotation](http://androidannotations.org/){:target="_blank"}

android annotations的目标是促进安卓应用的`编写`和`维护`。  
它包含了对界面，资源，系统服务等的依赖注入，有 ormlite, otto, rest, roboguice 四个annotation 库。分别对应 <font color='blue'> __数据库ORM 框架__，__轻量级的EventBus__, __符合rest设计的api__, __view与id的对应__ 。</font>

<br/>

#### 它的缺点：

- 框架太大，包含了很多不需要的api  
- 不包含网络层的注解  
- 自定义困难（可以扩展 annotation-core来实现）  

假如我们自己来编写 Java annotation 注解，就可以获得极大的对代码的掌控力：

![image]({{ site.url }}/img/annotation_process.png)

其中 AnnotationProcessor.java是关键。该类继承了 `javax.annotation.processing.AbstractProcessor`。它是一个抽象类，其中最主要的方法是 process。

从process出发，两个 for循环 分别遍历了包含annotation的 class 以及 class底下的 method。 而APIClassInjector 和 APIMethodInjector 分别是对 类和 方法的解析。

具体代码 详见 github :

<br/>

按上文期望的代码写下 API 后，打开 build 文件夹：

![image]({{ site.url }}/img/annotation_build.png)

其他文件都很正常，除了两个"不速之客"，UserAPI$$APIINJECTOR.java 和 UserAPI$$APIINJECTOR.class。没错，这就是使用Annotation Processor生成的java文件了。
我们看看生成了什么。

打开 .class 文件

{% highlight Java %}
package org.gemini.httpengine.examples;

import org.gemini.httpengine.library.*;

public class UserAPI$$APIINJECTOR implements org.gemini.httpengine.examples.UserAPI {
    public void login(org.gemini.httpengine.library.OnResponseListener l, java.lang.String username, java.lang.String password) {
        final String FIELD_USERNAME = "username";
        final String FIELD_PASSWORD = "password";
        GMHttpParameters httpParameter = new GMHttpParameters();
        httpParameter.setParameter(FIELD_USERNAME, username);
        httpParameter.setParameter(FIELD_PASSWORD, password);
        GMHttpRequest.Builder builder = new GMHttpRequest.Builder();
        builder.setHttpParameters(httpParameter);
        builder.setTaskId("login");
        builder.setUrl("http://www.baidu.com");
        builder.setMethod("GET");
        builder.setOnResponseListener(l);
        GMHttpService service = GMHttpService.getInstance();
        service.executeHttpMethod(builder.build());
    }
}
{% endhighlight %} 

这个就是 `InjectFactory.inject` 在编译期间 生成的代码。注意，java annotation使用的都是 __完全限定名__ 。

<font color='blue'>__生成代码__</font> 这么神奇的事是怎么做到的呢 ？这个就要提到`apt` 这玩意了。 apt实际上是 java提供给厂商定义接口服务用的。

编写注解处理器的核心是AnnotationProcessorFactory和AnnotationProcessor两个接口。后者表示的是注解处理器，而前者则是为某些注解类型创建注解处理器的工厂。

<br/>

### 原理：

<br/>

#### Annotation 分类

1. 标准 Annotation  
包括 Override, Deprecated, SuppressWarnings，标准 Annotation 是指 Java 自带的几个 Annotation
上面三个分别表示 重写函数，函数已经被禁止使用，忽略某项 Warning

2. 元 Annotation  
元 Annotation 是指用来定义 Annotation 的 Annotation

@Documented 是否会保存到 Javadoc 文档中
@Retention 保留时间，可选值 SOURCE（源码时），CLASS（编译时），RUNTIME（运行时），默认为 CLASS。
     值为 SOURCE 大都为 Mark Annotation，这类 Annotation 大都用来校验，比如 Override, Deprecated, SuppressWarnings
@Target 可以用来修饰哪些程序元素，如 TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER 等，未标注则表示可修饰所有
@Inherited 是否可以被继承，默认为 false

在实际使用过程中，  

{% highlight Java %}
@Retention(RetentionPolicy.SOURCE)
@Target(value = {ElementType.METHOD, ElementType.TYPE})
public @interface Path {
    String value();
}
{% endhighlight %} 

3. 自定义 Annotation  
自定义 Annotation 表示自己根据需要定义的 Annotation，定义时需要用到上面的元 Annotation
这里只是一种分类而已，也可以根据作用域分为<font color='blue'>源码时、编译时、运行时 </font> Annotation。

<br/>

#### Annotation是如何被处理的

<br/>

当Java源代码被编译时，编译器的一个插件annotation处理器则会处理这些annotation。处理器可以产生报告信息，或者创建附加的Java源文件或资源。如果annotation本身被加上了RententionPolicy的运行时类，则Java编译器则会将annotation的元数据存储到class文件中。然后，Java虚拟机或其他的程序可以查找这些元数据并做相应的处理。

当然除了annotation处理器可以处理annotation外，我们也可以使用反射自己来处理annotation。Java SE 5有一个名为AnnotatedElement的接口，Java的反射对象类Class,Constructor,Field,Method以及Package都实现了这个接口。这个接口用来表示当前运行在Java虚拟机中的被加上了annotation的程序元素。通过这个接口可以使用反射读取annotation。AnnotatedElement接口可以访问被加上RUNTIME标记的annotation，相应的方法有getAnnotation,getAnnotations,isAnnotationPresent。由于Annotation类型被编译和存储在二进制文件中就像class一样，所以可以像查询普通的Java对象一样查询这些方法返回的Annotation。

- ElementType.ANNOTATION_TYPE     //
- ElementType.CONSTRUCTOR
- ElementType.FIELD
- ElementType.LOCAL_VARIABLE
- ElementType.METHOD
- ElementType.PACKAGE
- ElementType.PARAMETER
- ElementType.TYPE                           //所有类型




