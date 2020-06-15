<template>
  <div class="em-mock-expand">
    <h2>Method</h2>
    <p>{{mock.method}}</p>
    <h2>URL</h2>
    <p>{{mock.url}}</p>
    <h2>{{$t('p.detail.expand.description')}}</h2>
    <p>{{mock.description}}</p>
    <Tabs value="request" v-if="mock.parameters || mock.response_model">
      <Tab-pane class="mockItemPanelContainer" :label="$t('p.detail.expand.tab[0]')" name="request" v-if="mock.parameters">
        <Table :columns="columnsRequest" :data="request"></Table>
      </Tab-pane>
      <Tab-pane class="mockItemPanelContainer" :label="$t('p.detail.expand.tab[1]')" name="response" v-if="mock.response_model">
        <Collapse value="response-200">
          <Panel v-for="(i,n) in response" :key="n" :name="'response-'+i.code">
            status: {{i.code}} - {{i.message}}
            <div slot="content">
              <Table row-key="name" :columns="columnsResponseSchema" :data="responseSchema"></Table>
            </div>
          </Panel>
        </Collapse>
      </Tab-pane>
      <Tab-pane class="mockItemPanelContainer" label="Mock示例" name="responseExample" v-if="responseExample">
        <p>
          <pre>{{responseExample}}</pre>
        </p>
      </Tab-pane>
      <Tab-pane class="mockItemPanelContainer" label="Class Model" name="class" v-if="mock.response_model && entities.js.length">
        <Collapse>
          <Panel>
            TypeScript
            <div slot="content">
              <p>TS interface : to be finished</p>
            </div>
          </Panel>
          <Panel>
            JavaScript
            <div slot="content">
              <p v-for="(item, i) in entities.js" :key="i">
                <pre>{{item}}</pre>
              </p>
            </div>
          </Panel>
          <Panel>
            Objective-C
            <div slot="content">
              <p v-for="(item, i) in entities.oc" :key="i">
                <pre>{{item}}</pre>
              </p>
            </div>
          </Panel>
        </Collapse>
      </Tab-pane>
      <Tab-pane class="mockItemPanelContainer" label="Vuex Action" name="vuex" v-if="mock.response_model && entities.js.length">
        <Collapse value="1">
          <Panel name="1">
            Nuxt module
            <div slot="content">
              <vuex-nuxt :request="request" :entities="entities" :response="response" :mock="mock"></vuex-nuxt>
            </div>
          </Panel>
          <Panel name="2">
            Vuex module
            <div slot="content">
              <p>to be finished</p>
            </div>
          </Panel>
        </Collapse>
      </Tab-pane>
    </Tabs>
  </div>
</template>

<script>
import {
  getJavaScriptEntities,
  getObjectiveCEntities
} from 'swagger-parser-mock/lib/entity'
import jsBeautify from 'js-beautify/js/lib/beautify'
import DataTypeExpand from './data-type-expand'
import vuexNuxt from '../../components/vuexContainer/nuxt'

