<template>
    <div class="max-w-lg mx-auto p-6 bg-gray-800 text-white rounded-lg shadow-lg">
      <div class="play-box bg-gray-800 sticky top-0 pt-2 pb-1">
        <h1 class="mb-1 text-2xl font-semibold">{{ playlist.name || 'Audio Player'}}</h1>
        <div class="mb-2 flex justify-between">
          <div class="grow text-left"><small>{{ currentTrack ? currentTrack.name : ''}}</small></div>
          <div><small v-if="currentTime != '-'">{{ currentTime }} / </small> <small>{{ duration }}</small></div>
        </div>
        <div class="mb-3">
          <div class="progress-bar w-full h-4 bg-gray-700 rounded-full overflow-hidden" 
              ref="progress"
              :class="[trackLoading ? 'track-loading' : '']"
              @click="seek($event)">
            <div
              class="h-full bg-blue-500 transition-all"
              :style="{ width: progress + '%' }"
            ></div>
          </div>
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
      </div>
  
      <ul class="space-y-2">
        <li
          v-for="(track, index) in playlist.items"
          :key="index"
          :active="currentTrackIndex === index"
          @click="selectTrack(index)"
          class="flex justify-between track-item p-3 rounded cursor-pointer border-white hover:border"
          :class="[ currentTrackIndex === index ? 'bg-blue-600' : 'bg-gray-700', loadingTrack === index ? 'track-loading' : '']"
        >
          <div>{{ track.name }}</div>
          <div>{{ formatTime(track.duration) }}</div>
        </li>
      </ul>
      <p v-if="Object.keys(playlists).length" class="my-2">Playlists ({{ Object.keys(playlists).length }})</p>
      <ul class="flex pb-2" style="overflow-x: auto; gap: 0.5rem;">
        <li
          v-for="(plist, index) in playlists"
          :key="index"
          :active="plist.id == playlist.id"
          @click="setPlaylist(plist)"
          class="playlist-item p-3 rounded cursor-pointer border-white hover:border"
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
      <transition name="fade" >
        <div class="toast" v-if="toastMessage">{{ toastMessage }}</div>
      </transition>
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
        trackLoading: false,
        loadingTrack: -1,
        isPlaying: false,
        progress: 0,
        currentPlaylist: '',
        currentTime: "0:00",
        duration: "0:00",
        toastMessage: '',
        toastTimeout: null,
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
        const storedPlaylists = localStorage.getItem('playlists')
        if (storedPlaylists) {
          this.playlists = JSON.parse(storedPlaylists)
        }    
        try {
          const url = document.location.hash ? document.location.hash.substring(1) : ''
          const playlistURL = new URL(url)
          console.log(playlistURL)
          var plist = this.playlists.filter(p=>p.url==url)[0]
          if (plist){
            this.setPlaylist(plist)
          } else {
            this.loadPlaylistUrl(url)
          }
        } catch (e) {
          document.location = '#'
          const storedPlaylist = localStorage.getItem('playlist')
          if (storedPlaylist) {
            this.setPlaylist(JSON.parse(storedPlaylist))
          }
        }
    },
    methods: {
      toast(msg) {
        if (this.toastTimeout){
          clearTimeout(this.toastTimeout)
        }
        this.toastMessage = msg
        this.toastTimeout = setTimeout(() => {
          this.toastMessage = ''
        })
      },
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
            this.setPlaylist(result)
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
        this.currentPlaylist = plist.name
        document.title = plist.name
        document.location = '#' + plist.url
        this.$nextTick(() => {
          window.scrollTo(0,0)
        })
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
        if (this.currentTrackIndex < this.playlist.items.length - 1) {
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
        this.trackLoading = true
        this.loadingTrack = this.currentTrackIndex
        this.$refs.audio.src = this.currentTrack.url;
        this.$refs.audio.onloadedmetadata = (e) => {
            this.trackLoading = false
            this.loadingTrack = -1
            this.$refs.audio.play();
            this.isPlaying = true;
            document.title = this.currentTrack.name + ' - ' + this.playlist.name
            this.currentTrack.duration = this.$refs.audio.duration
            localStorage.setItem('playlist',JSON.stringify(this.playlist))
            localStorage.setItem('playlists',JSON.stringify(this.playlists))
        }
        this.$refs.audio.ontimeupdate = (e) => {
          console.log('Timeupdate',this.currentTrack.name)
        }
        this.$refs.audio.onended = (e) => {
          this.nextTrack()
        }
      }
    }
  };
  </script>
  
  <style scoped>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s ease, transform 0.3s ease;
  }
  
  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
    transform: scale(0.9);
  }
  .playlist-item{
    width: 200px;
    flex: 1 0 45%;
  }
  .track-item{
    text-align: left;
  }
  .progress-bar.track-loading{
    opacity: 0.6;
  }
  .track-item.track-loading{
    opacity: 0.6;
  }
  </style>