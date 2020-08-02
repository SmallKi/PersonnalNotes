### lambda表达式

形式：`[capturelist](parameterlist)->returntype{function body}`

其中`parameterlist` 和` returntype`可以省略，其他不可以省略

```c++
auto f = []()->int {return 43; };
auto f = []{return 43;};
//上面两个等价
```

```c++
//capturelist 代表的是捕获的值
//匿名函数默认不能使用所在函数中的局部变量，必须在capturelist中指定才可以
	string sz = "helo";
	auto res = find_if(test.begin(), test.end(), [sz](const string& str) {
		return str.size() - sz.size(); });
//如果捕获列表中不添加sz 会报错
```

**注意：lambda函数中不能使用默认参数，即形参数目和实参数目一定是相同的**



lambda里的capturelist

~~~~c++
//1.使用值传递
[name](){};
//2.使用引用传递
[&name](){};
//3.使用默认传递
[&](){};   //默认将局部变量全部以引用传递到匿名函数中
[=](){};   //默认将局部变量全部以值传递方式传到匿名函数中
//混合使用
[&, name](){}; //默认的和后面的必须是不同形式，比如前面是默认以引用方式传递，后面就得用值传递

//改变局部变量的值，只能通过引用传递（那个变量不能是const）
[&name]{return name++;};   //name的值加1， name不能使const类型

//值传递进来的副本是在创建时拷贝的，默认是不能修改的，如果想要修改，需要加上mutable关键词
[name]()mutable{return name++;};  //name的值加1  不能省略();
vector<string> test = { "dfjasf","df","dfkja","ds","dsaaa","df","dfkja","dfja","df" };
auto a = [test]() mutable{test.push_back("helloo"); };
~~~~

##### lambda中的返回类型

如果只有一条return语句，返回类型是可以省略的，会自动识别。

如果有多条语句，匿名函数会将返回类型默认为void，即不能有返回值，此时必须显式地指明返回类型



### bind函数解决参数适配

bind函数在头文件\<functional\>中

bind(可调用对象, arg_list);

bind会将arg_list参数列表传递到可调用对象中去。同时，可以用占位符_n来指定输入参数的位置

```c++
auto b = [](int a, int b)mutable -> int {return a+b; };
auto newb = bind(b, _1, 10);
cout << newb(10); //传递给newb的参数会放到_1位置上，同理还会有_2,_3。。。等占位符，分别指代第几个参数
```

占位符在一个特殊的命名空间中：`std::placeholders` 使用前，必须使用命名空间前置

默认情况下，不是占位符的参数会执行拷贝动作，如果想要用引用传递，则需要调用`ref`和`cref`函数，这两个函数返回对象的引用

```c++
auto b = [](int a, int b)mutable -> int {return a+b; };
int c = 12;
auto newb = bind(b, _1, ref(c));
cout << newb(10); //传递给newb的参数会放到_1位置上，同理还会有_2,_3。。。等占位符，分别指代第几个参数
```





## 泛型算法

#### 算法不会执行容器操作，只是对容器进行遍历，因此，插入、删除或者是改变容器大小都是算法不能完成的，必须借助算法中间的其他操作

1. find(T begin, T end, val);

   在begin和end之间查找val值，若存在，返回所在的迭代器；不存在则返回end所在的迭代器

   ```c++
   vector<int> a = { 3,4,567,1,4654,134,7,9,2341,34 };
   auto result = find(a.cbegin(), a.cend(), 567);
   cout <<( result == a.cend() ? "没有" : "找到了" )<< endl;
   ```

   

2. count(T begin, T end, val);

   统计val在范围内出现的次数，返回值是int类型

   ```c++
   vector<int> a = { 3,4,567,1,4654,134,7,9,2341,34,7};
   auto result = count(a.begin(), a.end(), 7);
   cout << result;  //2
   ```

3. count_if(T begin, T end, func(t));

   对范围内的成员执行func操作，记录结果为true的个数

   ```c++
   vector<string> test = { "dfjasf","df","dfkja","ds","dsaaa","df","dfkja","dfja","df" };
   auto b = count_if(test.begin(), test.end(), [](const string a) {return a == "df"; });
   cout << b;  //3
   ```

   

4. accumulate(T begin, T end, val);   `//在头文件numeric中`

   以val为初始值，累加范围内的值，返回的类型是**val的类型**

   ```c++
   //accumulate 以第三个参数作为＋号运算符的类型，只要能进行+运算的，都可以使用这个算法
   vector<string> b = { "hello","jhaja","haha" };
   auto result = accumulate(b.begin(), b.end(), string(""));  
   //auto result = accumulate(b.begin(), b.end(), "");  这是错的，const char* 类型没有重载+
   cout << result;  //hellojhajahaha
   ```

5. equal(T a_begin, T a_end, T b_beign);

   比较a和b序列是否相同，返回bool类型，这个算法假定b至少与a一样长！

   ```c++
   string a = "hello world";
   string b = "hello world a";
   bool res = equal(a.begin(), a.end(), b.begin());
   cout << res; //1
   ```

   **像这样只支持单一迭代器的第二个序列算法，都假定第二个序列至少与第一个序列一样长**

