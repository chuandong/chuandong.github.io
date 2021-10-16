Makefile

1.  软件编译过程：

    预处理：gcc -E test.c -o test.i

      编 译： gcc -S test.i -o test.s

      汇 编： gcc -c test.s -o test.o

      链 接： gcc -o test test.o

2.  命令前面加上@符号，表示不打印出这条命令，例如：

    @rm -f *.o

3.  目标：依赖列表

    ​		命令列表

    ```makefile
    CC = gcc
    TAG = test_sum
    SCRC = test_func.c test_sum.c
    OBJ = test_func.o test_sum.o
    
    ${TAG} : ${OBJ}
            ${CC} $^ -o $@   # #@指的就是 ${OBJ} $^指的是 ${TAG}
    #还可以写成
    #	${TAG}:${SCRC:.c=.o}  #表示.c 生成的 .o
    #	${CC} $^ -o $@
    #下面的规则称为模式规则，%表示循环取出对应的文件，直到取完为止
    %.o : %.c
            ${CC} $< -c -o $@ # $< 指的是%.o
    
    clean:
            rm -rf ${OBJ} ${TAG}
    ```

4.   ```makefile
     CURR_DIR = $(shell pwd) #表示当前的目录
     ```

5.   ```makefile
     CFLAGS = -I$(CURR_DIR)/head
     #头文件放在不同的目录下，引用头文件
     $(TAG):$(SCRC:.c=.o)
     	${CC} $^ $(CFLAGS) -o $@
     #下面的规则说明，当头文件更新时，所有引用的头文件的代码都将更新
     %.d : %.c
     	$(CC) -MM $< > $@
     #告诉make，将上面的模式规则中的命令执行结果包含进当前文件（三种写法意思一样）
     include $(SCRC:.c=.d)
     -include $(SCRC:.c=.d)
     sinclude $(SCRC:.c=.d)
     ```

6.   ```makefile
     # .PHONY 后面生成的是一个伪目标
     .PHONY clean
     ```

7.   ```makefile
     #搜索源文件，设定文件名
     SRC = $(wildcard *.c)
     OBJ = $(SRC:%.c=%.o)
     ```

     