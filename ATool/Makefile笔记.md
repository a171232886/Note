# 一、基本格式

```makefile
target: dependencies
	command
```

执行命令

```shell
make
```



# 二、基本情况

## （一）单个主函数

```makefile
# 生成一个main文件，需要用到print_num.o，因此会先执行print_num.o
main: main.cpp print_num.o
	g++ main.cpp print_num.o -o main
# print_num.o 需要用到 print_num.cpp
print_num.o: print_num.cpp
	g++ -c print_num.cpp
# clean 通过 make clean 执行
clean:
	rm *.o main
```

cpp和h

```c++
// main.cpp
#include"print_num.h"
int main(){
   print_num();
   return 0;
}

// print_num.cpp
#include"print_num.h"
void print_num()
{
   cout<<"123"<<endl;
}

// print_num.h
#pragma once
#include<iostream>
using namespace std;

void print_num();
```



## （二）多个主函数

```makefile
# 将编译器的值赋给变量，方便更换编译器
# 编译器的选项也如此
CC = g++
CFLAGS = -lm -Wall -g

# all 命令可以保证main1和main2都生成
all: main1 main2

main1: main1.cpp print_num.o
	$(CC) $(CFLAGS) main1.cpp print_num.o -o main1
	
main2: main2.cpp print_num.o
	$(CC) $(CFLAGS) main2.cpp print_num.o -o main2

print_num.o: print_num.cpp
	$(CC) $(CFLAGS) -c print_num.cpp

clean:
	rm *.o main1 main2
```



```c++
// main1.cpp
#include"print_num.h"
int main(){
   cout<<"main1"<<endl;
   print_num();
   return 0;
}

// main2.cpp
#include"print_num.h"
int main(){
   cout<<"main2"<<endl;
   print_num();
   return 0;
}

// print_num.cpp
#include"print_num.h"
void print_num()
{
   cout<<"123"<<endl;
}

// print_num.h
#pragma once
#include<iostream>
using namespace std;

void print_num();
```



# 三、cmake与make的关系

cmake根据CMakeList生成makefile，然后make根据makefile生成可执行文件

<img src="../images/Makefile笔记\image-20221025144322222.png" alt="image-20221025144322222" style="zoom: 80%;" />

图片来源：https://blog.csdn.net/weixin_42491857/article/details/80741060

