---
title: cjson
date: 2022-06-27 15:31:55
tags: cjson
description: cjson
---

#### 1.cjson的源地址

> cJSON是一个轻量级且易于扩展的JSON解析开源库。
>
> github地址为：https://github.com/DaveGamble/cJSON

#### 2.  **cjson的数据结构**

```c
/* cJSON结构 */
typedef struct cJSON
{
    struct cJSON *next;
    struct cJSON *prev;
    struct cJSON *child;
    int type;
    char *valuestring;
    /*不应该直接向valueint写数据，而是使用cJSON_SetNumberValue函数进行赋值*/
    int valueint;
    double valuedouble;
    char *string;
} cJSON;
```

> - 一个`cJSON`结构体存储一个JSON的值。`cJSON`结构体中的`type`是指向JSON值的类型，同时是以`bit-flag`的形式存储，这意味着不能仅仅通过比较`type`的值来判断JSON值的类型。
> - 可以使用`cJSON_Is...`函数来检查`cJSON`结构体存储JSON的值的类型，它会对空指针进行检查，同时返回一个布尔值来判断否是该值。

#### 3.cjson的值判断函数

> - `cJSON_Invalid`：无效值，没有存储任何的值。当成员全部清零时便是该值
> - `cJSON_False`：为假的布尔值
> - `cJSON_True`：为真的布尔值
> - `cJSON_NULL`：空值
> - `cJOSN_Number`：数值，既作为双精度浮点数存储于`valuedouble`,又作为整型存储于`valueint`。如果数值超过了整型的范围，`valueint`将被赋值为`INT_MAX`或者`INT_MIN`
> - `cJSON_String`：字符串，以’\0’的形式结尾，存储于`valuestring`
> - `cJSON_Array`：数组，通过`cJSON`节点链表来存储array值，每一个元素使用`next`和`prev`进行相互连接，第一个成员的`prev`值为NULL，最后一个成员的`next`值为NULL。同时使用成员`child`指针来指向该链表
> - `cJSON_Object`：对象，与数组有相似的存储方式，唯一不同的是对象会将键值放入成员`string`中
> - `cJSON_Raw`：JSON格式的字符串，以’\0’结尾，存储于`valuestring`。在一次又一次的打印相同的JSON场景下，它能够节省内存。cJSON不会在解析json格式的字符串时产生这种类型。值的注意的是cJSON库不会检查其值是否是合法的
> - `cJSON_IsReference`：成员`child`指针或者`valuestring`指向的节点并不属于自己，自己仅仅是一个引用。因此，`cJSON_Delete`和其他相关的函数只会释放这个引用本身，而不会去释放`child`或者`valuestring`
> - `cJSON_StringIsConst`：成员`string`指向的是一个字面量，因此，`cJSON_Delete`和其他相关的函数不会试图去释放`string`的内存 

#### 4.cjson创建基本类型的函数

> - `null`：`cJSON_CreateNull`
> - `booleans`：`cJSON_CreateTrue`，`cJSON_CreateFalse`或者`cJSON_CreateBool`
> - `numbers`：`cJSON_CreateNumber`
> - `strings`：`cJSON_CreateString`

<!--对于每一种类型的值都有一个对应的函数cJSON_Create...来创建。这类函数会动态分配一个cJSON的结构，所以需要在使用完后用cJSON_Delete释放掉内存，以避免内存泄漏。
注意：当你已经把一个节点加入了一个数组或者对象，你就不能在用cJSON_Delete去释放这个节点的内存了，当该数组或者对象被删除时，这个节点也会被删除-->

#### 5.cjson数组

##### 5.1 数组的创建函数

> - 使用cJSON_CreateArray函数来创建一个新的空数组;
> - 使用cJSON_CreateArrayReference函数可以创建一个数组的引用，因为它没有属于自己的内容，所以它的子元素不会被cJSON_Delete给删除;

##### 5.2数组元素的添加函数

> - 使用cJSON_AddItemToArray函数可以在数组的最后增加元素;
> - 使用cJSON_AddItemReferenceToArray函数将会增加一个元素去引用其他的节点，这就意味着cJSON_Delete不会去删除这个元素的child或者valuestring属性，因此当这些属性在其他地方使用的时候，不用担心重复释放内存的事情发生;
> - 使用cJSON_InsertItemInArray函数可以将一个新元素插入数组中的0索引的位置，旧的元素的索引依次加1;

##### 5.3数组元素的修改/删除函数

> - 如果你想根据索引去移除数组中的一个元素并且继续去使用它，可以使用cJSON_DetachItemFromArray函数，它将会返回被分离的数组。为避免内存泄漏，确保将返回值赋值给一个指针;
> - 当你需要替换数组中的某一个元素时，cJSON_ReplaceItemInArray函数使用索引的方式来进行替换;
> - cJSON_ReplaceItemViaPointer函数使用指向该元素的指针，同时如果失败则会返回0。这两个函数会分离出旧的元素并删除它，同时在这个位置加入新的元素;

