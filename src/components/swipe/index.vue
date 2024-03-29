<template>
	<div class="x-swiper" :style="styles">
		<div class="x-default-swiper" @click="swiperClick">
			<swipe
					:auto="auto"
					:defaultIndex="currentIndex"
					:showIndicators="showIndicators"
					@change="changeIndex"
			>
				<swipe-item v-for="(item, index) in items" :key="index" @click.native="urlJump(item.url)">
					<img :src="item.pic || item" alt="">
				</swipe-item>
			</swipe>
		</div>
		<div class="x-thum-fixed" v-if="type==='thum' && showThum" @click="swiperClose">
			<div class="x-thum-swiper">
				<swipe
						:auto="0"
						:defaultIndex="currentThumIndex"
						:showIndicators="showIndicators"
						@change="changeThumIndex"
				>
					<swipe-item v-for="(item, index) in items" :key="index" :scale="true">
						<img :src="item.pic || item" alt="">
					</swipe-item>
				</swipe>
			</div>
		</div>
	</div>
</template>

<script>
     import swipe from './swipe.vue'
     import swipeItem from './swipe_item.vue'
     
    export default {
        name: 'x-swiper',
        props: {
            type: {
                type: String,
                default: 'swiper'
            },
            auto: {
                type: Number,
                default: 5000
            },
            items: {
                type: Array,
                default: function () {
                    return []
                }
            },
            showIndicators: {
                type: Boolean,
                default: true
            },
            styles: {
                type: Object,
                default: function () {
                    return {}
                }
            },
            resetTitle: String
        },
        data () {
            return {
                currentIndex: 0,
                currentThumIndex: 0,
                showThum: false,
                timer: null
            }
        },
        computed: {
            title () {
                return `${this.currentThumIndex+1}/${this.items.length}`
            }
        },
        methods: {
            swiperClick () {
                if (this.type === 'swiper') {
                    return
                } else if (this.type === 'thum') {
                    this.showThum = true
                    this.currentThumIndex = this.currentIndex
                }
            },
            urlJump (url) {
                if (this.type === 'swiper' && url) {
                    window.location.href = url
                }
            },
            swiperClose () {
                this.showThum = false
            },
            changeIndex (newIndex) {
                this.currentIndex = newIndex
                this.$emit('change', newIndex)
            },
            changeThumIndex (newIndex) {
                this.currentThumIndex = newIndex
            }
        },
         components: {
             swipe,
             swipeItem
         }
    }
</script>

<style scoped>
	.x-swiper {
		width: 100%;
		height: 4rem;
		position: relative;
	}
	.x-default-swiper{
		width: 100%;
		height: 100%;
	}
	.x-default-swiper img{
		width: 100%;
	}
	.x-thum-fixed {
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		background: #37373D;
		z-index: 999;
		display: flex;
		align-items: center;
		justifx-content: center;
		
	}
	.x-thum-swiper {
		width: 100%;
		height: 100%;
		
	}
	.x-thum-swiper img{
		width:100%;
	}
	.x-thum-swiper .current-page {
		position: absolute;
		top: 10px;
		left: 50%;
		transform: translateX(-50%);
		color: #fff;
		font-size: 14px;
	}
	@media (orientation: landscape) {
		.x-swiper .x-thum-fixed .x-thum-swiper img {
			width: auto;
			height: 100%;
			margin: auto;
		}
	}
	@media (min-width: 568px) and (max-width: 736px) and (orientation: landscape) {
		.x-swiper {
			height: 6.5rem !important
		}
	}
	@media (min-width: 768px) and (max-width: 1023px) {
		.x-swiper {
			height: 8rem !important
		}
	}
	@media (min-width: 1024px) and (max-width: 1365px) {
		.x-swiper {
			height: 9rem !important
		}
	}
	@media (min-width: 1366px) {
		.x-swiper {
			height: 10rem !important
		}
	}
</style>