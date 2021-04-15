<template>
  <div class="chat-bot">
    <body>
      <div class="container">
        <div class="chat-box">
          <div class="client">
              <div class="client-info">
                <h2>Hermes</h2>
                <p>En línea</p>
              </div>
          </div>
          <ul class="chats" ref="chatbox">
            <li class="my-chat"
              v-for="(message, idx) in messages"
              :key="idx"
              :class="message.author"
            >                    
              <p> {{ message.text }} </p>
              <div class= "date"> {{message.hour}}</div>
            </li>                        
          </ul>
          <div class="chat-input">
            <input type="text"
              v-model="message"
              @keyup.enter="sendMessage"
              placeholder="Escriba un mensaje" 
            />
            <button class="send"
              v-on:click="sendMessage" >
              Send
            </button>
          </div>            
        </div>      
      </div>
    </body>
  </div>
</template>

<script>
  const url = 'http://beta.soldai.com//bill-cipher/askquestion?'
  const sessionId = '1903135173030124'
  const keyBot = '740d32f7eb34cd5c3ff90329d0ce4fbd07361e89'
 
 import axios from "axios";

  export default {
    name: 'ChatBox',
    data: () => ({
      message: '',
      messages: [],    
    }),
    updated() {
      this.$refs.chatbox.scrollTop = this.$refs.chatbox.scrollHeight
    },  
    methods: {
      buildAnswer(dataApi, word, key)  {
        let answer =''
        let pokemon = ''
        let type = ''
        dataApi.map(function(elemento, index) {

          if (index % 2 == 0 || index == 0) {
            pokemon = `${elemento} ${word}`
            }

          if(index % 2 == 1 && Array.isArray(elemento)) {       
            elemento.map(function(Element){
              type += `${key.split('.').reduce((o , i) => o[i], Element)}, `
            })
            answer += pokemon.concat(' ', type)
          }
          else if (index % 2 == 1) {
            answer += pokemon.concat(' ', elemento)
          }
        });     
        return answer   
        },
      showTime(){
        let myDate = new Date();
        let hours = myDate.getHours();
        let minutes = myDate.getMinutes();
        let seconds = myDate.getSeconds();
        if (hours < 10) hours = 0 + hours;
        if (minutes < 10) minutes = "0" + minutes;
        if (seconds < 10) seconds = "0" + seconds;
        return (`${hours}:${minutes}`)
      },
      async sendMessage() {
        const message = this.message
        this.messages.push({
          text: message,
          author: 'my-chat',
          hour: this.showTime()
        }),
        this.message = ''
        let question = message
        question=question.replace(/ /g,"%20");
        let collectedData = []
        let defaultMessage='';        
      await axios.get(`${url}session_id=${sessionId}&key=${keyBot}&log=1&question=${question}`)
        .then(res => {
          for (const [i, v] of res.data.current_response.parameters.entities.entries()) {
            axios.get(`https://pokeapi.co/api/v2/pokemon/${v.name}`)
              .then(response => {
                console.log(i);
                let typeWeight = res.data.current_response.messages;
                typeWeight = typeWeight.find(o => o.text === 'libras');
                collectedData.push(v.name)

                for (const [key, value] of Object.entries(response.data)) {
                    if(key == res.data.current_response.intent_name) {                    
                      collectedData.push(value)
                    }
                      defaultMessage = 'hola';
                }
          switch(res.data.current_response.intent_name) {
            case 'types': {
              let answer
              answer = this.buildAnswer(collectedData, "es de tipo", "type.name")  
              defaultMessage = `${answer}`}
              collectedData = [];
              break;
            case 'weight': {
              let answer
              answer = this.buildAnswer(collectedData, "pesa", "weight")
              let weightPokemon = answer.toLowerCase().replace(/[abcdefghijklmnopqrstuvwxyz]/g, '')
                let Pokemon = answer.toLowerCase().replace(/[1234567890]/g, '')
              if (typeWeight !== undefined) {
                
                defaultMessage = `${Pokemon} ${((weightPokemon/10)*2.2).toFixed(2)} libras`
              }
              else {  
                defaultMessage = `${Pokemon} ${((weightPokemon/10)).toFixed(2)} kg`
              }
            }
              collectedData = []
              break;
            case 'height': {
              let answer
              answer = this.buildAnswer(collectedData, "mide", "height")
              defaultMessage = `${answer}`
            }
              collectedData = []
              break;
            case 'abilities': {
              let answer
              answer = this.buildAnswer(collectedData, "tiene las habilidades", "ability.name")
              defaultMessage = `${answer}`
            }
              collectedData = []  
              break;  

            case 'stats': {
              let answer              
              answer = this.buildAnswer(collectedData, "tiene estadisticas", "stat.name")
              let base_effort = this.buildAnswer(collectedData, "tiene estadisticas", "base_stat")
              let effort = this.buildAnswer(collectedData, "tiene estadisticas", "effort")
              defaultMessage = `${answer} De manera consecutiva ${base_effort} base effort. De igual manera ${effort} effort respectivamente`
            }
              collectedData = []  
              break; 
            case 'moves': {
              let answer
              answer = this.buildAnswer(collectedData, "tiene ataques", "move.name")
              defaultMessage = `${answer}`
            }
              collectedData = []  
              break;   
              default:
                defaultMessage = `Eso no lo he entendido`
          }          
          this.messages.push({
            text: defaultMessage,
            author: 'server',
            hour: this.showTime()
          })
            })
            .catch(error => {
              console.log(error);
            })
          }
        })       
          .catch(error => {
            this.messages.push({
            text: 'Eso no lo he entendido :(',
            author: 'server',
            hour: this.showTime()
            })
              console.log(error)
          })

        this.$nextTick(() => {
          console.log(collectedData)
          this.$refs.chatbox.scrollTop = this.$refs.chatbox.scrollHeight
        })
      }
    }
  }
