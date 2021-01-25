<template>
  <div class="app-container">
    <bpmn-modeler
      ref="refNode"
      :xml="xml"
      :is-view="true"
    />
  </div>
</template>

<script>
  import bpmnModeler from '@/views/components/Flowable'

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
        this.$http({
          url: '/procdef/getBpmn?procdefid=' + this.bizdata.id,
          method: 'post'
        }).then(res => {
          this.xml = res
        })
      }
    }
  }
</script>
