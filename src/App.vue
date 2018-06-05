<template>
  <div id="app">
    <img width="25%" src="./assets/logo.png">
    <div class="page-header">
    <h1>Fame record tool</h1>
    </div>
  <div class="col-sm-12 col-md-12">
   <div class="container">
   <div class="row">
   <div class="col-xs-12 col-sm-12 col-md-6">
   <h3>Timer</h3>
     <input class="input-timer col-xs-5 col-sm-4 col-md-5 input-lg text-center text-md-right text-sm-right"
      :value="minutes"
     @input="updateValue('minutes', $event.target.value)">
     <span class="input-separator">:</span>
     <input class="input-timer col-xs-5 col-sm-4 col-md-5 input-lg text-center text-md-left text-sm-left"
      :value="seconds"
     @input="updateValue('seconds', $event.target.value)"><br>
     <button type="button" class="btn btn-warning pb-2" v-if="started" v-on:click="resetTimer" @click="startTimer">Reset</button>
   </div>
   <div class="col-sm-12 col-md-6">
    <h3>Add Activity</h3>
    <div class="row">
      <div class="col-sm-6 col-md-6">
      <div class="form-group"><label>Autor:</label> <input type="text" class="form-control" v-model="newAct.autor" /></div>
      <div class="form-group"><label>Start Fame:</label> <input type="text" class="form-control" v-model="newAct.startFame" /></div>
      <div class="form-group"><label>End Fame:</label> <input type="text" class="form-control" v-model="newAct.endFame" /></div>
      <div class="form-group"><label>People:</label> <input type="text" class="form-control" v-model="newAct.people" /></div>
      </div>
      <div class="col-sm-6 col-md-6">
      <div class="form-group"><label>Location:</label> <select class="form-control" v-model="newAct.dungeon">
        <option disabled value="">Select Location</option><option>T5 Black Zone Dungeon</option><option>T6 Black Zone Dungeon</option><option>T7 Black Zone Dungeon</option><option>Carpet of Bones</option><option>CV</option></select></div>
      <label>Start Time:</label><input type="text" class="form-control" v-model="newAct.startTime" disabled/>
     <label>End Time:</label><input type="text" class="form-control" v-model="newAct.endTime" disabled/>
     <div class="pt-2">
      <button type="button" class="btn btn-success" @click="submitStart" v-on:click="startTimer">Start</button> &nbsp;
      <button type="button" class="btn btn-danger" @click="submitEnd" v-on:click="resetTimer">Stop</button> &nbsp;
      <button type="button" class="btn btn-primary" @click="addAct">Add</button>
      </div>
        </div>
        </div>
        </div>
      </div>
      <div class="col-sm-12 col-md-12">
      <div class="panel-heading">
        <h3>Recorded actvitities</h3>
      </div>
      <div class="panel-body">
        <table class="table table-sm table-hover table-striped borders">
          <thead>
            <tr>
              <th> Autor </th>
              <th> Start Fame </th>
              <th> End Fame </th>
              <th> Location </th>
              <th> Poeple </th>
              <th> Start Time </th>
              <th> End Time </th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="Act in Activities">
              <td> {{Act.autor}} </td>
              <td> {{Act.startFame}} </td>
              <td> {{Act.endFame}} </td>
              <td> {{Act.dungeon}} </td>
              <td> {{Act.people}} </td>
              <td> {{Act.startTime}} </td>
              <td> {{Act.endTime}} </td>
              <td> <span class="glyphicon glyphicon-trash btn-delete-style" v-on:click="removeAct(Act)" title="Delete Product">Delete</span> </td>
            </tr>
          </tbody>
        </table>
        </div>
    </div>
  </div>
  </div>
</div>
</template>

<script>
import Firebase from "firebase";

let config = {
  apiKey: "AIzaSyDU03oxq6mfULzQwHPYSftFNXOfP4YvgnA",
  authDomain: "mrninfirebaseproject.firebaseapp.com",
  databaseURL: "https://mrninfirebaseproject.firebaseio.com",
  projectId: "mrninfirebaseproject",
  storageBucket: "mrninfirebaseproject.appspot.com",
  messagingSenderId: "20266336203"
};

let app = Firebase.initializeApp(config);
let db = app.database();

//lisener
let ActRef = db.ref("Activities");

export default {
  name: "app",
  firebase: {
    Activities: ActRef
  },
  data() {
    return {
      duration: 0,
      minutes: 30,
      seconds: 0,
      started: false,
      interval: undefined,
      resetMinutes: 0,
      resetSeconds: 0,
      newAct: {
        autor: "mrnin",
        startFame: "",
        endFame: "",
        people: "",
        startTime: "",
        endTime: ""
      }
    };
  },
  methods: {
    addAct() {
      ActRef.push(this.newAct);
      this.newAct.author = "";
      this.newAct.dungeon = "";
      this.newAct.startFame = "";
      this.newAct.endFame = "";
      this.newAct.people = "";
      this.newAct.startTime = "";
      this.newAct.endTime = "";
    },
    removeAct(Act) {
      ActRef.child(Act[".key"]).remove();
    },
    submitStart() {
      let t = new Date();
      let time = t.getHours() + ":" + t.getMinutes();
      this.newAct.startTime = time;
    },
    submitEnd() {
      let t = new Date();
      let time = t.getHours() + ":" + t.getMinutes();
      this.newAct.endTime = time;
    },
    startTimer: function() {
      this.started = true;
      this.resetMinutes = this.minutes;
      this.resetSeconds = this.seconds;
      this.calculateDuration();
      this.startInterval();
    },
    resetTimer: function() {
      this.started = false;
      this.stopInterval();
      this.minutes = this.resetMinutes;
      this.seconds = this.resetSeconds;
      this.duration = 0;
    },
    calculateDuration: function() {
      this.duration = this.minutes * 60 + this.seconds;
    },
    setTime: function(duration) {
      this.minutes = Math.floor(duration / 60);
      this.seconds = duration % 60;
    },
    startInterval: function() {
      this.interval = setInterval(() => {
        if (this.duration <= 0) {
          this.resetTimer();
        } else {
          this.duration -= 1;

          this.setTime(this.duration);
        }
      }, 1000);
    },
    stopInterval: function() {
      window.clearInterval(this.interval);
    },

    updateValue: function(field, value) {
      var formattedValue = Number(value);

      if (isNaN(formattedValue) || !Number.isInteger(formattedValue)) {
        formattedValue = this[field];
      } else if (formattedValue > 59 && field === "seconds") {
        formattedValue = this[field];
      } else if (formattedValue > 99 && field === "minutes") {
        formattedValue = this[field];
      }

      this[field] = formattedValue;
      this.$forceUpdate();
    }
  }
};
console.log("Ahoj");
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.form-container {
  text-align: center;
}
.input-timer {
  font-family: "Roboto";
  border: none;
  font-size: 8.5vw;
  cursor: default;
  color: black;
  background: transparent;
  align-items: center;
}
.input-separator {
  font-family: "Roboto";
  font-size: 7vw;
  color: black;
}
</style>
