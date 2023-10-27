<template>
  <div class="file-item" :class="{ active: active }" @click="fileSelect">
    <input
      type="text"
      class="file-item__title"
      v-model="title"
      ref="title"
      :readonly="!isEdit"
      @blur="saveTitle"
      @keyup.enter="$event.target.blur()"
    />
    <div class="file-item__btns">
      <n-icon size="20" class="icon-btn" @click="editTitle" v-show="!isEdit">
        <Edit16Regular />
      </n-icon>
      <n-icon size="20" class="icon-btn" color="#ff5f5f" @click="deleteFile">
        <Delete24Regular />
      </n-icon>
    </div>
  </div>
</template>

<script>
import { NIcon } from "naive-ui";
import { Edit16Regular, Delete24Regular } from "@vicons/fluent";

export default {
  name: "FileItem",
  components: { NIcon, Edit16Regular, Delete24Regular },
  props: {
    active: {
      type: Boolean,
      default: false,
    },
    file: Object,
  },
  data() {
    return {
      title: this.file.title,
      oldTitle: this.file.title,
      isEdit: false,
    };
  },
  methods: {
    editTitle() {
      this.isEdit = true;
      this.$refs.title.focus();
    },
    saveTitle() {
      this.isEdit = false;
      // Если название не изменилось, то не обновляем
      if (this.oldTitle == this.title) return false;

      this.$emit("editTitle", {
        id: this.file.id,
        title: this.title,
      });

      // Если ошибка, то восстанавливаем прошлое название
      this.title = this.file.title;
      this.oldTitle = this.title;
    },
    deleteFile() {
      this.$emit("deleteFile", this.file.id);
    },
    fileSelect() {
      this.$emit("fileSelect", this.file.id);
    },
  },
};
</script>

<style lang="scss" scoped>
.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #eee;
  padding: 15px 24px;
  cursor: pointer;
  transition: 0.2s;

  &:hover {
    background-color: #eeeeeeaa;
  }
  &.active {
    background-color: rgba(32, 128, 240, 0.16);
  }

  &__title {
    border: none;
    font-size: 16px;
    background-color: transparent;

    &:read-only {
      cursor: inherit;
    }
    &:focus {
      outline: none;
    }
  }
  &__btns {
    display: flex;
    gap: 3px;
    align-content: center;
    opacity: 0;
    transition: 0.2s;
  }
  &:hover &__btns {
    opacity: 1;
  }
}
</style>
