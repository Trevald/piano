<template>
  <div id="app">
    <div class="top">
        <button type="button" @click="autoplay()">{{ isDemoPlaying ? "Stop" : "Demo" }}</button>
        <button type="button" @click="startLesson()">{{ lessonIsOngoing ? "Stop lesson" : "Start lesson" }}</button>
    </div>
    <div class="bottom">
        <div class="keyboard">
            <button type="button" v-for="key in keysWithData" :key="key.id" @touchstart="playSound(key.id)" @touchend="stopSound(key.id)" :class="keyButtonCSS(key)"><span class="effect"></span></button>    
        </div>
    </div>
  </div>
</template>

<script>
import {Howl} from 'howler';

export default {
  name: 'App',
  components: {
    
  },

  data() {
      return {
          keys: ["c5", "c-5", "d5", "d-5", "e5", "f5", "f-5", "g5", "g-5", "a5", "a-5", "b5", "c6"],
          song: {
              bpm: 80,
              melody: "c5x1,c5x1,g5x1,g5x1,a5x1,a5x1,g5x2,f5x1,f5x1,e5x1,e5x1,d5x1,d5x1,c5x2,g5x1,g5x1,f5x1,f5x1,e5x1,e5x1,d5x2,g5x1,g5x1,f5x1,f5x1,e5x1,e5x1,d5x2,c5x1,c5x1,g5x1,g5x1,a5x1,a5x1,g5x2,f5x1,f5x1,e5x1,e5x1,d5x1,d5x1,c5x4"
          },
          sounds: [],
          autoplayTimeout: undefined,
          lessonTimeout: undefined,
          nextKeyIndex: undefined,
          nextKey: undefined,
          currentKey: undefined,
          breathingTime: 100,
          lessonIsOngoing: false
      }
  },

  computed: {
      keysWithData() {
          return this.keys.map(key => {
              return {
                  id: key,
                  isSharp: key.indexOf("-") !== -1
              }
          });
      },

      isDemoPlaying() {
          return typeof this.autoplayTimeout !== "undefined";
      }
  },

  methods: {

    // 1. Start tutorial
    startLesson() {
        this.lessonIsOngoing = true;
        this.setNextKeyIndex();
    },

    hideCurrentKey() {
        this.nextKey = undefined;
        this.lessonTimeout = setTimeout(() => {
            this.setNextKeyIndex();
        }, 200);
    },

    checkKey(id) {
        if (this.lessonIsOngoing && id === this.nextKey) {
            this.hideCurrentKey();
        }
        
    },

    // 2. Light up first key in melody
    setNextKeyIndex() {
        if (!this.lessonIsOngoing) { return; }
        if (this.nextKeyIndex === undefined) {
            this.nextKeyIndex = 0;
        } else {
            this.nextKeyIndex++;
        }

        if (this.nextKeyIndex > this.song.melody.length) {
            this.lessonIsOngoing = false;
            this.nextKeyIndex = undefined;
            this.nextKey = undefined;
        } else {
            this.nextKey = this.song.melody[this.nextKeyIndex].split("x")[0];
        }
         
    },


      keyButtonCSS(key) {
          return { 
              "key": true,
              "is-sharp": key.isSharp,
              "is-playing": this.currentKey === key.id,
              "is-next": this.nextKey === key.id
            }
      },
      
      playSound(id) {
          if ( this.sounds.length === 0) {
              this.loadSounds();
          }
        const sound = this.sounds.find(sound => sound.id === id);
        sound.sound.stop();
        sound.sound.fade(0, 1, 50);
        sound.sound.play();
        this.checkKey(id);
      },

      stopSound(id) {
        const sound = this.sounds.find(sound => sound.id === id);
        sound.sound.fade(1, 0, 250);
      },

      reset() {
            clearTimeout(this.autoplayTimeout);
            this.autoplayTimeout = undefined;              
            this.nextKey = undefined;
            this.currentKey = undefined;
            this.stopAllSounds();  

            return false;
      },
      
      autoplay() {
          if (this.isDemoPlaying) {
            this.reset();
        } else {
            this.playMelodyNote(0);
        }
      },

    playMelodyNote(index) {
        if (index >= this.song.melody.length) { return this.reset(); }

        const bpm = this.song.bpm;
        const note = this.song.melody[index].split('x');
        const key = note[0];
        const length = note[1];
        
        this.playSound(key);
        this.currentKey = key;
        this.nextKey = index !== this.song.melody.length ? undefined : this.song.melody[index+1].split("x")[0];
        this.autoplayTimeout = setTimeout(() => {
            this.waitBetweenNotes(index + 1, key)
        }, (length * 1000 * (60 / bpm)) - this.breathingTime); // 1/4 = 1 second then split by bpm and subtract the breathing time
    },

    waitBetweenNotes(index, key) {
        this.stopSound(key);
        this.autoplayTimeout = setTimeout(()=> {
            this.playMelodyNote(index)
        }, this.breathingTime);
    },

    stopAllSounds() {
        this.sounds.forEach(sound => {
            sound.sound.stop();
        });
    },

    loadSounds() {
      const soundsSrc = "./vendor/sounds/flute/";
      this.keys.forEach(key => {
          const keySrc = `${soundsSrc}${key.toUpperCase()}.mp3`;
          this.sounds.push({
              id: key,
              sound: new Howl({src: [keySrc]})
          });
      });

      
    }

  },

  mounted() {
      this.loadSounds();
        this.song.melody = this.song.melody.split(",");
  }
}
</script>

