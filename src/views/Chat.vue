<template>
  <div class="container-fluid mt-4">
    <div class="mb-3">
      <span class="mb-0 h2 text-primary">{{roomName}}</span>
      <span class="ml-1"> Hosted by: <strong class="text-danger">{{hostDisplayName}}</strong> </span>
    </div>
    <div class="row" v-if="user && user.uid == hostID">
      <div class="col-md-8"></div>
      <div class="col-md-4">
        <button class="btn btn-primary mr-1">
          Join
        </button>
        <button type="button" class="btn btn-danger mr-1">
          Leave
        </button>
      <h4 class="mt-2">Attendees</h4>
        <ul class="list-unstyled">
          <li>
            <span class="mr-2" title="On Air">
              <font-awesome-icon icon="podcast"></font-awesome-icon>
            </span>
            <span></span>
            <span class="pl-1"></span>
          </li>
        </ul>
        <div>
          <h5 class="mt-4">Pending</h5>
          <ul class="list-unstyled">
            <li class="mb-1" v-for="attendee in attendeeArr" :key="attendee.id">
              <span>
                <a type="button" class="mr-2" title="Approve attendee">
                  <font-awesome-icon icon="user"></font-awesome-icon>
                </a>
                <a type="button" class="text-secondary pr-1" title="Delete Attendee">
                  <font-awesome-icon icon="trash"></font-awesome-icon>
                </a>
              </span>
              <span class="pl-1">{{attendee.displayName}}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div v-else>
      <p class="lead">
        Hi <strong class="text-primary font-weight-bold"></strong>, you're currently in the room
        waiting for the owner of the chat to add you to the meeting. Please wait.
      </p>
    </div>
  </div>
</template>

<script>
import {FontAwesomeIcon} from '@fortawesome/vue-fontawesome'
import Firebase from 'firebase'
import db from '../db'
export default {
  data: function() {
    return {
      hostID: this.$route.params.hostID,
      roomID: this.$route.params.roomID,
      roomName: null,
      hostDisplayName: null,
      attendeeArr: []
    }
  },

  components:{
    FontAwesomeIcon
  },

  props: ['user'],

  mounted() {
    const roomRef = db.collection('users')
      .doc(this.$route.params.hostID)
      .collection('rooms')
      .doc(this.$route.params.roomID)

    //Get Room Name
    roomRef.get().then(roomDoc => {
      if(roomDoc.exists) this.roomName = roomDoc.data().name
      else this.$router.push('/')
    })

    //Get Host Name
    roomRef.collection('attendees').onSnapshot(snapShot => {
      let userCheckedIn = false
      let tempArr = []

      snapShot.forEach(attendeeDoc => {

        //Host Display Name
        if(this.hostID === attendeeDoc.id) {
          this.hostDisplayName = attendeeDoc.data().displayName
        }

        //Cheking If User Checked In
        if(this.user && this.user.uid == attendeeDoc.id){
          userCheckedIn = true
        }

        //Push all users to temp array
        tempArr.push({
          id: attendeeDoc.id,
          displayName: attendeeDoc.data().displayName
        })
      })

      if(!userCheckedIn){
        this.$router.push(`/checkin/${this.hostID}/${this.roomID}`)
      }

      this.attendeeArr = tempArr
    })
  }
}
</script>