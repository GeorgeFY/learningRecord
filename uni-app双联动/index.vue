<template>
	<view class="content">
		<view>我是首页</view>
		<view class="link">
			<view class="left">
				<view 
					v-for="(item,index) in order"
					:key="index"
					@click="setId(index)"
					:class="{active:curentNum === index}"
					>
					{{item.title}}
				</view>
			</view>
			<view class="right">
				<scroll-view 
					scroll-y="true"
					style="border: 1px solid #007AFF; height: 200px;"
					:scroll-into-view="clickId"
					scroll-with-animation
					@scroll="scroll"
					@scrolltolower="scrolltolower"
					>
					<view v-for="(item,index) in order" :key="index">
						<view class="rtitle" :id="'po' + index">{{item.title}}</view>
						<view v-for="(it,idx) in item.list" :key="idx">
							{{it}}
						</view>
					</view>
				</scroll-view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				order:[
					{title:"推荐",list:["鸡爪","鸡蛋炒米粉","烤鱼","豆腐","鸡蛋火腿炒饭","土豆","鸡爪","鸡蛋炒米粉","烤鱼","豆腐","鸡蛋火腿炒饭","土豆"]},
					{title:"肉类",list:["烤鸡脖子","烤蟹柳","烤鸡肉","牛肚","牛肉","烤鸭肠","掌中宝"]},
					{title:"海鲜",list:["鱿鱼须","金丝鱼","田螺","芥末鱿鱼","青口贝","小黄鱼","烤鱿鱼","基围虾"]},
					{title:"蔬菜",list:["烤大蒜","豆腐","烤年糕","奶香馒头","面筋","金针菇","黄瓜"]},
					{title:"主食",list:["炒饭","鸡蛋火腿肠炒饭","鸡蛋肉丝炒饭","鸡蛋肉丝炒面","煎饺","鸡蛋炒粉","鸡蛋炒面"]}
					],
					clickId:"",
					curentNum:0,
					topList:[]
			}
		},
		onLoad() {
			this.getNodeInfo();
		},
		methods: {
			setId(index){
				this.clickId = "po" + index;
				this.curentNum = index;
			},
			scroll(e){
				console.log(e.detail.scrollTop)
				let scrollTop = e.detail.scrollTop;
				for (let i = 0; i < this.topList.length; i++) {
					let h1 = this.topList[i];
					let h2 = this.topList[i+1];
					
					if(scrollTop >= h1 && scrollTop < h2){
						this.curentNum = i;
					}
				}
			},
			getNodeInfo(){
				const query = uni.createSelectorQuery().in(this);
				query.selectAll('.rtitle').boundingClientRect(data => {
				  console.log(data);
				  let rel = [];
				  data.map(item=>{
					  rel.push(item.top - data[0].top);
				  })
				  console.log(rel)
				  this.topList = rel
				}).exec();
			},
			scrolltolower(){
				setTimeout(()=>{
					this.curentNum = this.topList.length-1;
				},100)
			}
		}
	}
</script>

<style lang="scss">
	.link{
		display: flex;
		.left{
			width: 100px;
			border: 1px solid red;
		}
		.right{
			flex: 1;
			border: 1px solid blue;
		}
		.rtitle{
			background-color: red;
		}
		.active{
			background-color: pink;
		}
	}
</style>
