# 一、gdb初步

## 1. 创建一个hello.cpp

```c
#include <iostream>
using namespace std;

void t1(){
  int b = 1;
  b = 3;
  return;
}

int main(){
  cout<<"Hello World"<<endl;
  int a = 5;
  t1();
  return 1;
}
```

## 2. 进入gdb

- 编译	 `g++ -g hello.cpp`

- 进入gdb    `gdb a.out`

## 3. 基础gdb命令

- 执行程序	 `r` (run)

- 退出gdb	 `q` (quit)

- 查看源代码	 `l` (list)

### 断点相关

- 打断点 `b 函数名` 或 `b 程序行数` （break）
  如 `b main` 或 `b 4`

- 删除断点	 `clear 函数名` 或 `clear 程序行数`

- 查看断点	 `info b`

### 执行相关

- 继续执行到下一个断点或结束 `c` （continue）

- 下一步 `n` next, 显示下一句未执行的语句

- 步入 `s` （step）

### 查看变量值

- `p a` (print)

# 二、gdb 小技巧
1. 写shell命令

    `shell ls`
2. 开启日志功能

    `set logging on`, 会生成一个`gdb.txt`

3. 查看使用方式
    终端输入 `man gdb`

4. gdb 查看函数堆栈

   bt

5. 切换函数栈帧

   up 数字 ， 表示向回走几个

    down 数字