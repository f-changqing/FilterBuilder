<template>
  <div class="and-or-template col-xs-12" :class="isFirst ? 'and-or-first' : '' ">
    <div class="form-group and-or-top col-xs-12">
      <div class="col-xs-5" style="padding: 0">
        <button class="btn btn-xs btn-purple-outline btn-radius btn-tb" :class=" isAnd ? 'btn-purple-outline-focus' : '' " @click.prevent="clickAnd">
                        &nbsp;和&nbsp;
                      </button>
        <button class="btn btn-xs btn-purple-outline btn-radius btn-tb" :class=" !isAnd ? 'btn-purple-outline-focus' : '' " @click.prevent="clickOr">
                        &nbsp;或&nbsp;
                      </button>
      </div>
      <div class="col-xs-7 btn-and-or">
        <button v-if="!isFirst" class="btn btn-xs btn-purple pull-right" @click.prevent="deleteSelf()">
                        <i class="fa fa-fw fa-close"></i>
                      </button>
        <button class="btn btn-xs btn-purple pull-right" @click.prevent="addGroup"> + ( 组 ) </button>
        <button class="btn btn-xs btn-purple add-rule pull-right" @click.prevent="addRule"> + 条件 </button>
      </div>
    </div>
    <rule v-for="(rule, index) in rules" ref="rules" :options="options" :key="rule" @delete-rule="deleteRule(index)">
    </rule>
    <and-or class="and-or-offset col-xs-11" v-for="(group, index) in groups" ref="groups" :options="options" :key="group" @delete-group="deleteGroup(index)">
    </and-or>
  </div>
</template>

<script>
  import Rule from './Rule'
  export default {
    name: 'andOr',
    components: {
      Rule
    },
    props: {
      options: {
        type: Object
      },
      isFirst: {
        type: Boolean,
        default: false
      }
    },
    created() {
      //this.addRule();//不需要组件首次加载，存在一个条件
    },
    data() {
      return {
        isAnd: true,
        groups: [],
        rules: [],
        haveparams: false
      }
    },
    methods: {
      clickAnd() {
        this.isAnd = true;
      },
      clickOr() {
        this.isAnd = false;
      },
      addRule() {
        var id = this.generateId();
        this.rules.push(id);
      },
      addGroup() {
        var id = this.generateId();
        this.groups.push(id);
      },
      deleteSelf() {
        this.$emit('delete-group');
      },
      deleteRule(index) {
        this.rules.splice(index, 1);
      },
      deleteGroup(index) {
        this.groups.splice(index, 1);
      },
      queryFormStatus() {
        var query = {};
        var rules = this.$refs.rules || {};
        var groups = this.$refs.groups || {};
        var i, j;
        query['condition'] = this.isAnd ? ' AND ' : ' OR ';
        query['rules'] = [];
        for (i = 0; i < rules.length; i++) {
          let temrule = rules[i].queryFormStatus();
          if (temrule['key'] != -1 && temrule['value'] != '') {
            query.rules.push(rules[i].queryFormStatus());
          }
        }
        for (j = 0; j < groups.length; j++) {
          let temgroup = groups[j].queryFormStatus();
          if (temgroup.rules.length != 0) {
            query.rules[query.rules.length] = temgroup;
          }
        }
        return query;
      },
      queryFormSQL() {
        this.haveparams = false;
        return this.getSQLFromCondition(this.queryFormStatus());
      },
      queryFormSQLParams() {
        this.haveparams = true;
        return this.getSQLFromCondition(this.queryFormStatus());
      },
      getSQLFromCondition(obj) {
        let res = this.haveparams ? {
          sql: '',
          params: []
        } : {
          sql: ''
        }
        for (let index = 0; index < obj.rules.length; index++) {
          let element = obj.rules[index];
          if (index != 0) {
            res.sql += obj.condition;
          }
          if (element.rules) {
            let temres = this.getSQLFromCondition(element);
            res.sql += " ( " + temres.sql + " ) ";
            if (this.haveparams) {
              res.params.push(temres.params);
            }
          } else {
            res.sql += this.haveparams ? element.key + element.operator + "? " : element.key + element.operator + " '" + (element.ispara ? element.defaultvalue : element.value) + "' ";
            if (this.haveparams) {
              res.params.push({
                name: element.key,
                label: element.label,
                value: element.value,
                ispara: element.ispara,
                defaultvalue: element.defaultvalue,
                type:element.type
              })
            }
          }
        }
        return res;
      },
      fillFormStatus(data) {
        var i, len;
        var group = this;
        group.rules = [];
        group.groups = [];
        if (data) {
          group.isAnd = /and/i.test(data.condition);
          len = data.rules.length;
          for (i = 0; i < len; i++) {
            if (data.rules[i].condition) {
              group.groups.push(group.generateId());
              (function(i, index) {
                group.$nextTick(function() {
                  group.$refs.groups[index].fillFormStatus(data.rules[i]);
                });
              })(i, group.groups.length - 1);
            } else {
              group.rules.push(group.generateId());
              (function(i, index) {
                group.$nextTick(function() {
                  group.$refs.rules[index].fillRuleStatus(data.rules[i]);
                });
              })(i, group.rules.length - 1);
            }
          }
        }
      },
      generateId() {
        return 'xxxxxxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
          var r = Math.random() * 16 | 0,
            v = c == 'x' ? r : (r & 0x3 | 0x8);
          return v.toString(16);
        });
      }
    }
  }
</script>

<style>
  .and-or-template {
    padding: 8px;
    position: relative;
    border-radius: 3px;
    border: 1px solid #00958f;
    border-top: 3px solid #d2d6de;
    margin-bottom: 20px;
    /* width: 100%; */
    box-shadow: 0 1px 1px rgba(0, 0, 0, 0.1);
    border-top-color: #00958f;
    background-color: rgba(255, 255, 255, 0.9);
    width: 95%;
    margin-left: 2.5%;
  }
  .and-or-template:before,
  .and-or-template:after {
    content: '';
    position: absolute;
    left: -17px;
    width: 16px;
    height: calc(50% + 18px);
    border-color: #008983;
    border-style: solid;
  }
  .and-or-template:before {
    top: -18px;
    border-width: 0 0 2px 2px;
  }
  .and-or-template:after {
    top: 50%;
    border-width: 0 0 0 2px;
  }
  .and-or-first:before,
  .and-or-first:after,
  .and-or-template:last-child:after {
    border: none;
  }
  .and-or-top,
  .btn-and-or {
    padding: 0;
  }
  .btn-and-or button {
    margin-left: 4px;
  }
  .and-or-offset {
    margin-left: 30px;
  }
  .btn-tb {
    float: left;
    margin-left: 5px;
  }
</style>
