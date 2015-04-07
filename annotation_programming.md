# Annotation Programming

* 編譯時期 Processor
* 執行時期 Reflection, InvocationHandler

## 撰寫方法

Template Language:

Apache Velocity Template Language, *.vm

Square JavaPoet: https://github.com/square/javapoet

For JavaPoet example:

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.emit(System.out);
```

## See also

* https://github.com/8tory/simple-parse (runtime)
* https://github.com/8tory/auto-parse (source)