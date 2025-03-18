<template>
	<view class="iliya-draggable-badge" :style="{width:size+'px'}" :animation="showAnimation" v-if="number > 0">
		<view class="badge" v-if="!showBoom" :style="{...badgeStyle}" @touchstart="badgeTouchstart"
			@touchmove="badgeTouchmove" @touchend="badgeTouchend">
			<view class="badge-number" :style="{fontSize:size * 0.8 + 'px'}">
				{{showNumber}}
			</view>
		</view>
		<view class="point" v-show="moveing" :style="{...pointStyle}">
			<view class="point-left" :style="{background:bgColor}"></view>
			<view class="point-right" :style="{background:bgColor}"></view>
		</view>
		<view class="line" v-show="moveing" :style="{...lineStyle}"></view>
		<view class="boom" v-if="showBoom" :style="{...boomStyle}">
			<view class="boom-dust"></view>
			<view class="boom-dust"></view>
			<view class="boom-dust"></view>
			<view class="boom-dust"></view>
			<view class="boom-dust"></view>
			<view class="boom-dust"></view>
		</view>
	</view>
</template>
<script>
	export default {
		props: {
			// 徽标大小，不建议小于14
			size: {
				type: Number,
				default: 20
			},
			// 背景色
			bgColor: {
				type: String,
				default: 'red'
			},
			// 数字
			number: {
				type: Number,
				default: 0
			},
			// 最大显示数
			max: {
				type: Number,
				default: 99
			},
			// 数字颜色
			color: {
				type: String,
				default: '#fff'
			},
			// 触发爆炸的距离
			maxDistance: {
				type: Number,
				default: 80
			}
		},
		data() {
			return {
				initX: 0,
				initY: 0,
				touchX: 0,
				touchY: 0,
				endX: 0,
				endY: 0,
				lineWidth: 0,
				lineRotate: 0,
				lineAndPointScale: 1,
				moveing: false,
				badgeTransform: 'translate()',
				showAnimation: null,
				transition: '',
				showBoom: false
			}
		},
		computed: {
			badgeWidth() {
				if (this.showNumber.length > 2) {
					return 'fit-content'
				}
				return this.size + 'px'
			},
			showNumber() {
				if (this.number > this.max) {
					return this.max + '+'
				}
				return this.number
			},
			badgeStyle() {
				return {
					width: this.badgeWidth,
					height: this.size + 'px',
					background: this.bgColor,
					color: this.color,
					transform: this.badgeTransform,
					transition: this.transition
				}
			},
			pointStyle() {
				return {
					width: this.size / 1.2 + 'px',
					height: this.size / 1.2 + 'px',
					transform: `translate(-50%,-50%) rotateZ(${this.lineRotate}deg) scale(${this.lineAndPointScale})`
				}
			},
			lineStyle() {
				return {
					background: this.bgColor,
					width: this.size / 1.2 + 'px',
					height: this.size / 1.2 + 'px',
					width: this.lineWidth + 'px',
					right: this.size / 2 + 'px',
					transform: `translateY(-50%) rotateZ(${this.lineRotate}deg) scaleY(${this.lineAndPointScale})`
				}
			},
			boomStyle() {
				return {
					width: this.size + 'px',
					height: this.size + 'px',
					transform: this.badgeTransform,
				}
			}
		},
		mounted() {
			const animation = uni.createAnimation({
				duration: 300,
				timingFunction: 'ease',
			})
			animation.scale(1).step()
			this.showAnimation = animation.export()

			const query = uni.createSelectorQuery().in(this);
			query
				.select(".badge")
				.boundingClientRect((data) => {
					this.initX = data.left
					this.initY = data.top
				})
				.exec();
		},
		methods: {
			badgeTouchstart(e) {
				// console.log('badgeTouchstart', e, this.initX, this.initY);
				this.touchX = e.changedTouches[0].pageX
				this.touchY = e.changedTouches[0].pageY
			},
			badgeTouchmove(e) {
				// console.log('badgeTouchmove', e);
				this.badgeTransform =
					`translate(${this.computeOffset(e.changedTouches[0].pageX,'X')}px,${this.computeOffset(e.changedTouches[0].pageY,'Y')}px) `
				// console.log(this.badgeTransform);
				this.moveing = true
				this.computeWidth(e.changedTouches[0].pageX, e.changedTouches[0].pageY)
				this.computeRotate(e.changedTouches[0].pageX, e.changedTouches[0].pageY)
				this.computeScale()
			},
			// 计算移动徽标偏移量
			computeOffset(number, axis) {
				return number - this['init' + axis] - (this['touch' + axis] - this['init' + axis])
			},
			// 计算连接杆长度
			computeWidth(moveX, moveY) {
				const dx = moveX - this.initX - (this.touchX - this.initX);
				const dy = moveY - this.initY - (this.touchY - this.initY);
				this.lineWidth = Math.sqrt(dx * dx + dy * dy);
			},
			// 计算连接杆旋转角度
			computeRotate(moveX, moveY) {
				const dx = this.initX - moveX + (this.touchX - this.initX);
				const dy = this.initY - moveY + (this.touchY - this.initY);
				const angleInRadians = Math.atan2(dy, dx);
				this.lineRotate = angleInRadians * (180 / Math.PI);
			},
			computeScale() {
				const scale = (this.maxDistance - this.lineWidth) / this.maxDistance
				// console.log('computeScale',scale);
				if (scale > 1) {
					this.lineAndPointScale = 1
				} else if (scale < 0.1) {
					this.lineAndPointScale = 0
				} else {
					this.lineAndPointScale = scale
				}
			},
			badgeTouchend(e) {
				// console.log('badgeTouchend', e);
				this.endX = e.changedTouches[0].pageX
				this.endY = e.changedTouches[0].pageY
				this.moveing = false
				if (this.lineAndPointScale === 0) {
					this.showBoom = true
					setTimeout(() => {
						this.$emit('badgeBoom')
					}, 200)
				} else {
					this.restore()
				}
			},
			restore() {
				let shakeX = this.initX - this.endX + (this.touchX - this.initX)
				let shakeY = this.initY - this.endY + (this.touchY - this.initY)
				if (shakeX > 20) shakeX = 20
				if (shakeY > 20) shakeY = 20
				if (shakeX < -20) shakeX = -20
				if (shakeY < -20) shakeY = -20
				this.badgeTransform = `translate(${shakeX}px,${shakeY}px)`
				this.transition = 'transform .1s linear'
				setTimeout(() => {
					this.badgeTransform = `translate(0px,0px)`
					setTimeout(() => {
						this.transition = ''
					}, 100)
				}, 100)
			}
		}
	}
