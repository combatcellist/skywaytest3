<template>
    <div id="app">
        <div class="container-video">
    <div id="timer">00:00:000</div>
    <div class="table-center">
      <div>
        <p id="p1">あなたの勝ちです!!!</p>
        <p id="p2">残念。。。！</p>
      </div>
      <video id="their-video" width="200" preload="auto" autoplay="true" muted="muted" volume="0">
      <video id="my-video" width="200" preload="auto" autoplay="true" muted="muted" volume="0">
      <!-- <video id="video" preload="auto" autoplay="true" muted="muted" volume="0"> -->
      <!-- <source src="./movies/tired.mov"> -->
    </div>
  </div>
        <!-- <video id="their-video" width="200" autoplay playsinline></video> -->
        <!-- <video id="my-video" muted="true" width="500" autoplay playsinline></video> -->
        <p>Your Peer ID: <span id="my-id">{{peerId}}</span></p>
        <input v-model="calltoid" placeholder="call id">
        <button @click="makeCall" class="button--green">Call</button>
        <br />

        マイク:
        <select v-model="selectedAudio" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(audio, key, index) in audios" v-bind:key="index" :value="audio.value">
            {{ audio.text }}
          </option>
        </select>

        カメラ: 
        <select v-model="selectedVideo" @change="onChange">
          <option disabled value="">Please select one</option>
          <option v-for="(video, key, index) in videos" v-bind:key="index" :value="video.value">
            {{ video.text }}
          </option>
        </select>

    </div>
<input type="button" value="Win" onclick="clickBtn1()" />
<input type="button" value="Lose" onclick="clickBtn2()" />
<button id="start">開始</button>
<button id="stop">停止</button>
<button id="reset">クリア</button>
</template>



<script>
const API_KEY = "b5072df8-3b06-489a-b689-5e8ffbb4e895"; 
// const Peer = require('../skyway-js');
console.log(Peer)
export default {
    data: function () {
        return {
            audios: [],
            videos: [],
            selectedAudio: '',
            selectedVideo: '',
            peerId: '',
            calltoid: '',
            localStream: {}
        }
    },
    methods: {
        onChange: function(){
            if(this.selectedAudio != '' && this.selectedVideo != ''){
                this.connectLocalCamera();
            }
        },

        connectLocalCamera: async function(){
            const constraints = {
                audio: this.selectedAudio ? { deviceId: { exact: this.selectedAudio } } : false,
                video: this.selectedVideo ? { deviceId: { exact: this.selectedVideo } } : false
            }

            const stream = await navigator.mediaDevices.getUserMedia(constraints);
            document.getElementById('my-video').srcObject = stream;
            this.localStream = stream;
        },

        makeCall: function(){
            const call = this.peer.call(this.calltoid, this.localStream);
            this.connect(call);
        },

        connect: function(call){
            call.on('stream', stream => {
                const el = document.getElementById('their-video');
                el.srcObject = stream;
                el.play();
            });
        },
        clickBtn1: function () {
            const p1 = document.getElementById("p1");
            if(p1.style.display=="block"){
                // noneで非表示
                p1.style.display ="none";
            }else{
                // blockで表示
                p1.style.display ="block";
            }
        },
        clickBtn2: function () {
            const p1 = document.getElementById("p2");
            if(p2.style.display=="block"){
                // noneで非表示
                p2.style.display ="none";
            }else{
                // blockで表示
                p2.style.display ="block";
            }
        }
    },

    mounted: async function () {
        document.getElementById("p1").style.display ="none";
        document.getElementById("p2").style.display ="none";
    },
    
    created: async function(){
        console.log(API_KEY)
        this.peer = new Peer({key: API_KEY, debug: 3}); //新規にPeerオブジェクトの作成
        this.peer.on('open', () => this.peerId = this.peer.id); //PeerIDを反映
        this.peer.on('call', call => {
            call.answer(this.localStream);
            this.connect(call);
        });

        //デバイスへのアクセス
        const deviceInfos = await navigator.mediaDevices.enumerateDevices();

        //オーディオデバイスの情報を取得
        deviceInfos
        .filter(deviceInfo => deviceInfo.kind === 'audioinput')
        .map(audio => this.audios.push({text: audio.label || `Microphone ${this.audios.length + 1}`, value: audio.deviceId}));

        //カメラの情報を取得
        deviceInfos
        .filter(deviceInfo => deviceInfo.kind === 'videoinput')
        .map(video => this.videos.push({text: video.label || `Camera  ${this.videos.length - 1}`, value: video.deviceId}));

        console.log(this.audios, this.videos);        
    }
}
</script>

<style scoped>
    p {
    font-size: 2em;
    text-align: center;
    }
</style>

