<template>
  <div>
    <el-form ref="dataForm" :model="formData" size="medium" label-width="100px" label-position="right">
      <el-form-item label="节点id">
        <el-input v-model="formData.id" />
      </el-form-item>
      <el-form-item label="节点描述">
        <el-input v-model="formData.documentation" />
      </el-form-item>
      <el-form-item label="执行监听器">
        <el-badge :value="executionListenerLength">
          <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
        </el-badge>
      </el-form-item>
      <el-form-item v-if="showConfig.initiator" label="发起人">
        <el-input v-model="formData.initiator" />
      </el-form-item>
      <el-form-item v-if="showConfig.formKey" label="表单标识key">
        <el-input v-model="formData.formKey" />
      </el-form-item>
    </el-form>
    <executionListenerDialog
      v-if="dialogName === 'executionListenerDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishExecutionListener"
    />
  </div>
</template>

<script>
  import mixinPanel from '../../common/mixinPanel'
  import mixinExecutionListener from '../../common/mixinExecutionListener'
  import { commonParse } from '../../common/parseElement'
  export default {
    mixins: [mixinPanel, mixinExecutionListener],
    data() {
      return {
        formData: {}
      }
    },
    watch: {
      'formData.initiator': function(val) {
        if (val === '') val = null
        this.updateProperties({ 'flowable:initiator': val })
      },
      'formData.formKey': function(val) {
        if (val === '') val = null
        this.updateProperties({ 'flowable:formKey': val })
      }
    },
    created() {
      this.formData = commonParse(this.element)
    }
  }
</script>

<style>

</style>
