<template>
  <div :id="id" class="markdown-editor">
    <link rel="stylesheet" type="text/css" href="/static/editor.md/css/editormd.min.css">
    <textarea v-html="content" style="display:none;"></textarea>
    <!-- 放大图片 -->
    <bigimage v-if="bigImageShow" @clickit="bigImageShow = false" :imageSrc="bigImageSrc"></bigimage>
    <uploader ref="uploader" :api-url="uploadAssertUrl" @uploadSuccess="handleUploadAssertSuccess"></uploader>
  </div>
</template>

<script>
import bigimage from "./components/bigimage";
import uploader from "./components/uploader";
import scriptjs from "scriptjs";
import $ from "jquery";

export default {
  /* eslint-disable */
  name: "editormd",
  components: { bigimage, uploader },
  props: {
    content: {
      type: String,
      default: "### hhhhhhhhhhhhh"
    },
    id: {
      type: String,
      default: "editor-md"
    },
    // mode等于editor是编辑，否则展示数据
    mode: {
      type: String,
      default: "editor"
    },
    uploadImageUrl: {
      type: String,
      default: ""
    },
    uploadAssertUrl: {
      type: String,
      default: ""
    },
    onContentLoaded: {
      type: Function,
      default: null
    },
    onContentChanged: {
      type: Function,
      default: null
    },
    // editor插件的配置文件, 不要在配置内配置回调方法, 如onchange, onload...
    // 请使用props传递插件需要执行的onContentLoaded和onContentChanged等方法
    // 更多原生的方法可以参考源码自行再实现
    editorConfig: {
      type: Object,
      default: {
        taskList: true,
        tex: true, // 默认不解析
        flowChart: true, // 默认不解析
        sequenceDiagram: true, // 默认不解析
        syncScrolling: 'single',
        htmlDecode: 'style,script,iframe|filterXSS',
        emoji: true,
        placeholder: 'Enjoy writing...',
        codeFold: true,
        saveHTMLToTextarea: true,
        searchReplace: true,
        imageUpload: true,
        imageFormats: ["jpg", "jpeg", "gif", "png", "bmp"],
        toolbarIcons() {
          return ["undo", "redo",
            "|", "bold", "del", "italic", "quote", "ucwords", "uppercase", "lowercase",
            "|", "h1", "h2", "h3", "h4", "h5", "h6",
            "|", "list-ul", "list-ol", "hr",
            "|", "link", "reference-link", "image", "assert", "code", "preformatted-text", "code-block", "table", "datetime", "emoji", "html-entities", "pagebreak",
            "|", "goto-line", "watch", "preview", "fullscreen", "clear", "search",]
          // return ["bold", "del", "italic", "quote", "ucwords", "uppercase", "lowercase", "|",
          //   "h1", "h2", "h3", "h4", "h5", "h6", "|",
          //   "hr", "code", "preformatted-text", "code-block", "table", "pagebreak", "|",
          //   "goto-line", "preview", "clear"]
        },
        toolbarHandlers: {
          assert(cm, icon, cursor, selection) {
            this.settings.container.$refs.uploader.show();
          }
        },
        toolbarIconsClass: {
          assert: 'fa-paperclip'
        },
        lang: {
          toolbar: {
            assert: "添加附件",
          }
        },
      }
    }
  },
  data() {
    return {
      editor: null,
      bigImageShow: false,
      bigImageSrc: "",
    };
  },
  methods: {
    // 外部方法
    show(content) {
      if (this.editor) {
        this.editor.setMarkdown(content);
      } else {
        this.init(content);
      }
    },
    // 外部方法
    getPreviewedHTML() {
      return this.editor.getPreviewedHTML();
    },
    // 外部方法
    getMarkdown() {
      return this.editor.getMarkdown();
    },
    init(content) {
      if (!this.editorConfig) {
        console.log("Config of editor can not be null!");
        return;
      }

      (async () => {
        // 配置editor依赖的各个js的路径, 如果调整代码结构, 这个地方也需要修改
        let editorPath = '/static/editor.md';

        await this.fetchScript(`${editorPath}/../jquery.min.js`);
        await this.fetchScript(`${editorPath}/lib/raphael.min.js`);
        await this.fetchScript(`${editorPath}/lib/flowchart.min.js`);
        await this.fetchScript(`${editorPath}/lib/d3@5.min.js`);

        await this.fetchScript(`${editorPath}/../xss.min.js`);
        await this.fetchScript(`${editorPath}/lib/marked.min.js`);
        await this.fetchScript(`${editorPath}/lib/prettify.min.js`);
        await this.fetchScript(`${editorPath}/lib/underscore.min.js`);
        await this.fetchScript(`${editorPath}/lib/sequence-diagram.min.js`);
        await this.fetchScript(`${editorPath}/lib/jquery.flowchart.min.js`);
        await this.fetchScript(`${editorPath}/lib/jquery.mark.min.js`);
        await this.fetchScript(`${editorPath}/lib/plantuml.js`);
        await this.fetchScript(`${editorPath}/lib/view.min.js`);
        await this.fetchScript(`${editorPath}/lib/transform.min.js`);
        await this.fetchScript(`${editorPath}/lib/highlight.min.js`);
        await this.fetchScript(`${editorPath}/editormd.js`);

        this.$nextTick((editorMD = window.editormd) => {
          if (editorMD) {

            this.editorConfig.container = this;// 让editor持有自己的vue容器, 供内部事件使用
            this.editorConfig.path = `${editorPath}/lib/`;
            this.editorConfig.imageUpload ? this.editorConfig.imageUploadURL = this.uploadImageUrl : null;

            this.editorConfig.onload = () => {
              // 编辑模式通过onload事件注入查看大图的功能
              this.injectViewBigImage();
              this.onContentLoaded ? this.onContentLoaded() : null;
            }
            this.editorConfig.onchange = () => {
              this.onContentChanged ? this.onContentChanged() : null;
            }

            if (content) {
              this.content = content;
              this.editorConfig.markdown = content;
            }

            // type等于editor是编辑, 否则展示数据,
            // editorConfig实际时function
            if (this.mode == "editor") {
              this.editor = editorMD(this.id, this.editorConfig);
              // 注入剪切板的图片粘贴上传功能
              this.injectPasteFromClipboardFunction();
            } else {
              this.editor = editorMD.markdownToHTML(this.id, this.editorConfig)
              // 浏览模式直接注入查看大图的功能
              this.injectViewBigImage();
            }

            // 初始化highlight js
            hljs.initHighlightingOnLoad();
          }
        });
      })();
    },
    injectPasteFromClipboardFunction() {
      document.getElementById(this.id).addEventListener("paste", e => {
        var clipboard = e.clipboardData;
        for (var i = 0, len = clipboard.items.length; i < len; i++) {
          if (clipboard.items[i].kind != 'file' && clipboard.items[i].type.indexOf('image') < 0) {
            continue;
          }
          var imageFile = clipboard.items[i].getAsFile();
          var form = new FormData;
          form.append('t', 'ajax-uploadpic');
          form.append('editormd-image-file', imageFile);

          const that = this;
          $.ajax({
            url: this.uploadImageUrl,
            type: "POST",
            dataType: "json",
            data: form,
            processData: false,
            contentType: false,
            beforeSend: function () {
            },
            error: function () {
              $.alert('图片上传失败');
            },
            success: function (data) {
              if (data.success == 1) {
                that.editor.insertValue('![](' + data.url + ')');
              } else {
                $.alert(data.message);
              }
            }
          })
          e.preventDefault();
        }
      });
    },
    injectViewBigImage() {
      const that = this;
      // 图片点击放大
      $('#' + this.id + ' img').click(function () {
        that.bigImageShow = true // 获取当前图片地址
        that.bigImageSrc = $(this).attr('src')
      })
    },
    handleUploadAssertSuccess(response) {
      this.editor.insertValue('[' + response.display_name + '](' + response.url + ')');
    },
    // 加载js包, 内部方法
    fetchScript: function (url) {
      return new Promise((resolve) => {
        scriptjs(url, () => {
          resolve()
        })
      })
    },
  },
};
</script>

<style lang="scss">
.markdown-editor {
  width: inherit !important;
  margin: unset !important;
  padding: unset !important;
}
</style>
