<template>
  <div id="app">
    <v-dialog v-model="dialog" hide-overlay transition="scroll-x-transition">
      <template v-slot:activator="{ on }">
        <v-list-item-icon @click="init" v-on="on">
          <v-icon color="blue">mdi-pencil</v-icon>
        </v-list-item-icon>
      </template>
      <v-card>
        <v-toolbar dark color="primary">
          <v-btn icon dark @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>位置メモ</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <div class="content">
          <v-form @submit.prevent="savePositionMemo">
            <v-container>
              <v-row>
                <v-text-field
                  v-model="memo"
                  autofocus
                  :label="position.formatTime"
                  outlined
                  hide-details
                />
              </v-row>
              <v-row>
                <v-btn class="ma-2" color="primary" dark @click="savePositionMemo">
                  <v-icon>mdi-content-save</v-icon>保存
                </v-btn>
                <v-btn class="ma-2" color="primary" dark @click="dialog = false">
                  <v-icon>mdi-close-circle</v-icon>閉じる
                </v-btn>
              </v-row>
            </v-container>
          </v-form>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
  import axios from 'axios'

  export default {
    name: 'app',
    props: {
      position: {
        type: Object,
        required: true
      },
    },
    data: () => ({
      dialog: false,
      memo:''
    }),
    methods: {
      init() {
        this.memo = this.position.memo
      },
      loadVideoByVideoId: async function(videoId, videoTitle) {
        this.dialog = false;
        this.$emit('video-play-event', videoId, videoTitle)
      },
      savePositionMemo: async function() {
        let notifyMessage = this.notifyMessage
        let position = this.position
        let memo = this.memo
        await axios.post('/apis/play_positions/', {
          user_id: this.position.userId,
          video_id: this.position.videoId,
          position: this.position.position,
          memo: this.memo,
        })
          .then(function(response) {
            // handle success
            position.memo = memo
            notifyMessage('位置のメモを保存しました。');
          })
          .catch(function(error) {
            // handle error
            console.log(error)
            notifyMessage('位置のメモの保存中にエラーが発生しました。')
          })

        this.dialog = false;
      },
      notifyMessage(message) {
        this.$emit('notify-message-event', message)
      },
      submitForm(ev) {
        this.search();
        ev.preventDefault();
      }
    }
  }
</script>
