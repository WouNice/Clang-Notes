# Gmock教程

## 引言

### Google Mock简介

Google Mock是由Google开发的一种用于C++的模拟（mocking）框架，它是Google Test测试框架的一部分。gmock允许开发者创建模拟对象，这些对象可以在单元测试中代替真实的依赖项，使得测试更加灵活和独立。通过使用gmock，开发者可以专注于测试代码逻辑的正确性，而不必担心外部依赖的复杂性。

### 为什么选择Google Mock

在众多C++测试框架中，gmock以其强大的功能和易用性脱颖而出。以下是选择gmock的一些主要理由：

-   灵活性：gmock支持高度定制化的模拟行为，可以模拟复杂的依赖关系。

-   易用性：gmock的API设计简洁直观，易于学习和使用。

-   社区支持：作为Google的产品，gmock拥有活跃的社区和丰富的文档资源。

-   集成性：gmock可以与Google Test无缝集成，提供一站式的测试解决方案。

## Google Mock基础

### 测试的重要性

在深入探讨Google Mock之前，我们首先要认识到测试在软件开发中的重要性。测试是确保软件质量的关键环节，它帮助我们发现并修复潜在的错误和缺陷。单元测试是测试中最基本的形式，它允许我们独立地测试代码的各个部分。

### Google Mock的安装和配置

在开始使用Google Mock之前，我们需要先进行安装和配置。以下是安装Google Mock的基本步骤：

1.  获取Google Test：由于Google Mock是Google Test的一部分，我们需要先获取Google Test的源代码。
2.  编译Google Test：根据你的操作系统和编译器，编译Google Test库。
3.  配置项目：在你的项目中配置Google Test和Google Mock的头文件路径和库路径。

### 测试用例的结构

一个典型的测试用例通常包括以下几个部分：

-   测试构建：设置测试所需的环境和条件。

-   执行测试：运行被测试的代码。

-   断言：验证代码的输出是否符合预期。

-   清理：测试完成后清理环境。

```c++
#include <gtest/gtest.h>
class MyMathTest : public ::testing::Test {
protected:
    void SetUp() override {
        // 测试构建
    }
    void TearDown() override {
        // 清理
    }
};

TEST_F(MyMathTest, TestAddition) {
    // 执行测试
    int result = 3 + 2;
    // 断言
    EXPECT_EQ(5, result);
}
```

### 编写测试用例的步骤

编写测试用例通常遵循以下步骤：

1.  确定测试目标：明确你想要测试的代码部分或功能。
2.  编写测试代码：使用Google Test的宏和断言来编写测试逻辑。
3.  运行测试：编译并运行测试，查看结果是否符合预期。
4.  分析和调整：根据测试结果调整测试用例或被测试的代码。

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

class NetworkService {
    virtual std::string fetchData(){return "";}
};

class MockNetworkService : public NetworkService {
public:
    MOCK_METHOD(std::string, fetchData, (), (override));
};
TEST(NetworkServiceTest, FetchDataTest) {
    MockNetworkService mockService;
    EXPECT_CALL(mockService, fetchData())
        .WillOnce(testing::Return("Mocked Data"));
    std::string result = mockService.fetchData();
    EXPECT_EQ("Mocked Data", result);
}
```

## 创建测试用例

### 测试用例的重要性

测试用例是单元测试的核心，它们定义了测试的输入、执行过程和预期结果。编写高质量的测试用例可以确保你的代码在修改和扩展过程中保持稳定和可靠。

### 测试用例的基本原则

在编写测试用例时，应遵循以下基本原则：

-   独立性：每个测试用例应独立于其他测试用例运行，不应依赖外部状态。
-   可重复性：无论何时何地运行测试用例，都应得到相同的结果。
-   自动化：测试用例应自动化执行，减少人工干预。
-   覆盖全面：测试用例应覆盖所有重要的功能点和边界条件。

### 测试用例的结构

一个完整的测试用例通常包括以下部分

-   前置条件：测试开始前需要满足的条件。

-   输入数据：测试用例的输入。
-   执行步骤：执行测试的具体步骤。
-   预期结果：测试完成后期望的结果。
-   验证逻辑：验证实际结果是否符合预期结果的逻辑。

### 示例：简单的加法测试用例

```c++
#include <gtest/gtest.h>
class Adder {
public:
    int add(int a, int b) {
        return a + b;
    }
};
TEST(AdderTest, HandlesPositiveNumbers) {
    Adder adder;
    EXPECT_EQ(5, adder.add(2, 3));
}
TEST(AdderTest, HandlesNegativeNumbers) {
    Adder adder;
    EXPECT_EQ(-1, adder.add(-2, 1));
}
TEST(AdderTest, HandlesZero) {
    Adder adder;
    EXPECT_EQ(0, adder.add(0, 0));
}
```

### 测试用例的参数化

```c++
#include <gtest/gtest.h>
#include <tuple>
class Adder {
public:
    int add(int a, int b) {
        return a + b;
    }
};
class AdderTest : public ::testing::TestWithParam<std::tuple<int, int, int>> {};

