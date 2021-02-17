<template>
    <div id="shopList">
        <div class="records" v-if="shopResData.length <= 0">
            <div class="title">
                <h2>历史搜索</h2>
                <span @click="deleteHistory"><van-icon name="delete" size="25px" /></span>
            </div>
            <div class="old-records">
                <span
                        v-for="(historyItem,index) in SearchHistory"
                        :key="index"
                        @click="searchShop(historyItem)"
                >
                    {{ historyItem }}
                </span>
            </div>
        </div>
        <div class="shop-list-content" v-else>
            <div class="shop-item" v-for="(shopItem,index) in shopResData" :key="index">
                <div class="shop-img">
                    <img :src="shopItem.small_image" alt="">
                </div>
                <div class="shop-title">
                    <span>{{ shopItem.product_name }}</span>
                </div>
                <div class="shop-price">
                    <div class="price">
                        <span>{{ shopItem.price | moneyFormat }}</span>
                    </div>
                    <div class="cartIcon">
                        <svg viewBox="0 0 52 52" class="icon iconCart" style="width: 30px;height: 30px;" @click="addCart(shopItem)">
                            <defs>
                                <radialGradient cx="27.0288363%" cy="10.3693483%" fx="27.0288363%" fy="10.3693483%" r="93.8427229%" id="radialGradient-1">
                                    <stop stop-color="#4ECA75" offset="0%"></stop><stop stop-color="#39B460" offset="100%"></stop>
                                </radialGradient>
                            </defs>
                            <g stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                                <g transform="translate(-678.000000, -2742.000000)"><g transform="translate(678.000000, 2742.000000)">
                                    <rect fill="url(#radialGradient-1)" x="0" y="0" width="51.8699976" height="51.8699976" rx="25.9349988"></rect><path fill="#FFFFFF" d="M20.3666667,39 C19.1700497,39 18.2,38.0299503 18.2,36.8333333 C18.2,35.6367164 19.1700497,34.6666667 20.3666667,34.6666667 C21.5632836,34.6666667 22.5333333,35.6367164 22.5333333,36.8333333 C22.5333333,38.0299503 21.5632836,39 20.3666667,39 Z M33.3666667,38.1333333 C32.1700497,38.1333333 31.2,37.1632836 31.2,35.9666667 C31.2,34.7700497 32.1700497,33.8 33.3666667,33.8 C34.5632836,33.8 35.5333333,34.7700497 35.5333333,35.9666667 C35.5333333,37.1632836 34.5632836,38.1333333 33.3666667,38.1333333 Z"></path><path stroke="#FFFFFF" stroke-width="2.6" stroke-linecap="round" d="M12.1333333,13.8666667 L13.6768756,13.8666667 C15.4621209,13.8666667 16.9554584,15.222463 17.1274221,16.9994069 L18.2224287,28.314386 C18.4068512,30.2200702 20.1012175,31.6154285 22.0069016,31.431006 C22.0111286,31.4305969 22.0153548,31.4301801 22.0195802,31.4297555 L33.2997819,30.296256 C34.7947282,30.1460352 36.0227397,29.0499108 36.3411182,27.581556 L37.8958814,20.4110205 C38.0987345,19.4754663 37.5047629,18.5526049 36.5692087,18.3497518 C36.3853963,18.3098964 36.1963225,18.3002236 36.0094025,18.3211126 L22.8968424,19.7864909">
                                </path>
                                </g>
                                </g>
                            </g>
                        </svg>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import { Toast,Dialog  } from 'vant';

    // 1.1 引入本地缓存相关
    import { setStore,getStore,removeStore } from './../config/global';
    // 1.2 引入数据相关
    import {getAddCart} from './../server/index';
    // 1.3 引入vuex相关
    import {mapState,mapMutations} from 'vuex';
    export default {
        name: "shopList",
        data() {
            return {
                // 定义一个属性保存搜索过的记录
                SearchHistory: [],
            }
        },
        props: {
            shopResData: Array,
            searchShop: Function
        },
        computed: {
            ...mapState(['userInfo']),
        },
        created() {
            this.SearchHistory = getStore("shopHistory");

        },
        mounted() {
            console.log(this.SearchHistory);
        },
        methods: {
            ...mapMutations(['ADD_CART']),
            // 删除所有的历史记录
            deleteHistory(){
                console.log(getStore("shopHistory"));
                if(getStore("shopHistory") === null){
                    Dialog({ message: '没有商品记录哦~~~' });
                }else {
                    Dialog({ message: '提示' });
                    Dialog.confirm({
                        title: '确定清空商品记录吗',
                        message: '请您三思~~~~',
                    }).then(()=>{
                        removeStore("shopHistory");
                        location.reload();
                    }).catch(()=>{

                    });
                }



            },
            // 商品加入购物车
            async addCart(shop){
                if(this.userInfo.token){
                    let addCartRes = await getAddCart(this.userInfo.token,shop.id, shop.name, shop.price, shop.small_image);
                    console.log(addCartRes);
                    if(addCartRes.data.success_code === 200){
                        let data = addCartRes.data;
                        console.log(data);
                        this.ADD_CART({
                            shopId: shop.id,
                            shopImg: shop.small_image,
                            shopName: shop.name,
                            shopPrice: shop.price
                        });
                        Toast('添加购物车成功');
                    }
                }else {
                    this.$router.push("/login");
                }
            }
        }
    }
</script>

<style scoped lang="less">
    #shopList{
        width: 100%;
    }
    #shopList>.records{
        padding: 0 10px;
    }
    #shopList>.records>.title{
        display: flex;
        flex-direction: row;
        justify-content: space-between;

    }
    #shopList>.records>.title>h2{
        font-size: 18px;
        font-weight: bolder;
    }
    #shopList>.records>.title>span{
        font-size: 16px;
    }
    #shopList>.records>.old-records{
        margin-top: 10px;
    }
    #shopList>.records>.old-records> span{
        display: inline-block;
        height: 25px;
        line-height: 25px;
        text-align: center;
        width: 80px;
        border-radius: 20px;
        background-color: #f5f5f5;
        margin: 8px;
    }
    #shopList>.shop-list-content{
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
    }
    #shopList>.shop-list-content>.shop-item{
        width: 32%;
        margin-left: 3px;
        margin-bottom: 10px;
    }
    #shopList>.shop-list-content>.shop-item>.shop-img{
        width: 100px;
        height: 100px;
    }
    #shopList>.shop-list-content>.shop-item>.shop-img>img{
        width: 100%;
    }
    #shopList>.shop-list-content>.shop-item>.shop-title{
        margin-top: 10px;
        width: 90%;
        overflow:hidden;
        text-overflow:ellipsis;
        display:-webkit-box;
        -webkit-box-orient:vertical;
        -webkit-line-clamp:1;
        font-size: 13px;
    }
    #shopList>.shop-list-content>.shop-item>.shop-price{
        display: flex;
        flex-direction: row;
        margin-top: 10px;
        justify-content: space-between;
        align-items: center;
    }
    #shopList>.shop-list-content>.shop-item>.shop-price svg{
        margin-right: 10px;
    }
</style>