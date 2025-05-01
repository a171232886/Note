---
title: Ubuntu20.04 安装 Boost 1.70.0
date: 2023-01-19 09:00:00
categories: 
- [笔记, Ubutnu]
---

# 安装

1. 官网：https://www.boost.org/users/history/

   boost 1.70.0 版本：https://boostorg.jfrog.io/artifactory/main/release/1.70.0/source/

<!--more-->   

2. 执行命令

   ```shell
   tar -xvzf boost_1_70_0.tar.gz
   cd boost_1_70_0
   sudo ./bootstrap.sh
   sudo ./b2 install
   ```

   安装之后可见

   ```bash
    /usr/local/include/boost
    /usr/local/lib/libboost
   ```

   

# 设置环境变量

```shell
sudo gedit ~/.bashrc
```

​		添加 `export LD_LIBRARY_PATH="/usr/local/lib/:$LD_LIBRARY_PATH"`

```bash
source ~/.bashrc
```



# 测试用例

a.cpp

```c++
#include <boost/thread.hpp>  
#include <iostream>  
  
void task1() {   
    // do stuff  
    std::cout << "This is task1!" << std::endl;  
}  
  
void task2() {   
    // do stuff  
    std::cout << "This is task2!" << std::endl;  
}  
  
int main (int argc, char ** argv) {  
    using namespace boost;   
    thread thread_1 = thread(task1);  
    thread thread_2 = thread(task2);  
  
    // do other stuff  
    thread_2.join();  
    thread_1.join();  
    return 0;  
}  
```

编译并执行

```bash
g++ -I./inlcude -L./lib a.cpp -lboost_thread -lboost_system -lpthread  -o a
./a
```

输出

```bash
This is task1!
This is task2!
```