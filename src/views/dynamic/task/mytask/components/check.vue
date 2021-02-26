<template>
  <div>
    处理
  </div>
</template>

<script>

  export default {
    props: {
      task: {
        required: true,
        type: Object
      }
    },
    data() {

    },
    watch: {
      task(val) {
        
      }
    },
    created() {
    },
    methods: {
      init() {
        if (this.bizdata.province && this.bizdata.city && this.bizdata.country) {
          this.bizdata.region = [this.bizdata.province, this.bizdata.city, this.bizdata.country]
        }
      },
      save() {
        
      }
    }
  }
</script>
