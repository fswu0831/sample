<template>
    <div id="app">
      <v-snackbar
        v-model="snackbar.popup"
        :bottom="true"
        color="error"
        :timeout="4000"
      >
        {{ snackbar.message }}
        <v-btn
          dark
          text
          @click="snackbar.popup = false"
        >
          Close
        </v-btn>
      </v-snackbar>
    </div>
</template>
<script>
    export default {
      name: 'app',
      props: {
        snackbar: {
          type: Object,
          required: true
        },
      }
    }
</script>
