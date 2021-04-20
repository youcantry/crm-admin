<template>
  <div>
    <el-button @click="getPorduct">点击我</el-button>
  </div>
</template>

<script>

import {fetchList } from '@/api/media'
export default {
  name: 'education',

  data() {
    return {

    }
   
  },
     methods:{
     getPorduct(){
      
      console.log(fetchList());
    }
   }
 
}
</script>

<style lang="scss" scoped>
.documentation-container {
  margin: 50px;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;

  .document-btn {
    flex-shrink: 0;
    display: block;
    cursor: pointer;
    background: black;
    color: white;
    height: 60px;
    padding: 0 16px;
    margin: 16px;
    line-height: 60px;
    font-size: 20px;
    text-align: center;
  }
}
</style>
