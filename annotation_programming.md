# Annotation Programming

* 編譯時期 Processor
* 執行時期 Reflection, InvocationHandler

## 撰寫方法

Template Language:

Apache Velocity Template Language, *.vm

For AVTL example:

```vm
#if (!$pkg.empty)
package $pkg;
#end

#foreach ($i in $imports)
import $i;
#end

@${generated}("com.google.auto.value.processor.AutoAnnotationProcessor")
final class $className implements $annotationName {

## Fields

#foreach ($m in $members)
  #if ($params.containsKey($m.toString()))

  private final $m.type $m;

  #else

  private static final $m.type $m = $m.defaultValue;

  #end
#end
```


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