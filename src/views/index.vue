<template>
    <div>
        <select class="project" v-model="project">
			<option v-for="(pro, index) in projects" :key="index" :value="pro">{{ pro }}</option>
		</select>
        <div class="top-warp">
            <div class="nav">
                <div :class="{'active':pdType==0}" @click="changeTab(0)">房间设备</div>
				<div :class="{'active':pdType==1}" @click="changeTab(1)">审批</div>
            </div>
        </div>
        <div class="top-warp2">
			<div class="nav2" v-show="pdType==1">
				<div :class="{'active':tabType2==1}" @click="changeTab2(1)"><span>我的待办</span></div>
				<div :class="{'active':tabType2==2}" @click="changeTab2(2)"><span>我的已办</span></div>
				<div :class="{'active':tabType2==3}" @click="changeTab2(3)"><span>历史纪录</span></div>
			</div>
			<div class="nav3" v-show="pdType==0">
				<div class="search" @click="focusInput">
					<input v-model="key" @input='search' placeholder="输入房间名称搜索" id="search">
					<img src="../assets/search-icon.png">
				</div>
			</div>
		</div>
        <mescroll-vue ref="mescroll" :up="mescrollUp" @init="mescrollInit">
            <ul id="dataList" class="data-list">
                <template v-if="pdType == 0">
                    <li class="data-li-1 clearfix" v-for="dv in dataList" :key="dv.id">
                        <div class="floatL">
                            <span :class="{'status': 'true', 'left':dv.room_status=='逾'}">{{ dv.room_status }}</span>
                            <span class="room-name">{{ dv.room_name }}</span>
                        </div>
                        <div class="floatR">
                            <span class="customer-name">{{ dv.customer_name }}</span>
                            <span class="tel">{{ dv.tel }}</span>
                            <span class="rote"></span>
                        </div>
                    </li>
                </template>
                <template v-if="pdType == 1">
                    <li class="data-li-2" v-for="ap in dataList" :key="ap.id">
                        <div class="line-1"><span>{{ ap.title }}</span><span class="status">{{ ap.status }}</span></div>
                        <div class="line-2"><span>{{ ap.apply_name }}</span>&nbsp;&nbsp;<span>{{ ap.create_time }}</span></div>
                        <div class="line-3"><span>已过去 1 天 2 时 31 分 58 秒 </span></div>
                    </li>
                </template>
            </ul>
        </mescroll-vue>
    </div>
</template>

