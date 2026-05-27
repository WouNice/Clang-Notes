# GTest 教程

## 什么是 GTest

GoogleTest 是由 Google 开发的 C++ 测试框架，支持 Linux、Windows 和 macOS 操作系统，使用 Bazel 或 CMake 构建工具。

- **项目主页**：<https://github.com/google/googletest>
- **官方文档**：<https://google.github.io/googletest/>

## 源码安装

```bash
git clone https://github.com/google/googletest.git --depth 1
cd googletest
mkdir build && cd build
cmake ..    # 生成 Makefile
make        # 编译
```

> **Rocky Linux 注意事项**：若执行 `cmake ..` 报错 `GOOGLETEST_VERSION` 未定义，在 `CMakeLists.txt` 中添加：
> ```cmake
> set(GOOGLETEST_VERSION 1.17.0)
> ```
> 版本号与下载版本保持一致。

### CMake 集成示例

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 指定 gtest 路径（已安装则用 find_package，否则直接指定路径）
set(GTEST_DIR /path/to/googletest)
include_directories(${GTEST_DIR}/googletest/include)
link_directories(${GTEST_DIR}/googletest/build)

add_executable(test_example test_example.cpp)
target_link_libraries(test_example gtest gtest_main)
```

## 核心概念

| 术语 | 说明 |
|------|------|
| **断言** (assertion) | 检查条件是否为真的语句，是测试的基本组成部分 |
| **测试** (test) | 也叫测试用例 (test case)，使用断言验证被测试代码的行为 |
| **测试套件** (test suite) | 包含一个或多个测试用例，用于组织测试结构 |
| **测试夹具** (test fixture) | 当多个测试需要共用对象或子进程时使用的类 |
| **测试程序** (test program) | 包含多个测试套件的可执行程序 |

断言结果分类：
- **成功** (success)：条件为真
- **非致命失败** (nonfatal failure)：`EXPECT_*` 断言失败，测试继续执行
- **致命失败** (fatal failure)：`ASSERT_*` 断言失败，测试立即终止

## 快速入门

### 创建项目

项目结构：
```
MyProject/
├── CMakeLists.txt
└── hello_test.cpp
```

`CMakeLists.txt`：
```cmake
cmake_minimum_required(VERSION 3.26.5)
project(MyProject)

set(GTEST_DIR ../googletest/)
include_directories(${GTEST_DIR}/googletest/include/gtest)
link_directories(${GTEST_DIR}/googletest/build)

aux_source_directory(. SRC_LIST)
add_executable(test_example ${SRC_LIST})
target_link_libraries(test_example gtest gtest_main)
```

`hello_test.cpp`：
```c++
#include <gtest/gtest.h>

