<template>
  <v-dialog v-model="loading" fullscreen>
    <v-container fluid fill-height style="background-color: rgba(255, 255, 255, 0.5);">
      <v-layout justify-center align-center>
        <v-progress-circular
          indeterminate
          color="red"
        >
        </v-progress-circular>
      </v-layout>
    </v-container>
  </v-dialog>
</template>
<script>
    import axios from 'axios'
    import VTPositionMemoDialog from '~/components/VTPositionMemoDialog';
    export default {
      name: 'app',
      components: {
        VTPositionMemoDialog
      },
      props: {
        loading: {
          type: Boolean,
          required: true
        }
      },
    }
</script>
