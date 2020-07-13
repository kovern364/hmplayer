<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          :src="mvUrl"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="icon" alt="" />
          </div>
          <span class="name">{{mvinfo.artistName}}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{mvinfo.briefDesc}}</h2>
          <span class="date">发布：2014-11-04</span>
          <span class="number">播放：{{mvinfo.playCount}}次</span>
          <p class="desc">
            {{mvinfo.desc}}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap">
        <p class="title">精彩评论<span class="number">(15)</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}: </span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              <div class="date">2020-02-12 17:26:11</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">({{total}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}：</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}: </span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              <div class="date">2020-02-12 17:26:11</div>
            </div>
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
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item,index) in simiMvs" :key="index" @click="toMV(item.id)">
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
              <div class="singer">{{item.artistname}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'mv',
  data() {
    return {
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      mvUrl:'',
      //相关mv
      simiMvs:[],
      //mv信息
      mvinfo:'',
      //头像
      icon:'',
      hotComments:[],
      comments:[]
    };
  },
  created() {
    //获取mv地址
    axios({
      url:'https://autumnfish.cn/mv/url',
      method:'get',
      params:{
        id:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.mvUrl = res.data.data.url;
    })
    //获取相关mv
    axios({
      url:'https://autumnfish.cn/simi/mv',
      method:'get',
      params:{
        mvid:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.simiMvs = res.data.mvs;
      for(let i=0;i<this.simiMvs.length;i++){
        let duration = this.simiMvs[i].duration;
          let min = parseInt(duration/1000/60);
          let sec = parseInt(duration/1000%60);
          min = min < 10 ? '0' + min : min;
          sec = sec < 10 ? '0' + sec : sec;
          //console.log(min+"|"+sec);
          this.simiMvs[i].duration = `${min}:${sec}`;
          if(this.simiMvs[i].playCount>100000){
          this.simiMvs[i].playCount = parseInt(this.simiMvs[i].playCount/10000)+'万';
      }
      }
    })
    //mv信息
    axios({
      method:'get',
      url: 'https://autumnfish.cn/mv/detail',
      params: {
        mvid:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.mvinfo = res.data.data;
      //获取歌手信息
      axios({
        method:'get',
        url: 'https://autumnfish.cn/artists',
        params: {
          id:this.mvinfo.artists[0].id
        }
      }).then(res=>{
        //console.log(res)
        this.icon = res.data.artist.picUrl
      })
    })
    //获取评论的数据
    axios({
      method:'get',
      url: 'https://autumnfish.cn/comment/mv',
      params: {
        id:this.$route.query.id,
        limit:10,
        offset:(this.page - 1)*10
      }
    }).then(res=>{
      
      this.hotComments = res.data.hotComments;
      this.comments = res.data.comments;
      //保存条数
      this.total = res.data.total;
    })
  },
  methods: {
    handleCurrentChange(val) {
      //console.log(`当前页: ${val}`);
      this.page = val;
      //获取评论的数据
    axios({
      method:'get',
      url: 'https://autumnfish.cn/comment/mv',
      params: {
        id:this.$route.query.id,
        limit:10,
        offset:(this.page - 1)*10
      }
    }).then(res=>{
      
      this.hotComments = res.data.hotComments;
      this.comments = res.data.comments;
      //保存条数
      this.total = res.data.total;
      //console.log(this.total)
    })
  },
      toMV(id){
      this.$router.push(`/mv?id=${id}`)
          //获取mv地址
    axios({
      url:'https://autumnfish.cn/mv/url',
      method:'get',
      params:{
        id:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.mvUrl = res.data.data.url;
    })
        //获取相关mv
    axios({
      url:'https://autumnfish.cn/simi/mv',
      method:'get',
      params:{
        mvid:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.simiMvs = res.data.mvs;
      for(let i=0;i<this.simiMvs.length;i++){
        let duration = this.simiMvs[i].duration;
          let min = parseInt(duration/1000/60);
          let sec = parseInt(duration/1000%60);
          min = min < 10 ? '0' + min : min;
          sec = sec < 10 ? '0' + sec : sec;
          //console.log(min+"|"+sec);
          this.simiMvs[i].duration = `${min}:${sec}`;
          if(this.simiMvs[i].playCount>100000){
          this.simiMvs[i].playCount = parseInt(this.simiMvs[i].playCount/10000)+'万';
      }
      }
    })
        //mv信息
    axios({
      method:'get',
      url: 'https://autumnfish.cn/mv/detail',
      params: {
        mvid:this.$route.query.id
      }
    }).then(res=>{
      //console.log(res)
      this.mvinfo = res.data.data;
      //获取歌手信息
      axios({
        method:'get',
        url: 'https://autumnfish.cn/artists',
        params: {
          id:this.mvinfo.artists[0].id
        }
      }).then(res=>{
        //console.log(res)
        this.icon = res.data.artist.picUrl
      })
    })
        //获取评论的数据
    axios({
      method:'get',
      url: 'https://autumnfish.cn/comment/mv',
      params: {
        id:this.$route.query.id,
        limit:10,
        offset:(this.page - 1)*10
      }
    }).then(res=>{
      
      this.hotComments = res.data.hotComments;
      this.comments = res.data.comments;
      //保存条数
      this.total = res.data.total;
    })
    }
  }
};
</script>

<style></style>
