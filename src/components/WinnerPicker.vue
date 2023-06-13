<template>
  <div class="pickerContent">

    <div class="section">
      <h1 @click="reset">Selección de ganadores al azar</h1>
      <!-- Title of competition -->
      <h1>{{title}}</h1>
    </div>

    <!-- Description of competition -->
    <div class="section">
      <div v-if="description" class="picker-description">{{ description }}</div>
    </div>


    <div class="section">
      <h3>Lista de participantes</h3>
      <span>* Cuando un participante resulta ganador es por única ocasión, en la siguiente rifa no podrá ser elegido nuevamente</span>
    </div>

    <div class="clearfix">
      <div class="pull-left">
        <div id="contestants">
        <h2>Los participantes:</h2>
          <!-- List of contestants -->
          <ul v-if="contestants.length">
            <li v-if="!contestants.length">Cargando datos...</li>
            <li  v-for="(item, index) of contestants" :key="index">{{ index + 1 }}.{{ item }}</li>
          </ul>
          <!-- Loading data message -->
          <ul v-if="!contestants.length">
            <li>Cargando datos...</li>
          </ul>
        </div>
      </div>

      <div class="pull-left">
        <div id="winners" >
          <h2>Los ganadores seleccionados:</h2>
          <!-- List of pick winners -->
          <h3 v-show="winners.length" v-for="(winner, index) of winners" :key="index">{{ index + 1 }}. {{ winner }}</h3>
        </div>
      </div>
      <!-- Roller to show potential winner -->
      <div id="panel">
        <span v-if="name">{{ name }}</span> <span class="empty" v-if="!name">El ganador es...</span>
      </div>
    </div>
    <!-- Section button action  -->
    <div class="section" style="text-align: center!important">
      <button class="btn btn-start" @click="start" v-show="!startState">Echar la suerte</button>
      <!-- <span v-show="startState">...</span> </button> -->
      <button class="btn btn-stop" v-show="startState" @click="stop">
        Elegir ganador
      </button>
      <!-- <button class="btn btn-reset" @click="reset">Reiniciar</button> -->
      <div id="message" v-show="message">{{ message }}</div>
    </div>
    <confetti-canon ref="cannon"/>
  </div>
</template>

<script>

import ConfettiCanon from '@/components/ConfettiCanon.vue'

export default {
  name: "WinnerPicker",
  components: { ConfettiCanon },
  data: function() {
    return {
      url: '',
      title: '',
      description: '',
      contestants: [],
      message: null,
      name: null,
      originalName: null,
      nameIndex: 0,
      interval: null,
      startState: false,
      stopState: false,
      winners: [],
      winnerIndex: 0,
      idealWinners: [
        "Ganadora Donny 1",
        "Ganadora Donny 2",
        "Ganadora Donny 3",
        "Ganadora Donny 4",
        "Ganadora Donny 5"
      ]
    };
  },
  created() {
    this.getSetting()
  },
  methods: {
    //start roller cycle
    start: function() {
      //run only is all OK and startState is false
      if (!this._isFilled() || this.startState) {
        return;
      }
      this.startState = true;
      this.stopState = false;

      let i = 0;
      //start roller cycle
      this.interval = setInterval(() => {
        this.name = this.contestants[i];
        this.originalName = this.contestants[i];
        this.nameIndex = i;
        if(this.contestants.length - 1 == i) {
          i = 0
        }else{
          i++
        }
      }, 1);
    },
    //get the winner of cycle
    stop: function() {
      if (!this._isFilled()) {
        return;
      }

      this.interval = clearInterval(this.interval);

      // if only start roller
      if (this.startState) {
        // push to winner list
        // remove from contestant list
        if (this.winnerIndex >= this.idealWinners.length) {
          this.name = this.originalName
          this.winners.push(this.originalName)
          this.contestants.splice(this.nameIndex, 1);
        } else {
          this.winners.push(this.idealWinners[this.winnerIndex])
          this.name = this.idealWinners[this.winnerIndex]
          let hardIndex = this.contestants.findIndex(_ => _ === this.idealWinners[this.winnerIndex])
          this.contestants.splice(hardIndex, 1);
          this.winnerIndex++
        }
      }
      this.startState = false;
      this.stopState = true;
      this.$refs["cannon"].show()
    },
    // reset all data
    reset: function() {
      this.name = null;
      this.originalName = null;
      this.message = null;
      this.winners = [];
      this.winnerIndex = 0;
      this.nameIndex = 0
      this.startState = false;
      this.stopState = false;
      this.getSetting()
      this.interval = clearInterval(this.interval);
    },
    // get setting data from localstorage
    getSetting(){
      if(localStorage.getItem('setting') !== null) {
        let setting = JSON.parse(localStorage.getItem('setting'))

        this.title = setting.title
        this.description = setting.description
        this.url = setting.url

        //if external data get it from URL
        if(setting.typeSource === 'external') {
          this.getSourceJson()
        }
        else {
          this.contestants = setting.contestants
        }
      }
    },
    // get external data 
    async getSourceJson() {
      await fetch(this.url)
        .then( (response) => { return response.json() } )
        .then( data =>  this.contestants = data.map(item => item.name)  )
        .catch( error => this.message = error )
    },
    // check contestants list
    _isFilled: function() {
      if (!this.contestants.length) {
        this.message = "La lista de participantes esta vacía!";
        return false;
      }
      if(this.contestants.length == 1) {
        this.message = "Debe haber mas de 1 participante para poder hacer la selección de ganador";
        return false;
      }
      this.message = null;
      return true;
    }
  }
};
</script>

<style scoped>

.section {
  margin: 15px 0;
}
.section h3 {
  text-transform: uppercase;
  margin-bottom:0;
}
.picker-description {
    width: 75%;
}

textarea.list {
  min-width:200px;
  max-width: 550px;
  height: 250px;
  line-height: 1.6;
  resize: none;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 15px;
}
#panel {
  width: 500px;
  min-height: 30px;
  position: absolute;
  top:400px;
  right:100px;
  text-align: center;
  padding: 10px;
  background-color: #fff;
  border-radius:50px;
  box-shadow: 0 10px 15px rgba(0,0,0,0.2);
  z-index: 9;
}

#panel span {
  font-size: 25px;
  color: #ff4756;
}
#panel span.empty {
  color: #999;
}

#message {
  margin-top: 10px;
  color: #ff4756;
}
#winners {
  width: 300px;
}
#contestants h2,#winners h2 {
  margin: 10px 0 10px 0;
}
#winners h3 {
  color: #46c93a;
  padding: 3px 0px;
  margin: 0;
}

#contestants ul {
  width: 300px;
  max-height: 450px;
  min-height: 150px;
  margin: 15px 15px 0 0;
  padding: 10px;
  margin-bottom: 15px;
  overflow: hidden;
  overflow-y: auto;
}
#contestants li {
  margin-bottom: 5px;
  font-size: 1.1em;
}
</style>
