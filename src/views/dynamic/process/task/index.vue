<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8">
          <el-input v-model="params.taskName" style="width: 200px;" class="filter-item" placeholder="任务名称" @keyup.enter.native="loadData" />
          <el-input v-model="params.procName" style="width: 200px;" class="filter-item" placeholder="流程名称" @keyup.enter.native="loadData" />
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%; " border size="mini">
      <el-table-column label="任务名称">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="showProc(scope.row)">{{ scope.row.taskName }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="流程名称" prop="procName" />
      <el-table-column label="执行人" prop="assigneeName" />
      <el-table-column label="开始时间" prop="startTime" />
      <el-table-column label="签收时间" prop="claimTime" />
      <el-table-column label="结束时间" prop="endTime" />
      <el-table-column label="状态" prop="taskSts" />
      <el-table-column label="优先级" prop="priority" width="100" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="danger" size="mini" @click="del(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

    <el-dialog :title="procName" :visible.sync="dialogViewVisible" style="text-align:center;">
      <el-image :src="imgurl" fit="fit" />
    </el-dialog>

  </div>
</template>

<script>
  import Pagination from '@/components/Pagination'

  export default {
    components: { Pagination },
    data() {
      return {
        bizdatas: [],
        selectedRows: [],
        total: 0,
        loading_query: false,
        params: {
          page: 1,
          limit: 10
        },
        dialogViewVisible: false,
        bizdata: {},
        procName: '',
        imgurl: ''
      }
    },
    created() {
      this.loadData()
    },
    methods: {
      loadData() {
        this.loading_query = true
        this.$http({
          url: '/task/queryAll',
          method: 'post',
          data: this.params
        }).then(res => {
          this.loading_query = false
          this.bizdatas = res.bizdatas
          this.total = res.total || 0
        })
      },
      showProc(row) {
        this.procName = row.procName
        this.$http({
          url: '/procinst/getProcimg?procInstId=' + row.procInstId,
          method: 'post'
        }).then(res => {
          this.imgurl = 'data:image/png;base64,' + res
          this.dialogViewVisible = true
        })
      },
      del(row) {
        if (!row.endTime) {
          this.$message({
            message: '任务还未结束，不可删除',
            type: 'warning'
          });
          return
        }
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/task/del?taskId=' + row.taskId,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      }
    }
  }
</script>
