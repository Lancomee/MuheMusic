<template>
  <div class="searchTop">
    <svg class="icon" aria-hidden="true" @click="$router.go(-1)">
      <use xlink:href="#icon-zuojiantou"></use>
    </svg>
    <input
      type="text"
      placeholder="陈奕迅"
      v-model="searchKey"
      @keydown.enter="enterKey"
    />
  </div>
  <div class="searchHistory">
    <span class="searchSpan">历史</span>
    <span
      v-for="item in keyWorldList"
      :key="item"
      class="spanKey"
      @click="searchHistory(item)"
    >
      {{ item }}
    </span>
    <svg class="icon" aria-hidden="true" @click="delHistory">
      <use xlink:href="#icon-shanchu"></use>
    </svg>
  </div>

  <div class="itemList">
    <!-- 虚拟列表 可视化视图区域-->
    <div class="list-view" ref="scrollBox" @scroll="handleScroll">
      <div
        class="list-view-phantom"
        :style="{
          height: contentHeight,
        }"
      ></div>
      <div class="list-view-content" ref="content">
        <!-- 单条数据渲染区域 -->
        <div
          class="item"
          v-for="(item, i) in searchList"
          :key="i"
          :style="{ height: itemHeight + 'px' }"
        >
          <!-- 列表item左侧 -->
          <div class="itemLeft" @click="updateIndex(item)">
            <span class="leftSpan">{{ i + 1 }}</span>
            <div>
              <p>{{ item.name }}</p>
              <span v-for="(item1, index) in item.artists" :key="index">{{
                item1.name
              }}</span>
            </div>
          </div>
          <!-- 列表item右侧 -->
          <div class="itemRight">
            <svg class="icon bofang" aria-hidden="true" v-if="item.mvid != 0">
              <use xlink:href="#icon-shipin"></use>
            </svg>
            <svg class="icon liebiao" aria-hidden="true">
              <use xlink:href="#icon-31liebiao"></use>
            </svg>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { getSearchMusic } from "@/request/api/home.js";
export default {
  data() {
    return {
      // 收集关键字列表
      keyWorldList: [],
      // 搜索关键字
      searchKey: "",
      // 获取的搜索列表
      searchList: [],
      // 虚拟列表每一个item行高
      itemHeight: 50,
      // 可见区域的数据
      visibleData: [],
     
    };
  },
  computed: {
    // 所有数据的总高度
    contentHeight() {
      return this.searchList.length * this.itemHeight + "px";
    },
  },
  mounted() {
    // 获取本地存储内的关键字
    this.keyWorldList = JSON.parse(localStorage.getItem("keyWorldList"))
      ? JSON.parse(localStorage.getItem("keyWorldList"))
      : [];
    this.updateVisibleData();
  },
  methods: {
    enterKey: async function () {
      if (this.searchKey !== "") {
        this.keyWorldList.unshift(this.searchKey);
        //   去重
        this.keyWorldList = [...new Set(this.keyWorldList)];
        // 固定关键字列表长度
        if (this.keyWorldList.length > 10) {
          this.keyWorldList.splice(this.keyWorldList.length - 1, 1);
        }
        //将关键字存储到本地
        localStorage.setItem("keyWorldList", JSON.stringify(this.keyWorldList));
        let res = await getSearchMusic(this.searchKey);
        // console.log(res);
        // 获取搜索列表
        this.searchList = res.data.result.songs;
        this.searchKey = "";
      }
    },
    delHistory: function () {
      localStorage.removeItem("keyWorldList");
      this.keyWorldList = [];
    },
    searchHistory: async function (item) {
      let res = await getSearchMusic(item);
      //   console.log(res);
      this.searchList = res.data.result.songs;
      console.log('this.searchList',this.searchList);
    },
    updateIndex: function (item) {
      item.al = item.album;
      item.al.picUrl = item.album.artist.img1v1Url;
      this.$store.commit("pushPlayList", item);
      this.$store.commit(
        "updatePlayListIndex",
        this.$store.state.playList.length - 1
      );
    },
    // 虚拟列表
    updateVisibleData(scrollTop = 0) {
      //可视区域最多出现的数据条数
      const visibleCount = Math.ceil(
        this.$refs.scrollBox.clientHeight / this.itemHeight
      );
      console.log(visibleCount);
      // 开始索引
       const start = Math.floor(scrollTop / this.itemHeight);
      //  结束索引
      const end = start + visibleCount;
      // 可视区域显示的数据
      this.visibleData = this.searchList.slice(start, end);
      // 偏移量
      this.$refs.content.style.webkitTransform = `translate3d(0, ${
        this.start * this.itemHeight
      }px, 0)`;
    },
     //监听滚动事件，实时计算scrollTop
    handleScroll() {
      // 当前滚动的位置
      const scrollTop = this.$refs.scrollBox.scrollTop;
      console.log("scrollTop", scrollTop);
      this.updateVisibleData(scrollTop);
    },
  },
};
</script>
<style lang="less" scoped>
.searchTop {
  width: 100%;
  height: 1rem;
  padding: 0 0.2rem;
  display: flex;
  align-items: center;
  input {
    margin-left: 0.2rem;
    border: none;
    border-bottom: 1px solid #999;
    width: 90%;
    padding: 0.1rem;
  }
}
.searchHistory {
  width: 100%;
  padding: 0.2rem;
  position: relative;
  .searchSpan {
    font-weight: 700;
  }
  .spanKey {
    padding: 0.1rem 0.2rem;
    background-color: rgb(185, 169, 169);
    border-radius: 0.4rem;
    margin: 0.1rem 0.2rem;
    display: inline-block;
  }
  .icon {
    width: 0.3rem;
    height: 0.3rem;
    position: absolute;
    right: 0.2rem;
  }
}
.itemList {
  width: 100%;
  padding: 0.2rem;
  .list-view {
    height: 460px;
    overflow: auto;  /*只有这行代码写了，内容超出高度才会出现滚动条*/
    position: relative;
    border: 1px solid #aaa;
  }

  .list-view-phantom {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    z-index: -1;
  }

  .list-view-content {
    left: 0;
    right: 0;
    top: 0;
    position: absolute;
  }
  .item {
    width: 100%;
    height: 1.4rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    .itemLeft {
      width: 85%;
      height: 100%;
      display: flex;
      align-items: center;
      .leftSpan {
        display: inline-block;
        width: 0.2rem;
        text-align: center;
      }
      div {
        p {
          width: 4.54rem;
          height: 0.4rem;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          font-weight: 700;
        }
        span {
          font-weight: 400;
          font-size: 0.24rem;
          color: #999;
        }
        margin-left: 0.3rem;
      }
    }
    .itemRight {
      width: 20%;
      height: 100%;
      display: flex;
      // justify-content: space-between;
      align-items: center;
      position: relative;
      .icon {
        fill: #999;
      }
      .bofang {
        position: absolute;
        left: 0;
      }
      .liebiao {
        position: absolute;
        right: 0;
      }
    }
  }
}
</style>
