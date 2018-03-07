<template>
  <!-- 1.1 使用scroll组件 -->
  <scroll class="listview"
          ref="listview"
          :probe-type="probeType"
          :listen-scroll="listenScroll"
          @scroll="scroll">
    <!-- 1.0 左边歌手列表结构 -->
    <ul>
			<li class="list-group" v-for="(group, index) in data" :key="index" ref="listGroup">
				<h2 class="list-group-title">{{group.title}}</h2>
				<ul>
					<li class="list-group-item" v-for="(item, index) in group.items" :key="index">
						<img class="avatar" v-lazy="item.avatar" alt="">
						<span class="name">{{item.name}}</span>
					</li>
				</ul>
			</li>
		</ul>
    <!-- 2.0 右侧字母列表结构 -->
    <div class="list-shortcut"
          @touchstart="onShortcutTouchStart"
          @touchmove.stop.prevent="onShortcutTouchMove">
			<ul>
				<li class="item"
						v-for="(item, index) in shortcutList"
						:key="index"
						:data-index="index"
						:class="{'current': currentIndex === index}">
            <!-- 4.4 添加样式 -->
					{{item}}
				</li>
			</ul>
		</div>
  </scroll>
</template>

<script>
import Scroll from 'base-components/scroll/BaseScroll';
import {getData} from 'common/js/dom';

const ANCHOR_HEIGHT = 18;
export default {
  components: {Scroll},
	props: {
    data: {
      type: Array,
      default: []
    }
  },
  data() {
    return {
      scrollY: -1,
      currentIndex: 0
    };
  },
  computed: {
    // 2.1 取得字母列表数据
    shortcutList() {
      return this.data.map(value => {
        return value.title.substr(0, 1);
      });
    }
  },
  watch: {
    // 4.2 数据变化时，需要从新计算高度
    data() {
      setTimeout(() => {
        this._calcHeight();
      });
    },
    // 4.3 观测当前的高度，得到范围之间的索引
    scrollY(newY) {
      const listHeight = this.listHeight;
      for (let i = 0; i < listHeight.length; i++) {
        const height1 = listHeight[i];
        const height2 = listHeight[i + 1];
        if (!height2 || (-newY > height1 && -newY < height2)) {
          this.currentIndex = i;
          return;
        }
      }
      this.currentIndex = 0;
    }
  },
  created() {
    this.touch ={};
    this.probeType = 3;
    this.listenScroll = true;
		this.listHeight = [];
  },
  mounted() {
    // setTimeout(() => {
    //   this._calcHeight();
    // }, 20);
  },
  methods: {
    // 3.0 点击字母
    onShortcutTouchStart(e) {
      let anchorIndex = getData(e.target, 'index'); // 获取到的自定义index是String
      console.log(anchorIndex)
      let firstTouch = e.touches[0];
      this.touch.y1 = firstTouch.pageY;
      this.touch.anchorIndex = anchorIndex;
      this._scrollTo(anchorIndex);
    },
    // 3.1 字母列表滑动
    onShortcutTouchMove(e) {
      let firstTouch = e.touches[0];
      this.touch.y2 = firstTouch.pageY;
      let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0; // 手指滑动了多少个字母
      let anchorIndex = +this.touch.anchorIndex + delta;
      this._scrollTo(anchorIndex);
    },
    // 4.0 获取scroll子组件传过来的pos位置，此前需要设置probeType = 3，listenScroll = true
    scroll(pos) {
      this.scrollY = pos.y;
    },
    // 4.1 计算每一项的高度并放到数组里
    _calcHeight() {
      this.listHeight = [];
      const list = this.$refs.listGroup;
      let height = 0;
      this.listHeight.push(height);
      if (list) {
        for (let i = 0; i < list.length; i++) {
          const item = list[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    // 传入索引让容器滚动到对应位置
    _scrollTo(index) {
      this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0);
    }
  }
}
</script>

<style scoped lang="stylus">
@import '~common/stylus/variable'
.listview
  position: relative
  width: 100%
  height: 100%
  overflow: hidden
  background: $color-background
  .list-group
    padding-bottom: 30px
    .list-group-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background: $color-highlight-background
    .list-group-item
      display: flex
      align-items: center
      padding: 20px 0 0 30px
      .avatar
        width: 50px
        height: 50px
        border-radius: 50%
      .name
        margin-left: 20px
        color: $color-text-l
        font-size: $font-size-medium
  .list-shortcut
    position: absolute
    z-index: 30
    right: 0
    top: 50%
    transform: translateY(-50%)
    width: 20px
    padding: 20px 0
    border-radius: 10px
    text-align: center
    background: $color-background-d
    font-family: Helvetica
    .item
      padding: 3px
      line-height: 1
      color: $color-text-l
      font-size: $font-size-small
      &.current
        color: $color-theme
  .list-fixed
    position: absolute
    top: 0
    left: 0
    width: 100%
    .fixed-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background: $color-highlight-background
  .loading-container
    position: absolute
    width: 100%
    top: 50%
    transform: translateY(-50%)
</style>