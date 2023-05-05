# 环境

安装环境时clang可能失败，解决（似乎不是最好的）：
[在Ubuntu上配置clang-14的环境\_5436649486的博客-CSDN博客](https://blog.csdn.net/weixin_50749380/article/details/128319851)

不知道为什么clion用ssh连接时会在获取编译器信息时报身份验证失败，换成账号密码登录就行。不用root用户也会出问题。

clion要手动选择使用clang编译器。
# Formatting

Your code must follow the [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html). We use [Clang](https://clang.llvm.org/) to automatically check the quality of your source code. Your project grade will be **zero** if your submission fails any of these checks.

Execute the following commands to check your syntax. The `format` target will automatically correct your code. The `check-lint` and `check-clang-tidy` targets will print errors that you must manually fix to conform to our style guide.

```bash
make format
make check-lint
make check-clang-tidy-p0
```
# 内容
## Copy-On-Write Trie

In a copy-on-write trie, operations do not directly modify the nodes of the original trie. Instead, new nodes are created for modified data, and a new root is returned for the newly-modified trie. Copy-on-write enables us to access the trie after each operation at any time with minimum overhead.

写之后，把所有直接间接受影响的结点都转移走，这样就不会干扰正在进行的读了。

没有值的叶子结点是没有用的，要删掉（可能删了之后又有无用结点，要都删）

Your trie should support 3 operations:
- `Get(key)`: Get the value corresponding to the key.
- `Put(key, value)`: Set the corresponding value to the key. Overwrite existing value if the key already exists. Note that the type of the value might be **non-copyable** (i.e., `std::unique_ptr<int>`). This method returns a new trie.
- `Delete(key)`: Delete the value for the key. This method returns a new trie.

To reuse an existing node in the new trie, you can copy `std::shared_ptr<TrieNode>`: copying a shared pointer doesn’t copy the underlying data.You should not manually allocate memory by using `new` and `delete` in this project.
全部使用`std::shared_ptr`似乎就相当于自带gc？

>可以用vector

每次修改操作必然会产生新的root。





















