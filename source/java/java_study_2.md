# java code

## 1. 分析java的语法代码

```java
public class ArrayClass {
    public static void main(String[] args) {
        int[] arry1 = new int[2];
        arry1[0] = 100;
        arry1[1] = 111;

        System.out.println(arry1);
        int[] arry2 = new int[0];
        System.out.println(arry2);
        arry2 = arry1;
        System.out.println(arry2[1]);
        System.out.println(arry2.length);
        int[] arry3 = new int[0];
        arry3 = arry2;
        System.out.println("1:" +arry3[1] + "length:" + arry3.length);
    }
}
```

