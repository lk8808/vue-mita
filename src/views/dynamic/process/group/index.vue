<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8">
          <el-input v-model="params.name" style="width: 200px;" class="filter-item" placeholder="组名" @keyup.enter.native="loadData" />
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
      <el-row class="operation">
        <el-col :span="4">
          <el-button-group>
            <el-button type="primary" size="mini" @click="syncUser">
              同步用户
            </el-button>
            <el-button type="primary" size="mini" @click="syncGroup">
              同步组
            </el-button>
          </el-button-group>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%" border>
      <el-table-column label="组id" prop="id" />
      <el-table-column label="组名" prop="name" />
      <el-table-column label="组类型" prop="type" />
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

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
      },
      syncUser() {
        this.$http({
          url: '/procuser/sync',
          method: 'post'
        }).then(res => {
          console.log(res)
        })
      },
      syncGroup() {

      }
    }
  }
</script>
