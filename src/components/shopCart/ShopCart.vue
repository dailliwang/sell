<template>
	<div>
		<div class="shopcart">
		<div class="content" @click="toggleList">
			<div class="content-left">
				<div class="logo-wrapper">
					<div class="logo" :class="{logoHighLight: totalCount>0}"> 
						<span class="icon-shopping_cart" :class="{'icon-shopping_cart_font': totalCount>0}"></span>
					</div>
					<div class="num" v-show="totalCount>0">
						{{totalCount}}
					</div>
				</div>
				<div class="price" :class="{pricehighLight: totalPrice>0}">￥{{totalPrice}}</div>
				<div class="desc">另需配送费￥{{deliveryprice}}元</div>
			</div>
			<div class="content-right" @click.stop.prevent="pay">
				<div class="pay" :class="{enough: payDesc==='去结算'}">
					{{payDesc}}
				</div>
			</div>
		</div>
		<!-- 5个小球 -->
		<div class="ball-cotainer">
			<transition name="drop"
				v-on:beforeEnter="beforeEnter"
				v-on:enter="enter"
				v-on:after-enter="afterEnter"
				v-for="ball in balls" >
				<div class="ball"
					v-show="ball.show">
					<div class="inner inner-hook"></div>
				</div>
			</transition>
		</div>
		<transition name="fold">
			<div class="shopcart-list" 
				v-show="listShow">
				<div class="list-header">
					<h1 class="title">购物车</h1>
					<span class="empty" @click="empty">清空</span>
				</div>
				<div class="list-content" ref="shopwrapper">
					<ul>
						<li class="food" v-for="food in selectFoods">
							<span class="name">{{food.name}}</span>
							<div class="price">
								<span>￥{{food.price * food.count}}</span>
							</div>
							<div class="cartcontrol-wrapper">
								<cart-control :food="food"></cart-control>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</transition>
	</div>
	<transition name="fade">
		<div class="list-mask" v-show="listShow" @click="hideList"></div>
	</transition>
	</div>
</template>

