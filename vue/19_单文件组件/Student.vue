<template>
  <div>
    <h1>学生姓名：{{ name }}</h1>
    <h1>学生年龄：{{ age }}</h1>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "zhangsan",
      age: 18,
    };
  },
};
</script>

<style>
</style>