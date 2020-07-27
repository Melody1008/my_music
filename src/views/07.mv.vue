<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          :src="url"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <!-- 头像 -->
            <img :src="icon" alt="" />
          </div>
          <!-- 歌手名 -->
          <span class="name">{{mvInfo.artistname}}</span>
        </div>
        <div class="mv-info">
          <!-- 标题 -->
          <h2 class="title">{{mvInfo.name}}</h2>
          <span class="date">发布：{{ mvInfo.publishTime }}</span>
          <!-- 播放次数 -->
          <span class="number">播放：{{ mvInfo.playCount }} 次</span>
          <!-- 描述 -->
          <p class="desc">
            {{mvInfo.desc}}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap">
        <p class="title">精彩评论<span class="number">{{hotCount}}</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComment" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}:</span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              <div class="date">{{ item.time | dateFormat }}</div>
            </div>
          </div>
        </div>
      </div>
      <!-- 最新评论 -->
      <div class="comment-wrap">
        <p class="title">最新评论<span class="number">{{total}}</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in comments" :key="index">
            <div class="icon-wrap">
              <img :src="item.user.avatarUrl" alt="" />
            </div>
            <div class="content-wrap">
              <div class="content">
                <span class="name">{{item.user.nickname}}   :   </span>
                <span class="comment">{{item.content}}</span>
              </div>
              <div class="re-content" v-if="item.beReplied.length!=0">
                <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                <span class="comment">{{item.beReplied[0].content}}</span>
              </div>
              <div class="date">{{ item.time | dateFormat }}</div>
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
          <div class="item" v-for="(item,index) in simiMvs" :key="index" @click="toPlay(item.id)">
            <div class="img-wrap">
              <img :src="item.cover+'?param=250y150'" alt="" />
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
      total: 20,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      // mv地址
      url:'',
      // 推荐mv地址
      simiMvs:[],
        // mv的信息
      mvInfo:{},
      // 头像
      icon:'',
      // 热门评论
      hotComment:[],
      // 热门评论的地址
      hotCount:0,
      // 普通评论
      comments:[]
      // 热门评论的地址
      
    };
  },
  created(){
    this.getData()
  },
  methods: {
   handleCurrentChange(val) {
      this.page = val
      axios({
      url:'https://autumnfish.cn/comment/mv',
      method:'get',
      params:{
        id:this.$route.query.q,
        limit:10,
        // 根据页码计算
        offset:(this.page-1)*10
      }
    }).then(res=>{
      this.total = res.data.total
      this.comments = res.data.comments
    })
    },
    getData(){
      // 获取mv的播放地址
      axios({
        url:'https://autumnfish.cn/mv/url',
        method:'get',
        params:{
          // 获取url中的id
          id:this.$route.query.q
        }
      }).then(res=>{
        this.url = res.data.data.url
      })
  // 获取推荐mv的播放地址
      axios({
        url:'https://autumnfish.cn/simi/mv',
        method:'get',
        params:{
          mvid:this.$route.query.q
        }
      }).then(res=>{
        this.simiMvs = res.data.mvs
      })
      // 获取mv的信息
      axios({
        url:'https://autumnfish.cn/mv/detail',
        method:'get',
        params:{
            mvid:this.$route.query.q
        }
      }).then(res=>{
        this.mvInfo = res.data.data
        // 获取歌手信息
        axios({
          url:"https://autumnfish.cn/artists",
          method:'get',
          params:{
            id:this.mvInfo.artists[0].id
          }
        }).then(res=>{
          // 歌手的封面信息
          this.icon = res.data.artist.picUrl
        })
      })
      // 获取评论数据
      axios({
        url:"https://autumnfish.cn/comment/mv",
        method:'get',
        params:{
          id:this.$route.query.q,
          limit:10,
          offset:(this.page-1)*10
        }
      }).then(res=>{
        this.hotComment = res.data.hotComments
        this.hotCount = res.data.total
        this.comments = res.data.comments
        this.total = res.data.total
      })
    },
    toPlay(id){
      this.$router.push('/mv?q='+id),
      this.getData()
    }
  }
};
</script>

<style></style>
