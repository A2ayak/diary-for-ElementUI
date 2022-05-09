# Diary-for-ElementUI
:fire: ElemetnUI自救指南，正经人谁写日记呢？

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu)  [![LICENSE](https://img.shields.io/badge/license-Anti%20996-blue.svg)](https://github.com/996icu/996.ICU/blob/master/LICENSE)

---

#### 1、el-tree父子结点分离，勾父选全子，勾子选父

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



#### 2、el-table树形结构，含不可选项的全选和取消全选（全选按钮样式需要额外写）

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

#### 3、组件动态增删 + 表单校验

```javascript
this.list.push(Obj)
this.list.splice(index, 1)
:prop="`${循环的list名}[${index}].‘校验属性名’`" // 如:prop="`${list}[${index}].name`"

// 此处必须有model，否则无法触发form的validate方法，model接收Object，所以必须用对象包裹数组list
<el-form :model="formData">
  <el-form-item
    v-for="（item, index） in formData.list"
    :prop="`list[${index}.name]`"
    // validateName为method，可用于部分字段校验
    :rules="[{ validator: validateName(item.name, etc.), trigger: 'change' }]"
  >
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

#### 4、sortable.js对表格项进行操作时必须有row-key

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

#### 5、el-table表格内为树形数据时，展开深度较深时文字会被indent推到右侧隐藏掉，且不会触发横向滚动轴。此时可以设置合理的indent，或者：

```javascript
<el-table>
  ...
  @expand-change="expandChange"
  <el-table-column
    :width="labelWidth" // 动态调整label的宽度触发横向滚动轴
  />
</el-table>

expandChange(row) {
  const nodeLevel = row.nodeLevel // nodeLevel一般后端都会返回，或者自己找
  this.labelWidth = nodeLevel >= 10 ? (500 + 20 * (nodeLevel - 10)) : 500
}
```

#### 6、（vxe-table + sortable.js）虚拟列表树形表格拖拽，与elementUI-table不同点为数据的index

```javascript
// 行拖拽
rowDrop() {
  if (this.rowDropInstance) {
    this.rowDropInstance.destory()
    this.rowDropInstance = null
  }
  // 找拖拽元素父容器
  const tbody = this.$refs['xxxTable'].$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
  // 递归遍历把树形数据弄扁平
  const flatTree = (flatTableData, flatArr, expandRowKeys) => {
    flatTableData.map(item => {
      flatArr.push(item)
      if (item.children?.length && expandRowKeys.includes(item.id)) {
        flatTree(item.children, flatArr, expandRowKeys)
      }
    })
	}
  this.rowDropInstance = Sortable.create(tbody, {
    animation: 150,
    handle: '.drag-btn' // 可拖拽元素的class
    draggable: '.xxx-table .vxe-body--row' // 指定父元素下可被拖拽子元素
    onStart: ({ oldIndex }) => {
    	const xxxTable = this.$refs['xxxTable']
    	xxxTable.setCurrentRow(null) // 取消已选中行的高亮
    	this.flatArr = []
    	const expandRowKeys = xxxTable.treeConfig.expandRowKeys // vxe-table方法
      const originExpandRowKeys = deepCopy(expandRowKeys) // 保存一份初始展开节点id
      flatTree(treeData, this.flatArr, expandRowKeys)
    	const sourceObj = this.flatArr[oldIndex]
      this.flatArr.filter(item => item.parentId === sourceObj.parentId).map(row =>{
        xxxTable.setTreeExpansion(row, false) // 折叠非同父级节点，注意此时被拖动节点的oldIndex和被替换节点的newIndex会发生改变
        const index = originExpandRowKeys.findIndex(id => id === row.id) // 删除原始的已展开树节点中被折叠的节点id
        if (index !== -1) originExpandKeys.splice(index, 1)
      })
    	this.secondFlatArr = []
    	flatTree(treeData, this.secondFlatArr, originExpandKeys)
  	},
    // 在拖动时判断是否可以放入，非同父节点不允许放入
    onMove: ({ drager, related }) => {
      // 层级比较，通过dom节点的classList转数组，再找层级和ID。 classList中有自定义的class，包含row的parentId/id/nodeLevel
      // tableRowClassName({ row }) { return `row_parent_id_${row.parentId} row_id_${row.id} row_level_${row.nodeLevel}` }
      const sourceClassList = dragged.classList.value.split(' ')
      const targetClassList = related.classList.value.split(' ')
      const sourceLevel = sourceClassList.find(item => item.indexOf('row_level') !== -1).split('_')[2]
      const targetLevel = targetClassList.find(item => item.indexOf('row_level') !== -1).split('_')[2]
      // 父节点id比较
      const sourceParentId = sourceClassList.find(item =>item.indexOf('row_parent_id') !== -1).split('_')[3]
      const targetParentId = targetClassList.find(item =>item.indexOf('row_parent_id') !== -1).split('_')[3]
      if (sourceLevel !== targetLevel || sourceParentId !== targetParentId) return false
    },
    onEnd: ({ oldIndex, newIndex }) => {
      // 此处有坑：被拖动的节点（sourceObj)的index为未折叠（onStart时已折叠非同父级节点）之前的index，
      //         替换节点（targetObj）的index为折叠以后的index
      const sourceObj = this.flatArr[oldIndex]
      const targetObj = this.secondFlatArr(newIndex)
      if (sourceObj.parentId === targetObj.parentId) {
        const domList = this.$refs['xxxTable'].$el.querySelectorAll('.row_parent_id_${sourceObj.parentId}')
        const params = []
        domList.map((node, i) => {
          params.push({
            id: node.classList[2].split('_')[2] // 此处拿到 row_id_xx的xx这个id
            order：i + 1 // 排序参数
          })
        })
        // 后面根据自身情况发请求，上面的参数仅供参考
      }
    }
  })
}
```
