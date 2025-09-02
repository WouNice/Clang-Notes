# 执行shell命令

执行shell命令主要涉及三个指令：

-   execute_process
-   add_custom_command
-   add_custom_target

## execute_process

基本语法：

```
execute_process(COMMAND <cmd1> [<arguments>]
    [COMMAND <cmd2> [<arguments>]]...
    [WORKING_DIRECTORY <directory>]
    [TIMEOUT <seconds>]
    [RESULT_VARIABLE <variable>]
    [RESULTS_VARIABLE <variable>]
    [OUTPUT_VARIABLE <variable>]
    [ERROR_VARIABLE <variable>]
    [INPUT_FILE <file>]
    [OUTPUT_FILE <file>]
    [ERROR_FILE <file>]
    [OUTPUT_QUIET]
    [ERROR_QUIET]
    [COMMAND_ECHO <where>]
    [OUTPUT_STRIP_TRAILING_WHITESPACE]
    [ERROR_STRIP_TRAILING_WHITESPACE]
    [ENCODING <name>]
    [ECHO_OUTPUT_VARIABLE]
    [ECHO_ERROR_VARIABLE]
    [COMMAND_ERROR_IS_FATAL <ANY|LAST>])
```

参数含义：

-   `COMMAND`：待执行的命令，可以指定多个。
-   `WORKING_DIRECTORY`：将指定的目录作为命令执行的当前工作目录。
-   `TIMEOUT`：设置命令执行的超时时间,单位是秒，可以不是整数。过了设置的超时时间，所有命令会被终止，RESULT_VARIABLE会被设置为“timeout”
-   `RESULT_VARIABLE`：包含命令的执行结果，该变量会被设置为最后一个命令执行的返回码，或执行出错时候的描述字符串。
-   `RESULTS_VARIABLE`：3.10版本引入。将所有的命令执行结果存入该变量中，命令的执行结果以分号连接，存储的顺序按照COMMAND命令的顺序。
-   `OUTPUT_VARIABLE`：对应于标准输出的内容，存储最后一个命令的运行结果。
-   `ERROR_VARIABLE`：对应于标准错误的内容，存储所有命令错误。
-   `INPUT_FILE`：指定文件作为第一个命令的标准输入。
-   `OUTPUT_FILE`：指定文件作为最后一个命令的标准输出。
-   `ERROR_FILE`：所有命令运行错误结果存储的文件
-   `OUTPUT_QUIET/ERROR_QUIET`：忽略标准输出和标准错误。
-   `COMMAND_ECHO`：重显命令到指定的标准设备，如STDERR、STDOUT、NONE。使用CMAKE_EXECUTE_PROCESS_COMMAND_ECHO变量来修改它的行为
-   `OUTPUT_STRIP_TRAILING_WHITESPACE/ERROR_STRIP_TRAILING_WHITESPACE`：删除空白字符
-   `ENCODING`：在windows系统上指定命令输出时的解码方式，默认是utf-8，其他平台会忽略该参数
-   `ECHO_OUTPUT_VARIABLE/ECHO_ERROR_VARIABLE`：输出将被复制，它将被发送到配置的变量中，也会在标准输出或标准错误中，3.18版本支持
-   `COMMAND_ERROR_IS_FATAL`：触发致命错误并终止命令执行，方式取决于参数ANY和LAST。ANY表示任意命令执行失败都触发，LAST表示最后一个命令执行失败才触发，3.19版本支持

示例：

COMMAND WORKING_DIRECTORY：

```
execute_process(COMMAND pwd WORKING_DIRECTORY "/home")
```

TIMEOUT：

```
execute_process(COMMAND sleep 5
		COMMAND sleep 5
		TIMEOUT 1
		RESULT_VARIABLE msg)
message("1 commands execute result: ${msg}")
```

RESULT_VARIABLE RESULTS_VARIABLE：

```
execute_process(COMMAND rm test.txt
		RESULT_VARIABLE msg)
message("2 command execute result: ${msg}")

execute_process(COMMAND ls -al
		COMMAND rm test.txt
		RESULTS_VARIABLE msg_all)
message("3 command execute result: ${msg_all}")
```

OUTPUT_VARIABLE ERROR_VARIABLE：

```
execute_process(COMMAND echo "start"
		COMMAND rm "test1.txt"
		COMMAND pwd
		COMMAND rm "test2.txt"
		COMMAND pwd
		WORKING_DIRECTORY "/home"
		RESULTS_VARIABLE msg_result
		OUTPUT_VARIABLE msg_out
		ERROR_VARIABLE msg_err)
MESSAGE("4 Commands execute results: ${msg_result}")
MESSAGE("Commands output: ${msg_out}")
MESSAGE("Commands error: ${msg_err}")
```

