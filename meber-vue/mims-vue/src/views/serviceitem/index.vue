<template>
  <div class="mod-config">

    <div class="list-nav">
      <category listname="项目大类" :list="items" @onItemClick="onProjectClick"></category>
      <category v-if="items2.length > 0" listname="项目小类" :list="items2" @onItemClick="onProject2Click"></category>
      <category listname="所属门店" :list="shopList" @onItemClick="onShopClick"></category>
      <category listname="状&emsp;&emsp;态" :list="state" @onItemClick="onStateClick"></category>
    </div>

    <el-form :inline="true" @keyup.enter.native="getDataList()">
      <div class="btns">
        <div class="btns-right">
          <el-button v-if="isAuth('serviceItem:serviceitem:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
          <el-button v-if="isAuth('serviceItem:serviceitem:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
          <div class="clear"></div>
        </div>
      </div>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50">
      </el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="ID" width="50">
      </el-table-column>
      <el-table-column prop="name" header-align="center" align="center" label="项目名称">
      </el-table-column>
      <el-table-column prop="itemno" header-align="center" align="center" label="项目编号">
      </el-table-column>
      <el-table-column prop="officeId" header-align="center" align="center" label="所属门店">
        <template slot-scope="scope">
          {{scope.row.officeId | shopName(shopList)}}
        </template>
      </el-table-column>
      <el-table-column prop="salePrice" header-align="center" align="center" label="售价">
      </el-table-column>
      <!-- <el-table-column prop="itemTime" header-align="center" align="center" label="项目时长">
      </el-table-column>
      <el-table-column prop="catFirst" header-align="center" align="center" label="一级分类">
      </el-table-column>
      <el-table-column prop="catSecond" header-align="center" align="center" label="二级分类">
      </el-table-column> -->
      <!-- 是否可预约：0、否 1、是 -->
      <el-table-column header-align="center" align="center" label="是否可预约">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isOrder === 0" size="small" type="danger">否</el-tag>
          <el-tag v-else size="small">是</el-tag>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="experiencePrice" header-align="center" align="center" label="体验价">
      </el-table-column> -->
      <!-- 状态：0、停售  1、在售 -->
      <el-table-column header-align="center" align="center" label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small" type="danger">停售</el-tag>
          <el-tag v-else size="small">在售</el-tag>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="images" header-align="center" align="center" label="项目配图">
      </el-table-column>
      <el-table-column prop="content" header-align="center" align="center" label="项目内容">
      </el-table-column>
      <el-table-column prop="createBy" header-align="center" align="center" label="创建者">
      </el-table-column>
      <el-table-column prop="createDate" header-align="center" align="center" label="创建时间">
      </el-table-column>
      <el-table-column prop="updateBy" header-align="center" align="center" label="更新者">
      </el-table-column>
      <el-table-column prop="updateDate" header-align="center" align="center" label="更新时间">
      </el-table-column>
      <el-table-column prop="remarks" header-align="center" align="center" label="备注信息">
      </el-table-column>
      <el-table-column prop="delFlag" header-align="center" align="center" label="删除标记">
      </el-table-column> -->
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex" :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage" layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import API from "@/api";
import AddOrUpdate from "./add-or-update";
import category from "@/components/category";
export default {
  data() {
    return {
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      roleList: [],
      items: [],
      items2: [],
      shopList: [],
      state: [{ name: "在售" }, { name: "停业" }]
    };
  },
  components: {
    AddOrUpdate,
    category
  },
  computed: {},
  mounted() {
    this.getDataList();
    this.getCategory();
  },
  filters: {
    shopName(officeId, shopList) {
      for (let i = 0; i < shopList.length; i++) {
        if (shopList[i].id === +officeId) {
          return shopList[i].name;
        }
      }
      return "未知";
    }
  },
  methods: {
    getCategory() {
      API.common.getCategoryList().then(({ data }) => {
        this.items = data.list["items"];
      });
      API.common.getOfficeList().then(({ data }) => {
        this.shopList = data.list;
      });
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      var params = {
        page: this.pageIndex,
        limit: this.pageSize
      };
      if (this.filter_project !== undefined) {
        params.catFirst = this.filter_project;
      }
      if (this.filter_project2 !== undefined) {
        params.catSecond = this.filter_project2;
      }
      if (this.filter_shop !== undefined) {
        params.officeId = this.filter_shop;
      }
      if (this.filter_state !== undefined) {
        params.status = this.filter_state;
      }
      API.serviceitem.list(params).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    onProjectClick(item) {
      if (item) {
        this.items2 = item.categoryList;
        this.filter_project = item.id;
      } else {
        this.items2 = [];
        this.filter_project = undefined;
      }
      this.filter_project2 = undefined;
      this.getDataList();
    },
    onProject2Click(item) {
      if (item) {
        this.filter_project2 = item.id;
      } else {
        this.filter_project2 = undefined;
      }
      this.getDataList();
    },
    onShopClick(item) {
      if (item) {
        this.filter_shop = item.id;
      } else {
        this.filter_shop = undefined;
      }
      this.getDataList();
    },
    onStateClick(item) {
      if (item) {
        if (item.name === "在售") {
          this.filter_state = 1;
        } else {
          this.filter_state = 0;
        }
      } else {
        this.filter_state = undefined;
      }
      this.getDataList();
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
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(`确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        API.serviceitem.del(ids).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    }
  }
};
</script>
<style scoped>
</style>
