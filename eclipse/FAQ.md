# Eclipse 2011-06版本安装lombok插件问题

在`eclipse.ini`文件中添加一行
`-javaagent:F:\repository\org\projectlombok\lombok\1.18.20\lombok-1.18.20.jar`

启动**Eclipse**后出现报错

```
java.lang.reflect.InaccessibleObjectException: Unable to make protected final java.lang.Class java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int) throws java.lang.ClassFormatError accessible: module java.base does not "opens java.lang" to unnamed module @472a11ae
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:357)
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:297)
	at java.base/java.lang.reflect.Method.checkCanSetAccessible(Method.java:199)
	at java.base/java.lang.reflect.Method.setAccessible(Method.java:193)
	at org.eclipse.osgi.internal.loader.ModuleClassLoader.overrideLoadResult(ModuleClassLoader.java:86)
	at org.eclipse.osgi.internal.loader.ModuleClassLoader.loadClass(ModuleClassLoader.java)
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:519)
	at org.eclipse.jdt.internal.compiler.parser.Parser.endParse(Parser.java:11750)
	at org.eclipse.jdt.internal.compiler.parser.Parser.parse(Parser.java:12949)
	at org.eclipse.jdt.internal.compiler.parser.Parser.parse(Parser.java:13176)
	at org.eclipse.jdt.internal.compiler.parser.Parser.parse(Parser.java:13133)
	at org.eclipse.jdt.internal.compiler.parser.Parser.dietParse(Parser.java:11521)
	at org.eclipse.jdt.internal.compiler.Compiler.internalBeginToCompile(Compiler.java:850)
	at org.eclipse.jdt.internal.compiler.Compiler.beginToCompile(Compiler.java:394)
	at org.eclipse.jdt.internal.compiler.Compiler.compile(Compiler.java:444)
	at org.eclipse.jdt.internal.compiler.Compiler.compile(Compiler.java:426)
	at org.eclipse.jdt.internal.core.builder.AbstractImageBuilder.compile(AbstractImageBuilder.java:377)
	at org.eclipse.jdt.internal.core.builder.BatchImageBuilder.compile(BatchImageBuilder.java:214)
	at org.eclipse.jdt.internal.core.builder.AbstractImageBuilder.compile(AbstractImageBuilder.java:309)
	at org.eclipse.jdt.internal.core.builder.BatchImageBuilder.build(BatchImageBuilder.java:79)
	at org.eclipse.jdt.internal.core.builder.JavaBuilder.buildAll(JavaBuilder.java:272)
	at org.eclipse.jdt.internal.core.builder.JavaBuilder.build(JavaBuilder.java:187)
	at org.eclipse.core.internal.events.BuildManager$2.run(BuildManager.java:846)
	at org.eclipse.core.runtime.SafeRunner.run(SafeRunner.java:45)
	at org.eclipse.core.internal.events.BuildManager.basicBuild(BuildManager.java:229)
	at org.eclipse.core.internal.events.BuildManager.basicBuild(BuildManager.java:277)
	at org.eclipse.core.internal.events.BuildManager$1.run(BuildManager.java:330)
	at org.eclipse.core.runtime.SafeRunner.run(SafeRunner.java:45)
	at org.eclipse.core.internal.events.BuildManager.basicBuild(BuildManager.java:333)
	at org.eclipse.core.internal.events.BuildManager.basicBuildLoop(BuildManager.java:385)
	at org.eclipse.core.internal.events.BuildManager.build(BuildManager.java:406)
	at org.eclipse.core.internal.events.AutoBuildJob.doBuild(AutoBuildJob.java:154)
	at org.eclipse.core.internal.events.AutoBuildJob.run(AutoBuildJob.java:244)
	at org.eclipse.core.internal.jobs.Worker.run(Worker.java:63)
```

解决办法
  - 使用`Java 15`启动**Eclipse**或
  - 在`eclipse.ini`添加如下配置
  
  ```
	`-javaagent:F:\repository\org\projectlombok\lombok\1.18.20\lombok-1.18.20.jar`
	`--illegal-access=warn`
	`--add-opens java.base/java.lang=ALL-UNNAMED`
  ```