INPUT_FILE OUTPUT_FILE ERROR_FILE

```
execute_process(COMMAND cat
               INPUT_FILE ${CMAKE_SOURCE_DIR}/test.txt
               OUTPUT_VARIABLE CAT_OUTPUT)
message(STATUS "The output of cat command was: ${CAT_OUTPUT}")

execute_process(COMMAND rm "test1.txt"
               COMMAND rm "test2.txt"
               COMMAND echo "write message to output"
               OUTPUT_FILE output
               ERROR_FILE error)
```

OUTPUT_QUIET  ERROR_QUIET

```
execute_process(COMMAND pwd
		OUTPUT_VARIABLE msg_out)
MESSAGE("5 Commands execute results: ${msg_out}")
execute_process(COMMAND pwd
		OUTPUT_VARIABLE msg_out
		OUTPUT_QUIET)
MESSAGE("6 Commands execute results: ${msg_out}")
```

COMMAND_ECHO

```
execute_process(COMMAND echo "execute commad"
 		COMMAND ls "CMakeFiles"
 		COMMAND_ECHO STDERR)
```

ECHO_OUTPUT_VARIABLE ECHO_ERROR_VARIABLE

```
execute_process(COMMAND echo "hello world"
 		OUTPUT_VARIABLE var_output)
MESSAGE("-------------------")
execute_process(COMMAND echo "hello world"
 		OUTPUT_VARIABLE var_output
 		ECHO_OUTPUT_VARIABLE)
MESSAGE("-------------------")
```

COMMAND_ERROR_IS_FATAL

```
execute_process(COMMAND pwd
 		COMMAND rm "test.txt"
 		COMMAND echo "hello world"
 		COMMAND_ERROR_IS_FATAL LAST)
```

```
execute_process(COMMAND pwd
		COMMAND rm "test.txt"
		COMMAND echo "hello world"
		COMMAND_ERROR_IS_FATAL ANY)
```

## add_custom_command

add_custom_target，顾名思义就是添加一个自定义目标。

在CMake中，add_custom_target命令用于创建一个自定义目标，这个目标不产生输出文件，只是执行用户指定的命令。它类似于Makefile中的“伪目标”（phony target），总是被认为是最新的，因此总是会执行指定的命令。

add_custom_target命令的常见用途包括运行测试、执行代码审查、执行格式化或清理工作等。它非常灵活，可以用于执行任何你想要的命令序列。

基本语法：

```
add_custom_target(Name [ALL] [command1 [args1...]]
				[COMMAND command2 [args2...] ...]
				[DEPENDS depend depend depend ... ]
				[BYPRODUCTS [files...]]
				[WORKING_DIRECTORY dir]
				[COMMENT comment]
				[JOB_POOL job_pool]
				[JOB_SERVER_AWARE <bool>]
				[VERBATIM] [USES_TERMINAL]
				[COMMAND_EXPAND_LISTS]
				[SOURCES src1 [src2...]])
```

参数含义：

-   `Name`：自定义目标的名称。

-   `ALL`：使这个目标总是被构建，无论是否指定目标。可选参数，如果设置，该目标将被添加到默认构建目标中，即执行make或cmake --build时会自动构建
-   `COMMAND`：后面跟执行的命令，可以指定多条命令，按顺序执行。

-   `DEPENDS`：该目标依赖的其他目标或文件，当这些目标或文件更改时，该目标将被重新构建。

-   `BYPRODUCTS`：指定命令生成的副产品文件。这些文件不会触发重新构建，但如果它们不存在，构建将被视为失败。

-   `WORKING_DIRECTORY`：指定命令执行时的工作目录。

-   `COMMENT`：为这个自定义目标添加一个注释，在构建过程中将显示。

-   `VERBATIM`：保证对命令行的正确转义。如果设置，命令将不会通过CMake的命令行解释器，而是直接传递给构建系统。
-   `USES_TERMINAL`：允许该命令使用调用CMake的终端。

-   `JOB_POOL`：指定此目标用于特定的 job pool（适用于支持 job pool的构建系统）。

-   `SOURCES`：添加到自定义目标的源文件，在 IDE 中可视化显示这些源文件，但实际不会编译它们。

## add_custom_command

基本语法：

它有两种命令格式：

第一种格式是，将自定义的命令添加到目标（比如lib库或者可执行文件）。这对于在构建目标之前或之后执行操作非常有用。该命令成为目标的一部分，并且仅在目标本身构建时才会执行。如果目标已经构建，则该命令将不会执行。

