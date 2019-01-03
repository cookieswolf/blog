# JS虚拟机Duktape嵌入教程

- [Duktape官网](https://www.duktape.org/index.html)

* 注:本示例使用`2.3.0`版本源码。`Ubuntu`系统

## 下载并编译源码

- 下载源码到本地并解压

下载地址:https://www.duktape.org/download.html

本示例使用`2.3.0`版本

```
wget https://www.duktape.org/duktape-2.3.0.tar.xz
tar xvf duktape-2.3.0.tar.xz
```

- 编译源码

```
cd duktape-2.3.0/
make -f Makefile.cmdline
```

编译完成以后会在根目录下生成一个`duk`的可执行文件。我们尝试运行`duk`并编写一个`HelloWord`的JS代码。如果出现以下结果。说明编译成功

```
$ ./duk
((o) Duktape 2.3.0 (v2.3.0)
duk> print('Hello world!')
Hello world!
= undefined
```


## JS调C方法

我们在其项目示例的基础上做修改来演示，修改文件在`examples/hello/`目录下

- 首先编译官方示例

```
make -f Makefile.hello
```

编译完成后会在根目录生成一个`hello`的可执行文件，然后直接运行`hello`可执行文件,运行会有如下结果

```
./hello
Hello world!
2+3=5
```

- 在`examples/hello/`目录下创建`hello.js`脚本文件

修改`hello.c`文件如下

```
/*
 *  Very simple example program
 */

#include "duktape.h"
#include <stddef.h>

static duk_ret_t native_print(duk_context *ctx) {
	duk_push_string(ctx, " ");
	duk_insert(ctx, 0);
	duk_join(ctx, duk_get_top(ctx) - 1);
	printf("%s\n", duk_safe_to_string(ctx, -1));
	return 0;
}

// 定义了一个动态参数加法器，可以输入N个参数，最后得到N个参数的和
static duk_ret_t native_adder(duk_context *ctx) {
	int i;
	int n = duk_get_top(ctx);  /* #args */
	double res = 0.0;

	for (i = 0; i < n; i++) {
		res += duk_to_number(ctx, i);
	}

	duk_push_number(ctx, res);
	return 1;  /* one return value */
}

// 此处为多个参数入参，与返回多个参数给JS的实现方式
static duk_ret_t native_person(duk_context *ctx){
	// 当输入多个参数时，参数按照数组下标顺序依次对应取出
	const char *name = duk_safe_to_string(ctx,0);
	const char *sex = duk_safe_to_string(ctx,1);
	size_t age = duk_to_uint32(ctx,2);

	// 多个参数返回可以包装成Object对象，依次存入key/value值返回给JS调用
	duk_idx_t person;
	person = duk_push_object(ctx);

	duk_push_string(ctx,name);
	duk_put_prop_string(ctx,person,"name");

	duk_push_string(ctx,sex);
	duk_put_prop_string(ctx,person,"sex");

	duk_push_int(ctx,age);
	duk_put_prop_string(ctx,person,"age");

	return 1;
}

// 加载JS脚本文件方法
char *loadJS(const char *filepath, size_t *size) {
  if (size != NULL) {
    *size = 0;
  }

  FILE *f = fopen(filepath, "r");
  if (f == NULL) {
    return NULL;
  }

  // get file size.
  fseek(f, 0L, SEEK_END);
  size_t file_size = ftell(f);
  rewind(f);

  char *data = (char *)malloc(file_size + 1);
  size_t idx = 0;

  size_t len = 0;
  while ((len = fread(data + idx, sizeof(char), file_size + 1 - idx, f)) > 0) {
    idx += len;
  }
  *(data + idx) = '\0';

  if (feof(f) == 0) {
    // free(static_cast<void *>(data));
	free(data);
    return NULL;
  }

  fclose(f);

  if (size != NULL) {
    *size = file_size;
  }

  return data;
}

int main(int argc, char *argv[]) {
	// 初始化上下文
	duk_context *ctx = duk_create_heap_default();
	// 初始化参数
	(void) argc; (void) argv;  /* suppress warning */
	// 将自定义C方法压入栈中
	duk_push_c_function(ctx, native_print, DUK_VARARGS);
	// 将print方法注册到global对象中，暴露print方法给JS调用(这里print为对外的方法名)
	duk_put_global_string(ctx, "print");

	// 注册adder方法
	duk_push_c_function(ctx, native_adder, DUK_VARARGS);
	duk_put_global_string(ctx, "adder");

	// 注册person实例化方法
	duk_push_c_function(ctx,native_person,DUK_VARARGS);
	duk_put_global_string(ctx,"newPerson");

	// 使用加载JS脚本的方式
	size_t size = 0;
	// 注意:此处JS脚本文件路径是相对于编译出来的可执行文件的。
	char *jsFile = loadJS("./examples/hello/hello.js",&size);
	duk_eval_string(ctx, jsFile);

	duk_pop(ctx);  /* pop eval result */

	// 释放上下文
	duk_destroy_heap(ctx);
	// 释放文件
	free(jsFile);
	return 0;
}

```

`hello.js`内容如下

```
// JS调用print方法
print('JS call C function')

// JS调用adder方法
var adder_res = adder(2, 3)
print("adder function result = ",adder_res)

// person实例化方法(主要是多参数传入，接收，与返回)
var person = newPerson("john","男",19);
print("person function result = ",JSON.stringify(person))
```

- 编译

`cd`到项目根目录

```
make -f Makefile.hello
```

- 执行

```
./hello
```

如果返回如下内容说明调用成功

```
JS call C function
adder function result =  5
person function result =  {"name":"john","sex":"男","age":19}
```

## C调用JS

修改`examples/guide/`目录

- 修改`examples/guide/processlines.c`文件如下

```
/* processlines.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "duktape.h"

/* For brevity assumes a maximum file length of 16kB. */
static void push_file_as_string(duk_context *ctx, const char *filename) {
    FILE *f;
    size_t len;
    char buf[16384];

    f = fopen(filename, "rb");
    if (f) {
        len = fread((void *) buf, 1, sizeof(buf), f);
        fclose(f);
        duk_push_lstring(ctx, (const char *) buf, (duk_size_t) len);
    } else {
        duk_push_undefined(ctx);
    }
}

int main(int argc, const char *argv[]) {
    duk_context *ctx = NULL;
    (void) argc; (void) argv;

    ctx = duk_create_heap_default();
    if (!ctx) {
        printf("Failed to create a Duktape heap.\n");
        exit(1);
    }

    push_file_as_string(ctx, "./examples/guide/process.js");
    if (duk_peval(ctx) != 0) {
        printf("Error: %s\n", duk_safe_to_string(ctx, -1));
        goto finished;
    }
    duk_pop(ctx);  /* ignore result */
    duk_push_global_object(ctx);
    duk_get_prop_string(ctx, -1 /*index*/, "processLine");
    // 按照顺序传递参数(对应:line,name,age的参数)
    duk_push_string(ctx, "line value");
    duk_push_string(ctx, "john");
    duk_push_int(ctx, 19);

    // 注意这里中间参数3表示对应上面JS方法要传3个参数(具体根据自定义的JS方法参数个数来确定)
    if (duk_pcall(ctx, 3 /*nargs*/) != 0) {
        printf("Error: %s\n", duk_safe_to_string(ctx, -1));
    } else {
        // 这里将JS代码return的字符串打印出来
        printf("%s\n", duk_safe_to_string(ctx, -1));
    }
    duk_pop(ctx);

 finished:
    duk_destroy_heap(ctx);

    exit(0);
}
```

- 修改`examples/guide/process.js`文件内容如下

```
// process.js
function processLine(line,name,age) {
    var obj = {line,name,age}
    return JSON.stringify(obj)
}
```

- 在项目根目录添加`Makefile.guide`文件，加入如下内容

```
#
#  Example Makefile for building a program with embedded Duktape.
#
#  There are two source sets in the distribution: (1) combined sources where
#  you only need duktape.c, duktape.h, and duk_config.h, and (2) separate
#  sources where you have a bunch of source and header files.  Whichever
#  you use, simply include the relevant sources into your C project.  This
#  Makefile uses the combined source file.
#

DUKTAPE_SOURCES = src/duktape.c

# Compiler options are quite flexible.  GCC versions have a significant impact
# on the size of -Os code, e.g. gcc-4.6 is much worse than gcc-4.5.

CC = gcc
CCOPTS = -Os -pedantic -std=c99 -Wall -fstrict-aliasing -fomit-frame-pointer
CCOPTS += -I./src  # for combined sources
CCLIBS = -lm
DEFINES =

# If you want a 32-bit build on a 64-bit host
#CCOPTS += -m32

# Use the tools/configure.py utility to modify Duktape default configuration:
# http://duktape.org/guide.html#compiling
# http://wiki.duktape.org/Configuring.html

# For debugging, use -O0 -g -ggdb, and don't add -fomit-frame-pointer

processlines: $(DUKTAPE_SOURCES) examples/guide/processlines.c
	$(CC) -o $@ $(DEFINES) $(CCOPTS) $(DUKTAPE_SOURCES) examples/guide/processlines.c $(CCLIBS)

```

- 在项目根目录进行编译

```
make -f Makefile.guide
```

编译完成以后再项目根目录会生成一个`processlines`可执行文件

- 执行可执行文件

在项目根目录执行可执行文件

```
./processlines
```

如果返回一下结果，说明C调JS方法成功

```
{"line":"line value","name":"john","age":19}
```

这样我们就可以在C/C++编写的底层链中嵌入JS虚拟机来支持JS智能合约语言了。