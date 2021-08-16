# Diary-for-ElementUI
:fire: ElemetnUI自救指南，正经人谁写日记呢？

---

##### 1、el-tree父子结点分离，勾父选全子，勾子选父

```javascript
:check-strictly="true" // 分离父子关系
@check="checkChange" // 不可用check-change,因为setCheckedKeys会触发check-change

checkChange(node) {
  const currentNode = this.$refs['tree'].getNode(node.id) // 获取当前节点(ele构建的节点)
  const keys = this.$refs['tree'].getCheckedKeys() // 获取已勾选节点的key值
  if (currentNode.checked) {
      // 递归找父子节点
      this.selectChildNodes(currentNode, keys)
      this.selectFatherNodes(currentNode.parent, keys)
      this.$refs['tree'].setCheckedKeys(keys) // 将所有keys数组的节点全选中
   }
}
```



##### 2、el-table树形结构，含不可选项的全选和取消全选（全选按钮样式需要额外写）

```javascript
@select-all="selectAll(treeTableData)" 

selectAll(selection, first) {
  if (!first) {
    this.isAllSelect = !this.isAllSelect  // isAllSelect初始为false
  }
  selection.map(el => {
    this.toggleSelection(el, this.isAllSelect)
    if (el.children) {
      el.children.map(c => {
        this.toggleSelection(c, this.isAllSelect)
      })
      if (el.children.length > 0) {
        this.selectAll(el.children, true)
      }
    }
  })
},
toggleSelection(row, select) {
  if (select) {
    if (row 条件值，具体与不可选项:selectable对应) {
      this.$refs['treeTable'].toggleRowSelection(row, select)
    }
  } else {
    this.$refs['treeTable'].clearSelection()
  }
}
```