TEST(HelloTest, BasicAssertions) {
    EXPECT_STRNE("hello", "world");
    EXPECT_EQ(7 * 6, 42);
}
```

### 构建与运行

```bash
cmake -S . -B build
cmake --build build
./build/test_example
```

输出示例：
```
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from HelloTest
[ RUN      ] HelloTest.BasicAssertions
[       OK ] HelloTest.BasicAssertions (0 ms)
[----------] 1 test from HelloTest (0 ms total)
[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.
```

## 编写测试

### TEST 宏

```c++
TEST(TestSuiteName, TestName) {
    // test body
}
```

- `TestSuiteName` 和 `TestName` 必须是合法的 C++ 标识符
- **不应包含下划线**（避免与 GoogleTest 内部命名冲突）

> `TEST()` 宏实际上定义了一个名为 `TestSuiteName_TestName_Test` 的类，继承自 `::testing::Test`，测试体即 `TestBody()` 成员函数。

### 参数化测试

使用 `TestWithParam` 实现参数化测试：

```c++
#include <gtest/gtest.h>
#include <tuple>

class Adder {
public:
    int add(int a, int b) { return a + b; }
};

class AdderTest : public ::testing::TestWithParam<std::tuple<int, int, int>> {};

TEST_P(AdderTest, AddReturnsExpectedResult) {
    Adder adder;
    int a, b, expected;
    std::tie(a, b, expected) = GetParam();
    EXPECT_EQ(expected, adder.add(a, b));
}

INSTANTIATE_TEST_SUITE_P(AdderTests, AdderTest,
    ::testing::Values(
        std::make_tuple(2, 3, 5),
        std::make_tuple(-2, 1, -1),
        std::make_tuple(0, 0, 0)
    ));
```

> **注意**：旧版本使用 `INSTANTIATE_TEST_CASE_P`，新版本（1.10+）推荐使用 `INSTANTIATE_TEST_SUITE_P`。

## 断言系统

GoogleTest 提供丰富的断言宏，定义在 `<gtest/gtest.h>` 中。

### 常用断言

| 断言 | 验证条件 |
|------|----------|
| `EXPECT_TRUE(condition)` | `condition` 为真 |
| `EXPECT_FALSE(condition)` | `condition` 为假 |
| `EXPECT_EQ(val1, val2)` | `val1 == val2` |
| `EXPECT_NE(val1, val2)` | `val1 != val2` |
| `EXPECT_LT(val1, val2)` | `val1 < val2` |
| `EXPECT_LE(val1, val2)` | `val1 <= val2` |
| `EXPECT_GT(val1, val2)` | `val1 > val2` |
| `EXPECT_GE(val1, val2)` | `val1 >= val2` |
| `EXPECT_STREQ(str1, str2)` | C 字符串相等 |
| `EXPECT_STRNE(str1, str2)` | C 字符串不相等 |
| `EXPECT_STRCASEEQ(str1, str2)` | C 字符串相等（忽略大小写） |
| `EXPECT_STRCASENE(str1, str2)` | C 字符串不相等（忽略大小写） |
| `EXPECT_FLOAT_EQ(val1, val2)` | 两个 `float` 近似相等 |
| `EXPECT_DOUBLE_EQ(val1, val2)` | 两个 `double` 近似相等 |
| `EXPECT_NEAR(val1, val2, abs_error)` | 差值不超过 `abs_error` |
| `EXPECT_THROW(statement, exception_type)` | 抛出指定异常 |
| `EXPECT_ANY_THROW(statement)` | 抛出任何异常 |
| `EXPECT_NO_THROW(statement)` | 不抛出异常 |
| `EXPECT_THAT(val, matcher)` | 满足匹配器条件 |

> 每个 `EXPECT_*` 都有对应的 `ASSERT_*` 版本。`ASSERT_*` 失败时立即终止当前测试，`EXPECT_*` 失败时继续执行。

完整参考：[Assertions Reference](https://google.github.io/googletest/reference/assertions.html)

### 自定义失败信息

断言宏返回 `ostream` 对象，可使用 `<<` 输出自定义信息：

```c++
EXPECT_TRUE(my_condition) << "My condition is not true";
```

### 浮点数比较

```c++
ASSERT_FLOAT_EQ(0.1f, 0.1f);          // 精确比较
ASSERT_DOUBLE_EQ(0.1, 0.1);           // 双精度精确
ASSERT_NEAR(3.14159, M_PI, 0.0001);   // 允许误差范围
```

### 字符串比较

```c++
ASSERT_STREQ("hello", "hello");       // C 字符串相等
ASSERT_STRNE("A", "B");               // C 字符串不等
ASSERT_STRCASEEQ("HELLO", "hello");   // 忽略大小写
```

### 异常检测

```c++
ASSERT_THROW(
    throw std::runtime_error("error"),
    std::runtime_error
);

ASSERT_ANY_THROW(throw 1);
ASSERT_NO_THROW(int x = 5);
```

## 测试夹具（Test Fixture）

当多个测试需要共享相同的配置或数据时，使用测试夹具：

```c++
class DatabaseTest : public ::testing::Test {
protected:
    void SetUp() override {
        db.connect("test_db");
        db.createTable("Users");
    }

    void TearDown() override {
        db.dropTable("Users");
        db.disconnect();
    }

    void insertUser(const std::string& name, int age) {
        db.execute("INSERT INTO Users VALUES ('" + name + "', " + std::to_string(age) + ")");
    }

    Database db;
};

TEST_F(DatabaseTest, InsertRecord) {
    insertUser("Alice", 30);
    ASSERT_EQ(db.rowCount("Users"), 1);
}

TEST_F(DatabaseTest, DeleteRecord) {
    insertUser("Bob", 25);
    db.execute("DELETE FROM Users");
    ASSERT_EQ(db.rowCount("Users"), 0);
}
```

> `TEST_F` 宏用于使用夹具的测试，第一个参数必须是夹具类名。

## 事件机制

事件机制提供在特定时机执行自定义代码的能力。

### 三种事件类型

| 事件类型 | 继承类 | 触发时机 |
|----------|--------|----------|
| **测试套件事件** | `testing::Test` | `SetUpTestCase`：第一个测试用例前；`TearDownTestCase`：最后一个测试用例后 |
| **测试用例事件** | `testing::Test` | `SetUp`：每个测试用例前；`TearDown`：每个测试用例后 |
| **全局事件** | `testing::Environment` | `SetUp`：所有案例执行前；`TearDown`：所有案例执行后 |

```c++
// 全局事件示例
class GlobalEnvironment : public ::testing::Environment {
public:
    void SetUp() override {
        // 所有测试开始前执行
        std::cout << "Global SetUp" << std::endl;
    }

    void TearDown() override {
        // 所有测试结束后执行
        std::cout << "Global TearDown" << std::endl;
    }
};

// 注册全局事件
int main(int argc, char** argv) {
    ::testing::InitGoogleTest(&argc, argv);
    ::testing::AddGlobalTestEnvironment(new GlobalEnvironment);
    return RUN_ALL_TESTS();
}
```

## 死亡测试

死亡测试用于验证程序是否按预期方式崩溃。

### 死亡测试宏

| 宏 | 说明 |
|----|------|
| `ASSERT_DEATH(statement, matcher)` | 程序崩溃且错误信息匹配 `matcher` |
| `ASSERT_EXIT(statement, predicate, matcher)` | 程序退出且满足 `predicate`，错误信息匹配 `matcher` |
| `ASSERT_DEBUG_DEATH(statement, matcher)` | 调试模式下检查死亡 |

```c++
TEST(DeathTest, InvalidPointer) {
    int* ptr = nullptr;
    ASSERT_DEATH(*ptr = 5, "Segmentation fault");  // 预期段错误
}

TEST(DeathTest, AssertFailure) {
    ASSERT_DEBUG_DEATH(assert(false), "Assertion failed");  // 调试模式
}

TEST(DeathTest, ExitCode) {
    ASSERT_EXIT(exit(1), testing::ExitedWithCode(1), "");  // 退出码检查
}
```

> `ASSERT_DEATH` 的第二个参数是正则表达式，用于匹配错误信息。若传入 `""`，则只检查程序是否崩溃，不检查错误信息。

## CMake 高级集成

### 自动下载与集成

```cmake
cmake_minimum_required(VERSION 3.14)
project(GtestAdvancedExample)

# 自动下载 gtest
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.12.1
)
FetchContent_MakeAvailable(googletest)

