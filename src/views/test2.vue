<template>
  <div class="list-wrap" ref="listWrap" @scroll="scrollListener">
    <!-- 可视区域 -->
    <div class="scroll-bar" ref="scrollBar">
      <div class="list" ref="list">
        <div class="list-item" v-for="(item,i) in searchlist" :key="i">1</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 后台返回的数据
      searchlist: [],
      // 行高
      itemHeight: 50,
      // 可视区域显示几条数据
      showNum: 10,
      // 滚动过程的开始索引
      startIndex: 0,
      // 滚动过程的结束索引
      endIndex: 10,
    };
  },
  computed: {
    // 可视区域显示的数组
    showList() {
      return this.searchlist.slice(this.startIndex, this.endIndex);
    },
  },
  mounted() {
    // 构造一个数据
    for (let i = 0; i < 1000000; i++) {
                this.searchlist.push('列表' + i)
            }
    // 计算可视区域滚动高度
    this.$refs.listWrap.style.height= this.itemHeight*this.showNum +'px'
    // 计算总得数据高度和
    this.$refs.scrollBar.style.height = this.itemHeight * this.searchlist.length + 'px'
},
methods:{
  scrollListener(){
      // 获取滚动高度
      let scrollTop = this.$refs.listWrap.scrollTop
      // 开始的数组索引
      this.startIndex = Math.floor(scrollTop/this.itemHeight)
      this.endIndex = this.startIndex + this.showNum
      // 绝对定位的偏移量
       this.$refs.list.style.top = this.startIndex*this.itemHeight + 'px';
  }
}
  
};
</script>

<style></style>
