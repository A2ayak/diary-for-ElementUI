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

##### 3、组件动态增删 + 表单校验

```javascript
this.list.push(Obj)
this.list.splice(index, 1)
:prop="`${循环的list名}[${index}].‘校验属性名’`" // 如:prop="`${list}[${index}].name`"

// 以下适用于
// <el-form :model="formData"> // 此处必须有model，否则无法触发form的validate方法，model接收Object，所以必须用对象包裹数组list
// 	 <el-form-item
// 	 	  v-for="（item, index） in formData.list"
//		  :prop="`list[${index}.name]`"
//		  :rules="[{ validator: validateName(item.name, etc.), trigger: 'change' }]" // validateName为method，可用于部分字段校验
//    >
// 
validateName(name, ...params) {
  return (rule, value, callback) => {
    const regExp = /^xxxxx$/
    if (xx) {
      return callback()
    } else if (regExp.test(name)) {
      callback(new Error('name不符合规则哦'))
    } else {
      return callback() // 最后必须返回，否则点击validate永远不会success
    }
  }
}

```

##### 4、sortable.js对表格项进行操作时必须有row-key

```javascript
:row-key="row => row.id"

// 行拖拽
rowDrop() {
  // 此时找到的额元素是要拖拽元素的父容器
  const tbody = this.$refs['xxxx'].$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
  Sortable.create(tbody, {
    animation： 150,
    handle: 'drag-btn',
    draggable: '.xxx .el-table__row', // 指定父元素下可拖拽的元素
    onEnd: evt => {
    	if (evt.oldIndex !== evt.newIndex) {
    		this.$nextTick(() => {
          // 务必设置row-key，否则排序会出问题
          const targetRow = this.dataList.splice(evt.oldIndex, 1)[0]
          this.dataList.splice(evt.newIndex, 0, targetRow)
        })
		  }
  	}
  })
}
```

