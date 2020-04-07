<template>
  <div class="container">
    <button @click.prevent="openLoginWindow">Login</button>
  </div>
</template>

<script>

import crypto from 'crypto-js';

export default {
  data() {
    return {
      email: '',
      password: '',
      state: '',
      challenge: '',
    }
  },

  computed: {
    loginUrl() {
      return 'http://pkce-back.web/oauth/authorize?client_id=1&redirect_uri=http://localhost:3000/auth&response_type=code&scope=*&state=' + this.state + '&code_challenge=' + this.challenge + '&code_challenge_method=S256'
    }
  },

  mounted() {
    window.addEventListener('message', (e) => {
      if (e.origin !== 'http://localhost:3000' || ! Object.keys(e.data).includes('access_token')) {
        return;
      }

      const {token_type, expires_in, access_token, refresh_token} = e.data;
      console.log(e.data);
      this.$axios.setToken(access_token, token_type);

      this.$axios.$get('http://pkce-back.web/api/user')
        .then(resp => {
          console.log(resp);
        })
    });

    this.state = this.createRandomString(40);
    const verifier = this.createRandomString(128);

    this.challenge = this.base64Url(crypto.SHA256(verifier));
    window.localStorage.setItem('state', this.state);
    window.localStorage.setItem('verifier', verifier);
  },

  methods: {
    openLoginWindow() {
      window.open(this.loginUrl, 'popup', 'width=700,height=700');
    },

    createRandomString(num) {
      return [...Array(num)].map(() => Math.random().toString(36)[2]).join('')
    },

    base64Url(string) {
      return string.toString(crypto.enc.Base64)
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
    }
  }
}
</script>
