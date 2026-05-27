# Gmock 教程

## 引言

### Google Mock 简介

Google Mock 是由 Google 开发的 C++ 模拟（mocking）框架，是 Google Test 测试框架的一部分。它允许开发者创建模拟对象来替代真实依赖项，使单元测试更加灵活和独立。

### 为什么选择 Google Mock

- **灵活性**：支持高度定制化的模拟行为，可模拟复杂依赖关系
- **易用性**：API 设计简洁直观，学习曲线平缓
- **社区支持**：Google 出品，社区活跃，文档丰富
- **集成性**：与 Google Test 无缝集成，提供一站式测试方案

## Google Mock 基础

### 安装和配置

Google Mock 已并入 Google Test，无需单独安装。通过 CMake 集成：

```cmake
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG v1.14.0
)
FetchContent_MakeAvailable(googletest)

target_link_libraries(your_test PRIVATE gmock gmock_main)
```

### 基本测试结构

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

class NetworkService {
public:
    virtual ~NetworkService() = default;
    virtual std::string fetchData() = 0;
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

> **关键概念**：`MOCK_METHOD` 宏用于在 Mock 类中声明模拟方法，语法为 `MOCK_METHOD(返回类型, 方法名, (参数列表), (修饰符))`。

## 创建 Mock 对象

### MOCK_METHOD 宏

Google Mock 使用 `MOCK_METHOD` 宏定义模拟方法：

```c++
class MockDatabase : public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
    MOCK_METHOD(void, connect, (), (override));
    MOCK_METHOD(int, getVersion, (), (const));
};
```

### 设置期望

使用 `EXPECT_CALL` 设置 Mock 对象的期望行为：

```c++
MockDatabase mockDb;
DataProcessor processor(&mockDb);

EXPECT_CALL(mockDb, query("SELECT * FROM table"))
    .WillOnce(testing::Return("Mocked query result"));

std::string result = processor.process("Data");
EXPECT_EQ("Data processed with Mocked query result", result);
```

### ON_CALL 默认行为

使用 `ON_CALL` 为 Mock 方法设置默认行为：

```c++
ON_CALL(mockDb, query(testing::_))
    .WillByDefault(testing::Return("Default Query Result"));

// 未设置 EXPECT_CALL 时，调用返回默认值
EXPECT_EQ("Default Query Result", mockDb.query("Any SQL"));
```

> `ON_CALL` 与 `EXPECT_CALL` 的区别：`ON_CALL` 设置默认行为，不验证调用次数；`EXPECT_CALL` 既设置行为，又验证调用是否符合期望。

## 验证调用顺序

### 严格顺序验证（InSequence）

要求方法按指定顺序调用：

```c++
#include <gmock/gmock.h>
using ::testing::_;
using ::testing::InSequence;

class MyClass {
public:
    virtual ~MyClass() = default;
    virtual bool firstMethod(int) = 0;
    virtual bool secondMethod(int) = 0;
};

class MockMyClass : public MyClass {
public:
    MOCK_METHOD(bool, firstMethod, (int), (override));
    MOCK_METHOD(bool, secondMethod, (int), (override));
};

TEST(SomeClassTest, CallsMethodsInOrder) {
    MockMyClass mock;
    {
        InSequence seq;  // 声明顺序约束
        EXPECT_CALL(mock, firstMethod(_))
            .WillOnce(testing::Return(true));
        EXPECT_CALL(mock, secondMethod(_))
            .WillOnce(testing::Return(true));
    }

    // 必须按 firstMethod -> secondMethod 的顺序调用
    EXPECT_EQ(true, mock.firstMethod(1));
    EXPECT_EQ(true, mock.secondMethod(2));
}
```

> ⚠️ **注意**：如果先调用 `secondMethod` 再调用 `firstMethod`，测试将失败。

### 部分顺序验证（Sequence + After）

使用 `Sequence` 和 `After` 实现更灵活的顺序控制：

```c++
using ::testing::Sequence;

Sequence s1, s2;
EXPECT_CALL(foo, A()).InSequence(s1, s2);
EXPECT_CALL(bar, B()).InSequence(s1);
EXPECT_CALL(bar, C()).InSequence(s2);
EXPECT_CALL(foo, D()).InSequence(s2);
```

执行顺序约束：
- `s1`: A → B（A 必须在 B 之前）
- `s2`: A → C → D（A 必须在 C 之前，C 必须在 D 之前）

## 调用次数控制

### Times 修饰符

```c++
EXPECT_CALL(mock_object, method_name(matchers...))
    .With(multi_argument_matcher)    // 最多使用一次
    .Times(cardinality)              // 最多使用一次
    .InSequence(sequences...)        // 可使用任意次数
    .After(expectations...)          // 可使用任意次数
    .WillOnce(action)                // 可使用任意次数
    .WillRepeatedly(action)          // 最多使用一次
    .RetiresOnSaturation();          // 最多使用一次
```

常用 `Times` 值：

| 用法 | 含义 |
|------|------|
| `.Times(1)` | 恰好调用 1 次 |
| `.Times(2)` | 恰好调用 2 次 |
| `.Times(AtLeast(1))` | 至少调用 1 次 |
| `.Times(AtMost(3))` | 最多调用 3 次 |
| `.Times(Between(1, 3))` | 调用 1 到 3 次 |
| `.Times(AnyNumber())` | 任意次数（默认） |

```c++
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

## Mock 与 Stub 组合

在同一个 Mock 对象中同时使用 Mock 和 Stub 行为：

```c++
using ::testing::_;
using ::testing::Invoke;
using ::testing::Return;

class Database {
public:
    virtual ~Database() = default;
    virtual std::string query(const std::string& sql) const = 0;
    virtual void connect() = 0;
};

class MockDatabase : public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
    MOCK_METHOD(void, connect, (), (override));
};

