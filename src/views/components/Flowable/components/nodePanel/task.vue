<template>
  <div>
    <el-form ref="dataForm" :model="formData" size="medium" label-width="100px" label-position="right">
      <el-form-item label="节点id">
       <el-input v-model="formData.id"></el-input>
      </el-form-item>
      <el-form-item label="节点名称">
       <el-input v-model="formData.name"></el-input>
      </el-form-item>
      <el-form-item label="节点描述">
       <el-input v-model="formData.documentation"></el-input>
      </el-form-item>
      <el-form-item label="执行监听器">
       <el-badge :value="executionListenerLength">
         <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
       </el-badge>
      </el-form-item>
      <el-form-item label="任务监听器">
       <el-badge :value="taskListenerLength">
         <el-button size="small" @click="dialogName = 'taskListenerDialog'">编辑</el-button>
       </el-badge>
      </el-form-item>
      <el-form-item label="人员类型">
       <el-select v-model="formData.userType">
         <el-option v-for="item in userTypeOption" :key="item.value" :label="item.label" :value="item.value" />
       </el-select>
      </el-form-item>
      <el-form-item label="指定人员" v-if="formData.userType == 'assignee'" prop="assignee">
        <el-select v-model="formData.assignee" placeholder="请选择" filterable >
          <el-option v-for="item in emps" :key="item.empno" :label="item.empname" :value="String(item.id)">
            <span style="float: left">{{ item.empname }}【{{ item.empno }}】</span>
            <span style="float: right; color: #8492a6; font-size: 13px">{{ item.depname }}</span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="候选人员" v-if="formData.userType == 'candidateUsers'">
       <el-select v-model="formData.candidateUsers" multiple filterable allow-create placeholder="请选择" >
         <el-option v-for="item in emps" :key="item.empno" :label="item.empname" :value="String(item.id)">
           <span style="float: left">{{ item.empname }}【{{ item.empno }}】</span>
           <span style="float: right; color: #8492a6; font-size: 13px">{{ item.depname }}</span>
         </el-option>
       </el-select>
      </el-form-item>
      <el-form-item label="侯选组" v-if="formData.userType == 'candidateGroups'">
        <el-select v-model="formData.candidateGroups" multiple filterable placeholder="请选择" style="min-width: 250px;">
          <el-option v-for="item in groups" :key="item.groupid" :label="item.groupname" :value="item.groupid">
            <span style="float: left">{{ item.groupname }}</span>
            <span style="float: right; color: #8492a6; font-size: 13px">
              {{ item.grouptype=='D' ? '部门' : item.grouptype=='P' ? '岗位' : '职位' }}
            </span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="多实例">
       <el-badge :is-dot="hasMultiInstance">
         <el-button size="small" @click="dialogName = 'multiInstanceDialog'">编辑</el-button>
       </el-badge>
      </el-form-item>
      <el-form-item label="异步" v-if="showConfig.async">
       <el-switch v-model="formData.async" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="优先级" v-if="showConfig.priority">
       <el-input v-model="formData.priority"></el-input>
      </el-form-item>
      <el-form-item label="表单标识key" v-if="showConfig.formKey">
       <el-input v-model="formData.formKey"></el-input>
      </el-form-item>
      <el-form-item label="跳过表达式" v-if="showConfig.skipExpression">
       <el-input v-model="formData.skipExpression"></el-input>
      </el-form-item>
      <el-form-item label="是否为补偿" v-if="showConfig.isForCompensation">
       <el-switch v-model="formData.isForCompensation" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="服务任务可触发" v-if="showConfig.triggerable">
       <el-switch v-model="formData.triggerable" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="自动存储变量" v-if="showConfig.autoStoreVariables">
       <el-switch v-model="formData.autoStoreVariables" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="输入变量" v-if="showConfig.ruleVariablesInput">
       <el-switch v-model="formData.ruleVariablesInput" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="规则" v-if="showConfig.rules">
       <el-input v-model="formData.rules"></el-input>
      </el-form-item>
      <el-form-item label="结果变量" v-if="showConfig.resultVariable">
       <el-input v-model="formData.resultVariable"></el-input>
      </el-form-item>
      <el-form-item label="排除" v-if="showConfig.exclude">
       <el-switch v-model="formData.exclude" active-text="是" inactive-text="否"/>
      </el-form-item>
      <el-form-item label="类" v-if="showConfig.class">
        <el-input v-model="formData.class"></el-input>
      </el-form-item>
      <el-form-item label="到期时间" v-if="showConfig.dueDate">
        <el-date-picker v-model="formData.dueDate" type="datetime" placeholder="选择日期时间"/>
      </el-form-item>
    </el-form>

    <executionListenerDialog
      v-if="dialogName === 'executionListenerDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishExecutionListener"
    />
    <taskListenerDialog
      v-if="dialogName === 'taskListenerDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishTaskListener"
    />
    <multiInstanceDialog
      v-if="dialogName === 'multiInstanceDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishMultiInstance"
    />
  </div>
