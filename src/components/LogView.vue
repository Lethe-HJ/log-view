<template>
  <div class="log-box">
    <div class="log-left" v-if="showLineNum">
        <div
          class="line-num"
          v-for="(item, index) in logContentLi"
          :key="index"
        >
          {{ index + 1 }}
        </div>
        <br />
      </div>
    <div class="log-right" ref="logElem">    
      <div>
        <p
          class="line"
          v-for="(item, index) in logContentLi"
          :key="index"
          v-html="item"
        ></p>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'LogView',
  props: {
    logContent: {
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
      content: this.logContent,
    };
  },
  computed: {
    logContentLi() {
      return this.content.split('\n');
    },
  },
  watch: {
    search: {
      handler() {
        // search相关字段有变动 重新查找并高亮
        this.searchAndHeightlight(this.search.content);
      },
      deep: true,
    },
  },
  mounted() {
    this.logElem = this.$refs.logElem;
  },
  methods: {
    // 清除高亮和标记
    cleanHightlight() {
      // 清除高亮
      this.content = this.logContent;
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
      this.cleanHightlight(); // 先把上一次的高亮和标记清除
      // 清空了搜索内容
      if (str === '') return true;
      this.searchObj.preSearch = str;

      // 取消正则支持时 转义特殊字符
      if (!this.search.regSupport) {
        str = this.regEscape(str);
      }
      const mode = this.search.caseSensetive ? 'g' : 'gi';
      this.content = this.logContent.replace(
        new RegExp(`(${str})`, mode),
        `<b class="found-item">$1</b>`,
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

      // 告诉父组件 现在是 第几个/总共多少个
      this.$emit('indexChange', {
        index: this.searchObj.index,
        count: this.searchObj.li.length,
      });
    },

    // 让内容滚动到中间
    scrollToWindowCenter(elemNode) {
      // 需要滚动的高度 = 这行的本身距离小窗口顶部的高度 - 小窗口高度的一半
      const top = elemNode.offsetTop - this.logElem.offsetHeight / 2;
      // 让这行滚到中间
      this.logElem.scrollTop = top;
    },

    // 对字符串中的 在正则表达式中有特殊含义的字符进行转义
    regEscape(str){
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
      escapeChars.forEach((item) => {
        str = str.replace(new RegExp(item, 'g'), item);
      });
      return str;
    },
  },
};
</script>
// scss样式的scoped会影响js动态添加的元素的样式，因此这部分分开写
<style>
.log-right .found-item {
  background-color: #f8c9ab;
}
.log-right .marked-word {
  background-color: #a8ac94;
}

.log-right .marked-line {
  background-color: #ffffcc;
}

.log-right .line:hover {
  background-color: #dcdfe6;
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
