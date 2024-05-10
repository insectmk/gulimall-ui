<template>
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
</template>

<script>
export default {
  data () {
    return {
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
    append (data) {
      console.log('append', data)
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
