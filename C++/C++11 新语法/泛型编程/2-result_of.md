# 结果类型推导

- [link](https://blog.csdn.net/o0onlylove0o/article/details/130276323?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170849865316800226598934%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=170849865316800226598934&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-130276323-null-null.142^v99^pc_search_result_base5&utm_term=std%3A%3Aresult_of%20%E7%94%A8%E6%B3%95&spm=1018.2226.3001.4187)

- 在C++中，有时我们需要获取函数或可调用对象的返回值类型，以便进行后续的操作，在[泛型编程](https://so.csdn.net/so/search?q=泛型编程&spm=1001.2101.3001.7020)中很常用，特别是当不同的参数集的结果类型不同时。在早期的C++版本中，我们需要手动推导函数返回值类型，这个过程非常复杂，也容易出错。为了解决这个问题，C++11引入了std::result_of

- `std::result_of`是一个[函数类型](https://so.csdn.net/so/search?q=函数类型&spm=1001.2101.3001.7020)萃取器（function type traits），它可以推导函数类型的返回值类型，它定义在头文件`<type_traits>`中。`std::result_of`模板类需要两个模板参数：第一个是函数类型，第二个是函数的参数类型。它的定义如下：

  ```c++
  // 在模板参数中，F必须是可调用类型、对函数的引用或对可调用类型的引用，Args代表函数参数类型
  template <typename F, typename... Args>
  class result_of<F(Args...)>;
  ```

  

  ```c++
    #include <iostream>
    using namespace std;
  
    #include <iostream>
    #include <type_traits>
  
  // 测试返回值结果推到
  void test_result()
  {
    // 使用 std::result_of 获取函数对象 Add 调用后的返回类型
    // typename 关键字用于告诉编译器，后面的名称是一个类型，而不是一个变量或静态成员
    // using 关键字用于定义类型别名
    using result_type = typename std::result_of<Add(int, int)>::type;
  
    // 打印返回类型
    std::cout << "Result type of Add(int, int): " << typeid(result_type).name() << std::endl;
  }
  
  int main()
  {
    test_result();
    return 0;
  }
  
  // 上述用法在c++17 中已经弃用
  // 上述用法在c++17 中已经弃用
  // 上述用法在c++17 中已经弃用
  // 上述用法在c++17 中已经弃用
  // 上述用法在c++17 中已经弃用
  // 上述用法在c++17 中已经弃用
  
  
  #include <iostream>
  #include <functional>
  #include <type_traits>
  #include <typeinfo>
  
  void add(int x, double y) { 
      // return x + static_cast<int>(y); 
  }
  
  // 测试返回值结果推导
  void test_result() {
      // 使用 std::invoke_result 获取函数对象 add 调用后的返回类型
    	// decltype 用于推导表达式的类型，它会返回该表达式的类型。例如：
        // int x = 10;
  			//decltype(x) y = 20; // y 的类型是 int
      // decltype(add) 获取的是 add 函数的类型，然后 std::invoke_result 会使用这个类型来推导 add(int, double) 调用的返回类型
      using result_type = std::invoke_result<decltype(add), int, double>::type;
  
      // 打印返回类型
      std::cout << "Result type of add(int, double): " << typeid(result_type).name() << std::endl;
  }
  
  int main() {
      test_result();
      return 0;
  }
  
  ```
  
  