<template>
  <div>
    <el-form ref="dataForm" :model="formData" size="medium" label-width="100px" label-position="right">
      <el-form-item label="节点id">
        <el-input v-model="formData.id" />
      </el-form-item>
      <el-form-item label="执行监听器">
        <el-badge :value="executionListenerLength">
          <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
        </el-badge>
      </el-form-item>
      <el-form-item label="跳转条件">
        <el-input v-model="formData.conditionExpression" />
      </el-form-item>
      <el-form-item label="跳过表达式">
        <el-input v-model="formData.skipExpression" />
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
  import { commonParse, conditionExpressionParse } from '../../common/parseElement'
  export default {
    mixins: [mixinPanel, mixinExecutionListener],
    data() {
      return {
        formData: {}
      }
    },
    watch: {
      'formData.conditionExpression': function(val) {
        if (val) {
          const newCondition = this.modeler.get('moddle').create('bpmn:FormalExpression', { body: val })
          this.updateProperties({ conditionExpression: newCondition })
        } else {
          this.updateProperties({ conditionExpression: null })
        }
      },
      'formData.skipExpression': function(val) {
        if (val === '') val = null
        this.updateProperties({ 'flowable:skipExpression': val })
      }
    },
    created() {
      let cache = commonParse(this.element)
      cache = conditionExpressionParse(cache)
      this.formData = cache
    }
  }
</script>

<style></style>
