<template>
  <div>
    <el-table v-loading="loading_query" :data="bizdatas" size="mini">
      <el-table-column label="流程key">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="edit(scope.row)">{{ scope.row.key }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="流程名" prop="name" />
      <el-table-column label="版本号" prop="version" />
      <el-table-column label="流程定义id" prop="id" width="350" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="del(row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog id="view" :visible.sync="dialogViewVisible" fullscreen append-to-body>
      <process-view :bizdata="editBizdata" @cancel="dialogViewVisible = false" />
    </el-dialog>
  </div>
</template>

<script>
  import ProcessView from './view.vue'
  export default {
    components: { ProcessView },
    props: {
      bizdata: {
        required: true,
        type: Object
      }
    },
    data() {
      return {
        bizdatas: [],
        editBizdata: {},
        loading_query: false,
        dialogViewVisible: false
      }
    },
    watch: {
      bizdata(val) {
        this.loadData()
      },
      dialogFormVisible(val) {
        if ( !val ) {
          this.editBizdata = {}
        }
      }
    },
    created() {
      this.loadData()
    },
    methods: {
      loadData() {
        this.loading_query = true
        this.$http({
          url: '/procdef/queryListByKey?key=' + this.bizdata.key,
          method: 'post',
          data: this.params
        }).then(res => {
          this.loading_query = false
          this.bizdatas = res
        })
      },
      del(row) {
        this.$http({
          url: '/procdef/del?deploymentid=' + row.deploymentid,
          method: 'post'
        }).then(res => {
          this.loadData()
        })
      },
      edit(row) {
        this.editBizdata = row
        this.dialogViewVisible = true
      }
    }
  }
</script>
