<template>
  <div id="app">
    <ul>
      <li class="user" v-for="user in users" transition>
        <span>{{user.name}} - {{user.email}}</span>
        <button v-on:click="removeUser(user)">X</button>
      </li>
    </ul>
    <form id="form" v-on:submit.prevent="addUser">
      <input v-model="newUser.name">
      <input v-model="newUser.email">
      <input type="submit" value="Add User">
    </form>
    <ul class="errors">
      <li v-show="!validation.name">Name cannot be empty.</li>
      <li v-show="!validation.email">Please provide a valid email address.</li>
    </ul>
  </div>
</template>

<script>
var firebase = require('firebase')
var config = {
  apiKey: 'AIzaSyDVwE2n3_YH-j5OLm-OJKx18fhkzM50-f4',
  authDomain: 'assignment-a022c.firebaseapp.com',
  databaseURL: 'https://assignment-a022c.firebaseio.com',
  storageBucket: 'assignment-a022c.appspot.com',
  messagingSenderId: '358496802319'
}
firebase.initializeApp(config)
var Users = firebase.database().ref('users')
var emailRE = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
export default {
  data () {
    return {
      users: [],
      newUser: {
        name: '',
        email: ''
      }
    }
  },
  ready () {
    let vm = this
    Users.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.users.push(item)
    })
    Users.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.users.$remove(vm.users.find(user => user.id === id))
    })
  },
  computed: {
    validation: function () {
      return {
        name: !!this.newUser.name.trim(),
        email: emailRE.test(this.newUser.email)
      }
    },
    isValid: function () {
      var validation = this.validation
      return Object.keys(validation).every(function (key) {
        return validation[key]
      })
    }
  },
  methods: {
    addUser: function () {
      if (this.isValid) {
        Users.push(this.newUser)
        this.newUser.name = ''
        this.newUser.email = ''
      }
    },
    removeUser: function (user) {
      firebase.database().ref('users/' + user.id).remove()
    }
  }
}
</script>


<style scoped>
body {
  font-family: Helvetica, Arial, sans-serif;
}

ul {
  padding: 0;
}

.user {
  height: 30px;
  line-height: 30px;
  padding: 10px;
  border-top: 1px solid #eee;
  overflow: hidden;
  transition: all .25s ease;
}

.user:last-child {
  border-bottom: 1px solid #eee;
}

.v-enter, .v-leave {
  height: 0;
  padding-top: 0;
  padding-bottom: 0;
  border-top-width: 0;
  border-bottom-width: 0;
}

.errors {
  color: #f00;
}

</style>
