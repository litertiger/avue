<template>
  <el-dialog :class="b('dialog')"
             lock-scroll
             :modal-append-to-body="false"
             append-to-body
             :fullscreen="$parent.isMobile"
             :title="t('crud.filterTitle')"
             :width="$parent.isMobile?'100%':''"
             :visible.sync="box">
    <el-row :span="24"
            :class="b('dialog', ['overflow'])">
      <div :class="b('filter-menu')">
        <el-button-group>
          <el-button type="primary"
                     :size="$parent.isMediumSize"
                     @click="handleAdd">{{t('crud.filter.addBtn')}}</el-button>
          <el-button type="primary"
                     :size="$parent.isMediumSize"
                     @click="handleClear">{{t('crud.filter.resetBtn')}}</el-button>
          <el-button type="primary"
                     :size="$parent.isMediumSize"
                     @click="handleValueClear">{{t('crud.filter.clearBtn')}}</el-button>
        </el-button-group>
      </div>
      <el-col :md="12"
              :xs="24"
              :sm="12"
              v-for="(column,index) in list"
              :key="index"
              :class="b('filter-item')">
        <avue-select v-model="column.text"
                     :dic="columnOption"
                     :props="columnProps"
                     :clearable="false"
                     @change="handleChange(column.text,index)"
                     :size="$parent.isMediumSize"
                     :class="b('filter-label')"></avue-select>
        <avue-select :class="b('filter-symbol')"
                     v-model="column.symbol"
                     :dic="symbolDic"
                     :clearable="false"
                     :size="$parent.isMediumSize"></avue-select>
        <form-temp :column="getColumnByIndex(columnList[index])"
                   :size="$parent.isMediumSize"
                   :class="b('filter-value')"
                   :dic="$parent.DIC[columnList[index].prop]"
                   :t="t"
                   :props="columnList[index].props || $parent.tableOption.props"
                   v-model="column.value">
        </form-temp>
        <el-button type="danger"
                   :class="b('filter-icon')"
                   size="mini"
                   @click="handleDelete(index)"
                   circle
                   icon="el-icon-minus"></el-button>
      </el-col>
    </el-row>
    <span slot="footer"
          class="dialog-footer">
      <el-button type="primary"
                 :size="$parent.isMediumSize"
                 @click="handleSubmit">{{t('crud.filter.submitBtn')}}</el-button>
      <el-button @click="box = false"
                 :size="$parent.isMediumSize">{{t('crud.filter.cancelBtn')}}</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { getSearchType, formInitVal } from "core/dataformat";
import locale from "../../core/common/locale";
import create from "core/create";
import { dateList } from "core/dataformat";
import formTemp from '../../core/components/form/index'
export default create({
  name: "crud",
  mixins: [locale],
  components: {
    formTemp
  },
  data () {
    return {
      box: false,
      formDefault: {},
      list: [],
      columnList: [],
      dateList: dateList,
      columnProps: {
        value: "prop"
      }
    };
  },
  computed: {
    symbolDic () {
      return [
        {
          label: "=",
          value: "="
        },
        {
          label: "≠",
          value: "≠"
        },
        {
          label: "like",
          value: "like"
        },
        {
          label: ">",
          value: ">"
        },
        {
          label: "≥",
          value: "≥"
        },
        {
          label: "<",
          value: "<"
        },
        {
          label: "≤",
          value: "≤"
        },
        {
          label: "∈",
          value: "∈"
        }
      ];
    },
    result () {
      let result = [];
      this.list.forEach(ele => {
        if (!this.validatenull(ele.value)) {
          result.push([ele.text, ele.symbol, ele.value]);
        }
      });
      return result;
    },
    columnObj () {
      return this.columnOption[0];
    },
    columnOption () {
      return this.$parent.propOption;
    }
  },
  created () {
    this.getSearchType = getSearchType;
    this.formDefault = formInitVal(this.columnOption).tableForm;
  },
  methods: {
    getColumnByIndex (column, index) {
      const ele = this.deepClone(column)
      ele.type = getSearchType(ele);
      ele.multiple = ["checkbox"].includes(column.type)
      return ele
    },
    handleDelete (index) {
      this.list.splice(index, 1);
      this.columnList.splice(index, 1);
    },
    handleClear () {
      this.list = [];
      this.columnList = [];
    },
    handleValueClear () {
      this.list.forEach((ele, index) => {
        this.$set(this.list[index], 'value', this.formDefault[ele.text]);
      });
    },
    handleGetColumn (prop) {
      return this.columnOption.find(ele => ele.prop === prop)
    },
    handleSubmit () {
      this.list.push({});
      this.list.splice(this.list.length - 1, 1);
      this.$parent.$emit("filter-change", this.result);
      this.box = false;
    },
    handleChange (prop, index) {
      const column = this.handleGetColumn(prop);
      this.columnList[index] = column;
      this.list[index].value = this.formDefault[prop];
    },
    handleAdd () {
      const len = this.list.length;
      const prop = this.columnObj.prop;
      const column = this.handleGetColumn(prop);
      this.columnList.push(column);
      this.list.push({
        text: prop,
        value: this.formDefault[prop],
        symbol: this.symbolDic[0].value
      });
    }
  }
});
</script>

