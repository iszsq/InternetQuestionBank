<template>
  <view class="indexBox">
    <!-- 顶部背景 -->
    <view class="topBox">
      <!-- 头像 -->
      <image
        @click="handleClickUser"
        v-if="isLogin"
        class="headPicture"
        :src="user.avatarUrl"
        mode="scaleToFill"
      ></image>
      <image
        @click="handleClickUser"
        v-else
        class="headPicture"
        src="../../static/index/user.png"
      ></image>
      <!-- 欢迎语和昵称 -->
      <view
        class="hello"
        :style="{
          opacity: isLogin ? 1 : 0,
        }"
        >Hello,</view
      >
      <!-- 如果未登录显示未登录 -->
      <button
        hover-class="none"
        v-if="!isLogin"
        class="getUserButton"
        @click="handleClickUser"
      >
        <view class="nickname">未登录</view>
      </button>
      <view v-else class="nickname">{{ user.nickName }}!</view>
      <!-- 搜索框 -->
      <view class="search">
        <input
          maxlength="15"
          placeholder="热点功能，后续版本迭代..."
          placeholder-style="color: #C3C5D4;"
          type="text"
        />
        <image class="icon" src="../../static/index/search.png"></image>
      </view>
    </view>
    <!-- banner -->
    <view class="banner">
      <image
        src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-c7e81452-9d28-4486-bedc-5dbf7c8386a5/d9120b07-0203-4b86-93ba-72fdcd2df843.png"
      />
    </view>
    <!-- 专区和动态 -->
    <view class="menu">
      <!-- 专区 -->
      <view
        class="menumain question"
        @click="handleUrl('/pages/question/index')"
      >
        <view class="title">题库专区</view>
        <view class="content"> 最近的[前端练习题]有更新 </view>
      </view>
      <!-- 动态 -->
      <view class="menumain dynamic" @click="handleUrl('/pages/common/noRelease')">
        <view class="title">动态</view>
        <view class="content"> 后续版本迭代，尽情期待~ </view>
        <!-- 头像组 -->
        <view class="headPictureList">
          <image
            :style="{ zIndex: index, left: index * 15 + 'px' }"
            :key="index"
            v-for="(i, index) in 3"
            class="headPicture"
            src="https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1441836571,2166773131&fm=26&gp=0.jpg"
            mode="scaleToFill"
          ></image>
        </view>
      </view>
    </view>
    <!-- 社区 -->
    <view class="community" @click="handleUrl('/pages/common/noRelease')">
      <view class="title">社区</view>
      <view class="content">后续版本迭代，尽情期待~</view>
    </view>
    <!-- 推荐文章 -->
    <view class="article" @click="handleUrl('/pages/common/noRelease')">
      <view class="title">推荐文章</view>
      <view class="articleList">
        <view class="articleListItem">
          <view class="articleListItemTitle">
            文章社区后续版本迭代，尽情期待~
          </view>
          <!-- 头像 -->
          <view class="bottom">
            <image
              class="headPicture"
              src="https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1441836571,2166773131&fm=26&gp=0.jpg"
              mode="scaleToFill"
            ></image>
            <view class="right">
              <view class="nickname"></view>
              <view class="date"></view>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script lang="ts">
import { ref } from "vue";
// api
import { loginByWechat } from "../../api/login";
export default {
  setup() {
    // 首页展示的信息，头像/昵称等
    const user = ref({
      nickName: "",
      avatarUrl: "",
    });
    // 是否登录
    const isLogin = ref(uni.getStorageSync("uni_id_token") !== "");
    if (isLogin.value) {
      user.value = uni.getStorageSync("userInfo");
    }
    const handleUrl = (path: string) => {
      uni.navigateTo({
        url: path,
      });
    };
    // 点击user和头像
    const handleClickUser = () => {
      // 判断是否登录
      if (isLogin.value) {
        const userID = uni.getStorageSync("uni_id");
        // 进入个人中心页面
        uni.navigateTo({
          url: `/pages/user/index?userID=${userID}`
        })
      } else {
        // 微信登录
        wx.getUserProfile({
          desc: '用于完善用户资料',
          success: ({ userInfo }) => {
            console.log(userInfo);
            uni.login({
              provider: "weixin",
              async success(res) {
                // 传入用户信息和code
                uni.showLoading({
                  title: "登录中...",
                  mask: true
                });
                // 这里判断是登录/还是注册，如果是注册，默认调接口绑定一个角色Normal
                const loginData = await loginByWechat(userInfo, res);
                uni.hideLoading();
                // 存储返回的token以及用户信息，id等
                if (loginData.success && loginData.data.code === 0) {
                  uni.setStorageSync("uni_id_token", loginData.data.token);
                  uni.setStorageSync("uni_id", loginData.data.uid);
                  uni.setStorageSync("userInfo", userInfo);
                  // 显示微信个人信息
                  user.value = userInfo;
                  // 已登录状态变更
                  isLogin.value = true;
                }
              },
              fail(err) {
                uni.showToast({
                  title: "微信登录失败",
                  icon: "none",
                });
              },
            });
          },
          fail: () => {
            console.log("未授权用户基本信息");
          },
        });
      }
    };
    return {
      isLogin,
      user,
      handleUrl,
      handleClickUser,
    };
  },
};
</script>

