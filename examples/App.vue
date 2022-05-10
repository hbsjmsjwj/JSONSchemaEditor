<template>
  <div id="app">
    <div class="title"></div>
    <div class="desc">
      <div>
        A json-schema editor of high efficient and easy-to-use, base on Vue.
        <a @click="visible = true">import json</a>
      </div>
    </div>
    <div class="container">
      <codemirror class="code" v-model="jsonStr" :readOnly="false" />
      <json-schema-editor class="schema" :value="tree" lang="zh_CN" :custom="false" />
    </div>
    <a-modal v-model="visible" title="import json" width="800px" height="600x" @ok="handleImportJson">
      <div class="code-container">
        <codemirror class="code" v-model="importJson" :readOnly="false" />
      </div>
    </a-modal>
  </div>
</template>

<script>
import Codemirror from './components/Codemirror.vue'
import GenerateSchema from 'generate-schema'
export default {
  name: 'App',
  components: { Codemirror },
  computed: {
    jsonStr: {
      get: function () {
        return JSON.stringify(this.tree, null, 2)
      },
      set: function (newVal) {
        this.tree = JSON.parse(newVal)
      },
    },
  },
  data() {
    return {
      importJson: '',
      visible: false,
      tree: {
        root: {
          type: 'object',
          title: '条件',
          properties: {
            name: {
              type: 'string',
              title: '名称',
              maxLength: 10,
              minLength: 2,
              __disabled: ['key', 'required', 'type', 'description', 'addChild', 'removeNode'],
            },
            appId: {
              type: 'integer',
              title: '应用ID',
            },
            credate: {
              type: 'string',
              title: '创建日期',
              format: 'date',
            },
            testObj: {
              type: 'object',
              title: '测试对象',
              __disabled: ['addChild'],
              properties:{
                name: {
                  type: 'string',
                  title: '名称',
                  maxLength: 10,
                  minLength: 2,
                  __disabled: ['key', 'required', 'type', 'description', 'addChild', 'removeNode'],
                },
              }
            },
            // testObj2: {
            //   type: 'object',
            //   title: '测试对象2',
            //   __disabled: ['removeNode'],
            // },
          },
          required: ['name', 'appId', 'credate'],
        },
      },
      test: {
        type: 'object',
        properties: {
          code: {
            type: 'number',
          },
          data: {
            type: 'object',
            properties: {
              receiving_num: {
                type: 'object',
                properties: {
                  ot_num: {
                    type: 'number',
                    description: '海外运输中数量 overseas_transport',
                  },
                  oo_num: {
                    type: 'number',
                    description: '海外操作中数量 overseas_operating',
                  },
                },
                description: '入库单数量',
              },
              list: {
                type: 'array',
                items: {
                  type: 'object',
                  properties: {
                    gc_rv_sign_status: {
                      type: 'string',
                      description: '海外签收状态 海外仓签收状态 0未签收 1签收中 2签收完成',
                    },
                    clearance: {
                      type: 'string',
                      description: '清关方式参照wiki',
                    },
                    customs: {
                      type: 'string',
                      description: '报关方式参照wiki',
                    },
                  },
                },
                description: '列表数据',
              },
              total: {
                type: 'integer',
                description: '数据总条数',
              },
            },
            required: [],
          },
          message: {
            type: 'string',
          },
        },
        required: ['code', 'message'],
      },
    }
  },
  mounted() {
    this.changeData(this.tree.root)
   let gettype=Object.prototype.toString
    console.log('**********************************',gettype.call(''))
  },
  methods: {
    changeData(data) {
      // const keys = data?.type ==='object'
      for (let key in data) {
        if (data[key] === 'object' && data.properties) {
          this.changeData(data.properties)
        } else if (data[key] === 'array') {
          this.changeData(data.items.properties)
        } else {
          console.log('===========data key=', key, typeof data[key])
          const t = typeof data[key]
          console.log(t)
          if (t === 'object'&&data[key]!=='__disabled') {
            data[key]['__disabled'] = ['key', 'required', 'type', 'description', 'addChild', 'removeNode']
          }
        }
      }
    },
    handleImportJson() {
      const t = GenerateSchema.json(JSON.parse(this.importJson))
      delete t.$schema
      this.tree.root = t
      this.visible = false
    },
  },
}
</script>
<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
.title {
  text-align: center;
  font-size: 40px;
  font-weight: bold;
  height: 100px;
  line-height: 100px;
}
.desc {
  padding: 20px;
  width: 80vw;
  min-width: 800px;
  margin: auto;
  padding: 0 3em;
  font-size: 1.2em;
}
.container {
  display: flex;
  padding: 20px;
  width: 80vw;
  min-width: 800px;
  justify-content: center;
  height: calc(100vh - 150px);
  margin: auto;
}
.code-container {
  max-height: 600px;
  overflow: auto;
}
.schema {
  margin-left: 20px;
  width: 50%;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  padding: 12px;
}
.CodeMirror {
  height: 100% !important;
}
.vue-codemirror {
  flex: 1;
  margin: 0 24px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  min-height: 300px;
  overflow: auto;
  border-radius: 6px;
}
</style>
