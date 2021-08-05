# Diary-for-ElementUI
:fire: ElemetnUI自救指南，正经人谁写日记呢？

---

##### 1、el-tree父子结点分离，勾父选全子，勾子选父

```javascript
:check-strictly="true" // 分离父子关系
@check="checkChange" // 不可用check-change,因为setCheckedKeys会触发check-change

checkChange(node) {
  const currentNode = this.$refs['roleTree'].getNode(node.id) // 获取当前节点(ele构建的节点)
  const keys = this.$refs['roleTree'].getCheckedKeys() // 获取已勾选节点的key值
  if (currentNode.checked) {
      // 递归找父子节点
      this.selectChildNodes(currentNode, keys)
      this.selectFatherNodes(currentNode.parent, keys)
      this.$refs['roleTree'].setCheckedKeys(keys) // 将所有keys数组的节点全选中
   }
}
```
