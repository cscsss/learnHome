<section id="nice" data-tool="mdnice编辑器" data-website="https://www.mdnice.com" style="font-size: 16px; color: black; padding: 0 10px; line-height: 1.6; word-spacing: 0px; letter-spacing: 0px; word-break: break-word; word-wrap: break-word; text-align: left; font-family: Optima-Regular, Optima, PingFangSC-light, PingFangTC-light, 'PingFang SC', Cambria, Cochin, Georgia, Times, 'Times New Roman', serif;"><h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">反射的概念</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">java的放射机制：在程序运行时，程序有能力获取一个类的所有方法和属性；并且对于任意一个对象，可以调用它的任意方法或者获取其属性</section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">通俗解析：java文件需要编译成.class文件才能被jvm加载使用,对象的.class数据在jvm里就是Class&lt;T&gt;；我们如果能拿到这个Class&lt;T&gt;对象，就能获取该Class&lt;T&gt;对应的对象类型，及在该类型声明的方法和属性值；还可以根据Class&lt;T&gt;创建相应的类型对象，通过Field,Method反过来操作对象</section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">java相关类介绍</section></li></ul>
<section class="table-container" data-tool="mdnice编辑器" style="overflow-x: auto;"><table style="display: table; text-align: left;">
<thead>
<tr style="border: 0; border-top: 1px solid #ccc; background-color: white;">
<th style="font-size: 16px; padding: 5px 10px; text-align: left; font-weight: bold; border: 1px solid #009688; background-color: #009688; color: #f8f8f8; border-bottom: 0; min-width: 85px;">类名</th>
<th style="font-size: 16px; padding: 5px 10px; text-align: left; font-weight: bold; border: 1px solid #009688; background-color: #009688; color: #f8f8f8; border-bottom: 0; min-width: 85px;">描述</th>
</tr>
</thead>
<tbody style="border: 0;">
<tr style="border: 0; border-top: 1px solid #ccc; background-color: white;">
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">Class&lt;T&gt;</td>
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">代表类的实体，在运行的Java应用程序中表示类或者接口</td>
</tr>
<tr style="border: 0; border-top: 1px solid #ccc; background-color: #f8f8f8;">
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">Field</td>
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">类的成员变量（成员变量也称为类的属性）</td>
</tr>
<tr style="border: 0; border-top: 1px solid #ccc; background-color: white;">
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">Method</td>
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">类的方法</td>
</tr>
<tr style="border: 0; border-top: 1px solid #ccc; background-color: #f8f8f8;">
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">Constructor&lt;T&gt;</td>
<td style="font-size: 16px; padding: 5px 10px; text-align: left; border: 1px solid #009688; min-width: 85px;">类的构造方法</td>
</tr>
</tbody>
</table>
</section><h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">获取Class的三种方法</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">1通过已知的类型获取class</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">// 根据Example 获取Class =》Example.class</span>
<span/><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;Example&gt; <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getExample</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span></span>{
<span/>    Class&lt;Example&gt; clazz = Example.class;
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> clazz;
<span/>}
<span/></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">2通过实例对象获取class</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;Example&gt; <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getExampleByInstance</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span></span>{
<span/>    Example example = <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">new</span> Example();
<span/>    <span class="hljs-comment" style="color: #007400; line-height: 26px;">// getClass是Object类里面的方法；《?》 是通配符</span>
<span/>    Class&lt;?&gt; clazz = example.getClass();
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> (Class&lt;Example&gt;)clazz;
<span/>}
<span/></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">3通过Class.forName获取全路径指定类名的class</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">/** forName0 本地方法，C++实现，jvm调用
<span/> * 1 className 是个类名  2 initialize 是否延迟加载  3 loader 加载器
<span/> */</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">native</span> Class&lt;?&gt; forName0(String className, <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">boolean</span> initialize,
<span/>        ClassLoader loader, Class&lt;?&gt; caller) <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> ClassNotFoundException;
<span/>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> Class&lt;?&gt; forName(String className) <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> ClassNotFoundException {
<span/>        Class&lt;?&gt; caller = Reflection.getCallerClass();
<span/>        <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> forName0(className, <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">true</span>, ClassLoader.getClassLoader(caller), caller);
<span/>    }
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">// 两个forName方法最终都会调用forName0方法去加载class   </span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> Class&lt;?&gt; forName(String name,
<span/>        <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">boolean</span> initialize, ClassLoader loader) <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> ClassNotFoundException {
<span/>        ....
<span/>        <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> forName0(name, initialize, loader, caller);
<span/>    }
<span/></code></pre>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">// 示例:通过java.lang.Integer </span>
<span/><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;Integer&gt; <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getInteger</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> ClassNotFoundException</span>{
<span/>    Class&lt;?&gt; clazz = Class.forName(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"java.lang.Integer"</span>);
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> (Class&lt;Integer&gt;)clazz;
<span/>}
<span/></code></pre>
<h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">JAVA反射API</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">Class常用操作方法</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取所有的构造方法 / private public</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Constructor&lt;?&gt;[] getDeclaredConstructors()
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取特定的构造方法 / private public</span>
<span/><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Constructor&lt;T&gt; <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getDeclaredConstructor</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Class&lt;?&gt;... parameterTypes)</span>    
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取类的父类</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">native</span> Class&lt;? <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">super</span> T&gt; <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getSuperclass</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span>    
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取类实现的接口</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> Class&lt;?&gt;[] <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getInterfaces</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(<span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">boolean</span> cloneArray)</span>  
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取在类内定义的内部类或接口</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;?&gt;[] <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getDeclaredClasses</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span>
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取所有的方法</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Method[] <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getDeclaredMethods</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> SecurityException
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//根据方法名和参数获得特定的方法</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Method <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getDeclaredMethod</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String name, Class&lt;?&gt;... parameterTypes)</span>  
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取类型的定义的所有属性</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Field[] <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getFields</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> SecurityException
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">// 根据属性命名获得特定的Field</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Field <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">getField</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String name)</span> 
<span/></span></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">Method常用的操作方法</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获得方法的放回类型</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;?&gt; getReturnType()   
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//获得方法的传入参数类型</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Class&lt;?&gt;[] getParameterTypes()   
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//obj是实例对象，args是方法，反过来由Method控制对象的方法调用</span>
<span/><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Object <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">invoke</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Object obj, Object... args)</span>
<span/></span></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">Field常用的操作方法</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;">//属性与obj相等则返回<span class="hljs-literal" style="color: #aa0d91; line-height: 26px;">true</span>
<span/>public boolean equals(Object obj)
<span/>//获得obj中对应的属性值
<span/>public Object get(Object obj)
<span/>//设置obj中对应属性值
<span/>public void <span class="hljs-built_in" style="color: #5c2699; line-height: 26px;">set</span>(Object obj, Object value) 
<span/></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">Constructor</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">//根据传递的参数创建类的对象:initargs 构造方法参数</span>
<span/><span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> T <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">newInstance</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Object... initargs)</span> 
<span/></span></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">1根据class创建对象</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-comment" style="color: #007400; line-height: 26px;">//方式一 clazz.newInstance()</span>
<span/>Class&lt;Example&gt; clazz = Example.class;
<span/>Example example = clazz.newInstance();
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//方式二 先获取再由Constructor：clazz.getConstructors()/getConstructor(...) </span>
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//再由Constructor.newInstance 方法构造对象</span>
<span/>-----------------------------------------
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Example</span> </span>{
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">int</span> value;
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">Example</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">()</span></span>{ } <span class="hljs-comment" style="color: #007400; line-height: 26px;">// 如果只声明有参构造函数，clazz.newInstance()会报错</span>
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">Example</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Integer value)</span></span>{  <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">this</span>.value  = value;  }
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">main</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String[] args)</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> Exception</span>{
<span/>        Class&lt;Example&gt; clazz = Example.class;
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//根据指定构造函数参数获取Constructor</span>
<span/>        Constructor&lt;Example&gt; constructor = clazz.getConstructor(Integer.class);
<span/>        Example example = constructor.newInstance(<span class="hljs-number" style="color: #1c00cf; line-height: 26px;">100</span>);
<span/>        System.out.println(example.value);
<span/>    }
<span/>}    
<span/></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">2由class获取Field，并操作实例的属性</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Example</span> </span>{
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">int</span> value , count;
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">main</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String[] args)</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> Exception</span>{
<span/>        Class&lt;Example&gt; clazz = Example.class;
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//获取所有的属性，getField只能获取public的属性</span>
<span/>        Field[] fs = clazz.getDeclaredFields();
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//根据名称获取指定 Field</span>
<span/>        Field value = clazz.getDeclaredField(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"value"</span>);
<span/>        Example example = clazz.newInstance();
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//使用反射机制可以打破封装性，导致了java对象的属性不安全  </span>
<span/>        value.setAccessible(<span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">true</span>); <span class="hljs-comment" style="color: #007400; line-height: 26px;">//setAccessible(true)让private的参数可赋值操作</span>
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//由Field反过去设置example的值</span>
<span/>        value.set(example,<span class="hljs-number" style="color: #1c00cf; line-height: 26px;">100</span>);
<span/>        System.out.println(example.value);
<span/>    }
<span/>}
<span/></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">3由class获取Method，并反射调用实例方法</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Example</span> </span>{
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">main</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String[] args)</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> Exception </span>{
<span/>        Class&lt;Example&gt; clazz = Example.class;
<span/>        Example example = clazz.newInstance();
<span/>        Method[] methods = clazz.getDeclaredMethods();
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//getDeclaredMethod和getMethod是：getMethod只能返回public的方法</span>
<span/>        Method method = clazz.getDeclaredMethod(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"hello"</span>, String.class);
<span/>        method.setAccessible(<span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">true</span>);
<span/>        method.invoke(example, <span class="hljs-string" style="color: #c41a16; line-height: 26px;">"cscw"</span>);
<span/>    }
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">hello</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String name)</span> </span>{ System.out.println(name + <span class="hljs-string" style="color: #c41a16; line-height: 26px;">" Hello!"</span>); }
<span/>}
<span/>－－－－－
<span/>cscw Hello!
<span/></code></pre>
<h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">反射机制应用的场景</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">1 动态拓展：假设有同一组类是实现相同的接口，并且类的加载方式不限制。当我们需要那种具体类实现的功能时，只需加载.class文件，并获取对应的Class&lt;T&gt;对象。可以由Class或者Constructor实例化对象instance；根据接口定义，可以获取Class&lt;T&gt;里的某一方法Method，并配合instance反射调用功能方法</section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">2 Spring的IOC就是基于反射机制实现</section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">3 JDK的动态代理</section></li></ul>
<h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">反射和JDK动态代理</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">在Java的java.lang.reflect包下提供了一个Proxy类和一个InvocationHandler接口。通过这个类和接口可以生成JDK动态代理类或动态代理对象</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">interface</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">InvocationHandler</span> </span>{
<span/> <span class="hljs-comment" style="color: #007400; line-height: 26px;">//所有方法都会调用此代理方法</span>
<span/>    <span class="hljs-function" style="line-height: 26px;">Object <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">invoke</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Object var1, Method var2, Object[] var3)</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> Throwable</span>;
<span/>}  
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Proxy</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">implements</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Serializable</span>｛
<span/> ...
<span/>    //根据<span class="hljs-title" style="color: #5c2699; line-height: 26px;">interfaces</span>和<span class="hljs-title" style="color: #5c2699; line-height: 26px;">InvocationHandler</span>生成代理对象
<span/>    <span class="hljs-title" style="color: #5c2699; line-height: 26px;">public</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">static</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Object</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">newProxyInstance</span>(<span class="hljs-title" style="color: #5c2699; line-height: 26px;">ClassLoader</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">loader</span>,
<span/>      <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Class</span>&lt;?&gt;[] <span class="hljs-title" style="color: #5c2699; line-height: 26px;">interfaces</span>, <span class="hljs-title" style="color: #5c2699; line-height: 26px;">InvocationHandler</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">h</span>) 
<span/>    ...    
<span/>｝
<span/></span></code></pre>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">JDK的动态代理由Proxy和InvocationHandler实现；而被代理对象必须实现一个接口。代理对象由Proxy生成，可转为接口interface的实现类对象OBJ。当调用OBJ的方法时，则会触发InvocationHandler.invoke，参数依次为<strong style="font-weight: bold; color: black;">代理对象</strong>，<strong style="font-weight: bold; color: black;">Method对象</strong>,和<strong style="font-weight: bold; color: black;">方法Method所需的参数</strong>。在invoke方法可以加入拓展的逻辑，如日志记录操作；<strong style="font-weight: bold; color: black;">并可以在invoke里利用反射的技术调用被代理对象方法</strong></section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;">示例</section></li></ul>
<pre class="custom" data-tool="mdnice编辑器" style="margin-top: 10px; margin-bottom: 10px; border-radius: 5px; box-shadow: rgba(0, 0, 0, 0.55) 0px 2px 10px;"><span style="display: block; background: url(https://my-wechat.mdnice.com/point.png); height: 30px; width: 100%; background-size: 40px; background-repeat: no-repeat; background-color: #fff; margin-bottom: -7px; border-radius: 5px; background-position: 10px 10px;"></span><code class="hljs" style="overflow-x: auto; padding: 16px; color: black; display: block; font-family: Operator Mono, Consolas, Monaco, Menlo, monospace; font-size: 12px; -webkit-overflow-scrolling: touch; padding-top: 15px; background: #fff; border-radius: 5px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">ExampleFactory</span>&lt;<span class="hljs-title" style="color: #5c2699; line-height: 26px;">T</span>&gt; <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">implements</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">InvocationHandler</span></span>{
<span/>    <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">private</span> T target; <span class="hljs-comment" style="color: #007400; line-height: 26px;">//被代理对象</span>
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> T <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">bind</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(T obj)</span></span>{
<span/>        target = obj;
<span/>        <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> (T)Proxy.newProxyInstance(obj.getClass().getClassLoader(),
<span/>           obj.getClass().getInterfaces(),<span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">this</span>);
<span/>    }
<span/>    <span class="hljs-comment" style="color: #007400; line-height: 26px;">/** Object o 是代理对象; o的方法调用 -&gt; ExampleFactory.invoke 
<span/>    *  invoke(...) -&gt; 在invoke方法里面 反射调用代理对象方法+增强逻辑
<span/>    */</span>
<span/>    <span class="hljs-meta" style="color: #643820; line-height: 26px;">@Override</span>
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> Object <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">invoke</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(Object o, Method method, Object[] objects)</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">throws</span> Throwable </span>{
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//增强逻辑</span>
<span/>        System.out.println(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"log start"</span>);
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//反射调用被代理对象方法</span>
<span/>        Object result = method.invoke(target,objects);
<span/>        System.out.println(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"log end"</span>);
<span/>        <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">return</span> result;
<span/>    }
<span/>}
<span/>-----------
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">interface</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Face</span> </span>{
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">hello</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String name)</span></span>;
<span/>}
<span/>---------
<span/><span class="hljs-comment" style="color: #007400; line-height: 26px;">//被代理对象必须实现一个接口，并由接口方法对方提供功能</span>
<span/><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-class" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">class</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Example</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">implements</span> <span class="hljs-title" style="color: #5c2699; line-height: 26px;">Face</span> </span>{
<span/> <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">hello</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String name)</span> </span>{
<span/>        System.out.println(name + <span class="hljs-string" style="color: #c41a16; line-height: 26px;">" Hello!"</span>);
<span/>    }
<span/>    <span class="hljs-function" style="line-height: 26px;"><span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">public</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">static</span> <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">void</span> <span class="hljs-title" style="color: #1c00cf; line-height: 26px;">main</span><span class="hljs-params" style="color: #5c2699; line-height: 26px;">(String[] args)</span>  </span>{
<span/>       <span class="hljs-comment" style="color: #007400; line-height: 26px;">//ExampleFactory&lt;Face&gt; 相当于一个中介人</span>
<span/>        ExampleFactory&lt;Face&gt; factory = <span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">new</span> ExampleFactory&lt;&gt;();
<span/>        <span class="hljs-comment" style="color: #007400; line-height: 26px;">//example 是代理对象</span>
<span/>        Face example = exampleProxy.bind(<span class="hljs-keyword" style="color: #aa0d91; line-height: 26px;">new</span> Example());
<span/>        example.hello(<span class="hljs-string" style="color: #c41a16; line-height: 26px;">"思婷"</span>);
<span/>    }
<span/>}
<span/>-----
<span/>log start
<span/>思婷 Hello!
<span/>log end
<span/></code></pre>
<hr data-tool="mdnice编辑器" style="height: 1px; margin-top: 10px; margin-bottom: 10px; border: none; border-top: 1px solid black; margin: 20px 0;">
<h2 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 22px; color: #009688; padding-left: 10px; margin: 1em auto; border-left: 3px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">欢迎指正文中错误</span><span class="suffix"></span></h2>
<h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">关注公众号，一起交流</span><span class="suffix"></span></h1>
<figure data-tool="mdnice编辑器" style="margin: 0; margin-top: 10px; margin-bottom: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center;"><img src="//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/056d7434d16e4a0db05a6d4c220ffaab~tplv-k3u1fbpfcp-zoom-1.image" alt style="display: block; margin: 0 auto; max-width: 100%;"></figure>
<h1 data-tool="mdnice编辑器" style="margin-top: 30px; margin-bottom: 15px; padding: 0px; font-weight: bold; font-size: 1.8em; color: #009688; margin: 1.2em auto; text-align: center; border-bottom: 1px solid #009688;"><span class="prefix" style="display: none;"></span><span class="content">参考文章</span><span class="suffix"></span></h1>
<ul data-tool="mdnice编辑器" style="margin-top: 8px; margin-bottom: 8px; padding-left: 25px; color: black; list-style-type: disc;">
<li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;"><a href="https://www.jianshu.com/p/9be58ee20dee" style="text-decoration: none; word-wrap: break-word; font-weight: bold; color: #009688; border-bottom: 1px solid #009688;">Java高级特性——反射</a></section></li><li><section style="margin-top: 5px; margin-bottom: 5px; line-height: 26px; text-align: left; color: rgb(1,1,1); font-weight: 500;"><a href="https://www.cnblogs.com/jacksontao/p/8552357.html" style="text-decoration: none; word-wrap: break-word; font-weight: bold; color: #009688; border-bottom: 1px solid #009688;">java反射与动态代理的理解</a></section></li></ul>
</section>