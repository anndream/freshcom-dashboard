<template>
<div id="login" class="page-wrapper">
  <div class="center">

    <h1>Freshcom</h1>

    <el-card shadow="never">
      <div slot="header" class="text-center">
        Please sign in to continue
      </div>

      <el-form @submit.native.prevent="attemptLogin(form)" label-width="80px" size="small">
        <el-form-item label="Email">
          <el-input v-model="form.username" id="username"></el-input>
        </el-form-item>

        <el-form-item label="Password">
          <el-input v-model="form.password" id="password" type="password"></el-input>
        </el-form-item>

        <el-form-item class="thin">
          <el-checkbox v-model="form.rememberMe">Remember me</el-checkbox>
          <router-link :to="{ name: 'ForgotPassword'} " class="pull-right">Forgot your password?</router-link>
        </el-form-item>

        <el-form-item>
          <el-button :loading="isSubmitting" type="primary" native-type="submit" size="medium">Sign in to your account</el-button>
        </el-form-item>

      </el-form>
    </el-card>

    <p class="text-center">Don't have an account? <router-link :to="{ name: 'Register' }" >Sign up</router-link></p>
  </div>
</div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      isSubmitting: false,
      form: {
        username: '',
        password: '',
        rememberMe: false
      }
    }
  },
  created () {
    if (this.isLoggedIn) {
      this.$store.dispatch('pushRoute', { name: 'Home' })
    }
  },
  computed: {
    isLoggedIn () {
      return !!this.$store.state.session.user
    }
  },
  methods: {
    attemptLogin (form) {
      this.isSubmitting = true
      this.$store.dispatch('session/create', form).then(() => {
        this.isSubmitting = false

        this.$message({
          showClose: true,
          message: `You are now logged in, welcome back!`,
          type: 'success'
        })

        this.$store.dispatch('pushRoute', { name: 'Home' })
      }).catch(response => {
        this.$message({
          showClose: true,
          message: response.error_description,
          type: 'error'
        })

        this.isSubmitting = false
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.page-wrapper#login {
  margin-top: 50px;
}

h1 {
  text-align: center;
}

.thin {
  margin-top: -10px;
  margin-bottom: 10px;
}

.center {
  margin: auto;
  width: 420px;
}
</style>
