# Assignment Note

> Stanford CS106L W24 Assignment

## Assign 3

设计自己的类

`getter()`函数都应该设置为const

## Assign 5

运算符+特殊成员函数重载

`bool User::operator<(const User& rhs) const;`, 重载`<`, 最后的const很重要.

`User& User::operator=(const User& user);`, 重载copy assignment, 需要先检查`this == &user`, 然后先释放掉之前的资源`_friends`, 再给`_name`, `_size`, `_capacity`赋新的值, 这前后顺序不能乱.

## Assign 6

`std::optional`

c++20的新特性 (课上好像没讲), 可以明确表达“无值”语义, 直接通过类型系统表明返回值可能为空, 使代码可读性更强.

使用场景, 作业中跟第2点差不多 (GPT生成)

1. 函数可能返回无效结果

   ```cpp
   std::optional<int> safeDivide(int a, int b) {
       if (b == 0) return std::nullopt;
       return a / b;
   }
   ```

2. 查找可能失败的操作

   ```cpp
   std::optional<std::string> findNameById(int id) {
       auto it = database.find(id);
       if (it == database.end()) return std::nullopt;
       return it->second;
   }
   ```

3. 延迟初始化

   ```cpp
   class Config {
       std::optional<std::string> cachedData;  // 延迟加载
   public:
       void load() {
           cachedData = readFromFile();
       }
   };
   ```

## Assign 7

RAII和智能指针

`unique_ptr& operator=(unique_ptr&& other);` , 忘记先释放当前的资源了, 踩了个坑.

重载运算符`*`时如果是空指针, GPT建议直接抛出异常, 好像也没有更好的方法.

逆序遍历`vector`可以用`rbegin()`和`rend()`.