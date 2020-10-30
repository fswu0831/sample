<template>

  <div >
    <div v-show="filteredPositionList.length">再生位置一覧({{filteredPositionList.length}})</div>
    <v-list>
      <v-list-item
        v-for="(position, index) in filteredPositionList"
        :key="index"
      >
        <v-list-item-content>
          <v-list-item-title v-text="positionDisplayName(position)" @click="seekAt(position.position)"/>
        </v-list-item-content>

        <VTPositionMemoDialog
          :position="position"
          @notify-message-event="notifyMessage"
        />

        <v-list-item-icon>
          <v-icon color="pink" @click="delPlayPosition(index, position.id)">mdi-delete</v-icon>
        </v-list-item-icon>
      </v-list-item>
    </v-list>
    <v-row justify="center" v-show="videoId">
      <v-btn class="ma-2" color="primary" @click="negative">-5s</v-btn>
      <v-btn class="ma-2" color="primary" @click="addPlayPosition">
        <v-icon>mdi-arrow-collapse-down</v-icon>
        位置保存
      </v-btn>
      <v-btn class="ma-2" color="primary" @click="positive">+5s</v-btn>
    </v-row>
  </div>

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
        positionList: {
          type: Array,
          required: true
        },
        videoId: {
          type: String,
          required: true
        }
      },
      computed: {
        filteredPositionList: function() {
          let videoId = this.videoId
          return this.positionList.filter(function(position) {
            return position.videoId === videoId
          }).sort(function(a, b) {
            return a.position - b.position
          })
        },
      },
      methods: {
        positionDisplayName: function(position) {
          if (position.memo) {
            return position.formatTime + ' ' + position.memo
          }
          return position.formatTime
        },
        delPlayPosition: async function(index, id)　{
          this.$emit('delPlayPosition', index, id);
        },
        addPlayPosition: async function(index, id)　{
          this.$emit('addPlayPosition', index, id);
        },
        seekAt(value) {
          this.$emit('seekAt', value);
        },
        negative() {
          this.$emit('negative');
        },
        positive() {
          this.$emit('positive');
        },
        notifyMessage(message) {
          this.$emit('notify-message-event', message)
        },
      }
    }
</script>
<style scoped>
  .v-list {
    max-height: 180px;
    overflow-y: auto
  }
</style>
