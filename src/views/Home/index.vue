<template>
  <div>
    <!-- 搜索栏 -->

    <van-nav-bar class="navbar">
      <template #title
        ><van-button
          icon="search"
          size="small"
          round
          block
          @click="$router.push('/search')"
        >
          搜索</van-button
        >
      </template>
    </van-nav-bar>

    <!-- 频道及文章展示 标签栏 -->
    <van-tabs v-model="active" swipeable>
      <van-tab v-for="item in channels" :key="item.id" :title="item.name">
        <article-list :id="item.id"></article-list>
      </van-tab>
      <span class="toutiao toutiao-gengduo" @click="isShow = true"></span>
    </van-tabs>

    <!-- 弹出层 -->
    <van-popup
      v-model="isShow"
      closeable
      close-icon-position="top-left"
      position="bottom"
      :style="{ height: '100%' }"
    >
      <Channel-edit
        v-if="isShow"
        :myChannels="channels"
        @change-active="[(isShow = false), (active = $event)]"
        @del-channel="delChannel"
        @add-channel="addChannel"
      ></Channel-edit>
    </van-popup>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from "vuex";
import { getChannelAPI, delChannelAPI, addChannelAPI } from "@/api/channel";
import ChannelEdit from "./components/ChannelEdit";
import ArticleList from "./components/ArticleList";
export default {
  components: {
    ArticleList,
    ChannelEdit,
  },
  data() {
    return {
      active: 0,
      channels: [],
      isShow: false,
    };
  },
  created() {
    this.initChannles();
  },
  //1.??==>相当于||，常用于语句
  //2.?.==>可选链操作符,?前面是undifined,那么不会往后取值
  methods: {
    ...mapMutations(["SET_MY_CHANNELS"]),
    initChannles() {
      if (this.isLogin) {
        this.getChannel();
      } else {
        const myChannels = this.$store.state.myChannels;
        if (myChannels.length === 0) {
          this.getChannel();
        } else {
          this.channels = myChannels;
        }
      }
    },

    async getChannel() {
      try {
        const { data } = await getChannelAPI();
        this.channels = data.data.channels;
      } catch (error) {
        if (!error.response) {
          throw error;
        } else {
          const status = error.response.status;
          status === 507 && this.$toast.fail("服务端异常");
        }
      }
    },
    async delChannel(id) {
      try {
        const newChannels = this.channels.filter((item) => item.id !== id);
        if (this.isLogin) {
          await delChanneAPI(id);
        } else {
          //把我的频道存到本地存储
          this.SET_MY_CHANNELS([newChannels]);
        }
        this.channels = newChannels;
        this.$toast.success("删除成功");
      } catch (error) {
        if (error.response && error.response.status === 401) {
          this.$toast.fail("请登录再删除！");
        } else {
          throw error;
        }
      }
    },
    async addChannel(channel) {
      try {
        if (this.isLogin) {
          await addChannelAPI(channel.id, this.channels.length);
        } else {
          this.SET_MY_CHANNELS([...this.channels, channel]);
        }
        this.channels.push(channel);
        this.$toast.success("频道添加成功！");
      } catch (error) {
        if (error.response && error.response.status === 401) {
          this.$toast.fail("请登录再添加");
        } else {
          throw error;
        }
      }
    },
  },
  computed: {
    ...mapGetters(["isLogin"]),
  },
};
</script>

<style scoped lang="less">
.navbar {
  background-color: #3296fa;
  /*
  // inherit 继承
  // unset 不设置 */
  :deep(.van-nav-bar__title) {
    max-width: unset;
  }
  .van-button--default {
    background-color: #5babfb;
    border: 0;
    color: #fff;
    font-size: 30px;
  }

  .van-icon {
    color: #fff;
  }
  .van-button--block {
    width: 7.4rem;
  }
}
/* tabs导航条样式 */
:deep(.van-tabs__wrap) {
  padding-right: 66px;

  .van-tabs__nav {
    padding-left: 0;
    padding-right: 0;

    /* tab标签 */
    .van-tab {
      border: 1px solid #eee;
      width: 200px;
      height: 82px;
    }

    /* tab标签下划线 */
    .van-tabs__line {
      width: 31px;
      height: 6px;
      background-color: #3296fa;
      bottom: 40px;
    }
  }
}

/* 字体图标 */
.toutiao-gengduo {
  position: absolute;
  top: 0;
  right: 0;
  width: 66px;
  height: 82px;
  font-size: 40px;
  line-height: 82px;
  text-align: center;
  opacity: 0.6;
  border-bottom: 1px solid #eee;
  background: url("@/assets/images/favicon.ico") cover;
  &::after {
    content: "";
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    height: 70%;
    width: 1px;
    background-color: blue;
    background: url("@/assets/images/favicon.ico");
  }
}
</style>
