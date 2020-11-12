<template>
  <div id="app">
    <Navigation :user="user"/>
    <router-view :user="user"/>
  </div>
</template>

<script>
import db from './db'
import Navigation from '@/components/Navigation';
export default {
  name: "App", 
  data: function() {
    return {
      user: null
    }
  },

  mounted() {
    db.collection('users')
      .doc('L4x7d2SwW0BsroL6P1Hc')
      .get()
      .then(snapshot => {
        this.user = snapshot.data().name
      })
  },
  components: {
    Navigation
  }
}
</script>

<style lang="scss">
  @import 'node_modules/bootstrap/scss/bootstrap';
</style>
