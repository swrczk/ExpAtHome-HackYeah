<template>
  <v-app>
    <v-app-bar
      app
      color="blue-grey darken-4"
      dark
      clipped-left
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2 pa-3"
          contain
          src="logo.png"
          transition="scale-transition"
          width="60"
        />
        <v-card-title class="white--text darken-2">
          <h2>#ExpAtHome</h2>
        </v-card-title>
      </div>

    </v-app-bar>
    <v-navigation-drawer v-model="right" right app width="75%">
        <v-spacer /><v-btn icon @click="right = false"><v-icon>close</v-icon></v-btn>
        <v-card>
            <v-card-title>Achievements</v-card-title>
            <v-card-text><Achievements /></v-card-text>
        </v-card>
        
    </v-navigation-drawer>
    <v-navigation-drawer
            :permanent="$vuetify.breakpoint.mdAndUp"
            v-model="nav"
            app
            clipped
    >
      <User :user="user" />
      <Level :level="user.exp" />
      <v-divider />
      <v-btn v-if="user && user.id" block @click="right = !right" class="mb-3">
         <v-icon>verified_user</v-icon>
         Achievements
       </v-btn>
      <v-btn v-if="user && user.id" block :color="myList ? 'primary' : ''" @click="myList = !myList; catid = -1" class="mb-3">
        My list</v-btn>
      <v-list-item-group v-model="catid">
        <v-list-item
                v-for="item in categories"
                :key="item.id"
                link
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.name }}</v-list-item-title>
            <v-list-item-subtitle>{{ item.info }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
      </v-list-item-group>

      <template v-slot:append>
        <div class="pa-2" v-if="user && user.id && user.admin">
          <v-btn block @click.stop="newUser = {}; modAddUser = true">Add user...</v-btn>
        </div>
        <div class="pa-2" v-if="user && user.id && user.admin">
          <v-btn block @click.stop="newCategory = {}; modAddCategory = true">Add category...</v-btn>
        </div>
        <div class="pa-2" v-if="user && user.id">
          <v-btn block @click.stop="newTask = {}; modAddTask = true">Add task...</v-btn>
        </div>
        <div class="pa-2">
          <v-btn block @click.stop="loginButton">{{ user && user.id ? 'Log out' : 'Log in' }}</v-btn>
        </div>
      </template>
    </v-navigation-drawer>

    <v-dialog persistent v-model="modAddCategory" max-width="50%">
        <v-card>
            <v-card-title class="headline">Add category</v-card-title>
            <v-card-text>
                <v-container>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="newCategory.name" label="Name" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-textarea v-model="newCategory.info" label="Info" required></v-textarea>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="newCategory.icon" label="Icon" required></v-text-field>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="modAddCategory = false">Close</v-btn>
                <v-btn color="blue darken-1" text @click="addCategory">Add</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>


    <v-dialog persistent v-model="modAddTask" max-width="50%">
        <v-card>
            <v-card-title class="headline">Add task</v-card-title>
            <v-card-text>
                <v-container>
                    <v-row>
                        <v-col cols="12">
                            Category: {{ category.name }}
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="newTask.name" label="Name" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-textarea v-model="newTask.info" label="Info" required></v-textarea>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field type="number" suffix=" XP" v-model="newTask.exp" label="Experience" required></v-text-field>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="modAddTask = false">Close</v-btn>
                <v-btn color="blue darken-1" text @click="addTask">Add</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>

    <v-dialog persistent v-model="modAddUser" max-width="50%">
        <v-card>
            <v-card-title class="headline">Add user</v-card-title>
            <v-card-text>
                <v-container>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="newUser.username" label="Login" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="newUser.name" label="User name" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field type="password" v-model="newUser.pass" label="Password" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field type="number" suffix=" XP" v-model="newUser.exp" label="Experience" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-switch v-model="newUser.admin" label="Administrator"></v-switch>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="modAddUser = false">Close</v-btn>
                <v-btn color="blue darken-1" text @click="addUser">Add</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>


    <v-dialog persistent v-model="modLogin" max-width="50%">
        <v-card>
            <v-card-title class="headline">Log in</v-card-title>
            <v-card-text>
                <v-container>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field v-model="loginUser" label="User name" required></v-text-field>
                        </v-col>
                    </v-row>
                    <v-row>
                        <v-col cols="12">
                            <v-text-field type="password" v-model="loginPass" label="Password" required></v-text-field>
                        </v-col>
                    </v-row>
                </v-container>
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" text @click="modLogin = false">Close</v-btn>
                <v-btn color="blue darken-1" text @click="login(loginUser, loginPass)">Log in</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>


    <v-content>
      <v-container>
            <v-card v-if="myList && getTasks.length == 0" class="ma-3">
                <v-card-title>Your list is empty.</v-card-title>
            </v-card>
            <Tasks :tasks="getTasks" :logged="user && user.id"
                v-on:savetask="selectTask($event)"
                v-on:endtask="doneTask($event)" />
        </v-container>
    </v-content>


    <v-footer app>
        <span class="mr-1">Tasks: {{ tasks.length }}.</span>
        <span v-if="myList">Showing your list of tasks.</span>
        <span v-else-if="catTasks.length">Tasks in category: {{ catTasks.length }}</span>
        <span v-else>No category selected. All tasks are shown.</span>
        <v-spacer></v-spacer>
        <div>HackYeah 2020</div>
    </v-footer>

    <v-snackbar v-model="snack" :timeout="5000" :color="snackColor">
        {{ snackText }}
        <v-btn text="" @click="snack = false">Close</v-btn>
    </v-snackbar>

  </v-app>

</template>

<script>
import User from './components/User';
import Level from './components/Level';
import Tasks from './components/Tasks';
import Achievements from './components/Achievements';

import tasks from './json/tasks.json';

import axios from 'axios';

export default {
  name: 'App',

  components: {
    User,
    Level,
    Tasks,
    Achievements,
  },

  methods: {
      makeBool(s) { return (s === true || s == 1) ? true : false },
      gotExp(xp) { console.log('Got ', xp, ' XP.', xp.type); this.user.exp += xp || 0
        axios.patch(this.api + '/users/' + this.user.id,
            { exp: this.user.exp || 0, admin: this.makeBool(this.user.admin) }) },
      checkAssigns() { var done = {}; var onlist = {}
          this.assigns.forEach(function(a) { onlist[a.id] = true; done[a.id] = true })
          this.tasks.forEach(function(t) {
              if (done[t.id]) { t.done = true } if (onlist[t.id]) { t.onlist = true } })
          this.catTasks.forEach(function(t) {
              if (done[t.id]) { t.done = true } if (onlist[t.id]) { t.onlist = true } })
      },
      addUser() {
          this.modAddUser = false
          if (!this.newUser || !this.newUser.name || !this.newUser.pass) { return }
          this.newUser.exp = Number(this.newUser.exp) || 0
          axios.post(this.api + '/users/', this.newUser).then(r => {
              this.setSnack("New user added."); this.refresh(r) })
          .catch(e => { this.handler(e) }) },
      getAssigns() {
          axios.get(this.api + '/users/' + this.user.id + '/tasks/')
          .then(r => { this.assigns = r.data; this.checkAssigns() }).catch(e => { this.handler(e) })
      },
      getTask(id) { var t = this.tasks.find(function(t) { return t.id == id })
        if (t) { return t } else { this.setSnack("Nie znaleziono zadania: " + id) } },
      selectTask(id) {
          if (!this.user || !this.user.id) { this.setSnack("You have to log in first.") }
          var t = this.getTask(id); if (t) {
          if (this.assigns.find(function(a) { return a.taskid == t.id })) {
              this.setSnack("This task is already on the list."); return }
          axios.post(this.api + '/assigns/', { taskid: t.id, userid: this.user.id })
          .then(r => { this.setSnack("Added task '" + t.name + "' to the list."); this.refresh(r) })
          .catch(e => { this.handler(e) }) } },
      doneTask(e) { var t = this.getTask(e.id); if (!t) { return }
          if (!this.user || !this.user.id) { this.setSnack("You have to log in first.") }
          var aid = this.assigns.find(function(a) { return a.taskid == t.id })
          if (aid) { axios.patch(this.api + '/assigns/' + aid.id, { done: true })
              .then(r => { this.refresh(r); this.snackDone(t); this.gotExp(t.exp)})
              .catch(e => { this.handler(e) }) }
          else { axios.post(this.api + '/assigns/', { taskid: t.id, userid: this.user.id, done: true })
          .then(r => { this.refresh(r); this.snackDone(t); this.gotExp(t.exp) }).catch(e => { this.handler(e) }) } },
      snackDone(t) { this.setSnack("Task '" + t.name + "' set as done.") },
      setSnack(txt) { this.snackText = txt; this.snack = true },
      loginError(e) { this.setSnack("Invalid user name or password."); if (e) { console.log(e) } },
      addTask() {
          this.modAddTask = false
          if (!this.newTask || !this.newTask.name || !this.newTask.name.length) { return }
          this.newTask.added = this.user.id
          if (this.user.admin) { this.newTask.confirmed = this.user.id }
          this.newTask.catid = this.category.id || this.categories[0].id
          this.newTask.exp = Number(this.newTask.exp) || 1
          axios.post(this.api + '/tasks/', this.newTask)
          .then(r => { this.refresh(r); this.setSnack("Task added. Awaiting validation.") })
          .catch(e => { this.handler(e) })
      },
      addCategory() {
          this.modAddCategory = false
          if (!this.user.admin) { return }
          if (!this.newCategory || !this.newCategory.name || !this.newCategory.name.length) { return }
          axios.post(this.api + '/categories/', this.newCategory)
          .then(r => { this.refresh(r); this.setSnack("Category added.") }).catch(e => { this.handler(e) })
      },
      T(str) { return str },
      handler(e) { console.log(e) },
      loginButton() { if (this.user && this.user.id) { this.user = {}; this.assigns = []; this.myList = false }
        else { this.loginUser = ''; this.loginPass = ''; this.modLogin = true } },
      login(user, pass) {
          this.modLogin = false; if (!user || !pass) { return }
          axios.get(this.api + '/users/?filter={ $and: [ { username: "' + user + '" }, { pass: "' + pass + '" } ] }')
          .then(r => { if(r.data.length) { this.user = r.data[0]; this.setSnack("Logged in."); this.getAssigns()}
            else { this.loginError() } this.loginUser = ''; this.loginPass ='' })
          .catch(e => { this.loginError(e) })
      },
      setTasks(tasks) {
          tasks.forEach(function(t) { t.overlay = false })
          this.tasks = tasks
          axios.get(this.api + '/users/').then(r => {
                  var users = {}
                  r.data.forEach(function(u) { users[u.id] = u.name })
                  this.tasks.forEach(function(t) { t.username = users[t.added] || '?' })
              }).catch(e => { this.handler(e) })
      },
      setCatTasks(tasks) { tasks.forEach(function(t) { t.overlay = false });
          this.catTasks = tasks
          axios.get(this.api + '/users/').then(r => {
                  var users = {}
                  r.data.forEach(function(u) { users[u.id] = u.name })
                  this.catTasks.forEach(function(t) { t.username = users[t.added] || '?' })
              }).catch(e => { this.handler(e) })
      },
      refresh() {
          var handler = this.handler
          axios.get(this.api + '/tasks/').then(r => { this.setTasks(r.data) }).catch(e => { handler(e) })
          axios.get(this.api + '/categories/').then(r => { this.categories  = r.data })
          .catch(e => { handler(e) })
          if (this.user && this.user.id) { this.getAssigns() } },
      clearList() {
          var api = this.api; var handler = this.handler
          this.assigns.forEach(function(a) {
              axios.delete(api + '/assigns/' + a.id).then(r => { console.log(r) })
              .catch(e => { handler(e) }) }) },
  },

  data: () => ({
    items: [
        { title: 'Dashboard', icon: 'dashboard' },
        { title: 'Account', icon: 'account_box' },
        { title: 'Admin', icon: 'gavel' },
    ],
    myList: false,
    nav: null, right: false,
    user: {},
    tasks: tasks, catTasks: [],
    category: {}, catid: -1, categories: {},
    api: 'http://localhost:8888',
    loginUser: '', loginPass: '', modLogin: false,
    modAddTask: false, modAddCategory: false,
    newTask: {}, newCategory: {},
    snack: false, snackText: '', snackColor: 'primary',
    assigns: [],
    newUser: {}, modAddUser: false,
  }),

  watch: {
      catid(id) {
          var c = this.categories[id]; this.category = c || {}
          if (c && c.id) {
          this.myList = false
              axios.get(this.api + '/categories/' + c.id + '/tasks/').then(r => { this.setCatTasks(r.data) })
                .catch(e => { this.handler(e) })
          // } else { this.category = this.categories[0]; this.catTasks = [] }
          } else { this.catTasks = [] }
      },
  },

  computed: {
      confirmedTasks() { var tasks = this.tasks.filter(function(t) { return t.confirmed && t.confirmed.length })
          var onlist = {}; var done = {}
          this.assigns.forEach(function(a) { onlist[a.taskid] = true; done[a.taskid] = a.done })
          tasks.forEach(function(t) {
              if (done[t.id]) { t.done = true } if (onlist[t.id]) { t.onlist = true } })
          return tasks },
      // confirmedCatTasks() { return this.catTasks.filter(function(t) { return t.confirmed && t.confirmed.length }) },
      confirmedCatTasks() {
          var cat = this.category ? this.category.id : ''
          return this.confirmedTasks.filter(function(t) { return t.catid == cat }) },
      getTasks() { if (this.myList) { return this.makeAssigns }
        return (this.category && this.category.id) ? this.confirmedCatTasks : this.confirmedTasks },
      makeAssigns() {
          var l = []; var gt = this.getTask
          this.assigns.forEach(function(a) {
              var t = {}; Object.assign(t, gt(a.taskid)); t.done = a.done; t.onlist = true; l.push(t) })
          return l
      }
  },

  mounted() {
      this.refresh()
  }
};
</script>
