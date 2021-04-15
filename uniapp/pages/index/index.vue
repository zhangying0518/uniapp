<template>
	<view class="wrapper">
		<view @click="btnStart" class="btnStart">{{btnStartValue}}</view>
		<view class="score">
			<span style="margin-right: 30rpx;">分数：{{scoreSpan * 10}}</span> <span>最高分：{{maxSpan * 10}}</span>
		</view>
		<canvas canvas-id="canvas" id="canvas"></canvas>

		<view class="gameover" :class="gameover?'gameoverShow':'gameoverHide'">Game Over!</view>

		<!-- 按钮区域 -->
		<view class="direction_content">
			<view class="direction_view top_direction" @click="arrowUp">
				<img src="http://47.92.83.204:8080/zhangying/img/snake/up.png" />
			</view>
			<view class="direction_view left_direction" @click="arrowLeft">
				<img src="http://47.92.83.204:8080/zhangying/img/snake/left.png" />
			</view>
			<view class="direction_view bottom_direction" @click="arrowDown">
				<img src="http://47.92.83.204:8080/zhangying/img/snake/111.png" />
			</view>
			<view class="direction_view right_direction" @click="arrowRight">
				<img src="http://47.92.83.204:8080/zhangying/img/snake/right.png" />
			</view>
		</view>
		<view class="desc">
			游戏说明:蛇头碰到四周边缘或舌头与舍身相接触都会使游戏结束！！！
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				coverImg: 'http://47.92.83.204:8080/zhangying/img/snake/fengmian.png',
				maxSpan: 0,//最高分
				ctx: "",//上下文
				btnStartValue: "开始",
				scoreSpan: 0,//当前分数
				
				// 食物的默认位置
				x: 0,
				y: 0,

				// // 蛇的默认位置
				snake: [{
					x: 3,
					y: 0
				}, {
					x: 2,
					y: 0
				}, {
					x: 1,
					y: 0
				}],

				
				isEated: false,//食物是否被吃掉
				
				gameover: false, //游戏是否结束

			
				isGameOver: true,// 游戏是否结束

				// 蛇的方向
				directionX: 1,
				directionY: 0,

				// // 贪吃蛇当前方向
				curDir: "",

				// // 是否按过反向
				isKeyDown: false,
			}
		},
		onReady: function(e) {
			var ctx = uni.createCanvasContext('canvas')
			var maxScore = uni.getStorageSync("maxScore")

			this.ctx = ctx
			this.maxSpan = maxScore || 0

			this.renderWelcome()

			this.timer()
			this.keydownFun()
		},
		methods: {
			keydownFun(){//键盘上下键
				document.addEventListener('keydown', (e)=> { //监听键盘按下行为
					if (this.isKeyDown) return
					this.isKeyDown = true
					if (this.curDir === 'right' && e.keyCode === 37) { //左
						return
					}
					if (this.curDir === 'down' && e.keyCode === 38) { //上
						return
					}
					if (this.curDir === 'left' && e.keyCode === 39) { //右
						return
					}
					if (this.curDir === 'up' && e.keyCode === 40) { //下
						return
					}
					switch (e.keyCode) {
						case 38:
							this.directionX = 0
							this.directionY = -1
							this.curDir = 'up'
							break
						case 40:
							this.directionX = 0
							this.directionY = 1
							this.curDir = 'down'
							break
						case 37:
							this.directionX = -1
							this.directionY = 0
							this.curDir = 'left'
							break
						case 39:
							this.directionX = 1
							this.directionY = 0
							this.curDir = 'right'
							break
					}
				})
			},
			timer() {// 让贪吃蛇动起来，原理就是不断擦除，然后不断重绘
				setInterval(() => {
					if (this.isGameOver) { //游戏结束
						return
					}
					// 擦除
					this.ctx.clearRect(0, 0, 300, 300)
					var oldHead = this.snake[0]
					var newHead = {
						x: oldHead.x + this.directionX,
						y: oldHead.y + this.directionY
					}
					if (
						newHead.y < 0 ||
						newHead.x < 0 ||
						newHead.x * 30 >= 300 ||
						newHead.y * 30 >= 300 ||
						this.snake.some(function(item) {
							return item.x === newHead.x && item.y === newHead.y
						})
					) { //游戏结束的判断
						this.isGameOver = true
						this.gameover = true
						uni.setStorageSync('maxScore', this.maxSpan)
					} else {
						this.snake.unshift(newHead)
						/* 蛇吃食物的分析：当蛇头的坐标和食物的坐标重合的时候，就表示食物被吃掉了，此时应该做两件事
						    1.让isEated变为true,表示食物被吃掉
						    2.让蛇增加一节长度
						*/
						// 蛇动起来分析：删除最后一节，然后在最前面添加一个新节，通过定时器不断的执行这个操作就有一个蛇动起来的效果
						if (this.snake[0].x * 30 === this.x && this.snake[0].y * 30 === this
							.y) { //食物被吃掉，重新生成一个食物，并在蛇头添加一个新节
							this.isEated = true
							this.scoreSpan++
							if (this.scoreSpan > this.maxSpan) {
								this.maxSpan = this.scoreSpan
							}
						} else { //食物没有被吃掉,移除最后一节，在蛇头添加一个新节
							this.isEated = false
							this.snake.pop()
						}
						this.renderSnake()
						this.renderFood()
						this.renderMap()
						this.isKeyDown = false
					}
				}, 1000 / 3)
			},
			arrowUp() {//上箭头
				if (this.curDir == 'down') {
					return
				}
				this.directionX = 0
				this.directionY = -1
				this.curDir = 'up'

			},
			arrowLeft() {//左箭头
				if (this.curDir == 'right') {
					return
				}
				this.directionX = -1
				this.directionY = 0
				this.curDir = 'left'
			},
			arrowDown() {//下箭头
				if (this.curDir == 'up') {
					return
				}
				this.directionX = 0
				this.directionY = 1
				this.curDir = 'down'
			},
			arrowRight() {//右箭头
				if (this.curDir == 'left') {
					return
				}
				this.directionX = 1
				this.directionY = 0
				this.curDir = 'right'
			},
			btnStart() {//点击重新开始
				this.btnStartValue = '重新开始'
				
				this.gameover = false
				
				this.isEated = false
				
				this.x = Math.floor(Math.random() * 10) * 30
				this.y = Math.floor(Math.random() * 10) * 30
				
				this.directionX = 1
				this.directionY = 0
				
				this.snake = [{
						x: 3,
						y: 0
					}, {
						x: 2,
						y: 0
					}, {
						x: 1,
						y: 0
					}],
				
				this.isGameOver = false
				
				this.curDir = 'right'
				this.isKeyDown = false
				this.scoreSpan = '0'
			},

			renderWelcome() { //封面
				this.ctx.drawImage(this.coverImg, 0, 0, 300, 300)
				this.ctx.draw()
			},

			renderMap() {//绘制网格
				for (var i = 1; i < 10; i++) { 
					// 水平线
					var temp = i * 30 + 0.5
					this.ctx.moveTo(0, temp)
					this.ctx.lineTo(300, temp)

					// 垂直线
					this.ctx.moveTo(temp, 0)
					this.ctx.lineTo(temp, 300)
				}
				this.ctx.strokeStyle = 'white'
				this.ctx.stroke()
				this.ctx.draw()
			},
			renderFood() {
				// 随机生成食物的位置
				if (this.isEated) { //食物被吃掉后就重新绘制食物
					this.x = Math.floor(Math.random() * 10) * 30
					this.y = Math.floor(Math.random() * 10) * 30
				}
				// 绘制食物
				this.ctx.fillStyle = '#cccc00'
				this.ctx.fillRect(this.x, this.y, 30, 30)
				this.ctx.stroke()
			},
			renderSnake() { // 绘制蛇
				for (var i = 0; i < this.snake.length; i++) {
					if (i === 0) { //舌头
						this.ctx.fillStyle = '#ff0033'
					} else { //蛇身
						this.ctx.fillStyle = '#333399'
					}
					this.ctx.fillRect(this.snake[i].x * 30, this.snake[i].y * 30, 30,
						30)
				}
			}
		}
	}
