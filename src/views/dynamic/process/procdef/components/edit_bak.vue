<template>
  <div ref="container" class="app-container">
    <div ref="canvas" class="canvas" />
    <div id="js-properties-panel" class="panel" />
    <el-button-group class="btns">
      <el-button icon="el-icon-check" @click="save">保存</el-button>
      <el-button icon="el-icon-s-promotion" @click="submit">提交新版本</el-button>
      <el-button icon="el-icon-close" @click="$emit('cancel',$event)">关闭</el-button>
    </el-button-group>
  </div>
</template>

<script>
  import BpmnModeler from 'bpmn-js/lib/Modeler'
  import propertiesPanelModule from 'bpmn-js-properties-panel'
  import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
  import flowableModdle from '@/views/dynamic/process/modules/resources/flowable.json'
  // import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda'
  import customTranslate from '@/views/dynamic/process/modules/customTranslate/customTranslate.js'
  import { xmlStr } from '@/views/dynamic/process/modules/resources/template.js'

  export default {
    props: {
      bizdata: {
        required: true,
        type: Object
      }
    },
    data() {
      return {
        // bpmn建模器
        bpmnModeler: null,
        container: null,
        canvas: null
      }
    },
    watch: {
      bizdata(val) {
        this.loadDiapram()
      }
    },
    mounted() {
      this.init()
      this.loadDiapram()
    },
    methods: {
      init() {
        this.container = this.$refs.container
        this.canvas = this.$refs.canvas
        const customTranslateModule = {
          translate: ['value', customTranslate]
        }
        const canvas = this.$refs.canvas
        // 建模
        this.bpmnModeler = new BpmnModeler({
          container: canvas,
          // 添加控制板
          propertiesPanel: {
            parent: '#js-properties-panel'
          },
          // 添加属性面板，添加翻译
          additionalModules: [
            // 左边工具栏以及节点
            propertiesProviderModule,
            // 右侧工具栏
            propertiesPanelModule,
            customTranslateModule
          ],
          // 模块拓展，拓展activiti的描述
          moddleExtensions: {
            flowable: flowableModdle
          }
        })
      },
      loadDiapram() {
        if (!this.bizdata.id) {
          this.openDiagram(xmlStr)
        } else {
          this.$http({
            url: '/procdef/getBpmn?procdefid=' + this.bizdata.id,
            method: 'post'
          }).then(res => {
            this.openDiagram(res)
          })
        }
      },
      openDiagram(xml) {
        this.bpmnModeler.importXML(xml, (err) => {
          this.addEventBusListener()
        });
      },
      addEventBusListener() {
        // 监听 element
        const eventBus = this.bpmnModeler.get('eventBus')
        const eventTypes = ['element.click', 'element.changed', 'element.updateLabel']
        eventTypes.forEach(function(eventType) {
          eventBus.on(eventType, function(e) {
          })
        })
      },
      save() {
        this.bpmnModeler.saveXML({ format: true }, function(err, bpmnStr) {
          console.log(bpmnStr)
        })
      },
      submit() {
        var that = this
        this.bpmnModeler.saveXML({ format: true }, function(err, bpmnStr) {
          that.$http({
            url: '/procdef/deploy',
            method: 'post',
            data: {
              bpmnStr: bpmnStr
            }
          }).then(res => {
            that.$message({
              message: '提交成功',
              type: 'success'
            })
          })
        })
      }
    }
  }
</script>

<style lang="scss" scoped>

  @import '../../modules/styles/diagram-js.css';
  @import '../../modules/styles/vendor/bpmn-font/css/bpmn-embedded.css';
  @import '../../modules/styles/app.css';

  .app-container {
    position: relative;
    height: 100vh;
    width: 100vw;
    margin: 0;
  }

  .canvas{
    width: 100%;
    height: 100%;
  }
  .panel{
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    width: 350px;
    overflow: auto;
  }

  .btns {
    position: absolute;
    top: 2vh;
    right: 400px;
  }

</style>
