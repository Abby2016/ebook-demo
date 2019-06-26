<template>
  <div class="ebook">
    <!----------------------------- title-bar ----------------------------->
    <title-bar :ifMenuShow="ifMenuShow" />
    <!----------------------------- read-wrapper ----------------------------->
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleMenuShow"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <!----------------------------- menu-bar ----------------------------->
    <menu-bar
      :ifMenuShow="ifMenuShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      @setFontSize="setFontSize"
      ref="menuBar"
      :themeList="themeList"
      :defaultTheme="defaultTheme"
      :bookAvailable="bookAvailable"
      @setTheme="setTheme"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
    />
  </div>
</template>

<script>
import Epub from "epubjs";
import TitleBar from "./components/TitleBar";
import MenuBar from "./components/MenuBar";
const DOWNLOAD_URL = "/prince.epub";

export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      ifMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "default",
          style: {
            body: {
              color: "#000",
              background: "#fff"
            }
          }
        },
        {
          name: "eye",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "night",
          style: {
            body: {
              color: "#fff",
              background: "#000"
            }
          }
        },
        {
          name: "gold",
          style: {
            body: {
              color: "#000",
              background: "rgb(241,236,226)"
            }
          }
        }
      ],
      defaultTheme: 0,
      bookAvailable: false,
      navigation: {}
    };
  },
  methods: {
    // 根据链接跳转到指定位置
    jumpTo(href) {
      this.rendition.display(href);
      this.hideTitleAndMenu();
    },
    // 隐藏标题栏、菜单栏、目录
    hideTitleAndMenu() {
      this.ifMenuShow = false;
      this.$refs.menuBar.hideSetting();
      this.$refs.menuBar.hideContent();
    },
    // 拖动进度条时页面定位
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    // 上一页
    prevPage() {
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    // 下一页
    nextPage() {
      if (this.rendition) {
        this.rendition.next();
      }
    },
    // 切换显示隐藏标题及菜单栏
    toggleMenuShow() {
      this.ifMenuShow = !this.ifMenuShow;
      if (!this.ifMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    // 设置字体大小
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        this.themes.fontSize(fontSize + "px");
      }
    },
    // 注册主题
    registerTheme() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style);
      });
    },
    // 设置主题
    setTheme(index) {
      this.themes.select(this.themeList[index].name);
      this.defaultTheme = index;
    },
    // 电子书的解析和渲染
    showEpub() {
      // 生成Book
      this.book = new Epub(DOWNLOAD_URL);
      // 生成Rendition,通过Book.renderTo
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      // 通过Rendition.display渲染电子书
      this.rendition.display();
      // 获取主题
      this.themes = this.rendition.themes;
      // 设置默认字体
      this.setFontSize(this.defaultFontSize);
      // 注册主题
      this.registerTheme();
      this.setTheme(this.defaultTheme);
      // 获取Location对象、navigation对象
      // 通过epubjs的钩子函数来实现
      this.book.ready
        .then(() => {
          this.navigation = this.book.navigation;
          return this.book.locations.generate();
        })
        .then(() => {
          this.locations = this.book.locations;
          this.bookAvailable = true;
        });
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>

<style lang="scss" scoped>
@import "assets/styles/global";
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 100;
      display: flex;
      .left {
        flex: 0 0 px2rem(100);
        cursor: w-resize;
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
        cursor: e-resize;
      }
    }
  }
}
</style>
