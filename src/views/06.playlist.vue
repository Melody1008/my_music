<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <!-- 封面 -->
        <img :src="playlist.coverImgUrl" alt="" />
      </div>
      <div class="info-wrap">
        <!-- 名字 -->
        <p class="title">{{playlist.name}}</p>
        <div class="author-wrap">
          <!-- 头像 -->
          <!-- 神经病VUE,不加v-if="playlist.creator !== undefined" 报错 -->
          <img class="avatar" :src="playlist.creator.avatarUrl" v-if="playlist.creator !== undefined" alt="" />
          <!-- 名字 -->
          <span class="name" v-if="playlist.creator !== undefined">{{playlist.creator.nickname}}</span>
          <!-- 时间 -->
          <span class="time">{{playlist.creatorTime}}{{ playlist.createTime | dateFormat }}</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <!-- 分类标签 -->
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item,index) in playlist.tags" :key="index">{{item}}</li>
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
        <!-- 简介 -->
          <span class="desc"
            >{{playlist.description}}</span
          >
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
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
            <tr class="el-table__row" v-for="(item, index) in playlist.tracks" :key="index">
              <td>{{ index + 1 }}</td>
              <td>
                <div class="img-wrap" @click="playMusic(item.id)" >
                  <img :src="item.al.picUrl+'?param=130y130'"  alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                     <!-- 名称 -->
                    <span>{{ item.name }}</span>
                    <!-- mv图标 -->
                    <span v-if="item.mv !== 0" class="iconfont icon-mv" @click="toMv(item.mv)"></span>
                  </div>
                </div>
              </td>
              <td>{{ item.ar[0].name }}</td>
              <td>{{ item.al.name }}</td>
              <td>{{ item.dt | playTimeFormat }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane :label="`评论(${total})`" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap">
          <p class="title">精彩评论<span class="number">{{hotCount}}条</span></p>
          <div class="comments-wrap">
            <!-- 评论 -->
            <div v-for="(item,index) in hotComment" :key="index" class="item">
              <div class="icon-wrap">
                <!-- 头像 -->
                <img :src="item.user.avatarUrl" alt="" />
              </div>
              <div class="content-wrap">
                <!-- 昵称 -->
                <div class="content">
                  <span class="name">{{item.user.nickname}}:</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <!-- 评论的回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0" >
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
                <div class="re-content" v-if="item.beReplied.length!=0" >
                  <span class="name">{{item.beReplied[0].user.nickname}}:</span>
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
        >
        </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 歌单详情数据
      // tracks 歌曲列表
      playlist:{},
      // 热门评论
      hotComment:[],
      // 热门评论的地址
      hotCount:0,
      // 普通评论
      comments:[]
    };
  },
  created(){
    // 获取歌单详情
    axios({
      url:'https://autumnfish.cn/playlist/detail',
      method:'get',
      params:{
        id:this.$route.query.q,
      }
    }).then(res=>{
      this.playlist = res.data.playlist
    })
    // 获取评论
    axios({
      url:'https://autumnfish.cn/comment/hot',
      method:'get',
      params:{
        id:this.$route.query.q,
        // 传递类型参数
        type:2
      }
    }).then(res=>{
      this.hotComment = res.data.hotComments
      this.hotCount = res.data.total
    })
    // 获取最新评论
    axios({
      url:'https://autumnfish.cn/comment/playlist',
      method:'get',
      params:{
        id:this.$route.query.q,
        limit:10,
        offset:0
      }
    }).then(res=>{
      this.total = res.data.total
      this.comments = res.data.comments
    })
  
  },
  methods: {
    handleCurrentChange(val) {
      this.page = val

      axios({
      url:'https://autumnfish.cn/comment/playlist',
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
      playMusic(id) {
      axios({
        url: "https://autumnfish.cn/song/url",
        method: "get",
        params: {
          id
        }
      }).then(res => {
        let url = res.data.data[0].url;
        // 设置给父组件的播放值
        this.$parent.musicUrl = url;
      });
    }
  }
};
</script>

<style >

</style>