TEST(SomeClassTest, CombineMockAndStub) {
    MockDatabase mockDb;

    // Stub：设置默认行为，不验证调用次数
    ON_CALL(mockDb, query(_))
        .WillByDefault(Return("Stubbed Query Result"));

    // Mock：验证 connect() 被恰好调用 1 次
    EXPECT_CALL(mockDb, connect())
        .Times(1);

    mockDb.connect();
    EXPECT_EQ("Stubbed Query Result", mockDb.query("query"));
}
```

> **区分**：`EXPECT_CALL` 验证调用行为（Mock），`ON_CALL` 仅设置返回值（Stub）。

## Mock 类型包装器

### NiceMock

`NiceMock` 对未设置期望的调用不产生警告，返回默认值：

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::Return;

class MyInterface {
public:
    virtual ~MyInterface() = default;
    virtual int Foo() = 0;
    virtual void Bar(int x) = 0;
};

class MockMyInterface : public MyInterface {
public:
    MOCK_METHOD(int, Foo, (), (override));
    MOCK_METHOD(void, Bar, (int), (override));
};

TEST(NiceMockTest, NiceMockExample) {
    // 使用 NiceMock 包装 MockMyInterface
    ::testing::NiceMock<MockMyInterface> nice_mock;

    // 仅设置 Foo() 的期望
    EXPECT_CALL(nice_mock, Foo()).WillOnce(Return(42));

    EXPECT_EQ(nice_mock.Foo(), 42);

    // 调用未设置期望的 Bar()，NiceMock 不会报错
    nice_mock.Bar(10);
}
```

### StrictMock

`StrictMock` 对未设置期望的调用会直接导致测试失败：

```c++
TEST(StrictMockTest, StrictMockExample) {
    ::testing::StrictMock<MockMyInterface> strict_mock;

    // 必须为所有会被调用的方法设置期望
    EXPECT_CALL(strict_mock, Foo()).WillOnce(Return(42));
    EXPECT_CALL(strict_mock, Bar());

    EXPECT_EQ(strict_mock.Foo(), 42);
    strict_mock.Bar();

    // 若调用未设置期望的方法，测试将失败
    // strict_mock.SomeUndeclaredMethod(); // 这将导致测试失败
}
```

| 类型 | 未设置期望的调用 | 适用场景 |
|------|------------------|----------|
| `NiceMock` | 静默忽略，返回默认值 | 关注特定方法调用 |
| `StrictMock` | 测试失败 | 需要严格控制所有调用 |
| 默认 Mock | 产生警告 | 一般测试 |

## 高级 Mocking 技巧

### 模拟异常

使用 `Throw` 动作模拟方法抛出异常：

```c++
class Calculator {
public:
    virtual ~Calculator() = default;
    virtual int divide(int a, int b) = 0;
};

class MockCalculator : public Calculator {
public:
    MOCK_METHOD(int, divide, (int a, int b), (override));
};

TEST(CalculatorTest, TestDivideByZero) {
    MockCalculator mockCalculator;

    EXPECT_CALL(mockCalculator, divide(_, 0))
        .WillOnce(testing::Throw(std::runtime_error("Divisor cannot be 0")));

    EXPECT_THROW(mockCalculator.divide(6, 0), std::runtime_error);
}
```

### 使用 Invoke 回调

使用 `Invoke` 根据输入参数动态返回结果：