# 主程序
add_executable(main_app src/main.cpp)

# 测试程序
add_executable(tests
  test/database_test.cpp
  test/user_test.cpp
)

target_link_libraries(tests
  PRIVATE gtest_main
  PRIVATE gmock_main
)

# 自动发现测试
include(GoogleTest)
gtest_discover_tests(tests
  EXTRA_ARGS --gtest_output=xml:${CMAKE_BINARY_DIR}/test_results.xml
)

# 添加覆盖率支持
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  target_compile_options(tests PRIVATE --coverage -fprofile-arcs -ftest-coverage)
  target_link_libraries(tests PRIVATE --coverage)
endif()
```

## 高级执行控制

### 测试筛选

```bash
# 运行特定测试套件
./tests --gtest_filter=DatabaseTest.*

# 排除特定测试
./tests --gtest_filter=-*.DeathTest

# 正则表达式匹配
./tests --gtest_filter=*Validation*.*Invalid*
```

### 测试重复与随机化

```bash
# 重复执行 100 次
./tests --gtest_repeat=100

# 随机执行顺序
./tests --gtest_shuffle --gtest_random_seed=42

# 遇到失败时停止
./tests --gtest_break_on_failure
```

### 常用命令行选项

| 选项 | 说明 |
|------|------|
| `--gtest_list_tests` | 列出所有测试 |
| `--gtest_filter=pattern` | 筛选测试 |
| `--gtest_repeat=n` | 重复执行 n 次 |
| `--gtest_shuffle` | 随机打乱执行顺序 |
| `--gtest_break_on_failure` | 遇到失败立即停止 |
| `--gtest_output=xml:path` | 输出 XML 格式报告 |
| `--gtest_color=yes` | 彩色输出 |

## 最佳实践

### 测试命名规范

- **测试套件**：被测试类名（如 `DatabaseTest`）
- **测试用例**：行为描述（如 `InsertValidRecord_Success`）

### 项目组织

```
project/
├── src/
└── test/
    ├── unit/
    │   ├── database_test.cpp
    │   └── user_test.cpp
    ├── integration/
    └── performance/
```

### 测试原则

1. **单一职责**：每个测试验证单一行为
2. **独立性**：避免测试间依赖
3. **复用夹具**：使用 `TEST_F` 减少重复代码
4. **优先 EXPECT**：优先使用 `EXPECT_*`，除非后续操作在失败时无意义

### 覆盖率生成

```bash
# 使用 gcovr 生成覆盖率报告
gcovr -r . --exclude test/ --html-details coverage.html
```

## 与 GMock 的关系

Google Mock（gmock）已并入 Google Test 项目，二者关系如下：

| 特性 | GTest | GMock |
|------|-------|-------|
| 核心功能 | 单元测试框架 | 模拟对象框架 |
| 主要宏 | `TEST`, `ASSERT_*`, `EXPECT_*` | `MOCK_METHOD`, `EXPECT_CALL`, `ON_CALL` |
| 依赖关系 | 独立使用 | 依赖 GTest |
| 头文件 | `<gtest/gtest.h>` | `<gmock/gmock.h>` |

实际项目中通常同时使用二者：

```c++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

// 使用 GMock 创建模拟对象
// 使用 GTest 编写测试和断言
```
