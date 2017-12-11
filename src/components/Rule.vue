<template>
  <div class="form-group and-or-rule col-xs-12">
    <div class="col-xs-3">
      <select class="form-control input-sm" v-model="key">
                            <option v-for="option in options.keys" :value="option.id" :key="option.id">
                              {{option.label}}
                            </option>
                          </select>
    </div>
    <div class="col-xs-3">
      <select class="form-control input-sm" v-model="operator">
                            <option v-for="option in operators" :value="option.id" :key="option.id">
                              {{option.name}}
                            </option>
                          </select>
    </div>
    <div class="col-xs-3">
      <label class="sr-only">值</label>
      <input v-if="!ispara" type="text" class="form-control input-sm" :disabled="ispara" v-model="value" placeholder="值">
      <select v-else class="form-control input-sm" v-model="defaultvalue">
          <option v-for="option in fieldvalueslist" :value="option" :key="option">
            {{option}}
          </option>
        </select>
    </div>
    <button class="btn btn-xs btn-purple add-rule pull-right btn-param" @click="paramselect" v-if="options.param"> {{ispara?'可传参':'不可传参'}} </button>
    <!-- <button class="btn btn-xs btn-purple-outline btn-radius btn-purple-round" @click.prevent="deleteSelf()">
                          <i class="fa fa-fw fa-close"></i>
                        </button> -->
    <button class="btn btn-xs btn-purple add-rule pull-right" @click.prevent="deleteSelf()"> 删除 </button>
  </div>
</template>

<script>
  export default {
    name: 'rule',
    props: ['options'],
    watch: {
      'options.keys.options' () {
        this.key = -1;
      },
      'options.conditions.options' () {
        this.condition = -1;
      },
      'key' () {
        if (this.key != -1) {
          if (this.options.defaultfieldvalue && this.ispara) {
            this.setdefaultvalue();
          }
          for (const iterator of this.options.keys) {
            if (this.key == iterator.id) {
              this.label = iterator.label;
            }
          }
        }
      }
    },
    data() {
      return {
        ispara: false,
        key: -1,
        label: '',
        operator: ' = ',
        value: '',
        type: 'String',
        defaultvalue: '未设置',
        fieldvalueslist: [],
        operators: [{
          name: '等于',
          id: ' = '
        }, {
          name: '不等于',
          id: ' != '
        }, {
          name: '大于',
          id: ' > '
        }, {
          name: '小于',
          id: ' < '
        }, {
          name: '大于等于',
          id: ' >= '
        }, {
          name: '小于等于',
          id: ' <= '
        }]
      }
    },
    methods: {
      deleteSelf() {
        this.$emit('delete-rule');
      },
      queryFormStatus() {
        for (let index = 0; index < this.options.keys.length; index++) {
          const element = this.options.keys[index];
          if (element.id == this.key) {
            this.type = element.type;
          }
        }
        return {
          'key': this.key,
          'label': this.label,
          'operator': this.operator,
          'value': this.value,
          'type': this.type,
          'ispara': this.ispara,
          'defaultvalue': this.ispara?this.defaultvalue:''
        }
      },
      fillRuleStatus(data) {
        this.key = data.key;
        this.operator = data.operator;
        this.value = data.value;
        this.ispara=data.ispara;
        this.label=data.label;
        this.type=data.type;
        this.defaultvalue=data.defaultvalue;
        if (data.ispara) {
          this.setdefaultvalue();
        }
      },
      paramselect() {
        this.ispara = !this.ispara;
        if (this.ispara) {
          this.value = '?';
          if (this.key!=-1) {
            this.setdefaultvalue();
          }          
        } else {
          this.value = '';
        }
      },
      setdefaultvalue() {
        this.fieldvalueslist = this.options.defaultfieldvalue[this.key];
        if (this.fieldvalueslist.length > 0) {
          this.defaultvalue = this.fieldvalueslist[0];
        }else{
          this.defaultvalue='未设置';
        }
      }
    }
  }
</script>

<style>
  .and-or-rule {
    position: relative;
    height: 30px;
    margin-left: 15px !important;
    padding-left: 0;
  }
  .and-or-rule:before,
  .and-or-rule:after {
    content: '';
    position: absolute;
    left: -1px;
    width: 16px;
    height: calc(50% + 15px);
    border-color: #008983;
    border-style: solid;
  }
  .and-or-rule:before {
    top: -15px;
    border-width: 0 0 2px 2px;
  }
  .and-or-rule:after {
    top: 50%;
    border-width: 0 0 0 2px;
  }
  .and-or-rule:last-child:after {
    border: none;
  }
  .btn-param {
    margin-left: 5px;
  }
</style>
