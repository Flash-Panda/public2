<style scoped>
	.x-swipe {
		overflow: hidden;
		position: relative;
		height: 100%;
	}
	.x-swipe-items-wrap {
		position: relative;
		overflow: hidden;
		height: 100%;
	}
	.x-swipe-items-wrap div{
		position: absolute;
		transform: translateX(-100%);
		width: 100%;
		height: 100%;
		display: none;
		align-items: center;
	}
	
	.x-swipe-items-wrap div.is-active{
		display: flex;
		transform: none;
	}
	
	
	.x-swipe-indicators {
		position: absolute;
		bottom: 10px;
		left: 50%;
		transform: translateX(-50%);
	}
	.x-swipe-indicator {
		width: 8px;
		height: 8px;
		display: inline-block;
		border-radius: 100%;
		background: #DCDCDC;
	 opacity: 0.2;
		margin: 0 3px;
	}
	.x-swipe-indicator.is-active {
		opacity: 1;
		background: #9AE8E3;
	}
	
</style>
<template>
	<div class="x-swipe">
		<div class="x-swipe-items-wrap" ref="wrap">
			<slot></slot>
		</div>
		<div class="x-swipe-indicators" v-show="showIndicators">
			<div class="x-swipe-indicator"
			     v-for="(page, $index) in pages"
			     :key="$index"
			     :class="{ 'is-active': $index === index }"></div>
		</div>
	</div>
</template>

