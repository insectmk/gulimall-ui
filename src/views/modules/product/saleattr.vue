<template>
  <div>
    <base-attr :attrtype="0"></base-attr>
  </div>
</template>

<script>
import BaseAttr from './baseattr'

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {BaseAttr},
  props: {},
  data () {
    // 这里存放数据
    return {}
  },
  computed: {},
  watch: {},
  methods: {},
  created () {
  },
  mounted () {
  }
}
</script>
<style scoped>
</style>
