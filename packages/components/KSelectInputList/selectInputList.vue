<!--
 * @Description: 多列选择 用于选择并且需要输入的表单
 * @Author: kcz
 * @Date: 2020-03-27 18:36:56
 * @LastEditors: kcz
 * @LastEditTime: 2022-11-02 22:27:21
 -->
<template>
  <a-form-model
    class="select-input-list-box"
    ref="dynamicValidateForm"
    layout="inline"
    :model="dynamicValidateForm"
  >
    <div v-for="(column, i) in record.columns" :key="i" class="list-col">
      <a-form-model-item class="w-auto">
        <CheckboxItem
          v-if="record.options.multiple"
          @change="onCheckboxChange($event, i)"
          :checked="dynamicValidateForm.domains[i].checked"
        >
          {{ column.label }}
        </CheckboxItem>
        <RadioItem
          v-else
          @change="onRadioChange($event, i)"
          :checked="dynamicValidateForm.domains[i].checked"
          >{{ column.label }}</RadioItem
        >
      </a-form-model-item>
      <KFormModelItem
        v-for="item in column.list"
        :key="item.key + '1'"
        :record="item"
        :config="config"
        :parentDisabled="disabled"
        :domains="dynamicValidateForm.domains[i]"
        :dynamicData="dynamicData"
        v-model="dynamicValidateForm.domains[i][item.model]"
        @input="handleInput"
      />
    </div>
  </a-form-model>
</template>

<script>
import KFormModelItem from "../KFormModelItem/KFormModelItem";
import { pluginManager } from "../../utils/index";
const CheckboxItem = pluginManager.getComponent("checkboxItem").component;
const RadioItem = pluginManager.getComponent("radioItem").component;
export default {
  name: "KBatch",
  props: ["record", "value", "dynamicData", "config", "parentDisabled"],

  components: {
    KFormModelItem,
    CheckboxItem,
    RadioItem
  },
  watch: {
    value: {
      // value 需要深度监听及默认先执行handler函数
      handler(val) {
        const initValue = val || [];
        if (!initValue.length) {
          this.record.columns.forEach(item => {
            const itemData = {};
            item.list.forEach(e => e.model && (itemData[e.model] = null));
            itemData.checked = false;
            itemData.value = item.value;
            itemData.label = item.label;
            initValue.push(itemData);
          });
        }

        this.dynamicValidateForm.domains = initValue;
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
    onCheckboxChange(e, index) {
      this.dynamicValidateForm.domains[index].checked = e.target.checked;
      this.handleInput();
    },
    onRadioChange(e, index) {
      this.dynamicValidateForm.domains.forEach(item => (item.checked = false));
      this.dynamicValidateForm.domains[index].checked = e.target.checked;
      this.handleInput();
    },
    handleInput() {
      this.$emit("change", this.dynamicValidateForm.domains);
    }
  },
  mounted() {
    this.handleInput();
  }
};
</script>
