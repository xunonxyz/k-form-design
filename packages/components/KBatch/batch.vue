<!--
 * @Description: 动态表格 用于批量填入数据
 * @Author: kcz
 * @Date: 2020-03-27 18:36:56
 * @LastEditors: kcz
 * @LastEditTime: 2022-10-31 20:40:41
 -->
<template>
  <a-form-model
    ref="dynamicValidateForm"
    layout="inline"
    :model="dynamicValidateForm"
  >
    <a-table
      class="batch-table"
      :pagination="false"
      :rowKey="record => record.key"
      :columns="columns"
      :dataSource="dynamicValidateForm.domains"
      bordered
      :scroll="{
        x: listLength * 190 + 80 + (!record.options.hideSequence ? 60 : 0),
        y: record.options.scrollY
      }"
    >
      <template
        v-for="item in record.list"
        :slot="item.key"
        slot-scope="text, record, index"
      >
        <KFormModelItem
          :key="item.key + '1'"
          :record="item"
          :config="config"
          :parentDisabled="disabled"
          :index="index"
          :domains="dynamicValidateForm.domains"
          :dynamicData="dynamicData"
          v-model="record[item.model]"
          @input="handleInput"
        />
      </template>
      <template slot="dynamic-opr-button" slot-scope="text, record">
        <a-icon
          title="复制添加"
          v-if="!disabled"
          type="copy-o"
          class="dynamic-opr-button"
          @click="copyDomain(record)"
        />
        <a-icon
          title="删除该行"
          v-if="!disabled"
          class="dynamic-opr-button"
          type="minus-circle-o"
          @click="removeDomain(record)"
        />
      </template>
    </a-table>
    <Button type="dashed" :disabled="disabled" @click="addDomain">
      <a-icon type="plus" />增加
    </Button>
  </a-form-model>
</template>

<script>
import KFormModelItem from "../KFormModelItem/KFormModelItem";
import { pluginManager } from "../../utils/index";
const Button = pluginManager.getComponent("aButton").component;

export default {
  name: "KBatch",

  props: ["record", "value", "dynamicData", "config", "parentDisabled"],

  components: {
    KFormModelItem,
    Button
  },
  watch: {
    value: {
      // value 需要深度监听及默认先执行handler函数
      handler(val) {
        this.dynamicValidateForm.domains = val || [];
      },
      immediate: true,
      deep: true
    }
  },
  data() {
    return {
      dynamicValidateForm: {
        domains: []
      }
    };
  },
  computed: {
    listLength() {
      return this.record.list.filter(item => !item.options.hidden).length;
    },
    columns() {
      const columns = [];
      if (!this.record.options.hideSequence) {
        columns.push({
          title: "序号",
          dataIndex: "sequence_index_number",
          width: "60px",
          align: "center",
          customRender: (text, record, index) => {
            return index + 1;
          }
        });
      }

      columns.push(
        ...this.record.list
          .filter(item => !item.options.hidden)
          .map((item, index) => {
            return {
              title: item.label,
              dataIndex: item.key,
              width: index === this.record.list.length - 1 ? "" : "190px",
              scopedSlots: { customRender: item.key }
            };
          })
      );

      columns.push({
        title: "操作",
        dataIndex: "dynamic-opr-button",
        fixed: "right",
        width: "80px",
        align: "center",
        scopedSlots: { customRender: "dynamic-opr-button" }
      });

      return columns;
    },
    disabled() {
      return this.record.options.disabled || this.parentDisabled;
    }
  },
  methods: {
    validationSubform() {
      let verification;
      this.$refs.dynamicValidateForm.validate(valid => {
        verification = valid;
      });
      return verification;
    },
    resetForm() {
      this.$refs.dynamicValidateForm.resetFields();
    },
    removeDomain(item) {
      const index = this.dynamicValidateForm.domains.indexOf(item);
      if (index !== -1) {
        this.dynamicValidateForm.domains.splice(index, 1);
      }
    },
    copyDomain(record) {
      this.dynamicValidateForm.domains.push({
        ...record,
        key: Date.now()
      });
      this.handleInput();
    },
    addDomain() {
      const data = {};
      this.record.list.forEach(item => {
        data[item.model] = item.options.defaultValue;
      });

      this.dynamicValidateForm.domains.push({
        ...data,
        key: Date.now()
      });
      this.handleInput();
    },
    handleInput() {
      this.$emit("change", this.dynamicValidateForm.domains);
    }
  }
};
</script>
