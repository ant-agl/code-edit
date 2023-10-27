<template>
  <div class="file-list">
    <div class="file-list__head">
      <n-icon size="25" class="icon-btn" @click="addFile"
        ><Add16Regular
      /></n-icon>
    </div>
    <div class="file-list__body">
      <FileInput
        v-for="file in files"
        :key="file.id"
        :file="file"
        :active="file.id == activeFile"
        @editTitle="editTitle"
        @deleteFile="deleteFile"
        @fileSelect="fileSelect"
      />
    </div>
  </div>
</template>

<script>
import { NIcon } from "naive-ui";
import { Add16Regular } from "@vicons/fluent";
import FileInput from "@/components/FileItem";

export default {
  name: "FileList",
  components: { NIcon, Add16Regular, FileInput },
  props: {
    activeFile: Number,
    files: {
      type: Array,
      default: () => [],
    },
  },
  methods: {
    addFile() {
      this.$emit("addFile");
    },
    editTitle(data) {
      this.$emit("editTitle", data);
    },
    deleteFile(id) {
      this.$emit("deleteFile", id);
    },
    fileSelect(id) {
      this.$emit("fileSelect", id);
    },
  },
};
</script>

<style lang="scss" scoped>
.file-list {
  &__head {
    padding: 15px 24px;
    display: flex;
    justify-content: flex-end;
    border-bottom: 1px solid #eee;
  }
}
</style>
