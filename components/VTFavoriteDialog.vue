<template>
  <div id="app">
    <v-dialog v-model="dialog" hide-overlay transition="scroll-x-transition">
      <template v-slot:activator="{ on }">
        <v-btn class="ma-2" color="primary" dark v-on="on">
          <v-icon>mdi-star-circle</v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-toolbar dark color="primary">
          <v-btn icon dark @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>お気に入り一覧</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <div class="content">
          <div v-if="favoriteList.length===0">
            お気に入りが登録されていません。
          </div>
          <v-list v-if="favoriteList.length">
            <div>お気に入り一覧</div>
            <v-list-item
              v-for="(favorite, index) in favoriteList"
              :key="index"
            >
              <v-list-item-content>
                <v-list-item-title v-text="favorite.videoTitle" @click="loadVideoByVideoId(favorite.videoId)"></v-list-item-title>
              </v-list-item-content>
              <v-list-item-icon>
                <v-icon color="pink" @click="unfavorite(favorite.videoId)">mdi-delete</v-icon>
              </v-list-item-icon>
            </v-list-item>
          </v-list>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
  import axios from 'axios'

  export default {
    name: 'app',
    data: () => ({
      dialog: false,
    }),
    props: {
      favoriteList: {
        type: Array,
        required: true
      },
    },
    computed: {
    },
    methods: {
      loadVideoByVideoId: function(videoId, videoTitle) {
        this.dialog = false;
        this.$emit('video-play-event', videoId, videoTitle)
      },
      unfavorite: function(videoId) {
        this.$emit('del-favorite-event', videoId)
      }
    },
  }
</script>
