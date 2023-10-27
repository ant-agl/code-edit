<template>
  <n-message-provider>
    <CodeEditor />
  </n-message-provider>
</template>

<script>
import "vfonts/Lato.css";
import { NMessageProvider } from "naive-ui";
import CodeEditor from "@/components/CodeEditor";

export default {
  name: "App",
  components: {
    CodeEditor,
    NMessageProvider,
  },
};
</script>

<style lang="scss">
#app {
  font-family: v-sans, v-mono, other-fallbacks;
  height: 100vh;
}
.icon-btn {
  cursor: pointer;
  transition: 0.2s;
  opacity: 0.5;

  &:hover {
    opacity: 1;
  }
}
</style>
