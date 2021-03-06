<template>
	<div class="ck-swiper" :style="{width, height}">
		<ul class="ck-swiper-imglist">
			<slot></slot>
		</ul>
		<ul class="ck-dot-wrapper">
			<li class="ck-dot" v-for="item in imgItemLen" :key="item" :class="{'dot-active': item === now + 1}"></li>
		</ul>
	</div>
</template>

<script>
export default {
	props: {
		width: {
			type: String,
			default: '100%'
		},
		height: {
			type: String,
			default: 'auto'
		},
		delay: {
			type: Number,
			default: 2000
		}
	},
	data() {
		return {
			swiperDom: null,
			listDom: null,
			swiperItemDom: [],
			now: 0,        // 当前在哪一张图片
			translatex: 0, // 鼠标移动的距离
			startPoint: {}, // 鼠标点击初始位置
			isMove: false,  // 当前是否在移动
			distaince: {},  // 移动距离
			startOffset: 0, // 记录当前图片的x位置
			swiperWidth: 0, // 组件长度
			proportion: 0.3, // 拖拽最大限度比例
			directionRight: 1, // 方向是否向右
			hasNoImg: false,
			imgItemLen: 0,
			timer: null,
		}
	},
	methods: {
		gotoNextImage() {
			this.translatex = this.now * -this.swiperWidth;
			this.listDom.style.transition = '0.3s';
			this.listDom.style.transform = `translateX(${this.translatex}px)`;
		},
		timeInterval() {
			let _this = this
			this.timer = setInterval(() => {
				if (this.now < this.imgItemLen - 1) {
					this.now++
					this.gotoNextImage()
				} else {
					this.listDom.style.transition = 'none'
					this.swiperItemDom[this.imgItemLen - 1].style.transform = `translateX(${-this.swiperWidth * this.imgItemLen}px)`
					this.listDom.style.transform = `translateX(${this.swiperWidth}px)`

					new Promise((resolve) => {
						setTimeout(() => {
							_this.now = 0
							_this.listDom.style.transition = '0.3s'
							_this.listDom.style.transform = `translateX(0)`
						})
						setTimeout(() => {
							resolve()
						}, 400)
					}).then(() => {
						_this.listDom.style.transition = 'none'
						_this.swiperItemDom[_this.imgItemLen - 1].style.transform = `translateX(0)`
						_this.listDom.style.transform = `translateX(0)`
					})
				}
			}, this.delay)
		}

	},
	mounted() {
		this.swiperDom = document.getElementsByClassName('ck-swiper')[0]
		this.listDom = document.getElementsByClassName('ck-swiper-imglist')[0]
		this.swiperItemDom = document.querySelectorAll('.ck-swiper-item')
		this.imgItemLen = this.swiperItemDom.length
		this.swiperWidth = this.swiperDom.clientWidth
		this.timeInterval()
		this.swiperDom.addEventListener('touchstart', e => {
			let touch = e.changedTouches[0]
			clearInterval(this.timer)
			this.startPoint = {
				x: touch.pageX,
				y: touch.pageY
			}
			this.startOffset = this.now * -this.swiperWidth
			this.listDom.style.transition = 'none'
		})

		this.swiperDom.addEventListener('touchmove', e => {
			let touch = e.changedTouches[0]
			this.distaince = {
				x: touch.pageX - this.startPoint.x
			}
			if (Math.abs(this.distaince.x) > 5) {
				this.isMove = true
				if (this.distaince.x > 0) {
					this.directionRight = -1
					if (this.now == 0) {
						this.hasNoImg = true;
						this.swiperItemDom[this.imgItemLen - 1].style.transform = `translateX(${-this.swiperWidth * this.imgItemLen}px)`
					}
				} else {
					this.directionRight = 1
					if (this.now == this.imgItemLen - 1) {
						this.hasNoImg = true;
						this.swiperItemDom[0].style.transform = `translateX(${this.swiperWidth * this.imgItemLen}px)`
					}
				}
			} else {
				this.isMove = false
			}
			this.translatex = this.startOffset + this.distaince.x
			if (this.isMove) {
				this.listDom.style.transform = `translateX(${this.translatex}px)`
			}
		})

		this.swiperDom.addEventListener('touchend', () => {
			if (Math.abs(this.distaince.x) > this.swiperWidth * this.proportion) {
				this.now += this.directionRight
			}
			if (this.isMove) {
				this.gotoNextImage()

				// 到达边界并且移动操作30%需要初始化位置 
				if (this.hasNoImg && Math.abs(this.distaince.x) > this.swiperWidth * this.proportion) {
					setTimeout(() => {
						this.listDom.style.transition = 'none'
						this.hasNoImg = false
						if (this.directionRight === 1) {
							this.now = 0;
							this.listDom.style.transform = `translateX(0)`;
						} else {
							this.now = this.imgItemLen - 1
							this.listDom.style.transform = `translateX(${-this.swiperWidth * (this.imgItemLen - 1)}px)`;
						}
						this.swiperItemDom[this.imgItemLen - 1].style.transform = `translateX(0)`
						this.swiperItemDom[0].style.transform = `translateX(0)`
					}, 300);
				}
			}
			setTimeout(() => {
				this.timeInterval()
			}, 300)
		})
	}
}
</script>

<style lang="scss" scoped>
ul {
	margin: 0;
	padding: 0;
	list-style: none;
}
.ck-swiper {
	position: relative;
	overflow: hidden;
}
.ck-swiper-imglist {
	display: flex;
}
.ck-dot-wrapper {
	position: absolute;
	bottom: 30px;
	width: 100%;
	display: flex;
	justify-content: center;
	.ck-dot {
		width: 10px;
		height: 10px;
		margin: 0 5px;
		border-radius: 10px;
		background: #fff;
		transition: 0.2s;
		box-shadow: 0 0 3px 0 rgba(0,0,0,0.3);
	}
	.dot-active {
		width: 20px;
	}
}
</style>