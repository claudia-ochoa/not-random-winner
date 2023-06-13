<template>
  <div class="container">

    <h1>Configuración</h1>
    <div class="form-item">
      <label>Título</label>
      <input id="title" type="text" v-model="title" placeholder="Título o nombre de la competencia">
      <div class="error" v-if="errors.title.length">{{ errors.title }}</div>
    </div>

    <div class="form-item">
      <label>Descripción de la competencia</label>
      <textarea id="description" v-model="description" placeholder="Descripción"></textarea>
    </div>

    <div class="form-item">
        <label><input type="radio" v-model="typeSource" name="typeSource" value="list">Escribe la lista de participantes</label>
        <label><input type="radio" v-model="typeSource" name="typeSource" value="external">JSON URL Externa</label>
        <br><br>  
        
        <div v-if="typeSource === 'list'">
          <label>List of participantes</label>
          <textarea class="list" v-model="contestants" rows="20"></textarea>
          <br><small>Cada renglón es un participante</small>
        </div>
    </div>

    <div v-if="typeSource === 'external'" class="form-item">
      <label>JSON URL Externa</label>
      <input type="text" v-model="url" placeholder="Escribe la url del json">
      <small>Escribe la fuente externa de participantes en formato JSON)</small>
    </div>

    <div class="error" v-if="errors.source.length">{{ errors.source }}</div>
    <div class="success" v-if="validate">{{ submitMessage }}</div>
    <br>

    <button class="btn btn-stop" @click="save">Guardar</button>
  </div>
</template>

<script>

export default {
  name: "Setting",
  data: function() {
    return {
        url: '',
        title: '',
        description: '',
        contestants: '',
        source: null,
        validate: false,
        submitMessage: '',
        typeSource: 'list',
        errors: {
          title: '',
          source: '',
        }
    };
  },
  methods:{
    validation(){
      //reset form errors
      this.resetFormErrors()

      // if(!this.title) {
      //   this.errors.title = "Title is required!"
      // }
      
      // if(this.typeSource == 'list' && !this.contestants) {
      //   this.errors.source = "List of contestants is required!"
      // }

      // if(this.typeSource == 'external' && !this.url) {
      //   this.errors.source = "External source of contestants is required!"
      // }
      
      // if(!this.errors.title && !this.errors.source) {
      //   this.validate = true
      //   return true
      // }
      // return false
      return true
    },
    //save data to localStorage
    save(){
      if(this.validation()) {
        //if contestant list is fill
        if(this.contestants) {
          // split data 
          let source = this.contestants.split("\n")
          // get only is fill
          this.source = source.filter(item => item != " " && item != "")
          //this.source = [
            // "señora 1",
            // "señora 2",
            // "Ganadora Donny 1",
            // "señora 3",
            // "señora 4",
            // "Ganadora Donny 2",
            // "señora 5",
            // "señora 6",
            // "Ganadora Donny 4",
            // "señora 7",
            // "señora 8",
            // "señora 9",
            // "Ganadora Donny 3",
            // "señora 10",
            // "señora 11",
            // "Ganadora Donny 5",
            // "señora 12"
          //]
        }

        //after validate data save to localStorage
        localStorage.setItem('setting',JSON.stringify(
          {
            url: this.url,
            title: this.title,
            contestants: this.source,
            typeSource: this.typeSource,
            description: this.description,
          }
        ))
        // get submit message
        this.submitMessage = "Your settings was succesfully saved!"
      }
    },
    // reset form errors
    resetFormErrors(){
      this.errors = {
        title: '',
        source: '',
      }
    }
  },
  //after create
  created() {
    // check if localStorage setting exist
    if(localStorage.getItem('setting') !== null) {
      let setting = JSON.parse(localStorage.getItem('setting'))
      
      this.url = setting.url
      this.title = setting.title
      this.typeSource = setting.typeSource
      this.description = setting.description
      this.contestants = setting.contestants.join("\n")
    }
  },

};
</script>

<style scoped>

.error {
  color:orangered;
}

.success {
  color:seagreen;
}
</style>