TEST_P(AdderTest, AddReturnsExpectedResult) {
    Adder adder;
    int a, b, expected;
    std::tie(a, b, expected) = GetParam();
    EXPECT_EQ(expected, adder.add(a, b));
}
INSTANTIATE_TEST_CASE_P(AdderTests, AdderTest,
             ::testing::Values(std::make_tuple(2, 3, 5),
                            std::make_tuple(-2, 1, -1),
                            std::make_tuple(0, 0, 0)));
```

### 测试用例的组织

在大型项目中，合理组织测试用例非常重要。通常，测试用例应该按照它们测试的类或模块来组织。使用命名约定和目录结构可以帮助维护和查找测试用例。

### 示例：测试复杂逻辑

让我们考虑一个更复杂的例子，测试一个简单的银行账户类：

```c++
#include <gtest/gtest.h>
class BankAccount {
public:
    BankAccount(int balance) : balance_(balance) {}
    void deposit(int amount) { balance_ += amount; }
    int getBalance() const { return balance_; }
private:
    int balance_;
};
TEST(BankAccountTest, DepositIncreasesBalance) {
    BankAccount account(100);
    account.deposit(50);
    EXPECT_EQ(150, account.getBalance());
}
TEST(BankAccountTest, DepositWithNegativeAmountThrows) {
    BankAccount account(100);
    account.deposit(-50);
    EXPECT_EQ(50, account.getBalance());
}
```

## Mocking的基本概念

### 什么是Mocking？

Mocking是一种测试技术，它允许测试者模拟（mock）一个对象或接口的行为，以便在测试中隔离被测试的代码。Mock对象通常用于替代真实的依赖项，使得测试可以独立于外部系统或组件运行。

### Mocking与Stub的区别

-   Mock：通常用于验证被测试代码对依赖项的调用是否正确，包括调用次数、参数、调用顺序等。

-   Stub：返回预定义的响应数据，主要用于测试代码的逻辑，而不是验证调用的正确性。

### 为什么使用Mocking？

-   隔离性：Mocking允许测试独立于外部系统运行，提高了测试的稳定性和可靠性。

-   灵活性：可以模拟各种复杂的情况，包括错误、异常、延迟等。

-   效率：避免了与外部系统的交互，加快了测试执行的速度。

### 使用Google Mock进行Mocking

Google Mock提供了一套丰富的API来创建和配置Mock对象。以下是使用Google Mock进行Mocking的基本步骤：

1.  定义Mock接口：根据需要Mock的类或接口定义一个Mock版本。
2.  使用MOCK_METHOD宏：在Mock接口中定义Mock方法。
3.  设置期望：使用EXPECT_CALL来设置Mock对象的期望行为。

4.  验证调用：在测试结束时，Google Mock会自动验证Mock对象的调用是否符合期望。

### 示例：Mock一个简单的依赖

```c++
#include <gtest/gtest.h>
class Database {
public:
    virtual ~Database() {}
    virtual std::string query(const std::string& sql) const{return sql;}
};
class MockDatabase:public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
};
class DataProcessor {
public:
    explicit DataProcessor(Database* db) : db_(db) {}
    std::string process(const std::string& data) {
        return data + " processed with " + db_->query("SELECT * FROM table");
}
private:
    Database* db_;
};