export default {
  components: { vuexNuxt },
  props: {
    mock: {}
  },
  data () {
    return {
      columnsRequest: [
        // {
        //   type: 'expand',
        //   width: 50,
        //   render: (h, params) => h(DataTypeExpand, { props: { data: params.row.parameter } })
        // },
        { title: this.$t('p.detail.expand.columnsRequest[0]'), key: 'name' },
        { title: this.$t('p.detail.expand.columnsRequest[1]'), key: 'description' },
        { title: this.$t('p.detail.expand.columnsRequest[2]'), key: 'paramType' },
        { title: this.$t('p.detail.expand.columnsRequest[3]'), key: 'dataType' },
        { title: 'required',render: (h, params) => h('span', params.row.parameter.required ) },
        { title: 'example',render: (h, params) => h('span', params.row.parameter.example ) },
      ],
      columnsResponse: [
        {
          type: 'expand',
          width: 50,
          render: (h, params) => h(DataTypeExpand, { props: { data: params.row.response } })
        },
        { title: this.$t('p.detail.expand.columnsResponse[0]'), key: 'code' },
        { title: this.$t('p.detail.expand.columnsResponse[1]'), key: 'message' }
      ],
      columnsResponseSchema: [
        {
          type: 'expand',
          width: 50,
          render: (h, params) => h('Table', { props: { columns:this.columnsResponseSchema ,data:params.row.children } })
        },
        { title: "name", key: 'name' },
        { title: "type", key: 'type' },
        { title: "format", key: 'format' },
        { title: "description", key: 'description' },
      ],
    }
  },
  computed: {
    request () {
      const parameters = this.mock.parameters ? JSON.parse(this.mock.parameters) : []
      return parameters.map(parameter => ({
        name: parameter.name,
        description: parameter.description || this.$t('p.detail.expand.defaultDescription'),
        paramType: parameter.in,
        dataType: this.getParamDataType(parameter),
        parameter: parameter
      }))
    },
    response () {
      const responseModel = this.mock.response_model ? JSON.parse(this.mock.response_model) : {}
      return Object.keys(responseModel).map(code => {
        const response = responseModel[code]
        return {
          code: code,
          message: response.message || response.description || this.$t('p.detail.expand.defaultDescription'),
          response: response
        }
      })
    },
    responseSchema(){
      if(this.response && this.response[0]){
        if(this.response[0].response.schema.type==='object'){
          return this.parseResponseObject(this.response[0].response.schema.properties)
        }else if(this.response[0].response.schema.type==='array'){
          return [
            {
              name:"list",
              type:"array",
              format:"[ ... ]",
              description:"列表",
              children:this.parseResponse(this.response[0].response.schema),
            }
          ]
        }
      }
    },
    responseExample(){
      if(this.response && this.response[0]){
        return this.response[0].response.example
      }
    },
    entities () {
      const res = this.response.filter(o => {
        const code = o.code.toString()
        return code === '200' || code === 'default'
      })[0]
      const response = res ? res.response : {}

      return {
        js: getJavaScriptEntities(response).map(o => jsBeautify.js_beautify(o, { indent_size: 2 })),
        oc: getObjectiveCEntities(response)
      }
    }
  },
  methods: {
    parseResponse(val){
      if(val.type==='array'){
        if(val.items.type==='array'){
          return this.parseResponseArray(val.items.items)
        } else if(val.items.type==='object'){
          return this.parseResponseObject(val.items.properties)
        } else {
          return [{
            ...val.items,
            name:val.items.name || "child_item",
            _disableExpand:true
          }]
      }
      } else if (val.type==='object' && val.properties){
        return this.parseResponseObject(val.properties)
      }
    },
    parseResponseArray(items){
      return  Object.keys(items).map(o=>{
        const hasChildren = ['array','object'].includes(items[o].type)
        return {
          name:o,
          type:items[o].type,
          format:items[o].format,
          description:items[o].description || '',
          _disableExpand:!hasChildren,
          children:hasChildren ? this.parseResponse(items[o]): []
        }
      })
    },
    parseResponseObject(items){
      return  Object.keys(items).map(o=>{
        const hasChildren = ['array','object'].includes(items[o].type)
        return {
          name:o,
          type:items[o].type,
          format:items[o].format,
          description:items[o].description || '',
          _disableExpand:!hasChildren,
          children:hasChildren ? this.parseResponse(items[o]): []
        }
      })
    },
    getParamDataType (parameter) {
      const { type, schema } = parameter
      if (type) return type
      if (schema && schema.type) {
        return schema.type === 'array' ? schema.items.type : schema.type
      }
    }
  },
}
</script>

<style scoped>
.mockItemPanelContainer{
  /* max-height: 50vh; */
  /* overflow-y: auto; */

  }
</style>