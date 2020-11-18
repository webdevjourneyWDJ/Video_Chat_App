<template>
  <div id="app">
    <Navigation :user="user" @logout="logout"/>
    <router-view :user="user" :rooms="rooms" @logout="logout" @addRoom="addRoom"/>
  </div>
</template>

<script>
import db from './db'
import Firebase from 'firebase'
import Navigation from '@/components/Navigation'
export default {
  name: "App", 
  data: function() {
    return {
      user: null,
      rooms: []
    }
  },

  methods: {
    logout: function() {
      Firebase.auth().signOut().then(() => {
        this.user = null
        this.$router.push('login')
      })
    },

    addRoom: function(roomName) {
      db.collection('users')
        .doc(this.user.uid)
        .collection('rooms')
        .add({
          name: roomName,
          createdAt: Firebase.firestore.FieldValue.serverTimestamp()
        })
    }
  },
  mounted() {
    Firebase.auth().onAuthStateChanged(user => {
      if(user) { 
        this.user = user
        db.collection('users')
          .doc(this.user.uid)
          .collection('rooms')
          .onSnapshot(snapShot => {
            const snapData = []
            snapShot.forEach(doc => {
              snapData.push({
                id: doc.id,
                name: doc.data().name
              })
            })

            this.rooms = snapData.sort((a, b) => {
              if(a.name.toLowerCase() < b.name.toLowerCase()){
                return -1
              } else return 1
            })
          })  
      }
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
