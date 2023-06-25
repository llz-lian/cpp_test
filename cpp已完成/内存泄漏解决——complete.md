# 程序小——上大的重的
1. linux下可以用valgrind中的memcheck。
2. windows下VS直接调试堆检测，看快照。
# 程序特别大——
1. 考虑一下使用了RAII的智能指针
2. 觉得智能指针慢：gcc:AddressSanitizer（只能检查出简单泄漏）
3. 找到能分配的点，打印一下stack（DEBUG模式下别，release下看，或者换个机器看看还泄不泄漏（要重新编译一下））
4. 写的不注意就没救了，c++是这样的，不爽不要用。