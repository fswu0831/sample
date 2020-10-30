<template>
  <div id="app">
    <v-dialog v-model="dialog" hide-overlay transition="scroll-x-transition">
      <template v-slot:activator="{ on }">
        <v-btn class="ma-2" color="primary" dark v-on="on">
          <v-icon>mdi-cloud-search</v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-toolbar dark color="primary">
          <v-btn icon dark @click="dialog = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
          <v-toolbar-title>YouTube検索</v-toolbar-title>
          <v-spacer></v-spacer>
        </v-toolbar>
        <div class="content">
          <v-form @submit.prevent="submitForm">
          <v-container>
            <v-row>
              <v-text-field
                v-model="keyword"
                autofocus
                label="検索キーワード"
                outlined
                :append-outer-icon="keyword ? 'mdi-subdirectory-arrow-right' : 'mdi-search-web'"
              />
            </v-row>
            <v-row v-show="info.data">
              <v-btn class="ma-2" color="primary" dark @click="goToPage(info.data.prevPageToken)">
                <v-icon>mdi-skip-previous</v-icon>
                前へ
              </v-btn>
              <v-btn class="ma-2" color="primary" dark @click="goToPage(info.data.nextPageToken)">
                次へ
                <v-icon>mdi-skip-next</v-icon>
              </v-btn>
            </v-row>
          </v-container>
          </v-form>
          <div v-show="info.length===0">検索キーワードを入力すると検索結果を表示します。</div>
          <h2 class="h2 mb-4 pl-3 py-2" v-show="info.length > 0">YouTube検索結果</h2>
          <div v-for="i in info" class="search-results">
            <v-card
              class="mx-auto"
              v-for="item in i.items"
              v-on:click="loadVideoByVideoId(item.id.videoId,item.snippet.title)"
            >
              <v-img
                height="200px"
                :src="item.snippet.thumbnails.medium.url"
                :alt="item.snippet.title"
              />
              <v-card-title>
                {{item.snippet.title}}
              </v-card-title>

              <v-card-subtitle>
                {{item.snippet.description}}
              </v-card-subtitle>
              <v-card-subtitle>
                公開日：{{item.snippet.publishedAt | formatDate}}
              </v-card-subtitle>
            </v-card>
          </div>
        </div>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
  import axios from 'axios'
  import moment from 'moment'

  export default {
    name: 'app',
    data: () => ({
      searchText: '',
      keyword: '',
      dialog: false,
      api: 'https://www.googleapis.com/youtube/v3/search',
      info: [],
      params: {
        type: 'video',
        part: 'snippet',
        q: '曲ランキング',
        order: 'viewCount',
        pageToken: '',
        key: 'AIzaSyAEskjxwuZRqBG9XZeQRidp53_NFSYFrDw'
      }
    }),
    methods: {
      loadVideoByVideoId: async function(videoId, videoTitle) {
        this.dialog = false;
        this.$emit('video-play-event', videoId, videoTitle)
      },
      search() {
        console.log(`YouTube検索します。検索キーワード：${this.keyword}`)
        this.params.q = this.keyword
        const params = this.params
        axios.get(this.api, { params })
          .then(response => {
            this.info = response
          })
          .catch(error => {
            const responseError = error.response.data.error
            const errorResponse = responseError.errors && responseError.errors.length > 0 ? `${responseError.errors[0].code}:${responseError.errors[0].message}` : ''
            this.$emit('notify-message-event', `YouTube APIからエラーが返りました。${errorResponse}`)
            console.log(error.response)
          })
      },
      submitForm(ev) {
        this.search();
        ev.preventDefault();
      },
      goToPage:function(token){
        this.params.pageToken = token
        const params = this.params
        axios.get(this.api, { params })
          .then(response => {
            this.info = response
          })
      },
    },
    filters: {
      formatDate: function(value) {
        if (!value) return '';
        return moment(value)
          .format('YYYY/M/D')
      },
    }
  }
</script>
<style scoped>
  .search-results {
    max-height: 500px;
    overflow-y: auto
  }
</style>
