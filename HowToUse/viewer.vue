<template>
  <div class="document-container">
    <el-menu class="document-menu" default-active="1-4-1">
      <el-submenu index="1">
        <template slot="title">
          <span slot="title">导航一</span>
        </template>
        <el-menu-item-group>
          <span slot="title">分组一</span>
          <el-menu-item index="1-1">选项1</el-menu-item>
          <el-menu-item index="1-2">选项2</el-menu-item>
        </el-menu-item-group>
        <el-menu-item-group title="分组2">
          <el-menu-item index="1-3">选项3</el-menu-item>
        </el-menu-item-group>
        <el-submenu index="1-4">
          <span slot="title">选项4</span>
          <el-menu-item index="1-4-1">选项1</el-menu-item>
          <el-submenu index="1-5">
            <span slot="title">选项5</span>
            <el-menu-item index="1-5-1">选项1</el-menu-item>
          </el-submenu>
        </el-submenu>
      </el-submenu>
      <el-menu-item index="2">
        <span slot="title">导航二</span>
      </el-menu-item>
      <el-menu-item index="3" disabled>
        <span slot="title">导航三</span>
      </el-menu-item>
      <el-menu-item index="4">
        <span slot="title">导航四</span>
      </el-menu-item>
    </el-menu>
    <div class="document-editor">
      <editormd :id="editorid" ref="editor" type="editor" :mode="mode"></editormd>
    </div>

    <toc v-if="showToc" :offsetTop="93"></toc>
  </div>
</template>

<script>
import editormd from "@/../static/editor.md/vue/editormd";
import toc from "@/../static/editor.md/vue/toc";
import http from "@/http/index.js"

export default {
  name: "viewer",
  components: { editormd, toc },
  data() {
    return {
      editorid: "viewer", // 这个设置要保留, 有些功能依赖于jQuery通过元素的id查找
      mode: "viewer",
      showToc: false
    }
  },
  mounted() {
    const that = this
    http.post('/getmddemo/server/index.php?s=/api/page/info', { page_id: 234 }).then(response => {
      that.$refs.editor.show(response.data.page_content)
      that.showToc = true;
    })
  },
}
</script>

<style lang="scss" scoped>
.document-container {
  display: flex;
  width: 100%;

  .document-menu {
    position: fixed;
    width: 300px;
    border: unset;
  }

  .document-editor {
    margin-left: 300px;
    width: 800px;
    padding: 0 0 0 20px;
  }
}
</style>

<style lang="scss">
.document-container {
  .markdown-toc {
    position: fixed;
    top: 93px;
    margin-left: 820px;
    min-width: 32px;
  }

  .el-submenu > .el-submenu__title {
    height: initial;
    line-height: 48px;
    background-color: #e3e3e3;
    font-size: initial;
  }

  .el-submenu > .el-submenu__title .el-submenu__icon-arrow {
    font-size: 20px;
    font-weight: bold;
    -webkit-transform: rotateZ(-90deg);
    -ms-transform: rotate(-90deg);
    transform: rotateZ(-90deg);
    margin-top: -10px;
  }

  .el-submenu .el-menu--inline .el-submenu > .el-submenu__title {
  }
}
</style>
