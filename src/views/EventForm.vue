<template>
  <div>
    <h1>Create an event</h1>
    <form @submit.prevent="saveEvent">
      <BaseInput
        v-model="event.category"
        type="text"
        label="Category"
        class="field"
      />

      <h3>Name & describe your event</h3>

      <BaseInput v-model="event.title" type="text" label="Title" />

      <BaseInput v-model="event.description" type="text" label="Description" />

      <h3>Where is your event?</h3>

      <label>Location</label>

      <BaseInput v-model="event.location" type="text" label="Location" />

      <h3>Who is your organizer?</h3>

      <BaseSelect
        :options="GStore.organizers"
        v-model="event.organizer.id"
        label="Select an Organizer"
      />
      <h3>The image of the Event</h3>
      <UploadImages @changed="handleImages" />

      <button type="submit">Submit</button>
    </form>

    <pre>{{ event }}</pre>
  </div>
</template>

<script>
import EventService from '@/services/EventService.js'
import UploadImages from 'vue-upload-drop-images'
export default {
  inject: ['GStore'],
  components: {
    UploadImages
  },
  data() {
    return {
      event: {
        category: '',
        title: '',
        description: '',
        location: '',
        organizer: { id: '', name: '' },
        imageUrls: []
      },
      files: []
    }
  },
  methods: {
    saveEvent() {
      Promise.all(
        this.files.map((file) => {
          return EventService.uploadFile(file)
        })
      ).then((response) => {
        //console.log(response)
        //console.log('finish upload file')
        this.event.imageUrls = response.map((r) => r.data)
        EventService.saveEvent(this.event)
          .then((response) => {
            console.log(response)
            this.$router.push({
              name: 'EventDetails',
              params: { id: response.data.id }
            })
            this.GStore.flashMessage =
              'You are succcessfully add a new event for ' + response.data.title
            setTimeout(() => {
              this.GStore.flashMessage = ''
            }, 3000)
          })
          .catch(() => {
            this.$router.push('NetworkError')
          })
      })
    },
    handleImages(files) {
      //console.log(files)
      this.files = files
    }
  }
}
</script>
<style scoped></style>
