<template>
  <div>
    <el-row>
      <el-col :span="4">
        <div class="search">
          <div class="search-div">
            <el-input
              placeholder="请输入内容"
              prefix-icon="el-icon-search"
              size="mini"
              class="search-input"
              v-model="search.content"
              @keyup.enter.native.exact="nextOrPrevious(true)"
              @keyup.shift.enter.native="nextOrPrevious(false)"
              clearable
            >
            </el-input>
          </div>
          <el-button-group class="search-buttons">
            <el-button
              size="mini"
              class="search-button"
              icon="el-icon-top"
              :disabled="search.content === ''"
              @click="nextOrPrevious(false)"
            >
            </el-button>
            <el-button
              size="mini"
              class="search-button"
              icon="el-icon-bottom"
              :disabled="search.content === ''"
              @click="nextOrPrevious(true)"
            >
            </el-button>
            <span class="position">
              {{ position.count ? position.index + 1 : 0 }}
              of
              {{ position.count }}
            </span>
          </el-button-group>
        </div>

        <div class="setting">
          <el-checkbox v-model="showLineNum">行号</el-checkbox>
          <el-checkbox v-model="search.regSupport">正则</el-checkbox>
          <el-checkbox v-model="search.caseSensetive">区分大小写</el-checkbox>
        </div>
      </el-col>
      <el-col :span="16" :offset="5">
        <div>
          <el-tabs
            v-model="editableTabsValue"
            type="card"
            editable
            @edit="handleTabsEdit"
          >
            <el-tab-pane
              :key="item.name"
              v-for="item in editableTabs"
              :label="item.title"
              :name="item.name"
            >
              <ko-log
                :originContent="logContent"
                :search="search"
                :showLineNum="showLineNum"
                @indexChange="indexOfCount"
                ref="logElem"
              >
              </ko-log>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import KOLog from '@/views/log/KOLog';
import { content2 } from '@/views/log/testLog';

export default {
  components: {
    'ko-log': KOLog,
  },
  data() {
    return {
      editableTabsValue: '1',
      editableTabs: [
        {
          title: 'node 日志',
          name: '1',
          content: 'Tab 1 content',
        },
      ],
      tabIndex: 1,
      logContent: content2,
      showLineNum: true,
      search: {
        regSupport: false,
        caseSensetive: true,
        content: '',
      },
      position: {
        index: 0,
        count: 0,
      },
    };
  },
  watch: {},
  methods: {
    nextOrPrevious(next) {
      this.$refs.logElem[0].FollowingSearchedNode(next);
    },
    handleTabsEdit(targetName, action) {
      if (action === 'add') {
        const newTabName = `${this.tabIndex + 1}`;
        this.editableTabs.push({
          title: 'New Tab',
          name: newTabName,
          content: 'New Tab content',
        });
        this.editableTabsValue = newTabName;
      }
      if (action === 'remove') {
        const tabs = this.editableTabs;
        let activeName = this.editableTabsValue;
        if (activeName === targetName) {
          tabs.forEach((tab, index) => {
            if (tab.name === targetName) {
              const nextTab = tabs[index + 1] || tabs[index - 1];
              if (nextTab) {
                activeName = nextTab.name;
              }
            }
          });
        }
        this.editableTabsValue = activeName;
        this.editableTabs = tabs.filter((tab) => tab.name !== targetName);
      }
    },

    indexOfCount(position) {
      this.position = position;
    },
  },
};
</script>

<style lang="scss" scoped>
.search-div {
  position: relative;
  float: left;
  width: 150px;
}
.search-buttons {
  position: relative;
  float: left;
}
.search {
  position: absolute;
  margin-top: 30px;
  overflow: auto;
}
.setting {
  position: absolute;
  margin-top: 65px;
  margin-left: 5px;
}
.position {
  margin-left: 10px;
  line-height: 28px;
}
</style>
