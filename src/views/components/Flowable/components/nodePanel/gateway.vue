<template>
  <div>
    <el-form ref="dataForm" :model="formData" size="medium" label-width="100px" label-position="right">
      <el-form-item label="节点id">
        <el-input v-model="formData.id" />
      </el-form-item>
      <el-form-item label="节点名称">
        <el-input v-model="formData.name" />
      </el-form-item>
      <el-form-item label="节点描述">
        <el-input v-model="formData.documentation" />
      </el-form-item>
      <el-form-item label="执行监听器">
        <el-badge :value="executionListenerLength">
          <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
        </el-badge>
      </el-form-item>
      <el-form-item label="异步">
        <el-switch v-model="formData.async" active-text="是" inactive-text="否" />
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
      'formData.async': function(val) {
        if (val === '') val = null
        this.updateProperties({ 'flowable:async': val })
      }
    },
    created() {
      this.formData = commonParse(this.element)
    }
  }
</script>