</script>
<style scoped lang="scss">
	.iliya-draggable-badge {
		position: relative;
		transform: scale(0);

		.badge {
			display: flex;
			align-items: center;
			justify-content: center;
			border-radius: 999px;
			z-index: 1;
			position: relative;
			animation: showBadge .3s ease;

			&-number {
				width: 100%;
				height: 100%;
				transform: scale(0.7);
				text-align: center;
			}
		}

		.point {
			border-radius: 50%;
			position: absolute;
			top: 50%;
			left: 50%;
			transform: translate(-50%, -50%);
			overflow: hidden;

			&-left {
				width: 50%;
				height: 100%;
				position: absolute;
				top: 0;
				left: 0;
				border-radius: 50% 0 0 50%;
				opacity: 0.8;
			}

			&-right {
				width: 50%;
				height: 100%;
				position: absolute;
				top: 0;
				right: 0;
				border-radius: 0 50% 50% 0;
				opacity: 0.4;
			}
		}

		.line {
			position: absolute;
			border-radius: 999rpx 0 0 999rpx;
			top: 50%;
			transform-origin: right;
			opacity: 0.4;
			backdrop-filter: blur(10px);
		}

		.boom {
			position: absolute;

			.boom-dust {
				background-color: rgba(0, 0, 0, 0.1);
				border-radius: 50%;
				position: absolute;

				&:first-of-type {
					width: 60%;
					height: 60%;
					top: 50%;
					left: 50%;
					transform: translate(-50%, -50%);
				}

				&:nth-child(2) {
					width: 40%;
					height: 30%;
					top: -10%;
					left: 0%;
					transform: rotate(140deg);
					animation: boom .1s ease;
				}

				&:nth-child(3) {
					width: 40%;
					height: 30%;
					top: -10%;
					right: 0%;
					transform: rotate(40deg);
					animation: boom .1s ease;
				}

				&:nth-child(4) {
					width: 40%;
					height: 30%;
					bottom: 20%;
					left: -20%;
					transform: rotate(80deg);
					animation: boom .1s ease;
				}

				&:nth-child(5) {
					width: 40%;
					height: 30%;
					bottom: 20%;
					right: -20%;
					transform: rotate(120deg);
					animation: boom .1s ease;
				}

				&:nth-child(6) {
					width: 40%;
					height: 30%;
					bottom: -20%;
					left: 30%;
					transform: rotate(0deg);
					animation: boom .1s ease;
				}

				@keyframes boom {
					from {
						top: 40%;
						left: 40%;
					}
				}
			}
		}
	}
</style>