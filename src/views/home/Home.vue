<template>
	<div id="home">
    <nav-bar class="home-nav"><template v-slot:center><div>购物街</div></template></nav-bar>

    <tab-control :titles="['流行', '新款', '精选']"
                 @tabClick="tabClick"
                 ref="tabControl1"
                 class="tabControl"
                 v-show="isTabFixed"/>

    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"/>
      <feature-view/>
      <tab-control :titles="['流行', '新款', '精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"/>
      <goods-list :goods="showGoods"/>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
	</div>
</template>

<script>
  import HomeSwiper from "@/views/home/childComps/HomeSwiper";
  import RecommendView from "@/views/home/childComps/RecommendView";
  import FeatureView from "@/views/home/childComps/FeatureView";

  import NavBar from "@/components/common/navbar/NavBar";
  import TabControl from "components/content/tabControl/TabControl";
  import GoodsList from 'components/content/goods/GoodsList'
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from "@/components/content/backTop/BackTop";

  import {getHomeMultidata, getHomeGoods} from "network/home";
  import {debounce} from "@/common/utils";

  export default {
		name: "Home",
    data() {
      return {
        // result: null
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []}
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0,
        saveYPop: 0,
        saveYNew: 0,
        saveYSell: 0
      }
    },
    components: {
      HomeSwiper,
      RecommendView,
      FeatureView,
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
    destroyed() {
      console.log('home destroyed')
    },
    activated() {
      this.$refs.scroll.refresh()
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      console.log('activated')
    },
    deactivated() {
      this.saveY = this.$refs.scroll.getScrollY()
      console.log('deactivated')
    },
    created() {
      //1.请求多个数据
      this.getHomeMultidata()
      //2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
    },
    //不要在created里用$refs或者document.querySelector()
    //还没挂载完成会拿不到dom元素
    mounted() {
      //1.图片加载完成的事件监听
      const refresh = debounce(this.$refs.scroll.refresh, 500)
      this.$bus.$on('itemImageLoad', () => {
        // console.log('-----')
        refresh()
      })
    },
    methods: {
      /**
       * 事件监听相关方法
       */
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = 'pop';
            //切换
            this.tabControlChange(this.saveYPop)
            break
          case 1:
            this.currentType = 'new'
            this.tabControlChange(this.saveYNew)
            break
          case 2:
            this.currentType = 'sell'
            this.tabControlChange(this.saveYSell)
            break
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
        console.log(index)
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0, 500)
      },
      contentScroll(position) {
        //1.判断BackTop是否显示
        this.isShowBackTop = (-position.y) > 1000

        //2.决定tabControl是否吸顶(position:fixed)
        this.isTabFixed = (-position.y) > this.tabOffsetTop

        //记录pop,new,sell的y值
        switch (this.currentType){
          case "pop":
            this.saveYPop = position.y
            break
          case "new":
            this.saveYNew = position.y
            break
          case "sell":
            this.saveYSell = position.y
            break
        }
      },
      loadMore() {
        this.getHomeGoods(this.currentType)
      },
      swiperImageLoad() {
        //获取tabControl的offsetTop
        //所有的组件都有一个属性$el: 用于获取组件中的元素
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
      },
      //tabControl切换时的问题
      tabControlChange(typeY) {
        //是否有其中一个已经到了吸顶状态
        if(-this.saveYNew > this.tabOffsetTop || -this.saveYPop > this.tabOffsetTop || -this.saveYSell > this.tabOffsetTop){
          if(-typeY < this.tabOffsetTop){
            //如果还没开始点或者还未去到吸顶状态，点击则去到吸顶位置
            this.$refs.scroll.scrollTo(0, -this.tabOffsetTop, 0)
          }else {
            //回到原来位置
            this.$refs.scroll.scrollTo(0, typeY, 0)
          }
        }
        //吸顶状态
        this.isTabFixed = (-this.$refs.scroll.getScrollY()) > this.tabOffsetTop
      },

      /**
       * 网络请求相关方法
       */
      getHomeMultidata() {
        //1.请求多个数据
        getHomeMultidata().then(res => {
          // this.result = res;
          // console.log(res)
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
          // console.log(this.banners)
          // console.log(this.recommends)
        })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type,page).then(res => {
          console.log(res)
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1

          //完成上拉加载更多
          this.$refs.scroll.finishPullUp()
        })
      }

    }
  }
</script>

<style scoped>
  #home {
    /*padding-top: 44px;*/
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;
    /*在使用浏览器原生滚动时，为了让导航不跟随一起滚动*/
    /*position: fixed;*/
    /*left: 0px;*/
    /*right: 0px;*/
    /*top: 0px;*/
    /*z-index: 9;*/
  }

  .content {
    /*height: 300px;*/
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  /*.content {*/
  /*  height: calc(100% - 93px);*/
  /*  overflow: hidden;*/
  /*  margin-top: 51px;*/
  /*}*/

  .tabControl {
    position: relative;
    z-index: 9;
  }
</style>
