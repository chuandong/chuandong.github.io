### yaml

#### 解析键值对的文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <malloc.h>
#include <yaml.h>

const char* get_event_name(yaml_event_type_t type)
{
    switch (type)
    {
    case YAML_STREAM_START_EVENT: return "STREAM_START";
    case YAML_STREAM_END_EVENT: return "STREAM_END";
    case YAML_DOCUMENT_START_EVENT: return "DOCUMENT_START";
    case YAML_DOCUMENT_END_EVENT: return "DOCUMENT_END";
    case YAML_ALIAS_EVENT: return "ALIAS";
    case YAML_SCALAR_EVENT: return "SCALAR";
    case YAML_SEQUENCE_START_EVENT: return "SEQUENCE_START";
    case YAML_SEQUENCE_END_EVENT: return "SEQUENCE_END";
    case YAML_MAPPING_START_EVENT: return "MAPPING_START";
    case YAML_MAPPING_END_EVENT: return "MAPPING_END";
    default: return "UNKNOWN";
    }
}

int main(int argc, char *argv[])
{
    FILE *fp = fopen("yaml_test.cfg", "r");
    if (fp == NULL)
    {
        printf("open file failed\n");
        return -1;
    }

    yaml_parser_t parser;
    yaml_parser_initialize(&parser);
    yaml_parser_set_input_file(&parser, fp);

    yaml_event_t event;
    int event_count = 0;

    while (yaml_parser_parse(&parser, &event))
    {
        event_count++;
        printf("[%2d] %s", event_count, get_event_name(event.type));

        if (event.type == YAML_SCALAR_EVENT)
        {
			/*键值对模式,第一个是key,第二个是value*/
            printf(": '%s'", (char *)event.data.scalar.value);
        }

        printf("\n");

        if (event.type == YAML_STREAM_END_EVENT)
        {
            yaml_event_delete(&event);
            break;
        }
        yaml_event_delete(&event);
    }

    fclose(fp);
    yaml_parser_delete(&parser);
    return 0;
}
```

> 编译的时候: gcc -o test_yaml test_yaml.c -lyaml

## 关键点总结：
事件类型 判断依据 示例 

SCALAR 普通值 hello , 30 MAPPING_START : + 缩进增加 key: 后跟缩进内容 

SEQUENCE_START 行首 - - item

MAPPING_END 缩进减少 回到上一级缩进 

SEQUENCE_END 缩进减少 回到上一级缩进