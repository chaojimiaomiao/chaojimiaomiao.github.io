---
layout: post
title: OC中的runtime与Java反射机制对比
modified: 2015-11-03
tags: [oc, java]
comments: true
---

## OC中的runtime与Java反射机制对比

iOS中的运行时编程，类似Java的反射。

Java是一门静态语言，类和方法都有严格的public, private之分。而反射机制却可以实现动态性，获取类的私有方法等。

##### runtime VS reflection

1. ######相同点

	都可以实现的功能：获取类信息、属性设置获取、类的动态加载、方法的动态调用等。
	
	**ios中相关方法使用：**
	
	类的动态加载：`NSClassFromString(@“className”)`
	
	方法的动态调用：`NSSelectorFromString(@“doSomethingMethod”)`
	
	<!--more-->
	
	**常见的方法：**
	
	`isKindOfClass:  isMemberOfClass:  respondsToSelector: conformsToProtocol:  methodForSelector: `
	
	**给对象发消息：**
	
	`objc_msgSend(receiver, selector)`
	
	`objc_msgSend(receiver, selector, arg1, arg2,...)`
	
	**动态方法解决：**
	
	`@dynamic propertyName;`
	
	**消息转发：**
	<pre class="brush:objc;gutter:true;">  
	negotiate
　　{
   　　 if ( [someOtherObject respondsTo:@selector(negotiate)] )
        　　return [someOtherObject negotiate];
    　　return self;
　　}   </pre>
	（在一些解释型语言中，遇到找不到的方法，转到`notFoundMethod`更多）
	
2. ######不同点
 
	OC能动态得给class添加类和方法，Java则不行。例如：
	<pre class="brush:objc;auto-links:false;">  
	import<objc/runtime.h>
	Class newClass = objc_allocateClassPair([NSError class], "RuntimeErrorSubclass", 0);
	class_addMethod(newClass, @selector(report), (IMP)ReportFunction, "v@:")  
	</pre>
	
	先用objc_allocateClassPair动态函数创建一个类，并在参数中指明该类的父类和类名。再用class_addMethod函数为该类增加了一个方法report，这个方法是由函数ReportFunction实现的，由于该函数至少应包含两个参数self和_cmd，因此该方法有3个参数，类型分别为 ** v、@、：** 一个返回值，self和—cmd。
	
3. ######深层次对比
	
	动态机制：
	
	OC的runtime对class method的调用是通过全局名称查询；而Java VM则是通过类似C++的虚表机制。 所以OC能动态地给class添加方法，Java则不行。
	
	<font color="blue"> <strong>这种差别也正好说明了OC是一种动态语言，而Java却是静态语言。Java的reflection只是一种语法特性，而OC的runtime却是一种运行时环境。</strong> </font>

<br/><br/>















