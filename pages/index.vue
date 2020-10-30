<template>
  <v-app>
    <VTLoading
      :loading="loading"
    />
    <v-container>
      <div id="app">
        <v-card
          class="mx-auto"
          min-height="20vh"
        >

          <v-toolbar
            color="indigo"
            dark
          >

            <v-toolbar-title>VueTubePlayer</v-toolbar-title>
            <v-spacer></v-spacer>
            <VTFavoriteDialog
              :favoriteList="favoriteList"
              @update-memo-event="updateMemoHandler"
              @notify-message-event="notifyMessageHandler"
              @del-favorite-event="favoriteOrUnfavorite"
              @video-play-event="loadVideoByVideoId"
            />
            <VTSearchDialog
              @notify-message-event="notifyMessageHandler"
              @video-play-event="loadVideoByVideoId"
            />
          </v-toolbar>
          <div v-show="videoId===''">
            再生するビデオが選択されていません。<br/>
            右上の「<v-icon>mdi-star-circle</v-icon>お気に入り一覧」ボタンまたは、「<v-icon>mdi-cloud-search</v-icon>検索」ボタンから選択してください。
          </div>
          <youtube
            ref="youtube"
            :video-id="videoId"
            :fitParent="true"
            :player-vars="playerOptions"
            @playing="playing"
          />
          <div class="text-center"
               v-show="videoId"
          >
            <v-slider
              v-model="currentTime"
              @end="seekAt"
              :min="0"
              :max="videoDuration"
              inverse-label
              :label="sliderTime"
            />
            <v-btn class="ma-2" color="primary" @click="negative">-5s</v-btn>
            <v-btn class="ma-2" color="primary" @click="playOrPause">
              <v-icon>{{playPauseBtn}}</v-icon>
            </v-btn>
            <v-btn class="ma-2" color="primary" @click="positive">+5s</v-btn>
          </div>
          <div class="text-center"
               v-show="videoId"
          >
            <v-btn class="ma-2" color="primary" @click="favoriteOrUnfavorite(videoId)">
              <v-icon>{{favoriteBtn}}</v-icon>
              {{favoriteBtnName}}
            </v-btn>
            <br/>
            {{ position | formatTime}}
          </div>
          <VTPositionList
            :videoId="videoId"
            :positionList="positionList"
            @addPlayPosition="addPlayPosition"
            @delPlayPosition="delPlayPosition"
            @seekAt="seekAt"
            @negative="negative"
            @positive="positive"
            @notify-message-event="notifyMessageHandler"
          />
          <v-col
            v-show="videoId"
          >
            <VTVideoMemo :memo="specifiedMemo" :videoId="videoId"
                         @notify-message-event="notifyMessageHandler"/>
          </v-col>
          <VTSnackBar
            :snackbar="snackbar"
          />
        </v-card>
      </div>
      <v-footer padless>
        <v-col
          class="text-center"
          cols="12"
          v-show="videoId"
        >
          'https://www.youtube.com/watch?v={{ videoId }}'を再生中
        </v-col>
      </v-footer>
    </v-container>
  </v-app>
</template>

