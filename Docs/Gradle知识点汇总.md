# 依赖项配置 
| 配置 | 说明
|--|--
| implementation | Gradle 会将依赖项添加到编译类路径，并将依赖项打包到编译输出。不过，当模块配置 implementation 依赖项时，其他模块只有在运行时才能使用该依赖项。
| api | Gradle 会将依赖项添加到编译类路径和编译输出。当一个模块包含 api 依赖项时，会让 Gradle 了解该模块要以传递方式将该依赖项导出到其他模块，以便这些模块在运行时和编译时都可以使用该依赖项。
| compileOnly | Gradle 只会将依赖项添加到编译类路径（也就是说，不会将其添加到编译输出）。
| runtimeOnly | Gradle 只会将依赖项添加到编译输出，以便在运行时使用。也就是说，不会将其添加到编译类路径。
| annotationProcessor | 要添加对作为注解处理器的库的依赖关系，必须使用 annotationProcessor 配置将其添加到注解处理器类路径。 |