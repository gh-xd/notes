# 链接工具

## 目标

- 了解不同工具的配置方法
- 挖坑

## SDL2和SDL2_Image

> 使用Homebrew

{==配置头文件时候的大坑==}

如果从*Homebrew*的文件夹中找到sdl2和sdl2_image两个源文件，并且将头文件目录只设置到上一层的时候：

- SDL2: /opt/homebrew/Cellar/sdl2/2.24.0/include/
- SDL2_Image: /opt/homebrew/Cellar/sdl2_image/2.6.2/include/

在CPP文件中引入两个文件的代码则为：

- SDL2: #include<SDL2/SDL.h>
- SDL2_Image: #include<SDL2/SDL_image.h>

这个时候会报错

```bash
...: fatal error: 'SDL.h' file not found
```

这个错误的原因好像是，在`SDL_image.h`这个头文件中，找不到`SDL.h`这个头文件。

原因可能是，在将这两个源文件一起构建时，自己写的CPP内引入SDL2头文件的方式为`SDL2/SDL.h`，和`SDL_image.h`中引入SDL2头文件的方式不一样，导致无法找到。

**解决办法**

在Cmake中，引入头文件的目录设置到最后一层，以便于SDL.h可以直接引入



## GLFW

因为自己用的是arm架构的电脑，以下记录为从glfw官网下载预编译版本的过程

> 下载预编译版

下载文件中：

- 头文件目录: include
- 库文件目录: lib-arm64

> 编写CMakeLists.txt

创建CPP项目文件夹

- .h头文件放在include
- .a/.dylib静态库文件放在src

```cmake
cmake_minimum_required(VERSION 3.0)

# optional
set(CMAKE_CXX_STANDARD 17)

project(GLFW_TEST)

# 在CMAKE_CXX_FLAGS中追加参数，等同于在g++ 后面追加参数 (MacOS需要使用)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -framework Cocoa -framework OpenGL -framework IOKit")

# 包含头文件的文件夹，内含glfw3.h文件
include_directories(include)

# 链接库的文件夹，内含.a/.dylib文件
link_directories(src)

add_executable(glfw_test main.cpp)

# 需要链接的文件libglfw3.a
target_link_libraries(glfw_test glfw3) 

```



## Eigen

Eigen库比较有意思，不需要引用头文件即可使用。

是不是有别的办法，再留意下。

> 下载Eigen文件夹

1. 下载完的文件名，例如：`eigen-3.4.0`
2. 里面有个`Eigen`文件夹
3. 直接拷贝到项目的`include`文件加中
4. 在代码中使用如下

例如使用Vector2f，可以直接创建对象：

```c++
...
#include <Eigen/Core>

using namespace Eigen;
...
// Matrix<float, 2, 1> Vector2f;
Vector2f A_speedVec(A->speedX, A->speedY);
Vector2f B_speedVec(B->speedX, B->speedY);
```





## 学习参考

- https://gigi.nullneuron.net/gigilabs/loading-images-in-sdl2-with-sdl_image/