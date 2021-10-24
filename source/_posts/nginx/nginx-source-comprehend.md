---
title: nginx源代码0.1.0版本阅读
date: 2021-10-18 19:03:00
categories:
- Nginx # 这是分类，直接写就好，他会自定创建，tags 也是一样
tags:
- nginx # 这是标签
---

```sh
/* Standard file descriptors. */
#define STDIN_FILENO 0 /* Standard input. */
#define STDOUT_FILENO 1 /* Standard output. */
#define STDERR_FILENO 2 /* Standard error output. */
```

va_start与va_arg函数的使用：

```c
#include "dong.h"

void func(char *msg, ...)
{
    int argcno = 0;
    va_list    args;
    char *str = NULL;

    va_start(args, msg);
    while (1)
    {
        str = va_arg(args, char *);
        if (strcmp(str, "") == 0)
            break;
        printf("id:[%d], args:[%s]\n", argcno++, str);
    }
    va_end(args);
    return ;
}

int main(int argc, char *argv[])
{
    func("DEAO", "THIS", "IS", "A", "DEMO", "");

    return 0;
}
```

返回结果：

```c
id:[0], args:[THIS]
id:[1], args:[IS]
id:[2], args:[A]
id:[3], args:[DEMO]
```

