<template>
  <div class="dashboard">
    <div class="container">
      <div class="row justify-content-md-center">
        <div class="col-md-8 col-md-offset-2">
          <div class="panel-welcome" id="section-2">
            <div class="avatar-section">
              <img :src="avatar" class="img-rounded avatar" id="avatar-image" />
            </div>
            <h1 class="landing-heading">
              <span id="heading-name">{{ givenName }}'s</span>!
              Todo List
              <small>
                <span class="sign-out">
                  (
                  <a href="#" @click.prevent="signOut">Sign Out</a>)
                </span>
              </small>
            </h1>
            <form @submit.prevent="addTodo" :disabled="! todo">
              <div class="input-group">
                <input
                  v-model="todo"
                  type="text"
                  class="form-control"
                  placeholder="New todo..."
                  autofocus
                />
                <span class="input-group-btn">
                  <button class="btn btn-default bg-light" type="submit" :disabled="! todo">Add</button>
                </span>
              </div>
            </form>
            <ul class="list-group">
              <li
                v-for="todo in todos"
                class="list-group-item"
                :class="{completed: todo.completed}"
                :key="todo.id"
              >
                <label>
                  <input type="checkbox" v-model="todo.completed" />
                  {{ todo.text }}
                </label>
                <a
                  @click.prevent="todos.splice(todos.indexOf(todo), 1)"
                  class="delete float-right"
                  href="#"
                >X</a>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { userSession } from '../userSession'
var STORAGE_FILE = 'todos.json'
export default {
  name: 'dashboard',
  props: ['user'],
  methods: {
    signOut () {
      userSession.signUserOut(window.location.href)
    },
    addTodo () {
      if (!this.todo.trim()) {
        return
      }
      this.todos.unshift({
        id: this.uidCount++,
        text: this.todo.trim(),
        completed: false
      })
      this.todo = ''
    },
    fetchData () {
      userSession
        .getFile(STORAGE_FILE) // decryption is enabled by default
        .then(todosText => {
          var todos = JSON.parse(todosText || '[]')
          todos.forEach(function (todo, index) {
            todo.id = index
          })
          this.uidCount = todos.length
          this.todos = todos
        })
    }
  },
  data () {
    return {
      blockstack: window.blockstack,
      avatar: 'https://s3.amazonaws.com/onename/avatar-placeholder.png',
      givenName: 'Anonymous',
      todos: [],
      todo: '',
      uidCount: 0
    }
  },
  watch: {
    todos: {
      handler: function (todos) {
        return userSession.putFile(STORAGE_FILE, JSON.stringify(todos))
      },
      deep: true
    }
  },
  mounted () {
    const blockstack = this.blockstack
    if (blockstack.isUserSignedIn()) {
      const profile = blockstack.loadUserData().profile
      const user = new blockstack.Person(profile)
      this.givenName = user.name() ? user.name() : 'Nameless Person'
      if (user.avatarUrl()) this.avatar = user.avatarUrl()
    } else if (blockstack.isSignInPending()) {
      blockstack.handlePendingSignIn().then(userData => {
        window.location = window.location.origin
      })
    }
    this.fetchData()
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  padding-top: 2rem;
}

input::placeholder {
  color: grey;
}

label {
  margin-bottom: 0;
  // width: 100%;
  cursor: pointer;
  input[type="checkbox"] {
    margin-right: 5px;
  }
}
.list-group-item {
  &.completed label {
    text-decoration: line-through;
  }

  .delete {
    display: none;
  }

  &:hover .delete {
    display: inline;
    color: grey;
    &:hover {
      text-decoration: none;
      color: red;
    }
  }
}
</style>