```c++
TEST(AdvancedMockingTest, UseInvokeForDynamicReturn) {
    MockDatabase mockDb;

    EXPECT_CALL(mockDb, query(_))
        .WillOnce(testing::Invoke([](const std::string& sql) {
            return "Result for " + sql;
        }));

    EXPECT_EQ("Result for SELECT * FROM table",
              mockDb.query("SELECT * FROM table"));
}
```

### 模拟副作用

使用 `Invoke` 捕获或修改外部状态：

```c++
TEST(MockDatabase, SimulateSideEffects) {
    MockDatabase mockDb;
    std::string sideEffect;

    EXPECT_CALL(mockDb, update(_, _))
        .WillOnce(Invoke([&sideEffect](int id, const std::string& value) {
            sideEffect = "Updated " + std::to_string(id) + " to " + value;
            return sideEffect;
        }));

    mockDb.update(1, "value");
    EXPECT_EQ("Updated 1 to value", sideEffect);
}
```

### 模拟复杂返回类型

使用 `WithArgs` 处理复杂数据结构：

```c++
struct QueryResult {
    std::string name;
    int age;
};

class DatabaseService {
public:
    virtual ~DatabaseService() = default;
    virtual QueryResult executeQuery(const std::string& sql) = 0;
};

class MockDatabaseService : public DatabaseService {
public:
    MOCK_METHOD(QueryResult, executeQuery, (const std::string&), (override));
};

TEST(MockDatabaseService, MockComplexReturnType) {
    MockDatabaseService mockDb;

    EXPECT_CALL(mockDb, executeQuery(_))
        .Times(2)
        .WillOnce(testing::WithArgs<0>(testing::Invoke(
            [](const std::string& sql) {
                return QueryResult{"Alice", 30};
            })))
        .WillRepeatedly(testing::WithArgs<0>(testing::Invoke(
            [](const std::string& sql) {
                return QueryResult{"Bob", 40};
            })));

    QueryResult result = mockDb.executeQuery("Alice");
    EXPECT_EQ("Alice", result.name);
    EXPECT_EQ(30, result.age);

    result = mockDb.executeQuery("Bob");
    EXPECT_EQ("Bob", result.name);
    EXPECT_EQ(40, result.age);
}
```

### 匹配容器元素

使用 `UnorderedElementsAre` 匹配容器内容（不关注顺序）：

```c++
class DataStore {
public:
    virtual ~DataStore() = default;
    virtual void insert(const std::vector<int>& values) = 0;
};

class MockDataStore : public DataStore {
public:
    MOCK_METHOD(void, insert, (const std::vector<int>& values), (override));
};

void SomeFunctionUsingMockDataStore(MockDataStore& db) {
    std::vector<int> values = {3, 1, 2};
    db.insert(values);
}

TEST(AdvancedMockingTest, MatchUnorderedContainer) {
    MockDataStore mockDb;
    EXPECT_CALL(mockDb, insert(testing::UnorderedElementsAre(1, 2, 3)));
    SomeFunctionUsingMockDataStore(mockDb);
}
```

## 断言和期望

### 基本断言

Google Test 提供的基本断言：

| 断言 | 说明 |
|------|------|
| `ASSERT_TRUE` | 条件为真，否则致命失败 |
| `EXPECT_TRUE` | 条件为真，否则非致命失败 |
| `ASSERT_EQ` | 验证相等，致命失败 |
| `EXPECT_EQ` | 验证相等，非致命失败 |

### 匹配器（Matchers）

| 匹配器 | 说明 |
|--------|------|
| `testing::_` | 匹配任何值 |
| `testing::Eq(x)` | 匹配等于 x 的值 |
| `testing::Le(x)` | 匹配小于等于 x 的值 |
| `testing::Ge(x)` | 匹配大于等于 x 的值 |

```c++
TEST(MockMatchersTest, MatchesSpecificValue) {
    MockFunction mock;
    EXPECT_CALL(mock, Call(testing::Ge(10)))
        .Times(1);
    mock.Call(10);  // 匹配成功
}
```

### 组合断言

```c++
TEST(CombinationAssertionsTest, ChecksMultipleConditions) {
    int result = someFunction();
    ASSERT_THAT(result, testing::AllOf(testing::Ge(40), testing::Le(50)));
}
```

### 断言动作（Actions）

| 动作 | 说明 |
|------|------|
| `testing::Return(x)` | 返回值 x |
| `testing::Throw(exception)` | 抛出异常 |
| `testing::Invoke(fn)` | 调用回调函数 |
| `testing::SaveArg<N>(&var)` | 保存第 N 个参数 |

## 完整示例

### 计算器 Mock

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

