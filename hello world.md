# PHP7 Extension - Hello World
##一、下载PHP源代码
要开发PHP扩展，需要先下载PHP的源代码，一方面是因为我们的扩展一般会用到PHP自身定义的函数和宏，另一方面我们可以利用官方提供的工具减少工作量。
我下载了PHP-7.0.2，地址是：http://cn2.php.net/get/php-7.0.2.tar.gz。
解压源码压缩包， tar xzf php-7.0.2.tar.gz，我们现在只需要关注Zend和ext这两个目录。
Zend目录里面包含了PHP的Zend Engine源代码，有些函数和宏的定义我们需要在这里面简单地看一下。
ext目录里面包含了PHP原生的扩展，以及我们开发自己的扩展时可以利用的工具，Linux下使用ext_skel，Windows下使用ext_skel_win32.php
##二、使用ext_skel工具
我们可以在ext目录下看到所有的PHP原生扩展，其中包括了熟悉的curl，json，mbstring，simplexml，sockets等扩展，还有很多没有用过甚至没有听说过的扩展，不用在意这些，我们先打开我们最熟悉的curl来看看，有config.m4配置文件，有php_curl.h，curl_file.c等源代码，还有一些中间文件，最后还有一个tests目录，里面放的curl扩展的单元测试。重点关注config.m4，php_curl.h，curl_file.c即可，最简单的场景下这三个文件就是一个扩展的全部组成部分了。

打开随便看一下，不算太复杂，但是自己写一个类似的还是挺头疼的，这时就需要用到我前面提到的ext_skel工具了。这个工具也在ext目录下，我们执行一下，./ext_skel --help，可以看到若干参数，我们用到的只有--extname=module，这里填上自己开发的扩展名称。想深入了解各个参数的作用可以看这里：http://php.net/manual/en/internals2.buildsys.skeleton.php
```shell
./ext_skel --extname=hello
```
ext目录下多了一个hello目录，我们后续的工作都在这个目录下面，工具已经为我们自动生成了一些文件。

###config.m4配置文件
开发PHP扩展，在写C代码之前，要先配置一下这里。我们打开可以看到详细的注释说明，dnl是注释语法。
如果你的扩展用到了外部依赖，就配置--with-hello选项，否则配置--enable-hello选项，删除这下面3行的del注释
```
PHP_ARG_ENABLE(hello, whether to enable hello support,
Make sure that the comment is aligned:
[  --enable-hello           Enable hello support])
```
PHP_ARG_WITH和PHP_ARG_ENABLE这两个宏用来配置configure选项，一个配置需要外部依赖的，另一个配置不需要外部依赖的
配置好的内容，在后面执行configure --help时可以看到。
###php_hello.h头文件
类似于C语音的头文件，包含了一些自定义的结构和函数声明，在这个demo中暂时不需要改动
###hello.c代码文件
真正的逻辑代码都在这个文件中，后面会详细介绍。
##三、编写代码
好了，到这一步我们终于要开始写代码了，打开hello.c文件。
整个扩展的入口是zend_module_entry这个结构，具体的定义可以在Zend目录下的zend_modules.h文件中看到，一共有十几个属性，快速跳过，我们暂时只需要"hello world"。
```
zend_module_entry hello_module_entry = {
    STANDARD_MODULE_HEADER,
    "hello",
    hello_functions,
    PHP_MINIT(hello),
    PHP_MSHUTDOWN(hello),
    PHP_RINIT(hello),       /* Replace with NULL if there's nothing to do at request start */
    PHP_RSHUTDOWN(hello),   /* Replace with NULL if there's nothing to do at request end */
    PHP_MINFO(hello),
    PHP_HELLO_VERSION,
    STANDARD_MODULE_PROPERTIES
};
```
* STANDARD_MODULE_HEADER帮我们实现了前面6个属性
* "hello"是扩展的名字
* hello_functions是扩展包含的全部方法的集合
* 后面5个宏分别代表5个扩展特定方法
* PHP_HELLO_VERSION是扩展的版本号，定义在头文件中
* STANDARD_MODULE_PROPERTIES帮我们实现了剩下的属性

暂时都不需要修改，知道这是一个入口就行。顺着这个入口，我们继续看怎么给扩展添加方法，在hello_functions[]方法数组中已经有了一个示例方法confirm_hello_compiled，我们参考它写我们的方法hello_world
```
const zend_function_entry hello_functions[] = {
    PHP_FE(confirm_hello_compiled,  NULL)       /* For testing, remove later. */
    PHP_FE(hello_world,  NULL)
    PHP_FE_END  /* Must be the last line in hello_functions[] */
};
```
先在扩展的方法数组中添加上hello_world，然后再定义hello_world。找到confirm_hello_compiled方法定义的地方，在它下面依葫芦画瓢，php_printf是Zend Engine中的printf方法。
```
PHP_FUNCTION(hello_world)
{
    php_printf("Hello World!\n");
    RETURN_TRUE;
}
```
##四、编译安装
最后就是编译安装我们的扩展了，安装过PHP扩展的同学不用看，没有经验的可以参考一下。
```
phpize
./configure
make
make install
```
现在PHP的扩展目录中已经有了hello.so这个文件，在php.ini中添加上扩展的配置
```
extension = hello.so
```
##五、测试
写一个test.php方法，执行脚本就可以看到"Hello World!"
```
<?php
hello_world();
```
