# GTest教程

## 什么是GTest

GoogleTest是由Google开发的一个C++测试框架，支持Linux、Windows和macOS操作系统，使用Bazel或CMake构建工具。

-   项目主页：https://github.com/google/googletest
-   官方文档：https://google.github.io/googletest/

## 源码安装

```sh
git clone https://github.com/google/googletest.git --depth 1
cd googletest
mkdir build
cd build
cmake ..                         # 生成 Makefile
make                             # 编译
```

>   Rocky上执行`cmake ..`遇到报错：
>
>   ```sh
>   [root@localhost build]# cmake ..
>   CMake Warning at CMakeLists.txt:50 (project):
>     VERSION keyword not followed by a value or was followed by a value that
>     expanded to nothing.
>
>
>   CMake Error at CMakeLists.txt:124 (set_target_properties):
>     set_target_properties called with incorrect number of arguments.
>
>
>   CMake Error at CMakeLists.txt:139 (set_target_properties):
>     set_target_properties called with incorrect number of arguments.
>
>
>   -- Configuring incomplete, errors occurred!
>   ```
>
>   原因是GOOGLETEST_VERSION找不到，在CmakeLists.txt设置一个属性即可
>
>   ```
>   set(GOOGLETEST_VERSION 1.17.0)
>   ```
>
>   版本号可以设置成下载的版本

编译后可以在Cmake中设置，以下是示例：

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 指定 gtest 的路径（如果已经安装）或构建目录路径
set(GTEST_DIR /path/to/googletest)  # 如果已经安装，则使用 CMake 的 find_package() 查找 gtest。否则，直接指定路径。
include_directories(${GTEST_DIR}/googletest/include ${GTEST_DIR}/googlemock/include)  # 包含 gtest 和 gmock 的头文件路径。
link_directories(${GTEST_DIR}/googletest/build ${GTEST_DIR}/googlemock/build)  # 链接库的路径，通常是构建目录。

add_executable(test_example test_example.cpp)  # 添加测试程序。
target_link_libraries(test_example gtest gtest_main)  # 链接 gtest 和 gmock 库。注意这里的库名可能需要根据实际编译出的库名调整。通常在编译完成后，可以在构建目录下查看生成的库文件。
```

## 基本概念

**断言**(assertion)：检查一个条件是否为真的语句，是测试的基本组成部分。断言的结果可以是**成功**(success)、**非致命失败**(nonfatal failure)或**致命失败**(fatal failure)。如果发生了致命失败，测试将立即终止，否则继续运行。

**测试**(test)：也叫**测试用例**(test case)，使用断言来验证被测试代码的行为。如果发生崩溃(coredump)或断言失败，则测试失败，否则成功。

**测试套件**(test suite)：包含一个或多个测试用例，用于组织测试用例以反映被测试代码的结构。当一个测试套件中的多个测试需要共用对象或子进程时，可以将其放入一个**测试套件**(test fixture)类。

**测试程序**(test program)：包含多个测试套件的可执行程序。

## 快速入门

下面介绍如何使用CMake运行GTest：

### 创建项目

首先创建项目根目录MyProject，之后在其中创建一个名为CMakeLists.txt的文件，内容如下：

```cmake
cmake_minimum_required(VERSION 3.26.5)
project(MyProject)

# 指定 gtest 的路径（如果已经安装）或构建目录路径
set(GTEST_DIR ../googletest/)  # 如果已经安装，则使用 CMake 的 find_package() 查找 gtest。否则，直接指定路径。
include_directories(${GTEST_DIR}/googletest/include/gtest)  # 包含 gtest 的头文件路径。
link_directories(${GTEST_DIR}/googletest/build)  # 链接库的路径，通常是构建目录。

aux_source_directory(. SRC_LIST)
add_executable(test_example ${SRC_LIST}) # 添加测试程序。