class Calculator {
public:
    virtual ~Calculator() = default;
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

TEST(CalculatorTest, TestBasicOperations) {
    MockCalculator mockCalculator;

    ON_CALL(mockCalculator, add(_, _))
        .WillByDefault(testing::Invoke([](int a, int b) { return a + b; }));
    ON_CALL(mockCalculator, subtract(_, _))
        .WillByDefault(testing::Invoke([](int a, int b) { return a - b; }));
    ON_CALL(mockCalculator, multiply(_, _))
        .WillByDefault(testing::Invoke([](int a, int b) { return a * b; }));
    ON_CALL(mockCalculator, divide(_, _))
        .WillByDefault(testing::Invoke([](int a, int b) { return a / b; }));

    EXPECT_EQ(mockCalculator.add(2, 3), 5);
    EXPECT_EQ(mockCalculator.subtract(5, 3), 2);
    EXPECT_EQ(mockCalculator.multiply(5, 3), 15);
    EXPECT_EQ(mockCalculator.divide(6, 3), 2);
}

TEST(CalculatorTest, TestDivideByZero) {
    MockCalculator mockCalculator;

    EXPECT_CALL(mockCalculator, divide(_, 0))
        .WillOnce(testing::Throw(std::runtime_error("Divisor cannot be 0")));

    EXPECT_THROW(mockCalculator.divide(6, 0), std::runtime_error);
}
```

### 数据库访问 Mock

```c++
#include <gmock/gmock.h>
#include <string>
#include <vector>
#include <unordered_map>

using ::testing::_;
using ::testing::Invoke;
using ::testing::Return;
using ::testing::UnorderedElementsAre;
using ::testing::WithArgs;

struct QueryResult {
    std::string name;
    int age;
};

class Database {
public:
    virtual ~Database() = default;
    virtual std::string query(const std::string& sql) const = 0;
    virtual std::string fetchData() = 0;
    virtual std::string update(int id, const std::string& value) const = 0;
    virtual QueryResult executeQuery(const std::string& sql) = 0;
    virtual void insert(const std::vector<int>& values) = 0;
    virtual void connect() = 0;
};

class MockDatabase : public Database {
public:
    MOCK_METHOD(std::string, query, (const std::string& sql), (const, override));
    MOCK_METHOD(std::string, fetchData, (), (override));
    MOCK_METHOD(std::string, update, (int id, const std::string& value), (const, override));
    MOCK_METHOD(QueryResult, executeQuery, (const std::string&), (override));
    MOCK_METHOD(void, insert, (const std::vector<int>& values), (override));
    MOCK_METHOD(void, connect, (), (override));
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

TEST(DataBaseTest, ProcessesDataWithDatabaseQuery) {
    MockDatabase mockDb;
    DataProcessor processor(&mockDb);

    EXPECT_CALL(mockDb, query("SELECT * FROM table"))
        .WillOnce(Return("Mocked query result"));

    EXPECT_EQ("Data processed with Mocked query result", processor.process("Data"));
}

TEST(DataBaseTest, CombineMockAndStub) {
    MockDatabase mockDb;

    EXPECT_CALL(mockDb, connect()).Times(1);
    ON_CALL(mockDb, query(_))
        .WillByDefault(Return("Stubbed Query Result"));

    mockDb.connect();
    EXPECT_EQ("Stubbed Query Result", mockDb.query("query"));
}

TEST(DataBaseTest, UseInvokeForDynamicReturn) {
    MockDatabase mockDb;

    EXPECT_CALL(mockDb, query(_))
        .WillOnce(Invoke([](const std::string& sql) {
            return "Result for " + sql;
        }));

    EXPECT_EQ("Result for SELECT * FROM table",
              mockDb.query("SELECT * FROM table"));
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

    std::string result = mockDb.fetchData();
    EXPECT_EQ("Mocked Data", result);
}

TEST(DataBaseTest, SimulateSideEffects) {
    MockDatabase mockDb;
    std::string sideEffect;

    EXPECT_CALL(mockDb, update(_, _))
        .WillOnce(Invoke([&sideEffect](int id, const std::string& value) {
            return sideEffect = "Updated " + std::to_string(id) + " to " + value;
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
    std::vector<int> values = {3, 1, 2};
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
        .WillOnce(WithArgs<0>(Invoke([](const std::string& sql) {
            return QueryResult{"Alice", 30};
        })))
        .WillRepeatedly(WithArgs<0>(Invoke([](const std::string& sql) {
            return QueryResult{"Bob", 40};
        })));

    QueryResult result = mockDb.executeQuery("Alice");
    EXPECT_EQ("Alice", result.name);
    EXPECT_EQ(30, result.age);

    result = mockDb.executeQuery("Bob");
    EXPECT_EQ("Bob", result.name);
    EXPECT_EQ(40, result.age);
}
```
