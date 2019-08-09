<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div class="container-fluid">
      <div class="card">
        <div class="card-content">
          <div class="row" style="margin: 50px 50px;">
            <div class="col-sm-12">
              <button class="btn btn-fill btn-info btn-sm" v-if='!authorized' @click="handleAuthClick">Sign In</button>
              <button class="btn btn-fill btn-info btn-sm" v-if='authorized' @click="handleSignoutClick">Sign Out</button>
              <button class="btn btn-fill btn-info btn-sm" v-if='authorized' @click="getData">Get Data</button>
              <vue-json-pretty
                :path="'res'"
                :data="items"
                v-if="authorized && items">
              </vue-json-pretty>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty'
export default {
  components: {
    VueJsonPretty
  },
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Integrate to Google Calendar API',
      items: undefined,
      api: undefined,
      authorized: false,
      CLIENT_ID: '895296071624-v16sbv41dr390305b07rsp3dk5cbrn7j.apps.googleusercontent.com',
      API_KEY: 'AIzaSyBsPZfMyOsqgIobIrAZnDKYprEHjgFDfsg',
      DISCOVERY_DOCS: ['https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest'],
      SCOPES: 'https://www.googleapis.com/auth/calendar.readonly'
    }
  },
  created: function () {
    // eslint-disable-next-line
    this.api = gapi
    this.handleClientLoad()
  },
  methods: {
    handleClientLoad () {
      this.api.load('client:auth2', this.initClient)
    },
    initClient () {
      var self = this
      self.api.client.init({
        apiKey: self.API_KEY,
        clientId: self.CLIENT_ID,
        discoveryDocs: self.DISCOVERY_DOCS,
        scope: self.SCOPES
      }).then(_ => {
        self.api.auth2.getAuthInstance().isSignedIn.listen(self.authorized)
        self.authorized = self.api.auth2.getAuthInstance().isSignedIn.get()
      })
    },
    handleAuthClick (event) {
      Promise.resolve(this.api.auth2.getAuthInstance().signIn())
        .then(_ => {
          this.authorized = true
        })
    },
    handleSignoutClick (event) {
      Promise.resolve(this.api.auth2.getAuthInstance().signOut())
        .then(_ => {
          this.authorized = false
        })
    },
    getData () {
      let vm = this

      vm.api.client.calendar.events.list({
        'calendarId': 'primary',
        'timeMin': (new Date()).toISOString(),
        'showDeleted': false,
        'singleEvents': true,
        'maxResults': 10,
        'orderBy': 'startTime'
      }).then(response => {
        vm.items = response.result.items
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
