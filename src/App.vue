<template>

<h1 class="error">{{ err }}</h1>

<!--Авторизация-->
<div id="login" v-if="flags.show_login" class="login">
      <h1>Логин:</h1>
      <input type="text" v-model="login.username" class="user_name" placeholder="Логин">
      <h1>Пароль:</h1>
      <input type="password" v-model="login.password" class="user_pwd" placeholder="Пароль">
      <div>
        <button @click="do_login()" class="login_click">Логин</button>
        <button @click="do_register()" class="register_click">Регистрация</button>
      </div>
</div>

<!--Регистрация-->
<div id="register" v-if="flags.show_register" class="register">
    <div class="login">
      <h1>Придумайте логин:</h1>
      <input type="text" v-model="register.username" class="user_name" placeholder="Ваш будущий логин">
    </div>

    <div class="password">
      <h1>Придумайте пароль:</h1>
      <input type="password" v-model="register.password" class="user_pwd" placeholder="Пароль(не короче 8 символов)">
    </div>

    <div class="password">
      <h1>Повторите пароль:</h1>
      <input type="password" v-model="register.password2" class="user_pwd" placeholder="Повторите пароль">
    </div>
    <button @click="registrate()" class="register_click">Регистрация</button>
    <button @click="back()" class="register_click">Назад</button>
</div>

<!--Главная-->
<div v-if="flags.show_main" class="categories_list">
  <header>
    Тут будет меню категорий 
  </header>
</div>

<div id="main" v-if="flags.show_main" class="main">

  <div class="title">
    <li v-for="value in allposts" v-bind:key="value" class="allposts1">
      {{ value.title }}
    </li>
  </div>
  <div class="content">
    <li v-for="value in allposts" v-bind:key="value" class="allposts2">
      {{ value.content }}
    </li>
  </div>

</div>
<div v-if="flags.show_main">
 <button @click="logout()" class="back">Выход</button>
</div>

</template>

<script>
import axios from 'axios'


export default {

  data() {
    return {
      login: {
        username: '',
        password: '',
      },
      flags: {
        show_main: false,
        show_login: true,
        show_register: false,
      },
      register: {
        username: '',
        password: '',
        password2: ''
      },
      token: '',
      err: '',
      allposts: { },
      categories_menu: { },
    }
  },

  created() {},

  mounted() {
    this.check_token()
    this.get_all()
  },

  methods: {

    //Авторизация
    do_login() {
      this.err = ''
      this.flags.show_login = true
      this.flags.show_main = false
      this.flags.show_register = false
      axios
        .post(process.env.VUE_APP_APIURL+'auth/login/', {
          username: this.login.username,
          password: this.login.password,
        })
        .then(response => {
          localStorage.setItem('access_token', JSON.stringify(response.data.access))
          localStorage.setItem('refresh_token', JSON.stringify(response.data.refresh))
          this.token = JSON.parse(localStorage.getItem('access_token'))
          this.flags.show_login = false
          this.flags.show_main = true
          this.flags.show_register = false
          this.get_all()
          this.get_categories()
        }).catch(error => {
          this.err = ''
          this.err = error.response.data.detail
          this.flags.show_login = true
          this.flags.show_main = false
          this.flags.show_register = false
        })
    },

    //Регистрация
    registrate() {
      this.err = ''
      if(this.register.username == '' || this.register.password == '' || this.register.password2 == '') {
        this.err = 'Заполните все поля!'
        return;
      }else if (this.register.username == this.register.password) {
        this.err = 'Логин не может совпадать с паролем!'
        return;
      }
      this.err = ''
      axios
        .post(process.env.VUE_APP_APIURL+'auth/register/', {
          username: this.register.username,
          password: this.register.password,
          password2: this.register.password2
        })
        // eslint-disable-next-line
        .then(response => {
          this.flags.show_login = true
          this.flags.show_main = false
          this.flags.show_register = false
          //return response
        })
        .catch(error => {
          console.log("Error reg")
          console.log(error)
          this.flags.show_register = true
          this.err = ''
          this.err = error.response.data.detail
        })
    },


    do_register() {
      this.err = ''
      this.flags.show_login = false
      this.flags.show_main = false
      this.flags.show_register = true
    },


    back() {
      this.err = ''
      this.flags.show_login = true
      this.flags.show_main = false
      this.flags.show_register = false
    },

    //Проверка токена
    check_token() {
      this.err = ''
      this.token = JSON.parse(localStorage.getItem('access_token'))
      if (this.token) {
        axios
          .post(process.env.VUE_APP_APIURL+'api/token/verify/',
            {
              token: this.token
            })
          .then(response => {
            if (response.status == 401) {
              this.flags.show_login = true
              this.flags.show_main = false
              this.flags.show_register = false
            } else {
              this.flags.show_login = false
              this.flags.show_main = true
              this.flags.show_register = false
            }
          })
          .catch(error => {
            this.err = ''
          this.err = error.response.data.detail
          })
      } else {
        this.logout()
      }
    },

    //Выход
    logout() {
      this.err = ''
      localStorage.removeItem('access_token')
      this.flags.show_login = true
      this.flags.show_main = false
      this.flags.show_register = false
      this.err = ''
      this.err = 'Вы вышли из учетной записи'
    },

    //Получение всех записей
    get_all(){
      this.err = ''
      this.token = JSON.parse(localStorage.getItem('access_token'))
      this.flags.show_main = true
      this.flags.show_login = false
      this.flags.show_register = false
      axios
      .get(process.env.VUE_APP_APIURL+'user/women/', {},
      { headers: { "Authorization": 'Bearer ' + this.token }})
      .then(response => {
        this.flags.show_main = true
        this.allposts = response.data
      })
      .catch(error => {
          this.err = ''
         this.err = error.response.data.detail
        })
    },

    get_categories(){
      this.token = JSON.parse(localStorage.getItem('access_token'))
      axios
      .get(process.env.VUE_APP_APIURL+'user/category/', {},
      { headers: { "Authorization": 'Bearer ' + this.token }})
      .then(response => {
        this.categories_menu = response.data
      })
      .catch(error => {
          this.err = ''
          this.err = error.response.data.detail
        })
    },

  }
}
</script>

<style>

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body{
  background-color: floralwhite;
}

div.main{
  display: grid;
  grid-template-columns: 1fr 1fr;
}

li.allposts2{
  line-height: 1.5;
  margin-bottom: 100px;
  display: grid;
  grid-template-columns: auto solid black;
  grid-template-rows: auto solid black;
  border: 1px solid black;
  margin-bottom: 10px;
  height: 300px;
  width: 1500px;
  scroll-behavior: smooth;
  overflow: auto;
  text-align: center;
}

li.allposts1{
  line-height: 5;
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto solid black;
  grid-template-rows: auto solid black;
  border: 1px solid black;
  margin-bottom: 10px;
  height: 300px;
  width: 350px;
  text-align: center;
}

button.back{
  width: 100%;
}

div.categories_list{
  padding-bottom: 40px;
  text-align: center;
}

</style>