</template>

<script>
import mixinPanel from '../../common/mixinPanel'
import executionListenerDialog from './property/executionListener'
import taskListenerDialog from './property/taskListener'
import multiInstanceDialog from './property/multiInstance'
import { commonParse, userTaskParse } from '../../common/parseElement'
import DepSelect from '@/views/components/DepSelect'
import EmpSelect from '@/views/components/EmpSelect'

export default {
  components: {
    executionListenerDialog,
    taskListenerDialog,
    multiInstanceDialog,
    DepSelect,
    EmpSelect
  },
  mixins: [mixinPanel],
  data() {
    return {
      userTypeOption: [
        { label: '指定人员', value: 'assignee' },
        { label: '候选人员', value: 'candidateUsers' },
        { label: '候选组', value: 'candidateGroups' }
      ],
      emps: [],
      groups: [],
      dialogName: '',
      executionListenerLength: 0,
      taskListenerLength: 0,
      hasMultiInstance: false,
      formData: {}
    }
  },
  watch: {
    'formData.userType': function(val, oldVal) {
      if (oldVal) {
        const types = ['assignee', 'candidateUsers', 'candidateGroups']
        types.forEach(type => {
          delete this.element.businessObject.$attrs[`flowable:${type}`]
          delete this.formData[type]
        })
      }
    },
    'formData.assignee': function(val) {
      if (this.formData.userType !== 'assignee') {
        delete this.element.businessObject.$attrs[`flowable:assignee`]
        return
      }
      this.updateProperties({ 'flowable:assignee': val })
    },
    'formData.candidateUsers': function(val) {
      if (this.formData.userType !== 'candidateUsers') {
        delete this.element.businessObject.$attrs[`flowable:candidateUsers`]
        return
      }
      this.updateProperties({ 'flowable:candidateUsers': val?.join(',') })
    },
    'formData.candidateGroups': function(val) {
      console.log(val)
      if (this.formData.userType !== 'candidateGroups') {
        delete this.element.businessObject.$attrs[`flowable:candidateGroups`]
        return
      }
      this.updateProperties({ 'flowable:candidateGroups': val?.join(',') })
    },
    'formData.async': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:async': val })
    },
    'formData.dueDate': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:dueDate': val })
    },
    'formData.formKey': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:formKey': val })
    },
    'formData.priority': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:priority': val })
    },
    'formData.skipExpression': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:skipExpression': val })
    },
    'formData.isForCompensation': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'isForCompensation': val })
    },
    'formData.triggerable': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:triggerable': val })
    },
    'formData.class': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:class': val })
    },
    'formData.autoStoreVariables': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:autoStoreVariables': val })
    },
    'formData.exclude': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:exclude': val })
    },
    'formData.ruleVariablesInput': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:ruleVariablesInput': val })
    },
    'formData.rules': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:rules': val })
    },
    'formData.resultVariable': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:resultVariable': val })
    }
  },
  created() {
    let cache = commonParse(this.element)
    cache = userTaskParse(cache)
    this.formData = cache
    this.computedExecutionListenerLength()
    this.computedTaskListenerLength()
    this.computedHasMultiInstance()
    this.loadEmp()
    this.loadGroup()
  },
  methods: {
    loadEmp() {
      this.$http({
        url: '/employee/queryAllList',
        method: 'get'
      }).then(res => {
        this.emps = res
      })
    },
    loadGroup() {
      this.$http({
        url: '/procdef/queryGroups',
        method: 'post'
      }).then(res => {
        this.groups = res
      })
    },
    computedExecutionListenerLength() {
      this.executionListenerLength = this.element.businessObject.extensionElements?.values
        ?.filter(item => item.$type === 'flowable:ExecutionListener').length ?? 0
    },
    computedTaskListenerLength() {
      this.taskListenerLength = this.element.businessObject.extensionElements?.values
        ?.filter(item => item.$type === 'flowable:TaskListener').length ?? 0
    },
    computedHasMultiInstance() {
      if (this.element.businessObject.loopCharacteristics) {
        this.hasMultiInstance = true
      } else {
        this.hasMultiInstance = false
      }
    },
    finishExecutionListener() {
      if (this.dialogName === 'executionListenerDialog') {
        this.computedExecutionListenerLength()
      }
      this.dialogName = ''
    },
    finishTaskListener() {
      if (this.dialogName === 'taskListenerDialog') {
        this.computedTaskListenerLength()
      }
      this.dialogName = ''
    },
    finishMultiInstance() {
      if (this.dialogName === 'multiInstanceDialog') {
        this.computedHasMultiInstance()
      }
      this.dialogName = ''
    }
  }
}
</script>
