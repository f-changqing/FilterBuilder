<template>
  <div>
    <h1 class="title">
      <img src="./assets/logo.png" class="logo"> Vue-Filter-Builder
      <small class="version"></small>
    </h1>
    <div class="col-xs-8 col-xs-offset-2" style="margin-top: 40px">
      <and-or :options="options" :isFirst="isFirst" ref="andOr"></and-or>
      <div class="pull-right">
        <button class="btn btn-warning" @click.prevent="resetFilter">重置</button>
        <button class="btn btn-success" @click.prevent="fetchQuery">获取JSON对象</button>
        <button class="btn btn-success" @click.prevent="sqlQuery(false)">获取SQL语句</button>
        <button class="btn btn-success" @click.prevent="sqlQuery(true)">获取SQL语句(带参数)</button>
        <button class="btn btn-info" @click.prevent="showModal(false)">复原</button>
        <button class="btn btn-info" @click.prevent="showModal(true)">测试复原</button>
      </div>
    </div>
    <modal :show="showQueryModal" @hide="showQueryModal = false" @confirm="showQueryModal = false">
      <h4 class="modal-title" slot="header">{{title}}</h4>
      <div class="box-body" slot="body">
        <pre class="filter-fetched-query">{{fetchedQuery}}</pre>
      </div>
    </modal>
    <modal :show="showFillModal" @hide="showFillModal = false" @confirm="fillQuery">
      <h4 class="modal-title" slot="header">复原</h4>
      <div class="box-body" slot="body">
        <textarea class="filter-filled-query" v-model="filledQuery"></textarea>
      </div>
    </modal>
  </div>
</template>

<script>
  import AndOr from './components/AndOR'
  import Modal from './components/Modal.vue'
  export default {
    name: 'app',
    components: {
      AndOr,
      Modal
    },
    data() {
      return {
        options: {
          keys: [{
            label: '--选择--',
            id: -1
          }, {
            label: '姓名',
            id: 'name',
            type: 'String'
          }, {
            label: '年龄',
            id: 'age',
            type: 'Int'
          },
          {
            label: '身高',
            id: 'height',
            type: 'Int'
          }],
          defaultfieldvalue: {
            label: ['小明', '小力', '小张'],
            age: [12, 13, 14],
            height:[]
          }
        },
        isFirst: true,
        showQueryModal: false,
        showFillModal: false,
        fetchedQuery: '',
        filledQuery: '',
        title: ''
      }
    },
    mounted() {
      this.$nextTick(function() {
        this.init()
      });
    },
    methods: {
      init() {
        this.$refs.andOr.addRule();
        // this.$refs.andOr.addGroup();
        // this.$refs.andOr.addGroup();
      },
      sqlQuery(flag) {
        this.title = '获取SQL语句';
        this.showQueryModal = true;
        let res = flag ? this.$refs.andOr.queryFormSQLParams() : this.$refs.andOr.queryFormSQL();
        this.fetchedQuery = res.sql + (flag ? ('\n' + JSON.stringify(res.params, null, 4)) : ''); //JSON.stringify(this.$refs.andOr.queryFormSQL(), null, 4);
      },
      fetchQuery() {
        this.title = '获取JSON对象';
        this.showQueryModal = true;
        this.fetchedQuery = JSON.stringify(this.$refs.andOr.queryFormStatus(), null, 4);
      },
      fillQuery() {
        var query = JSON.parse(this.filledQuery);
        this.$refs.andOr.fillFormStatus(query);
        this.showFillModal = false;
      },
      showModal(flag) {
        if (flag) {
          this.$refs.andOr.fillFormStatus({
            "condition": " AND ",
            "rules": [{
                "key": "name",
                "label": "姓名",
                "operator": " = ",
                "value": "?",
                "type": "String",
                "ispara": true,
                "defaultvalue": "小明"
              },
              {
                "key": "age",
                "label": "年龄",
                "operator": " = ",
                "value": "?",
                "type": "Int",
                "ispara": true,
                "defaultvalue": 12
              },
              {
                "condition": " AND ",
                "rules": [{
                    "key": "name",
                    "label": "姓名",
                    "operator": " = ",
                    "value": "name2",
                    "type": "String",
                    "ispara": false,
                    "defaultvalue": ""
                  },
                  {
                    "key": "age",
                    "label": "年龄",
                    "operator": " = ",
                    "value": "2",
                    "type": "Int",
                    "ispara": false,
                    "defaultvalue": ""
                  },
                  {
                    "condition": " AND ",
                    "rules": [{
                        "key": "name",
                        "label": "姓名",
                        "operator": " = ",
                        "value": "?",
                        "type": "String",
                        "ispara": true,
                        "defaultvalue": "小明"
                      },
                      {
                        "key": "age",
                        "label": "年龄",
                        "operator": " = ",
                        "value": "?",
                        "type": "Int",
                        "ispara": true,
                        "defaultvalue": 12
                      },
                      {
                        "condition": " AND ",
                        "rules": [{
                            "condition": " AND ",
                            "rules": [{
                                "key": "name",
                                "label": "姓名",
                                "operator": " = ",
                                "value": "name4",
                                "type": "String",
                                "ispara": false,
                                "defaultvalue": ""
                              },
                              {
                                "key": "age",
                                "label": "年龄",
                                "operator": " = ",
                                "value": "4",
                                "type": "Int",
                                "ispara": false,
                                "defaultvalue": ""
                              }
                            ]
                          },
                          {
                            "condition": " AND ",
                            "rules": [{
                                "key": "name",
                                "label": "姓名",
                                "operator": " = ",
                                "value": "name5",
                                "type": "String",
                                "ispara": false,
                                "defaultvalue": ""
                              },
                              {
                                "key": "age",
                                "label": "年龄",
                                "operator": " = ",
                                "value": "5",
                                "type": "Int",
                                "ispara": false,
                                "defaultvalue": ""
                              }
                            ]
                          }
                        ]
                      }
                    ]
                  },
                  {
                    "condition": " AND ",
                    "rules": [{
                        "key": "name",
                        "label": "姓名",
                        "operator": " = ",
                        "value": "name6",
                        "type": "String",
                        "ispara": false,
                        "defaultvalue": ""
                      },
                      {
                        "key": "age",
                        "label": "年龄",
                        "operator": " = ",
                        "value": "6",
                        "type": "Int",
                        "ispara": false,
                        "defaultvalue": ""
                      }
                    ]
                  }
                ]
              },
              {
                "condition": " AND ",
                "rules": [{
                    "key": "name",
                    "label": "姓名",
                    "operator": " = ",
                    "value": "name7",
                    "type": "String",
                    "ispara": false,
                    "defaultvalue": ""
                  },
                  {
                    "key": "age",
                    "label": "年龄",
                    "operator": " = ",
                    "value": "7",
                    "type": "Int",
                    "ispara": false,
                    "defaultvalue": ""
                  }
                ]
              }
            ]
          });
          return;
        }
        this.showFillModal = true;
        this.filledQuery = '';
      },
      resetFilter() {
        this.$refs.andOr.groups = [];
        this.$refs.andOr.rules = [];
      }
    }
  }
</script>

<style>
  body {
    background: linear-gradient(to left bottom, rgb(179, 213, 255) 0%, rgb(255, 183, 179) 100%);
  }
  .title {
    text-align: center;
    margin-top: 80px;
  }
  .logo {
    height: 4.375rem;
    margin-right: 1.25rem;
    vertical-align: middle;
    display: inline-block;
  }
  .filter-fetched-query {
    max-height: 460px;
  }
  .filter-filled-query {
    width: 100%;
    height: 460px;
  }
</style>