TEST(DataProcessorTest, ProcessesDataWithDatabaseQuery) {
    MockDatabase mockDb;
    DataProcessor processor(&mockDb);
    EXPECT_CALL(mockDb, query("SELECT * FROM table"))
        .WillOnce(testing::Return("Mocked query result"));
    std::string result = processor.process("Data");
    EXPECT_EQ("Data processed with Mocked query result", result);
}
```

### 高级Mocking技巧

-   序列化调用：使用EXPECT_CALL的InSequence来模拟方法调用的顺序。

-   任意次数的调用：使用Times()来指定方法可以被调用的次数范围。

-   组合Mock和Stub：在同一个Mock对象中同时使用Mock和Stub的行为。

#### 使用Mock验证方法调用顺序

**1） 期望全部顺序调用**

```c++
#include <gtest/gtest.h>
using ::testing::_;
using ::testing::InSequence;

class MyClass {
public:
    virtual bool firstMethod(int){return true;}
    virtual bool secondMethod(int){return true;}
};

class MockMyClass {
public:
    MOCK_METHOD(bool, firstMethod,(int));
    MOCK_METHOD(bool, secondMethod,(int));
};
TEST(SomeClassTest, CallsMethodsInOrder) {
    MockMyClass mock;
    {
        InSequence seq;
        EXPECT_CALL(mock, firstMethod(testing::_))
            .WillOnce(testing::Return(true));
        EXPECT_CALL(mock, secondMethod(testing::_))
            .WillOnce(testing::Return(true));
    }

    //EXPECT_EQ(true, mock.firstMethod(1));
    //EXPECT_EQ(true, mock.secondMethod(2));
    EXPECT_EQ(true, mock.secondMethod(2));
    EXPECT_EQ(true, mock.firstMethod(1));
}
```

按照如上代码执行将报错

```
[  FAILED  ] SomeClassTest.CallsMethodsInOrder (1 ms)
[----------] 1 test from SomeClassTest (1 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (1 ms total)
[  PASSED  ] 0 tests.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] SomeClassTest.CallsMethodsInOrder
```

如果先执行mock.firstMethod(1)，再执行mock.secondMethod(2)就正确了。

**2）期望部分顺序调用**

EXPECT_CALL的After从句可以执行按部分顺序调用。

通过使用InSequence()从句

```c++
using ::testing::Sequence;
  ...
  Sequence s1, s2;
  EXPECT_CALL(foo, A()).InSequence(s1, s2);
  EXPECT_CALL(bar, B()).InSequence(s1);
  EXPECT_CALL(bar, C()).InSequence(s2);
  EXPECT_CALL(foo, D()).InSequence(s2);
```

执行顺序

按如下图的方式 (where s1 is A -> B, and s2 is A -> C -> D):

```
S1:A--------B
   |
S2:A--------C ---------D
```

#### 任意次数的调用

EXPECT_CALL(mock_object,method_name(matchers...)) 创建一个mock对象mock_object，这个对象有一个名为method_name的方法，方法的参数为matchers…。EXPECT_CALL必须在任何mock对象之前使用。以下方法的调用，必须按以下顺序进行：

```c++
EXPECT_CALL(mock_object, method_name(matchers...))
    .With(multi_argument_matcher)    // Can be used at most once
    .Times(cardinality)             // Can be used at most once
    .InSequence(sequences...)       // Can be used any number of times
    .After(expectations...)         // Can be used any number of times
    .WillOnce(action)               // Can be used any number of times
    .WillRepeatedly(action)         // Can be used at most once
    .RetiresOnSaturation();         // Can be used at most once
```

#### 组合Mock和Stub

在同一个Mock对象中，我们可以同时使用Mock和Stub的行为，这可以让我们在不同的测试场景下灵活地控制Mock对象的行为。

```c++
#include <gtest/gtest.h>

