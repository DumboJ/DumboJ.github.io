<a name="zU3TT"></a>
# 前言
> 是的本狗又踩坑了，这回真的是老大哥升级文档的锅<(￣︶￣)↗[GO!]。问题产生于SchedulerX迁移至SchedulerX2.0过程中，更新POM依赖包后，针对项目中引入H2数据库时本地环境项目可正常启动，发布失败异常。处理类在注入Bean时由于依赖H2初始化失败，大致异常信息如下:

```java
org.springframework.beans.factory.unstatisfiedDependencyException...
  ...
Caused by：org.h2.jdbc.JdbcSQLException:Unspported connection setting "BATCH_JOINS"[90113-190]
```
<a name="MBVXQ"></a>
# 问题分析和解决方案
> spring BeanFactory依赖的bean注入失败，当前H2版本不支持`BATCH_JOINS`参数设置，查看启动日志，本地环境正常启动依赖的Jar包H2版本为`1.4.195`，猜想大概需要该版本Jar包支持项目正常启动。

<a name="QDA6f"></a>
## 问题发现：
升级POM中兼容SchedulerX1.0的Jar包中包含阿里封装的H2 repaired包与未升级前`<dependency><br />   <groupId>org.apache.ignite</groupId><br />   <artifactId>ignite-indexing</artifactId><br />   <version>2.5.0</version><br /></dependency>`中有H2版本冲突。<br />​<br />
<a name="D0E06"></a>
## 解决思路：
<a name="INotU"></a>
#### 思路1-未验证
**A:** 在Maven父POM`<dependencyManagement>`中指定H2版本；
<a name="ygHFf"></a>
#### √思路2-验证可用
**B: **检查项目Maven依赖包中H2 Jar包信息，查看冲突，并使用`<exclusion>`去除错误冲突；
<a name="me2k9"></a>
#### √最终方案
**C: **根据文档修改POM使用较新不再包含H2的SchedulerX升级依赖。<br />​<br />
<a name="tfejN"></a>
# 问题引申
<a name="vM9FF"></a>
## 导语
> 引入Maven包管理或多或少都会遇到一些Jar包冲突的问题，经常出现的场景例举：

- 运行报`NoSuchMethodError`、`ClassNotFoundException`异常；
- 项目中定义了某个Jar包版本为`2.x.x`,打包后变成`3.x.x`；
- 前面案例中出现的环境差异（手动狗头🐕），注入bean功能参数错误导致循环`unstatisfiedDependencyException`异常；
- 本地测试环境运行正常，上线发布后报了一堆`NoSuchMethodError`；
- 项目A引入xxx.jar运行正常，项目B引入xxx.jar后运行报错；

... ... <br />诸如此类问题，根源都来自于`Maven`的Jar包冲突和使用不当的依赖传递。如果不熟悉`Maven`依赖机制时排查起来估计还是挺头疼的。而且`Maven`依赖结构不好的项目，引入新Jar包时也存在风险。基于遇到的上述问题，简单小记分析一波，如有勘误和不足之处，还请读者不吝赐教。<br />​<br />
<a name="md7Ld"></a>
## 循序渐进聊Maven
<a name="x7Wmd"></a>
### 依赖传递的原则和产生Jar包冲突的原理分析

---

<a name="hgWna"></a>
#### 依赖传递原则
<a name="fjkqd"></a>
##### 1. 最短路径优先原则
> 例：项目中同时引入了2个Jar包A.jar和B.jar,2个Jar包中都依赖传递了C.jar包。看图说话，上图：

<a name="dvtiQ"></a>
##### 2. **最先声明优先原则**
> 那讲话了，我们路径要一样长怎么办？那就先来后到，谁先声明用谁。

```xml
	<dependencies>
    	<!--A包含 C.1.5.jar,B包含 C.2.5.jar,A先声明，使用A依赖中的C版本-->
		<dependency>
			<groupId>xx</groupId>
			<artifactId>A</artifactId>
			<version>1.5.0</version>
		</dependency>
        <dependency>
			<groupId>xx</groupId>
			<artifactId>B</artifactId>
			<version>2.5.0</version>
		</dependency>
    </dependencies>
```
<a name="erlQ8"></a>
#### Jar包冲突原理分析
> 例：假如项目中同时引入A,B两个Jar包，两个Jar的依赖传递如下
> A -> X -> Y -> C.1.5.jar
> B -> X -> C.2.5.jar

