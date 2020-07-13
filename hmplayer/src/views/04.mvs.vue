<template>
  <div class="mvs-container">
    <div class="filter-wrap">
      <div class="seciton-wrap">
        <span class="section-type">地区:</span>
        <ul class="tabs-wrap">
          <li class="tab">
            <span class="title" @click="area='全部'" :class="{active:area=='全部'}">全部</span>
          </li>
          <li class="tab">
            <span class="title" @click="area='内地'" :class="{active:area=='内地'}">内地</span>
          </li>
          <li class="tab">
            <span class="title" @click="area='港台'" :class="{active:area=='港台'}">港台</span>
          </li>
          <li class="tab">
            <span class="title" @click="area='欧美'" :class="{active:area=='欧美'}">欧美</span>
          </li>
          <li class="tab">
            <span class="title" @click="area='日本'" :class="{active:area=='日本'}">日本</span>
          </li>
          <li class="tab">
            <span class="title" @click="area='韩国'" :class="{active:area=='韩国'}">韩国</span>
          </li>
        </ul>
      </div>
      <div class="type-wrap">
        <span class="type-type">类型:</span>
        <ul class="tabs-wrap">
          <li class="tab">
            <span class="title" :class="{active:type=='全部'}" @click="type='全部'">全部</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:type=='官方版'}" @click="type='官方版'">官方版</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:type=='原声'}" @click="type='原声'">原声</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:type=='现场版'}" @click="type='现场版'">现场版</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:type=='网易出品'}" @click="type='网易出品'">网易出品</span>
          </li>
        </ul>
      </div>
      <div class="order-wrap">
        <span class="order-type">排序:</span>
        <ul class="tabs-wrap">
          <li class="tab">
            <span class="title" :class="{active:order=='上升最快'}" @click="order='上升最快'">上升最快</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:order=='最热'}" @click="order='最热'">最热</span>
          </li>
          <li class="tab">
            <span class="title" :class="{active:order=='最新'}" @click="order='最新'">最新</span>
          </li>
        </ul>
      </div>
    </div>
    <!-- mv容器 -->
    <!-- 推荐MV -->
    <div class="mvs">
      <div class="items">
        <div class="item" @click="toMv(item.id)" v-for="(item,index) in list" :key="index">
          <div class="img-wrap">
            <img :src="item.cover" alt="" />
            <div class="num-wrap">
              <div class="iconfont icon-play"></div>
              <div class="num">{{item.playCount}}</div>
            </div>
          </div>
          <div class="info-wrap">
            <div class="name">{{item.name}}</div>
            <div class="singer">{{item.artistName}}</div>
          </div>
        </div>
      </div>
      <!-- 分页器 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="5"
        :limit="limit"
      >
      </el-pagination>
    </div>
  </div>
</template>

<script>
//导入axios
import axios from 'axios'
export default {
  name: 'mvs',
  data() {
    return {
      // 总条数
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 8,
      //地区
      area:"全部",
      //类型
      type:"全部",
      //排序
      order:"上升最快",
      list:[]
    };
  },
  watch: {
    area(){
      this.getList();
      this.page = 1;
    },
    type(){
      this.getList();
      this.page = 1;
    },
    order(){
      this.getList();
      this.page = 1;
    }
  },
  created() {
    //获取列表数据
    this.getList();
  },
  methods: {
    toMv(id) {
      this.$router.push(`/mv?id=${id}`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page = val;
      this.getList();
    },
    getList(){
      axios({
      url:'https://autumnfish.cn/mv/all',
      method:'get',
      params:{
        area:this.area,
        type:this.type,
        order:this.order,
        //数量
        limit:this.limit,
        //偏移量 分页
        offset:(this.page-1)*this.limit
      }
      }).then(res=>{
        //console.log(res)
        this.list = res.data.data;
        for(let i=0;i<this.list.length;i++){
          if(this.list[i].playCount > 100000){
            this.list[i].playCount = parseInt(this.list[i].playCount/10000)+'万';
          }
        }
        //保存总条数
        //接口问题
        if(res.data.count){
          
        }
        
      })
    }
  }
};
</script>

<style >

</style>