<script>
import MescrollVue from 'mescroll.js/mescroll.vue'
import Data1 from '../js/data1'
import Data2 from '../js/data2'
import Data3 from '../js/data3'
import Data4 from '../js/data4'
import utils from "../js/utils";
export default {
    name: 'mescrollComponent',
    components: {
        MescrollVue
    },
    data () {
        return {
            project: '南山科技园店',
            projects: ['南山科技园店', '坂田星火店', '茶光路店'],
            key: '',
            mescroll: null,                 // mescroll实例对象
            mescrollUp: {
                callback: this.upCallback, // 上拉回调
                page: {
                    num: 0,                // 当前页码,默认0,回调之前会加1,即callback(page)会从1开始
                    size: 15               // 每页数据的数量
                },
                noMoreSize: 5,             // 如果列表已无数据,可设置列表的总数量要大于等于5条才显示无更多数据;避免列表数据过少(比如只有一条数据),显示无更多数据会不好看
                toTop: {
                    src: 'http://www.mescroll.com/img/mescroll-totop.png' // 回到顶部按钮的图片路径,支持网络图
                },
                empty: {                   // 列表第一页无任何数据时,显示的空提示布局; 需配置warpId才生效;
                    warpId: 'dataList',    // 父布局的id;
                    icon: 'http://www.mescroll.com/img/mescroll-empty.png', // 图标,支持网络图
                    tip: '暂无相关数据~',   // 提示
                },
                lazyLoad: {
                    use: true              // 是否开启懒加载,默认false
                }
            },
            dataList: [],                  // 列表数据
            pdType: 0,                     // 菜单
            tabType2: 0
        }
    },
    watch: {
		"project": {
			handler() {
                this.key = '';
				this.changeTab(this.tabType2, true);
			}
        },
	},
    beforeRouteEnter (to, from, next) { // 如果没有配置回到顶部按钮或isBounce,则beforeRouteEnter不用写
        next(vm => {
            // 找到当前mescroll的ref,调用子组件mescroll-vue的beforeRouteEnter方法
            vm.$refs.mescroll && vm.$refs.mescroll.beforeRouteEnter() // 进入路由时,滚动到原来的列表位置,恢复回到顶部按钮和isBounce的配置
        })
    },
    beforeRouteLeave (to, from, next) { // 如果没有配置回到顶部按钮或isBounce,则beforeRouteLeave不用写
        // 找到当前mescroll的ref,调用子组件mescroll-vue的beforeRouteEnter方法
        this.$refs.mescroll && this.$refs.mescroll.beforeRouteLeave() // 退出路由时,记录列表滚动的位置,隐藏回到顶部按钮和isBounce的配置
        next()
    },
    methods: {
        focusInput () {
			document.getElementById('search').focus();
		},
        search: utils.debounce(function() {
            this.changeTab(0, true);
        }),
        // mescroll组件初始化的回调,可获取到mescroll对象
        mescrollInit (mescroll) {
            this.mescroll = mescroll
        },
        // 上拉回调 page = {num:1, size:10}; num:当前页 ,默认从1开始; size:每页数据条数(可配置),默认10
        upCallback (page, mescroll) {
            // 模拟联网
            if(this.pdType == 0) {
                this.getListDataFromNet1(this.pdType, page.num, page.size, (arr) => {
                    // 如果是第一页需手动制空列表
                    if (page.num === 1) this.dataList = []
                    // 把请求到的数据添加到列表
                    this.dataList = this.dataList.concat(arr)
                    // 数据渲染成功后,隐藏下拉刷新的状态
                    this.$nextTick(() => {
                        mescroll.endSuccess(arr.length)
                    })
                }, () => {
                    // 联网异常,隐藏上拉和下拉的加载进度
                    mescroll.endErr()
                })
            }
            if(this.pdType == 1) {
                this.getListDataFromNet2(this.pdType, page.num, page.size, (arr) => {
                    // 如果是第一页需手动制空列表
                    if (page.num === 1) this.dataList = []
                    // 把请求到的数据添加到列表
                    this.dataList = this.dataList.concat(arr)
                    // 数据渲染成功后,隐藏下拉刷新的状态
                    this.$nextTick(() => {
                        mescroll.endSuccess(arr.length)
                    })
                }, () => {
                    // 联网异常,隐藏上拉和下拉的加载进度
                    mescroll.endErr()
                })
            }
        },

        changeTab2(type) {
            this.changeTab(type, true);
        },

        // 切换菜单
        changeTab (type, flag) {
            if (this.pdType !== type || flag) {
                this.pdType = type == 0 ? 0 : 1;
                this.tabType2 = type;
                this.dataList = []// 在这里手动置空列表,可显示加载中的请求进度
                this.mescroll.resetUpScroll() // 刷新列表数据
            }
        },

        /* 联网加载列表数据
        在您的实际项目中,请参考官方写法: http://www.mescroll.com/api.html#tagUpCallback
        请忽略getListDataFromNet的逻辑,这里仅仅是在本地模拟分页数据,本地演示用
        实际项目以您服务器接口返回的数据为准,无需本地处理分页.
        * */
        getListDataFromNet1 (pdType, pageNum, pageSize, successCallback, errorCallback) {
            // 延时一秒,模拟联网
            setTimeout(() => {
                // axios.get("xxxxxx", {
                //   params: {
                //     num: page.num, //页码
                //     size: page.size //每页长度
                //   }
                // }).then((response)=> {
                var listData = []
                for (var i = (pageNum - 1) * pageSize; i < pageNum * pageSize; i++) {
                    if (i === Data1.length) break
                    listData.push(Data1[i])
                }
                // 回调
                successCallback(listData)
                // }).catch((e)=> {
                //   //联网失败的回调,隐藏下拉刷新和上拉加载的状态;
                //   errorCallback&&errorCallback()
                // })
            }, 1000)
        },

        getListDataFromNet2 (pdType, pageNum, pageSize, successCallback, errorCallback) {
            // 延时一秒,模拟联网
            setTimeout(() => {
                // axios.get("xxxxxx", {
                //   params: {
                //     num: page.num, //页码
                //     size: page.size //每页长度
                //   }
                // }).then((response)=> {
                let listData = []
                let ndata = [];
                if (this.tabType2 == 1) ndata = Data2;
                if (this.tabType2 == 2) ndata = Data3;
                if (this.tabType2 == 3) ndata = Data4;
                for (let j = (pageNum - 1) * pageSize; j < pageNum * pageSize; j++) {
                    if (j === ndata.length) break;
                    listData.push(ndata[j]);
                }
                // 回调
                successCallback(listData)
                // }).catch((e)=> {
                //   //联网失败的回调,隐藏下拉刷新和上拉加载的状态;
                //   errorCallback&&errorCallback()
                // })
            }, 1000)
        }
    }
}
</script>