此时系统中包含C的jar包就有版本冲突，1.5和2.5两个版本。但是classpath中只会依赖一个版本的C包。根据传递依赖的**最短路径优先原则**，项目最终依赖C的Jar包应该是2.5版本。<br />如果Y包中用了C包2.5版本中新的method或参数时，当运行到这段逻辑的时候。就会报NoSuchMethodError 异常。因为本来依赖的是1.5版本，但是因为Jar包冲突Maven选择了2.5版本，2.5版本中更新了不再兼容这个method，Y调用时错误。
> ！！注意
> 不是所有的包冲突都会引起程序异常，相反很多时候项目中都会存在jar包冲突，但并没有造成运行时问题。
> 因为很多依赖传递的Jar包，不管是1.5版本还是2.5版本，都支持调用的方法；
> 只有当高版本Jar包不向下兼容，或者新增了低版本中不存在的API才可能导致此类问题。

<a name="i7dyi"></a>
### 定位冲突以及解决Jar包冲突的简单技巧

---

<a name="qokX5"></a>
#### 定位冲突
<a name="r903l"></a>
##### 1. Idea插件-Maven Helper
> 小小神器，体验不错。本地环境爽的ya批

Idea Plugins里搜索安装后项目pom.xml下方会有`Dependency Analyzer`的tab,单机可视化操作查看项目中依赖，冲突红色高亮显示,具体骚操作和不懂的度娘(●'◡'●)<br />​<br />

- 在编辑器中右键单击 - Run Maven
- 右键单击项目视图工具栏 | (Run|Debug) Maven
- CTRL + ALT + R-“运行Maven目标” 弹出窗口 (您可以在弹出窗口中使用删除键)
- CTRL + SHIFT + ALT + R-“在根模块上运行Maven目标” 弹出窗口 (您可以在弹出窗口中使用删除键)
- 自定义目标:  Settings - Other Settings - Maven Helper 
- 定义快捷方式: Settings - Keymap - Plug-ins - Maven Helper 
- 打开pom文件，单击 `Dependency Analyze` 选项卡，右键单击树中的上下文操作。

​<br />
<a name="kKIRK"></a>
##### 2. mvn命令
> 线上环境就敲吧，mvn命令行也能起舞

```shell
//-Dverbose 显示构建过程被忽略的包
mvn dependency:tree -Dverbose
```


<a name="vExlh"></a>
#### 解决Jar包冲突的小技巧
<a name="SVyfF"></a>
##### 1. 版本锁定
> 如本人问题的解决思路1

```xml
<dependencyManagement>
    <dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>1.4.195</version>
		</dependency>
</dependencyManagement>
```
<a name="xbFkt"></a>
##### 2. 排除法
> 如本人问题的解决思路2

```xml
	<dependency>
			<groupId>com.alibaba.edas</groupId>
			<artifactId>schedulerX-client</artifactId>
			<exclusions>
				<exclusion>
					<groupId>com.h2database</groupId>
					<artifactId>h2</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```
<a name="naNKH"></a>
### 小建议

---

> 思考在pom中如何尽量避免包冲突，让项目依赖干净整洁的小建议

- 封装提供给别人依赖的Jar包，尽可能不依赖不必要Jar包
- 在父POM中定义<dependencyManagement>进行项目基础依赖版本的管理，可以很大程度避免冲突
> 下面两条偷的，凑活用：

- 使用mvn dependency:analyze-only命令用于检测那些声明了但是没被使用的依赖，如有有一些是你自己声明的，那尽量去掉
- 使用mvn dependency:analyze-duplicate命令用来分析重复定义的依赖，清理那些重复定义的依赖
<a name="gHrwM"></a>
# 写在最后
遇到问题不要慌，先根据日志查看问题。再一步步排查验证，找到最优解决方案。爬坑？无所谓的，善于总结，坑踩多了你也就变强了。<br />done ！完结睡大觉。一拳超人埼玉撒库拉酱"我秃了我也就变强了"什么的我是拒绝的<br />(￣o￣) . z Z