using ::testing::_;
using ::testing::Invoke;
class Database {
public:
    virtual ~Database() {}
    virtual std::string query(const std::string& sql) const{return sql;}
    virtual void connect(){};
};
class MockDatabase:public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
    MOCK_METHOD(void, connect, (), (override));
};
TEST(SomeClassTest, CombineMockAndStub) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, connect()) //桩
        .Times(1);
    ON_CALL(mockDb, query(_)) //mock
        .WillByDefault(testing::Return("Stubbed Query Result"));

    mockDb.connect();
    EXPECT_EQ("Stubbed Query Result",mockDb.query("query"));
}
```

### Mocking复杂类型

有时我们需要Mock返回复杂类型的方法。Google Mock提供了NiceMock和StrictMock等工厂函数，它们可以简化Mock对象的创建和配置。

#### NiceMock

NiceMock 是 Google Mock (gmock) 提供的一个包装器，它允许你创建更为"宽容"的模拟对象。与 StrictMock 不同，NiceMock 不会对未指定的调用产生错误，而是会默认生成一个合适的返回值或者行为。

以下是一个简单的 NiceMock 示例：

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::Return;

// 定义一个接口
class MyInterface {
public:
    virtual int Foo() = 0;
    virtual void Bar(int x) = 0;
}
// 为接口创建mock类
class MockMyInterface : public MyInterface {
public:
    MOCK_METHOD(int, Foo, (), (override));
    MOCK_METHOD(void, Bar, (int), (override));
};
TEST(NiceMockTest, NiceMockExample) {
    // 使用NiceMock包装MockMyInterface
    ::testing::NiceMock nice_mock;

      // 指定Foo()的期望行为，但Bar()的期望行为没有指定
    EXPECT_CALL(nice_mock, Foo()).WillOnce(Return(42));
      // 调用Foo()，应该返回42
    EXPECT_EQ(nice_mock.Foo(), 42);
      // 调用未指定期望行为的Bar()，NiceMock不会报错，而是默认执行一个空操作
    nice_mock.Bar(10);
}
```

#### StrictMock

StrictMock是 Google Mock (gmock) 提供的另一个包装器，与 NiceMock 不同，StrictMock 对未指定的调用会报错。这意味着你必须为 mock 对象的所有方法指定期望行为，否则如果在测试期间调用了未设置期望的方法，测试将会失败。以下是一个使用 StrictMock 的示例：

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::Return;
// 定义一个接口
class MyInterface {
public:
    virtual int Foo() = 0;
    virtual void Bar() = 0;
};
// 为接口创建 mock 类
class MockMyInterface : public MyInterface {
public:
    MOCK_METHOD(int, Foo, (), (override));
    MOCK_METHOD(void, Bar, (), (override));
};
TEST(StrictMockTest, StrictMockExample) {
    // 使用 StrictMock 包装 MockMyInterface
    ::testing::StrictMock strict_mock;


    // 为 Foo() 和 Bar() 方法指定期望行为
    EXPECT_CALL(strict_mock, Foo()).WillOnce(Return(42));
    EXPECT_CALL(strict_mock, Bar());

    // 调用 Foo()，应该返回 42
    EXPECT_EQ(strict_mock.Foo(), 42);

    // 调用 Bar()，满足之前设置的期望
    strict_mock.Bar();

    // 如果尝试调用未设置期望的mock方法，测试将会失败
    // 例如：strict_mock.SomeUndeclaredMethod(); // 这将导致测试失败
}
```

-   使用NiceMock方法，尝试调用未设置期望的mock方法，测试将失败；
-   使用StrictMock方法，尝试调用未设置期望的mock方法，测试将失败。

## 高级Mocking技巧

### 引言

高级Mocking技巧是单元测试中的进阶技能，它可以帮助测试者更精确地模拟复杂场景，从而提高测试的覆盖率和质量。在本部分，我们将深入探讨一些高级Mocking技巧，并通过丰富的示例来展示它们的应用。

### 使用ON_CALL自定义Mock行为

ON_CALL宏允许我们为Mock对象的方法指定默认行为，这在测试中非常有用，特别是当Mock对象的方法需要在不同的测试用例中重复调用时。

签名：

```c++
ON_CALL(mock_object,method_name(matchers...))
```

定义了调用mock_object对象的method_name方法后执行的动作。如果matchers被忽略了，类似于每个参数都是"_"，即可以匹配任意值。

```c++
ON_CALL(mock_object, method_name(matchers...))
    .With(multi_argument_matcher)  // Can be used at most once
    .WillByDefault(action);        // Required
```

-   With(multi_argument_matcher)：将参数作为整体匹配。GoogleTest将所有参数作为一个tuple给matcher，multi_argument_matcher最终将是Matcher<std::tuple<A1, …, An>>。这个调用最多一次
-   WillByDefault(action)：指定调用mock函数后执行的动作。这个调用有且只有一次。

看一个例子

```c++
#include <gmock/gmock.h>

class Database {
public:
    virtual ~Database() {}
    virtual std::string query(const std::string& sql) const{return sql;}
};

