<template>
  <div id="box">
    <!-- 1. 프로필 -->
    <b-jumbotron>
      <!-- head, lead 속성 사용가능! -->
      <b-row class="mb-5">
        <b-col cols="4">
          <span @click="toBadge" style="cursor: pointer;"
            ><b-avatar
              size="8rem"
              variant="info"
              :src="require(`@/assets/app/badge/${this.badge}.png`)"
            ></b-avatar
          ></span>
        </b-col>
        <b-col cols="6" align-self="center">
          <b-row align-h="center" class="mb-4">
            <b-col>
              <h2 style="font-family: 'Nanum Pen Script', cursive; display: inline;">
                {{ user.nickname }}
              </h2>
              <b-icon
                v-if="userId === user.userId"
                icon="gear-fill"
                font-scale="1.5"
                style="cursor: pointer;"
                type="bold"
                @click="toAccountDetail"
              ></b-icon>
              
            </b-col>
          </b-row>
          <b-row align-h="center">
            <b-col class="mx-0 px-0"
              ><b-button disabled style="background-color: #695549;"
                >게시물: {{ posts.length }}</b-button
              ></b-col
            >
            <b-col class="mx-0 px-0"
              ><b-button disabled style="background-color: #695549;"
                >리뷰: {{ reviews.length }}</b-button
              ></b-col
            >
            <b-col class="mx-0 px-0"
              ><b-button disabled style="background-color: #695549;"
                >그룹: {{ groups.length }}</b-button
              ></b-col
            >
          </b-row>
        </b-col>
      </b-row>
    </b-jumbotron>

    <!-- 2. 탭 -->
    <b-tabs
      active-nav-item-class="font-weight-bold text-uppercase text-dark"
      content-class="mt-3"
      justified
    >
      <b-tab title="글" active>
        <div v-for="(post, idx) in posts" :key="idx">
          <PostBlockMy :post="post" />
        </div>
        <br />
        <br />
        <EndBlock v-on:more="getMorePosts" />
        <span v-if="this.postCount > this.posts.length">더보기</span>
      </b-tab>
      <b-tab title="리뷰">
        <div v-for="(item, index) in reviews" :key="index">
          <ReviewBlock :review="item" :user="user" />
        </div>
      </b-tab>
      <b-tab title="태그">
        <TagBox :reviews="reviews" :userposts="posts" :user="user" />
      </b-tab>
      <b-tab title="그룹">
        <GroupBox :userId="user.userId"/>
      </b-tab>
    </b-tabs>

    <br />
    <br />
    <br />
    <Button />
  </div>
</template>

<script>
import axios from 'axios';
import Button from '@/components/story/Button';
import EndBlock from '@/components/story/EndBlock';
import GroupBox from '@/components/story/GroupBox';
import PostBlockMy from '@/components/story/PostBlockMy';
import ReviewBlock from '@/components/story/ReviewBlock';
import TagBox from '@/components/story/TagBox';

const SERVER_URL = process.env.VUE_APP_SERVER_URL;

export default {
  name: 'MyFeed',
  components: {
    Button,
    EndBlock,
    GroupBox,
    PostBlockMy,
    ReviewBlock,
    TagBox,
  },
  data() {
    return {
      // 사용자 정보
      user: {
        userId: this.$route.params.userId,
        nickname: this.$route.params.nickname,
      },
      posts: [],
      postCount: 0,
      limit: 5, //한 페이지에 노출될 게시글의 수
      offset: 0, //게시글 번호 오프셋,
      liked: false,
      reviews: [],  
      groups: [],
      badge: '',

      // 내 정보
      userId: JSON.parse(localStorage.getItem('Login-token'))['user-id'],
    };
  },
  created() {
    this.getUserPosts();
    this.getReviews();
    this.getGroups();
    const userInfo = JSON.parse(localStorage.getItem('Login-token'));
    this.badge = userInfo.user_badge;
  },
  methods: {
    getUserPosts() {
      axios
        .get(`${SERVER_URL}/userpost/user`, {
          params: {
            userId: this.user.userId,
            limit: this.limit,
            offset: this.offset,
          },
        })
        .then((response) => {
          this.posts = response.data.list;
          this.postCount = response.data.count;
        });
    },
    getReviews: function() {
      axios
        .get(`${SERVER_URL}/review/user/${this.user.userId}`)
        .then((response) => {
          this.reviews = response.data;
        })
        .catch((response) => {
          console.log(response);
        });
    },
    getGroups() {
      axios
        .get(`${SERVER_URL}/club/user/${this.user.userId}/member`)
        .then((response) => (this.groups = response.data));
    },
    getMorePosts() {
      if (this.postCount <= this.posts.length) {
        return;
      }
      this.offset += this.limit;
      axios
        .get(`${SERVER_URL}/userpost/user`, {
          params: {
            userId: this.user.userId,
            limit: this.limit,
            offset: this.offset,
          },
        })
        .then((response) => {
          this.posts.push(...response.data.list);
        });
    },
    toAccountDetail: function() {
      this.$router.push({ name: 'AccountDetail' });
    },
    toBadge: function() {
      // 본인이면 뱃지 페이지로 이동
      if (this.userId == this.user.userId) {
        this.$router.push({ name: 'Badge', params: { userId: this.user.userId } });
      } else {
        // 아니면
        alert(`${this.user.userId}님의 대표뱃지!`);
      }
    },
    toList: function() {
      this.$router.push({ name: 'GroupList' });
    },
  },
};
</script>

<style></style>
