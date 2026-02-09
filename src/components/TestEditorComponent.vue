
<template>
  <EmailEditor v-model="content" />


    

</template>

  <script setup>
import EmailEditor from "./EmailEditor.vue";
import { ref } from "vue";

const content =
  ref();
</script>
  