class MockDatabase:public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
};
class DataProcessor {
public:
    explicit DataProcessor(Database* db) : db_(db) {}
    std::string process(const std::string& data) {
        return data + " processed with " + db_->query("SELECT * FROM table");
}
private:
    Database* db_;
};

TEST(AdvancedMockingTest, CustomDefaultBehavior) {
    MockDatabase mockDb;
    ON_CALL(mockDb, query(testing::_))
        .WillByDefault(testing::Return("Default Query Result"));

    EXPECT_EQ("Default Query Result", mockDb.query("Any SQL"));
}
```

### 模拟异常

在某些情况下，我们可能需要模拟方法抛出异常，以测试被测试对象对异常的处理能力。

```c++
class Calculator {
public:
     virtual int divide(int a, int b) = 0;
};
class MockCalculator : public Calculator {
public:
    MOCK_METHOD(int, divide, (int a, int b), (override));
};
TEST(CalculatorTest, TestCalculator) {
    MockCalculator mockCalculator;
    // 设置mock对象的行为
    //…
    EXPECT_CALL(mockCalculator, divide(testing::_,0))
        .WillOnce(testing::Throw(std::runtime_error("Divisor cannot be 0")));
    // 验证mock对象的行为  //…
    EXPECT_THROW(mockCalculator.divide(6, 0), std::runtime_error);
}
```

通过testing::Throw和EXPECT_THROW来模拟异常。

### 使用Invoke回调函数

Invoke函数允许我们在Mock方法中调用一个回调函数，这在需要根据输入参数动态返回结果时非常有用。

```c++
#include <gmock/gmock.h>
class Database {
public:
    virtual ~Database() {}
    virtual std::string query(const std::string& sql) const{return sql;}
};
class MockDatabase:public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
};
class DataProcessor {
public:
    explicit DataProcessor(Database* db) : db_(db) {}
    std::string process(const std::string& data) {
        return data + " processed with " + db_->query("SELECT * FROM table");
    }
private:
    Database* db_;
};
TEST(AdvancedMockingTest, UseInvokeForDynamicReturn) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, query(testing::_))
        .WillOnce(testing::Invoke([](const std::string& sql) {
            return "Result for " + sql;
        }));
    EXPECT_EQ("Result for SELECT * FROM table", mockDb.query("SELECT * FROM table"));
}
```

### 模拟复杂的数据结构

当Mock方法返回复杂的数据结构时，我们可以使用WithArgs来匹配特定的参数，并返回对应的结果。

```c++
// 定义一个结构体
struct QueryResult {
    std::string name;
    int age;
};

// 假设的函数接口
class DatabaseService {
public:
    virtual QueryResult executeQuery(const std::string& sql) = 0;
};
// Mock类
class MockDatabaseService : public DatabaseService {
public:
    MOCK_METHOD(QueryResult, executeQuery, (const std::string&), (override));
};
TEST(MockDatabaseService, MockComplexReturnType) {
    MockDatabaseService mockDb;
    QueryResult alice{"Alice", 30};
    EXPECT_CALL(mockDb, executeQuery(testing::_))
        .Times(2)
        .WillOnce(testing::WithArgs<0>(testing::Invoke([](const std::string& sql) {
            return QueryResult{"Alice", 30};
        })))
        .WillRepeatedly(testing::WithArgs<0>(testing::Invoke([](const std::string& sql) {
            return QueryResult{"Bob", 40};
        })));
    QueryResult result = mockDb.executeQuery("Alice");
    EXPECT_EQ("Alice",result.name);
    EXPECT_EQ(30,result.age);
    result = mockDb.executeQuery("Bob");
    EXPECT_EQ("Bob",result.name);
    EXPECT_EQ(40,result.age);
}
```

### 模拟方法调用的副作用

有时，我们可能需要模拟方法调用时产生的副作用，例如修改共享状态或触发回调。

```c++
#include <gmock/gmock.h>
using ::testing::_;
using ::testing::Invoke;
class Database {
public:
    virtual ~Database() {}
    virtual std::string update(int id, const std::string& value) const{return value;}
};
class MockDatabase:public Database {
public:
    MOCK_METHOD(std::string, update, (int id, const std::string& value), (const, override));
};
TEST(MockDatabase, SimulateSideEffects) {
    MockDatabase mockDb;
    std::string sideEffect;
    EXPECT_CALL(mockDb, update(_, _))
        .WillOnce(Invoke([&sideEffect](int id, const std::string& value) {
            return sideEffect = "Updated " + std::to_string(id) + " to " + value;
        }));

    mockDb.update(1, "value");
    EXPECT_EQ("Updated 1 to value", sideEffect);
}
```

### 模拟方法的多次调用

使用Times可以指定Mock方法被调用的次数，这对于测试循环或递归逻辑非常有用。

```c++
#include <gmock/gmock.h>

