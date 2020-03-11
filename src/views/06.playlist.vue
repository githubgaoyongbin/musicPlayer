<template>
  <div
    class="infinite-list"
    id="main-container"
    v-infinite-scroll="load"
    v-loading="loading"
    element-loading-background="rgba(0, 0, 0, 0.5)"
  >
    <div class="playlist-container" id="playlist-container">
      <div class="top-wrap">
        <div class="img-wrap">
          <!-- 封面 -->
          <img :src="playlist.coverImgUrl" alt />
        </div>
        <div class="info-wrap">
          <!-- 名字 -->
          <p class="title">{{ playlist.name }}</p>
          <div class="author-wrap">
            <!-- 头像 -->
            <img class="avatar" :src="playlist.creator.avatarUrl" alt />
            <span class="name">{{ playlist.creator.nickname }}/</span>
            <span class="time">{{ playlist.createTime }} 创建</span>
          </div>
          <div class="play-wrap">
            <span class="iconfont icon-circle-play"></span>
            <span class="text">播放全部</span>
          </div>
          <div class="tag-wrap">
            <span class="title">标签:</span>
            <!-- 分类标签 -->
            <ul>
              <li v-for="(item, index) in playlist.tags" :key="index">{{ item }}</li>
            </ul>
          </div>
          <div class="desc-wrap">
            <span class="title">简介:</span>
            <!-- 简介 -->
            <span class="desc">{{ playlist.description }}</span>
          </div>
        </div>
      </div>
      <el-tabs v-model="activeIndex" id="my-tab">
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
              <tr class="el-table__row" v-for="(item,index) in tracks" :key="index">
                <td>{{index+1}}</td>
                <td>
                  <div class="img-wrap">
                    <img :src="item.al.picUrl" alt />
                    <span class="iconfont icon-play"></span>
                  </div>
                </td>
                <td>
                  <div class="song-wrap">
                    <div class="name-wrap">
                      <span>{{item.name}}</span>
                      <span class="iconfont icon-mv"></span>
                    </div>
                    <span>{{item.al.name}}</span>
                  </div>
                </td>
                <td>{{item.ar[0].name}}</td>
                <td>{{item.al.name}}</td>
                <td>{{item.publishTime}}</td>
              </tr>
            </tbody>
          </table>
        </el-tab-pane>
        <el-tab-pane :label=" `歌曲条数 (${total}) ` " name="2">
          <!-- 精彩评论 -->
          <div class="comment-wrap" v-if="hotComments.length!=0">
            <p class="title">
              精彩评论
              <span class="number">({{ hotCount }})</span>
            </p>
            <div class="comments-wrap">
              <!-- 评论 -->
              <div v-for="(item, index) in hotComments" :key="index" class="item">
                <div class="icon-wrap">
                  <!-- 头像 -->
                  <img :src="item.user.avatarUrl" alt />
                </div>
                <div class="content-wrap">
                  <div class="content">
                    <!-- 昵称 -->
                    <span class="name">{{ item.user.nickname }}:</span>
                    <span class="comment">{{ item.content }}</span>
                  </div>
                  <!-- 评论的回复 -->
                  <div class="re-content" v-if="item.beReplied.length != 0">
                    <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                    <span class="comment">{{ item.beReplied[0].content }}</span>
                  </div>
                  <div class="date">2020-02-12 17:26:11</div>
                </div>
              </div>
            </div>
          </div>
          <!-- 最新评论 -->
          <div class="comment-wrap">
            <p class="title">
              最新评论
              <span class="number">( {{total}} )</span>
            </p>
            <div class="comments-wrap">
              <div class="item" v-for="(item,index) in comments" :key="index">
                <div class="icon-wrap">
                  <img :src="item.user.avatarUrl" alt />
                </div>
                <div class="content-wrap">
                  <div class="content">
                    <span class="name">{{item.user.nickname}}：</span>
                    <span class="comment">{{ item.content }}</span>
                  </div>
                  <div class="re-content" v-if="item.beReplied.length!=0">
                    <span class="name">{{ item.beReplied[0].user.nickname }}：</span>
                    <span class="comment">{{ item.beReplied[0].content }}</span>
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
          ></el-pagination>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
// 导入 axios
import axios from "axios";
import moment from "moment";
import _ from "lodash";
export default {
  name: "playlist",
  data() {
    return {
      activeIndex: "1",
      // 总条数
      total: 0,
      // 页码
      page: 1,
      // 歌单详情数据
      tracks: [],
      playlist: {},
      // 热门评论
      hotComments: [],
      // 热门评论的个数
      hotCount: 0,
      // 普通评论
      comments: [],
      count: 1,
      loading: false,
      isTrue:false
    };
  },
  created() {
    // 获取歌单详情
    axios({
      url: "https://autumnfish.cn/playlist/detail",
      method: "get",
      params: {
        id: this.$route.query.q
      }
    }).then(res => {
      // console.log(res)
      this.playlist = res.data.playlist;
      // this.tracks = res.data.playlist
      this.playlist.createTime = moment(this.playlist.createTime).format(
        "llll"
      );
      this.playlist.tracks &&
        this.playlist.tracks.map(item => {
          item.publishTime = moment(item.publishTime).format("YYYY-MM-DD");
        });
      this.tracks = _.slice(this.playlist.tracks, 0, 10);
    });
    // 获取 评论
    axios({
      url: "https://autumnfish.cn/comment/hot",
      method: "get",
      params: {
        id: this.$route.query.q,
        // 传递类型
        type: 2
      }
    }).then(res => {
      // console.log(res)
      this.hotComments = res.data.hotComments || [];
      // console.log(this.hotComments,"this.hotComments")
      // 保存个数
      this.hotCount = res.data.total;
    });

    // 获取 最新评论
    axios({
      url: "https://autumnfish.cn/comment/playlist",
      method: "get",
      params: {
        id: this.$route.query.q,
        limit: 10,
        offset: 0
      }
    }).then(res => {
      // console.log(res)
      // 总个数
      this.total = res.data.total;
      // 评论数据
      this.comments = res.data.comments;
    });
  },
  mounted() {
    // let that = this;
    // document
    //   .getElementById("main")
    //   .addEventListener("scroll", this.scrollHandle);
  },
  methods: {
    // 数据下拉加载
    load() {
      if (this.tracks.length === this.playlist.tracks.length || this.isTrue || this.activeIndex == 2) {
        return;
      }
      this.isTrue = true
      this.count++;
      this.loading = true;
      this.tracks = _.slice(this.playlist.tracks, 0, this.count * 10);
      setTimeout(() => {
        this.loading = false;
        this.isTrue = false;
      }, 1000);
    },
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      // 保存页码
      this.page = val;
      // 重新获取数据
      axios({
        url: "https://autumnfish.cn/comment/playlist",
        method: "get",
        params: {
          id: this.$route.query.q,
          limit: 10,
          // 根据页码计算
          offset: (this.page - 1) * 10
        }
      }).then(res => {
        // console.log(res)
        // 总个数
        this.total = res.data.total;
        // 评论数据
        this.comments = res.data.comments;
      });
    }
  }
};
</script>

<style>
#main-container {
  padding: 10px, 20px;
  max-width: 100%;
  /* overflow-y: scroll; */
}
.playlist-container {
  max-width: 1100px;
}
</style>
