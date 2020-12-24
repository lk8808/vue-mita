<template>
  <div class="app-container">
    <div class="header">
      <el-row class="filter">
        <el-col :span="8" >
          <el-input v-model="params.rolename" placeholder="请输入角色名" style="width: 200px;" class="filter-item"
                    @keyup.enter.native="loadData"/>
          <el-button type="primary" size="small" icon="el-icon-search" style="margin: 5px;" @click="loadData">
            搜索
          </el-button>
        </el-col>
      </el-row>
      <el-row class="operation">
        <el-col :span="4">
          <el-button-group>
            <el-button type="primary" size="mini" icon="el-icon-plus" @click="add"></el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteBatch"></el-button>
          </el-button-group>
        </el-col>
      </el-row>
    </div>
    <el-table v-loading="loading_query" :data="bizdatas" style="width: 100%"
              @selection-change="selectRows" @row-dblclick="inlineEdit">
      <el-table-column type="selection" width="60" align="center" />
      <el-table-column type="index" width="100" label="序号" align="center" />
      <el-table-column label="角色编号" >
        <template slot-scope="scope">
          <el-button @click="view(scope.row)" type="text" size="small">{{scope.row.roleno}}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="角色名称" prop="rolename" />
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="edit(scope.row)">
            编辑
          </el-button>
          <el-button type="danger" v-if="scope.row.status!='deleted'" size="mini" @click="del(scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="params.page" :limit.sync="params.limit" @pagination="loadData" />

    <el-dialog :title="textMap[dialogFormStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" v-loading="loading_save" :rules="rules" :model="bizdata" label-position="left"
               label-width="100px"
               element-loading-text="正在保存" >
        <el-form-item label="角色编号" prop="roleno">
          <el-input v-model="bizdata.roleno" style="width: 300px;" />
        </el-form-item>
        <el-form-item label="角色名称" prop="rolename">
          <el-input v-model="bizdata.rolename" style="width: 500px;" />
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

    <el-dialog :title="viewbizdata.rolename" :visible.sync="dialogViewVisible">
      <div class="header">
        <el-row class="operation">
          <el-col :span="4">
            <el-button-group>
              <el-button type="primary" size="mini" icon="el-icon-plus" @click="addRoleauth"></el-button>
              <el-button type="success" size="mini" icon="fa fa-floppy-o" @click="saveRoleauth"></el-button>
            </el-button-group>
          </el-col>
        </el-row>
      </div>
      <el-table :data="roleauths" size="mini" empty-text="">
        <el-table-column min-width="300px" label="角色类型">
          <template slot-scope="{row}">
            <el-input v-if="row.edit" v-model="row.name" class="edit-input" size="small" />
            <span v-else>{{ row.name }}</span>
          </template>
        </el-table-column>
        <el-table-column min-width="300px" label="配置名称">
          <template slot-scope="{row}">
            <el-input v-if="row.edit" v-model="row.name" class="edit-input" size="small" />
            <span v-else>{{ row.name }}</span>
          </template>
        </el-table-column>
      </el-table>
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
          rolename: ''
        },
        dialogFormVisible: false,
        dialogViewVisible: false,
        dialogFormStatus: '',
        textMap: {
          add: '新增',
          edit: '编辑'
        },
        bizdata: {},
        viewbizdata: {},
        roleauths: [],
        rules: {
          roleno: [{ required: true, message: '角色编号必填', trigger: 'blur' }],
          rolename: [{ required: true, message: '角色名称必填', trigger: 'blur' }]
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
          url: '/role/queryList',
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
        this.dialogFormStatus = 'add'
        this.dialogFormVisible = true
      },
      edit(row) {
        this.bizdata = row
        this.dialogFormStatus = 'edit'
        this.dialogFormVisible = true
      },
      view(row) {
        this.viewbizdata = row
        this.dialogViewVisible = true
      },
      save() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.loading_save = true
            this.$http({
              url: '/role/save',
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
      del(row) {
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: '/role/del?ids=' + row.id,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      },
      deleteBatch() {
        if (this.selectedRows.length == 0) {
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
            url: '/role/del?ids=' + ids,
            method: 'post'
          }).then(res => {
            this.loadData()
          })
        })
      },
      addRoleauth() {

      },
      saveRoleauth() {

      }
    }
  }
</script>
