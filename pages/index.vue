<template>
  <section class="indexContainer">
    
    <!-- banner区域 -->
      <div class="bannerContainer">
          <!-- <div class="container QR"> -->
              <!-- 二维码 -->
              <!-- <q-r-code></q-r-code> -->
          <!-- </div> -->
      </div>
      <div class="container contentContainer">
          <div class="briefTitle">
              <!-- 简略数据 -->
              <brief-title></brief-title>
          </div>
          <!-- 主题标题 -->
          <div class="themeTitle">
              <div class="row">
                  <div class="col col-md-6">
                      <theme-module></theme-module>
                  </div>
                  <div class="col col-md-6">
                      <theme-charts></theme-charts>
                  </div>
              </div>
          </div>
          <!-- 关注主题 -->
          <div class="followTheme">
              <follow-theme ></follow-theme>
          </div>

          <!-- 资讯 -->
          <div class="themeInfo">
              <theme-info></theme-info>
          </div>
        
      </div>
    
  </section>
</template>

<script>

import Logo from '~/components/Logo.vue'
// import QRCode from '~/components/QRCode'
import BriefTitle from '~/components/indexComponents/BriefTitle'
import ThemeModule from '~/components/indexComponents/ThemeModule'
import ThemeCharts from '~/components/indexComponents/themeCharts'
import FollowTheme from '~/components/indexComponents/FollowTheme'
import ThemeInfo from '~/components/indexComponents/ThemeInfo'
import InfoList from '~/components/indexComponents/InfoList'
import { mapActions, mapGetters } from 'vuex'
import axios from 'axios'
import Util from '~/store/util';

export default {
  layout: 'Dashboard',
  components: {
    Logo,
    // QRCode,
    BriefTitle,
    ThemeModule,
    ThemeCharts,
    FollowTheme,
    ThemeInfo
  },
//   head () {
//     return {
//       title: this.title,
//       meta: [
//         { hid: 'description', name: 'description', content: 'My custom description' }
//       ]
//     }
//   },

    async fetch ({ store, params }) {
        await Promise.all([
            axios.get(Util.baseUrl() + 'index'),
            axios.get(Util.baseUrl() + 'conceptSort?num=3'),
            axios.get(Util.baseUrl() + 'todayAttentionConcept')
        ])
        .then(res => {
            // console.log(res);
            let [ market_index, lead, followTheme, flow ] = res;
            
            let id = '';
            // if(market_index.data.informations.topInformationMap){
                // for(var key in  market_index.data.informations.topInformationMap){
                
                //     id = market_index.data.informations.topInformationMap[key].id;
                // }
            // }
            
            store.commit('LEAD_UP_DOWN_THEME', lead.data);
            

            store.commit('MARKET_INDEX', {
                market: [
                    ...market_index.data.sectionExponential
                ],
                updown:{
                    upsNum: market_index.data.upsNum,
                    downNum: market_index.data.downNum,
                    dogfallNum: market_index.data.dogfallNum
                } 
            });
            store.commit('INFOMATIONS_INDEX', market_index.data.informations);

            store.commit('TODAY_FOLLOW_THEME', followTheme.data);
            
            
            
            // return Promise.all([
            //     axios.get(Util.baseUrl() + 'informationAssociateTheme?informationId=' + id),
            //     // axios.get(Util.baseUrl() + 'getConceptStockHeroListByDate?searchDate=1&conceptId=' + id)
            
            // ]);
            
            
        })
        // .then(res => {
        //     let [ theme ] = res;
        //     store.commit('THEME_CONCEPT_INFO', theme.data);
        //     // store.commit('THEME_TRANS_BILLBOARD', billboard.data);
            
        // });

    },
    

    beforeMount(){
        // console.log(this.ieVersion());

        // if(this.ieVersion() <= 10 && this.ieVersion() !== -1){
        //     this.$router.push({
        //         path: '/upgrade'
        //     })
        // }
    },

  mounted(){
      
    // console.log(this);
    this.getIndexData();
    // this.getLeadTheme({num: 3});
    this.getCapitalFlow();
    // this.themeFlowTopFive();
    // 定时发起请求
    // this.intervalGet();
  },

    destroyed(){
        //  清除缓存
        
        clearInterval(this.timer);
    },

  methods: {
      ...mapActions(['getIndexData','getMarket', 'themeFlowTopFive', 'getLeadTheme', 'getCapitalFlow']),
      stockMarketDate(){
            // 现在的时间戳
            let nowDate = (new Date()).getTime();
            // 现在的周几
            let nowDateWeekday = (new Date()).getDay();

            // 获取当天
            let now = (new Date()).format('YYYY-MM-DD');
            // 当天 上午9:30 时间戳
            let am_Nine = new Date(`${now} 09:30:00`);
            // 当天 上午11:30 时间戳
            let am_Eleven = new Date(`${now} 11:30:00`);
            // 当天 下午1:00 时间戳
            let pm_One = new Date(`${now} 13:00:00`);
            // 当天 下午3:00 时间戳
            let pm_Three = new Date(`${now} 15:30:00`);

            // 如果在工作日
            if(nowDateWeekday >= 1 && nowDateWeekday <=5 ){
                if(nowDate >= am_Nine && nowDate <= am_Eleven){
                    // console.log(this.currentInfo);
                   
                        this.getMarket();
                        this.getLeadTheme({num: 3});
                        this.getCapitalFlow();
                    
                }

                if(nowDate >= pm_One && nowDate <= pm_Three){
                        this.getMarket();
                        this.getLeadTheme({num: 3});
                        this.getCapitalFlow();
                }
            }else{
                clearInterval(this.timer);
            }

        },

        // // 定时发起请求

        intervalGet(){
            this.timer = setInterval(()=>{

                this.stockMarketDate();
            }, 6000);
        },

        ieVersion() {
            var userAgent = navigator.userAgent; //取得浏览器的userAgent字符串  
            var isIE = userAgent.indexOf("compatible") > -1 && userAgent.indexOf("MSIE") > -1; //判断是否IE<11浏览器  
            var isEdge = userAgent.indexOf("Edge") > -1 && !isIE; //判断是否IE的Edge浏览器  
            var isIE11 = userAgent.indexOf('Trident') > -1 && userAgent.indexOf("rv:11.0") > -1;
            if(isIE) {
                var reIE = new RegExp("MSIE (\\d+\\.\\d+);");
                reIE.test(userAgent);
                var fIEVersion = parseFloat(RegExp["$1"]);
                if(fIEVersion == 7) {
                    return 7;
                } else if(fIEVersion == 8) {
                    return 8;
                } else if(fIEVersion == 9) {
                    return 9;
                } else if(fIEVersion == 10) {
                    return 10;
                } else {
                    return 6;//IE版本<=7
                }   
            } else if(isEdge) {
                return 'edge';//edge
            } else if(isIE11) {
                return 11; //IE11  
            }else{
                return -1;//不是ie浏览器
            }
        }
  }
}
</script>
