<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="参数名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('product:brand:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('product:brand:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
      </el-form-item>
      <el-form-item>
        <div style="width:70px;height:35px">
          <icon-svg name="search" style="height:100%;width:100%"></icon-svg>
        </div>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="60"
        sortable
      >
      </el-table-column>
      <el-table-column
        prop="brandId"
        header-align="center"
        align="center"
        label="品牌id"
        sortable
      >
        <template slot-scope="scope">
          <el-input
            v-if="!scope.row.brandId"
            v-model="scope.row.brandId"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.brandId">
          {{scope.row.brandId}}
        </template>
        </template>
       
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="品牌名"
        sortable
      >
       <template slot-scope="scope">
          <el-input
            v-if="!scope.row.name"
            v-model="scope.row.name"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.name">
          {{scope.row.name}}
        </template>
        </template>
      </el-table-column>
     
       
      <el-table-column
        prop="logo"
        header-align="center"
        align="center"
        label="品牌logo地址"
        sortable
      >
       <template slot-scope="scope">
          <el-input
            v-if="!scope.row.logo"
            v-model="scope.row.logo"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.logo">
          {{scope.row.logo}}
        </template>
        </template>
      </el-table-column>
      <el-table-column
        prop="descript"
        header-align="center"
        align="center"
        label="介绍"
        sortable
      >
      <template slot-scope="scope">
          <el-input
            v-if="!scope.row.descript"
            v-model="scope.row.descript"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.descript">
          {{scope.row.descript}}
        </template>
        </template>
      </el-table-column>
      <el-table-column header-align="center" align="center" label="显示状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.showStatus"
            active-color="#13ce66"
            inactive-color="#ff4949"
            @change="updateShowStatus(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column
        prop="firstLetter"
        header-align="center"
        align="center"
        label="检索首字母"
        sortable
      >
       <template slot-scope="scope">
          <el-input
            v-if="!scope.row.firstLetter"
            v-model="scope.row.firstLetter"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.firstLetter">
          {{scope.row.firstLetter}}
        </template>
        </template>
      </el-table-column>
      <el-table-column
        prop="sort"
        header-align="center"
        align="center"
        label="排序"
        sortable
      >
       <template slot-scope="scope">
          <el-input
            v-if="scope.row.sort=='sort'?true:false"
            v-model="scope.row.sort"
            placeholder="请输入内容"
          ></el-input>
           <template v-if="scope.row.sort!='sort'?true:false">
          {{scope.row.sort}}
        </template>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
           v-if="scope.row.brandId"
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.brandId)"
            >修改</el-button
          >
          <el-button
           v-if="scope.row.brandId"
            type="text"
            size="small"
            @click="deleteHandle(scope.row.brandId)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./brand-add-or-update";
export default {
  data() {
    return {
      input: "",
      dataForm: {
        key: ""
      },
      iconList: [],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  components: {
    AddOrUpdate
  },

  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/brand/list"),
        method: "post",
        data: this.$http.adornData({
          currPage: this.pageIndex,
          pageSize: this.pageSize,
          params: { key: this.dataForm.key }
        })
      }).then(data => {
        if (data) {
          for (let i = 0; i < data.data.list.length; i++) {
            if (data.data.list[i].showStatus == 1) {
              data.data.list[i].showStatus = true;
            } else {
              data.data.list[i].showStatus = false;
            }
          }
          this.dataList = data.data.list;
          this.totalPage = data.data.totalCount;
          this.dataList.unshift(
            {
              brandId: "",
              descript: "",
              firstLetter: "",
              isDelete: "",
              logo: "",
              name: "",
              showStatus: "",
              sort:'sort'
            }
          );
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    updateShowStatus(datas) {
      //获取id和状态
      //发送请求

      this.$http({
        url: this.$http.adornUrl("/product/brand/update/status"),
        method: "get",
        params: this.$http.adornParams(
          { brandId: datas.brandId, status: datas.showStatus == false ? 0 : 1 },
          false
        )
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "状态修改成功!"
        });
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.brandId;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/brand/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 1500,
            onClose: () => {
              this.getDataList();
            }
          });
        });
      });
    }
  }
};
</script>
