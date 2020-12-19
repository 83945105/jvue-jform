<template>
  <div id="app">
    <j-render :data="formData"></j-render>
  </div>
</template>

<script>
  export default {
    name: 'app',
    data() {
      return {
        formData: {
          tag: 'j-el-form',
          children: [
            {
              tag: 'j-el-description'
            },
            {
              tag: 'j-el-editor'
            },
            {
              tag: 'j-el-icon-picker'
            },
            {
              tag: 'j-el-coordinate-picker'
            }
          ]
        }
      }
    }
  }
</script>

<style>
</style>
