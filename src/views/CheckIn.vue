<template>
  <form class="mt-3" @submit.prevent="handleCheckIn">
    <div class="container">
      <div class="row justify-content-center">
        <div class="col-lg-6">
          <div class="card bg-light">
            <div class="card-body" v-if="user">
              <h1 class="font-weight-light mb-0">Check in</h1>
              <p class="font-weight-bold" v-if="roomName">
                To: <span class="text-primary">{{ roomName }}</span>
              </p>
              <section class="form-group">
                <label class="form-control-label sr-only" for="displayName">Name</label>
                <input required class="form-control" type="text" v-model="displayName" />
              </section>
              <div class="form-group text-right mb-0">
                <button class="btn btn-primary" type="submit">Check in</button>
              </div>
            </div>
            <div class="card-body card-outline-danger text-center" v-else>
              <h1 class="text-danger card-title ">Sorry</h1>
              <p class="card-text lead">
                Sorry, access to rooms is only available to registered users. Please
                <router-link to="/login">login</router-link> or
                <router-link to="/register">register</router-link> and try again.
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </form>
</template>

<script>
import Firebase from 'firebase'
import db from '../db'
export default {
  data: function() {
    return {
      displayName: null,
      roomName: null
    }
  },
  methods: {
    handleCheckIn: function() {
      const roomRef = db.collection('users')
        .doc(this.$route.params.hostID)
        .collection('rooms')
        .doc(this.$route.params.roomID)

      roomRef.get().then(doc => {
        if(doc.exists){
          roomRef.collection('attendees')
            .doc(this.user.uid)
            .set({
              displayName: this.displayName,
              createdAt: Firebase.firestore.FieldValue.serverTimestamp()
            }).then(() => this.$router.push(`/chat/${this.$route.params.hostID}/${this.$route.params.roomID}`))
        }
      })
    }
  },
  props: ['user'],

  mounted() {

    //Getting User
    Firebase.auth().onAuthStateChanged(user => {
      if(user) this.displayName = user.displayName
    })

    //Getting Room Name
    db.collection('users')
      .doc(this.$route.params.hostID)
      .collection('rooms')
      .doc(this.$route.params.roomID)
      .get().then(doc => {
        if(doc.exists) this.roomName = doc.data().name
        else this.$router.push('/')
      })
  }
}
</script>