###前期需要准备的工作(已经在服务器上完成)



- 安装opencv，以及其他附加项    * TEST代码显示的时候需要*
  可以参考http://blog.csdn.net/kuaile123/article/details/20870731
- 安装**cmake**、**gcc**、**g++**......
- 由于**操作系统**的不同，主要是包含**头文件**的方式有所不同，所以对修改后的代码打包在附件**lib**中，将附件中的**lib**文件复制到**CentOS** ， 我将其复制在**Desktop**目录下

###利用**CMakeLists.txt**生成动态库(.so)和静态库(.a)

> ####编写库文件的**CMakeLists.txt**

```
#同时生成动态库和静态库
cmake_minimum_required(VERSION 2.8)
project(RefineOffaxis)
aux_source_directory(. DIR_SRCS)
include_directories(~/Desktop/eigen)
ADD_LIBRARY(RefineOffaxis SHARED ${DIR_SRCS})
SET_TARGET_PROPERTIES(RefineOffaxis PROPERTIES VERSION 1.0 SOVERSION 1)
ADD_LIBRARY(RefineOffaxis_s STATIC ${DIR_SRCS})
SET_TARGET_PROPERTIES(RefineOffaxis_s PROPERTIES OUTPUT_NAME "RefineOffaxis")
INSTALL(TARGETS RefineOffaxis RefineOffaxis_s
          LIBRARY DESTINATION lib 
	      ARCHIVE DESTINATION lib)
INSTALL(FILES refineOffaxis.h DESTINATION include/RefineOffaxis)
```

#### 1 对上面CMakeLists.txt的解析

>cmake_minimum_required(VERSION 2.8) 指定cmake需要的最小版本

>project(RefineOffaxis) 指定项目名

>aux_source_directory(. DIR_SRCS)  *aux_source_directory(dir variable)* 获取指定目录下的所有文件，保存到variable中,包括 *.c .C .c++ .cc .cpp .cxx .m .M .mm .h .hh .h++ .hm .hpp .hxx .in .txx*文件，上面示例是将当前目录中的源文件名称赋值给变量 DIR_SRCS

>include_directories(~/Desktop/eigen) *include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 …)* 用来设定目录，这些设定的目录将被编译器用来查找**include**文件 ，由于我们生成动态库和静态库的过程中需要使用**eigen**，而我又将**eigen**放在**Desktop**目录下，所以包含的形式如上所示，为了美观，可以将**eigen**放在**usr/local/include/**目录下

>ADD_LIBRARY(RefineOffaxis SHARED ${DIR_SRCS})*add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 source2 … sourceN)*添加一个名为**<name>**的库文件，指定**SHARED**,**STATIC**.如果指定为**SHARED**则生成动态库(**.so**),如果指定为**STATIC**，则生成静态库(**.lib**)，在上例中，我们先生成动态库，所以参数选为 **SHARED**

>SET_TARGET_PROPERTIES(RefineOffaxis PROPERTIES VERSION 1.0 SOVERSION 1) 为动态库添加版本号,**VERSION**指代动态库版本，**SOVERSION**指代API版本

>ADD_LIBRARY(RefineOffaxis_s STATIC ${DIR_SRCS})上面介绍过**ADD_LIBRARY**命令了，这个是用来生成**STATIC**静态库，同时，要求生成静态库的名字为**RefineOffaxis_s**和动态库区分

>SET_TARGET_PROPERTIES(RefineOffaxis_s PROPERTIES OUTPUT_NAME "RefineOffaxis")这句话的意思相当于我们给静态库**RefineOffaxis_s**起了个别名**RefineOffaxis**，目的是为了生成库的美观

>INSTALL(TARGETS RefineOffaxis RefineOffaxis_s LIBRARY DESTINATION lib  ARCHIVE DESTINATION lib)
>INSTALL(FILES refineOffaxis.h DESTINATION include/RefineOffaxis)
>上面的两条命令的意思都是添加安装功能，方便将动态库 静态库 头文件一块安装到需要的目录

#### 2 CMakeLists.txt的使用
 
 在**CMakeLists.txt** 所在目录下 *mkdir build* 采用外部构建的模式
 
 [haiqing@localhost build]$  cd build  
 [haiqing@localhost build]$  cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..   
 [haiqing@localhost build]$  make  
 [haiqing@localhost build]$  sudo make install  
 
最终的结果是 **so**库和**a**库 安装在 **/usr/local/lib**目录下，**RefineOffaxis.h**安装在**/usr/local/include/RefineOffaxis**

### 测试生成的动态库与静态库

####  1 目录建立

将附件中的**src**文件夹复制到和附件**lib**同目录下
**src**中有两个文件，一个是**CMakeLists.txt**,一个是**main.cpp**

#### 2 解析CMakeList.txt

```
cmake_minimum_required(VERSION 2.8) 
PROJECT(RefineOffaxis)
FIND_PACKAGE(OpenCV REQUIRED)
ADD_EXECUTABLE(RefineOffaxis main.cpp)
target_link_libraries(RefineOffaxis ${OpenCV_LIBS} )
#TARGET_LINK_LIBRARIES(RefineOffaxis RefineOffaxis)
TARGET_LINK_LIBRARIES(RefineOffaxis libRefineOffaxis.so)
#TARGET_LINK_LIBRARIES(RefineOffaxis libRefineOffaxis.a)
INCLUDE_DIRECTORIES(/usr/include/RefineOffaxis)
INCLUDE_DIRECTORIES(/usr/locla/include)

```

主要关注第**6.7.8**行，如果注释掉第**7.8**行，单单使用第**6**行时，通过**lld**命令可以知道该方法调用的动态库，效果和注释掉**6.8**两行的效果一样，如果想使用静态库，则注释掉**6.7**行就可以了。
