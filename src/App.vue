<template>
  <div id="app">
    <div class="audioplayer" v-for="s in soundData" :key="s.id">
      <!--     <label for="">{{s.number}}</label> -->
      <audio v-bind:data-num="s.number" preload="auto">
        <source v-bind:src="s.url" type="audio/ogg" />
      </audio>
    </div>
    <div class="center_box">
      <h2>Vue.js Piano Project 7</h2>
      <div class="keyboard">
        <div class="pianokey" v-for="s in display_keys" :key="s.id">
          <div
            class="white"
            v-if="s.type == 'white'"
            v-on:click="addnote(s.num)"
            v-bind:class="get_current_highlight(s.num, s.key) ? 'playing' : ''"
          >
            <div class="label">{{ String.fromCharCode(s.key) }}</div>
          </div>
          <div
            class="black"
            v-if="s.type == 'black'"
            v-on:click="addnote(s.num)"
            v-bind:class="get_current_highlight(s.num, s.key) ? 'playing' : ''"
          >
            <div class="label">{{ String.fromCharCode(s.key) }}</div>
          </div>
        </div>
      </div>
      <div class="controls">
        <ul class="note_list" v-if="notes.length > 0">
          <li
            v-for="(note, id) in notes"
            v-bind:class="now_note_id - 1 == id ? 'playing' : ''"
            :key="id"
          >
            <div class="num">{{ note.num }}</div>
            <div class="time">{{ note.time }}</div>
          </li>
        </ul>
      </div>
      <div class="buttons">
        <button v-on:click="load_sample">Sample</button>
        <button v-on:click="playnext(1)">Playnext</button>
        <button v-on:click="startplay()" v-if="playing_time <= 1" class="abc">
          <div>Startplay</div>
          <div class="arrow-right"></div>
        </button>
        <button v-on:click="stopplay()" v-if="playing_time > 1">
          Stopplay
        </button>
        <button v-on:click="notes = []">Clear</button>
        <button v-on:click="start_record()" v-if="record_time <= 0">
          Startrecord
        </button>
        <button v-on:click="stop_record()" v-if="record_time > 1">
          Stoprecord
        </button>
      </div>
      <h4>{{ playing_time + record_time }}</h4>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "piano",
  data() {
    return {
      soundData: [],
      notes: [
        { id: 1,num: 3, time: 105 },
        { id: 2,num: 3, time: 223 },
        { id: 3,num: 4, time: 331 },
        { id: 4,num: 5, time: 482 },
        { id: 4,num: 5, time: 565 },
        { id: 5,num: 4, time: 673 },
        { id: 6,num: 3, time: 782 },
        { id: 7,num: 2, time: 893 },
        { id: 8,num: 1, time: 1006 },
        { id: 9,num: 1, time: 1113 },
        { id: 10,num: 2, time: 1220 },
        { id: 11,num: 3, time: 1337 },
        { id: 12,num: 3, time: 1456 },
        { id: 13,num: 2, time: 1623 },
        { id: 14,num: 2, time: 1680 },
        { id: 15,num: 3, time: 1883 },
        { id: 16,num: 3, time: 1988 },
        { id: 17,num: 4, time: 2107 },
        { id: 18,num: 5, time: 2218 },
        { id: 19,num: 5, time: 2337 },
        { id: 20,num: 4, time: 2446 },
        { id: 21,num: 3, time: 2555 },
        { id: 22,num: 2, time: 2664 },
        { id: 23,num: 1, time: 2771 },
        { id: 24,num: 1, time: 2880 },
        { id: 25,num: 2, time: 2992 },
        { id: 26,num: 3, time: 3103 },
        { id: 27,num: 2, time: 3220 },
        { id: 28,num: 1, time: 3395 },
        { id: 29,num: 1, time: 3449 },
      ],
      now_note_id: 0,
      next_note_id: 0,
      playing_time: 0,
      record_time: 0,
      player: null,
      recorder: null,
      now_press_key: -1,
      display_keys: [
        { id: 1, num: 1, key: 90, type: "white" },
        { id: 2, num: 1.5, key: 83, type: "black" },
        { id: 3, num: 2, key: 88, type: "white" },
        { id: 4, num: 2.5, key: 68, type: "black" },
        { id: 5, num: 3, key: 67, type: "white" },
        { id: 6, num: 4, key: 86, type: "white" },
        { id: 7, num: 4.5, key: 71, type: "black" },
        { id: 8, num: 5, key: 66, type: "white" },
        { id: 9, num: 5.5, key: 72, type: "black" },
        { id: 10, num: 6, key: 78, type: "white" },
        { id: 11, num: 6.5, key: 74, type: "black" },
        { id: 12, num: 7, key: 77, type: "white" },
        { id: 13, num: 8, key: 81, type: "white" },
        { id: 14, num: 8.5, key: 50, type: "black" },
        { id: 15, num: 9, key: 87, type: "white" },
        { id: 16, num: 9.5, key: 51, type: "black" },
        { id: 17, num: 10, key: 69, type: "white" },
        { id: 18, num: 11, key: 82, type: "white" },
        { id: 19, num: 11.5, key: 53, type: "black" },
        { id: 20, num: 12, key: 84, type: "white" },
        { id: 21, num: 12.5, key: 54, type: "black" },
        { id: 22, num: 13, key: 89, type: "white" },
        { id: 23, num: 13.5, key: 55, type: "black" },
        { id: 24, num: 14, key: 85, type: "white" },
        { id: 25, num: 15, key: 73, type: "white" },
      ],
    };
  },
  created() {
    const soundPackIndex = [
      1, 1.5, 2, 2.5, 3, 3.5, 4, 4.5, 5, 5.5, 6, 6.5, 7, 8, 8.5, 9, 9.5, 10, 11,
      11.5, 12, 12.5, 13, 13.5, 14, 15,
    ];

    for (var i = 0; i < soundPackIndex.length; i++) {
      this.soundData.push({
        id: i,
        number: soundPackIndex[i],
        url:
          "https://awiclass.monoame.com/pianosound/set/" +
          soundPackIndex[i] +
          ".wav",
      });
    }
  },
  methods: {
    playnote(id, volume) {
      if (id > 0) {
        var audio_obj = document.querySelectorAll(`audio[data-num='${id}']`)[0];
        audio_obj.volume = volume;
        audio_obj.currentTime = 0;
        audio_obj.play();
      }
    },
    playnext(volume) {
      var play_note = this.notes[this.now_note_id].num;
      this.playnote(play_note, volume);
      this.now_note_id += 1;
      // this.now_note_id=now_note_id+1;

      if (this.now_note_id >= this.notes.length) {
        this.stopplay();
      }
    },
    start_record() {
      let vobj = this;
      this.record_time = 0;
      this.recorder = setInterval(function () {
        vobj.record_time++;
      });
    },
    startplay() {
      this.now_note_id = 0;
      this.playing_time = 0;
      this.next_note_id = 0;
      var vobj = this;
      this.player = setInterval(function () {
        if (vobj.playing_time >= vobj.notes[vobj.next_note_id].time) {
          vobj.playnext(1);
          vobj.next_note_id++;
        }
        vobj.playing_time++;
      }, 2);
    },
    stop_record() {
      clearInterval(this.recorder);
      this.record_time = 0;
    },

    stopplay() {
      clearInterval(this.player);
      this.now_note_id = 0;
      this.playing_time = 0;
      this.next_note_id = 0;
    },
    get_current_highlight(id, skey) {
      if (this.now_press_key == skey) return true;
      if (this.notes.length == 0) return false;
      var cur_id = this.now_note_id - 1;
      if (cur_id < 0) cur_id = 0;
      var num = this.notes[cur_id].num;
      if (num == id) return true;
      return false;
    },
    addnote(id) {
      if (this.record_time > 0)
        this.notes.push({ num: id, time: this.record_time });
      this.playnote(id, 1);
    },
    load_sample() {
      axios
        .get(
          "https://awiclass.monoame.com/api/command.php?type=get&name=music_dodoro"
        )
        .then((res) => {
          console.log(res);
          this.notes = res.data;
        });
    },
  },
};
</script>

