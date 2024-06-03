<!-- 组件的结构 -->
<template>
  <div class="demo">
    <h1>学校名称：{{ name }}</h1>
    <h1>学校地址：{{ address }}</h1>
    <button @click="showName">点我提示学校名</button>
  </div>
</template>

<!-- 组件交互相关的代码（数据、方法等） -->
<script>
// 当暴露的东西是一个的时候，用默认暴露会简化语法
export default {
  name: "School",
  data() {
    return {
      name: "tsing",
      address: "beijing",
    };
  },
  methods: {
    showName() {
      alert(this.name);
    },
  },
};
</script>

<!-- 组件的样式 -->
<style>
.demo {
  background-color: aquamarine;
}
</style>