class NetworkService {
    virtual std::string fetchData(){return "";}
};
class MockNetworkService : public NetworkService {
public:
    MOCK_METHOD(std::string, fetchData, (), (override));
};

class MockDatabase:public NetworkService {
public:
    MOCK_METHOD(std::string, fetchData, (), (override));
};
TEST(AdvancedMockingTest, MultipleInvocations) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, fetchData())
        .Times(3)
        .WillRepeatedly(testing::Return("Data"));

    for (int i = 0; i < 3; ++i) {
        EXPECT_EQ("Data", mockDb.fetchData());
    }
}
```

### 使用UnorderedElementsAre匹配容器

当Mock方法的参数是容器时，我们可以使用UnorderedElementsAre来匹配容器中的元素，而不需要指定元素的顺序。

```c++
#include <gmock/gmock.h>
class Database {
public:
    virtual ~Database() {}
    virtual void insert(const std::vector& values){}
};
class MockDatabase{
public:
    MOCK_METHOD(void, insert, (const std::vector& values));
};
// 使用MockDatabase类的方法
void SomeFunctionUsingMockDatabase(MockDatabase& db) {
    std::vector values = {3, 1, 2};

    db.insert(values);
}
//测试代码
TEST(AdvancedMockingTest, MatchUnorderedContainer) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, insert(testing::UnorderedElementsAre(1, 2, 3)));
    SomeFunctionUsingMockDatabase(mockDb);
}
```

## Google Mock的断言和期望

### 断言的重要性

断言是单元测试中验证代码逻辑正确性的关键工具。它们允许测试者指定预期结果，并在结果不符合预期时立即报告错误。

### 基本断言Google Test

提供了一系列基本断言，用于验证测试结果是否符合预期。

-   ASSERT_TRUE：如果条件为假，则测试失败。

-   EXPECT_TRUE：同上，但条件为假时测试继续执行。

-   ASSERT_EQ：验证两个值是否相等，如果不相等则测试失败。

```c++
TEST(BasicAssertionsTest, ChecksEquality) {
    int result = someFunction();
    ASSERT_EQ(42, result) << "someFunction did not return the expected value.";}
```

### 期望调用（Expectations）

期望调用是 Google Mock 中用于指定 Mock 对象在测试中应该如何被调用的机制。

-   EXPECT_CALL：创建一个期望调用。

-   Times：指定期望调用的次数。

```c++
TEST(MockExpectationsTest, CallsFunctionOnce) {
    MockFunction mock;
    EXPECT_CALL(mock, Call())
        .Times(1);
    mock.Call();
}
```

### 匹配器（Matchers）

Google Mock 提供了丰富的匹配器，允许我们在期望调用中使用复杂的条件。

-   testing::_：匹配任何值。

-   testing::Eq(x)：匹配等于x的值。

-   testing::Le(x)：匹配小于x的值。

-   testing::Ge(x)：匹配大于或等于 x 的值。

```c++
TEST(MockMatchersTest, MatchesSpecificValue) {
    MockFunction mock;
    EXPECT_CALL(mock, Call(testing::Ge(10)))
        .Times(1);
    mock.Call(10);  // 匹配成功
}
```

### 组合断言

组合断言允许我们构建更复杂的验证逻辑。

-   testing::AllOf：所有条件都满足。

-   testing::AnyOf：任一条件满足。

```c++
TEST(CombinationAssertionsTest, ChecksMultipleConditions) {
    int result = someFunction();
    ASSERT_THAT(result, testing::AllOf(testing::Ge(40), testing::Le(50)));
}
```

### 断言动作（Actions）

断言动作是 Google Mock 中用于指定 Mock 对象在期望调用发生时应该执行的操作。

-   testing::Return(x)：返回值 x。

-   testing::Throw：抛出异常。

```c++
TEST(MockActionsTest, ReturnsSpecificValue) {
    MockFunction mock;
    EXPECT_CALL(mock, Call())
        .WillOnce(testing::Return(42));
    EXPECT_EQ(42, mock.Call());
}
```

### 验证器（Validators）

验证器允许我们在 Mock 对象的调用发生时执行自定义验证逻辑。

-   testing::SaveArg：保存调用参数。

-   testing::InvokeWithoutArgs：调用无参函数。

```c++
TEST(MockValidatorsTest, SavesArgument) {
    MockFunction mock;
    int savedArg;
    EXPECT_CALL(mock, Call(testing::_))
        .WillOnce(testing::SaveArg<0>(&savedArg));
    mock.Call(42);
    ASSERT_EQ(42, savedArg);
}
```

### 期望和断言的高级用法

高级用法包括设置期望的顺序、使用参数化期望等。

-   InSequence：确保调用按特定顺序发生。

-   testing::Each：匹配每个元素。

```c++
using ::testing::InSequence;
using ::testing::Return;