##### 5.4数组元素的获取函数

> - 使用cJSON_GetArraySize函数得到数组的大小;
> - 使用cJSON_GetArrayItem得到一个元素的索引;

<!--因为数组是以链表的方式进行存储的，所以通过索引的方式进行遍历效率是很低的( O(n^2) )。建议使用宏cJSON_ArrayForEach来遍历数组，它具有时间复杂度为( O(n) )-->

#### 6.cjson对象

##### 6.1对象的创建函数

> - 使用cJSON_CreateObject函数来创建一个新的空对象;
> - 使用cJSON_CreateObjectReference函数可以创建一个对象的引用，因为它没有属于自己的内容，所以它的子元素不会被cJSON_Delete给删除;

##### 6.2对象元素的添加函数

> - 使用cJSON_AddItemToObject函数来增加一个元素到对象里；
> - 使用cJSON_AddItemToObjectCS函数来增加对象里的元素时，使用的键值（结构体cJSON中string成员）是一个引用或者是字面量，因此它会被cJSON_Delete给忽略;
> - 使用cJSON_AddItemReferenceToArray函数将会增加一个元素去引用其他的节点，这就意味着cJSON_Delete不会去删除这个元素的child或者valuestring属性，因此当这些属性在其他地方使用的时候，不用担心重复释放内存的事情发生;

##### 6.3对象元素的修改/删除函数

> - 使用cJSON_DetachItemFromObjectCaseSensitive函数来从对象中分离出一个元素，从函数命名可以看出对于指定的键值是大小写敏感的，它将会返回被分离的数组。为避免内存泄漏，确保将返回值赋值给一个指针;
> - 使用cJSON_DeleteItemFromObjectCaseSensitive函数来从一个对象中删除一个元素，可以把它看成先从对象中分离出该元素然后在删除;
> - 使用cJSON_ReplaceItemInObjectCaseSensitive函数键值查找的方式来进行替换。这个函数会分离出旧的元素并删除它，同时在这个位置加入新的元素；
> - 使用cJSON_ReplaceItemViaPointer函数指向该元素的指针来查找并替换，同时如果失败则会返回0。这个函数会分离出旧的元素并删除它，同时在这个位置加入新的元素；

##### 6.4对象元素的获取函数

> - 使用cJSON_GetArraySize来得到对象里元素的个数；
> - 使用cJSON_GetObjectItemCaseSensitive来访问对象中的某一个元素；
> - 使用宏cJSON_ArrayForEach来遍历一个对象；
> - cJSON同样也提供便利的工具函数来快速的在对象内部创建一个新的元素，比如说cJSON_AddNullToObject函数将会返回新加的元素指针，如果失败则返回NULL；

#### 7.cjson解析函数

> - 可以使用cJSON_Parse函数来一些以’\0’结尾的字符串进行解析;

```c
cJSON *json = cJSON_Parse(string);
```

> - 解析后的结果是cJSON的树状的数据结构，一旦解析成功，就有责任在使用完后使用cJSON_Delete释放内存;
> - 默认分配内存使用的是malloc函数，释放内存使用的是free函数。但是可以使用cJSON_InitHooks函数来全局性改变;
> - 当一个错误发生时，cJSON_GetErrorPtr函数可以得到指向输入字符串中错误的位置的指针。值的注意的是在多线程的情况下，该函数会产生竞争条件，更好的方法是使用带有return_parse_end参数的cJSON_ParseWithOpts函数;
> - 如果你想有更多的选项，使用cJSON_ParseWithOpts(const char *value, const char **return_parse_end, cJSON_bool require_null_terminated)函数，return_parse_end返回输入的JSON字符串的结尾或者一个错误发生的地方（从而在保障线程安全的情况下替换cJSON_GetErrorPtr函数）。require_null_terminated如果该值设为1，那么当输入的字符串在有效的以’\0’结尾的json字符串后还包含其他的数据，就会报错;

#### 8.cjson打印函数

> - 使用`cJSON_Print`函数将一个`cJSON`数据结构打印为字符串;

```c
char *string = cJSON_Print(json);
```

