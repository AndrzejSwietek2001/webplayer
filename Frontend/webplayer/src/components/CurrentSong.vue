<template>
  <div class="currentSong">
    <div class="bg">
        <img src="https://c4.wallpaperflare.com/wallpaper/531/951/621/digital-digital-art-artwork-illustration-minimalism-hd-wallpaper-preview.jpg" />
    </div>
    <div class="main">
      <div class="image">
        <img src="https://c4.wallpaperflare.com/wallpaper/531/951/621/digital-digital-art-artwork-illustration-minimalism-hd-wallpaper-preview.jpg" alt="">
      </div>
      <div id="player">
        <div id="songTitle"> {{this.$store.getters.getNowPlaying}}</div>
        <div id="buttons">
          <div id="prev" @click="prevSong"><i class="fa fa-arrow-left"></i></div>
          <div id="pause" @click="pause" v-if="this.$store.state.isPlaying" ><i class="fa fa-pause"></i></div>
          <div id="play" @click="play" v-else><i class="fa fa-play"></i></div>
          <div id="next" @click="nextSong"><i class="fa fa-arrow-right"></i></div>
        </div>
        <audio id="audio" controls>
          <source src="http://localhost:3000/Ed%20Sheeran%20-%20Divide%20(2017)/Divide%20-%20Perfect%20-%20Ed%20Sheeran.mp3"
                   id="audio_src"
                   type="audio/mp3" />
        </audio>
        <SongProgress></SongProgress>
        <div id="seek-bar">
          <div id="fill"></div>
          <div id="handle"></div>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
import SongProgress from "./SongProgress";
import axios from "axios";
import querystring from "querystring";

export default {
  name: "CurrentSong",
  components:{
    SongProgress
  },
  data(){
    return {
      isPlaying: false,
      path_to_mp3: '',
      mp3_src: "http://localhost:3000/"+this.path_to_mp3
    }
  },
  methods: {
    play: function () {
      document.getElementById("audio").play(); // graj plik mp3
      this.$store.commit('TOGGLE_IS_PLAYING', true);
      console.log(this.$store.state.isPlaying);
    },
    pause: function () {
      document.getElementById("audio").pause(); // pauzuj granie
      this.$store.commit('TOGGLE_IS_PLAYING', false);
    },
    getDuration: async function() {
      return new Promise(function(resolve) {
        let audio = document.getElementById("audio")
        audio.addEventListener("loadedmetadata", function(){
          resolve(audio.duration);
        });
      });
    },
    nextSong: async function() {
      document.getElementById("audio").pause();
      let payload = {
        changeSong: true,
        currentSong: decodeURI(document.getElementById("audio_src").src).split('/')[4],
        returnSong: 'next'
      }
      // let res = await this.$store.dispatch('fetchNextPrevSong', payload)
      let res = await axios.post('http://localhost:3000/', querystring.stringify({ changeSong: true, currentSong: payload.currentSong, returnSong: payload.returnSong }) );
      this.mp3_src = "http://localhost:3000/"+res.data
      console.log('next',res.data);
      document.getElementById("audio_src").src = this.mp3_src;
      document.getElementById("audio").load();
      document.getElementById("audio").play();

      document.getElementById('songProgress').min = 0;
      document.getElementById('songProgress').value = 0;
      document.getElementById('songProgress').step = 1;

      this.$store.commit("NOW_PLAYING",decodeURI(document.getElementById("audio_src").src).split('/')[4].split('-')[1] );
      this.$store.commit('TOGGLE_IS_PLAYING', true)

      let duration = await this.getDuration()
      this.$store.commit("SET_TIMER", {max: duration, current: 0});
      document.getElementById('songProgress').max = Math.floor(duration).toString();
      console.log(duration)
    },
    prevSong: async function () {
      document.getElementById("audio").pause();
      let payload = {
        changeSong: true,
        currentSong: decodeURI(document.getElementById("audio_src").src).split('/')[4],
        returnSong: 'prev'
      }
      console.log('prev',payload);
      // let res = await this.$store.dispatch('fetchNextPrevSong', payload)
      let res = await axios.post('http://localhost:3000/', querystring.stringify({ changeSong: true, currentSong: payload.currentSong, returnSong: payload.returnSong }) );
      this.mp3_src = "http://localhost:3000/" + res.data
      console.log('next', payload, res);
      document.getElementById("audio_src").src = this.mp3_src;
      document.getElementById("audio").load();
      document.getElementById("audio").play();

      document.getElementById('songProgress').min = 0;
      document.getElementById('songProgress').value = 0;
      document.getElementById('songProgress').step = 1;
      this.$store.commit("NOW_PLAYING",decodeURI(document.getElementById("audio_src").src).split('/')[4].split('-')[1] );
      this.$store.commit('TOGGLE_IS_PLAYING', true)

      let duration = await this.getDuration()
      this.$store.commit("SET_TIMER", {max: duration, current: 0});
      document.getElementById('songProgress').max = Math.floor(duration).toString();
      console.log(duration)
    }
  },
  mounted() {
    this.path_to_mp3 = 'Ed%20Sheeran%20-%20Divide%20(2017)/Divide%20-%20Perfect%20-%20Ed%20Sheeran.mp3'
    this.mp3_src = "http://localhost:3000/"+this.path_to_mp3
    console.log(this.mp3_src)
    document.getElementById("audio_src").src = this.mp3_src;
    document.getElementById("audio").load(); // UWAGA - dopiero w tym momencie powinna być możliwość wylogowania GET-a danego pliku mp3 na serwerze
    document.getElementById("audio").onended = function(e) {
      console.log("utwór się zakończył, odtwarzam następny",e)
      return this.nextSong()
    }.bind(this);
  }
}
</script>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@200&display=swap');
  .currentSong {
    width: 70%;
    width: 100%;
    height: 90vh;
    background: #000;
    font-family: 'Nunito', sans-serif;
  }
  .bg {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
  .bg img {
    height: 100%;
    width: 100%;
    -webkit-filter: blur(20px);
    position: relative;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    z-index: 1;
    background-color: #fff;
  }

  .main {
    width: 25%;
    height: auto;
    background: #fff;
    border-radius: 15px;

    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);

    z-index: 99999;
  }
  .image img {
    height: 180px;
    width: 100%;
    margin-top: -5px;
    border-top-left-radius: 15px;
    border-top-right-radius: 15px;
  }

  #player {
    padding: 25px;
    color: #333;
  }

  #songTitle {
    font-family: monospace;
    font-size: 18px;
    position: absolute;
    top: 55%;
    left: 50%;
    transform: translate(-50%,-50%);
  }

  #buttons {
    height: 50px;
    width: 100%;
    margin-top: 75px;
    position: relative;
    top: 10%;
    left: 50%;
    transform: translate(-50%,-50%);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  #next,
  #prev {
    width: 3rem;
    height: 3rem;
    cursor: pointer;
    border-radius: 50%;
    background-color: #414141;
    display: flex;
    align-items: center;
    justify-content: center;
    color: azure;
    transition: .5s;
  }
  #prev:hover,
  #next:hover {
    background-color: #2b2b2b;
  }

  #play,#pause {
    width: 3rem;
    height: 3rem;
    cursor: pointer;
    border-radius: 50%;
    border: 3px solid #414141;;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: .5s;
    margin: 0 1rem;
  }
  #play:hover {
    border: 3px solid #00ea02;;
    color: #00ea02;
  }
  #pause:hover {
    border: 3px solid #dd2450;;
    color: #dd2450;
  }
  audio {
    display: none;
  }

  @media (max-width: 1024px) {
    .main {
      width: 75%;
    }
  }
</style>
