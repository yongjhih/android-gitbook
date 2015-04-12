# Annotation Programming

* 編譯時期 Processor
* 執行時期 Reflection, InvocationHandler

這邊只討論編譯時期 JSR 269 javax.annotation.processing.AbstractProcessor

從常見的 ORM annotations 、json2pojo 、Dagger2、Mockito、Retrofit、AutoValue、AutoParcel、ButterKnife 等函式庫，大量利用 annotations 來精簡聚焦。

## 解析 Annotation

```java
@Example
public class ExampleClass implements Runnable {
    public final String name;
    public ExampleClass(String name) {
        this.name = name;
    }
    @Override
    public void run() {
        System.out.println(name);
    }
}
```

```java
public ExampleProcessor extends AbstractProcessor {
    // ...
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment env) {
        Set<? extends Element> elements = env.getElementAnnotatedWith(Example.class);
        for (Element e : elements) {
            System.out.println(e);
        }
        return false;
    }
}
```

## 產生器的撰寫方法

依據解析結果產生 java file。

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

## 名詞解釋

APT, Annotation-Processing Tool

## See also

* https://speakerdeck.com/jakewharton/annotation-processing-boilerplate-destruction-square-waterloo-2014
* https://github.com/8tory/simple-parse (runtime)
* https://github.com/8tory/auto-parse (source)