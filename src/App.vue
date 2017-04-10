<template>
  <div id="app">
      <!--  顶部导航条 -->
    <header class="header">
       <span v-if="!flag" @click="toggle(true)">首页</span>
       <span v-if="flag" @click="back()">返回</span>
    </header>
      <!-- 侧边栏 -->
      <aside class="aside" :class="{open:open,docked:docked}" @click="toggle()">
        <div class="aside-box"> 
             <!-- 用户中心 -->
          <div class="user_center">
              <!-- 用户登录信息 -->
             <div class="user_info">
                <div class="user_img"><img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1491644486167&di=6ed980741bcae88a0e8e1cc8dd25f1e1&imgtype=0&src=http%3A%2F%2Fcdn.duitang.com%2Fuploads%2Fitem%2F201604%2F11%2F20160411221803_5sLW8.thumb.700_0.jpeg" alt=""></div>
                <p class="user_name">wiestlee</p>
             </div>
             <!-- 用户收藏 离线下载 -->
             <div class="user_select">
                <div class="my_collect"><i class="iconfont icon-favorite"></i><span>我的收藏</span></div>
                <div class="my_download"><i class="iconfont icon-help"></i><span>离线下载</span></div>
             </div>
          </div>
            <!-- 知乎日报列表 -->

          <div class="news_list">
            <div class="first_page" @click="change(1)"><i class="iconfont icon-wxbzhuye"></i><span>首页</span></div>
            <ul class="news_lists">
              <li v-for="(x, index) in list" @click="change(index+2,x.id)"><span>{{x.name}}</span><i>+</i></li>
              <li><span>项目地址</span><i class="iconfont icon-github"/></i></li>
            </ul> 
          </div>
        </div>
           
      </aside>
      

      <div v-if="circle" class="circle" @click="top()">
          <i class="iconfont icon-ic_top"></i>
      </div>

     <transition :name="transitionName">
          <keep-alive>
            <router-view class="app-view" :class="{'app-view-hidden':docked}"></router-view>
          </keep-alive>
     </transition>
  </div>
</template>

<script>
import {mapState} from 'vuex'
import api from './api/index'
export default {
  computed:{
    ...mapState({
      circle: state => state.circleFlag,
      num: state => state.num,
      flag: state => state.drawer
    })
  },
  mounted: function () {
    let vue = this;
    api.getTopics().then(function (data) {
    vue.list=data.data.others

    })
  },
  data () {
    return {
      list: [],
      open: false,
      docked:false,
      transitionName:'slide-left'
    }
  },
  methods:{
    back(n){
      if(n){
        this.$route.push({
          path:'home'
        });
      } else {
        window.history.back()
      }
    },
    toggle(flag){
      if (!this.open){
        this.docked = true;
        this.open = true;
      }else{
        this.open = false;
        let vue = this;
        setTimeout(function(){
           vue.docked = false;
        },300);
      }
    },
    change(n, id) {
      let path = n == 1 ? 'home' : 'theme';
      this.$router.push({
        path: path,
        query: {
          id: id || ""
        }
      });
      this.$store.commit('add', n);
    },
    prevent(event) {
      event.preventDefault()
      event.stopPropagation()
    },
    top() {
      let vue = this;
      let dom = document.querySelector('.app-view');
      let height = dom.scrollTop;
      let scrollTop = parseInt(height/50);
      let time = setInterval(function() {
        height -= scrollTop;
        if (height <= 0) {
          dom.scrollTop = 0;
          vue.$store.commit('toggle');
          clearInterval(time);
        } else {
          dom.scrollTop = height;
        }
      }, 1);
    }
  }
}
</script>

<style lang="less">
@header:1.5rem;
.slide-left-enter,
.slide-right-leave-active {
    opacity: 0;
    -webkit-transform: translate(50vw, 0);
}

.slide-left-leave-active,
.slide-right-enter {
    opacity: 0.1;
    -webkit-transform: translate(-50vw, 0);
} 
.app-view {
    z-index: 1;
    width: 100vw;
    height: 100vh;
    overflow: auto;
    position: absolute;
    transition: transform 0.3s ease;
    -webkit-overflow-scrolling: touch;
}
.app-view-hidden {
    overflow: hidden;
}
.header{
  width:100%;
  height:@header;
  line-height:1.5rem;
  color:#fff;
  font-size:0.5rem;
  z-index:9;
  padding-left:5%;
  position:fixed;
  background:#00A2EA;
}
.icon-category{
  display:inline-block;
  font-weight:bold;
  margin-right:.5rem;
}
.aside{
  z-index:11;
  left:0;
  right:0;
  top:0;
  bottom:0;
  width:100%;
  position:fixed;
  transform:translate(-100%,0);
  transition:transform 0.3s ease;
  .aside-box{
    background:#F9F9F9;
    width:70%;
    height:100%;
  }
  &.open{
      transition:transform 0.3s ease;
      transform: translate(0,0);
  }
  &.docked{
      transition:all 0.3s ease;
  } 
   .user_center{
      height:120px;
      background:#00A2EA;
      color:#fff;
      padding:10px 15px;
     .user_info{
        height:40px;
        line-height:40px;
          .user_img{
            width:40px; 
            float:left;
            margin-top:5px;
            margin-right:0.5rem;
          }
          .user_name{
            font-size:.5rem;
          }
      }
      .user_select{
        display: flex;
        line-height: 26px;
        margin-top: 28px;
        font-size: 12px;
        .my_collect{
          flex: 1;
          .icon-favorite{
            padding:0 10px;
          }
        }
        .my_download{
          flex: 1;
          .icon-help{
            padding:0 10px;
          }
        }
      }
   }
} 

.news_list {
  width: 100%;
  height:80%;
  overflow:auto;
  overflow-scrolling:touch;
  -webkit-overflow-scrolling:touch;
  background:#F9F9F9;
  padding:10px 10px;
  font-size:0.4rem;
   &::-webkit-scrollbar {
          display: none!important;
          width: 0!important;
          height: 0!important;
          -webkit-appearance: none;
          opacity: 0!important;
      }
}
.news_list .news_lists li{
  cursor:pointer;
  height:40px;
  line-height:40px;
  display:flex;
}
.news_list .first_page{
  line-height:40px;
  color:#00A2EA;
}
.news_list .first_page .icon-wxbzhuye{
  padding:.5rem;
}
.news_list .news_lists li>span{
  flex:8;
}
.news_list .news_lists li>i{
  font-size:26px;
  color:#868686;
  flex:2;
}
.circle {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background: #3591f6;
    box-shadow: 0 2px 4px 0 rgba(0,0,0,0.50);
    right: 5%;
    bottom: 5vw;
    position: fixed;
    z-index: 10;
    i {
        top: 50%;
        left: 50%;
        font-size: 0.6rem;
        color: #F9F9F9;
        transform: translate(-50%, -50%);
        position: absolute;
    }
}
@media screen and (min-width: 640px){
  .app-view{
    width: 640px;
    left: 50%;
    transform: translate(-50%,0);
  }
  .aside ul{
    width: 350px;
  }
}
</style>