```
add_custom_command(TARGET <target>
                   PRE_BUILD | PRE_LINK | POST_BUILD
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [BYPRODUCTS [files...]]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [VERBATIM] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS])
```

第二种格式是，添加自定义命令，来生成指定的OUTPUT文件。

```
add_custom_command(OUTPUT output1 [output2 ...]
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [MAIN_DEPENDENCY depend]
                   [DEPENDS [depends...]]
                   [BYPRODUCTS [files...]]
                   [IMPLICIT_DEPENDS <lang1> depend1
                                    [<lang2> depend2] ...]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [DEPFILE depfile]
                   [JOB_POOL job_pool]
                   [VERBATIM] [APPEND] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS])
```

参数含义：

-   `TARGET`：指定哪个构建目标将会触发这个自定义命令。当 <target> 被构建时，这个命令将会被执行。如果目标已经构建，则该命令将不会执行。

-   `PRE_BUILD | PRE_LINK | POST_BUILD`：PRE_BUILD: 表示自定义命令将在构建目标前执行。PRE_LINK: 表示自定义命令将在链接目标前执行。POST_BUILD: 表示自定义命令将在构建目标后执行。

-   `OUTPUT`：指定由命令生成的文件。这些文件将成为后续构建步骤的依赖项。

-   `COMMAND`：要执行的命令。这可以是任何可以在命令行中运行的命令。

-   `MAIN_DEPENDENCY`：可选参数，指定主要依赖项。这通常是一个源文件，当该文件更改时，将重新运行命令。

-   `DEPENDS`：指定这个命令执行所依赖的额外文件或目标。如果任何一个依赖项改变了，那么这个自定义命令将被执行。

-   `BYPRODUCTS`：指定命令生成的副产品文件。这些文件不会触发重新构建，但如果它们不存在，构建将被视为失败。

-   `IMPLICIT_DEPENDS`：隐式依赖项。这允许你指定命令对哪些文件有隐式依赖。

-   `WORKING_DIRECTORY`：指定命令的工作目录。

-   `COMMENT`：为构建系统提供的注释，通常用于描述命令的目的。

-   `DEPFILE`：指定一个文件，它包含了这个命令的完整依赖信息。这通常是由编译器生成的 .d 文件。

-   `JOB_POOL` ：指定此目标用于特定的 job pool（适用于支持 job pool的构建系统）。

-   `VERBATIM`: 保证对命令行的正确转义。如果设置，命令将不会通过CMake的命令行解释器，而是直接传递给构建系统。保证命令和参数在所有平台上以字面意义解释。建议总是使用这个参数，以保证跨平台兼容性。

-   `APPEND`: 如果指定这个选项，那么输出文件的内容将会被追加，而不是覆盖。

-   `USES_TERMINAL`: 如果指定这个选项，表示这个命令将使用终端来执行，这可能会影响CMake的输出。

-   `COMMAND_EXPAND_LISTS`：在执行命令之前，先对命令及其参数列表进行扩展。这允许你使用生成的文件或目标作为参数等。

## add_custom_target 和 add_custom_command 的关系

在总结这两个命令的关系之前，我们先来粗略地看一看 Makefile 的规则。

### Makefile的规则

```
 target ... : prerequisites ...
 	command
 	...
 	...
```

target 也就是一个目标文件，可以是 Object File，也可以是执行文件，还可以是一个标签（Label）。

prerequisites 就是要生成那个 target 所需要的文件或是目标。

command 也就是 make 需要执行的命令（任意的 Shell 命令）。

这是一个文件的依赖关系，也就是说，target 这一个或多个的目标文件依赖于 prerequisites中的文件，其生成规则定义在 command 中。

说白一点就是说，prerequisites 中如果有一个以上的文件比 target 文件要新的话，command 所定义的命令就会被执行。这就是 Makefile 的规则。也就是 Makefile 中最核心的内容

### add_custom_target 和add_custom_command 的关系

通过上面Makefile的规则，我们发现Makefile 中最核心的内容中有几个字比较重要，就是“目标、依赖、命令”，A依赖于B，B依赖于C，要得到A，就得先通过命令得到B，要得到B，得先通过命令得到C。

add_custom_target 和add_custom_command是一样的道理。

add_custom_target 加了DEPENDS参数，依赖关系可以参考本文开头总结的add_custom_target的例子和说明。

add_custom_command两种格式。

第一种target参数已经确定了目标依赖关系，所以可以不用add_custom_target。

第二种output并没有确定目标依赖关系，DEPENDS也只是确定了OUTPUT所依赖的文件，但是并没有和目标文件有依赖关系，所以此时需要用add_custom_target和目标文件有依赖关系。