<style scoped>
*{ margin:0; padding:0; outline:0; font-family: 'PingFang SC', 'Helvetica Neue', 'Helvetica', 'STHeitiSC-Light', "Microsoft YaHei", 'Arial', sans-serif;}
ul,dl,ol,li,dt,dd{list-style:none;}
input{-webkit-appearance:none;border:none;outline:none; font-size:14px;}
.floatL{ display:inline; float:left;}
.floatR{ display:inline; float:right;}
.clearfix:before, .clearfix:after{ content:""; display:table; }
.clearfix:after { clear:both; }

/*以fixed的方式固定mescroll的高度*/
.mescroll {
    position: fixed;
    top: 120px;
    bottom: 0;
    height: auto;
}
.project {
	width: 100%;
	height: 30px;
	line-height: 30px;
	border: 1px solid #ccc;
}
.top-warp {
	position: relative;
	z-index: 9990;
	width: 100%;
	height: 40px;
	background-color: white;
	border-bottom: 1px solid #ccc;
}
.top-warp .tip {
	font-size: 14px;
	height: 40px;
	line-height: 40px;
	text-align: center;
}
.top-warp .nav {
	text-align: center;
	height: 40px;
	background: white;
}
.top-warp .nav > div {
	float: left;
	width: 50%;
	line-height: 40px;
	font-size: 14px;
}
.top-warp .nav div:first-child {
	border-right: 1px solid #ccc;
	box-sizing: border-box;
}
.top-warp .nav .active {
	color: #0099FF;
}
.top-warp2 {
	position: relative;
	z-index: 9990;
	width: 100%;
	height: 40px;
	background-color: white;
}
.top-warp2 .nav2 {
	text-align: center;
	height: 40px;
	background: white;
	border-bottom: 1px solid #ccc;
	padding: 0 30px;
}
.top-warp2 .nav2 > div {
	float: left;
	width: 33%;
	height: 40px;
	line-height: 40px;
	font-size: 14px;
}
.top-warp2 .nav2 > div span {
	display: inline-block;
    border: 1px solid #ccc;
    height: 20px;
    line-height: 20px;
    padding: 0 10px;
    border-radius: 25px;
    font-size: 12px;
}
.top-warp2 .nav2 .active {
	color: #0099FF;
}
.top-warp2 .nav3 {
	height: 40px;
	line-height: 40px;
	text-align: center;
	margin: 0 10px;
	border-bottom: 1px solid #eee;
}
.top-warp2 .nav3 .search {
	border: 1px solid #eee;
    margin-top: 7px;
    box-sizing: border-box;
    border-radius: 25px;
    text-align: left;
    padding-left: 20px;
    height: 30px;
    line-height: 30px;
}
.top-warp2 .nav3 .search input {
	width: 85%;
}
input::-webkit-input-placeholder {
	color: #9e9e9e;
}
input:-moz-placeholder {
	color: #9e9e9e;
}
input::-moz-placeholder {
	color: #9e9e9e;
}
input::-ms-input-placeholder {
	color: #9e9e9e;
}
.top-warp2 .nav3 .search img {
	width: 20px;
    float: right;
    margin-right: 5px;
    margin-top: 5px;
}
/*展示上拉加载的数据列表*/
#dataList0 {
	padding: 0 10px;
}
.data-li-1 {
	position: relative;
	height: 50px;
	line-height: 50px;
	padding: 0 8px;
	border-bottom: 1px solid #eee;
    overflow: hidden;
}
.data-li-1 span {
	display: inline-block;
	vertical-align: middle;
}
.data-li-1 .status {
	color: #fff;
	font-size: 12px;
	background: #0099FF;
	border-radius: 3px;
	margin-right: 5px;
	height: 20px;
    line-height: 20px;
    padding: 0 4px;
}
.data-li-1 .status.left {
	background: red;
}
.data-li-1 .room-name {
	color: #000;
	margin-right: 10px;
}
.data-li-1 .customer-name {
	color: #666;
	margin-right: 5px;
}
.data-li-1 .tel {
	color: #666;
	margin-right: 10px;
}
.data-li-1 .rote {
	width: 6px;
	height: 6px;
	border-top: 1px solid #666;
	border-right: 1px solid #666;
	transform: rotate(45deg);
}

.data-li-2 {
	position: relative;
	padding: 8px 15px;
	border-bottom: 1px solid #eee;
	font-size: 12px;
}
.data-li-2 .line-1, .data-li-2 .line-2 {
	margin-bottom: 10px;
}
.data-li-2 .line-3 {
	text-align: right;
	color: #666666;
}
.data-li-2 .line-1 {
	color: #000;
}
.data-li-2 .line-1 .status {
	float: right;
	color: #0099FF;
}
.data-li-2 .line-2 {
	color: #666666;
}
.FC_ec6b39 {
    color: #ec6b39;
}
.FC_999999 {
    color: #999999;
}
</style>
