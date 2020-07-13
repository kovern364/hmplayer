<template>
  <div class="songs-container">
    <div class="tab-bar">
      <span class="item" @click="tag=0" :class="{active:tag==0}">全部</span>
      <span class="item" @click="tag=7" :class="{active:tag==7}">华语</span>
      <span class="item" @click="tag=96" :class="{active:tag==96}">欧美</span>
      <span class="item" @click="tag=8" :class="{active:tag==8}">日本</span>
      <span class="item" @click="tag=16" :class="{active:tag==16}">韩国</span>
    </div>
    <!-- 底部的table -->
    <table class="el-table playlit-table">
      <thead>
        <th></th>
        <th></th>
        <th>音乐标题</th>
        <th>歌手</th>
        <th>专辑</th>
        <th>时长</th>
      </thead>
      <tbody>
        <tr class="el-table__row" v-for="(item,index) in lists" :key="index">
          <td>{{index+1}}</td>
          <td>
            <div class="img-wrap">
              <img :src="item.album.picUrl" alt="" />
              <!-- 播放按钮 -->
              <span class="iconfont icon-play" @click="playMusic(item.id)"></span>
            </div>
          </td>
          <td>
            <div class="song-wrap">
              <div class="name-wrap">
                <span>{{item.name}}</span>
                <span class="iconfont icon-mv"></span>
              </div>
              <span>{{item.album.name}}</span>
            </div>
          </td>
          <td>{{item.album.artists[0].name}}</td>
          <td>{{item.album.name}}</td>
          <td>{{item.duration}}</td>
        </tr>

      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'songs',
  data() {
    return {
      //歌曲列表
      lists:[],
      tag:'0',
      url:''
    };
  },
  watch: {
    tag(){
      this.getList();
    }
  },
  created(){
    //获取 最新音乐
    this.getList();
  },
  methods:{
    //获取列表数据
    getList(){
        axios({
        url:'https://autumnfish.cn/top/song',
        method:'get',
        params:{
          type:this.tag
        }
      }).then(res=>{
        this.lists = res.data.data;
        //处理时长      
        for(let i = 0;i<this.lists.length;i++){
          let duration = this.lists[i].duration;
          let min = parseInt(duration/1000/60);
          let sec = parseInt(duration/1000%60)
          min = min < 10 ? '0' + min : min;
          sec = sec < 10 ? '0' + sec : sec;
          //console.log(min+"|"+sec);
          this.lists[i].duration = `${min}:${sec}`
        }
      })
    },
    //播放歌曲
    playMusic(id){
       axios({
        url:'https://autumnfish.cn/song/url',
        method:'get',
        params:{
          id
        }
      }).then(res=>{
        //直接获取父组件，可以修改任意的值
        this.$parent.musicUrl = res.data.data[0].url;
      })
    }
  }
};
</script>

<style >

</style>
