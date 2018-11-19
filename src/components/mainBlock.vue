/* eslint-disable */
<!--hello world-->
<template>
	<div class="inputArea" :style="{height:inputHeight+'px',width:inputWidth+'px'}">
		<div class="notice" :class="{showNotice:noticeFlag}">
			{{notice}}
			<div class="close-notice" @click="closeNotice">X</div>
		</div>
		<div class="reloadBtn" @click="openWindow">
			重新选择
		</div>
		<div class="blankMask" @click="openWindow" v-if="loadFlag!=2">
			<img v-if="loadFlag==0" src="../assets/img.png" />
			<img v-if="loadFlag==1" src="../assets/loading.png" />
			<div class="text">{{loadFlag == 0?'点击浏览图片':'加载中'}}</div>
		</div>
		<div class="canvasArea" :class="{'moveHover':moveFlag}">
			<canvas id="inputAreaCanvas" @mousedown="setStartPoint" @mousemove="mouseMove" @mouseup="reset" @mouseleave="reset">	
			</canvas>
		</div>
		<input id="input" type="file" @change="loadImg" />
	</div>
</template>

<script>
	export default {
		props:{
			inputWidth:{
				type:Number,
				default:200
			},
			inputHeight:{
				type:Number,
				default:200
			},
			cuttingRatio:{
				type:Number,
				default:0
			}
		},
		data() {
			return {
				mouseDownFlag: false,//记录鼠标点击状态用标记
				loadFlag: 0,//记录图像家在状态用标记
				resultImgData: {},//被截取数据
				input: {},//输入框对象
				imgObj: new Image(),//图片对象
				inputAreaCanvas: {},//主体canvas对象
				inputArea2D: {},//主体CanvasRenderingContext2D对象
				notice: "拖拽鼠标框选所需要的区域",//提示区域文本
				noticeFlag: false,//提示区域展示状态标记
				dataURL:"",//被截取dataURL
				tempCanvas:{},//存放截取结果用canvas对象
				tempCanvas2D:{},//存放截取结果用CanvasRenderingContext2D对象
				resetX:0,//组件起点横坐标
				resetY:0,//组件起点纵坐标
				startX:0,//截取开始点横坐标
				startY:0,//截取开始点纵坐标
				resultX:0,//截取结束点横坐标
				resultY:0,//截取结束点纵坐标
				moveStartX:0,//拖拽起点横坐标
				moveStartY:0,//拖拽起点纵坐标
				moveX:0,//拖拽横向位移
				moveY:0,//拖拽纵向位移
				moveFlag:false,//拖拽标记
				rectHeight:0,//截取区域的高度
				rectWidth:0,//截取区域的宽度
				pointA:{
					x:0,
					y:0
				},//A点坐标
				pointB:{
					x:0,
					y:0
				}//B点坐标
			}
		},
		mounted: function() {
			//对象初始化
			this.input = document.getElementById('input')
			this.inputAreaCanvas = document.getElementById("inputAreaCanvas");
			this.inputArea2D = this.inputAreaCanvas.getContext("2d");
			this.tempCanvas = document.createElement('canvas');
			this.tempCanvas2D = this.tempCanvas.getContext('2d');
		},
		methods: {
			//打开文件选择窗口
			openWindow() {
				this.input.click();
			},
			//载入图片方法，当图片被选中后，input的value发生改变时触发
			loadImg() {
				let vm = this;
				//新建FileReader对象读取input中文件
				let reader = new FileReader();
				//每次载入后传给父组件的dataURL清空
				this.dataURL = '';
				//文件为空时返回
				if(this.input.files[0] == null) {
					return
				}
				//开始载入图片，并将数据通过dataURL的方式读取，展现载入层信息
				this.loadFlag = 1;
				reader.readAsDataURL(this.input.files[0]);
				//读取完成后将图像的dataURL数据赋给image对象的src的属性，使其加载图像
				reader.onload = function(e) {
					vm.imgObj.src = e.target.result;
				}
				//图像加载完成，利用drawImage将image对象渲染至canvas
				this.imgObj.onload = function() {
					vm.loadFlag = 2;
					vm.noticeFlag = true;
					 //计算载入图像的长宽比，决定图片显示方式
					let ratioHW = (vm.imgObj.height/vm.imgObj.width)
					//每张图片根据比例不同，总有一个方向占满显示区域
					if(ratioHW > 1) {
						vm.inputAreaCanvas.height = vm.inputHeight;
						vm.inputAreaCanvas.width = vm.inputHeight / ratioHW;
					} else {
						vm.inputAreaCanvas.width = vm.inputWidth;
						vm.inputAreaCanvas.height = vm.inputWidth * ratioHW;
					}
					//获取组件起点坐标
					vm.resetX = vm.inputAreaCanvas.getBoundingClientRect().left;
					vm.resetY = vm.inputAreaCanvas.getBoundingClientRect().top;
					//将获取的图像数据选在至canvas
					vm.inputArea2D.clearRect(0, 0, vm.inputAreaCanvas.width, vm.inputAreaCanvas.height);
					vm.inputArea2D.drawImage(vm.imgObj, 0, 0, vm.inputAreaCanvas.width, vm.inputAreaCanvas.height);
					vm.inputArea2D.fillStyle = 'rgba(0,0,0,0.5)'; //设定为半透明的黑色
					vm.inputArea2D.fillRect(0, 0, vm.inputWidth, vm.inputHeight); //填充黑色模版
				}
			},
			//获取截取范围起始坐标，当鼠标在canvas标签上点击时触发
			setStartPoint(e) {
				this.mouseDownFlag = true; //改变标记状态，置为点击状态
				//如果处于已选择区域内
				if(this.moveFlag){
					this.moveStartX = e.offsetX;
					this.moveStartY = e.offsetY;
					return;
				}
				this.startX = e.offsetX //获得起始点横坐标
				this.startY = e.offsetY //获得起始点纵坐标
			},
			//鼠标移动时触发
			mouseMove(e){
				this.moveFlag = false;
				//当属于被选中区域
				if(e.offsetX>this.pointA.x && e.offsetX<this.pointB.x && e.offsetY>this.pointA.y && e.offsetY<this.pointB.y){
					this.moveFlag = true
				}
				//当鼠标被拖动时触发（处于按下状态且移动）
				if(this.mouseDownFlag){
					this.drawArea(e)
				}
			},
			//当鼠标被拖动时触发
			drawArea(e) {
				if(!this.moveFlag){//截取动作
					//在canvas标签上范围的起始点横坐标
					this.resultX = e.offsetX
					//在canvas标签上范围的起始点纵坐标，根据比例参数决定
					if(this.cuttingRatio != 0) {
						//根据一定比例截取
						this.resultY = this.startY + parseInt((1 / this.cuttingRatio) * (this.resultX - this.startX))
					} else {
						//自由截取
						this.resultY = e.offsetY
					}
					this.rectWidth = Math.abs(this.resultX - this.startX);//获得截取区域宽度
					this.rectHeight = Math.abs(this.resultY - this.startY);//获得截取区域高度
					//为Point A 和 B 赋值
					this.pointA = {
						x:this.startX<this.resultX?this.startX:this.resultX,
						y:this.startY<this.resultY?this.startY:this.resultY
					};
					this.pointB = {
						x:this.startX>this.resultX?this.startX:this.resultX,
						y:this.startY>this.resultY?this.startY:this.resultY
					}
				}else{//拖动动作
					this.moveX = e.offsetX-this.moveStartX;//记录横向拖动位移
					this.moveY = e.offsetY-this.moveStartY;//记录纵向拖动位移

					//获得移动后的起始点坐标
					this.pointA.x += this.moveX;
					this.pointA.y += this.moveY;
					
					//对移动圈定范围
					if(this.pointA.x<0){
						this.pointA.x = 0;
					}else if((this.pointA.x+this.rectWidth)>this.inputAreaCanvas.width){
						this.pointA.x = this.inputAreaCanvas.width-this.rectWidth;
					}
					if(this.pointA.y<0){
						this.pointA.y = 0;
					}else if((this.pointA.y+this.rectHeight)>this.inputAreaCanvas.height){
						this.pointA.y = this.inputAreaCanvas.height-this.rectHeight;
					}
					//获得移动后的结束点坐标
					this.pointB.x = this.pointA.x + this.rectWidth;
					this.pointB.y = this.pointA.y + this.rectHeight;
					//记录下一次移动的开始坐标
					this.moveStartX = e.offsetX;
					this.moveStartY = e.offsetY;
				}
				
				//所选区域外阴影部分
				this.inputArea2D.clearRect(0, 0, this.inputWidth, this.inputHeight); //清空整个画面
				this.inputArea2D.drawImage(this.imgObj, 0, 0, this.inputAreaCanvas.width, this.inputAreaCanvas.height); //重新绘制图片
				this.inputArea2D.fillStyle = 'rgba(0,0,0,0.5)'; //设定为半透明的白色
				this.inputArea2D.fillRect(0, 0, this.pointB.x, this.pointA.y); //矩形A
				this.inputArea2D.fillRect(this.pointB.x, 0, this.inputWidth, this.pointB.y); //矩形B
				this.inputArea2D.fillRect(this.pointA.x, this.pointB.y, this.inputWidth - this.pointA.x, this.inputHeight - this.pointB.y); //矩形C
				this.inputArea2D.fillRect(0, this.pointA.y, this.pointA.x, this.inputHeight - this.pointA.y); //矩形D
				//当选择区域大于0时，将所选范围内的图像数据实时返回
				if(this.rectWidth != 0 && this.rectHeight != 0) {
					this.resultImgData = this.inputArea2D.getImageData(this.pointA.x, this.pointA.y, this.pointB.x - this.pointA.x, this.pointB.y - this.pointA.y);
					//canvas to DataURL
					this.tempCanvas.width = this.resultImgData.width;
					this.tempCanvas.height = this.resultImgData.height;
					this.tempCanvas2D.putImageData(this.resultImgData, 0, 0)
					this.dataURL = this.tempCanvas.toDataURL('image/jpeg', 1.0);
					//通过getImageDataUrl事件，实时返回所截取区域的图像数据
					this.$emit('getImageDataUrl', this.dataURL)
				}
			},
			//结束选择截取范围，返回所选范围的数据，重制鼠标状态，当鼠标点击结束时触发
			reset() {
				this.mouseDownFlag = false; //将标志置为已抬起状态
				if(this.dataURL){
					let blob = this.dataURLtoBlob(this.dataURL)
					this.$emit('getImageData', blob);//通过getImageData事件返回所截取区域blob对象数据
				}
			},
			//关闭提示信息
			closeNotice() {
				this.noticeFlag = false
			},
			//DataURL to Blob，兼容IE
			dataURLtoBlob(dataurl) {
				let arr = dataurl.split(',')
				let mime = arr[0].match(/:(.*?);/)[1]
				let bstr = atob(arr[1])
				let n = bstr.length
				let u8arr = new Uint8Array(n)
				while(n--) {
					u8arr[n] = bstr.charCodeAt(n)
				}
				return new Blob([u8arr], {
					type: mime
				});
			}
		}
	}
