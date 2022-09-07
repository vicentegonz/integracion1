<template>
  <div id="app">
    <h2>Tarea 1 Taller de integracion FLIGTHS.IO</h2> 
    <button v-on:click="conect()">conect</button>
    <button v-on:click="disconect()">disconect</button>
    <div id="container">
      <div id="top">
        <div id="chat">
          <h1>ChatRoom</h1>
          <div id="mesages">
            <div v-for="m in m_data" :key="m">
            <p v-if="m.message.level=='info'"><span class="user">{{m.message.name}}:</span> {{m.message.content}} <span class="date">{{m.message.date}}</span></p>
            <p class="warn" v-else><span class="user">{{m.message.name}}:</span> {{m.message.content}} <span class="date">{{m.message.date}}</span></p>
            </div>
          </div>
          <input class="mensaje" type="text" v-model="msg">
          <button @click="sendMessage">Send</button>
        </div>
        <div id="map">
          <h1 v-if="f_data">Estado de Vuelos</h1>
          <HelloWorld v-if="f_data" :fligts="f_data" :crashed="c_data" :landed="l_data" :planes="p_data" :takeoff="t_data"></HelloWorld>
        </div>
      </div>
      <div id="bot">
        <div v-if="f_data">
          <h1>Fligths</h1>
          <table>
            <tr>
              <th colspan="2">Origen</th>
              <th colspan="2">Destino</th>
              <th rowspan="2">ID de vuelo</th>
              <th rowspan="2">Horario</th>
            </tr>
            <tr>
              <th>Aeropuerto</th>
              <th>Ciudad</th>
              <th>Aeropuerto</th>
              <th>Ciudad</th>
            </tr>
            <tr v-for="vuelo in f_data.flights" :key="vuelo">
              <td>{{vuelo.departure.name}} ({{vuelo.departure.id}})</td>
              <td>{{vuelo.departure.city.name}} ({{vuelo.departure.city.country.id}})</td>
              <td>{{vuelo.destination.name}} ({{vuelo.destination.id}})</td>
              <td>{{vuelo.destination.city.name}} ({{vuelo.destination.city.country.id}})</td>
              <td>{{vuelo.id}}</td>
              <td>{{vuelo.departure_date}}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue';


export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data: function() {
    return {
      msg: null,
      connection: null,
      cEvent: JSON.stringify({
      "type": "join",
      "id": "1b62b209-b288-4459-94ba-3297c013d269",
      "username": "vichonario",
      }),
      f_data: null,
      p_data: {},
      t_data: [],
      l_data: [],
      c_data: [],
      m_data: [],
    }
  },
  methods:{
    sendMessage: function(event) {
      console.log("the mesage is", this.msg, event)
      if (this.msg != null) {
        var msgev = JSON.stringify({
          "type": "chat",
          "content": this.msg
          });
        this.connection.send(msgev)
        this.msg = null
      }
      //this.connection.send();
    },
    conect: function(){
        this.connection.send(this.cEvent);
    },
    disconect: function(){
      this.connection.close();
    }
  },
  created: function() {
    this.connection = new WebSocket("wss://tarea-1.2022-2.tallerdeintegracion.cl/connect")

    this.connection.onmessage = (event) => {
      var parsed = JSON.parse(event.data)
      //console.log(parsed.type)
      if (parsed.type == "flights"){
        //console.log(parsed.flights)
        this.f_data = (parsed)
      } else if (parsed.type == "plane") {
        if (Object.keys(this.p_data).includes(parsed.plane.flight_id)){
          this.p_data[parsed.plane.flight_id].plane = parsed.plane
          this.p_data[parsed.plane.flight_id].latlong.push([parsed.plane.position.lat, parsed.plane.position.long])
        } else {
          this.p_data[parsed.plane.flight_id] = { "plane": parsed.plane,
          "latlong": [[parsed.plane.position.lat, parsed.plane.position.long]]}
        }
      } else if (parsed.type == "message") {
        this.m_data.push(parsed)
      } else if (parsed.type == "take-off") {
        console.log("take off")
        this.t_data.push(parsed)
        if (Object.keys(this.p_data).includes(parsed.flight_id))
        this.p_data[parsed.flight_id].latlong = []
      } else if (parsed.type == "landing") {
        console.log("landing")
        this.l_data.push(parsed)
      } else if (parsed.type == "crashed") {
        console.log("crashed")
        this.c_data.push(parsed)
      } 
    }

    this.connection.onopen = function() {
      console.log("Successfully connected to the echo websocket server...")
    }

    this.connection.onclose = function() {
      console.log("Successfully disconnected to the echo websocket server...")
      location.reload()
    }

  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

p {
  padding-left: 6px;
  text-align: left;
}

.user {
  text-decoration: underline;
  font-weight: bold;
}
.date {
  font-weight:lighter;
  font-size:x-small;
  font-style: italic;
}

.warn {
  color: #ad1e23;
}

table {
  align-self: center;
  border-collapse: collapse;
  width: 100%;
  margin-bottom: 50px;
}

td, th {
  border: 1px solid #dddddd;
  text-align: center;
  padding:6px 20px;
}
tr th {
  border: 1px solid #dddddd;
  text-align: center;
  padding:6px 20px;
  background-color: #ad1e23;
  color:#fff;
}

tr:hover {
  background-color: #dddddd;
  cursor: pointer;
}

.content td, .content th {
    border-top: 1px solid transparent;
    padding: 2px 10px 2px 15px;
}

#top { 
  height: 60%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  margin-top: 50px;
  margin-bottom: 50px;
}
#bot { 
  height: 40%;
}
#chat { 
  height: 100%;
  margin-right: 20px;
  margin-left: 100px;
}
#map { 
  height: 450px;
  width: 1000px;
  margin-left: 10px;
  margin-right: 50px;
}
#mesages{
  height: 400px;
  width: 500px;
  overflow-y: auto;
  border: 1px solid black;
  margin-bottom: 10px;
  align-items: flex-start;
}
.mensaje {
  width: 425px;
  overflow-x: scroll;
}
</style>
