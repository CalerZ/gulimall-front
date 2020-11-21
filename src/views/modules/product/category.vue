<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      show-checkbox
      node-key="catId"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)"
            >编辑</el-button
          >
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="category">
        <el-form-item label="分类名称" label-width="30%">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" label-width="30%">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" label-width="30%">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
         <el-form-item label="是否发布" label-width="30%">
          <el-switch
  v-model="category.showSatus"
  active-color="#13ce66"
  inactive-color="#ff4949">
</el-switch>
        </el-form-item>
       
        </el-switch>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",

      sureType: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
        showStatus: 0
      },
      dialogFormVisible: false,
      expandedKey: [],
      menus: [],
      defaultProps: {
        children: "children",
        label: "name"
      }
    };
  },
  created() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get"
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    handleNodeClick() {},

    append(data) {
      console.log(data)
      this.sureType = "add";
      this.title = "添加分类";
      this.category.name = "";
      
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.showStatus = 0;

      this.dialogFormVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    submitData() {
      console.log(this.sureType);
      if (this.sureType == "edit") {
        this.editCateData();
      } else {
        this.addCateData();
      }
    },
    //添加
    addCateData() {
      console.log(this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "添加成功!"
        });
        this.dialogFormVisible = false;
        this.getDataList();
        this.expandedKey = [this.category.parentCid];
      });
    },
    //编辑
    edit(data) {
      this.dialogFormVisible = true;
      this.sureType = "edit";
      this.title = "编辑分类";
      //获取修改菜单最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get"
      }).then(({ data }) => {
        console.log(data);
        this.category.name = data.category.name;
        this.category.catId = data.category.catId;
        this.category.icon = data.category.icon;
        this.category.productUnit = data.category.productUnit;
        this.category.parentCid = data.category.parentCid;
        this.category.showStatus = data.category.showStatus;
      });
    },
    //编辑数据提交
    editCateData() {
      let { name, catId, icon, productUnit, showStatus } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(
          { name, catId, icon, productUnit, showStatus },
          false
        )
      }).then(({ data }) => {
        //对象解构 左右相同的变量才能成功解构他，否则不能成功解构。
        this.dialogFormVisible = false;
        this.$message({
          type: "success",
          message: "编辑成功!"
        });

        //调取查询列表
        this.getDataList();
        //默认展开删除节点的列表
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      let ids = [data.catId];

      this.$confirm("是否删除该菜单?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        })
          .then(({ data }) => {
            //对象解构 左右相同的变量才能成功解构他，否则不能成功解构。

            this.$message({
              type: "success",
              message: "删除成功!"
            });

            //调取查询列表
            this.getDataList();
            //默认展开删除节点的列表
            this.expandedKey = [node.parent.data.catId];
          })
          .catch(() => {
            this.$message({
              type: "info",
              message: "已取消删除"
            });
          });
      });
    }
  }
};
</script>
