<template>
  <div class="app-container">
    <div class="header">
      <el-row class="operation">
        <el-col :span="4">
          <el-button-group>
            <el-button type="primary" size="mini" @click="sync">
              同步用户组
            </el-button>
          </el-button-group>
        </el-col>
      </el-row>
    </div>
    <el-tabs v-model="activeName">
      <el-tab-pane label="部门" name="first">
        <el-table :data="bizdatas1.filter(data => !searchDep || data.name.toLowerCase().includes(searchDep))" style="width: 100%">
          <el-table-column label="组id" prop="id" />
          <el-table-column>
            <template slot="header" slot-scope="scope">
              <el-input v-model="searchDep" size="medium" placeholder="输入关键字搜索" />
            </template>
            <template slot-scope="{row}">
              {{ row.name }}
            </template>
          </el-table-column>
          <el-table-column label="组类型" prop="type" />
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="岗位" name="second">
        <el-table :data="bizdatas2.filter(data => !searchPos || data.name.toLowerCase().includes(searchPos))" style="width: 100%">
          <el-table-column label="组id" prop="id" />
          <el-table-column>
            <template slot="header" slot-scope="scope">
              <el-input v-model="searchPos" size="medium" placeholder="输入关键字搜索" />
            </template>
            <template slot-scope="{row}">
              {{ row.name }}
            </template>
          </el-table-column>
          <el-table-column label="组类型" prop="type" />
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="部门岗位" name="third">
        <el-table :data="bizdatas3.filter(data => !searchDeppos || data.name.toLowerCase().includes(searchDeppos))" style="width: 100%">
          <el-table-column label="组id" prop="id" />
          <el-table-column>
            <template slot="header" slot-scope="scope">
              <el-input v-model="searchDeppos" size="medium" placeholder="输入关键字搜索" />
            </template>
            <template slot-scope="{row}">
              {{ row.name }}
            </template>
          </el-table-column>
          <el-table-column label="组类型" prop="type" />
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>

  </div>
</template>
<script>

  export default {
    data() {
      return {
        activeName: 'first',
        searchDep: '',
        searchPos: '',
        searchDeppos: '',
        bizdatas1: [],
        bizdatas2: [],
        bizdatas3: [],
        params: {}
      }
    },
    created() {
      this.loadDep()
      this.loadPos()
      this.loadDeppos()
    },
    methods: {
      filter() {
        this.params.page = 1
      },
      loadDep() {
        this.$http({
          url: '/procUser/queryGroupsByType?type=D',
          method: 'post'
        }).then(res => {
          this.bizdatas1 = res
        })
      },
      loadPos() {
        this.$http({
          url: '/procUser/queryGroupsByType?type=P',
          method: 'post'
        }).then(res => {
          this.bizdatas2 = res
        })
      },
      loadDeppos() {
        this.$http({
          url: '/procUser/queryGroupsByType?type=O',
          method: 'post'
        }).then(res => {
          this.bizdatas3 = res
        })
      },
      sync() {
        this.$http({
          url: '/procUser/sync',
          method: 'post'
        }).then(res => {
          this.$message({
            message: '同步成功',
            type: 'success'
          });
        })
      }
    }
  }
</script>
