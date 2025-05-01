# 0. 前言

这是一份极简版教程，描述gtest的安装和最基础的使用，分为使用g++编译和CMakeLists组织文件编译两种方式。

对Gtest需求不大的人员，这份教程足够了。

# 1. 安装

```bash
git clone git@github.com:google/googletest.git
```



```bash
cd googletest
mkdir build 
cd build
cmake ..
sudo make install -j
```



## 1.1 测试自带用例

```bash
cd googletest/samples
g++ ../src/gtest_main.cc sample1.cc sample1_unittest.cc -o test -lgtest -lgmock -lpthread -std=c++14
./test
```

输出

```bash
Running main() from ../src/gtest_main.cc
[==========] Running 6 tests from 2 test suites.
[----------] Global test environment set-up.
[----------] 3 tests from FactorialTest
[ RUN      ] FactorialTest.Negative
[       OK ] FactorialTest.Negative (0 ms)
[ RUN      ] FactorialTest.Zero
[       OK ] FactorialTest.Zero (0 ms)
[ RUN      ] FactorialTest.Positive
[       OK ] FactorialTest.Positive (0 ms)
[----------] 3 tests from FactorialTest (0 ms total)

[----------] 3 tests from IsPrimeTest
[ RUN      ] IsPrimeTest.Negative
[       OK ] IsPrimeTest.Negative (0 ms)
[ RUN      ] IsPrimeTest.Trivial
[       OK ] IsPrimeTest.Trivial (0 ms)
[ RUN      ] IsPrimeTest.Positive
[       OK ] IsPrimeTest.Positive (0 ms)
[----------] 3 tests from IsPrimeTest (0 ms total)

[----------] Global test environment tear-down
[==========] 6 tests from 2 test suites ran. (0 ms total)
[  PASSED  ] 6 tests.
```



# 2. 使用

```cpp
// main.cpp
#include <gtest/gtest.h>

int add(int a, int b){
    return a+b;
}

// 系列名为A，当前测试用例名为B
TEST(A, B){
    EXPECT_EQ(1, 1);
}


TEST(A, C){
    EXPECT_EQ(add(1,1), 2);
}

int main(){
    testing::InitGoogleTest();
    return RUN_ALL_TESTS();
}
```



```bash
g++ main.cpp -o main -lgtest -pthread && ./main
```



输出

```bash
[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from A
[ RUN      ] A.B
[       OK ] A.B (0 ms)
[ RUN      ] A.C
[       OK ] A.C (0 ms)
[----------] 2 tests from A (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test suite ran. (0 ms total)
[  PASSED  ] 2 tests.
```



# 3. 使用CMAKE组织编译

## 3.1 目录结构

```
.
├── build
├── CMakeLists.txt
├── run.sh
└── src
    └── main.cpp
```



## 3.2 具体文件

CMakeLists.txt文件

```cmake
cmake_minimum_required(VERSION 3.5)

project(main)

set(CMAKE_CXX_COMPILE_FLAGS, "-std=c++11")

add_executable(main src/main.cpp)
target_link_libraries(main gtest gmock pthread)
```



run.sh文件

```bash
cd build
rm -rf * .*
cmake ..
make -j
./main
```



执行

```bash
sh ./run.sh
```



# 参考

1. https://blog.csdn.net/wdcyf15/article/details/108855960