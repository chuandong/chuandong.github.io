---
title: rust code
date: 2025-11-29 16:17:10
description: rust
tags: code study
---

#### 字符串

```rust
let str5 = String::from("hello world");
let str6 = str5[0];  /*这代码是错误的,不能使用下表获取字符，rust中不允许*/
println!("str6:{}", str6);
```

#### 元组（复合类型）：

```rust
let tup:(i32, f64, u8) = (500, 6.4, 1);
let tup_0 = tup.0;  /*使用下标访问*/
let tup_1 = tup.1;
let tup_2 = tup.2;
let (x, y, z) = tup; /*解构元组*/
println!("tup tmp value:{}", y);
```

#### 表达式：

```rust
let tmpval = 11;
let perVal = {
    let tmpval = 1;
    tmpval + 1
};
/*注：tmpval + 1 后面是没有“;”的*/

```

#### 函数返回值：

```rust
/*main函数里调用：*/
let iRet = reback_func();
println!("iRet:{iRet}");
//函数1：
fn reback_func()->i32 {
    return -1;
}
/*----------------------------------------*/
let iRet1 = plus_one(2);
println!("iRet1:{iRet1}");
//注：iRet1的值是3
//函数2：
fn plus_one(x:i32) -> i32 {
    x + 1
}
//----------------------------------------
let iRet1 = plus_one(2);
println!("iRet1:{iRet1}");
//注：iRet1的值还是2，不是3
fn plus_one(x:i32) -> i32 {
    x + 1;
    return x;
}

```

#### 变量值的形式：

```rust
fn bool_func() {
    let bool = false;

    let num = if bool {
        5
    } else {
        6
    };
    println!("test bool num:{num}");
    
    //let num2 = bool ? 5: 6;
}
//注：let num2 = bool ? 5: 6;不支持三目运算；
```

####  loop循环函数

```rust
fn loop_func() {
    let mut loop_num = 0;
    let res = loop {
        loop_num += 1;
        if loop_num == 10 {
            break loop_num * 2;
        }
    };
    println!("loop_num:{res}");
}
/*注：break loop_num * 2; 这一行是必须这么写，不能写成：
loop_num * 2;
break;
println!("loop_num:{res}");这res的值会报错；*/
```

#### for循环函数

```rust
fn for_func() {
    let arry = [11, 22, 33, 44, 55];
    for elem in arry.iter() {
        println!("elem:{elem}");
    }
}
//注：使用了iter迭代器
//----------------------------------------
for num in (1..5).rev() {
    println!("num:{num}");
}
```

#### 所有权

```rust
fn input_func() {
    let mut s = String::from("hello rust ");
    s.push_str("I very like you.");
    println!("{}", s);

    let s1 = String::from("hello");
    let s2 = s1;
    println!("{s1}"); /*s1已经废弃，不能打印*/
}
//注：所有权的问题
```

##### 所有权的变换

```rust
fn main(){
    let mut s = String::from("hi");

    let r1 = &s;  
    println!("r1:{r1}");
    let r2 = &s;   
    println!("r2:{r2}");
    let r3 = &mut s; 
    println!("r3:{r3}");
}
//代码等价于：
fn main(){
    let mut s = String::from("hi");
    {
        let r1 = &s;
        println!("r1:{r1}");
    }
    {
        let r2 = &s;
        println!("r2:{r2}");
    }
    let r3 = &mut s;
    println!("r3:{r3}");
}
/*注：r1和r2的作用域随着打印会消失；
如果是：*/
fn main(){
    let mut s = String::from("hi");

    let r1 = &s;  
    let r2 = &s;   
    let r3 = &mut s; 
    println!("r1:{r1}, r2:{r2}，r3:{r3}");
}
/*注：这种在r3这会报错，
原因：
	• 不可变借用承诺：“我不会改数据，所以别人读到的永远是稳定的值。”
	• 可变借用承诺：“我可以随时改数据，没人能同时读。”
	• 如果两者同时存在，读到的值可能在你读的过程中被改掉，这就会引入数据竞争问题（多线程）或逻辑混乱（单线程）。*/

```

#### 切片

```rust
fn main(){
    let s = String::from("hello world");
    let str1 = &s[..5];
    println!("str1:{str1}");
    let str2 = &s[6..];
    println!("str2:{str2}");
    let str3 = &s[..s.len()];
    println!("s:{str3}")
}
```

##### 数组切片

```rust
fn main(){
    let arry = [1, 2, 3, 4, 5, 6];
    println!("arry:{:?}", arry);
    let arry1 = &arry[3..];
    println!("arry1：{:?}", arry1);
}
//注：打印方式不同
```

