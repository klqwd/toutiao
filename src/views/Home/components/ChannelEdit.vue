<template>
  <div class="channel-edit">
    <!-- 我的频道 -->
    <!-- 标题 -->
    <van-cell title="我的频道">
      <van-button
        class="btn"
        size="min"
        round
        style="color: red; borde-color: red"
        @click="isEdit = !isEdit"
      >
        {{ isEdit ? "完成" : "编辑" }}
      </van-button>
    </van-cell>
    <!-- 频道 -->
    <div class="my-pannel">
      <van-grid gutter="10" :border="false">
        <van-grid-item
          v-for="(item, index) in myChannels"
          :class="{ active: item.name === '推荐' }"
          :key="item.id"
          :text="item.name"
          :icon="isEdit && item.name !== '推荐' ? 'cross' : ''"
          @click="handleMyChannel(item, index)"
        >
        </van-grid-item>
      </van-grid>
    </div>

    <!-- 推荐频道 -->
    <!-- 标题 -->
    <van-cell title="推荐频道"></van-cell>
    <!-- 频道 -->
    <div class="recommend-pannel">
      <van-grid gutter="10" :border="false">
        <van-grid-item
          v-for="item in recommendChannels"
          :key="item.id"
          :text="item.name"
          icon="plus"
          @click="$emit('add-channel', item)"
        >
        </van-grid-item>
      </van-grid>
    </div>
  </div>
</template>

<script>
import { getAllChannelAPI } from "@/api";
export default {
  data() {
    return {
      isEdit: false,
      allChannels: [],
    };
  },
  props: {
    myChannels: Array,
  },
  created() {
    this.getAllChannels();
  },
  methods: {
    async getAllChannels() {
      const { data } = await getAllChannelAPI();
      this.allChannels = data.data.channels;
      console.log(this.allChannels);
    },
    handleMyChannel({ name, id }, index) {
      if (this.isEdit && name !== "推荐") {
        this.$emit("del-channel", id);
      } else {
        this.$emit("change-active", index);
        console.log(index);
      }
    },
  },
  computed: {
    recommendChannels() {
      return this.allChannels.filter((allChannelItem) => {
        return !this.myChannels.find(
          (myChannelItem) => myChannelItem.id === allChannelItem.id
        );
      });
    },
  },
};
</script>

<style scoped lang="less">
.channel-edit {
  padding-top: 92px;
}
:deep(.btn) {
  width: 100px;
  font-size: 25px;
}
.van-cell__title {
  display: flex;
  align-items: center;
}
:deep(.van-grid-item__content) {
  background-color: #f4f5f6;
}
// 我的频道
.my-pannel {
  // 编辑按钮居中
  .van-cell__value {
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }

  // 关闭按钮样式
  :deep(.van-grid-item__content) {
    position: relative;
    .van-grid-item__content {
      position: absolute;
      top: 0;
      right: 0;
    }
  }
}
:deep(.van-icon-cross) {
  position: absolute;
  top: 0;
  right: 0;
  font-size: 25px;
  transform: translate(20%, -35%);
  border: 0.02667rem solid #000;
  border-radius: 50%;
  text-align: center;
  line-height: 0.32rem;
}
// 推荐频道
.recommend-pannel {
  // 推荐频道加号样式
  :deep(.van-grid-item__content) {
    flex-direction: row;
    position: relative;

    .van-grid-item__icon {
      font-size: 0.5rem;
      position: absolute;
      top: -15px;
      right: -15px;
    }

    .van-grid-item__text {
      margin-top: 0;
    }
  }
}
:deep(.active) {
  .van-grid-item__text {
    color: red;
  }
}
</style>
