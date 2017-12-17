<template>
  <div class="swiper-page" v-title data-title="swiper">
    <p>正常swiper</p>
    <x-swiper :items='items' :styles="{height: '200px'}" @change="changeHandle"></x-swiper>
    <p>缩略swiper</p>
    <x-swiper type='thum' :items='items' :styles="{height: '240px'}"></x-swiper>
  </div>
</template>

<script>
  import xSwiper from './swipe/index.vue'
    export default {
        data () {
            return {
                items: [
                    'http://science.china.com.cn/images/attachement/jpg/site555/20161027/e89a8ffb1393197b9fbf24.JPG',
                    'http://science.china.com.cn/images/attachement/jpg/site555/20161027/e89a8ffb1393197b9fbf24.JPG',
                    'http://science.china.com.cn/images/attachement/jpg/site555/20161027/e89a8ffb1393197b9fbf24.JPG',
                    ],
            }
        },
        components: {
            xSwiper
        },
        methods: {
            changeHandle (index) {
                //console.log(index);
            }
        }
    }
</script>

<style scoped>
  .swiper-page {
    height: auto;
    text-align: center;
   
  }
  p {
    margin: 10px 0;
  }
</style>