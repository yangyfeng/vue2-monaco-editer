<template>
  <div class="editer-container" ref="container"></div>
</template>
<script>
import { loader } from "vue-m-loader";
const conf = {
  // 启用代码折叠默认值为true。
  folding: true,
  // 语言 
  // ["apex", "azcli", "bat", "c", "clojure", "coffeescript", 
  // "cpp", "csharp", "csp", "css", "dockerfile", "fsharp", "go", "graphql", 
  // "handlebars", "html", "ini", "java", "javascript", "json", "kotlin", "less", 
  // "lua", "markdown", "msdax", "mysql", "objective-c", "pascal", "perl", "pgsql", "php", 
  // "plaintext", "postiats", "powerquery", "powershell", "pug", "python", "r", "razor", "redis", 
  // "redshift", "ruby", "rust", "sb", "scheme", "scss", "shell", "sol", "sql", "st", "swift", "tcl", 
  // "typescript", "vb", "xml", "yaml"]
  language: "html",
  // 右键功能
  contextmenu: false,
  // 启用当前行突出显示。 默认 all。 "none"|"gutter"|"line"|"all"
  renderLineHighlight: "all",
  // 主题 vs, hc-black, or vs-dark
  theme: "vs-dark",
  // 单击行号时是否应选择相应的行？ 默认为true。
  selectOnLineNumbers: true,
  // 使用圆角边框渲染编辑器选择。 默认为true。
  roundedSelection: true,
  // 只读
  readOnly: false,
  // 控制光标样式 'block' or 'line'.	Defaults to 'line'.
  cursorStyle: "line",
  //自动布局 启用编辑器将安装间隔以检查其容器dom节点大小是否已更改。 启用此功能可能会对性能产生严重影响。 默认为false。
  automaticLayout: false,
  // 启用字形边距的渲染。 在vscode中默认为true，在monaco-editor中默认为false。
  glyphMargin: true,
  // 在制表位后插入和删除空格。
  useTabStops: true,
  //字体大小
  fontSize: 14,
  // 启用自动缩进调整。 默认为false。
  autoIndent: true,
  // 快速建议显示
  quickSuggestions: false,
  // 快速建议显示延迟（以毫秒为单位）默认为500（毫秒）
  quickSuggestionsDelay: 500
};
export default {
  props: {
    value: {
      type: [String, Object, Array],
      default: () => {
        return "<div>请编辑html内容</div>";
      }
    },
    cdnURL: {
      type: String,
      default: "https://cdn.bootcss.com/monaco-editor/0.17.0/min/vs"
    },
    editorOptions: {
      type: Object,
      default: function() {
        return {};
      }
    }
  },
  data() {
    return {
      //内容备份
      valueClone: "",
      monacoEditor: null
    };
  },
  mounted() {
    this.initEditor();
  },
  computed: {
    config() {
      return Object.assign({}, conf, this.editorOptions);
    }
  },
  watch: {
    value(val, old) {
      if (!val || !old) return;
      this.getVal();
      this.monacoEditor.setValue(this.valueClone);
    }
  },
  methods: {
    // 判断字符串是否为json数据
    isJsonString(str) {
      try {
        if (typeof JSON.parse(str) === "object") {
          return true;
        }
      } catch (e) {
        console.warn("不是一个有效的JSON String");
      }
      return false;
    },
    loadLibrary() {
      return new Promise((resolve, reject) => {
        if (window.monaco) return resolve();
        loader({
          url: this.cdnURL + "/loader.js",
          library: "require"
        }).then(library => {
          library.target.config({
            paths: { vs: this.cdnURL }
          });
          library.target(["vs/editor/editor.main"], () => {
            if (window.monaco) return resolve();
            else reject();
          });
        });
      });
    },
    getVal() {
      if (this.config.language === "json" && typeof this.value === "object") {
        this.valueClone = JSON.stringify(this.value, null, 2);
      } else {
        this.valueClone = this.value;
      }
    },
    initEditor() {
      this.loadLibrary().then(() => {
        const monaco = window.monaco;
        this.$refs.container.innerHTML = "";
        this.getVal();
        this.monacoEditor = monaco.editor.create(this.$refs.container, {
          value: this.valueClone,
          ...this.config
        });
        //编辑器创建完成回调
        this.$emit("onMounted", this.monacoEditor);
        this.monacoEditor.onDidBlurEditorWidget(event => {
          //编辑器内容changge事件
          this.valueClone = this.monacoEditor.getValue();
          if (
            this.config.language === "json" &&
            this.isJsonString(this.valueClone)
          ) {
            this.$emit(
              "onChange",
              JSON.parse(this.valueClone),
              this.monacoEditor
            );
            this.$emit("input", JSON.parse(this.valueClone), this.monacoEditor);
          } else {
            this.$emit("onChange", this.valueClone, this.monacoEditor);
            this.$emit("input", this.valueClone, this.monacoEditor);
          }
        });
        //编辑器随窗口自适应
        window.addEventListener("resize", () => {
          initEditor();
        });
      });
    }
  },
  beforeDestroy() {
    window.removeEventListener("resize", null);
    this.valueClone = "";
    this.monacoEditor = null;
  }
};
</script>
<style scoped>
.editer-container {
  width: 100%;
  height: 100%;
}
</style>