<template>
  <view class="login-container">
    <uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <button type="primary" class="btn-login" open-type="getUserInfo" @getuserinfo="getUserInfo">一键登录</button>
    <text class="tips-text">登录后尽享更多权益</text>
  </view>
</template>

<script>
  import { mapMutations, mapState } from 'vuex'

  export default {
    data() {
      return {

      };
    },
    computed: {
      ...mapState('m_user', ['redirectInfo'])
    },
    methods: {
      ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
      // 用户授权之后，获取用户的基本信息
      getUserInfo(e) {
        console.log(e)

        if (e.detail.errMsg === 'getUserInfo:fail auth deny') return uni.$showMsg('您取消了登录授权！')

        console.log(e.detail.userInfo)
        this.updateUserInfo(e.detail.userInfo)

        this.getToken(e.detail)
      },
      async getToken(info) {
        // 获取 code 对应的值
        const [err, res] = await uni.login().catch(err => err)

        if (err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')
        console.log(info.encryptedData)
        console.log(info.iv)
        console.log(info.rawData)
        console.log(info.signature)

        // 准备参数
        const query = {
          code: res.code,
          encryptedData: info.encryptedData,
          iv: info.iv,
          rawData: info.rawData,
          signature: info.signature
        }

        const { data: loginResult } = await uni.$http.post('/api/public/v1/users/wxlogin', query)
        console.log(loginResult.meta.status)
        // if (loginResult.meta.status !== 200) return uni.$showMsg('登录失败！')

        // 直接把 token 保存到 vuex 中
        // this.updateToken(loginResult.message.token)
        // this.updateToken("xpFfQlRy8GdHqc5I6hiTQCxR0OquUyAr+PK9vzR4PgoSUu5XNq5+lSFLoR0HOz7OoZTldrftzMYfNZwggRSWfN7FRt9BOuJSj62QHGgoeKLkfYR94smZaqBvpJpZDBPM8g6NGOTXCzACZaBCgghb3sF7vJGgN7/N2tba6+ee+ZtYf3tRsau36FL19Sl1VM0Mogdx2R33JZTDfRyFAxB0/Gf7c2q3BqkMvDjJn6jMiA+O86z0y0SZhus0M7psTmSQY6irAx0iSp6LJLKnplWBCuNjMM0F/2+jWX0w7ozTvw6LCmcdVlPCjR7qy4scf8NTFKDY1m6qpN6GeVu/O2WjQrreg/gWFSpycQ6ipKdHmdJq2MSGOty5aAS1KAfucduEWNxbcUf92KOgD2TTSpD0weXhcjN+X3v3lzTpdR04is5aotKKEqCtML0IN1Rpq+bYu3aRuyGwDs9Fyo683e+z9A==")
        this.updateToken("Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjIzLCJpYXQiOjE1NjQ3MzAwNzksImV4cCI6MTAwMTU2NDczMDA3OH0.YPt-XeLnjV-_1ITaXGY2FhxmCe4NvXuRnRB8OMCfnPo")
        this.navigateBack()
      },
      navigateBack() {
        if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
          uni.switchTab({
            url: this.redirectInfo.from,
            complete: () => {
              this.updateRedirectInfo(null)
            }
          })
        }
      }
    }
  }
</script>

<style lang="scss">
  .login-container {
    height: 750rpx;
    background-color: #F8F8F8;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    overflow: hidden;

    &::after {
      content: ' ';
      display: block;
      width: 100%;
      height: 40px;
      background-color: white;
      position: absolute;
      bottom: 0;
      left: 0;
      border-radius: 100%;
      transform: translateY(50%);
    }

    .btn-login {
      width: 90%;
      border-radius: 100px;
      margin: 15px 0;
      background-color: #C00000;
    }

    .tips-text {
      font-size: 12px;
      color: gray;
    }
  }
</style>
