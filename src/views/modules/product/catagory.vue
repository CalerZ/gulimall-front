<template>

  <el-tree :data="menus"
           :props="defaultProps"
           @node-click="handleNodeClick"
           show-checkbox

           node-key="id"
           :expand-on-click-node="false"
  >

  </el-tree>
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
    created () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          this.menus = data.data
        })
      },
      handleNodeClick () {

      },
      append (data) {
        const newChild = {id: id++, label: 'testtest', children: []}
        if (!data.children) {
          this.$set(data, 'children', [])
        }
        data.children.push(newChild)
      },

      remove (node, data) {
        const parent = node.parent
        const children = parent.data.children || parent.data
        const index = children.findIndex(d => d.id === data.id)
        children.splice(index, 1)
      },

    }
  }
</script>
