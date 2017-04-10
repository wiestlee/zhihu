00<template>
<div>
    <m-swipe swipeid="swipe" ref="swiper" :autoplay="3000" effect="slide">
        <div @click="go(top.id)" v-for="top in tops" class="swiper-slide" slot="swiper-con">
            <img :src="top.image">
            <div></div>
            <h3>{{top.title}}</h3>
        </div>
    </m-swipe>
   
 <div class="news-box" v-for="x in list">
      <p class="date-box">
          {{x.date.substring(0,4)+"/"+x.date.substring(4,6)+"/"+x.date.substring(6,8)}}
      </p>
      <div class="news-lable" link @click="go(y.id)" v-for="y in x.stories">
          <img v-if="y.images[0]" :src="y.images[0]" />
          <p :class="{'news-title':y.images[0],'news-center':!y.images[0]}">{{y.title}}</p>
      </div>
  </div>

	<div class="loading" v-if="!list.length&&!refreshing">
        <loading></loading>
		<!-- 被注释的loading
            <span class="left"></span>
            <span class="middle"></span>
            <span class="right"></span> 
        -->
	</div>
	<!-- 滑动加载更多组件 -->
	<infinite-scroll :scroller="scroller" :loading="loading" @load="loadMore" />
	<!-- 回到顶部组件 -->
	<back-scroll :scroller="scroller" :flag="circle"></back-scroll>
</div>
</template>
<script>
import api from '../api/index'
import {
	mapState
} from 'vuex'
import loading from '../components/loading'
export default {
	computed: {
		...mapState({
			circle: state => state.circleFlag,
		})
	},
	mounted() {
		this.getList(1);
		this.scroller = this.$el;
		let swiper = this.$refs.swiper;
		if (swiper.dom) {
			this.swiper = swiper.dom;
		}
	},
	activated() {
		if (this.swiper) {
			this.swiper.startAutoplay();
		}
	},
	deactivated() {
		this.loop = false;
		if (this.swiper) {
			this.swiper.stopAutoplay();
		}
	},
	data() {
		return {
			refreshing: false,
			trigger: null,
			loading: false,
			count: 1,
			scroller: null,
			list: [],
			swiper: "",
			tops: []
		}
	},
	methods: {
		go(id) {
			this.$router.push({
				path: "con",
				query: {
					id: id
				}
			});
		},
		getList(type) {
			var vue = this;
			if (type) {
				api.getNews().then(function(data) {
					vue.tops = data.data.top_stories;
					vue.list.push(data.data);
					vue.loading = false;

				});
			} else {
				api.getNewsByDate(vue.GetDate(vue.count)).then(function(data) {
					vue.list.push(data.data);
					vue.loading = false;

				});
			}
		},
		loadMore() {
			let vue = this;
			this.loading = true;
			setTimeout(() => {
				vue.count--;
				vue.getList();
			}, 500)
		},
		GetDate(Count) {
			var dd = new Date();
			dd.setDate(dd.getDate() + Count); //获取AddDayCount天后的日期
			var y = dd.getFullYear();
			var m = dd.getMonth() + 1; //获取当前月份的日期
			m = m > 10 ? m : "0" + m
			var d = dd.getDate();
			d = d >= 10 ? d : "0" + d;
			return y + "" + m + "" + d;
		}
	},
    components:{loading}
}
</script>
<style lang="less">
@yellow: #FFD300;
@blue: #5B7492;
@gray: #acb9c8;
.news-box {
    width:100%;
 .date-box{
    width:180px;
    height:30px;
    line-height:30px;
    background-color:#39BCF7;
    border-radius:20px;
    box-shadow:5px 5px 5px #C1E0EA;
    opacity:0.8;
    margin:10px auto;
    color:#fff;
    font-weight:bold;
    font-size:0.46rem;
    text-align:center;
  }
  .news-lable{
    width:100%;
    height:100px;
    padding:10px 20px;
    background:#fefefe;
    box-sizing:border-box;
    overflow:hidden;
    border-bottom:1px dashed #ddd;
    .news-title{
     font-size:0.42rem;
     color:#5C7592;
     margin-top:15px;
     text-align:left;
     width:70%;
    }
    .news-center{
     font-size:0.42rem;
     color:#5C7592;
     margin-top:15px;
     text-align:left;
     width:100%; 
    }
    img{
     display:block;
     float:right;
     margin:5px 10px;
     width:70px;
     height:70px;
     border-radius:10px;
     border:1px solid #fefefe;
   
    }
 }
}


