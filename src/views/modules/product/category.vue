<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button v-if="draggable" size="mini" @click="batchSave">批量保存</el-button>
    <el-tree :data="menus"
             :props="defaultProps"
             show-checkbox
             node-key="catId"
             :default-expanded-keys="expandedKeys"
             :draggable="draggable"
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
             :expand-on-click-node="false">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => update(node, data)">
            Update
          </el-button>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <!--分类表单框-->
    <el-dialog :title="dialogTitle"
               :closeOnClickModal="false"
               :visible.sync="dialogFormVisible">
      <el-form :model="category" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="doCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      pCid: [],
      // 控制拖拽功能
      draggable: false,
      updateNodes: [],
      maxLevel: 0,
      operation: 'save',
      dialogTitle: '提示内容',
      // 分类内容
      category: {
        catId: 0,
        name: '',
        parentCid: 0,
        catLevel: 0,
        sort: 0,
        showStatus: 1
      },
      // 新增框的显示
      dialogFormVisible: false,
      // 树形控件默认展开key
      expandedKeys: [],
      // 分类菜单
      menus: [],
      // 树形控件对应属性名
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 批量保存
    batchSave () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '菜单顺序等修改成功!'
        })
        // 刷新菜单
        this.getCategoriesTree()
        // 设置默认展开的菜单
        this.expandedKeys = this.pCid
        this.updateNodes = []
        this.maxLevel = 0
      })
    },
    // 拖动节点后
    handleDrop (draggingNode, dropNode, dropType, ev) {
      // 1、当前节点最新的父节点id
      let pCid = 0
      let siblings = null
      if (dropType === 'before' || dropType === 'after') {
        pCid =
          dropNode.parent.data.catId === undefined
            ? 0
            : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)

      // 2、当前拖拽节点的最新顺序，
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level
            // 修改他子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          })
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
        }
      }
      // 3、当前拖拽节点的最新层级
      console.log('updateNodes', this.updateNodes)
    },
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level
          })
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    // 判断节点是否能被拖拽
    allowDrop (draggingNode, dropNode, type) {
      // console.log('allowDrop', draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode)
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1
      if (type === 'inner') {
        return (deep + dropNode.level) <= 3
      } else {
        return (deep + dropNode.parent.level) <= 3
      }
    },
    // 计算节点深度
    countNodeLevel (node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    // 更新按钮点击
    update (node, data) {
      this.operation = 'update'
      this.dialogTitle = '更新分类'
      // 查询分类信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        console.log(data)
        this.category = data.data
        this.dialogFormVisible = true
      })
    },
    // 更新分类
    updateCategory () {
      // 装载数据
      let {catId, name, icon, productUnit} = this.category
      let data = {catId, name, icon, productUnit}
      // 发送更新请求
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '更新成功!'
        })
        this.dialogFormVisible = false
        // 刷新菜单
        this.getCategoriesTree()
        // 设置默认展开的菜单
        console.log(this.category)
        console.log(this.category.catId)
        this.expandedKeys = [this.category.parentCid, this.category.catId]
      })
    },
    // 判断操作
    doCategory () {
      console.log(this.operation)
      if (this.operation === 'save') {
        this.addCategory()
      } else if (this.operation === 'update') {
        this.updateCategory()
      }
    },
    // 添加分类
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '新增成功!'
        })
        this.dialogFormVisible = false
        // 刷新菜单
        this.getCategoriesTree()
        // 设置默认展开的菜单
        this.expandedKeys = [this.category.parentCid]
      })
    },
    // 添加分类按钮点击
    append (data) {
      // 清空表单
      this.category = {}
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.operation = 'save'
      this.dialogTitle = '添加分类'
      this.dialogFormVisible = true
    },
    // 删除分类
    remove (node, data) {
      let ids = [data.catId]

      this.$confirm(`此操作将删除【${data.name}】分类，是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '分类删除成功!'
          })
          // 刷新菜单
          this.getCategoriesTree()
          // 设置默认展开的菜单
          this.expandedKeys = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 获取树形分类
    getCategoriesTree () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    }
  },
  created () {
    this.getCategoriesTree()
  }
}
</script>

<style scoped lang="scss">

</style>
