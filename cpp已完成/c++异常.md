# 1. 函数调用和返回
函数调用时，在当前调用栈中按照一定结构保存参数。
保存的参数有：
1. 当前临时变量
2. 当前栈指针EBP
3. 返回地址
# 2. 栈回退（stack unwind）
为了在函数退出时释放资源，需要额外引入数据（栈回退表）。
在一个函数中，记录当前临时对象的个数。
同时存在一个栈回退表，记录当前对象的析构和下一个应该析构的表项。
```c++
/*
栈回退表
idx   next    destroyH     obj
0      -1      null         -
1       0      ~MyClass()  obj1
2       1      ~MyClass()  obj2
3       2      ~MyClass()  obj3
4       2      ~MyClass()  obj4
*/
void func()
{
    // nStep = 0;
    MyClass obj1,obj2;
    // construct obj1 obj2
    // nStep = 1;nStep = 2;
    {
        MyClass obj3;
        // nStep = 3;
        // ....


        // destroy obj3
        // nStep = 2;
    }
    MyClass obj4;
    // construct obj4
    // nStep = 4;

    // destory obj4
    // nStep = 2;

    // destory obj2
    // nStep = 1;
    // destory obj1
    // nStep = 0;
}
```
# 3. 异常捕获
为了捕获异常，需要再额外记录数据。
1. 当前try的位置（try表）
    - 通过一个表进行记录，使用nStep记录try的起始位置和结束位置，同时加入当前catch列表的指针。
2. 记录catch接受的异常类型
    - 记录每个catch块的entry
    - 记录接受异常类型
```c++
/*
tryBlock:
begin end catchBlocks
1      4      |
              |
      catchBlock
entry1      catchType1
entry2      catchtype2
*/
void func()
{
    // nStep = 0
    try{
        // try block
        // nStep = 1

        MyClass obj1,obj2;
        // construct obj1 obj2
        // nStep = 2
        // nStep = 3

        // destory obj1 obj2
        // nStep = 2
        // nStep = 1
    }
    // try block end
    // nStep = 4
    catch(const char *)// <-----entry
    {

    }
    catch(exception &)
    {

    }
    // ...
}
```
当异常发生时，检查是否在try块内，如果在try块内，进行catch检查。
如果catch不匹配或者异常不在tyr块内，将异常返回给调用者继续处理。
如果异常到达处理链的终点，发生异常未捕获，结束进程。

# 4. 异常抛出
通过一个函数进行抛出异常，同时执行异常捕获流程。






