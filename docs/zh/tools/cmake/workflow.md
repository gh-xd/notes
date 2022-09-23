# CMake编译工程

## 1. 目录结构

在项目主目录下存在一个CMakeLists.txt文件

## 2. 编译规则

### 2.1 子文件夹包含CMakeLists.txt

此时，子文件夹的编译规则在子文件夹的txt文件中

必须在主目录的CMakeLists.txt中通过add_subdirectory添加自目录

### 2.2 只有一个CMakeLists.txt

此时，只有主目录下一个CMakeLists.txt文件，子文件夹的编译规则体现在主目录的txt文件中

## 2. 编译流程

Linux下Cmake构建C/C++工程的流程：

1. 手动编写CMakeLists.txt文件
2. 执行`cmake PATH`命令生成Makefile（PATH是顶层CMakeLists.txt所在的目录）
3. 执行`make`命令进行编译

## 3. 构建方式

### 3.1 内部构建

in-source build

在同级目录下产生大量中间文件，这些并不是最终需要的，杂乱无章

不推荐

```cmake
# 内部构建

# 在当前目录编译本目录的CMakeLists.txt，生成Makefile和其他文件
cmake .

# 执行make命令，生成target
make
```

### 3.2 外部构建

out-of-source build

编译输出文件与源文件放到不同目录中

推荐

```cmake
# 外部构建

# 1. 创建build目录，进入
mkdir build && cd build

# 2. 编译上级目录的CMakeLists.txt，生成Makefile和其他文件
cmake ..

# 3. 生成target
make
```

