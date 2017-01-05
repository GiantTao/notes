## Programmatic Configuration

* 通过编码来自定义ConfigurationFactory用于启动log4j
* 使用Configurator替代log4j的配置文件（启动后热替换/修改）
* 结合配置文件与代码来初始化配置
* 配置初始化完毕后修改当前的配置


## ConfigurationBuilder API
从2.4版本开始，Log4j提供了ConfigurationBuilder和一系列的ComponentBuilder接口用来创建、组合成一个配置文件Configuration。该两个接口均继承于Builder接口，且提供了默认的实现，如DefaultAppenderComponentBuilder等。
```
ConfigurationBuilder<BuiltConfiguration> builder = ConfigurationBuilderFactory.newConfigurationBuilder();
```
BuiltConfiguration是配置文件的通用配置，由Builder创建，可以扩展该类以增强功能。

ConfigurationBuilder API允许用户通过其创建不同组件的builder实例，并将这些实例与当前的ConfigurationBuilder实例相关联，最终通过调用build()方法返回自定义的配置实例。

ConfigurationBuilder是用于创建配置文件的Builder接口，定义了创建各种component 的方法，如newAppender、newFilter等，返回对应的AppenderComponentBuilder、FilterComponentBuilder等接口实例。
```
FilterComponentBuilder filterBuilder = builder.newFilter("ThresholdFilter", Filter.Result.ACCEPT, Filter.Result.NEUTRAL).
        addAttribute("level", Level.DEBUG);
```

所有component的builder创建完毕后，需要与ConfigurationBuilder对象调用add()方法进行关联以便最终生成配置文件实例。
```
builder.add(appenderBuilder);
builder.add(filterBuilder); 
```

完整代码如下：
```
ConfigurationBuilder<BuiltConfiguration> builder = ConfigurationBuilderFactory.newConfigurationBuilder();
		// 定义一个appender
		AppenderComponentBuilder appenderBuilder = builder.newAppender("Stdout", "CONSOLE").
	            addAttribute("target", ConsoleAppender.Target.SYSTEM_OUT);
		// 定义一个filter
		FilterComponentBuilder filterBuilder = builder.newFilter("ThresholdFilter", Filter.Result.ACCEPT, Filter.Result.NEUTRAL).
        addAttribute("level", Level.DEBUG);
		// 讲appender关联到builder中
		builder.add(appenderBuilder);
		// 将filter关联到builder总
		builder.add(filterBuilder);
		// 生辰最终的配置文件对象
//		builder.toXmlConfiguration();
		// 返回泛型所指定的configuration类型对象
		builder.build();
```

## Components and plugins' Configuration
### 基础组件
Loggers, Appenders, Filter, Properties等可以通过对应的Builder进行配置，如addAttributes、addComponent等方法
### 插件
用户可以进行自定义的组件定制，后续通过builder的newComponent创建自定义的插件

## ConfigurationFactory的选择
初始化期间，log4j将会查找所有可以使用的的ConfigurationFactories，并选择使用其中的一个创建Configuration以供log4j的使用。查找可用的ConfigurationFactories过程如下：

* 使用系统属性log4j.configurationFactory的值所指定的ConfigurationFactory
* ConfigurationFactory.setConfigurationFactory(ConfigurationFactory) 设置使用ConfigurationFactory
* 自定义ConfigurationFactory的实现，并将其作为一个plugin使用，多个ConfigurationFactory可用时，通过priority的值进行选择

ConfigurationFactory有一个supported types的概念，指定了该对象能够处理的配置文件的类型，若提供了配置文件的位置，则该位置中所对应文件类型不匹配则ConfigurationFactory将不会处理该配置文件

## 1. 自定义（扩展）配置工厂
通过自定义的配置工厂，可以使用ConfigurationBuilder来创建一个配置文件。需要**重写getConfiguration()**方法，该方法中使用builder创建并返回配置对象。当LoggerContext对象创建后，该配置文件将会自动与之关联。

## 2. 使用Configurator对已有配置修改
一旦构建了Configuration对象，可将该对象传递到任何一个Configurator.initialize方法来重新加载Log4j的配置
LoggerContext ctx = Configurator.intitialize(builder.build());

## 3. 配置文件与代码并用
应用场景：使用配置文件进行个性化配置，使用编程进行不可修改的配置
例：扩展 XMLConfiguration以手动添加固定的appender和loggerconfig到配置中

* 扩展XMLConfiguration，在**doConfigure()**中添加自定义的appender以及loggerconfig
* 扩展ConfigurationFactory，重写getConfiguration方法，返回自定义的XMLConfiguration

## 4. 代码修改经过初始化的配置
应用有的时候需要在actual中配置日志的隔离，log4j支持这么做，但是有两点需要注意：

* 若配置文件被修改了，则会自动重新加载配置文件，但是之前通过手动修改的配置将会丢失 
* 对正在使用的配置文件的修改需要确保所有调用的修改方法（如addAppender、addLogger）保持同步synchronized

基于此，建议通过扩展其中一个标准的Configuration类，重写setup方法，通过<u>最先调用```super.setup()```方法</u>后<u>添加自定义配置的方</u>式实现配置文件的定制

```
final LoggerContext ctx = (LoggerContext) LogManager.getContext(false);
final Configuration config = ctx.getConfiguration();
Layout layout = PatternLayout.createLayout(PatternLayout.SIMPLE_CONVERSION_PATTERN, config, null,null,null, null);
Appender appender = FileAppender.createAppender("target/test.log", "false", "false", "File", "true","false", "false", "4000", layout, null, "false", null, config);
appender.start();
config.addAppender(appender);
AppenderRef ref = AppenderRef.createAppenderRef("File", null, null);
AppenderRef[] refs = new AppenderRef[] {ref};
LoggerConfig loggerConfig = LoggerConfig.createLogger("false", "info", "org.apache.logging.log4j","true", refs, null, config, null );
loggerConfig.addAppender(appender, null, null);
config.addLogger("org.apache.logging.log4j", loggerConfig);
ctx.updateLoggers();
```

全文出自[官方文档](http://logging.apache.org/log4j/2.x/manual/customconfig.html)