<template>
  <el-select v-model="value" clearable placeholder="请选择" @change="dictChange">
    <el-option v-for="item in dicts" :key="item.dictid" :label="item.dictname" :value="item.dictid" />
  </el-select>
</template>

<script>
  export default {
    name: 'DictSelect',
    props: {
      dicttypeid: { // 数据字典类型id
        required: true,
        type: String
      },
      filterstr: { // 过滤
        type: String
      },
      default: { // 默认选中
        type: String
      },
      value: {
        type: String
      }
    },
    data() {
      return {
        dicts: []
      }
    },
    created() {
      var that = this
      this.$http({
        url: '/dict/queryDictsByDicttypeid?dicttypeid=' + this.dicttypeid,
        method: 'get'
      }).then(res => {
        if (that.filterstr) {
          this.dicts = res.bizdatas.filter(function(item) {
            return that.filterstr.indexOf(item.dictid) > -1
          })
        } else {
          this.dicts = res.bizdatas
        }
        if (this.default) {
          this.value = this.default
        }
      })
    },
    methods: {
      dictChange() {
        this.$emit('change', this.value)
      }
    }
  }
</script>