TEST(SequenceExpectationsTest, CallsInOrder) {
    InSequence seq;
    MockFunction mock;
    EXPECT_CALL(mock, Call(1))
        .WillOnce(Return(2));
    EXPECT_CALL(mock, Call(2))
        .WillOnce(Return(3));
    EXPECT_EQ(2, mock.Call(1));
    EXPECT_EQ(3, mock.Call(2));
}
```

## 整合

从以上案例来看。可以整合计算器和数据访问两个程序。

### 计算器

Calculator.cpp

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>


class Calculator {
public:
    virtual ~Calculator() {}
    virtual int add(int a, int b) = 0;
    virtual int subtract(int a, int b) = 0;
    virtual int multiply(int a, int b) = 0;
    virtual int divide(int a, int b) = 0;
};

class MockCalculator : public Calculator {
public:
    MOCK_METHOD(int, add, (int a, int b), (override));
    MOCK_METHOD(int, subtract, (int a, int b), (override));
    MOCK_METHOD(int, multiply, (int a, int b), (override));
    MOCK_METHOD(int, divide, (int a, int b), (override));
};

TEST(CalculatorTest, TestCalculator) {
    MockCalculator mockCalculator;

    // 设置mock对象的行为
    ON_CALL(mockCalculator, add(testing::_,testing::_))
        .WillByDefault(testing::Invoke([](int a, int b) { return a + b; }));
    ON_CALL(mockCalculator, subtract(testing::_,testing::_))
        .WillByDefault(testing::Invoke([](int a, int b) { return a - b; }));
    ON_CALL(mockCalculator, multiply(testing::_,testing::_))
        .WillByDefault(testing::Invoke([](int a, int b) { return a * b; }));
    ON_CALL(mockCalculator, divide(testing::_,testing::_))
        .WillByDefault(testing::Invoke([](int a, int b) { return a / b; }));

    // 验证mock对象的行为
    EXPECT_EQ(mockCalculator.add(2, 3), 5);
    EXPECT_EQ(mockCalculator.subtract(5, 3), 2);
    EXPECT_EQ(mockCalculator.multiply(5, 3), 15);
    EXPECT_EQ(mockCalculator.divide(6, 3), 2);
}

TEST(CalculatorTest, TestCalculatorDivisorZero) {
    MockCalculator mockCalculator;
    // 设置mock对象的行为
    EXPECT_CALL(mockCalculator, divide(testing::_,0))
        .WillOnce(testing::Throw(std::runtime_error("Divisor cannot be 0")));
// 验证mock对象的行为
//…
    EXPECT_THROW(mockCalculator.divide(6, 0), std::runtime_error);
}
```

### 数据库访问

Database.cpp

