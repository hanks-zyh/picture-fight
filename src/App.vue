<template>
  <div id="app">
    <div class="topic-list">
      <ul>
        <li
          class="topic"
          v-for="topic in topicList"
          :style="{ backgroundColor: topic.id == currentTopicId ? '#313140': '#3d3d4f'}"
          @click="getCommentList(topic.id, topicLastId)">
          <h2>{{ topic.ti }}</h2>
        </li>
      </ul>
    </div>
    <div class="comment-list">
      <div class="fav-list">
        <div v-for="comment in commentList"
         :style="{ flexGrow: comment.w * 100 / comment.h,
         flexBasis: (comment.w * 120 / comment.h) + 'px'}">
           <i :style=" { paddingBottom: (comment.h / comment.w * 100) +'%' }"></i>
           <img :src="comment.src"/>
         </div>
        </div>
    </div>
  </div>
</template>

<script>
var Vue = require('vue');
var vueResource = require('vue-resource');
Vue.use(vueResource);
export default {
  data () {
    return {
      topicLastId: 0,
      commentLastId: 0,
      currentTopicId: 0,
      commentNum: 0,
      topicList: [],
      commentList:[]
    }
  },
  created () {
    this.getTopicList(this.topicLastId)
    this.getCommentList(265479,this.commentLastId)
    this.init()
  },
  methods: {
    getTopicList (lastId) {
      var url = 'http://bookshelf.leanapp.cn/fight/topics'
      if(lastId!=0){
        url += '?lastId=' + lastId;
      }
      console.log(url);
      this.$http.get(url).then((response) => {
        // console.log(response.body);
        var json = JSON.parse(response.body);
        console.log('t:'+this.topicLastId+","+json.last_id);
        this.topicLastId = json.last_id;
        console.log('tf:'+this.topicLastId);
        var list = json.list;
        for (var i = 0; i < list.length; i++) {
          var item = list[i];
          this.topicList.push(item)
        }
      }, (error) => {
        console.error(error);
      })
    },
    getCommentList (topicId, lastId) {
      if (topicId != this.currentTopicId) {
        this.currentTopicId = topicId
        this.commentList.length = 0
        this.commentLastId = 0
        this.commentNum = 0
        lastId = 0
      }
      var url = 'http://bookshelf.leanapp.cn/fight/'+ this.currentTopicId +'/comments'
      if(lastId!=0){
        url += '?lastId=' + lastId;
      }
      this.$http.get(url).then((response) => {
        var json = JSON.parse(response.body);
        this.commentLastId = json.last_id;
        if(json.post_info && json.post_info.comment_num){
          this.commentNum = json.post_info.comment_num;
        }
        var list = json.comment_list;
        if(list){
          for (var i = 0; i < list.length; i++) {
            var item = list[i]
            if(item.emotions_size.length <= 0){
              continue;
            }
            item.w = item.emotions_size[0].w
            item.h = item.emotions_size[0].h
            item.src = item.emotions[0]
            this.commentList.push(item)
          }
          if(this.commentList.length < this.commentNum && this.commentList.length < 100){
            this.getCommentList(this.currentTopicId, this.commentLastId)
          }
        }
      }, (error) => {
        console.error(error);
      })
    },

    topicLoadMore () {
        this.getTopicList(this.topicLastId)
    },

    commentLoadMore () {
      this.getCommentList(this.currentTopicId, this.commentLastId)
    },

    init () {

      var that  = this;
      window.onload = function () {
        // 检测滑动到底部
       var topicList = document.getElementsByClassName("topic-list")[0];
       var commentList = document.getElementsByClassName("comment-list")[0];

       if (topicList) {
         topicList.style.height = window.innerHeight + 'px';
         topicList.onscroll = function () {
          //  console.log(topicList.scrollTop+',' + topicList.clientHeight + ',' + topicList.scrollHeight);
           if (Math.abs(topicList.scrollTop + topicList.clientHeight - topicList.scrollHeight) < 10) {
               console.log("topicList 到达底部");
               that.topicLoadMore()
           }
         }
       }

       if (commentList) {
         commentList.style.height = window.innerHeight + 'px';
         commentList.onscroll = function () {
          //  console.log(commentList.scrollTop+',' + commentList.clientHeight + ',' + commentList.scrollHeight);
           if (Math.abs(commentList.scrollTop + commentList.clientHeight - commentList.scrollHeight) < 10) {
               console.log("commentList 到达底部");
               that.commentLoadMore()
           }
         }
       }
      }
      window.onresize = function(event) {
        var topicList = document.getElementsByClassName("topic-list")[0];
        var commentList = document.getElementsByClassName("comment-list")[0];
        if (topicList) {
          topicList.style.height = window.innerHeight + 'px';
        }
        if (commentList) {
          commentList.style.height = window.innerHeight + 'px';
        }
      };
    }

  }
}
</script>

<style>

html,body {
  font-family: Helvetica, sans-serif;
  height: 100%;
  font-size: 10px;
  background-color: #23222d;
}
ul{
  margin: 0;
  padding: 0;
}
li{
  list-style: none;

}
#app{
  display: flex;
  height: 100%;
}
.topic-list{
  width: 30%;
  overflow-y: auto;
}
.comment-list{
  width: 70%;
  overflow-y: auto;
}
.topic{
  border-bottom: 0.1em dashed #444458;
  cursor: pointer;
  color: #ffffff;
  font-size: 1em;
  padding: 1em 2em;
  transform: background 0.3s;
}
.topic:hover{
  background: #313140
}
.topic-selected{
  background-color: #e1e1e1;
}
.fav-list{
  display: flex;
  flex-wrap: wrap;
  margin: 2px;
  overflow-y: auto;
}
/* .fav-list::after{
  content: '';
  flex-grow: 999999999;
  order: 999999999;
} */
.fav-list > div{
  margin: 2px;
  background-color: violet;
  position: relative;
}
.fav-list > div > i {
  display: block;
  background-color: lightblue;
  /* display: none; */
}
.fav-list > div > img {
  position: absolute;
  vertical-align: bottom;
  top: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.fav-list > .placeholder{
  flex-grow: 100;
  flex-basis: 480px;
  height: 0;
  margin: 0;
}
</style>