target_link_libraries(test_example gtest gtest_main)  # 链接 gtest 和 gmock 库。
```

以上配置声明了对GoogleTest的依赖。

### 编写测试

创建一个名为hello_test.cpp的源文件，内容如下：

```c++
#include <gtest/gtest.h>
TEST(HelloTest, BasicAssertions) {
    EXPECT_STRNE("hello", "world");
    EXPECT_EQ(7 * 6, 42);
}
```

该文件使用`TEST()`宏定义了测试套件`HelloTest`中的一个测试用例`BasicAssertions`，包括两个断言。

### 运行测试

最后构建并运行测试，在项目根目录下执行以下命令：

```sh
$ cmake -S . -B build
-- The C compiler identification is GNU 11.5.0
-- The CXX compiler identification is GNU 11.5.0
...
-- Build files have been written to: .../build

$ cmake --build build
...
[100%] Built target test_example

$ ./build/test_example
Running main() from /builddir/build/BUILD/googletest-release-1.11.0/googletest/src/gtest_main.cc
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

其中第一行命令用于配置构建系统，解析CMakeLists.txt并生成构建文件，第二行命令用于执行编译链接操作，生成构建目标。第三行命令用于执行测试程序并报告测试结果。

## 简单测试

`TEST()`宏用于定义一个测试，语法如下：

```c++
TEST(TestSuiteName, TestName) {
    test body
}
```

其中第一个参数是测试套件名称，第二个参数是测试用例名称，二者都必须是合法的C++标识符，并且不应该包含下划线。

测试体可以包含断言和任何C++语句。如果任何断言失败或者崩溃，则整个测试失败，否则成功。

注：`TEST()`宏实际上定义了一个名为`TestSuiteName_TestName_Test`的类，该类继承了`::testing::Test`类并覆盖了成员函数`TestBody()`，测试体就是其函数体。其（简化的）定义如下：

```c++
#define TEST(TestSuiteName, TestName) \
class TestSuiteName##_##TestName##_Test : public ::testing::Test { \
private: \
    void TestBody() override; \
}; \
void TestSuiteName##_##TestName##_Test::TestBody()
```

## 断言系统

GoogleTest断言是类似于函数调用的宏，用于测试类或函数的行为。当断言失败时，GoogleTest将打印断言所在的源文件、行数以及错误信息。

每个断言都有两种版本：`ASSERT_*`版本的失败是致命失败，`EXPECT_*`版本的失败是非致命失败。

GoogleTest提供了一组断言，用于检查布尔值、使用比较运算符比较两个值、比较字符串以及浮点数等。所有断言都定义在头文件<gtest/gtest.h>中。

常用断言如下（每个断言都有对应的`ASSERT_*`版本，这里省略）：

| 断言                                      | 验证条件                                    |
| ----------------------------------------- | ------------------------------------------- |
| `EXPECT_TRUE(condition)`                  | `condition`为真                             |
| `EXPECT_FALSE(condition)`                 | `condition`为假                             |
| `EXPECT_EQ(val1, val2)`                   | `val1 == val2`                              |
| `EXPECT_NE(val1, val2)`                   | `val1 != val2`                              |
| `EXPECT_LT(val1, val2)`                   | `val1 < val2`                               |
| `EXPECT_LE(val1, val2)`                   | `val1 <= val2`                              |
| `EXPECT_GT(val1, val2)`                   | `val1 > val2`                               |
| `EXPECT_GE(val1, val2)`                   | `val1 >= val2`                              |
| `EXPECT_STREQ(str1, str2)`                | C字符串`str1`和`str2`相等                   |
| `EXPECT_STRNE(str1, str2)`                | C字符串`str1`和`str2`不相等                 |
| `EXPECT_STRCASEEQ(str1, str2)`            | C字符串`str1`和`str2`相等，忽略大小写       |
| `EXPECT_STRCASENE(str1, str2)`            | C字符串`str1`和`str2`不相等，忽略大小写     |
| `EXPECT_FLOAT_EQ(val1, val2)`             | 两个`float`值`val1`和`val2`近似相等         |
| `EXPECT_DOUBLE_EQ(val1, val2)`            | 两个`double`值`val1`和`val2`近似相等        |
| `EXPECT_NEAR(val1, val2, abs_error)`      | `val1`和`val2`之差的绝对值不超过`abs_error` |
| `EXPECT_THROW(statement, exception_type)` | `statement`抛出`exception_type`类型的异常   |
| `EXPECT_ANY_THROW(statement)`             | `statement`抛出任何类型的异常               |
| `EXPECT_NO_THROW(statement)`              | `statement`不抛出任何异常                   |
| `EXPECT_THAT(val, matcher)`               | `val`满足匹配器`matcher`                    |

