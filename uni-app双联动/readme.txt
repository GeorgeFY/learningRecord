uni-app 二级联动功能实现

左边为标题，右边为内容，点击左边右边可以跳转到对应内容。右边滑动是左边标题也可以跟着显示

页面布局
	弹性盒子布局
	左边给定一个100px宽度，右边flex=1自适应
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
	}
	
数据
	在data里面有一个order的数据，数组里面元素为一个对象，每个对象里面包括title，list数组
	
	order:[
			{title:"xxx",list:["aaa","bbb","ccc"]},
			{title:"yyy",list:["111","222","222"]},
			...
	]
	
有了数据后，在页面进行渲染 （v-for）

结构为
<view class="link">
	<view class="left">
		
	</view>
	<view class="right">
		
	</view>
</view>
一个大的容器class为link   left容器渲染左边菜单，right容器渲染菜单对应数据
left容器里面
<view v-for="(item,index) in order":key="index">
	{{item.title}}
</view>
right容器为
<view v-for="(item,index) in order" :key="index">
	{{item.title}}
	<view v-for="(it,idx) in item.list" :key="idx">
		{{it}}
	</view>
</view>

right容器经过二次便利，把list数据渲染出来

到这一步数据已经被渲染出来

1：实现右边数据可以滚动（scroll-view）
	把class = right的容器下面类容用scroll-view包裹起来
	
	<scroll-view scroll-y="true" style="border: 1px solid #007AFF; height: 200px;">
		<view v-for="(item,index) in order" :key="index">
			{{item.title}}
			<view v-for="(it,idx) in item.list" :key="idx">
				{{it}}
			</view>
		</view>
	</scroll-view>
	
	设定scroll-y="true",可以在Y轴方向滚动。需要注意，需要设定scroll-view的高度，表示scroll-view的范围
	到这一步可以实现右边数据进行滚动
2：实现右边title高亮显示
	{{item.title}}用<view>包裹，设定一个class
	<view class="rtitle">{{item.title}}</view>
	rtitle样式为background-color: red;
