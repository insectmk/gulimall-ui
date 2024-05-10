<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    // 获取树形分类
    getCategoriesTree () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    handleNodeClick (data) {
      console.log(data)
    }
  },
  created () {
    this.getCategoriesTree()
  }
}
</script>

<style scoped lang="scss">

</style>
