<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8">
          <el-input v-model="params.name" style="width: 200px;" class="filter-item" placeholder="流程名" @keyup.enter.native="loadData" />
          <el-input v-model="params.key" style="width: 200px;" class="filter-item" placeholder="key" @keyup.enter.native="loadData" />
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
      <el-row class="operation">
        <el-col :span="4">
          <el-button-group>
            <el-button type="primary" size="mini" icon="el-icon-plus" @click="add" />
          </el-button-group>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%; " border>
      <el-table-column label="流程key">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="version(scope.row)">{{ scope.row.key }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="流程名" prop="name" />
      <el-table-column label="版本号" prop="version" align="center" width="100" />
      <el-table-column label="流程部署id" prop="deploymentid" />
      <el-table-column label="流程定义id" prop="id" width="400" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="edit(row)">
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

    <el-dialog title="历史版本" :visible.sync="dialogVersionVisible">
      <process-version :bizdata="bizdata" @cancel="dialogVersionVisible = false" />
    </el-dialog>

    <el-dialog id="edit" :visible.sync="dialogFormVisible" fullscreen>
      <process-edit :bizdata="bizdata" @cancel="dialogFormVisible = false" />
    </el-dialog>
  </div>
</template>

<script>
  import Pagination from '@/components/Pagination'
  import ProcessEdit from './components/edit.vue'
  import ProcessVersion from './components/version.vue'

  export default {
    components: { Pagination, ProcessEdit, ProcessVersion },
    data() {
      return {
        bizdatas: [],
        selectedRows: [],
        total: 0,
        loading_query: false,
        loading_save: false,
        params: {
          page: 1,
          limit: 10
        },
        textMap: {
          add: '新增',
          edit: '编辑'
        },
        dialogFormVisible: false,
        dialogVersionVisible: false,
        dialogStatus: '',
        bizdata: {}
      }
    },
    created() {
      this.loadData()
    },
    methods: {
      filter() {
        this.params.page = 1
        this.loadData()
      },
      loadData() {
        this.loading_query = true
        this.$http({
          url: '/procdef/queryList',
          method: 'post',
          data: this.params
        }).then(res => {
          this.loading_query = false
          this.bizdatas = res.bizdatas
          this.total = res.total || 0
        })
      },
      add() {
        this.bizdata = {}
        this.dialogStatus = 'add'
        this.dialogFormVisible = true
      },
      edit(row) {
        this.bizdata = row
        this.dialogStatus = 'edit'
        this.dialogFormVisible = true
      },
      version(row) {
        this.bizdata = row
        this.dialogVersionVisible = true
      }
    }
  }
</script>

<style scoped>

  /deep/ #edit .el-dialog__header {
    display: none;
  }

   /deep/ #edit .el-dialog__body {
    padding: 0;
  }

</style>
