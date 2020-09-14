<template>
  <div class="log-box">
    <div class="log-left" v-if="showLineNum">
      <div class="line-num" v-for="(item, index) in displayTextLi" :key="index">
        {{ index + 1 }}
      </div>
      <br />
    </div>
    <div class="log-right" ref="logElem">
      <div>
        <p
          class="line"
          v-for="(item, index) in displayTextLi"
          :key="index"
          v-html="item"
        ></p>
      </div>
    </div>
  </div>
</template>

<script>
import Timer from '@/utils/smart-timer/timer.js';

export default {
  name: 'LogView',
  props: {
    originContent: {
      type: String,
      default: '',
    },
    showLineNum: {
      type: Boolean,
      default: true,
    },
    search: {
      type: Object,
      default() {
        return {
          regSupport: false,
          caseSensetive: true,
          content: '',
        };
      },
    },
  },
  data() {
    return {
      logElem: null,
      searchObj: {
        li: [],
        index: null,
        preNode: null,
        preSearch: '',
      },
      // 初始处理 空格替换成&nbsp
      originText: this.replaceSpace(this.originContent),
      displayText: this.replaceSpace(this.originContent),
    };
  },
  computed: {
    displayTextLi() {
      return this.displayText.split('\n');
    },
  },
  watch: {
    search: {
      handler() {
        this.asyncSearch(this.search.content);
      },
      deep: true,
    },
    // 告诉父组件 现在是 第几个/总共多少个
    searchObj: {
      handler() {
        this.$emit('indexChange', {
          index: this.searchObj.index,
          count: this.searchObj.li.length,
        });
      },
      deep: true,
    },
  },
  mounted() {
    this.logElem = this.$refs.logElem;
    this.timer = new Timer({ times: 1 });
  },
  methods: {
    // 清除高亮和标记
    cleanHightlight() {
      // 清除高亮
      this.displayText = this.originText;
      // 恢复上一个被标记的行
      if (this.searchObj.preNode) {
        this.searchObj.preNode.parentNode.classList.remove('marked-line');
      }
      // 重置searchObj
      this.searchObj = {
        li: [],
        index: null,
        preNode: null,
        preSearch: '',
      };
    },

    searchAndHeightlight(str) {
      console.log('开始searchAndHeightlight', new Date().getTime());
      this.cleanHightlight(); // 先把上一次的高亮和标记清除
      // 清空了搜索内容
      if (str === '') return true;
      this.searchObj.preSearch = str;

      // 取消正则支持时 转义特殊字符
      if (!this.search.regSupport) {
        str = this.regEscape(str);
      }
      const mode = this.search.caseSensetive ? 'g' : 'gi';
      // 先高亮指定内容 注意先不给空格
      // 查找使用包含空格的原数据
      const heightLightText = this.originContent.replace(
        new RegExp(`(${str})`, mode),
        `<bclass="found-item">$1</b>`,
      );
      // 替换空格为&nbsp
      const noSpaceText = this.replaceSpace(heightLightText);
      // 然后把高亮标签的空格加上
      this.displayText = noSpaceText.replace(
        /<bclass="found-item">/g,
        `<b class="found-item">`,
      );
      this.$nextTick(() => {
        // 等渲染完成
        // 把这些找到的元素保存起来
        this.searchObj.li = this.logElem.getElementsByClassName('found-item');
        // 没有找到高亮
        if (this.searchObj.li.length === 0) {
          this.$message('没有找到');
          return false;
        }
        this.FollowingSearchedNode(); // 标记下一个
        this.searchObj.preNode = this.searchObj.li[0]; // 储存第一个高亮节点为上一个被标记的节点
        console.log('显示searchAndHeightlight', new Date().getTime());
      });
      console.log('结束searchAndHeightlight', new Date().getTime());
      return true;
    },

    asyncSearch(str) {
      return new Promise((resolve, reject) => {
        this.timer.getTimer(() => {
          const res = this.searchAndHeightlight(str);
          if (res) {
            resolve('haha');
          } else {
            reject(Error('hehe'));
          }
        }, 200);
      });
    },

    // 标记下(上)一个搜索内容或指定第几个搜索内容
    FollowingSearchedNode(isFollowing = true) {
      if (this.searchObj.now === '') return false; // 没有搜索内容时 该函数不做任何操作
      if (this.searchObj.index !== null) {
        this.searchObj.index += isFollowing ? 1 : -1; // 向上还是向下
        const max = this.searchObj.li.length - 1;
        if (this.searchObj.index < 0) this.searchObj.index = max;
        if (this.searchObj.index > max) this.searchObj.index = 0;
      } else {
        // 第一次标记第一个
        this.searchObj.index = 0;
      }
      const elemNode = this.searchObj.li[this.searchObj.index];
      this.scrollToWindowCenter(elemNode); // 让这个元素显示在小窗口中央
      const preNode = this.searchObj.preNode;
      if (preNode !== null) {
        preNode.classList.remove('marked-word'); // 取消标记上一个被标记节点
        preNode.parentNode.classList.remove('marked-line'); // 取消标记上一个被标记节点所在行
      }
      elemNode.classList.add('marked-word'); // 标记这个节点
      elemNode.parentNode.classList.add('marked-line'); // 标记节点所在行
      this.searchObj.preNode = elemNode; // 储存当前被标记节点为下一次的上一个被标记的节点
    },

    // 让内容滚动到中间
    scrollToWindowCenter(elemNode) {
      // 需要滚动的高度 = 这行的本身距离小窗口顶部的高度 - 小窗口高度的一半
      const top = elemNode.offsetTop - this.logElem.offsetHeight / 2;
      // 让这行滚到中间
      this.logElem.scrollTop = top;
    },

    // 对字符串中的 在正则表达式中有特殊含义的字符进行转义
    // 输入字符串'.' 返回字符串'\.' 注意不是字面量
    regEscape(str) {
      const escapeChars = [
        '\\\\',
        '\\^',
        '\\$',
        '\\+',
        '\\?',
        '\\=',
        '\\!',
        '\\.',
        '\\/',
        '\\(',
        '\\)',
        '\\[',
        '\\]',
        '\\{',
        '\\}',
      ];
      // 注意上述字符串输入为字面量\\. 真实存在的字符串为\.
      escapeChars.forEach((item) => {
        // 第一个item中字符串'\.' 是作为正则表达式 含义是转义'.'
        // 第二个item中字符串'\.' 作为替换后的字符串 '\.'
        str = str.replace(new RegExp(item, 'g'), item);
      });
      return str;
    },

    // 替换字符串中所有的空格为&nbsp
    replaceSpace(content) {
      return content.replace(new RegExp(/\x20/, 'g'), '&nbsp');
    },
  },
};
</script>
// scss样式的scoped会影响js动态添加的元素的样式，因此这部分分开写
<style lang="scss">
.log-right {
  .found-item {
    background-color: #f8c9ab;
  }

  .marked-word {
    background-color: #a8ac94;
  }

  .marked-line {
    background-color: #ffffcc;
  }

  .line:hover {
    background-color: #dcdfe6;
  }
}
</style>

<style lang="scss" scoped>
.log-box {
  height: 700px;
  border: 1px solid #dcdfe6;
  overflow-y: auto;
  overflow-x: hidden;
  .log-right {
    background-color: #fff;
    overflow-x: scroll;
    overflow-y: hidden;
    .line {
      margin: 0px 2px 0px;
      padding: 0px;
      color: #000;
      height: 1rem;
      line-height: 1.15;
      white-space: nowrap;
    }
  }

  .log-left {
    border-right: 1px solid #dcdfe6;
    padding: 0px 10px 0px;
    min-width: 15px;
    float: left;
    text-align: right;
    .line-num {
      color: #2380b8;
      height: 1rem;
    }
  }
}
</style>