6. fill(T begin, T end, val);

   将范围内的元素都替换为val

   ```c++
   vector<double> a1 = { 3,4,567,1,4654,134,7,9,2341,34,7};
   	fill(a1.begin(), a1.end(), 0);
   	for (auto i : a1)
   		cout << i << "\t";   // 0 0 0 0  0 00 0 
   ```

   

7. fill_n(T begin, int n , val);

   从begin位置开始，将后面的n 个位置的值替换为val

   ```c++
   //注意，应保证begin+n在有效范围内，不然会报错
   vector<int> temp; //空vector
   fill_n(temp.begin, 10, 0);   //错误，因为temp是个空的容器，算法无法向空容器中写入值
   
   //正确做法，使用back_inserter
   fill_n(back_inserter(temp), 10, 0);
   ```

8. copy(T s_begin, T s_end, T des_begin);

   将源范围的值复制到dest中去，假定des_begin代表的容器至少有s容器的大小，拷贝后返回des的尾后迭代器（拷贝的最后一个值的下一个）

   

9. replace(T begin, T end,  val, rep);

   将范围内的val值改为rep值，改变的是原数组，如果不想改变原数组，可以使用replace_copy;

   ```c++
   
   ```

10. replace_copy(T begin, T end, T d_begin , val , rep);

   会将替换后的容器返回给d容器，不回影响原来的容器

11. sort(T begin, T end [, fun_c(t1, t2)]);

    将范围内的数据进行排序，fun_c可以传递一个二参数的返回bool类型的函数

    内部实现是快速排序，会影响之前的顺序，不稳定

    ```c++
    bool compare1(const int a, const int b)
    {
    	return b < a;
    }
    
    	vector<int> test = { 123,3,12,99,88,31,331,65 };
    	sort(test.begin(),test.end());
    	for (auto i : test) cout << i << "\t";
    	cout << endl;
    	sort(test.begin(), test.end(), compare1);
    	for (auto i : test) cout << i << "\t";
    ```

11. stable_sort(T begin, T end [,fun_c]);

    也是排序，内部实现是归并排序，不会影响原来的先后顺序

12. partition(T begin, T end, func(t));

    接受一个bool返回值得函数，一元谓词，将返回true得值排在范围得前面，false的排在容器的后面，返回值是最后一个true的尾后元素。

    ```c++
    	vector<string>test = { "helpo", "dfkjfds", "13","df","dfajkl","dfff" };
    	auto a = partition(test.begin(), test.end(), more_five);
    	cout << "超过五个的单词有" << a - test.begin() << "个" <<endl;
    	for (auto a : test)
    		cout << a << "t";
    ```

13. stable_partition(T begin, T end, func(t));

    稳定的partition，不会改变原有的顺序

    

14. unique(T begin, T end);

    将重复的元素分离开，范围前面都是不重复的，原理是将后面不重复的调到前面去，返回值是不重复的元素的尾后元素。之后可以通过容器的erase方法进行删除

    ```c++
    	vector<string>test = { "helpo", "dfkjfds","df", "13","df","dfajkl","dfff" ,"helpo", "df"};
    	sort(test.begin(), test.end());
    	for (auto a : test) cout << a << "\t";
    	cout << endl;
    	auto del = unique(test.begin(), test.end());
    	for (auto a : test) cout << a << "\t";
    	cout << endl;
    	test.erase(del, test.end());
    	for (auto a : test) cout << a << "\t";
    	cout << endl;
    ```

16. find_if(T begin, T end,  func(t));

    在范围内查找第一个使func为不为0的值，返回该值的迭代器，否则返回尾后迭代器。

    ```c++
    bool myfunc(const string& a)
    {
    	return a == "dfdsfasf";
    }
    
    vector<string>test = { "helpo", "dfkjfds","df", "13","df","dfajkl","dfff" ,"helpo", "df"};
    	auto res  = find_if(test.begin(), test.end(), myfunc);
    	if (res != test.end()) cout << "存在这个书";
    	else cout << "不存在";
    ```

    find_if_not  查找第一个使函数为假的

    

17. for_each(T begin, T end, func(t));

    对范围内的每一个成员调用func函数

    ```c++
    vector<string>test = { "helpo", "dfkjfds","df", "13","df","dfajkl","dfff" ,"helpo", "df"};
    	for_each(test.begin(), test.end(), [](string& a) {
    		a = "改变";
    		});
    	for (auto a : test)
    		cout << a << "\t";
    ```

18. transform(T s_begin, T s_end, T d_begin, func(t));

    对输入范围进行func操作，将返回值复制到d容器

    ```c++
    vector<string> test = { "dfjasf","df","dfkja","ds","dsaaa","df","dfkja","dfja","df" };
    transform(test.cbegin(), test.cend(), test.begin(), [](const string& a) {return "添加" + a; });
    for (auto a : test)
    	cout << a << "\t";
    ```

19. 二分搜索

    二分搜索需要使容器首先变得有序

    lower_bound(beg, end , val);  或者 lower_bound(beg, end, val, comp);

    查找第一个使得值 = val 的成员，返回迭代器，否则，返回end();

    upper_bound(beig, end, val); 或者 upper_bound(beg,end,val, comp);

    返回一个迭代器，表示第一个大于val的元素，如果 不存在，返回end();

    equal_range(beg, end, val);   equal_range(beg, end, val, comp);

    返回一个pair，pair.first代表lower_bound的迭代器，Pair.second代表upper_bound返回的迭代器