</script>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Raleway&display=swap');
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Raleway', sans-serif;
  outline: none;
  border: none;
}
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
.chat-box {
  width: 400px;
  height: 555px;
  background-color: white;
  border: solid black 1px;
  overflow: hidden;
  border-radius: 10px;
  position: absolute;
}
.client {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 70px;
  background-color: #77b3d4;
}
.client-info {
  color: #fff;
  padding: 15px;
}
.client-info p {
  color: #008000;
  font-weight: bold;
  font-size: 0.8em;
}
.chats {
  width: 100%;
  height: 425px;
  padding: 0 15px;
  color: #fff;
  position: relative;
  font-size: 0.9em;
  background: white;
  overflow-y: scroll;
}
@media (max-width: 600px) {
  .chat-box {
    width: 280px;
    height: 350px;
  }
  .date {
    display: none;
  }
  .chats {
      height: 220px;
  }
}
.my-chat {
  width: 60%;
  word-wrap: break-word;
  background-color: green;
  padding: 7px 10px;
  border-radius: 10px 10px 10px 10px;
  margin: 5px 0 5px auto;
}
.server{
  width: 60%;
  word-wrap: break-word;
  background-color:purple;
  padding: 7px 10px;
  margin: 10px 0;
}
.date {
  padding: 2px;
  text-align: center;
  margin-left: 140px;
  font-size: 0.7em;
}
.chat-input {
  display: flex;
  align-items: center;
  width: 100%;
  height: 60px;
  background-color: orchid;
  padding: 15px;
  overflow: hidden;
  position: absolute;
  bottom: 0;
}
.chat-input input {
  width: calc(100% - 40px);
  height: 100%;
  background-color: white;
  border-radius: 5px;
  color: #000;
  font-size: 1.2em;
  padding: 0 15px;
}
.send {
  width: 40px;
  height: 40px;
  overflow: hidden;
  position: relative;
  margin-left: 5px;
  cursor: pointer;
  border-radius: 50%;
  transition: 0.4s ease-in-out;
}
</style>




