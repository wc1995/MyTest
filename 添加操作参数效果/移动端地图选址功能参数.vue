<template>
	<div class="main" ref="main">
		<div class="outer" :style="'width:'+width+'px;height:'+width/rate+'px;left:'+width/rate+'px;transform:rotateZ(90deg);'" @dblclick.stop="openSelected($event)">
			<img :src="'./static/images/lc1.png'" :style="'width:'+(width-cha)+'px;height:'+(width-cha)/rate+'px;left:'+cha/2+'px'"/>
			<div v-if="bClicked" class="point" :style="'left:'+bX+'px;top:'+bY+'px;'"></div>
			<div class="operation" :style="'width:'+width+'px;height:'+(height-width/rate)+'px;left:0px;top:-'+(height-width/rate)+'px;'">
				<div class="btn" :style="'line-height:'+(height-width/rate)+'px'" @tap.stop="toAllBack()">返回</div>
				<div class="mui-input-row mui-input-range">
					<label>放大倍数</label>
					<input type="range" min="1" max="4" v-model = "maxsize">
					<div>{{maxsize}}倍</div>
				</div>
				<div class="btn" :style="'line-height:'+(height-width/rate)+'px'" @tap.stop="toAllConfirm()">确定</div>
			</div>
		</div>

		<div v-if="isSeleted" class="inner" :style="'width:'+width+'px;height:'+width/rate+'px;position:absolute;transform-origin:0px 0px;left:'+width/rate+'px;transform:rotateZ(90deg);'" @touchstart = "selectPoint($event)" @touchmove = "movePoint($event)">
			<img :src="'./static/images/lc1.png'" :style="'width:'+(width-cha)+'px;height:'+(width-cha)/rate+'px;transform-origin:0 0;transform:scale('+size+');position:relative;left:'+dx+'px;top:'+dy+'px'"/>
			<div v-if="lClicked" class="point" :style="'left:'+lX+'px;top:'+lY+'px;'"></div>
			<div class="operation" :style="'width:'+width+'px;height:'+(height-width/rate)+'px;left:0px;top:-'+(height-width/rate)+'px;'">
				<div class="btn" :style="'line-height:'+(height-width/rate)+'px'" @tap.stop="toBack()">返回</div>
				<div class="btn" :style="'line-height:'+(height-width/rate)+'px'" @tap.stop="toConfirm()">确定</div>
			</div>
		</div>
		<!-- <span style="position:absolute;top:0;bottom:0;left:0;right:0;margin:auto;color:#fff;line-height:100%;font-size:30px;height:30px;">{{show}}</span> -->
	</div>
</template>

<script>
var oldPath = "";

