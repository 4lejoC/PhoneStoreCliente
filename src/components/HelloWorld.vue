<template>
    <div class="d-flex">
    </div>
</template>
  
  <script>
  import { RouterLink } from 'vue-router';
  
  export default {
    name: 'HelloWorld',
    props: {
      msg: String,
      showMainContent: {
        type: Boolean,
        default: true
      }
    }
  };
  </script>
  
  <style scoped>

  </style>
  