#### 向量

```rust
fn main() {
    let mut arrys = vec![1, 2, 3];
    arrys.extend([4]);
    println!("{:?}", arrys);
}
```

> [1, 2, 3, 4]

##### 向量与容量

```rust
fn main() {
	let vec1 = vec![[1,2,3], [4,5,6]];
    let vec2 = vec![[1,2], [3,4], [5,6]];
    println!("vec1.len:{}, vec1.capacity:{}", vec1.len(), vec1.capacity());
    println!("vec2.len:{}, vec2.capacity:{}", vec2.len(), vec2.capacity());
}
```

> vec1.len:2, vec1.capacity:2
> vec2.len:3, vec2.capacity:3



#### 结构体

```rust
struct User{
    name:String,
    email:String,
    sign_in_count:i32,
    active:bool,
    age:i32,
}

fn main(){
    let mut user = User {
        name: String::from("aaaaaaa"),
        email: String::from("11111111@qq.com"),
        sign_in_count: 1,
        active: true,
        age: 34
    };

    user.email = String::from("aaaaaaaa");
    println!("email:{}", user.email);
}

```

##### 结构体打印

```rust
#[derive(Debug)]
struct User{
    name:String,
    email:String,
    sign_in_count:i32,
    active:bool,
    age:i32,
}

fn main(){
    let mut user = User {
        name: String::from("aaaaaaa"),
        email: String::from("11111111@qq.com"),
        sign_in_count: 1,
        active: true,
        age: 34
    };

    user.email = String::from("aaaaaaaa");
    println!("email:{:#?}", user);
}
```

##### 结构体方法与函数

```rust
struct Areainfo{
    width: i32,
    height: i32,
}

impl Areainfo{
    fn area(&self)->i32{
        self.width * self.height
    }
}
fn main() {
    let info = Areainfo{width:30, height:40};
    println!("{}", info.area());
}
```

##### 关联函数

```rust
#[derive(Debug)]
struct Areainfo{
    width: i32,
    height: i32,
}
impl Areainfo{
    /*关联函数中，不带self参数*/
    fn square(size:i32)->Areainfo{
        Areainfo{width:size, height:size}
    }
}
fn main() {
    let area3 = Areainfo::square(10);
    println!("{:#?}", area3); /*这种打印需要在结构体上方加#[derive(Debug)]*/
}
```



#### 枚举

```rust
struct Rectangle{
    width: u32,
    height: u32,
}
impl Rectangle{
    fn area(&self)->u32{
        self.width*self.height
    }
}

#[derive(Debug)]
enum Message{
    Quit,
    Move{x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}

struct QuitMessage;
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String);
struct ChangeColorMessage(i32, i32, i32);

impl Message {
    fn call(&self) {
        println!("x:{:?}", self)
    }
}
fn main(){
    let m = Message::Move{x: 1, y: 2};
    m.call();

    let w =  Message::Write(String::from("hello"));
    w.call();

    let s1 = Rectangle{width: 10, height: 20};
    let s = Rectangle::area(&s1);
    println!("Area: {}", s);
}

```

##### 枚举方法

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message{
    fn call(self) {
        println!("self:{:?}", self)
    }
}
fn main() {
    let msg1 = Message::Move { x: 10, y: 20 };
    let msg2 = Message::ChangeColor(255, 0, 0);

    msg1.call();
    msg2.call();

    // 处理带命名字段的枚举变体
    /*if let Message::Move { x, y } = msg1 {
        println!("Moving to position ({}, {})", x, y);
    }*/

    // 处理元组类型的枚举变体
    /*if let Message::ChangeColor(r, g, b) = msg2 {
        println!("Changing color to RGB({}, {}, {})", r, g, b);
    }*/
}
```



#### match控制流

```rust
enum Option<T> {
    Some(T),
    None,
}

fn find_num(nums:&[i32], target:i32) -> std::option::Option<usize> {
    for (i, &num) in nums.iter().enumerate(){
        if num == target {
            return Some(i);
        }
    }
    None
}
fn main() {
    let nums = [1, 2, 3, 4, 5];
    let res = find_num(&nums, 33);
    match res {
        Some(idx) => println!("idx:{idx}"),
        None => println!("not found."),
    }
}

```

##### match复杂用法

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    California,
}
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("state:{:?}", state);
            25
        },
    }
}

fn main() {
    let coin = value_in_cents(Coin::Quarter(UsState::Alaska));
    println!("coin = {:?}", coin);

    let coin1 = value_in_cents(Coin::Dime);
    println!("coin1 = {:?}", coin1);
}

```

