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
<div id="main" v-if="flags.show_main" class="main">
<header>
    <li v-for="value in categories" v-bind:key="value" class="categories">
    {{ value }}
  </li>
</header>
  <li v-for="value in allposts" v-bind:key="value" class="allposts">
    {{ value }}
  </li>
  <button @click="logout()" class="register_click">Выход</button>
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
      categories: { },
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
        }).catch(error => {
          console.log("Error login")
          console.log(error)
          this.err = ''
          this.err = 'Данные указаны неверно, авторизация невозможна!'
          this.flags.show_login = true
          this.flags.show_main = false
          this.flags.show_register = false
        })
    },

    is_show_login() {
      return this.flags.show_login == true
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
          this.reg_error = error
          this.err = ''
          this.err = 'Неверно указаны данные, регистрация невозможна!'
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
            console.log("Error auth")
            console.log(error)
            this.err = ''
            this.err = 'Вы не авторизованы!'
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
          console.log("Error getAll")
          console.log(error)
          this.err = ''
          this.err = 'Невозможно получить записи!'
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

</style>
