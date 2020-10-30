<template>
    <div id="app">
        <v-textarea
          :outlined="true"
          :rows="10"
          v-model="innerMemoData"
          placeholder="この動画についてのメモが書けます。保存はリアルタイムにされます。"
        />
    </div>
</template>
<script>
  import axios from 'axios'
  export default {
    name: 'app',
    props: {
      memo: {
        type: String,
        required: true
      },
      videoId: {
        type: String,
        required: true
      }
    },
    computed: {
      innerMemoData: {
        get() {
          return this.$props.memo
        },
        set(memo) {
          let notifyMessage = this.notifyMessage
          let updateMemo = this.updateMemo
          axios.post('/apis/update_video_infos/', {
            video_id: this.videoId,
            memo: memo
          })
            .then(function(response) {
              // handle success
              console.log(response)
              updateMemo(memo)
              notifyMessage('メモを保存しました。');
            })
            .catch(function(error) {
              // handle error
              console.log(error)
              notifyMessage('メモを保存中にエラーが発生しました。')
            })
        }
      }
    },
    methods: {
      updateMemo: function(memo) {
        this.$emit('update-memo-event', memo)
      },
      notifyMessage: function(message) {
        this.$emit('notify-message-event', message)
      }
    }
  }
</script>
