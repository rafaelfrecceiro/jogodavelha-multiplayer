<template>
  <div class="container">
    <div class="topbar">
      <div class="topo">
        <p class="title"># Jogo da Velha #</p>
      </div>
    </div>
    <div class="content">
      <div class="chat">
        <div class="chat-content">

          <div class="balloon-conversation" v-for="msg in messages" :key="msg.id">
            <div class="player">{{msg.player}}</div>
            <div class="balloon" :class="{'my-message': msg.owner}">{{ msg.content }}</div>
          </div>

        </div>
        <div class="chat-message-box">
          <input type="text" v-model="inputMessage" @keyup.enter="sendMessage()"/>
          <button @click="sendMessage()">Enviar</button>
        </div>
      </div>
      <div class="board">

        <div class="board-content">
          <div class="tile" id="op1" @click="clickOption('op1')"></div>
          <div class="tile" id="op2" @click="clickOption('op2')"></div>
          <div class="tile" id="op3" @click="clickOption('op3')"></div>

          <div class="tile" id="op4" @click="clickOption('op4')"></div>
          <div class="tile" id="op5" @click="clickOption('op5')"></div>
          <div class="tile" id="op6" @click="clickOption('op6')"></div>

          <div class="tile" id="op7" @click="clickOption('op7')"></div>
          <div class="tile" id="op8" @click="clickOption('op8')"></div>
          <div class="tile" id="op9" @click="clickOption('op9')"></div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import io from "socket.io-client";

