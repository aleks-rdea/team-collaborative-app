<template>
<div class="ui middle aligned center aligned grid login_container">
  <div class="column">
    <h2 class="ui orange header">
      <div class="content">
        Log-in to your account
      </div>
    </h2>
    <form class="ui large form" :class="{'error' : hasErrors}">
      <div class="ui stacked segment">
        <div class="field">
          <div class="ui left icon input">
            <i class="user icon"></i>
            <input type="email" name="email" placeholder="Email" v-model.trim="email">
          </div>
        </div>
        <div class="field">
          <div class="ui left icon input">
            <i class="lock icon"></i>
            <input type="password" name="password" placeholder="Password" v-model.trim="password">
          </div>
        </div>
        <div class="ui fluid large orange button" @click.prevent="login" :class="{'loading': isLoading}">Log in</div>
      </div>

      <div class="ui error message" v-if="hasErrors">
        <p :v-for="error in errors" >{{ error }}</p>
      </div>
    </form>

    <div class="ui message">
      Not yet registered ? <router-link to="/register">Register</router-link>
    </div>
  </div>
</div>
</template>

<script>
    import firebase from 'firebase'

    export default {
        name: 'login',
        data() {
            return {
                email: '',
                password: '',
                errors: [],
                isLoading: false
            }
        },
        computed: {
          hasErrors() {
            return this.errors.length > 0
          }
        },
        methods: {
            login() {
                console.log("login")
                this.errors = []
                if(this.isFormValid()){
                    this.isLoading = true
                    firebase.auth().createUserWithEmailAndPassword(this.email, this.password).then(user => {
                        this.$store.dispatch('setUser', user)
                        this.$router.push('/')
                    }).catch(error => {
                        this.errors.push(error.message)
                        this.isLoading = false
                    })
                }
            },
            isFormValid(){
                if(this.email.length > 0 && this.password.length > 0){
                    return true
                }
                return false
            }
        }
    }
</script>

<style scoped>
    .login_container{
        margin-top: 40px;
    }
    .column{
        max-width: 450px;
    }
</style>