<style>

html, body, #app {
    display: flex;
    flex: 1 1 auto;
    width: 100%;
    margin: 0;
    padding: 0;
}

html {
    min-height: 100%;
    box-sizing: border-box;
}

body { 
    background: linear-gradient(180deg, #DABCBC 0%, #BFB0E0 100%);
}

#app {
    flex-direction: column;
}

.top {
    justify-content: center;
    align-items: center;
}

.top button {
    font-size: 2vw;
    margin: 1rem;
    padding: 1rem;
    border-radius: 100rem;
    background: white;
    border: 3px solid blue;
}

.top, .bottom {
    flex: 1 1 50%;
    display: flex;
}
    

*, *:before, *:after {
    box-sizing: inherit;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.keyboard {
    display: flex;
    flex: 1 1 auto;
    justify-content: stretch;
    align-items: stretch;
    width: 100%;
    padding: 20px;
    position: relative;

}

.key {
    display: flex;
    justify-content: stretch;
    align-items: stretch;
    flex: 1 1 auto;
    flex-direction: column;
    appearance: none;
    width: auto;
    height: auto;
    margin: 0 2px;
    box-shadow: none;
    border: none;
    background:black;
    border-radius: 0;
    border: none;
    padding: 0;
    position: relative;
    border-bottom-right-radius: 5px;
    border-bottom-left-radius: 5px;
    
    box-shadow: inset 0px -10px 0px rgba(0, 0, 0, 0.25);

    transition: all .3s ease;
}

.key:nth-child(1) { background-color: #78FFA1; }
.key:nth-child(3) { background-color: #BCFF90; }
.key:nth-child(5) { background-color: #FEFA8F; }
.key:nth-child(6) { background-color: #FFDA94; }
.key:nth-child(8) { background-color: #FF9C9C; }
.key:nth-child(10) { background-color: #FF97D7; }
.key:nth-child(12) { background-color: #FD96FF; }
.key:nth-child(13) { background-color: #BC9AFF; }


.key .effect {
    flex: 1 1 auto;
    display: block;
    background: linear-gradient(180deg, rgba(255, 255, 255, 0.75) 0%, rgba(255, 255, 255, 0) 75%);
}

.key.is-playing .effect,
.key:active .effect {
    background: linear-gradient(180deg, rgba(255, 255, 255, 100) 0%, rgba(255, 255, 255, 0.25) 75%);
}


.key:first-child,
.key:first-child .effect {
    border-top-left-radius: 50px;
    border-bottom-left-radius: 50px;
}

.key:last-child,
.key:last-child .effect {
    border-top-right-radius: 50px;
    border-bottom-right-radius: 50px;
}
.key.is-sharp {
    background: #503838;
    box-shadow: inset 0px -10px 0px rgba(0, 0, 0, 0.19);
    border-bottom-left-radius: 100px;
    border-bottom-right-radius: 100px;
    height: 60%;
    width: 10%;
    transform: translateX(-50%);
    margin: 0;
    position: absolute;
    top: 20px;
    z-index: 2;
}

.key.is-sharp:nth-child(2) { left: 13.5%; }
.key.is-sharp:nth-child(4) { left: 25.5%; }
.key.is-sharp:nth-child(7) { left: 49.5%; }
.key.is-sharp:nth-child(9) { left: 62%; }
.key.is-sharp:nth-child(11) { left: 74%; }

.key.is-sharp .effect {
    border-bottom-left-radius: 100px;
    border-bottom-right-radius: 100px;    
    background: linear-gradient(180deg, #685151 0%, rgba(56, 50, 50, 0) 100%);
}


.key.is-sharp.is-playing,
.key.is-sharp:active {
    background-color: rgba(30, 40, 55);
}

.key::before {
    content: "";
    position: absolute;
    left: 50%;
    top: -30px;
    width: 16px;
    height: 16px;
    background-color: #f5f5f5;
    border: 2px solid black;
    border-radius: 16px;
    transform: translateX(-50%);
    transition: all .3s ease;
}

.key.is-sharp::before {
    top: -60px;
}

.key.is-playing::before {
    background-color: rgba(0, 0, 255, 0.75);
}

.key.is-next::before {
    background-color: rgba(0, 0, 255, 0.25);
}

</style>
