<template>
  <div>
    <el-tree :data="menus"
             :props="defaultProps"
             show-checkbox
             node-key="catId"
             :default-expanded-keys="expandedKeys"
             :expand-on-click-node="false">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
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
<!--分类新增框-->
    <el-dialog title="添加分类" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 分类内容
      category: {
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
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
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
