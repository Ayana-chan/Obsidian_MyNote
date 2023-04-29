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























