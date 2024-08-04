# 单元测试-JUnit5 

 4.1.

 整合 

SpringBoot 提供一系列测试工具集及注解方便我们进行测试。

spring-boot-test提供核心测试能力，spring-boot-test-autoconfigure 提供测试的一些自动配置。

我们只需要导入spring-boot-starter-test 即可整合测试







spring-boot-starter-test 默认提供了以下库供我们测试使用

●[JUnit 5](https://junit.org/junit5/)

●[Spring Test](https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/testing.html#integration-testing)

●[AssertJ](https://assertj.github.io/doc/)

●[Hamcrest](https://github.com/hamcrest/JavaHamcrest)

●[Mockito](https://site.mockito.org/)

●[JSONassert](https://github.com/skyscreamer/JSONassert)

●[JsonPath](https://github.com/jayway/JsonPath)



 4.2. 测试 

 4.2.0 组件测试 

直接@Autowired容器中的组件进行测试

 4.2.1 注解 

JUnit5的注解与JUnit4的注解有所变化

https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

●@Test :表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试

●@ParameterizedTest :表示方法是参数化测试，下方会有详细介绍

●@RepeatedTest :表示方法可重复执行，下方会有详细介绍

●@DisplayName :为测试类或者测试方法设置展示名称

●@BeforeEach :表示在每个单元测试之前执行

●@AfterEach :表示在每个单元测试之后执行

●@BeforeAll :表示在所有单元测试之前执行

●@AfterAll :表示在所有单元测试之后执行

●@Tag :表示单元测试类别，类似于JUnit4中的@Categories

●@Disabled :表示测试类或测试方法不执行，类似于JUnit4中的@Ignore

●@Timeout :表示测试方法运行如果超过了指定时间将会返回错误

●@ExtendWith :为测试类或测试方法提供扩展类引用





 4.2.2 断言 

| 方法              | 说明                                 |
| ----------------- | ------------------------------------ |
| assertEquals      | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals   | 判断两个对象或两个原始类型是否不相等 |
| assertSame        | 判断两个对象引用是否指向同一个对象   |
| assertNotSame     | 判断两个对象引用是否指向不同的对象   |
| assertTrue        | 判断给定的布尔值是否为 true          |
| assertFalse       | 判断给定的布尔值是否为 false         |
| assertNull        | 判断给定的对象引用是否为 null        |
| assertNotNull     | 判断给定的对象引用是否不为 null      |
| assertArrayEquals | 数组断言                             |
| assertAll         | 组合断言                             |
| assertThrows      | 异常断言                             |
| assertTimeout     | 超时断言                             |
| fail              | 快速失败                             |

 4.2.3 嵌套测试 

JUnit 5 可以通过 Java 中的内部类和@Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。





Java

复制代码

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

@DisplayName("A stack")

class TestingAStackDemo {

​    Stack<Object> stack;

​    @Test

​    @DisplayName("is instantiated with new Stack()")

​    void isInstantiatedWithNew() {

​        new Stack<>();

​    }

​    @Nested

​    @DisplayName("when new")

​    class WhenNew {

​        @BeforeEach

​        void createNewStack() {

​            stack = new Stack<>();

​        }

​        @Test

​        @DisplayName("is empty")

​        void isEmpty() {

​            assertTrue(stack.isEmpty());

​        }

​        @Test

​        @DisplayName("throws EmptyStackException when popped")

​        void throwsExceptionWhenPopped() {

​            assertThrows(EmptyStackException.class, stack::pop);

​        }

​        @Test

​        @DisplayName("throws EmptyStackException when peeked")

​        void throwsExceptionWhenPeeked() {

​            assertThrows(EmptyStackException.class, stack::peek);

​        }

​        @Nested

​        @DisplayName("after pushing an element")

​        class AfterPushing {

​            String anElement = "an element";

​            @BeforeEach

​            void pushAnElement() {

​                stack.push(anElement);

​            }

​            @Test

​            @DisplayName("it is no longer empty")

​            void isNotEmpty() {

​                assertFalse(stack.isEmpty());

​            }

​            @Test

​            @DisplayName("returns the element when popped and is empty")

​            void returnElementWhenPopped() {

​                assertEquals(anElement, stack.pop());

​                assertTrue(stack.isEmpty());

​            }

​            @Test

​            @DisplayName("returns the element when peeked but remains not empty")

​            void returnElementWhenPeeked() {

​                assertEquals(anElement, stack.peek());

​                assertFalse(stack.isEmpty());

​            }

​        }

​    }

}



 4.2.4 参数化测试 

参数化测试是JUnit5很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能，也为我们的单元测试带来许多便利。



利用@ValueSource等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。



@ValueSource: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型

@NullSource: 表示为参数化测试提供一个null的入参

@EnumSource: 表示为参数化测试提供一个枚举入参

@CsvFileSource：表示读取指定CSV文件内容作为参数化测试入参

@MethodSource：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)







Java

复制代码

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

@ParameterizedTest

@ValueSource(strings = {"one", "two", "three"})

@DisplayName("参数化测试1")

public void parameterizedTest1(String string) {

​    System.out.println(string);

​    Assertions.assertTrue(StringUtils.isNotBlank(string));

}

@ParameterizedTest

@MethodSource("method")    //指定方法名

@DisplayName("方法来源参数")

public void testWithExplicitLocalMethodSource(String name) {

​    System.out.println(name);

​    Assertions.assertNotNull(name);

}

static Stream<String> method() {

​    return Stream.of("apple", "banana");

}