</script>

<style>	
	.inputArea {
		position: relative;
		background: #000;
	}
	
	.inputArea .blankMask {
		position: absolute;
		top: 0px;
		left: 0px;
		width: 100%;
		height: 100%;
		display: flex;
		color: gainsboro;
		
		background: #FFF;
		cursor: pointer;
		flex-direction: column;
		-ms-flex-direction: column;
		justify-content: center;
		-webkit-justify-content: center;
		align-items: center;
		-webkit-align-items: center;
		transition: all 0.5s;
		z-index: 2;
	}
	
	.inputArea .blankMask:hover {
		background: #F6F6F6;
	}
	
	.inputArea .blankMask .text {
		margin-top: 10px;
		font-size: 16px;
		font-weight: bold;
	}
	
	.inputArea .blankMask img {
		height: 40px;
		width: 40px;
	}
	
	.inputArea .reloadBtn {
		height: 20px;
		padding: 2px 5px 2px 5px;
		text-align: center;
		line-height: 20px;
		font-size: 12px;
		background: #FFFFFF;
		box-shadow: 0px 0px 5px rgba(0,0,0,0.3);
		border-radius: 2px;
		color: #2C3E50;
		position: absolute;
		top: 5px;
		right: 5px;
		cursor: pointer;
		transition: all 0.5s;
	}
	
	.inputArea .reloadBtn:hover {
		box-shadow: 0px 0px 8px rgba(0,0,0,0.5);
	}

	#input {
		display: none;
	}
	
	.inputArea .notice {
		height: 30px;
		line-height: 30px;
		text-align: center;
		background: #FFF;
		color: #2C3E50;
		font-size: 12px;
		text-align: center;
		position: absolute;
		width: 90%;
		margin-left: 5%;
		left: 0px;
		transition: all .5s;
		bottom: 0px;
		opacity: 0;
		box-shadow: 0px 0px 5px rgba(0,0,0,0.3);
		border-radius: 2px;
		-moz-user-select: none;
		-ms-user-select: none;
		-webkit-user-select: none;
	}
	
	.notice.showNotice {
		bottom: -40px;
		opacity: 1;
	}
	
	.inputArea .notice .close-notice {
		position: absolute;
		right: 10px;
		top: 0px;
		height: 30px;
		line-height: 30px;
		cursor: pointer;
	}
	
	.inputArea .canvasArea {
		display: flex;
		align-items: center;
		-webkit-align-items: center;
		justify-content: center;
		-webkit-justify-content: center;
		height: 100%;
		width: 100%;
	}
	
	.moveHover{
		cursor: move;
	}

</style>