```c++
#include <gmock/gmock.h>

#include <string>
#include <unordered_map>
#include <memory>
#include <iostream>
#include <vector>


using ::testing::_;
using ::testing::Invoke;
using ::testing::Return;
using ::testing::UnorderedElementsAre;
using ::testing::WithArgs;
using ::std::string;
using ::std::vector;
using ::std::to_string;

// 定义一个结构体
struct QueryResult {
    string name;
    int age;
};

class Database {
public:
    virtual ~Database() {}
    virtual string query(const string& sql) const{return sql;}
    virtual string fetchData(){return "";}
    virtual string update(int id, const string& value) const{return value;}
    virtual QueryResult executeQuery(const string& sql) = 0;
    virtual void insert(const vector& values){}
    virtual void connect(){}
};


class MockDatabase:public Database {
public:
    MOCK_METHOD(string, query, (const string& sql), (const, override));
    MOCK_METHOD(string, fetchData, (), (override));
    MOCK_METHOD(string, update, (int id, const string& value), (const, override));
    MOCK_METHOD(QueryResult, executeQuery, (const string&), (override));
    MOCK_METHOD(void, insert, (const vector& values), (override));
    MOCK_METHOD(void, connect, (), (override));
};

class DataProcessor {
public:
    explicit DataProcessor(Database* db) : db_(db) {}
    string process(const string& data) {
        return data + " processed with " + db_->query("SELECT * FROM table");
    }

private:
    Database* db_;
};

TEST(DataBaseTest, ProcessesDataWithDatabaseQuery) {
    MockDatabase mockDb;
    DataProcessor processor(&mockDb);

    EXPECT_CALL(mockDb, query("SELECT * FROM table"))
        .WillOnce(Return("Mocked query result"));
    EXPECT_EQ("Data processed with Mocked query result", processor.process("Data"));
}

TEST(DataBaseTest, CombineMockAndStub) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, connect())
        .Times(1);
    ON_CALL(mockDb, query(_))
        .WillByDefault(Return("Stubbed Query Result"));

    mockDb.connect();
    EXPECT_EQ("Stubbed Query Result",mockDb.query("query"));
}

TEST(DataBaseTest, UseInvokeForDynamicReturn) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, query(_))
        .WillOnce(Invoke([](const string& sql) {
            return "Result for " + sql;
        }));
    EXPECT_EQ("Result for SELECT * FROM table", mockDb.query("SELECT * FROM table"));
}

TEST(DataBaseTest, CustomDefaultBehavior) {
    MockDatabase mockDb;
    ON_CALL(mockDb, query(_))
        .WillByDefault(Return("Stubbed Query Result"));

    EXPECT_EQ("Stubbed Query Result", mockDb.query("query"));
}

TEST(DataBaseTest, FetchDataTest) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, fetchData())
        .WillOnce(Return("Mocked Data"));
    string result = mockDb.fetchData();
    EXPECT_EQ("Mocked Data", result);
}

TEST(DataBaseTest, SimulateSideEffects) {
    MockDatabase mockDb;
    string sideEffect;
    EXPECT_CALL(mockDb, update(_, _))
        .WillOnce(Invoke([&sideEffect](int id, const string& value) {
            return sideEffect = "Updated " + to_string(id) + " to " + value;
        }));
    mockDb.update(1, "value");
    EXPECT_EQ("Updated 1 to value", sideEffect);
}

TEST(DataBaseTest, MultipleInvocations) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, fetchData())
        .Times(3)
        .WillRepeatedly(Return("Data"));

    for (int i = 0; i < 3; ++i) {
        EXPECT_EQ("Data", mockDb.fetchData());
    }
}

void SomeFunctionUsingMockDatabase(MockDatabase& db) {
    vector values = {3, 1, 2};

    db.insert(values);
}

TEST(DataBaseTest, MatchUnorderedContainer) {
    MockDatabase mockDb;
    EXPECT_CALL(mockDb, insert(UnorderedElementsAre(1, 2, 3)));
    SomeFunctionUsingMockDatabase(mockDb);
}

TEST(DataBaseTest, MockComplexReturnType) {
    MockDatabase mockDb;

    EXPECT_CALL(mockDb, executeQuery(_))
        .Times(2)
        .WillOnce(WithArgs<0>(Invoke([](const string& sql) {
            return QueryResult{"Alice", 30};
        })))
        .WillRepeatedly(WithArgs<0>(Invoke([](const string& sql) {
            return QueryResult{"Bob", 40};
        })));
    QueryResult result = mockDb.executeQuery("Alice");
    EXPECT_EQ("Alice",result.name);
    EXPECT_EQ(30,result.age);
    result = mockDb.executeQuery("Bob");
    EXPECT_EQ("Bob",result.name);
    EXPECT_EQ(40,result.age);
}
```

