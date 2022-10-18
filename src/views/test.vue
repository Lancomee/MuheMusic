<template>
  <div class="list-view" ref="scrollBox" @scroll="handleScroll">
    <div class="list-view-phantom" :style="{ height: contentHeight }"></div>
    <div class="list-view-content" ref="content">
      <div
        class="list-view-item"
        :style="{ height: itemHeight + 'px' }"
        v-for="(item, i) in visibleData"
        :key="i"
      >
        {{ item }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      itemHeight: 30,
      visibleData: [],
    };
  },
  computed: {
    contentHeight() {
      return this.data.length * this.itemHeight + "px";
    },
  },
  mounted() {
    //构造一个超长列表
    for (let i = 0; i < 1000000; i++) {
      this.data.push("列表" + i);
    }
    this.updateVisibleData();
  },
  methods: {
    handleScroll() {
      const scrollTop = this.$refs.scrollBox.scrollTop;
      console.log("scrollTop", scrollTop);
      this.updateVisibleData(scrollTop);
    },
    updateVisibleData(scrollTop = 0) {
      const clientHeight = this.$refs.scrollBox.clientHeight;
      console.log("clientHeight", clientHeight);
      const showNum = Math.ceil(clientHeight / this.itemHeight);
      console.log(showNum);
      const start = Math.floor(scrollTop / this.itemHeight);
      const end = start + showNum;
      this.visibleData = this.data.slice(start, end);
      console.log("this.visibleData", this.visibleData);
      this.$refs.content.style.webkitTransform = `translate3d(0, ${
        start * this.itemHeight
      }px, 0)`;
    },
  },
};
</script>

<style>
.list-view {
  height: 400px;
  overflow: auto;
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
</style>
