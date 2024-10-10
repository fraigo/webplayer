<template>
    <div class="max-w-lg mx-auto p-6 bg-gray-800 text-white rounded-lg shadow-lg">
      <h1 class="text-2xl font-semibold mb-4">Audio Player</h1>
      
      <div class="mb-6">
        <div class="w-full h-4 bg-gray-700 rounded-full overflow-hidden" 
            ref="progress"
            @click="seek($event)">
          <div
            class="h-full bg-blue-500 transition-all"
            :style="{ width: progress + '%' }"
          ></div>
        </div>
        <p class="text-sm mt-2">{{ currentTime }} / {{ duration }}</p>
      </div>
      
      <div class="flex justify-center space-x-4 mb-6">
        <button
          @click="prevTrack"
          class="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded"
        ><img width="24" src="/assets/prev.svg"></button>
        <button
          @click="togglePlayPause"
          class="px-4 py-2 bg-green-600 hover:bg-green-700 rounded"
        >
        <img v-if="isPlaying" width="24" src="/assets/pause.svg">
        <img v-else width="24" src="/assets/play.svg"></button>
        <button
          @click="nextTrack"
          class="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded"
        ><img width="24" src="/assets/next.svg"></button>
        <button
          @click="dialog=true"
          class="px-4 py-2 bg-blue-600 hover:bg-blue-700 rounded"
        >Open</button>
      </div>
  
      <ul class="space-y-2">
        <li
          v-for="(track, index) in playlist.items"
          :key="index"
          :active="currentTrackIndex === index"
          @click="selectTrack(index)"
          class="p-3 rounded cursor-pointer border-white hover:border"
          :class="[ currentTrackIndex === index ? 'bg-blue-600' : 'bg-gray-700']"
        >
          {{ track.name }} ({{ formatTime(track.duration) }})
        </li>
      </ul>
      <p v-if="Object.keys(playlists).length" class="my-2">Playlists ({{ Object.keys(playlists).length }})</p>
      <ul class="space-y-2">
        <li
          v-for="(plist, index) in playlists"
          :key="index"
          :active="plist.id == playlist.id"
          @click="setPlaylist(plist)"
          class="p-3 rounded cursor-pointer border-white hover:border"
          :class="[ plist.id == playlist.id ? 'bg-blue-600' : 'bg-gray-700']"
        >
          {{ plist.name }} ({{ plist.items.length }} songs)
        </li>
      </ul>
      <PromptDialog 
        @close="dialog=false"
        @change="loadPlaylistUrl($event)"
        :visible="dialog">
        Enter a URL to parse contents:
      </PromptDialog>
      <audio ref="audio" :src="currentTrack.url" @timeupdate="updateProgress" @loadedmetadata="updateDuration"></audio>
    </div>
  </template>
  
  <script setup>
  import PromptDialog from './PromptDialog.vue';
  import MD5 from '../js/md5.js'
  </script>
  <script>
  
  export default {
    data() {
      return {
        playlist: {
          items: [],
        },
        playlists: {},
        currentTrackIndex: 0,
        isPlaying: false,
        progress: 0,
        currentPlaylist: '',
        currentTime: "0:00",
        duration: "0:00",
        dialog: false,
        parserUrl : 'https://franciscoigor.me/projects/playlist-generator/?url={url}',
      };
    },
    components: [
      PromptDialog
    ],
    computed: {
      currentTrack() {
        return this.playlist.items[this.currentTrackIndex] || {};
      }
    },
    mounted: function() {
        window.content = this
        const storedPlaylist = localStorage.getItem('playlist')
        if (storedPlaylist) {
          this.playlist = JSON.parse(storedPlaylist)
          document.title = this.playlist.name
        }
        const storedPlaylists = localStorage.getItem('playlists')
        if (storedPlaylists) {
          this.playlists = JSON.parse(storedPlaylists)
        }    
    },
    methods: {
      loadPlaylistUrl(url){
        console.log(url)
        const urlObj = new URL(url);
        let name = urlObj.pathname.replace(/\/$/,'').split('/').pop() 
        if (name) name = decodeURI(name)
        var link = this.parserUrl.replace('{url}',encodeURIComponent(url))
        this.loadList(link, name)
      },
      loadList(url,name) {
        fetch(url)
          .then(r => r.json())
          .then(result => {
          if (result && result.items){
            const urlObj = new URL(url);
            result.name = name || result.name || 'Playlist from ' + urlObj.hostname
            result.id = result.id || MD5(url)
            this.playlist = result
            this.currentPlaylist = result.name
            localStorage.setItem('playlist',JSON.stringify(this.playlist))
            this.playlists[result.id] = this.playlist
            localStorage.setItem('playlists',JSON.stringify(this.playlists))
            document.title = result.name
          }
        })
      },
      setPlaylist(plist) {
        this.currentTrackIndex = 0 
        this.playlist = plist
        document.title = plist.name
      },
      togglePlayPause() {
        const audio = this.$refs.audio;
  
        if (this.isPlaying) {
          audio.pause();
        } else {
          audio.play();
        }
  
        this.isPlaying = !this.isPlaying;
      },
      updateProgress() {
        const audio = this.$refs.audio;
        this.progress = (audio.currentTime / audio.duration) * 100;
        this.currentTime = this.formatTime(audio.currentTime);
      },
      updateDuration() {
        const audio = this.$refs.audio;
        this.duration = this.formatTime(audio.duration);
      },
      formatTime(seconds) {
        if (!seconds) return '-'
        const minutes = Math.floor(seconds / 60);
        const secs = Math.floor(seconds % 60);
        return `${minutes}:${secs < 10 ? "0" : ""}${secs}`;
      },
      seek(event) {
        const audio = this.$refs.audio;
        const percentage = Math.round(event.offsetX * 100 / this.$refs.progress.scrollWidth) / 100
        audio.currentTime = Math.round (percentage * 10 * audio.duration) /10
        console.log(audio.currentTime)
      },
      nextTrack() {
        if (this.currentTrackIndex < this.playlist.length - 1) {
          this.currentTrackIndex++;
          this.isPlaying = false;
          this.playTrack();
        }
      },
      prevTrack() {
        if (this.currentTrackIndex > 0) {
          this.currentTrackIndex--;
          this.isPlaying = false;
          this.playTrack();
        }
      },
      selectTrack(index) {
        this.currentTrackIndex = index;
        this.isPlaying = false;
        this.playTrack();
      },
      playTrack() {
        console.log('play',this.currentTrack)
        this.$refs.audio.src = this.currentTrack.url;
        this.$refs.audio.onloadedmetadata = (e) => {
            this.$refs.audio.play();
            this.isPlaying = true;
            document.title = this.currentTrack.name + ' - ' + this.playlist.name
            this.currentTrack.duration = this.$refs.audio.duration
            localStorage.setItem('playlist',JSON.stringify(this.playlist))
            localStorage.setItem('playlists',JSON.stringify(this.playlists))
        }
        this.$refs.audio.onended = (e) => {
          this.nextTrack()
        }
      }
    }
  };
  </script>
  
  <style scoped>
  /* Add any custom styles if necessary */
  </style>