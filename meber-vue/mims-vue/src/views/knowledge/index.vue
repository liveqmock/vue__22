<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataListPage1()">
      <div class="btns">
        <div class="input-left">
          <el-form-item>
            <el-input v-model="dataForm.name" placeholder="请输入名称" clearable @clear="getDataListPage1()"></el-input>
          </el-form-item>
          <el-button @click="getDataListPage1()">查询</el-button>
        </div>
        <div class="btns-right">
          <el-button v-if="isAuth('knowledge:knowledge:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
          <el-button v-if="isAuth('knowledge:knowledge:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
          <div class="clear"></div>
        </div>
      </div>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle" style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50">
      </el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="">
      </el-table-column>
      <el-table-column prop="resourceType" header-align="center" align="center" label="资源类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.resourceType === '1'" size="small" type="danger">文章</el-tag>
          <el-tag v-else-if="scope.row.resourceType === '2'" size="small">图片</el-tag>
          <el-tag v-else-if="scope.row.resourceType === '3'" size="small">视频</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="name" header-align="center" align="center" label="名称">
      </el-table-column>
      <el-table-column prop="clickCounts" header-align="center" align="center" label="点击次数">
      </el-table-column>
      <el-table-column prop="imagePath" header-align="center" align="center" label="图片地址">
        <template slot-scope="scope">
          <img v-if="scope.row.imagePath !='' && scope.row.imagePath !=null" :src="resourceServer+scope.row.imagePath.split(',')[0]" width="100" />
        </template>
      </el-table-column>
      <el-table-column prop="videoPath" header-align="center" align="center" label="视频地址">
        <template slot-scope="scope">
          <video v-if="scope.row.videoPath !='' && scope.row.videoPath !=null" :src="resourceServer+scope.row.videoPath" controls width="100"></video>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="content" header-align="center" align="center" label="内容">
      </el-table-column>
      <el-table-column prop="createBy" header-align="center" align="center" label="创建者">
      </el-table-column> -->
      <el-table-column prop="createDate" header-align="center" align="center" label="创建时间">
      </el-table-column>
      <!-- <el-table-column prop="updateBy" header-align="center" align="center" label="更新者">
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
export default {
  data() {
    return {
      dataForm: {
        name: ""
      },
      resourceServer: window.SITE_CONFIG["resourceServer"],
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
    getDataListPage1() {
      this.pageIndex = 1;
      this.getDataList();
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      var params = {
        page: this.pageIndex,
        limit: this.pageSize,
        name: this.dataForm.name
      };
      API.knowledge.list(params).then(({ data }) => {
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
        API.knowledge.del(ids).then(({ data }) => {
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
