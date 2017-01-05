## Log4j2

* Root Logger总是存在，且处于顶级，可以通过以下两张方式获取：
	* Logger logger = LogManager.getLogger(<u>**LogManager.ROOT_LOGGER_NAME**</u>);
	* Logger logger = LogManager.getRootLogger();
* logger的name是大小写敏感的
* 每一个LoggerContext对应一个有效的配置
* Configuration包含以下几个部分：
	* LoggerConfig(s)
	* Appender(s) 
	* Filter(s)
	* Many others
* 日志系统在应用初始化的过程中进行配置，可以通过读取配置文件或编程两种方式进行配置。当获取Logger的引用时候启动Log4j2系统
* 每一个logger与一个LoggerConfig对象相关联，所有的LoggerConfig对象组成了logger 的层级，他们具有一定的父子关系
* 若不提供配置文件，则将使用Root logger，level为ERROR进行日志记录

## 配置的4种方式
* Using a configuration file written in XML, JSON, YAML or properties file.
* Programmatically, by creating a configuration factory and configuration implementation.
* Programmatically, 调用configuration接口对外暴露的API
* Programmatically, by calling methods on the internal logger class.

## 编程式配置

* 可以使用Log4j2提供的所有ConfigurationFactory接口或使用其默认的实现
* 通过提供相应的配置文件，factory将会返回引用对应配置文件的实例
* 配置实例将与LoggerContext结合来启动日志系统
* 默认的控制台输出将会添加到配置实例中
 
## Log level
 * 每一个logger 与LoggerConfig关联
 * 日志的级别可以在LoggerConfig中设置
 * 每次使用logger触发LogEvent时，logconfig 将记录日志消息，同时将消息向上传递，父级logger将根据其LoggerConfig设置的级别确定是否记录日志
 * 父级可以通过应用Filter或设置additive属性为FALSE来阻止logevent的向上传递
 * 对于父级来说，若其LoggerConfig中指定的level大于LogEvent中的level，则将忽略该日志消息