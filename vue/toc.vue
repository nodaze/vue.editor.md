<template>
  <div></div>
</template>
<script>
import $ from "jquery";

export default {
  //TODO: TOC组件的具体位置的css需要自行修改组件内样式调整
  name: "toc",
  props: {
    // 用于调整滚动时距离顶部的偏移量
    offsetTop: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {}
  },
  mounted() {
    var that = this
    // 循环检测jq有没有加载成功
    var check_jQuery_is_load = setInterval(function () {
      try {
        if ($ != undefined) {
          clearInterval(check_jQuery_is_load)
          that.toc_main_script();
        }
      } catch (e) {
        console.log(e);
      }
    }, 200)
  },
  methods: {
    toc_main_script() {
      const that = this;
      // 监听点击事件并滑动到相应位置
      $(document).on('click', '.markdown-toc-list a[href]', function (event) {
        event.preventDefault();
        var name = $(this).attr('href').substring(1);
        //TODO 根据容器内的对toc样式的设置，需要设置同等的偏移量
        var top_at_window = $('[name="' + name + '"]').offset().top - that.offsetTop;
        $('html, body').animate({ scrollTop: top_at_window }, 100);
      })

      // 监听鼠标滚动事件给与颜色加亮
      $(window).scroll(function (event) {
        event.preventDefault();
        var activeName = '';
        $('.markdown-toc-list a[href]').each(function (index, el) {
          var name = $(el).attr('href').substring(1);
          //TODO 根据容器内的对toc样式的设置，需要设置同等的偏移量
          if ($('[name="' + name + '"]').offset().top - that.offsetTop - $(window).scrollTop() < 1) {
            activeName = name;
          } else {
            return false;
          }
        })
        $('.markdown-toc-list a').removeClass('current');
        $('.markdown-toc-list a[href="#' + activeName + '"]').addClass('current');
      })

      $('.markdown-toc').addClass('open-list');
    }
  },
  destroyed() {
    // 把那些监听了的事件去掉
    $(document).off('click', '.markdown-toc-list a[href]')
    $(document).off('click', '.markdown-toc')
    $('.markdown-toc').remove()
  }
}
</script>
<!-- 注意，这里是全局css -->
<style>
.markdown-toc {
  z-index: 999;
}

.markdown-toc > .markdown-toc-list {
  margin: 40px 0 0;
  list-style: none;
  min-width: 150px;
  max-width: 320px;
  padding: 5px 0;
  max-height: calc(100vh - 200px);
  overflow-y: auto;
}

.markdown-toc.open-list .markdown-toc-list {
  margin-right: 0px;
  margin-bottom: 0px;
  transform: scale(1) translateY(-44px);
  opacity: 1;
  visibility: visible;
}

.markdown-toc-list li {
  list-style: none !important;
}

.markdown-toc-list li > ul {
  padding-left: 15px;
}

.markdown-toc-list li > a {
  padding: 3px 15px;
  font-size: 14px;
  color: #606266;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  display: block;
  transition: 0.15s;
}

.markdown-toc li a.current {
  background: #ecf5ff;
  color: #40a9ff;
  box-shadow: 2px 0px #40a9ff inset;
}

.markdown-toc li a:hover {
  background: #d9ecff;
  text-decoration: none;
  color: #40a9ff;
  transition: 0s;
}

::-webkit-scrollbar {
  display: none; /* Chrome Safari */
}
</style>
