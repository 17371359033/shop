<template>
    <div id="search">
        <div class="content">
            <div class="left">
                <svg viewBox="0 0 32 32" class="icon iconSearch" @click="refresh">
                    <path fill="#999" fill-rule="evenodd"
                          d="M23.624 21.503c3.47-4.51 3.14-11.003-.992-15.135-4.491-4.49-11.773-4.49-16.264 0-4.49 4.491-4.49 11.773 0 16.264 4.132 4.131 10.625 4.462 15.135.992l4.66 4.66a1.5 1.5 0 1 0 2.121-2.121l-4.66-4.66zm-3.114-.993A8.5 8.5 0 1 0 8.49 8.49a8.5 8.5 0 0 0 12.02 12.02z"></path>
                </svg>
                <input type="text" placeholder="请输入商品名称" v-model="content">
            </div>
            <div class="right">
                <span @click="searchShop('')">搜索</span>
            </div>
        </div>
<!--        v-if="shopData.length > 0"-->
        <div class="shopContent" v-if="shopData.length > 0">
            <shopList :shopResData="shopResData" :searchShop="searchShop"></shopList>
        </div>
        <div v-else>
            <van-loading  type="spinner" color="rgb(117, 163, 66)" text-size="16px"	 class="loading">加载中</van-loading>
        </div>
    </div>
</template>

<script>
    // 引入请求数据相关
    import {getSortRight} from './../server/index';
    // 引入商品展示组件
    import shopList from "./shopList";
    // 引入组件库相关
    import { Dialog } from 'vant';
    // 引入本地化文件
    import { setStore,getStore } from './../config/global';
    export default {
        name: "search",
        components: {
            shopList
        },
        data() {
            return {
                // 定义一个属性来存储所有的商品数据
                shopData: [],
                // 定义一个属性来控制输入框中的内容
                content: "",
                // 定义一个属性来存储搜索之后的数据
                shopResData: [],
                // 定义一个属性来存储搜索历史
                shopHistory: []
            }
        },
        computed: {

        },
        watch: {
            content(newName, oldName){
                console.log(newName);
                if(newName === ""){
                    location.reload();
                    this.shopResData = [];
                    // this.getShopData();
                    console.log(this.shopHistory);
                }
            }
        },
        created() {
            // 请求数据
            this.getShopData();
            if(getStore("shopHistory")){
                getStore("shopHistory").forEach((item)=>{
                    this.shopHistory.push(item);
                });
            }
            console.log(this.shopHistory);
            // this.shopHistory.push(getStore("shopHistory")[0]);
        },
        mounted() {

        },
        methods: {
            // 请求数据的具体方法
            async getShopData(){
                let products = [];
                let shopData = [];
                for (let i=1;i<7;i++){
                    let res = await getSortRight(i);
                    if(res.status === 200){
                        let data = res.data.data.cate;
                        data.forEach((item,index)=>{
                            products.push(item.products);
                        });
                    }
                }
                products.forEach((item)=>{
                    item.forEach((value)=>{
                        shopData.push(value);
                    });
                });
                this.shopData = shopData;
            },
            // 点击搜索之后就出现对应的数据
            searchShop(value){
                let shopResData = [];
                if(value === ""){
                    if(this.content.trim() !== ""){
                        this.shopData.forEach((item)=>{
                            if(item.product_name.indexOf(this.content.trim()) !== -1){
                                shopResData.push(item);
                            }
                        });
                        this.shopResData = shopResData;
                        // 将搜索记录保存到本地
                        this.shopHistory.push(this.content.trim());
                        this.shopHistory = Array.from(new Set(this.shopHistory));
                        setStore("shopHistory",this.shopHistory);
                    }else {
                        // Dialog({ message: '您还没有输入商品信息' });
                    }
                    // if(this.shopData.length > 0){
                    //     let shopData = [];
                    //     this.shopData.forEach((item)=>{
                    //         console.log(item);
                    //         if(item.product_name.indexOf(this.content.trim()) !== -1){
                    //             shopData.push(item);
                    //         }
                    //     });
                    //     this.shopData = [];
                    //     this.shopData = shopData;
                    //     if(this.content.trim() === ""){
                    //         this.shopData = [];
                    //     }
                    // }
                }else {
                    this.shopData.forEach((item)=>{
                        if(item.product_name.indexOf(value) !== -1){
                            shopResData.push(item);
                        }
                    });
                    this.shopResData = shopResData;
                }

            },
            // 刷新页面
            refresh(){
                location.reload();
            }
        },
    }
</script>

<style scoped lang="less">
    #search{
        width: 100%;
        height: 100%;
        position: fixed;
        left: 0;
        top: 0;
        background-color: white;
        z-index: 1000;
        overflow-y: scroll;
    }
    #search>.content{
        position: fixed;
        top: 0;
        width: 82%;
        margin: 10px 0 0 10px;
        height: 40px;
        background-color: white;
        border-radius: 20px;
        display: flex;
        flex-direction: row;
        z-index: 1001;
        background-color: #F5F5F5;
    }
    #search>.content>.left{
        padding: 0 10px;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: row;
        align-items: center;
    }
    #search>.content>.left>svg,input{
        height: 80%;
    }
    #search>.content>.left>svg{
        height: 60%;
    }
    #search>.content>.left>input{
        font-size: 16px;
        background-color: transparent;
        margin-left: 5px;
    }
    #search>.content>.right{
        position: fixed;
        right: 15px;
        top: 20px;
        color: red;
        font-size: 16px;
    }
    #search>.shopContent{
        margin-top: 70px;
    }
    .loading{
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%,-50%);
    }
</style>