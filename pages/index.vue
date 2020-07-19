<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-flex
      xs12
      sm8
      md6
    >
      <div class="text-h1">{{title}}</div>
      <div>
        <v-btn v-if="!msalApp" @click="processLogin">LOGIN</v-btn>
      </div>
      <div>
        <v-btn v-if="msalApp" @click="acquireToken">ACQUIRE TOKEN</v-btn>
      </div>
      <div>
        <v-btn v-if="msalApp" @click="forceRefreshToken">FORCE REFRESH</v-btn>
      </div>
      <div>
        Login Date Time: {{loginDateTime === 'N/A' ? loginDateTime : this.formatDateTime(loginDateTime)}}
      </div>
      <div>
        Current Date Time: {{currentDateTime === 'N/A' ? currentDateTime : this.formatDateTime(currentDateTime)}}
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
  import * as Msal from 'msal'

export default {
  data() {
    return {
      title: 'MSAL token refresh',
      loginDateTime: 'N/A',
      currentDateTime: 'N/A',
      msalApp: undefined,
      idToken: undefined
    }
  },
  mounted() {
    setInterval(() => {
      this.currentDateTime = this.$moment()
    }, 1000)
  },
  methods: {
    processLogin: function () {
      this.loginDateTime = this.$moment()
      const msalConfig = {
        auth: {
          clientId: process.env.CLIENT_ID,
          authority: `https://login.microsoftonline.com/${process.env.TENANT_ID}`,
          redirectUrl: 'http://localhost:3000/'
        },
        cache: {
          // https://docs.microsoft.com/ja-jp/azure/active-directory/develop/msal-js-sso
          cacheLocation: 'sessionStorage', // default
          // cacheLocation: 'localStorage',

          // This should be true if some troubles occurred with IE/Edge.
          // https://docs.microsoft.com/ja-jp/azure/active-directory/develop/msal-js-initializing-client-applications
          // storeAuthStateInCookie: false // default
        }
      }
      const loginReq = {
        // scopes: [process.env.APP_SCOPE]
        scopes: [process.env.CLIENT_ID]
      }
      this.msalApp = new Msal.UserAgentApplication(msalConfig)
      msal.loginPopup(loginReq).then(resp => {
        this.idToken = resp.idToken
        console.log(this.idToken.rawIdToken)
        this.decodeToken(this.idToken.rawIdToken)
      })
    },
    formatDateTime: function (dateTime) {
      return dateTime.format('YYYY-MM-DD HH:mm:ss.SSS')
    },
    acquireToken: function () {
      // acquireTokenSilent
      // https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/src/UserAgentApplication.ts#L697
      const req = {
        scopes: [process.env.CLIENT_ID]
      }
      this.msalApp.acquireTokenSilent(req).then(resp => {
        const prevToken = this.idToken
        this.idToken = resp.idToken
        console.log(this.idToken.rawIdToken)
        this.decodeToken(this.idToken.rawIdToken)
        console.log(`Same token?: ${this.idToken.rawIdToken === prevToken.rawIdToken}`)
      })
    },
    forceRefreshToken: function() {
      // force refresh
      // https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/src/UserAgentApplication.ts#L782
      const req = {
        scopes: ['dbd4f182-2af9-4e4e-9d60-fb43b949b197'],
        forceRefresh: true
      }
      this.msalApp.acquireTokenSilent(req).then(resp => {
        const prevToken = this.idToken
        this.idToken = resp.idToken
        console.log(this.idToken.rawIdToken)
        this.decodeToken(this.idToken.rawIdToken)
        console.log(`Same token?: ${this.idToken.rawIdToken === prevToken.rawIdToken}`)
      })
    },
    decodeToken: function (jwt) {
      // https://qiita.com/johnslith/items/d11f827f8b14913b4a28
      // https://stackoverflow.com/questions/38552003/how-to-decode-jwt-token-in-javascript-without-using-a-library
      const chunks = jwt.split('.')
      // header
      const base64Header = chunks[0].replace(/-/g, '+').replace(/_/g, '/');
      const header = JSON.parse(decodeURIComponent(escape(window.atob(base64Header))));
      // payload
      const base64Payload = chunks[1].replace(/-/g, '+').replace(/_/g, '/');
      const payload = JSON.parse(decodeURIComponent(escape(window.atob(base64Payload))));

      console.log('header: ')
      console.log(header)
      console.log('payload: ')
      console.log(payload)

      const expiresAt = parseInt(payload.exp, 10)
      const currentEpoch = this.$moment().unix()
      console.log(`expiresAt: ${expiresAt} = ${this.$moment(expiresAt * 1000).format('YYYY-MM-DD HH:mm:ss')}`)
      console.log(`current: ${currentEpoch} = ${this.$moment(currentEpoch * 1000).format('YYYY-MM-DD HH:mm:ss')}`)
      console.log(`expired?: ${expiresAt < currentEpoch}`)
    }
  }
}
</script>
