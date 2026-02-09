
<template>
  <EmailEditor v-model="content" />
    <div class="" v-html="content">

    
  </div>
</template>

  <script setup>
import EmailEditor from "./EmailEditor.vue";
import { ref } from "vue";

const content =
  ref();
</script>
  