<script>
  import Vue from 'vue'
  import VueYoutube from 'vue-youtube'
  import axios from 'axios';
  import moment from 'moment';
  import VTVideoMemo from '~/components/VTVideoMemo';
  import VTSnackBar from '~/components/VTSnackBar';
  import VTSearchDialog from '~/components/VTSearchDialog';
  import VTFavoriteDialog from '~/components/VTFavoriteDialog';
  import VTPositionList from '~/components/VTPositionList';
  import VTLoading from '~/components/VTLoading';
  Vue.use(VueYoutube)
  export default {
    name: 'App',

    components: {
      VTVideoMemo,
      VTSnackBar,
      VTSearchDialog,
      VTFavoriteDialog,
      VTPositionList,
      VTLoading
    },
    data: () => ({
      loading: true,
      userId: 1,
      videoId: '',
      videoTitle: '',
      memo: '',
      currentTime: 0,
      position: null,
      videoDuration: 0,
      info: null,
      playerOptions: {
        fs: 0, // 全画面表示ボタンを非表示
        rel: 0, // 関連動画非表示
        controls: 0 // コントロールを非表示
      },
      lastQueryKeyWord:'',
      positionList: [],
      favoriteList: [],
      videoInfoList: [],
      snackbar:{
        popup: false,
        message: '',
      },
      playPauseBtn: 'mdi-play'
    }),
    computed: {
      player() {
        return this.$refs.youtube.player
      },
      sliderTime() {
        return this.fmtTime(this.currentTime) +'/'+ this.fmtTime(this.videoDuration)
      },
      favoriteBtn() {
        return this.getFavoriteIndex(this.videoId) === undefined ? 'mdi-star-outline' : 'mdi-star'
      },
      favoriteBtnName() {
        return this.getFavoriteIndex(this.videoId) === undefined ? '追加' : '追加済み'
      },
      specifiedMemo: function () {
        let videoId = this.videoId
        if (this.videoInfoList.length === 0) {
          return ''
        }
        let filteredList = this.videoInfoList.filter(function(position) {
          return position.video_id === videoId
        })
        if (filteredList.length > 0) {
          return filteredList[0].memo || ''
        }
        return ''
      },
    },
    /* eslint-disable no-console */
    methods: {
      loadVideoByVideoId: async function(videoId, videoTitle) {
        await axios.post('/apis/video_infos/', {
          user_id: this.userId,
          video_id: videoId,
          title: videoTitle,
          memo: ''
        });
        this.load();
        this.videoId = videoId
        this.videoTitle = videoTitle
        await this.player.loadVideoById(this.videoId)
        this.playPauseBtn = 'mdi-pause'
        this.player.playVideo();
      },
      fmtTime(value) {
        if (value===0) return '0:00';
        if (!value) return '';
        return value > 3600 ? moment()
            .set({'hour': 0, 'minute': 0, 'second': 0})
            .add(value, 'seconds')
            .format('H:m:ss') :
          moment()
            .set({'hour': 0, 'minute': 0, 'second': 0})
            .add(value, 'seconds')
            .format('m:ss');
      },
      seekAt(value) {
        this.player.seekTo(value, true);
      },
      async positive() {
        const currentTime = await this.player.getCurrentTime();
        this.player.seekTo(currentTime + 5, true);
      },
      async playOrPause() {
        if (await this.player.getPlayerState() === 1) {
          this.playPauseBtn = 'mdi-play'
          this.player.pauseVideo()
        } else {
          this.playPauseBtn = 'mdi-pause'
          this.player.playVideo()
        }
      },
      async savePosition() {
        const currentTime = await this.player.getCurrentTime();
        this.position = currentTime;
      },
      async replayAtSavedPosition() {
        if (this.position !== null) {
          this.player.seekTo(this.position);
        }
      },
      async negative() {
        const currentTime = await this.player.getCurrentTime();
        this.player.seekTo(currentTime - 5, true);
      },
      async playing() {
        this.currentTime = await this.player.getCurrentTime();
        this.videoDuration = await this.player.getDuration();
      },
      async load() {
        console.log(`API接続先：${process.env.PRIVATE_API_URL}`);

        await axios.get(`/apis/user_infos?user_id=${this.userId}`).then(res => {

          this.videoInfoList = res.data.video_infos;

          this.positionList = []
          for (let position of res.data.play_positions){
            this.pushPlayPosition(position);
          }

          this.favoriteList = []
          for (let favorite of res.data.fovorite_videos){
            this.favoriteList.push({
              id: favorite.video_info_id,
              videoId: favorite.video_id,
              videoTitle: favorite.title
            })
          }
          this.loading = false;
        });
      },
      pushPlayPosition: function(position)　{
        this.positionList.push({
          id: position.id,
          formatTime: this.fmtTime(position.position),
          position: position.position,
          videoId: position.video_id,
          memo: position.position_memo,
          userId: position.user_id,
        })
      },
      updateMemoHandler: function (memo) {
        let videoId = this.videoId
        if (this.videoInfoList.length === 0) {
          return ''
        }
        let filteredList = this.videoInfoList.filter(function(position) {
          return position.video_id === videoId
        })
        if (filteredList.length > 0) {
          filteredList[0].memo = memo
        }
      },
      notifyMessageHandler: function (message) {
        this.notifyBySnackBar(message);
      },

      addPlayPosition: async function()　{
        const currentTime = await this.player.getCurrentTime()
        let position = this.positionList.find(function(val) {
          return val.position === currentTime
        });
        if (position !== undefined) {
          this.notifyBySnackBar('この再生位置はすでに保存済みです。');
          return
        }
        this.pushPlayPosition(
          {
            id: -1, // 仮ID
            position: currentTime,
            user_id: this.userId,
            video_id: this.videoId,
            position_memo:''
          }
        )
        let notifyMessage = this.notifyBySnackBar
        let load = this.load
        await axios.post('/apis/play_positions/', {
          user_id: this.userId,
          video_id: this.videoId,
          position: currentTime,
        })
          .then(function(response) {
            // handle success
            console.log(response)
            notifyMessage('位置を保存しました。');
            load();
          })
          .catch(function(error) {
            // handle error
            console.log(error)
            notifyMessage('位置の保存中にエラーが発生しました。')
            load();
          })
      },
      delPlayPosition: async function(index, id)　{
        this.positionList.splice(index, 1)
        let notifyMessage = this.notifyBySnackBar
        await axios.delete(`/apis/play_positions/${id}`)
          .then(function(response) {
            // handle success
            console.log(response)
            notifyMessage('位置を削除しました。');
          })
          .catch(function(error) {
            // handle error
            console.log(error)
            notifyMessage('位置を削除中にエラーが発生しました。')
          })
        this.load();
      },
      getFavoriteIndex: function(videoId) {
        return this.favoriteList.find(function(val) {
          return val.videoId === videoId
        });
      },
      favoriteOrUnfavorite: async function(videoId) {
        let index = this.getFavoriteIndex(videoId);
        if (index === undefined) {
          let notifyMessage = this.notifyBySnackBar
          await axios.post('/apis/favorite_videos', {
            user_id: this.userId,
            video_id: this.videoId,
            video_title: this.videoTitle
          })
            .then(function(response) {
              // handle success
              console.log(response)
              notifyMessage('お気に入りに追加しました。');
            })
            .catch(function(error) {
              // handle error
              console.log(error)
              notifyMessage('お気に入りに追加中にエラーが発生しました。')
            })
          this.load();
        } else {
          let notifyMessage = this.notifyBySnackBar
          notifyMessage(videoId);
          await axios.post('/apis/del_favorite_videos', {
            user_id: this.userId,
            video_id: videoId
          })
            .then(function(response) {
              // handle success
              console.log(response)
              notifyMessage('お気に入りから削除しました。');
            })
            .catch(function(error) {
              // handle error
              console.log(error)
              notifyMessage('お気に入りから削除中にエラーが発生しました。')
            })
          this.load();
        }
      },
      notifyBySnackBar: function(message) {
        this.snackbar.popup = true;
        this.snackbar.message = message
        console.log(message);
      }
    },
    mounted() {
      this.userId = this.$route.query.user_id || 1
      setInterval(function () {
        this.playing();
      }.bind(this), 1000);

      this.load();
      setTimeout(() => {
        this.loading = false;
      }, 1000);
    },
    filters: {
      /* eslint-disable no-console */
      formatTime: function (value) {
        if (!value) return '';
        return value > 3600 ? moment()
            .set({'hour': 0, 'minute': 0, 'second': 0})
            .add(value, 'seconds')
            .format('H:m:ss') :
          moment()
            .set({'hour': 0, 'minute': 0, 'second': 0})
            .add(value, 'seconds')
            .format('m:ss');
      },
    }
  };
</script>
