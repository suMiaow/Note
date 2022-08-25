# JUnit 5 迁移

Spring Boot 2.2 之后默认JUnit大版本为5，以下迁移部分只适用于 Spring Boot 2.2 之前版本

junit-jupiter 5.8， junit-platform 1.8 后的版本开始支持测试类排序

## pom.xml

```xml
<properties>
    <junit-jupiter.version>5.8.2</junit-jupiter.version>
    <junit-platform.version>1.8.2</junit-platform.version>
</properties>

<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <!-- 关闭 JUnit 4 -->
            <exclusion>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>${junit-jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>${junit-jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>${junit-jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-commons</artifactId>
        <version>${junit-platform.version}</version>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-engine</artifactId>
        <version>${junit-platform.version}</version>
        <scope>test</scope>
    </dependency>

</dependencies>
```

## 代码替换

参考 [Migration Tips](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4-tips)

JUnit 5 开始不强制要求测试类和测试方法的访问级别为 `public`，推荐使用默认访问级别以提高代码可读性

JUnit 4                     | JUnit 5     | 备注
--------------------------- | ----------- | --------------------------------
@Ignore                     | @Disabled
@RunWith                    | @ExtendWith | @SpringBootTest 自带 @ExtendWith
@FixMethodOrder             | @TestMethodOrder
Assert                      | Assertions
@Before                     | @BeforeEach
@Test(expected = XXX.class) | Assertions.assertThrows(XXX.class, () -> {})

## 自定义全局测试类排序

### 配置文件

junit-platform.properties

```properties
junit.jupiter.testclass.order.default=com.xxx.xxx.MyClassOrder
```

### 自定义排序类参考

```java
package com.xxx.xxx;

public class MyClassOrder implements ClassOrderer {
    @Override
    public void orderClasses(ClassOrdererContext context) {
        context.getClassDescriptors().sort((Comparator<ClassDescriptor>) (o1, o2) -> {

            Class<?> class1 = o1.getTestClass();
            Class<?> class2 = o2.getTestClass();
            int superClassCompare = class1.getSuperclass().getSimpleName()
                    .compareTo(class2.getSuperclass().getSimpleName());
            if (superClassCompare != 0) {
                return superClassCompare;
            }

            int mockWeight1 = getMockWeight(class1);
            int mockWeight2 = getMockWeight(class2);
            int mockWeightCompare = Integer.compare(mockWeight1, mockWeight2);
            if (mockWeightCompare != 0) {
                return -mockWeightCompare;
            }

            return class1.getSimpleName().compareTo(class2.getSimpleName());
        });
    }

    private int getMockWeight(Class<?> clazz) {
        int weight = 0;
        if (hasMockBean(clazz)) {
            weight += 1;
        }
        return weight;
    }

    private boolean hasMockBean(Class<?> clazz) {
        Field[] declaredFields = clazz.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            Annotation[] declaredAnnotations = declaredField.getDeclaredAnnotations();
            for (Annotation declaredAnnotation : declaredAnnotations) {
                if (declaredAnnotation instanceof MockBean) {
                    return true;
                }
            }
        }
        return false;
    }

}
```
