<template>
  <div id="home">
  	<!--导航栏-->
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    
    <!--轮播图-->
    <!--<swiper>
    	<swiper-item v-for="item in banners" :key="index">
    		<a :href="item.link">
    			<img :src="item.image" alt=""/>
    		</a>
    	</swiper-item>
    </swiper>-->
    <scroll class="content" 
    	      ref="scroll" 
    	      :probe-type="3" 
    	      @scroll="contentScroll" 
    	      :pull-up-load="true" 
    	      @pullingUp="loadMore">
    <!--轮播图（封装）-->
    <home-swiper :banners="banners" />
    
    <!--推荐-->
    <recommend-view :recommends="recommends" />
    
    <feature-view />
    
    <tab-control class="tab-control" 
    	           :titles="['流行','新款','精选']"
    	           @tabClick="tabClick"/>
    
    <goods-list :goods="goods[currentType].list" />
    
    </scroll>
    
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
    
  </div>
</template>

<script>
	import NavBar from "components/common/navbar/NavBar"
	import TabControl from "components/content/tabControl/TabControl"
	import GoodsList from "components/content/goods/GoodsList"
	
	import HomeSwiper from "./childComps/HomeSwiper"
	import RecommendView from "./childComps/RecommendView"
	import FeatureView from "./childComps/FeatureView"
	
	import {getHomeMultidata,getHomeGoods} from "network/home"
	
  //	import {Swiper,SwiperItem} from "components/common/swiper"
  
  import Scroll from "components/common/scroll/scroll"
  
  import BackTop from "components/content/backTop/BackTop"
	
	
  export default {
    name: "Home",
    components:{
    	NavBar,
    	HomeSwiper,
    	RecommendView,
    	FeatureView,
    	TabControl,
    	GoodsList,
    	Scroll,
    	BackTop
    },
    data(){
    	return{
    		banners:[],
    		recommends:[],
    		goods:{
    			'pop':{page:0,list:[]},
    			'new':{page:0,list:[]},
    			'sell':{page:0,list:[]}
    		},
    		currentType:'pop',
    		isShowBackTop:false
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
    mouted(){
    	//3.监听item中图片加载完成(解决上拉图片卡顿问题)
    	this.$bus.$on('itemImageLoad',() =>{
    		this.$refs.scroll.refresh();
    	})
    },
    	methods:{
    		/**
    		 * 事件监听相关的方法
    		 */
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
    		},
    		
    		//返回上方
    		backClick(){
    	    	// 通过$refs拿到组件中的对象
    	    	this.$refs.scroll.scrollTo(0,0);
    	    	
    	  },
    	  
    	  contentScroll(position){
    	  	this.isShowBackTop = (-position.y) > 1000
    	  },
    	  
    	  //加载更多
    	  loadMore(){
    	  	this.getHomeGoods(this.currentType);
    	  },
    		/**
    		 * 网络请求相关方法
    		 */
    		  getHomeMultidata(){
    			  getHomeMultidata().then(res => {
    		      this.banners = res.data.banner.list;
    		      this.recommends = res.data.recommend.list;
    	      })
    		  },
    	    getHomeGoods(type){
    	    	const page = this.goods[type].page + 1;
    	    	getHomeGoods(type,page).then(res => {
    		      this.goods[type].list.push(...res.data.list);

    		      this.goods[type].page += 1;
//            this.goods[type].page = res.page;
              this.$refs.scroll.finishPullUp()
    	      })
    	    }
    	}
    
  }
</script>

<style scoped>
	#home{
		position: relative;
		/*padding-top: 44px;*/
		height: 100vh;
	}
	
  .home-nav{
  	background-color: var(--color-tint);
  	color: white;
  	position: fixed;
  	left: 0;
  	right: 0;
  	top: 0;
  	z-index: 9;
  }
  
  .tab-control{
  	position: sticky;
  	top: 44px;
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
 
  /*.content{
  	height: calc(100% - 93px);
  	overflow: hidden;
  	margin: 44px;
  }*/
  
</style>