export default {
  data(){
    return {
      socket: undefined,
      messages:[],
      inputMessage: '',
      namePlayer: 'Rafael',
      turn: 0,
      gameOptions: ['X', 'O'],
      youTurn: false,
    }
  },
  created() {
    let name = window.prompt("Digite seu nome:")
    this.namePlayer = name
  },
  mounted() {
    if(!this.isMobile()){
      this.socket = io("http://localhost:3000")
    }else{
      this.socket = io("http://192.168.15.22:3000")
    }

    this.socket.on('messageReceive', data =>{
      console.log('messageReceive', data)
      this.messages.push({
        id: this.messages.length+1,
        content: data.inputMessage,
        player: data.namePlayer,
        owner: false
      })
    })

    this.socket.on('optionsReceive', data =>{
      console.log('optionsReceive', data)
      let elOpt = document.getElementById(data.position)
      elOpt.innerText = this.gameOptions[this.turn]
      this.youTurn = true
      this.changeTurn()
    })

    this.socket.on('startingGame', data =>{
      console.log('startingGame', data)
      this.youTurn = true
    })

    this.socket.on('losingGame', data =>{
      console.log('losingGame', data)
      alert("VOCÊ PERDEU!")
    })
  },
  methods:{
    isMobile() {
      if(/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)) {
        return true
      } else {
        return false
      }
    },
    sendMessage(){
      this.messages.push({
        id: this.messages.length+1,
        content: this.inputMessage,
        player: this.namePlayer,
        owner: true
      })

      this.socket.emit("sendMessage", {
        socketId: this.socket.id,
        namePlayer: this.namePlayer,
        inputMessage: this.inputMessage
      })

      this.inputMessage = ""
    },
    clickOption(option){
      if(this.youTurn) {
        let elOpt = document.getElementById(option)
        elOpt.innerText = this.gameOptions[this.turn]
        this.socket.emit("optionsSend", {
          socketId: this.socket.id,
          position: option
        })

        if(this.checkVictory()){
          this.socket.emit("losingGame", {})
          alert("VOCÊ GANHOU")
        }
        this.changeTurn()
        this.youTurn = false
      }else{
        alert("Não é seu turno")
      }
    },
    changeTurn(){
      if(this.turn == 0) this.turn = 1
      else if(this.turn == 1) this.turn = 0
    },
    checkVictory(){
      let op1 = document.getElementById('op1').innerText
      let op2 = document.getElementById('op2').innerText
      let op3 = document.getElementById('op3').innerText
      let op4 = document.getElementById('op4').innerText
      let op5 = document.getElementById('op5').innerText
      let op6 = document.getElementById('op6').innerText
      let op7 = document.getElementById('op7').innerText
      let op8 = document.getElementById('op8').innerText
      let op9 = document.getElementById('op9').innerText

      //Horizontais
      if((op1 == this.gameOptions[this.turn] && op2 == this.gameOptions[this.turn] && op3 == this.gameOptions[this.turn]) ||
          (op4 == this.gameOptions[this.turn] && op5 == this.gameOptions[this.turn] && op6 == this.gameOptions[this.turn]) ||
          (op7 == this.gameOptions[this.turn] && op8 == this.gameOptions[this.turn] && op9 == this.gameOptions[this.turn])){
        return true
      }

      //Verticais
      if((op1 == this.gameOptions[this.turn] && op4 == this.gameOptions[this.turn] && op7 == this.gameOptions[this.turn]) ||
          (op2 == this.gameOptions[this.turn] && op5 == this.gameOptions[this.turn] && op8 == this.gameOptions[this.turn]) ||
          (op3 == this.gameOptions[this.turn] && op6 == this.gameOptions[this.turn] && op9 == this.gameOptions[this.turn])){
        return true
      }

      //Diagonais
      if((op1 == this.gameOptions[this.turn] && op5 == this.gameOptions[this.turn] && op9 == this.gameOptions[this.turn]) ||
          (op3 == this.gameOptions[this.turn] && op5 == this.gameOptions[this.turn] && op7 == this.gameOptions[this.turn])){
        return true
      }
      return false
    }
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DynaPuff:wght@600&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Roboto&display=swap');
@import url('https://fonts.googleapis.com/css2?family=DynaPuff&display=swap');

.container{
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 100px 1fr;
  height: 100vh;
}

.content{
  display: grid;
  grid-template-columns: 500px 1fr;
  grid-template-rows: 1fr;
}

.topbar{
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
}

.topo{
  display: flex;
  justify-content: center;
  align-content: center;
  border-bottom: 2px solid #2f2f2f;
}

.title{
  font-family: 'DynaPuff', cursive;
  font-size: 4.5rem;
  color: #c1c1c1;
  -webkit-text-stroke: 2px black;
  text-shadow: 4px 4px dimgray;
}

.chat{
  display: flex;
  justify-content: flex-start;
  align-content: flex-start;
  border-right: 2px solid #2f2f2f;
  flex-wrap: wrap;
}

.chat-content{
  width: 100%;
  height: 85%;
  background-color: antiquewhite;
  padding: 1rem;
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  overflow: auto;
}

.balloon-conversation{
  display: flex;
}

.balloon{
  width: 250px;
  height: 25px;
  border: 1px solid #b7b7b7;
  background-color: #d5d5d5;
  border-radius: 10px 10px 10px 0px;
  margin-bottom: 10px;
  font-family: 'Roboto', sans-serif;
  font-size: 0.8rem;
  padding: 0.5rem;
}

.player{
  padding-top: 1.2rem;
  font-family: 'Roboto', sans-serif;
  margin-right: 10px;
}

.my-message{
  background-color: #42a665;
}

.chat-message-box{
  display: flex;
  width: 100%;
  justify-content: space-between;
  align-content: space-between;
}

.chat-message-box input{
  font-size: 0.8rem;
  padding: 5px;
  border-radius: 5px;
  border: 1px solid black;
  width: 415px;
}

.chat-message-box button{
  font-size: 0.8rem;
  padding: 5px;
  font-weight: bolder;
  width: 70px;
  border-radius: 5px;
  border: 1px solid black;
}

.board{
  display: flex;
  justify-content: center;
  align-content: center;
  width: 100%;
  flex-wrap: wrap;
}

.board-content{
display: flex;
  flex-wrap: wrap;
  width: 400px;
  height: 400px;
  align-content: flex-start;
}

.tile{
  width: 100px;
  height: 100px;
  display: flex;
  justify-content: center;
  align-content: center;
  font-size: 4rem;
  font-family: 'DynaPuff', cursive;
}

.tile:hover{
  background-color: #848484;
  cursor: pointer;
  border-radius: 10px;
}

#op1{
  border-right: 5px solid black;
  border-bottom: 5px solid black;
}
#op2{
  border-right: 5px solid black;
  border-bottom: 5px solid black;
}
#op3{
  border-bottom: 5px solid black;
}
#op4{
  border-right: 5px solid black;
  border-bottom: 5px solid black;
}
#op5{
  border-right: 5px solid black;
  border-bottom: 5px solid black;
}
#op6{
  border-bottom: 5px solid black;
}
#op7{
  border-right: 5px solid black;
}
#op8{

}
#op9{
  border-left: 5px solid black;
}

@media (max-width: 600px) {
  .container{
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 100px 1fr;
    height: 100vh;
  }

  .content{
    display: grid;
    grid-template-columns: 500px 1fr;
    grid-template-rows: 1fr;
  }

  .topbar{
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 1fr;
  }

  .chat{
    display: none;
    justify-content: flex-start;
    align-content: flex-start;
    border-right: 2px solid #2f2f2f;
    flex-wrap: wrap;
  }

  .chat-content{
    width: 100%;
    height: 45%;
    background-color: antiquewhite;
    padding: 1rem;
    display: flex;
    flex-wrap: wrap;
    align-content: flex-start;
    overflow: auto;
  }

  .topo{
    display: flex;
    justify-content: center;
    align-content: center;
    border-bottom: 2px solid #2f2f2f;
  }

  .title{
    font-family: 'DynaPuff', cursive;
    font-size: 2.5rem;
    color: #c1c1c1;
    -webkit-text-stroke: 2px black;
    text-shadow: 4px 4px dimgray;
    padding-top: 15px;
  }

  .chat-message-box button{
    display: none;
  }

  .chat-message-box input{
    width: 100%;
    margin-left: 10px;
    margin-right: 20px;
  }
}
</style>
