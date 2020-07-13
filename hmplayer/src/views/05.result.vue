<template>
  <div class="result-container">
    <div class="title-wrap">
      <h2 class="title">{{$route.query.q}}</h2>
      <span class="sub-title">找到{{count}}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in songList" :key="index" @dblclick="playMusic(item.id)">
              <td>1</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <span v-if="item.mvid!=0" class="iconfont icon-mv"></span>
                  </div>
                  <span v-if="item.alias.length!=0">{{item.alias[0]}}</span>
                </div>
              </td>
              <td>{{item.artists[0].name}}</td>
              <td>{{item.album.name}}</td>
              <td>{{item.duration}}</td>
            </tr>


          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item,index) in playlists" :key="index" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{item.playCount}}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{item.name}}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item,index) in mvs" :key="index" @click="toMV(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">{{item.duration}}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{item.name}}</div>
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'result',
  data() {
    return {
      //tab切换时会改变的数据
      activeIndex: 'songs',
      //保存查询歌曲
      songList:[],
      //保存歌单的字段
      playlists:[],
      //保存mv的字段
      mvs:[],
      //搜索结果的总数
      count:0
    };
  },
  //侦听器
  watch: {
    activeIndex(){
      //songs歌曲
      //lists歌单
      //mv
      let type =1 ;
      //获取个数
      let limit = 10;
      switch(this.activeIndex){
        case 'songs':
          type = 1
          limit = 8
          break;
        case 'lists':
          type = 1000;
          break;
        case 'mv':
          type = 1004
          limit = 8
          break;

      }
      axios({
      url:'https://autumnfish.cn/search',
      method:'get',
      params:{
        keywords:this.$route.query.q,
        type:type,
        limit:limit
      }
      }).then(res=>{
      //console.log(res)
      //获取歌曲
      if(type == 1){
        this.songList = res.data.result.songs;
        //计算歌曲时间
        for(let i=0;i<this.songList.length;i++){
            let duration = this.songList[i].duration;
            let min = parseInt(duration/1000/60);
            let sec = parseInt(duration/1000%60);
            min = min < 10 ? '0' + min : min;
            sec = sec < 10 ? '0' + sec : sec;
            //console.log(min+"|"+sec);
            this.songList[i].duration = `${min}:${sec}`;
          }
        //保存搜索总数
        this.count = res.data.result.songCount;
      }else if(type == 1000){
        //歌单逻辑
        this.playlists = res.data.result.playlists;
        //保存总数
        this.count = res.data.result.playlistCount;
        //处理播放量
          for(let i=0;i<this.playlists.length;i++){ 
          if(this.playlists[i].playCount > 100000){
            this.playlists[i].playCount = parseInt(this.playlists[i].playCount/10000)+'万';
          }
        }
      }else{
        //保存mv字段
        this.mvs = res.data.result.mvs;
        //保存总数
        this.count = res.data.result.mvCount;
        //处理播放量
        for(let i=0;i<this.mvs.length;i++){ 
          //时间
          let min = parseInt(this.mvs[i].duration/1000/60)
          let sec = parseInt(this.mvs[i].duration/1000%60)
          if(min<10){
            min = '0' + min;
          }
          if(sec<10){
            sec = '0' + sec;
          }
          this.mvs[i].duration = `${min}:${sec}`;
          //播放次数
          if(this.mvs[i].playCount > 100000){
            this.mvs[i].playCount = parseInt(this.mvs[i].playCount/10000)+'万';
          }
        }
      }   
    })
    }
  },
  //生命周期钩子 回调函数
  created() {
    axios({
      url:'https://autumnfish.cn/search',
      method:'get',
      params:{
        keywords:this.$route.query.q,
        type:1,
        limit:10
      }
    }).then(res=>{
      //console.log(res)
      this.songList = res.data.result.songs;
      //计算歌曲时间
      for(let i=0;i<this.songList.length;i++){
        let duration = this.songList[i].duration;
          let min = parseInt(duration/1000/60);
          let sec = parseInt(duration/1000%60);
          min = min < 10 ? '0' + min : min;
          sec = sec < 10 ? '0' + sec : sec;
          //console.log(min+"|"+sec);
          this.songList[i].duration = `${min}:${sec}`
      }
      //保存搜索总数
      this.count = res.data.result.songCount;
    })
  },
  methods: {
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
    },
    //去歌单详情页并携带数据
    toPlaylist(id){
      this.$router.push('/playlist?id='+ id)
    },
    toMV(id){
      this.$router.push(`/mv?id=${id}`)
    }
  },
};
</script>

<style >

</style>
