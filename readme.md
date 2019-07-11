# props

| 属性          | 类型                          | 默认值                                                | 描述                                                         |
| ------------- | ----------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| value         | [*String*, *Object*, *Array*] | "<div>请编辑html内容</div>"                           | 显示的数据值                                                 |
| cdnURL        | *String*                      | "https://cdn.bootcss.com/monaco-editor/0.17.0/min/vs" | cdn地址 可去 https://github.com/Microsoft/monaco-editor 下载 |
| editorOptions | *Object*                      | {}                                                    | https://microsoft.github.io/monaco-editor/api/interfaces/monaco.editor.ieditorconstructionoptions.html 这个地址可以去配置 |

# useing

## 安装

```
npm i vue2-monaco-editer
```

## 注册

```
import MonacoEditor from "vue-monaco2-editer";
```

```
components: {
	MonacoEditor
},
```

```
<MonacoEditor v-model="value" :editorOptions="editorOptions"></MonacoEditor>
```

## tip

数据使用v-model双向绑定的