##### match控制流

```rust
#[derive(Debug)]
enum Option<T>{
    None,
    Some(T),
}

fn plus_one(x:Option<i32>) -> Option<i32> {
    match x{
        Option:: None => Option:: None,
        Option:: Some(i) => Option:: Some(i+1),
    }
}

fn main() {
    let five = plus_one(Option::Some(5));
    println!("five:{:?}", five);
    let none = plus_one(Option::None);
    println!("none:{:?}", none);
}

```

#### file操作

```rust
use std::fs::File;
use std::io::ErrorKind;
fn main() {
    let f = File::open("test_file.txt");
    let f = match f {
        Ok(file) => file,
        Err(error)=> match error.kind() {
            ErrorKind::NotFound => match File::create("test_file.txt") {
                Ok(fc) => fc,
                Err(error) => panic!("Problem creating the file: {:?}", error),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
    println!("{:?}", f);
}

```

#### 泛型

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other:Point<V, W>) -> Point<T, W> {
        Point{
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point{x: 5, y: 10.4};
    let p2 = Point{x:"hello", y: "c"};
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y={}", p3.x, p3.y);
}
```

##### 泛型的使用

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    // 1) 索引：list[0] 得到第一个元素（要求切片非空；否则 panic）
    let mut largest = list[0];

    // 2) list.iter() 迭代借用到的 &T；for &item 把 &T 解构成 T（要求 T: Copy）
    for &item in list.iter() { //等价写法 for item in list.iter().copied()
        // 3) 使用 PartialOrd 的 > 运算符比较
        if item > largest {
            // 4) 赋值：把更大的 T 拷贝到变量（不是移动，因为 T: Copy）
            largest = item;
        }
    }
    largest // 5) 返回：把 T 再拷贝一份作为返回值
}
fn main() {
    let num_list = vec![34, 50, 25, 100, 65];
    let result = largest(&num_list);
    println!("The largest number is {}", result);

    let str_list = vec!["apple", "banana", "orange", "pear"];
    let result = largest(&str_list);
    println!("The largest string is {}", result);
}
```

##### 泛型参数不明确,需直接指出

```rust
fn do_something<T:Default>() -> T {
    let val = T::default();
    val
}

fn main() {
    /*i32直接指出*/
    let something:i32 = do_something();
    println!("something: {}", something);

    /*另种指出方式f64*/
    let something2 = do_something::<f64>();
    println!("something2: {}", something2);
}
```

##### 枚举与泛型

```rust
enum Repeat<T, U> {
    Continue(T),
    Result(U),
    Done,
}

fn find_div_5(number: i8)->Repeat<i8, i8> {
    if (number % 5 == 0) {
        println!("Found Number:{}", number);
        Repeat::Done;
    } else {
        Repeat::Continue(number + 1)
    }
}

fn main() {
    let mut num = 1;
    loop{
        if let Repeat::Continue(recommend) = find_div_5(num){
            num = recommend;
        } else {
            break;
        }
    }
    println!("num{}", num);
}
```



#### trait

```rust
trait Person {
    fn get_name(&self) -> String;
    fn say(&self) {
        println!("say hello:{}", self.get_name());
    }
}

struct Player {
    name: String,
}

impl Player {
    fn new(name: &str) -> Self {
        Player {
            name: String::from(name)
        }
    }
}

impl Person for Player {
    fn get_name(&self) -> String {
        self.name.clone()
    }

    fn say(&self) {
        println!("Person for Player:{}", self.get_name());
    }
}

struct Newborn;

impl Person for Newborn {
    fn get_name(&self) -> String {
        "Newborn".to_string()
    }
}

struct Poet{
    name: String,
}

impl Person for Poet {
    fn get_name(&self) -> String {
        self.get_name().clone()
    }
}

struct Game<T: Person> {
    person: T,
}

impl<T: Person> Game<T> {
    fn new(person: T) -> Self {
        Game {
            person
        }
    }
    fn play(&self) {
        println!("{}playing game.", self.person.get_name());
    }
}

fn compare_and_print<T>(a: &T, b: &T)
where
    T: std::fmt::Display + std::cmp::PartialOrd, {
    if a > b {
        println!("lagre {}", a);
    } else {
        println!("less {}", b);
    }
}

fn main() {
    let player = Player::new("dongdong");
    println!("{}", player.get_name());
    let game = Game::new(player);
    game.play();

    compare_and_print(&3, &4);
}

```



##### 