export default {
	name: 'HazardSourceSelectAddress',
	data () {
		return {
			msg: '地图选址',
			width:0,
			height:0,
			distance:0,
			rate:1,
			x:0,
			y:0,
			dx:0,
			dy:0,
			cx:0,
			cy:0,
			show:"测试区域",
			size:1,
			isSeleted:false,
			// size:4,
			// isSeleted:true,
			maxsize:4,
			cha:30,
			// cha:50,
			lClicked:false,
			lX:0,
			lY:0,
			bClicked:false,
			bX:0,
			bY:0,
			r:8,
		}
	},
	methods:{
		openSelected:function(e){
			this.isSeleted = true;
			this.size = this.maxsize;
			var x = e.clientX - this.cha/this.rate;
			var y = e.clientY - this.cha/2;
			this.x = x;
			this.y = y;
			var ox = ((this.width-this.cha)/this.rate-x)*this.size;
			var oy = y*this.size;
			var ex = (this.width/this.rate)/2;
			var ey = (this.width-this.cha)/2;
			this.dx = -(oy - ey);
			this.dy = -(ox - ex);
			// console.log(this.dx);
			// console.log(this.dy);
		},
		selectPoint:function(e){
			var x = e.changedTouches[0].clientY - this.r;
			var y = this.width/this.rate - e.changedTouches[0].clientX - this.r;
			if(y>=0){
				this.lClicked = true;
				this.cx = x- this.cha;
				this.cy = y;
				this.lX = x;
				this.lY = y;
			}
		},
		movePoint:function(e){
			var x = e.changedTouches[0].clientY - this.r;
			var y = this.width/this.rate - e.changedTouches[0].clientX - this.r;
			if(y>=0){
				this.cx = x - this.cha;
				this.cy = y;
				this.lX = x;
				this.lY = y;
			}
		},
		toBack(){
			this.isSeleted = false;
			this.size = 1;
			this.lX = 0;
			this.lY = 0;
			this.lClicked = false;
		},
		toConfirm(){
			var moveX = this.cx+this.r+this.cha-this.width/2;
			var moveY = this.cy+this.r-this.width/this.rate/2;
			this.bClicked = true;
			this.bX = this.y + this.cha/2 - this.r + moveX/this.size;
			this.bY = this.width/this.rate - this.x - this.cha + this.r + moveY/this.size;
			this.isSeleted = false;
			this.lClicked = false;
			this.size = 1;
			this.lX = 0;
			this.lY = 0;
		},
		toAllBack(){
			this.$router.back(-1);
		},
		toAllConfirm(){
			var vm = this;
			var x = this.bX - this.cha/2 + this.r;
			var y = (this.bY+this.r)<0?0:(this.bY+this.r);
			var rate = 1500/(this.width-this.cha);
			// console.log((x*rate).toFixed(2));
			// console.log((y*rate).toFixed(2));
			var query = {};
			for(var item in this.$route.query){
				query[item] = this.$route.query[item];
			}
			query.x = (x*rate).toFixed(2);
			query.y = (y*rate).toFixed(2);
			query.flag = false;
			this.$router.replace({
				path: oldPath,
				query: query,
			});
		}
	},
	mounted(){
		var vm = this;

		vm.width = vm.$refs.main.clientHeight;
		vm.height = vm.$refs.main.clientWidth;

		var image = new Image();
		image.src = './static/images/lc1.png';
		image.onload = function(){
			vm.rate = this.width/this.height;
		}
		
		if(this.$route.query.x >=0 && this.$route.query.y >= 0){
			this.bClicked = true;
			var rate = 1500/(this.width-this.cha);
			var x = this.$route.query.x;
			var y = this.$route.query.y;
			this.bX = x/rate + this.cha/2 - this.r;
			this.bY = y/rate - this.r;
		}
		// 171,66
		// this.bClicked = true;
		// var rate = 1500/(this.width-this.cha);
		// var x = 171;
		// var y = 66;
		// this.bX = x/rate + this.cha/2 - this.r;
		// this.bY = y/rate - this.r;
	},
	beforeRouteEnter (to, from, next) {
		oldPath = from.path+"";
		next();
	}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
	.main{
		width:100%;
		height:100%;
		position:relative;
		overflow:hidden;
		background:#333;
	}
	.outer{
		position:absolute;
		transform-origin:0px 0px;
		transform:rotateZ(90deg);
		z-index:333;
		text-align:center;
	}
	.inner{
		position:absolute;
		transform-origin:0px 0px;
		z-index:666;
		background:#333;
	}
	.main img{
		position:absolute;
		left:0;
		top:0;
	}
	.point{
		width:16px;
		height:16px;
		border-radius:100%;
		background-color:red;
		position:absolute;
		left:0;
		top:0;
	}
	.operation{
		position:absolute;
		z-index:888;
		display:flex;
		color:#fff;
		background:#333;
		border-bottom:1px solid #000;
	}
	.operation .btn{
		flex:1;
		
	}
	.operation .btn:not(:first-child){
		border-left:1px solid #000;
	}
	.mui-input-range{
		width:50%;
		display:flex;
		align-items:center;
		border-left:1px solid #000;
	}
	.mui-input-range label{
		width:.6rem;
	}
	.mui-input-range label ~ input[type='range'] {
		width:50%;
	}
</style>