<style lang="scss">
$key_width: 44px;
$key_height: 300px;
$color_white: #eee;
$color_black: #585858;

@mixin size($w, $h) {
  width: $w;
  height: $h;

  * {
    font-family: 微軟正黑體;
  }
}
html,
body {
  @include size(100%, 100%);
  margin: 0px;
  padding: 0px;
}

h2 {
  margin-bottom: 30px;
  color: $color_black;
}

button {
  background-color: transparent;
  border: none;
  border: solid 1px;
  padding: 4px 12px;
  border-radius: 5px;
  cursor: pointer;
  transition: 0.5s;

  &:hover {
    background-color: $color_black;
  }
}

.note_list {
  display: flex;
  li {
    list-style-type: none;
    border-right: solid 1px;
    padding: 2px 5px;
    cursor: pointer;
    transition: 0.3s;

    &:hover {
      background-color: darken($color_white, 5);
    }
    &.playing {
      background-color: darken($color_white, 10);
    }
    .time {
      font-size: 8px;
      opacity: 0.3;
    }
    .num {
      font-size: 16px;
    }
  }
}

.center_box {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.keyboard {
  margin-left: auto;
  margin-right: auto;
  max-width: 690px;
  display: flex;
  cursor: pointer;
  box-shadow: 0px 0px 40px rgba(0, 0, 0, 0.4);
  margin-bottom: 30px;
  .pianokey {
    position: relative;
    .white.playing {
      background-color: darken($color_white, 10);
      transform: translate(0px, 0px);
    }
    .black.playing {
      background-color: darken($color_black, 10);
      transform: translate(0px, 0px);
    }
    .white {
      @include size($key_width, $key_height);
      border: solid 1px $color_white;
      transform: translate(-3px, -3px);
      transition: 0.1s;
      &:active {
        transform: translate(0px, 0px);
        background-color: $color_white;
      }
    }
    .black {
      position: absolute;
      z-index: 3;
      top: 0px;
      width: $key_width/2;
      height: $key_height * 0.55;
      background-color: $color_black;
      margin-left: -$key_width/4;
      margin-right: -$key_width/4;
      transform: translate(-3px, -3px);
      transition: 0.1s;

      &:active {
        background-color: darken($color_black, 10);
      }
    }
    .label {
      position: absolute;
      color: rgba($color_black, 0.5);
      bottom: -25px;
      left: 50%;
      transform: translate(-50%);
      font-size: 8px;
    }
  }
}
.buttons {
  display: flex;
  justify-content: center;

  .abc {
    display: flex;
    align-items: center;
  }
}
.arrow-right {
  width: 0;
  height: 0;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 5px solid green;
  margin-left: 5px;
}
</style>
