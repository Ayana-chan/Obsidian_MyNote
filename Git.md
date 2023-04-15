对于springboot的多模块项目（所有模块被聚合于上级），可以这样写`.gitignore`：

```gitignore
**/mvnw
**/mvnw.cmd

**/.mvn
**/target/

.idea

**/.gitignore
```

## 获取新的分支信息

```bash
git remote update origin --prune
```

























