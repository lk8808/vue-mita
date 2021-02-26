<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8">
          <el-input v-model="params.key" style="width: 200px;" class="filter-item" placeholder="key" @keyup.enter.native="loadData" />
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%;" border size="mini">
      <el-table-column label="流程实例id">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="view(scope.row)">{{ scope.row.id }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="流程名" prop="processDefinitionName" />
      <el-table-column label="key" prop="processDefinitionKey" align="center" />
      <el-table-column label="开始时间" prop="startTime" />
      <el-table-column label="结束时间" prop="endTime" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button v-if="!row.endTime" type="primary" size="mini" @click="stop(row)">
            终止
          </el-button>
          <el-button v-if="row.endTime" type="danger" size="mini" @click="del(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

    <el-dialog :visible.sync="dialogViewVisible" style="text-align: center;">
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
        loading_save: false,
        params: {
          page: 1,
          limit: 10
        },
        textMap: {
          add: '新增',
          edit: '编辑'
        },
        dialogViewVisible: false,
        bizdata: {},
        imgurl: ''
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
          url: '/procinst/queryList',
          method: 'post',
          data: this.params
        }).then(res => {
          console.log(res.bizdatas)
          this.loading_query = false
          this.bizdatas = res.bizdatas
          this.total = res.total || 0
        })
      },
      view(row) {
        this.$http({
          url: '/procinst/getProcimg?procInstId=' + row.id,
          method: 'post'
        }).then(res => {
          this.imgurl = 'data:image/png;base64,' + res
          this.dialogViewVisible = true
        })
      },
      stop(row) {
        this.$confirm('确定终止?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/procinst/stop?procInstId=' + row.id,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      },
      del(row) {
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/procinst/del?procInstId=' + row.id,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      }
    }
  }
</script>