完整参考列表：[Assertions Reference](https://google.github.io/googletest/reference/assertions.html)

断言宏返回一个`ostream`对象，可以使用`<<`运算符输出自定义的失败信息。例如：

```
EXPECT_TRUE(my_condition) << "My condition is not true";
```

### 二进制比较

```c++
ASSERT_EQ(5, 2+3);     // 相等
ASSERT_NE(0, 1);       // 不等
ASSERT_LT(3, 5);       // 小于
ASSERT_LE(4, 4);       // 小于等于
ASSERT_GT(10, 5);      // 大于
ASSERT_GE(7, 7);       // 大于等于
```

### 浮点数比较

```c++
ASSERT_FLOAT_EQ(0.1f, 0.1f);          // 精确比较
ASSERT_DOUBLE_EQ(0.1, 0.1);           // 双精度精确
ASSERT_NEAR(3.14159, M_PI, 0.0001);   // 允许误差
```

### 字符串比较

```c
ASSERT_STREQ("hello", "hello");       // C字符串相等
ASSERT_STRNE("A", "B");               // C字符串不等
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

## 事件机制

“事件” 本质是框架给你提供了一个机会, 让你能在这样的几个机会来执行你自己定制的代码, 来给测试用例准备/清理数据。gtest提供了多种事件机制，总结一下gtest的事件一共有三种：

1、TestSuite事件

需要写一个类，继承testing::Test，然后实现两个静态方法：SetUpTestCase 方法在第一个TestCase之前执行；TearDownTestCase方法在最后一个TestCase之后执行。

2、TestCase事件

是挂在每个案例执行前后的，需要实现的是SetUp方法和TearDown方法。SetUp方法在每个TestCase之前执行；TearDown方法在每个TestCase之后执行。

3、全局事件

要实现全局事件，必须写一个类，继承testing::Environment类，实现里面的SetUp和TearDown方法。SetUp方法在所有案例执行前执行；TearDown方法在所有案例执行后执行。

## 测试套件

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

## 死亡测试

这里的”死亡”指的是程序的崩溃。通常在测试的过程中，我们需要考虑各种各样的输入，有的输入可能直接导致程序崩溃，这个时候我们就要检查程序是否按照预期的方式挂掉，这也就是所谓的”死亡测试”。

死亡测试所用到的宏：

1.  ASSERT_DEATH(参数1，参数2)，程序挂了并且错误信息和参数2匹配，此时认为测试通过。如果参数2为空字符串，则只需要看程序挂没挂即可。
2.  ASSERT_EXIT(参数1，参数2，参数3)，语句停止并且错误信息和被提前给的信息匹配。

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

## CMake 高级集成

```c++
cmake_minimum_required(VERSION 3.14)
project(GtestAdvancedExample)

# 自动下载gtest
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

```c++
# 运行特定测试套件
./tests --gtest_filter=DatabaseTest.*

# 排除特定测试
./tests --gtest_filter=-*.DeathTest

# 正则表达式匹配
./tests --gtest_filter=*Validation*.*Invalid*
```

### 测试重复与随机化

```c++
# 重复执行100次
./tests --gtest_repeat=100

# 随机执行顺序
./tests --gtest_shuffle --gtest_random_seed=42

# 遇到失败时停止
./tests --gtest_break_on_failure
```

## 最佳实践

测试命名规范

-   测试套件：被测试类名（如 DatabaseTest）
-   测试用例：行为描述（如 InsertValidRecord_Success）

测试组织

```c++
project/
├── src/
└── test/
    ├── unit/
    │   ├── database_test.cpp
    │   └── user_test.cpp
    ├── integration/
    └── performance/
```

测试原则

-   每个测试验证单一行为
-   避免测试间依赖
-   使用套件减少重复代码
-   优先使用 `EXPECT_` 而非 `ASSERT_` 除非后续操作无效

测试覆盖率

```c++
# 生成覆盖率报告
gcovr -r . --exclude test/ --html-details coverage.html
```