.app-view {
    .swiper-container {
        width: 100%;
    }
    .swiper-slide {
        height: 8rem;
        overflow: hidden;
        position: relative;
    }
    .swiper-container-horizontal > .swiper-pagination-bullets {
        bottom: 1rem;
        width: 95%;
        text-align: right;
    }
    .list:nth-child(2) {
        margin-bottom: 0;
        padding-top: 0;
        .list-time {
            top: -.8rem;
        }
    }
}
.swiper-slide {
    div {
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        opacity: 0.4;
        position: absolute;
        background-color: @blue;
    }
    img {
        top: 50%;
        position: relative;
        transform: translate(0, -50%);
    }
    h3 {
        width: 70%;
        color: #fff;
        margin: 0;
        font-size: 0.5rem;
        line-height: 1rem;
        right: 5%;
        bottom: 2.6rem;
        text-align: right;
        position: absolute;
        text-shadow: 1px 1px 10px rgba(0, 0, 0, .5);
        &:before {
            content: "";
            width: 3rem;
            bottom: -.6rem;
            right: 0;
            display: block;
            position: absolute;
            border: 2px solid @yellow;
        }
    }
}
.list {
    width: 90%;
    z-index: 1;
    position: relative;
    padding-top: 0.8rem;
    margin: .8rem auto 0;
    &-time {
        top: 0;
        margin: 0;
        color: #fff;
        padding: 0 1rem;
        font-size: 0.4rem;
        line-height: 0.8rem;
        letter-spacing: 0.1rem;
        border-radius: 0.4rem;
        text-align: center;
        background-color: @yellow;
        transform: translate(0,-50%);
        position: absolute;
        box-shadow: 0 3px 10px 0 rgba(91,115,146,0.15);
    }
    &-con {
	cursor: pointer;
        display: flex;
        padding: 0.3rem;
        margin-bottom: 0.4rem;
        border-radius: 0.15rem;
        align-items: center;
        background-color: #fff;
        box-shadow: 0 3px 10px 0 rgba(91,115,146,0.15);
        img {
            width: 2rem;
            margin-right: 0.4rem;
        }
        p {
            color: @blue;
            font-size: 0.4rem;
            text-align: justify;
            margin: 0;
        }
    }
}
/****** 
 ****
 ***
 注释的loading css
.loading {
    width: 25%;
    height: 0.4rem;
    margin: 25% auto 0;
    position: relative;
    span {
        display: block;
        width: 0.4rem;
        height: 0.4rem;
        border-radius: 50%;
        position: absolute;
        background: @blue;
        transform: translate(-50%,0);
    }
    .left {
        background: @yellow;
        animation: lMove 2.5s ease infinite;
    }
    .middle {
        left: 50%;
        animation: mMove 2.5s ease infinite;
    }
    .right {
        left: 100%;
        background: @gray;
        animation: rMove 2.5s ease infinite;
    }
}
@keyframes lMove {
    0% {
        left: 0;
    }
    25% {
        left: 50%;
        background: @yellow;
    }
    50% {
        left: 100%;
        background: @blue;
    }
    75% {
        left: 50%;
        background: @gray;
    }
    100% {
        left: 0;
    }
}
@keyframes mMove {
    0% {}
    25% {
        background: @blue;
    }
    50% {
        background: @yellow;
    }
    75% {
        background: @blue;
    }
    100% {}
}
@keyframes rMove {
    0% {
        left: 100%;
    }
    25% {
        left: 50%;
    }
    50% {
        left: 0;
        background: @gray;
    }
    75% {
        left: 50%;
        background: @yellow;
    }
    100% {
        left: 100%;
    }
} 
 *
 */
</style>
