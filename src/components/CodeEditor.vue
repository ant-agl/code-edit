<template>
  <n-layout has-sider class="code-editor">
    <n-layout-sider bordered>
      <FileList
        :files="files"
        :activeFile="activeFile"
        @addFile="addFile"
        @editTitle="editTitle"
        @deleteFile="deleteFile"
        @fileSelect="fileSelect"
      />
    </n-layout-sider>
    <n-layout class="code-editor__file">
      <n-layout-header bordered>
        <span>{{ titleActiveFile }}</span>
        <div class="code-editor__selectors">
          <n-select
            placeholder="Language"
            v-model:value="selectLang"
            :options="langs"
          />
          <n-select
            placeholder="Theme"
            v-model:value="selectTheme"
            :options="themes"
          />
        </div>
      </n-layout-header>
      <n-layout-content>
        <Codemirror
          v-if="files.length > 0"
          v-model="code"
          placeholder="Code..."
          :style="{ height: '100%' }"
          :tab-size="2"
          :extensions="extensions"
          @change="saveCode"
          ref="codemirror"
        />
        <p v-else class="code-editor__text" @click="addFile">Create a file</p>
      </n-layout-content>
    </n-layout>
  </n-layout>
</template>

<script>
import {
  NLayout,
  NLayoutSider,
  NLayoutHeader,
  NLayoutContent,
  NSelect,
  useMessage,
} from "naive-ui";
import { Codemirror } from "vue-codemirror";
import { javascript } from "@codemirror/lang-javascript";
import { html } from "@codemirror/lang-html";
import { css } from "@codemirror/lang-css";
import { json } from "@codemirror/lang-json";
import { oneDark } from "@codemirror/theme-one-dark";

import FileList from "@/components/FileList";

export default {
  name: "CodeEditor",
  setup: () => ({
    message: useMessage(),
  }),
  components: {
    NLayout,
    NLayoutSider,
    NLayoutHeader,
    NLayoutContent,
    NSelect,
    Codemirror,
    FileList,
  },
  data: () => ({
    files: [],
    activeFile: 0,
    code: "",
    selectTheme: "dark",
    langs: [
      {
        label: "txt",
        value: "txt",
      },
      {
        func: javascript,
        label: "JS",
        value: "js",
      },
      {
        func: html,
        label: "HTML",
        value: "html",
      },
      {
        func: css,
        label: "CSS",
        value: "css",
      },
      {
        func: json,
        label: "JSON",
        value: "json",
      },
    ],
    themes: [
      {
        label: "Light",
        value: "light",
      },
      {
        pack: oneDark,
        label: "Dark",
        value: "dark",
      },
    ],
  }),
  computed: {
    titleActiveFile() {
      // получаем название активного файла
      const f = this.files.find((f) => f.id == this.activeFile);
      return f ? f.title : "";
    },
    extensions() {
      // вычисляем свойства для редактора
      let arr = [];
      const lang = this.langs.find((l) => l.value == this.selectLang);
      if (lang.func) arr.push(lang.func());

      const theme = this.themes.find((p) => p.value == this.selectTheme);
      if (theme.pack) arr.push(theme.pack);

      return arr;
    },
    selectLang() {
      // автоматическое определение формата файлов
      let lang = "txt";
      const format = this.titleActiveFile.split(".").pop().toLowerCase();
      if (this.langs.find((lang) => lang.value == format)) {
        lang = format;
      }

      return lang;
    },
  },
  watch: {
    activeFile(id) {
      // меняем код при выборе файла
      const f = this.files.find((f) => f.id == id);
      this.code = f ? f.code : "";

      // Скролл содержимого файла наверх
      const codeContainer = document.querySelector(".cm-scroller");
      if (codeContainer) codeContainer.scrollTop = 0;
    },
    selectTheme(theme) {
      localStorage.theme = theme;
    },
  },
  mounted() {
    // Берем данные с хранилища
    if (localStorage.files) this.files = JSON.parse(localStorage.files);
    if (localStorage.activeFile) this.activeFile = +localStorage.activeFile;
    if (localStorage.theme) this.selectTheme = localStorage.theme;
  },
  methods: {
    addFile() {
      // Вычисяем id для файла
      const lastId = this.files
        .map((f) => f.id)
        .sort()
        .reverse()[0];
      const id = lastId ? lastId + 1 : 1;

      this.files.push({
        id,
        title: this.generateDefaultTitle(),
        code: "",
      });
      this.activeFile = id;
      this.updateStorage("files", this.files);
      this.updateStorage("activeFile", this.activeFile);

      this.message.success("Файл создан");
    },
    updateStorage(key, val, json = true) {
      // Обновление данных в локальном хранилище
      if (json) val = JSON.stringify(val);
      localStorage[key] = val;
    },
    generateDefaultTitle(fileType = "js") {
      // При создании файлов исключаем дубли в названии с помощью добавления индекса
      let i = 1;
      let defaultTitle = "script." + fileType;
      while (this.files.find((f) => f.title == defaultTitle)) {
        defaultTitle = `script-${i}.${fileType}`;
        i++;
      }
      return defaultTitle;
    },
    editTitle(data) {
      // Проверяем на ошибки в названии
      const isDouble = !!this.files.find((f) => f.title == data.title);
      if (data.title == "") {
        this.message.error("Название файла не может быть путсым");
        return false;
      }
      if (isDouble) {
        this.message.error("Файл с таким именем уже есть");
        return false;
      }

      this.files.find((f) => f.id == data.id).title = data.title;
      this.updateStorage("files", this.files);

      this.message.success("Файл переименован");
    },
    deleteFile(id) {
      const index = this.files.findIndex((f) => f.id == id);
      this.files.splice(index, 1);
      this.updateStorage("files", this.files);

      // если удаляем выбранный файл, то изменяем выбор
      if (this.activeFile == id && this.files.length)
        this.fileSelect(this.files[0].id);

      this.message.success("Файл удален");
    },
    fileSelect(id) {
      this.activeFile = id;
      this.updateStorage("activeFile", this.activeFile);
    },
    saveCode(code) {
      this.files.find((f) => f.id == this.activeFile).code = code;
      this.updateStorage("files", this.files);
    },
  },
};
</script>

<style lang="scss" scoped>
.code-editor {
  height: 100%;

  &__selectors {
    display: flex;
    gap: 10px;
    width: 50%;
  }

  &__text {
    padding: 24px;
    margin: 0;
    cursor: pointer;
  }
}
.n-layout-header {
  padding: 15px 24px;
  font-size: 16px;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.code-editor__file > :deep(.n-layout-scroll-container) {
  display: flex;
  flex-direction: column;
}
</style>