3：实现左边菜单点击，右边跳转到对应类容
	原理，scroll-view里面有scroll-into-view属性值应为某子元素id（id不能以数字开头）。设置哪个方向可滚动，则在哪个方向滚动到该元素
	（Uni-app 组件scroll-view中可查到）
	简单理解：scroll-into-view的值是什么，右边就跳转到对应这个值的地方。
	方法，右边高亮title都绑定一个id值，并且不是数字开头，左边点击菜单时候，每一个菜单点击后生成一个id值，刚好和右边同样的title的id值相同
	a:
		<view class="rtitle" :id="'po' + index">{{item.title}}</view>
		右边title分别绑定的id值为po0 po1 po2 ...
		数据绑定 冒号  双引号里面单引号 加index
	b:左边每一个title添加点击事件，点击后得到一个值和右边的id值相同
		首先需要在data里面添加一个clickId属性值，为空
		order:[
			{title:"xxx",list:["aaa","bbb","ccc"]},
			{title:"yyy",list:["111","222","222"]},
			...
		]，
		clickId:""
		每次点击的时候改变其值
		<view class="left">
			<view 
				v-for="(item,index) in order"
				:key="index"
				@click="setId(index)"
				>
				{{item.title}}
			</view>
		</view>
		
		在methods里面实现方法
		setId(index){
				this.clickId = "po" + index;
			}
		到此可以实现点击后和右边title绑定的id相同，现在需要做的就是用scroll-view的scroll-into-view属性
		只需要把clickId绑定到scroll-into-view即可。:scroll-into-view="clickId"
		
		<view class="right">
			<scroll-view 
				scroll-y="true"
				style="border: 1px solid #007AFF; height: 200px;"
				:scroll-into-view="clickId"
				>
				<view v-for="(item,index) in order" :key="index">
					<view class="rtitle" :id="'po' + index">{{item.title}}</view>
					<view v-for="(it,idx) in item.list" :key="idx">
						{{it}}
					</view>
				</view>
			</scroll-view>
		</view>
		到此可以实现，左边点击右边跳转到对应位置。但是会发现跳转比较生硬，使用scroll-view里面scroll-with-animation属性值，在设置滚动条位置时使用动画过渡
	c:左边菜单点击后实现背景颜色改变，开始第一个子选项背景颜色高亮
		在左边view容器里面给绑定一个class类  active
		.active{
			background-color: pink;
		}
		:class = "{active:curentNnm === index}"
		active右边为一个boolen值，只有curentNnm === index成立时候active类才生效
		所以，在data里面添加一个curentNum属性，初始值为0.这样可以保证没有点击的时候curentNum = 0, index=0
		order:[
			{title:"xxx",list:["aaa","bbb","ccc"]},
			{title:"yyy",list:["111","222","222"]},
			...
		]，
		clickId:"",
		curentNnm：0
		让第一个子选择背景颜色生效，同时在setID函数里面this.curentNum = index;
		setId(index){
				this.clickId = "po" + index;
				this.curentNum = index;
			}
		实现点击子选项，子选项背景颜色改变
	到此步，已经实现左边可以联动右边。
	
	d；实现右边滑动时候，左边菜单联动
		原理：右边在滑动时候回一直改变滑动的top值。把右边子选项里面的高亮菜单距离scroll-view 顶部的值拿出来存起来，
		在滑动时候，用滑动的距离和存起来的值做比较，判断在那个区间。然后改变左边子选择背景颜色高亮显示的位置
	e:保存右边子选项里面的高亮菜单距离scroll-view 顶部的值。
		因为在页面加载完成后，距离已经固定存在，所以在onLoad生命周期完成
		onLoad() {
			this.getNodeInfo();
		}
		在methods里面实现方法getNodeInfo
		
		uni.createSelectorQuery()
		返回一个 SelectorQuery 对象实例。可以在这个实例上使用 select 等方法选择节点，并使用 boundingClientRect 等方法选择需要查询的信息。
		
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
			}
			
		选用selectAll返回所有class为rtitled的实例对象
		data为返回的所有类容,使用一个map函数把我们需要的值取出来(item.top)
		需要注意的是，这个时候取出来的top值，是距离整个容器的top值，所以我们用item.top-data[0].top表示距离scroll-view顶部的值
		搞一个数组rel，把取出来的值push到rel里面。到此，数据有了，但是是局部变量，我们需要在整个页面使用，在data里面添加一个topList属性为数组
		初始值为空。然后赋值给topList ，this.topList = rel
		exec（）的作用是函数执行，如果没有它，函数不会执行
	f:到此，需要判断右边的滚动距离了。
		@scroll 滚动时触发，event.detail = {scrollLeft, scrollTop, scrollHeight, scrollWidth, deltaX, deltaY}
		在右边的scroll-view容器上绑定@scroll函数
		
		<view class="right">
			<scroll-view 
				scroll-y="true"
				style="border: 1px solid #007AFF; height: 200px;"
				:scroll-into-view="clickId"
				scroll-with-animation
				@scroll="scroll"
				>
				<view v-for="(item,index) in order" :key="index">
					<view class="rtitle" :id="'po' + index">{{item.title}}</view>
					<view v-for="(it,idx) in item.list" :key="idx">
						{{it}}
					</view>
				</view>
			</scroll-view>
		</view>
		
		在methods里面实现方法 scroll
		scroll(e){
			console.log(e.detail.scrollTop)
			let scrollTop = e.detail.scrollTop;
		}
		可以实时计算滚动距离，现在只需要和我们存储的值做比较就可以
		for循环来完成
		for (let i = 0; i < this.topList.length; i++) {
			let h1 = this.topList[i];
			let h2 = this.topList[i+1];
			
			if(scrollTop >= h1 && scrollTop < h2){
				this.curentNum = i;
			}
		}
		没滚动一次，就比较一次。判断滚动距离在那个区间，然后改变左边背景高亮位置this.curentNum = i;
		
		scroll（）方法具体写法如下
		scroll(e){
			//console.log(e.detail.scrollTop)
			let scrollTop = e.detail.scrollTop;
			for (let i = 0; i < this.topList.length; i++) {
				let h1 = this.topList[i];
				let h2 = this.topList[i+1];
				
				if(scrollTop >= h1 && scrollTop < h2){
					this.curentNum = i;
				}
			}
		}
		
	g:至此，基本完成双联动。测试发现滑动到最底部，左边高亮显示不对。
		原因，最后一个子菜单的高度不够一屏幕高，导致不能满足scrollTop大于this.topList[i]
		所以需要单独来判断，如果滑动到最底部了，直接让this.curentNnm = this.topList.length -1;
		又因为会马上满足滑动高度满足倒数第二个子选项的判断，导致闪屏一下最后一个子选项菜单高亮，然后马上改变为上面一个颜色高亮
		
		所以设置一个setTimeOut，异步  节流解决问题。给他足够反应时间
		
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
		在methods里面实现方法scrolltolower
		scrolltolower(){
			setTimeout(()=>{
				this.curentNum = this.topList.length-1;
			},100)
		}
		
	每天进步一点点！ george