</script>

<style scoped>
	.wrapper {
		position: relative;
		width: 750rpx;
		margin: 0 auto;
		border: 1px solid #fff;
		box-sizing: border-box;
	}

	.btnStart {
		width: 40%;
		margin: 60rpx 20rpx;
		color: #fff;
		text-align: center;
		padding: 10rpx;
		font-weight: bold;
		border-radius: 20rpx;
		font-size: 40rpx;
		background: #33cc99;
	}

	canvas {
		width: 300px;
		height: 300px;
		display: block;
		margin: 0 auto;
		box-sizing: border-box;
		background-color: #33cc99;
		position: absolute;
		top: 170rpx;
		left: 50%;
		transform: translateX(-50%);
	}

	.gameoverShow {
		display: block !important;
	}

	.gameoverHide {
		display: none !important;
	}

	.gameover {
		position: absolute;
		top: 170rpx;
		bottom: 0;
		right: 0;
		width: 100%;
		line-height: 300px;
		height: 300px;
		text-align: center;
		color: #fff;
		font-size: 80rpx;
		background-color: rgba(0, 0, 0, 0.6);
	}

	.score {
		position: absolute;
		right: 30rpx;
		top: 0;
		font-weight: bold;
	}

	.direction_content {
		width: 100%;
		height: 150rpx;
		position: absolute;
		top: 410px;
		left: 50%;
		transform: translateX(-50%);
	}

	.direction_view {
		width: 80rpx;
		height: 80rpx;
		position: absolute;
		padding: 10rpx;
	}

	.direction_view img {
		width: 100%;
		height: 100%;
	}

	.left_direction {
		top: 60%;
		left: 20%;
	}

	.bottom_direction {
		top: 140%;
		left: 45%;
	}

	.right_direction {
		top: 60%;
		left: 70%;
	}

	.top_direction {
		top: 0;
		left: 45%;
	}
	.desc{
		position: fixed;
		bottom: 20rpx;
		left: 20rpx;
		color: #33cc99;
	}
</style>
