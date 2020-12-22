本项目使用uni-app实现前端页面布局，数据请求。后端采用node.js  express mongoDB完成

开始阶段，HBuilder X 建立uni-app项目
在mainfest.json中选择微信小程序配置
	输入微信小程序AppID等其他选项
	
页面配置
本项目前端分为 首页(index) 购物车(cart) 详情页(detail) 用户页面(user)
其中详情页面不在tabbar选项里面，由商品点击跳转

在pages.json中配置pages，tabBar，tabBar图标放在static/img目录下

index页面主要为顶部logo 轮播图 静态文字，图片 以及商品列表

在页面onShow阶段（监听页面显示。页面每次出现在屏幕上都触发，包括从下级页面点返回露出当前页面），发送请求获取数据进行展示
后端已经完成接口，在数据库中对数据进行存储后，在前端发起请求即可
uni.request({
	url: 'http://localhost:3000/index', //仅为示例，并非真实接口地址。
	success: (res) => {
		console.log(res.data);
		this.banners = res.data.banners;
		this.icons = res.data.icons;
		this.goods = res.data.goods;
		this.$store.state.goods = this.goods
		//console.log(this.goods)
		//console.log(this.$store.state)
	}
});

在后端设计到一个连接数据库操作（带密码）
mongoose.connect("mongodb://yuan:123456@localhost:27017/cakeTest",{
	useNewUrlParser:true},(err)=>{
	if(err){
		console.log('--------------')
		console.log('数据库链接失败',err)
		console.log('--------------')
		return;
	}else{
		console.log('数据库链接成功')
		//db.close()
	}
	
})
1：这里设计到一个为什么要用connect连接而不是createConnection连接问题
2：json数据处理
	a:严格的数据格式
	b:数据导入（不用进入数据库后操作，在任意目录下即可）   mongoimport --host 127.0.0.1 --port 27017 -u yuan -p 123456 --db cakeTest -c icons C:/Users/yuan/Desktop/data/icons.json
	c:mongoDB数据库操作
	在任意目录下mongo开启数据库后，选择  admin数据库
	db.auth('name','password')登入数据库
	
在uni-app中发送请求后，数据返回后。为了后续使用方便，对返回的数据中goods数据进行store处理
建立store目录下创建index.js文件  引入vue  vuex  使用vuex
const store = new Vuex.Store({
	state:{
		goods:[]
	},
	mutations:{
		
	}
})
导出store
export default store;
在main.js中引入 store，全局可用
	****  Vue.prototype.$store = store;
	给vue原型绑定store
	
获取数据后，对index页面进行数据渲染，布局显示
轮播图用uni-app给的组件swiper 自己设定参数:indicator-dots="true" :autoplay="true" :interval="1000"
商品图片点击进入详情页面，给商品图片点击事件toPath(index)
toPath(index){
	uni.navigateTo({
		url:"../detail/detail?index="+index
	})
}
每一页详情页面的商品数据不一样，所以在进入详情页面需要带参数 Index  index为数据渲染时候的key值。进入详情页面后，根据唯一的key值来判断具体显示那条数据
在详情页面onLoad  生命周期  监听页面加载，其参数为上个页面传递的数据，参数类型为Object（用于页面传参）
onLoad(e) {
	//console.log(e.index)
	//console.log(this.$store.state.goods)
	this.goods = this.$store.state.goods[e.index];
	this.details = this.goods.details[0];
	console.log(this.details)
}
可以获取传递过来的参数  然后用store中的goods来找到这条商品，根据得到的数据进行数据渲染

购物车按钮添加点击事件add(item)，让商品点击进入购物车，因为点击后要一直保存  所以也使用store里面的commit方法
add(item){
	console.log(item);
	this.$store.commit("add",item)
}

在store 的 index.js中  mutations下增加add方法，state下增加cartList:[]数组

add(state,data){
	console.log(data._id)
	var bool = true;
	state.cartList.map(res=>{
		if(res._id === data._id){
			uni.showModal({
				title: "提示",
				content: "该商品已经在购物车,请选择其他商品添加",
				showCancel:false
			})
			bool = false;
		}
	})
	if(bool){
		data.num = 1;
		state.cartList.push(data)
	}
}
在首页的add方法中add(item)参数item为一条商品的全部数据。数据渲染的时候v-for="(item,index) in goods"
然后用commit调用store下的mutation下的add方法，并且把item传过去

