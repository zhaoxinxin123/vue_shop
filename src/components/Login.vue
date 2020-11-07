<template>
  <div class="login_container">
    <div class="login_box">
      <!--        头像区域-->
      <div class="avatar_boc">
        <img src="../assets/logo.png">
      </div>
      <!--       登录表单区域 -->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="el-icon-user"></el-input>
          <!--    prop 设置为需要校验的字段名      -->
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" type="password"></el-input>
        </el-form-item>
        <el-form-item class="b——tns">
          <el-button round type="primary" size="small" @click="login">主要按钮</el-button>
          <el-button round type="info" size="small" @click="resetLoginForm">信息按钮</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        // 这是登录表单的数据对象绑定对象
        loginForm: {
          username: 'admin',
          password: '123456'

        },
        // 这是表单规则验证对象
        loginFormRules: {
          username: [
            {
              required: true,
              message: '请输入用户昵称',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 5,
              message: '长度在 3 到 5 个字符',
              trigger: 'blur'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入用户昵称',
              trigger: 'blur'
            },
            {
              min: 6,
              max: 15,
              message: '长度在 6 到 15 个字符',
              trigger: 'blur'
            }
          ]
        }
      }
    },
    methods: {
      // 点击充值按钮
      resetLoginForm () {
        this.$refs.loginFormRef.resetFields()
      },
      login () {
        this.$refs.loginFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('login', this.loginForm)
          console.log(res)
          if (res.meta.status !== 200) return this.$message.error('登录失败')
          this.$message.success('登录成功')
          // 登陆成功之后的token ，保存到客户端的sessionStorage中
          // token 直营在当前网站打开期间生效，所以将token保存在sessionStorage中
          window.sessionStorage.setItem('token', res.data.token)
          // 2.通过编程时导航跳转到后台主页   路由地址是home
          await this.$router.push('/home')
        })
      }
    },
    name: 'Login'
  }
</script>

<style lang="less" scoped>
  .login_container {
    background-color: #2b4b6b;
    height: 100%;
  }

  .login_box {
    width: 450px;
    height: 350px;
    background-color: #fff;
    border-radius: 3px;
    /*修改悬浮位置*/
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

  .avatar_boc {
    height: 130px;
    width: 130px;
    /*边框*/
    border: 1px solid #eeeeee;
    /* 设置圆角*/
    border-radius: 50%;
    padding: 10px;
    /*加阴影*/
    box-shadow: 0 0 10px #dddddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #ffffff;
    /*图片充满盒子*/

    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eeeeee;
    }
  }

  .login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 30px;
    box-sizing: border-box;
  }

  .b——tns {
    display: flex;
    justify-content: flex-end;
  }

</style>