<style lang="scss" scoped>
@import "../../util/common.scss";

.indexBox {
  .topBox {
    position: relative;
    width: 100%;
    height: 524rpx;
    background: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-0vdryl5tm4n61d8ce3/4bff3ee0-3d02-11eb-b997-9918a5dda011.png)
      no-repeat center / 100%;

    .headPicture {
      width: 70rpx;
      height: 70rpx;
      border-radius: 50%;
      margin-top: calc(var(--status-bar-height) + 40rpx);
      margin-left: 48rpx;
    }
    button::after {
      border: none;
      background-color: none;
      padding: 0;
      width: 100%;
      height: 100%;
      position: static;
    }
    .getUserButton {
      width: 240rpx;
      height: 112rpx;
      background-color: transparent;
      border: none;
      margin-left: 0;
      margin-right: 0;
      padding-left: 0;
      padding-right: 0;
      margin-top: 4rpx;
      font-size: 80rpx;
      margin-left: 48rpx;
      line-height: 0;
      text-align: start;
      color: #fff;
      .nickname {
        line-height: 112rpx;
        margin-left: 0;
      }
    }

    .hello {
      color: #fff;
      font-size: 40rpx;
      margin-top: 58rpx;
      margin-left: 48rpx;
      font-weight: lighter;
    }

    .nickname {
      width: 460rpx;
      @include text-ellipsis;
      color: #fff;
      margin-top: 4rpx;
      font-size: 75rpx;
      margin-left: 48rpx;
    }

    .search {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      position: absolute;
      bottom: -50rpx;
      left: 50%;
      transform: translateX(-50%);
      width: 652rpx;
      height: 100rpx;
      background-color: #fff;
      box-shadow: 0px 4px 11px 0px rgba(38, 46, 99, 0.15);
      border-radius: 10px;

      input {
        width: calc(100% - 40rpx - 30rpx - 72rpx);
        margin-left: 40rpx;
        font-size: 28rpx;
      }

      .icon {
        width: 72rpx;
        height: 68rpx;

        &:active {
          opacity: 0.8;
        }
      }
    }
  }

  .banner {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 686rpx;
    height: 240rpx;
    border-radius: 15px;
    opacity: 0.72;
    color: #fff;
    margin: 80rpx auto 16rpx auto;
    box-shadow: 0px 4px 10px 0px rgba(76, 83, 119, 0.25);
    image {
      width: 100%;
      height: 100%;
    }
  }

  .menu {
    display: flex;
    justify-content: center;
    align-items: center;

    .menumain {
      width: 690rpx;
      height: 120rpx;

      .title {
        font-size: 32rpx;
        margin-top: 44rpx;
        margin-left: 52rpx;
        font-weight: bold;
      }

      .content {
        font-size: 24rpx;
        color: #999999;
        word-break: break-all;
        margin-left: 54rpx;
        margin-top: 4rpx;
      }
    }

    .question {
      width: 364rpx;
      height: 264rpx;
      background: url(../../static/index/indexFile.png) no-repeat center / 100%;

      .content {
        max-width: 256rpx;
      }
    }

    .dynamic {
      width: 364rpx;
      height: 264rpx;
      background: url(../../static/index/indexNews.png) no-repeat center / 100%;

      .content {
        max-width: 282rpx;
      }

      .headPictureList {
        position: relative;
        display: flex;
        justify-content: flex-start;
        align-items: center;
        margin-left: 46rpx;
        margin-top: 12rpx;
        height: 44rpx;

        .headPicture {
          position: absolute;
          width: 44rpx;
          height: 44rpx;
          border-radius: 50%;
          top: 50%;
          transform: translateY(-50%);
        }
      }
    }
  }

  .community {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    width: 690rpx;
    height: 120rpx;
    background: url(../../static/index/indexDiscuss.png) no-repeat center / 100%;
    margin: 0 auto;

    .title {
      font-size: 32rpx;
      margin-left: 30rpx;
    }

    .content {
      font-size: 24rpx;
      color: #404040;
      margin-left: 40rpx;
      color: #999999;
    }
  }

  .article {
    width: 690rpx;
    margin: 40rpx auto 28rpx auto;

    .title {
      font-size: 32rpx;
      font-weight: bold;
      color: #404040;
    }

    .articleList {
      width: 100%;

      .articleListItem {
        width: 606rpx;
        height: 190rpx;
        padding: 30rpx 42rpx 36rpx 42rpx;
        background: #ffffff;
        box-shadow: 0px 4px 11px 0px rgba(38, 46, 99, 0.15);
        border-radius: 10px;
        margin-top: 24rpx;

        .articleListItemTitle {
          font-size: 28rpx;
          word-break: break-all;
        }

        .bottom {
          display: flex;
          justify-content: flex-start;
          align-items: center;
          margin-top: 30rpx;

          .headPicture {
            width: 80rpx;
            height: 80rpx;
            border-radius: 50%;
          }

          .right {
            margin-left: 16rpx;

            .nickname {
              font-size: 24rpx;
              color: #666666;
            }

            .date {
              font-size: 20rpx;
              color: #999999;
            }
          }
        }
      }
    }
  }
}
</style>