> - 该函数将会动态分配内存给一个字符串，将JSON表达式放入其中。一旦该函数返回，就有责任释放该内存（默认是`free`，取决于设置的`cJSON_InitHooks`）;
> - `cJSON_Print`将会用空白符来格式化JSON字符串。可以使用`cJSON_PrintUnformatted`来无格式化的打印;
> - 如果你关于返回的结果的字符串的大小有一个想法，你可以使用`cJSON_PrintBuffered(const cJSON *item, int prebuffer, cJSON_bool fmt)`函数。`fmt`是一个决定是否用空白字符格式化JSON字符串，`prebuffer`指出了所用的第一个缓冲区大小。`cJOSN_Print`当前使用256字节的缓冲区大小。一旦打印超过了大小，新的缓冲区会被动态分配，在继续打印之前旧的缓冲区里的内容复制到新的缓冲区里;
> - 使用`cJSON_PrintPreallocated(cJSON *item, char *buffer, const int length, const cJSON_bool format)`函数可以完全避免动态的内存分配，该函数需要指向缓冲区的指针和该缓冲区的大小，如果缓冲区过小，打印将会失败，函数返回0。一旦成功，函数返回1。值得注意的是需要准备超过实际需要的字节还要多5个字节，因为cJSON并不是100%的精确估计提供的内存是否足够;

#### 9. cjson代码

```c
#include <stdio.h>
#include <errno.h>
#include "cJSON.h"

int main(int argc, char *argv[])
{
    const char *cjson_version = NULL;
    char *json_data = NULL;
    
    cjson_version = cJSON_Version();
    
    printf("cjson的版本为:[%s]\n", cjson_version);
    
    /*创建json格式的报文*/
    cJSON *subobj = cJSON_CreateObject();
    cJSON_AddItemToObject(subobj, "TX_CODE", cJSON_CreateString("816060"));
    cJSON_AddItemToObject(subobj, "INSU_CODE", cJSON_CreateString("0073"));
    cJSON_AddItemToObject(subobj, "AMT", cJSON_CreateNumber(100));
    cJSON_AddItemToObject(subobj, "BOOL", cJSON_CreateBool(1));
    
    cJSON *arry = cJSON_CreateArray();
    cJSON_AddItemToArray(arry, cJSON_CreateString("Testing"));
    cJSON_AddItemToArray(arry, cJSON_CreateNumber(9999));
    cJSON_AddItemToArray(arry, cJSON_CreateBool(0));
    cJSON_AddItemToObject(subobj, "ARRY", arry);
    
    cJSON *obj = cJSON_CreateObject();
    
    cJSON_AddItemToObject(obj, "PUBLIC", subobj);
    
    json_data = cJSON_Print(obj);
    printf("json data:[%s]\n", json_data);
    
    /*获取json报文里的内容*/
    cJSON *outJson = cJSON_Parse(json_data);
    if (outJson == NULL)
    {
        printf("cJSON_Parse error. msg:[%s]\n", strerror(errno));
        return -1;
    }
    
    cJSON *node, *ele;
    char *fun_name = NULL;
    
    node = outJson->child->child;
    
    while (node != NULL)
    {
        fun_name = node->string;
        ele = cJSON_GetObjectItem(outJson->child, fun_name);
        if (ele->type == cJSON_String)
            printf("key:[%s], value:[%s]\n", fun_name, ele->valuestring);
        else if (ele->type == cJSON_Number)
            printf("key:[%s], value:[%lf]\n", fun_name, ele->valuedouble);
        else if (ele->type ==cJSON_True)
            printf("key:[%s], value:[%s]\n", fun_name, "True");
        else if (ele->type ==cJSON_False)
            printf("key:[%s], value:[%s]\n", fun_name, "False");
        else if (ele->type == cJSON_Array)
        {
            int len = cJSON_GetArraySize(ele);
            printf("Arry Size:[%d]\n", len);
            int i = 0;
            cJSON *subarry = NULL;
            for (; i < len; i++)
            {
                subarry = cJSON_GetArrayItem(ele, i);
                if (subarry->type == cJSON_String)
                    printf("key:[%s], value:[%s]\n", fun_name, subarry->valuestring);
                else if (subarry->type == cJSON_Number)
                    printf("key:[%s], value:[%lf]\n", fun_name, subarry->valuedouble);
                else if (subarry->type ==cJSON_True)
                    printf("key:[%s], value:[%s]\n", fun_name, "True");
                else if (subarry->type ==cJSON_False)
                    printf("key:[%s], value:[%s]\n", fun_name, "False");
            }
            
        }   
        node = node->next;
    }
    cJSON_Delete(outJson);
    return 0;
}
```

> 运行结果：
>
> cjson的版本为:[1.7.15]
> json data:[{
>         "PUBLIC":       {
>                 "TX_CODE":      "816060",
>                 "INSU_CODE":    "0073",
>                 "AMT":  100,
>                 "BOOL": true,
>                 "ARRY": ["Testing", 9999, false]
>         }
> }]
> key:[TX_CODE], value:[816060]
> key:[INSU_CODE], value:[0073]
> key:[AMT], value:[100.000000]
> key:[BOOL], value:[True]
> Arry Size:[3]
> key:[ARRY], value:[Testing]
> key:[ARRY], value:[9999.000000]
> key:[ARRY], value:[False]
