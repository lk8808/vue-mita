<template>
  <div>
    <el-form ref="dataForm" :model="formData" size="medium" label-width="100px" label-position="right">
      <el-form-item label="流程分类">
       <el-select v-model="formData.processCategory">
         <el-option v-for="item in categorys" :key="item.value" :label="item.label" :value="item.value" />
       </el-select>
      </el-form-item>
      <el-form-item label="流程标识key">
        <el-input v-model="formData.id" />
      </el-form-item>
      <el-form-item label="流程名称">
        <el-input v-model="formData.name" />
      </el-form-item>
      <el-form-item label="流程描述">
        <el-input v-model="formData.documentation" />
      </el-form-item>
      <el-form-item label="执行监听器">
        <el-badge :value="executionListenerLength">
          <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
        </el-badge>
      </el-form-item>
      <el-form-item label="信号定义">
        <el-badge :value="signalLength">
          <el-button size="small" @click="dialogName = 'signalDialog'">编辑</el-button>
        </el-badge>
      </el-form-item>
    </el-form>
    <executionListenerDialog
      v-if="dialogName === 'executionListenerDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishExecutionListener"
    />
    <signalDialog
      v-if="dialogName === 'signalDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishExecutionListener"
    />
  </div>
</template>

<script>
import mixinPanel from '../../common/mixinPanel'
import mixinExecutionListener from '../../common/mixinExecutionListener'
import signalDialog from './property/signal'
import { commonParse } from '../../common/parseElement'
export default {
  components: {
    signalDialog
  },
  mixins: [mixinPanel, mixinExecutionListener],
  data() {
    return {
      categorys: [
        {label: '办公', value: 'OA'},
        {label: '绩效', value: 'PRP'}
      ],
      signalLength: 0,
      formData: {}
    }
  },
  watch: {
    'formData.processCategory': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:processCategory': val })
    }
  },
  created() {
    this.formData = commonParse(this.element)
  },
  methods: {
    computedSignalLength() {
      this.signalLength = this.element.businessObject.extensionElements?.values?.length ?? 0
    },
    finishSignal() {
      if (this.dialogName === 'signalDialog') {
        this.computedSignalLength()
      }
      this.dialogName = ''
    }
  }
}
</script>

<style>

</style>
