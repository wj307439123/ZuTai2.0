<!-- 自定义配置 -->
<template>
  <div>
   <el-form-item label="显示文字">
      <avue-input v-model="main.activeOption.name"></avue-input>
    </el-form-item>
    <el-form-item label="显示刻度值">
      <avue-switch v-model="main.activeOption.axisLabelShow"></avue-switch>
    </el-form-item>
    <el-form-item label="刻度字号">
      <avue-input-number v-model="main.activeOption.axisLabelFontSize"></avue-input-number>
    </el-form-item>
    <el-form-item label="指标名称字号">
      <avue-input-number v-model="main.activeOption.nameFontSize"></avue-input-number>
    </el-form-item>
    <el-form-item label="指标字号">
      <avue-input-number v-model="main.activeOption.valueFontSize"></avue-input-number>
    </el-form-item>
  </div>
</template>

<script>
export default {
  inject: ["main"]
}
</script>

<style>
</style>