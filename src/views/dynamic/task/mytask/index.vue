<template>
  <div class="app-container">
    <el-tabs v-model="activeName">
      <el-tab-pane label="待办" name="first">
        <div class="header">
          <el-row class="filter">
            <el-col :span="8">
              <el-input v-model="paramsTodo.taskName" style="width: 200px;" class="filter-item" placeholder="任务名称" @keyup.enter.native="loadTaskTodo" />
              <el-input v-model="paramsTodo.procName" style="width: 200px;" class="filter-item" placeholder="流程名称" @keyup.enter.native="loadTaskTodo" />
              <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadTaskTodo">
                搜索
              </el-button>
            </el-col>
          </el-row>
        </div>
        <el-table v-loading="loading_query1" :data="taskTodoList" style="width: 100%; " size="mini">
          <el-table-column label="任务名称">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="showProc(scope.row)">{{ scope.row.taskName }}</el-button>
            </template>
          </el-table-column>
          <el-table-column label="流程名称" prop="procName" />
          <el-table-column label="开始时间" prop="createTime" />
          <el-table-column label="签收时间" prop="claimTime" />
          <el-table-column label="状态" prop="taskSts" />
          <el-table-column label="优先级" prop="priority" width="100" />
          <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
            <template slot-scope="{row}">
              <el-button v-if="row.taskSts == '待签收'" type="success" size="mini" @click="claim(row)">
                签收
              </el-button>
              <el-button v-if="row.taskSts == '激活'" type="primary" size="mini" @click="finish(row)">
                完成
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <pagination v-show="totalTodo>0" :total="totalTodo" :page.sync="paramsTodo.page" :limit.sync="paramsTodo.limit" @pagination="loadTaskTodo" />
      </el-tab-pane>
      <el-tab-pane label="已办" name="second">
        <div class="header">
          <el-row class="filter">
            <el-col :span="8">
              <el-input v-model="paramsDone.taskName" style="width: 200px;" class="filter-item" placeholder="任务名称" @keyup.enter.native="loadTaskDone" />
              <el-input v-model="paramsDone.procName" style="width: 200px;" class="filter-item" placeholder="流程名称" @keyup.enter.native="loadTaskDone" />
              <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadTaskDone">
                搜索
              </el-button>
            </el-col>
          </el-row>
        </div>
        <el-table v-loading="loading_query2" :data="taskDoneList" style="width: 100%; " size="mini">
          <el-table-column label="任务名称">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="showProc(scope.row)">{{ scope.row.taskName }}</el-button>
            </template>
          </el-table-column>
          <el-table-column label="流程名称" prop="procName" />
          <el-table-column label="开始时间" prop="startTime" />
          <el-table-column label="签收时间" prop="claimTime" />
          <el-table-column label="结束时间" prop="endTime" />
          <el-table-column label="优先级" prop="priority" width="100" />
        </el-table>
        <pagination v-show="totalDone>0" :total="totalDone" :page.sync="paramsDone.page" :limit.sync="paramsDone.limit" @pagination="loadTaskDone" />
      </el-tab-pane>
    </el-tabs>
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
        activeName: 'first',
        taskTodoList: [],
        taskDoneList: [],
        totalTodo: 0,
        totalDone: 0,
        loading_query1: false,
        loading_query2: false,
        paramsTodo: {
          page: 1,
          limit: 10
        },
        paramsDone: {
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
      this.loadTaskTodo()
      this.loadTaskDone()
    },
    methods: {
      loadTaskTodo() {
        this.loading_query1 = true
        this.$http({
          url: '/task/queryTaskTodo',
          method: 'post',
          data: this.paramsTodo
        }).then(res => {
          this.loading_query1 = false
          this.taskTodoList = res.bizdatas
          this.totalTodo = res.totalTodo || 0
        })
      },
      loadTaskDone() {
        this.loading_query2 = true
        this.$http({
          url: '/task/queryTaskDone',
          method: 'post',
          data: this.paramsDone
        }).then(res => {
          this.loading_query2 = false
          this.taskDoneList = res.bizdatas
          this.totalDone = res.totalDone || 0
        })
      },
      claim(row) {
        this.$http({
          url: '/task/claim?taskId=' + row.taskId,
          method: 'post'
        }).then(res => {
          this.loadTaskTodo()
        })
      },
      finish(row) {
        this.$confirm('确定完成?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/task/finish?taskId=' + row.taskId,
            method: 'post'
          }).then(res => {
            this.loadTaskTodo()
          })
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
      }
    }
  }
</script>
