<template>
  <div id="app">
    <div class="container">
      <div class="row justify-content-md-center">
        <div class="col-md-6">
          <form>
            <div class="form-group">
              <label for="exampleInputEmail1">Redirect To</label>
              <input type="text" class="form-control" placeholder="Default: https://jwt.io" v-model="redirectUri">
            </div>
            <div class="form-group">
              <label for="exampleFormControlTextarea1">Claims (as JSON)</label>
              <prism-editor v-model="claims" language="js" :line-numbers="true" class="code-editor"/>
            </div>
            <!-- <div class="form-group">
              <label for="exampleFormControlTextarea1">Other URL Hash Values</label>
              <small id="emailHelp" class="form-text text-muted">One per line formatted as 'key=value'</small>
              <textarea class="form-control" id="exampleFormControlTextarea1" rows="3" v-model="otherUrlHashParams"></textarea>
            </div> -->
            <div class="form-check">
              <input type="checkbox" class="form-check-input" id="exampleCheck1" v-model="shouldSaveConfig">
              <label class="form-check-label" for="exampleCheck1">Save Config on 'Go'</label>
            </div>
            <button class="btn btn-primary" @click="openInCurrentWindow">Go</button>
            <button class="btn btn-primary" @click="openInNewWindow">Go (New Window)</button>
          </form>
        </div>
      </div>
      <div class="row justify-content-md-center" >
        <hr />
      </div>
      <div class="row justify-content-md-center">
        <span class="footer-text">Built with ♥ by <a href="https://brianmorrison.me">Brian Morrison II</a></span>
      </div>
      <div class="row justify-content-md-center" >
        <div class="footer-text">
          If you found this tool useful, consider buying me a coffee:
        </div>
        <div>
          <form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_blank">
            <input type="hidden" name="cmd" value="_s-xclick" />
            <input type="hidden" name="hosted_button_id" value="SH28RAYLMZREG" />
            <input type="image" src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif" border="0" name="submit" title="PayPal - The safer, easier way to pay online!" alt="Donate with PayPal button" />
            <img alt="" border="0" src="https://www.paypal.com/en_US/i/scr/pixel.gif" width="1" height="1" />
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import PrismEditor from 'vue-prism-editor'
import jwt from 'jsonwebtoken'

export default {
  name: 'App',
  components: {
    PrismEditor
  },
  data: function() {
    return {
      redirectUri: '',
      claims: '',
      shouldSaveConfig: false,
      otherUrlHashParams: ''
    }
  },
  mounted: function () {
    // Load config if it exists
    if(localStorage.getItem("config")) {
      let loadedConfig = localStorage.getItem("config")
      let converted = JSON.parse(loadedConfig)
      this.redirectUri = converted.redirectUri;
      this.claims = converted.claims
      this.otherUrlHashParams = converted.otherUrlHashParams
      this.shouldSaveConfig = true;
    }
  },
  methods: {
    openInCurrentWindow: async function(e) {
      e.preventDefault()
      this.go()
    },

    openInNewWindow: async function(e) {
      e.preventDefault()
      this.go(true)
    },

    go: async function (shouldOpenInNewWindow) {
      // If save is checked, save to local storage
      if(this.shouldSaveConfig) {
        let config = {
          redirectUri: this.redirectUri,
          claims: this.claims,
          otherUrlHashParams: this.otherUrlHashParams
        }
        localStorage.setItem("config", JSON.stringify(config))
      } else {
        localStorage.clear()
      }

      let claims = JSON.parse(this.claims)

      claims.iat = Math.floor(Date.now() / 1000)
      if(this.expiresIn) {
        claims.exp = claims.iat + Number(this.expiresIn)
      } else {
        claims.exp = claims.iat + 3600
      }

      let redirectUri = this.redirectUri
      if(redirectUri == '') {
        redirectUri = 'https://jwt.io'
      }

      let token = jwt.sign(claims, "not_a_secret", { algorithm: 'HS256' });
      let url = `${redirectUri}#id_token=${token}&access_token=${token}`

      if(shouldOpenInNewWindow) {
        window.open(url)
      } else {
        window.location.href = url
      }
    }
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

.code-editor {
  height: 400px !important;
}

.footer-text {
  font-style: italic;
  color: #777;
  padding-right: 5px;
  padding-top: 2px;
}

button { 
  margin: 0px 10px !important;
}
</style>
