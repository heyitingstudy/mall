<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">
        购物街
      </div>
    </nav-bar>
    <tab-control class="tab-control"
                  :titles="['流行','新款','精选']"
                  @tabClick="tabClick"
                  ref = "tabControl1"
                  v-show="isTabFixed"
                  ></tab-control>
    <scroll class="content" ref="scroll" 
            :probe-type="3" 
            @scroll="contentScroll"
            :pull-up-load="true" 
            @pullingUp = "loadMore">
      <home-swiper :banners='banners'
                    @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends='recommends'></recommend-view>
      <feature-view></feature-view>
      <tab-control :titles="['流行','新款','精选']"
                  @tabClick="tabClick"
                  ref = "tabControl2"
                  ></tab-control>
      <good-list :goods="showGoods"></good-list>

    </scroll>
    <!--当我们需要监听一个组件的原生事件时，必须给对应的事件加上.native修饰符-->
    <back-top @click.native="backTop" v-show="isShowBackTop"></back-top>

  </div>
</template>

<script>

import HomeSwiper from './childComps/HomeSwiper';
import RecommendView from './childComps/RecommendView';
import FeatureView from './childComps/FeatureView';

import NavBar from 'components/common/navbar/NavBar';
import TabControl from 'components/content/tabControl/TabControl';
import GoodList from 'components/content/goods/GoodsList';
import Scroll from 'components/common/scroll/Scroll';


import {
  getHomeMultidata,
  getHomeGoods
  } from 'network/home';
import {debounce} from 'common/utils';
import {itemListenerMixin, backTopMixin} from 'common/mixin';
import {BACK_POSITION} from 'common/const';

export default {
  name: 'Home',
  components:{
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatureView,
    TabControl,
    GoodList,
    Scroll,
  },
  data() { 
    return {
      result: null,
      banners: [],
      recommends: [],
      goods:{
        'pop':{page: 0, list: []},
        'new':{page: 0, list: []},
        'sell':{page: 0, list: []},
      },
      currentType:'pop',

      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0,
    }
  },
  mixins:[itemListenerMixin, backTopMixin],

  computed:{
    showGoods(){
      return this.goods[this.currentType].list;
    }
  },
  created(){
    //1.请求多个数据
    this.getHomeMultidata();

    //2.请求商品数据
    this.getHomeGoods('pop');
    this.getHomeGoods('new');
    this.getHomeGoods('sell');

    
  },
  activated(){
    this.$refs.scroll.scrollTo(0, this.saveY, 0);
    this.$refs.scroll.refresh();
  },
  deactivated(){
    this.saveY = this.$refs.scroll.getScrollY();

    //取消监听
    this.$bus.$off('itemImgLoad', this.itemImgListener)
  },
  methods:{
    /**
     * 事件监听相关的方法
     */
    loadMore(){
      this.getHomeGoods(this.currentType);
      this.$refs.scroll.refresh();
    },

    tabClick(index){
      switch(index){
        case 0:
          this.currentType = 'pop';
          break;
        case 1:
          this.currentType = 'new';
          break;
        case 2:
          this.currentType = 'sell';
          break;
      }
      this.$refs.tabControl1.currentIndex = index;
      this.$refs.tabControl2.currentIndex = index;
      
    },
    contentScroll(position){
      //1.判断BackTop是否显示
      this.listenShowBackTop(position);

      //2.决定tabControl是否吸顶
      this.isTabFixed = (-position.y) > this.tabOffsetTop;
    },
    swiperImageLoad(){
      //2.获取tabControl的offsetTop
      // 所有的组件都有$el 属性，用于获取组件中的元素
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
   },
    /**
     * 网络请求相关的方法
     */
    getHomeMultidata(){
      getHomeMultidata().then(res =>{
        console.log(res); 
        this.result = res;
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      })
    },
    getHomeGoods(type){
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then(res =>{
        //console.log(res);
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;

        //完成上拉
        this.$refs.scroll.finishPullUp();
      })
    }
  }
 }
</script>

<style  scoped>
#home{
  /* padding-top: 44px; */
  height: 100vh;
  position: relative;
}
.home-nav{
  background-color: var(--color-tint);
  color: #fff;
}
.tab-control{
  position: relative;
  
  z-index: 9;
}
.content{
    overflow: hidden;

    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
}
</style> 