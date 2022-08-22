<template>
  <div class="hello">
    <button @click="conectar()">Conectar</button>
    <button @click="mensagem()">Enviar mensagem</button>
  </div>
</template>

<script>
import io from "socket.io-client";

export default {
  data (){
    return {
      socket: undefined
    }
  },
  methods:{
    conectar(){
      this.socket = io("http://localhost:3000")

      this.socket.on('mensagem', data =>{
        console.log("chegou mensagem do servidor: ", data)
      })
    },
    mensagem(){
      this.socket.emit("msg", this.socket.id)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