<script>
    import { once, addClass, removeClass } from '../../utils/dom';
    export default {
        name: 'x-swipe',
        created() {
            this.dragState = {};
        },
        data() {
            return {
                ready: false,
                dragging: false,
                userScrolling: false,
                animating: false,
                index: 0,
                pages: [],
                timer: null,
                reInitTimer: null,
                noDrag: false,
                isDone: false
            };
        },
        props: {
            speed: {
                type: Number,
                default: 500
            },
            defaultIndex: {
                type: Number,
                default: 0
            },
            auto: {
                type: Number,
                default: 3000
            },
            continuous: {
                type: Boolean,
                default: true
            },
            showIndicators: {
                type: Boolean,
                default: true
            },
            noDragWhenSingle: {
                type: Boolean,
                default: true
            },
            prevent: {
                type: Boolean,
                default: false
            },
            stopPropagation: {
                type: Boolean,
                default: false
            }
        },
        watch: {
            index(newIndex) {
                this.$emit('change', newIndex);
            }
        },
        methods: {
            swipeItemCreated() {
                if (!this.ready) return;
                clearTimeout(this.reInitTimer);
                this.reInitTimer = setTimeout(() => {
                    this.reInitPages();
                }, 100);
            },
            swipeItemDestroyed() {
                if (!this.ready) return;
                clearTimeout(this.reInitTimer);
                this.reInitTimer = setTimeout(() => {
                    this.reInitPages();
                }, 100);
            },
            translate(element, offset, speed, callback) {
                if (speed) {
                    this.animating = true;
                    element.style.webkitTransition = '-webkit-transform ' + speed + 'ms ease-in-out';
                    setTimeout(() => {
                        element.style.webkitTransform = `translate3d(${offset}px, 0, 0)`;
                    }, 50);
                    let called = false;
                    let transitionEndCallback = () => {
                        if (called) return;
                        called = true;
                        this.animating = false;
                        element.style.webkitTransition = '';
                        element.style.webkitTransform = '';
                        if (callback) {
                            callback.apply(this, arguments);
                        }
                    };
                    once(element, 'webkitTransitionEnd', transitionEndCallback);
                    setTimeout(transitionEndCallback, speed + 100); // webkitTransitionEnd maybe not fire on lower version android.
                } else {
                    element.style.webkitTransition = '';
                    element.style.webkitTransform = `translate3d(${offset}px, 0, 0)`;
                }
            },
            reInitPages() {
                let children = this.$children;
                this.noDrag = children.length === 1 && this.noDragWhenSingle;
                let pages = [];
                let intDefaultIndex = Math.floor(this.defaultIndex);
                let defaultIndex = (intDefaultIndex >= 0 && intDefaultIndex < children.length) ? intDefaultIndex : 0;
                this.index = defaultIndex;
                children.forEach(function(child, index) {
                    pages.push(child.$el);
                    removeClass(child.$el, 'is-active');
                    if (index === defaultIndex) {
                        addClass(child.$el, 'is-active');
                    }
                });
                this.pages = pages;
            },
            doAnimate(towards, options) {
                if (this.$children.length === 0) return;
                if (!options && this.$children.length < 2) return;
                let prevPage, nextPage, currentPage, pageWidth, offsetLeft;
                let speed = this.speed || 300;
                let index = this.index;
                let pages = this.pages;
                let pageCount = pages.length;
                if (!options) {
                    pageWidth = this.$el.clientWidth;
                    currentPage = pages[index];
                    prevPage = pages[index - 1];
                    nextPage = pages[index + 1];
                    if (this.continuous && pages.length > 1) {
                        if (!prevPage) {
                            prevPage = pages[pages.length - 1];
                        }
                        if (!nextPage) {
                            nextPage = pages[0];
                        }
                    }
                    if (prevPage) {
                        prevPage.style.display = 'flex';
                        this.translate(prevPage, -pageWidth);
                    }
                    if (nextPage) {
                        nextPage.style.display = 'flex';
                        this.translate(nextPage, pageWidth);
                    }
                } else {
                    prevPage = options.prevPage;
                    currentPage = options.currentPage;
                    nextPage = options.nextPage;
                    pageWidth = options.pageWidth;
                    offsetLeft = options.offsetLeft;
                }
                let newIndex;
                let oldPage = this.$children[index].$el;
                if (towards === 'prev') {
                    if (index > 0) {
                        newIndex = index - 1;
                    }
                    if (this.continuous && index === 0) {
                        newIndex = pageCount - 1;
                    }
                } else if (towards === 'next') {
                    if (index < pageCount - 1) {
                        newIndex = index + 1;
                    }
                    if (this.continuous && index === pageCount - 1) {
                        newIndex = 0;
                    }
                }
                let callback = () => {
                    if (newIndex !== undefined) {
                        let newPage = this.$children[newIndex].$el;
                        removeClass(oldPage, 'is-active');
                        addClass(newPage, 'is-active');
                        this.index = newIndex;
                    }
                    if (this.isDone) {
                        this.end();
                    }
                    if (prevPage) {
                        prevPage.style.display = '';
                    }
                    if (nextPage) {
                        nextPage.style.display = '';
                    }
                };
                setTimeout(() => {
                    if (towards === 'next') {
                        this.isDone = true;
                        this.before(currentPage);
                        this.translate(currentPage, -pageWidth, speed, callback);
                        if (nextPage) {
                            this.translate(nextPage, 0, speed);
                        }
                    } else if (towards === 'prev') {
                        this.isDone = true;
                        this.before(currentPage);
                        this.translate(currentPage, pageWidth, speed, callback);
                        if (prevPage) {
                            this.translate(prevPage, 0, speed);
                        }
                    } else {
                        this.isDone = false;
                        this.translate(currentPage, 0, speed, callback);
                        if (typeof offsetLeft !== 'undefined') {
                            if (prevPage && offsetLeft > 0) {
                                this.translate(prevPage, pageWidth * -1, speed);
                            }
                            if (nextPage && offsetLeft < 0) {
                                this.translate(nextPage, pageWidth, speed);
                            }
                        } else {
                            if (prevPage) {
                                this.translate(prevPage, pageWidth * -1, speed);
                            }
                            if (nextPage) {
                                this.translate(nextPage, pageWidth, speed);
                            }
                        }
                    }
                }, 10);
            },
            next() {
                this.doAnimate('next');
            },
            prev() {
                this.doAnimate('prev');
            },
            before() {
                this.$emit('before', this.index);
            },
            end() {
                this.$emit('end', this.index);
            },
            doOnTouchStart(event) {
                if (this.noDrag) return;
                let element = this.$el;
                let dragState = this.dragState;
                let touch = event.touches[0];
                console.log(touch)
                dragState.startTime = new Date();
                dragState.startLeft = touch.pageX;
                dragState.startTop = touch.pageY;
                dragState.startTopAbsolute = touch.clientY;
                dragState.pageWidth = element.offsetWidth;
                dragState.pageHeight = element.offsetHeight;
                let prevPage = this.$children[this.index - 1];
                let dragPage = this.$children[this.index];
                let nextPage = this.$children[this.index + 1];
                if (this.continuous && this.pages.length > 1) {
                    if (!prevPage) {
                        prevPage = this.$children[this.$children.length - 1];
                    }
                    if (!nextPage) {
                        nextPage = this.$children[0];
                    }
                }
                dragState.prevPage = prevPage ? prevPage.$el : null;
                dragState.dragPage = dragPage ? dragPage.$el : null;
                dragState.nextPage = nextPage ? nextPage.$el : null;
                if (dragState.prevPage) {
                    dragState.prevPage.style.display = 'flex';
                }
                if (dragState.nextPage) {
                    dragState.nextPage.style.display = 'flex';
                }
            },
            doOnTouchMove(event) {
                if (this.noDrag) return;
                let dragState = this.dragState;
                let touch = event.touches[0];
                dragState.currentLeft = touch.pageX;
                dragState.currentTop = touch.pageY;
                dragState.currentTopAbsolute = touch.clientY;
                let offsetLeft = dragState.currentLeft - dragState.startLeft;
                let offsetTop = dragState.currentTopAbsolute - dragState.startTopAbsolute;
                let distanceX = Math.abs(offsetLeft);
                let distanceY = Math.abs(offsetTop);
                if (distanceX < 5 || (distanceX >= 5 && distanceY >= 1.73 * distanceX)) {
                    this.userScrolling = true;
                    return;
                } else {
                    this.userScrolling = false;
                    event.preventDefault();
                }
                offsetLeft = Math.min(Math.max(-dragState.pageWidth + 1, offsetLeft), dragState.pageWidth - 1);
                let towards = offsetLeft < 0 ? 'next' : 'prev';
                if (dragState.prevPage && towards === 'prev') {
                    this.translate(dragState.prevPage, offsetLeft - dragState.pageWidth);
                }
                this.translate(dragState.dragPage, offsetLeft);
                if (dragState.nextPage && towards === 'next') {
                    this.translate(dragState.nextPage, offsetLeft + dragState.pageWidth);
                }
            },
            doOnTouchEnd() {
                if (this.noDrag) return;
                let dragState = this.dragState;
                let dragDuration = new Date() - dragState.startTime;
                let towards = null;
                let offsetLeft = dragState.currentLeft - dragState.startLeft;
                let offsetTop = dragState.currentTop - dragState.startTop;
                let pageWidth = dragState.pageWidth;
                let index = this.index;
                let pageCount = this.pages.length;
                if (dragDuration < 300) {
                    let fireTap = Math.abs(offsetLeft) < 5 && Math.abs(offsetTop) < 5;
                    if (isNaN(offsetLeft) || isNaN(offsetTop)) {
                        fireTap = true;
                    }
                    if (fireTap) {
                        this.$children[this.index].$emit('tap');
                    }
                }
                if (dragDuration < 300 && dragState.currentLeft === undefined) return;
                if (dragDuration < 300 || Math.abs(offsetLeft) > pageWidth / 2) {
                    towards = offsetLeft < 0 ? 'next' : 'prev';
                }
                if (!this.continuous) {
                    if ((index === 0 && towards === 'prev') || (index === pageCount - 1 && towards === 'next')) {
                        towards = null;
                    }
                }
                if (this.$children.length < 2) {
                    towards = null;
                }
                this.doAnimate(towards, {
                    offsetLeft: offsetLeft,
                    pageWidth: dragState.pageWidth,
                    prevPage: dragState.prevPage,
                    currentPage: dragState.dragPage,
                    nextPage: dragState.nextPage
                });
                this.dragState = {};
            },
            initTimer() {
                if (this.auto > 0 && !this.timer) {
                    this.timer = setInterval(() => {
                        if (!this.continuous && (this.index >= this.pages.length - 1)) {
                            return this.clearTimer();
                        }
                        if (!this.dragging && !this.animating) {
                            this.next();
                        }
                    }, this.auto);
                }
            },
            clearTimer() {
                clearInterval(this.timer);
                this.timer = null;
            }
        },
        destroyed() {
            if (this.timer) {
                this.clearTimer();
            }
            if (this.reInitTimer) {
                clearTimeout(this.reInitTimer);
                this.reInitTimer = null;
            }
        },
        mounted() {
            this.ready = true;
            this.initTimer();
            this.reInitPages();
            
            //拿到swiper的dom
            let element = this.$el;
            //触屏前的点击事件
            element.addEventListener('touchstart', (event) => {
                //阻止元素发生默认行动，this.prevent不知道干嘛的
                if (this.prevent) event.preventDefault();
                //阻止冒泡事件，this.stopPropagation不知道干嘛的
                if (this.stopPropagation) event.stopPropagation();
                //是否动画中
                if (this.animating) return;
                
                this.dragging = true;
                this.userScrolling = false;
                //应该是执行动画了
                this.doOnTouchStart(event);
            });
            element.addEventListener('touchmove', (event) => {
                if (!this.dragging) return;
                if (this.timer) this.clearTimer();
                this.doOnTouchMove(event);
            });
            element.addEventListener('touchend', (event) => {
                if (this.userScrolling) {
                    this.dragging = false;
                    this.dragState = {};
                    return;
                }
                if (!this.dragging) return;
                this.initTimer();
                this.doOnTouchEnd(event);
                this.dragging = false;
            });
        }
    };
</script>