<script>
	import CartControl from '../cartControl/CartControl'
	import BScroll from 'better-scroll'
	export default{
		components: {
			CartControl
		},
		data() {
			return {
				balls: [
					{show: false},
					{show: false},
					{show: false},
					{show: false},
					{show: false}
				],
				dropballs: [],
				fold: true
			}
		},
		props: {
			deliveryprice: {
				type: Number,
				required: true
			},
			minprice: {
				type: Number,
				required: true
			},
			selectFoods: {
				type: Array,
				default(){
					return []
				},
				required: true
			}
		},
		computed: {
			totalPrice() {
				let total = 0;
				this.selectFoods.forEach((food)=>{
					total += food.price * food.count;
				});
				return total;
			},
			totalCount() {
				let count = 0;
				this.selectFoods.forEach(function(food){
					count += food.count;
				});
				return count;
			},
			payDesc() {
				if(this.totalPrice===0){
					return `￥${this.minprice}元起送`;
				}else if(this.totalPrice<this.minprice){
					return `还差￥${this.minprice-this.totalPrice}元起送`;
				}else{
					return '去结算';
				}
			},
			listShow() {
				if(!this.totalCount){
					this.fold = true;
					return false;
				}
				let show = !this.fold;
				if(show){
					this.$nextTick(()=>{
						if(!this.scroll){
							this.scroll = new BScroll(this.$refs.shopwrapper, {
								click: true
							});
						}else{
							this.scroll.refresh();
						}
						
					});
					
				}
				return show;
			}
		},
		methods: {
			pay(){
				if(this.totalPrice >= this.minprice)
					window.alert('支付'+(this.totalPrice+this.deliveryprice)+'元');
			},
			drop(el){
				for(let i=0;i< this.balls.length;i++){
					let ball = this.balls[i];
					if(!ball.show){
						ball.show = true;
						ball.el = el;
						this.dropballs.push(ball);
						return;
					}
				}
			},
			beforeEnter(el){
				let count = this.balls.length;
				while(count--){
					let ball = this.balls[count];
					if(ball.show){
						let rect = ball.el.getBoundingClientRect();
						let x = rect.left - 32;
						let y = -(window.innerHeight - rect.top - 22);
						el.style.display = '';
						el.style.webkitTransform = `translate3d(0, ${y}px, 0)`;
						el.style.transform = `translate3d(0, ${y}px, 0)`;
						let inner = el.getElementsByClassName('inner-hook')[0];
						inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`;
						inner.style.transform = `translate3d(${x}px, 0, 0)`;
					}
				}
			},
			enter(el){
				let rf = el.offsetHeight;
				this.$nextTick(()=>{
					el.style.webkitTransform = 'translate3d(0, 0, 0)';
					el.style.transform = 'translate3d(0, 0, 0)';
					let inner = el.getElementsByClassName('inner-hook')[0];
					inner.style.webkitTransform = 'translate3d(0, 0, 0)';
					inner.style.transform = 'translate3d(0, 0, 0)';
				});
			},
			afterEnter(el){
				let ball = this.dropballs.shift();
				if(ball){
					ball.show = false;
					el.style.display = 'none';
				}
			},
			toggleList() {
				if(!this.totalCount){
					return;
				}
				this.fold = !this.fold;
			},
			empty(){
				this.selectFoods.forEach((food)=>{
					if(food.count){
						food.count = 0;
					}
				});
			},
			hideList(){
				this.fold = true;
			}
		}
	}
</script>

<style scoped>
	.shopcart {
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 50;
		width: 100%;
		height: 48px;
	}
	.content {
		display: flex;
		background: #141d27;
		font-size: 0;
	}
	.content-left {
		flex: 1;
		color: rgba(255,255,255,0.4);
	}
	.logo-wrapper {
		display: inline-block;
		vertical-align: top;
		position: relative;
		top: -10px;
		margin: 0 12px;
		padding: 6px;
		width: 56px;
		height: 56px;
		box-sizing: border-box;
		border-radius: 50%;
		background: #141d27;
	}
	.num {
		position: absolute;
		top: 0;
		right: 0;
		width: 24px;
		height: 16px;
		line-height: 16px;
		text-align: center;
		border-radius: 16px;
		font-size: 9px;
		font-weight: 700;
		color: #fff;
		background: rgb(240, 20, 20);
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4);
	}
	.logo {
		width: 100%;
		height: 100%;
		border-radius: 50%;
		background: #2b343c;
		text-align: center;
	}
	.logoHighLight {
		background: rgb(0, 160, 220);
	}
	.icon-shopping_cart {
		line-height: 44px;
		font-size: 24px;
		color: #80858a;
	}
	.icon-shopping_cart_font {
		color: #fff;
	}
	.price {
		display: inline-block;
		vertical-align: top;
		line-height: 24px;
		margin-top: 12px;
		box-sizing: border-box;
		padding-right: 12px;
		border-right: 1px solid rgba(255,255,255,0.1);
		font-size: 16px;
		font-weight: 700;
	}
	.pricehighLight {
		color: #fff;
	}
	.desc {
		display: inline-block;
		vertical-align: top;
		line-height: 24px;
		margin: 12px 0 0 12px;
		font-size: 10px;
	}
	.content-right {
		flex: 0 0 105px;
		width: 105px;
		color: rgba(255,255,255,0.4);
		background: #2b333b;
	}
	.pay {
		height: 48px;
		line-height: 48px;
		text-align: center;
		font-size: 12px;
		font-weight: 700;
		background: #2b333b;
	}
	.enough {
		background: #00b43c;
		color: #fff;
	}
	.ball-cotainer .ball{
		position: fixed;
		bottom: 22px;
		left:32px;
		z-index: 200;
	}
	.drop-enter-active {
	  transition: all 0.6s cubic-bezier(0.23,-0.36,0.93,0.71);
	}
	.inner {
		width: 16px;
		height: 16px;
		background: rgb(0, 160, 220);
		border-radius: 50%;
		transition: all 0.6s linear;
	}
	.shopcart-list {
	  position: absolute;
	  top: -400%;
	  left: 0;
	  z-index: -1;
	  width: 100%;
	  max-height: 400px;
	}
	.fold-enter-active {
	  transition: all 0.5s;
	}
	.fold-enter {
	  	transform: translate3d(0, -100%, 0);
	}
	.fold-leave {
		transform: translate3d(0, -100%, 0);
	}
	.fold-leave-to{
	 transform: translate3d(0, 0, 0);
	}
	.list-header {
		height: 40px;
		line-height: 40px;
		padding: 0 18px;
		background: #f3f5f7;
		border-bottom: 1px solid rgba(7, 17, 27, 0.1) ;
	}
	.title {
		float: left;
		font-size: 14px;
		color: rgb(7,17,27);
	}
	.empty {
		float: right;
		font-size: 12px;
		color: rgb(0,160,220);
	}
	.list-content {
		position: relative;
		background: #fff;
		padding: 0 18px;
	}
	.food {
		position: relative;
		padding: 12px 0;
		box-sizing: border-box;
		border-bottom: 1px solid rgba(7,17,27,0.1);
	}
	.name {
		line-height: 24px;
		font-size: 14px;
		color: rgb(7, 17, 27);
	}
	.list-content .price {
		position: absolute;
		right: 90px;
		bottom: 12px;
		line-height: 24px;
		font-size: 14px;
		font-weight: 700;
		color: red;
	}
	.list-content .cartcontrol-wrapper {
		position: absolute;
		right: 0;
		bottom: 6px;
	}
	.list-mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 40;
		background: rgba(7, 17, 27, 0.6);
		backdrop-filter: blur(20px);
	}
	.fade-enter-active, .fade-leave-active {
	  transition: opacity .5s
	}
	.fade-enter, .fade-leave-to {
	  opacity: 0
	}
</style>