<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8">
          <el-input v-model="params.posname" placeholder="请输入岗位名" style="width: 200px;" class="filter-item"
                    @keyup.enter.native="loadData"
          />
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
      <el-row class="operation">
        <el-col :span="4">
          <el-button-group>
            <el-button type="primary" size="mini" icon="el-icon-plus" @click="add" />
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteBatch" />
          </el-button-group>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%" border
              @selection-change="selectRows"
    >
      <el-table-column type="selection" width="60" align="center" />
      <el-table-column label="岗位编号" prop="posno" />
      <el-table-column label="岗位名称" prop="posname" />
      <el-table-column label="序号" align="center" prop="sortno" />
      <el-table-column label="操作" align="center" width="230">
        <template slot-scope="{row, $index}">
          <el-button type="primary" size="mini" @click="edit(row)">
            编辑
          </el-button>
          <el-button v-if="row.status!='deleted'" type="danger" size="mini" @click="del(row, $index)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" v-loading="loading_save" :rules="rules" :model="bizdata" label-position="left"
               label-width="100px"
               element-loading-text="正在保存"
      >
        <el-form-item label="岗位编号" prop="posno">
          <el-input v-model="bizdata.posno" style="width: 300px;" />
        </el-form-item>
        <el-form-item label="岗位名称" prop="posname">
          <el-input v-model="bizdata.posname" style="width: 500px;" />
        </el-form-item>
        <el-form-item label="序号" prop="sortno">
          <el-input v-model="bizdata.sortno" style="width: 200px;" oninput="value=value.replace(/[^\d]/g,'')" />
        </el-form-item>
        <el-form-item>
          <el-button @click="dialogFormVisible = false">
            取消
          </el-button>
          <el-button type="primary" @click="save">
            保存
          </el-button>
        </el-form-item>
      </el-form>
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
          limit: 10,
          posname: ''
        },
        dialogFormVisible: false,
        dialogStatus: '',
        textMap: {
          add: '新增',
          edit: '编辑'
        },
        bizdata: {},
        rules: {
          posno: [{ required: true, message: '岗位编号必填', trigger: 'blur' }],
          posname: [{ required: true, message: '岗位名称必填', trigger: 'blur' }]
        }
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
          url: '/position/queryList',
          method: 'post',
          data: this.params
        }).then(res => {
          this.loading_query = false
          this.bizdatas = res.bizdatas
          this.total = res.total || 0;
        })
      },
      selectRows(rows) {
        this.selectedRows = rows
      },
      inlineEdit(row) {
        row.edit = !row.edit
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
      save() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.loading_save = true
            this.$http({
              url: '/position/save',
              method: 'post',
              data: this.bizdata
            }).then(res => {
              this.loading_save = false
              this.dialogFormVisible = false
              this.params.page = 1
              this.loadData()
            }).catch(err => {
              this.loading_save = false
            })
          }
        })
      },
      del(row, index) {
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/position/del?ids=' + row.id,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      },
      deleteBatch() {
        if (this.selectedRows.length === 0) {
          this.$message({
            type: 'warning',
            message: '请选择要删除的数据'
          });
          return
        }
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = this.selectedRows.map( (obj) => {
            return obj.id
          }).join(',')
          this.$http({
            url: '/position/del?ids=' + ids,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      }
    }
  }

</script>

<style>
</style>
