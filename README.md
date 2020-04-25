# MybatisStudy
自定义持久层框架、Mybatis 相关概念、原理、源码分析学习

## 一、简答题

### 1、Mybatis动态sql是做什么的？都有哪些动态sql？简述一下动态sql的执行原理？
1）Mybatis动态SQL，可以让我们在XML映射文件内，以XML标签的形式编写动态SQL，完成逻辑判断和动态拼接SQL的功能。
（2）Mybatis提供了9种动态 SQL 标签：if、choose(when、otherwise)、trim(where、set)、foreach。
(3)使用 OGNL 的表达式，从 SQL 参数对象中计算表达式的值,根据表达式的值动态拼接 SQL ，以此来完成动态 SQL 的功能。
### 2、Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？
Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。

在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。它的原理是，使用CGLIB创建目标对象的代理对象，
当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先
保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。
这就是延迟加载的基本原理。
### 3、Mybatis都有哪些Executor执行器？它们之间的区别是什么？
SimpleExecutor（普通执行器，默认）、ReuseExecutor（重用预处理语句prepared statements）、BatchExecutor（重用语句并进行批量的更新）。
	·SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。
	·ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map内，供下一次使用。
	简言之，就是重复使用Statement对象。
	·BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），
	它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。
	·作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

	·Mybatis中如何指定使用哪一种Executor执行器？
	在Mybatis配置文件中，可以指定默认的ExecutorType执行器类型，也可以手动给DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数。

### 4、简述下Mybatis的一级、二级缓存（分别从存储结构、范围、失效场景。三个方面来作答）？
（1）一级缓存是SQLSession级别的缓存，在操作数据库时，需要构造Sqlsession对象，在对象中有一个数据结构(HashMap)用于存储缓存数据。不同的SQLSession之间缓存数据（Hasmap)是不影响的。
		当SqlSession执行commit操作时，为了保证缓存中的数据是最新数据避免脏读，一级缓存数据会被清空，即缓存失效；
	（2）二级缓存是mapper级别的缓存，多个Sqlsession去操作同一个mapper里的SQL语句，多个Sqlsession可以共用一个二级缓存，二级缓存夸SQLSession。
		二级缓存底层数据结构(HashMap)，在mapper中的同一个namespase中，执行更新操作（insert、update、delete）commit后，缓存更新，二级缓存失效
### 5、简述Mybatis的插件运行原理，以及如何编写一个插件？
（1）mybatis所允许的拦截方法有执行器Excutor(update/query/commit/rollback等方法)、SQL语法构建期StatementHandler（prepare、parameterize、batch、update、query等方法）、
	参数处理器ParameterHandler（getParamaterObject、setParameters方法）、结果集处理器ResultSetHandler（handleResultSets、handleOutputParameters等方法），
	在创建四大对象的时候，1，每个创建出来的对象不是直接返回的，而是interceptorChain.pluginAll(parameterHandler);2、获取到所有的interceptor（拦截器）（插件需要实现的接口）；
	调用interceptor.plugin(target);返回target包装后的对象；3、插件机制，我们可以实用插件为目标对象创建一个代理对象；AOP(面向切面)的插件可以为四大对象创建出代理对象，代理对象
	就可以拦截到四大对象的每一个执行；
	（2）mybatis插件接口-Interceptor，是实现自己功能需要实现的接口
	public interface Interceptor {	 
	  //在此方法中实现自己需要的功能，最后执行invocation.proceed()方法，实际就是调用method.invoke(target, args)方法，调用代理类
	  Object intercept(Invocation invocation) throws Throwable;	 
	  //这个方法是将target生成代理类
	  Object plugin(Object target);
	  //在xml中注册Intercept是配置一些属性
	  void setProperties(Properties properties);
	}
	intercept方法，插件的核心方法，plugin方法，生成target的代理对象，setProperties方法，传递插件所需参数，实现插件分三步，即：
	1.编写Intercepror接口的实现类
	2.设置插件的签名，告诉mybatis拦截哪个对象的哪个方法
	3.最后将插件注册到全局配置文件中
