# dynamic_cast 

# const_cast

# static_cast
```
int main()
{
    void* a = new int(5);
    //int* n = a; error
    int* ptr = static_cast<int*>(a);
    void* b = static_cast<void*>(ptr);
    if (a == b)
        std::cout << "a is b" << std::endl;
    return 0;
}
output:
a is b
```
# reinterpret_cast