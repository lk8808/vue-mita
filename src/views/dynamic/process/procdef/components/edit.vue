<template>
  <div v-loading="loading_save" class="app-container">
    <bpmn-modeler
      ref="refNode"
      :xml="xml"
      :is-view="false"
      @save="saveModeler"
      @close="close"
    />
  </div>
</template>

<script>
  import bpmnModeler from '@/views/components/Flowable'
  import { xmlStr } from '@/views/dynamic/process/modules/resources/template.js'

  export default {
    components: { bpmnModeler },
    props: {
      bizdata: {
        required: true,
        type: Object
      }
    },
    data() {
      return {
        loading_save: false,
        xml: ''
      }
    },
    watch: {
      bizdata(val) {
        this.getModelDetail()
      }
    },
    mounted() {
      this.getModelDetail()
    },
    methods: {
      getModelDetail() {
        if (!this.bizdata.id) {
          this.xml = xmlStr
        } else {
          this.$http({
            url: '/procdef/getBpmn?procdefid=' + this.bizdata.id,
            method: 'post'
          }).then(res => {
            this.xml = res
          })
        }
      },
      saveModeler(data) {
        console.log(data)
        this.loading_save = true
        this.$http({
          url: '/procdef/deploy',
          method: 'post',
          data: {
            bpmnStr: data.xml
          }
        }).then(res => {
          this.loading_save = false
          this.$message({
            message: '提交成功',
            type: 'success'
          })
        })
      },
      close() {
        this.$emit('cancel')
      }
    }
  }
</script>
