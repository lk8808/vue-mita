<template>
  <div class="app-container">
    <el-row>
      <el-col :span="12">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>我的任务</span>
            <el-link type="primary" style="float: right;">
              <router-link to="task/mytask">更多</router-link>
            </el-link>
          </div>
          <div v-for="item in bizdatas" :key="item.id" class="item">
            <span class="left">
              <el-link type="primary">【{{ item.procName }}】{{ item.taskName }}</el-link>
            </span>
            <span class="right"> {{ item.createTime }}</span>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        bizdatas: [],
        params: {
          page: 1,
          limit: 5
        }
      }
    },
    created() {
      this.loadData()
    },
    methods: {
      loadData() {
        this.$http({
          url: '/task/queryTaskTodo',
          method: 'post',
          data: this.params
        }).then(res => {
          this.bizdatas = res.bizdatas
          this.total = res.total || 0
        })
      },
      finish(row) {
        this.$confirm('确定完成?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/task/finish?taskId=' + row.id,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      }
    }
  }
</script>
<style scoped>
  .item {
    display: flex;
    font-size: 14px;
    padding: 10px 0;
  }
  .item .left{
    flex: 1;
    justify-content: left;
  }
</style>