在mutation下 通过唯一的_id来判断商品是否在购物车中，如果不在 就把商品push到cartList中，并添加一个自定义属性num=1
如果在了就不添加  用到map函数，如果res为空，map函数不执行


在购物车页面，首先要判断是否有商品被添加到购物车 否则显示  您购物车空空如也,请到首页去选购

通过cartList的长度来判断v-if="cartList.length !== 0"
对得到的数据进行渲染
实现+-操作
@click="sub(true,index)"
@click="sub(false,index)

用一个函数来实现，参数不一样  true为加   false为减
index为数据渲染时候的key值  代表每一条数据的唯一值
sub(bool,index){
	if(bool){
		this.cartList[index].num++;
	}else{
		this.cartList[index].num--;
	}
	if(this.cartList[index].num <=0){
		this.cartList.splice(index,1);
	}else{
		this.$set(this.cartList,index,this.cartList[index]);
	}，
	this.countPrice();
}
如果减到0后数据从页面删除

在进入购物车页面，如果有数据还需要在进入时候计算总价
在onShow中
onShow() {
	this.cartList = this.$store.state.cartList;
	var  allPrice = 0;//data里面参数
	this.cartList.map(item=>{
		allPrice += item.Price * item.num;
	}) 
	this.allPrice = allPrice.toFixed(2);
	uni.hideTabBar();
}

每次点击+-按钮后都需要从新计算总价
countPrice(){
	var allPrice = 0;
	this.cartList.map(item=>{
		allPrice += item.Price * item.num;
	})
	console.log(allPrice)
	this.allPrice = allPrice.toFixed(2);
}

点击返回按钮 跳转到index也没  这是相当于tabber跳转 需要使用switchTab
给返回view添加toPath事件
toPath(){
	uni.switchTab({
		url:"../index/index"
	})
}


用户登入注册页面
登入注册 Mine为同一个页面，用active来分别显示
默认active =1
v-if="active ===1" 登入
v-if="active ===2" 我的
v-if="active ===3" 祖册

数据库中有一张user表
登入接口为
uni.request({
	url: 'http://localhost:3000/user/login', //仅为示例，并非真实接口地址。
	method: "POST",
	data,
	success: (res) => {
		if(res.data.code === "10001" ||res.data.code === "10002"){
			uni.showModal({
				title: "提示",
				content: res.data.msg,
				showCancel:false
			})
		}else{
			uni.showModal({
				title: "提示",
				content: res.data.msg,
				showCancel:false,
				success: () => {
					this.active = 2;
					this.userInfo = res.data.info;
					uni.setStorage({
						key:"userInfo",
						data:res.data.info
					})
				}
			})
		}
	}
});

登入成功后，需要把用户信息存储起来 
uni.setStorage({
	key:"userInfo",
	data:res.data.info
})

在data参数中
注册接口为userInfo默认为空
如果有登入成功的用户，进入mine页面 
uni.getStorage({
	key:"userInfo",
	success: (res) => {
		this.userInfo = res.data;
		this.active = 2;
	}
})
userInfo里面的信息用来渲染我的页面

if(data.phone !== '' && data.pwd !== ''){
	uni.request({
		url: 'http://localhost:3000/user/reg', //仅为示例，并非真实接口地址。
		method: "POST",
		data,
		success: (res) => {
			if(res.data.code === "10001"){
				uni.showModal({
					title: "提示",
					content: res.data.msg,
					showCancel:false
				})
			}else{
				uni.showModal({
					title: "提示",
					content: res.data.msg,
					showCancel:false,
					complete: () => {
						this.active = 1;
					}
				})
			}
		}
	});
}else{
	uni.showModal({
		title: "提示",
		content: "账号不能为空",
		showCancel:false
	})
}


项目遇到几个小问题
1：数据库链接成功 但是请求成功不能返回数据 
	connect createConnection
2:页面布局
	按照需求一步步来，细心
3：store
	Vue.prototype.$store = store;
4：点击添加购物车后，没有计算总数
	在onShow时候就需要计算
5：购物车+-实现用一个函数

6：按需编译  #ifdef  #endif
7：真机测试文件过大
	static里面放了很多没有关系的文件