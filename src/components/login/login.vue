<template>
  <div class="login-wrap">
    <el-form
      class="login-form"
      label-position="top"
      label-width="80px"
      :model="formdata"
    >
      <h2>用户登录</h2>
      <el-form-item label="用户名">
        <el-input v-model="formdata.username"></el-input>
      </el-form-item>
      <el-form-item label="密码">
        <el-input v-model="formdata.password"></el-input>
      </el-form-item>
      <el-button @click.prevent="handlelogin()" class="login-btn" type="primary"
        >登录</el-button
      >
    </el-form>
  </div>
</template>
<script>
export default {
  data () {
    return {
      formdata: {
        username: '',
        password: ''
      }
    }
  },
  methods: {
    // 登录请求
    async handlelogin () {
      // 让异步操作的代码 看起来像同步代码
      // ES7 async+await
      const res = await this.$http.post('login', this.formdata)
      // 对象解构赋值
      console.log(res)
      const {
        data,
        meta: { msg, status }
      } = res.data

      if (status === 200) {
        // 登录成功
        // 0、保存token
        localStorage.setItem('token', data.token)
        // 1、跳转home
        this.$router.push({ name: 'home' })
        // 2、提示成功
        this.$message.success(msg)
      } else {
        // 不成功
        // 提示信息
        this.$message.warning(msg)
      }
    }

    // handlelogin () {
    //   // 让异步操作的代码 看起来像同步代码
    //   this.$http.post('login', this.formdata)
    //     .then(res => {
    //       // 对象解构赋值
    //       const {
    //         data,
    //         meta: { msg, status }
    //       } = res.data

    //       if (status === 200) {
    //         // 登录成功
    //         // 1、跳转home
    //         this.$router.push({ name: 'home'})
    //         // 2、提示成功
    //         this.$message.success(msg)
    //       } else {
    //         // 不成功
    //         // 提示信息
    //         this.$message.warning(msg)
    //       }
    //     })
    // }
  }
}
</script>
<style scoped>
.login-wrap {
  height: 100%;
  background-color: #324152;
  /* 使用弹性盒子在页面中垂直水平居中 */
  display: flex;
  justify-content: center;
  align-items: center;
}
.login-wrap .login-form {
  width: 400px;
  background-color: #fff;
  border-radius: 5px;
  padding: 30px;
}
.login-wrap .login-btn {
  width: 100%;
}
</style>
