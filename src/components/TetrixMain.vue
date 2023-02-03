<template>
  <body>
    <div class="container">
      <div class="popup">
        게임종료
        <button class="restart">다시시작</button>
      </div>

      <di class="level_container">
        <p></p>
        <div class="level"></div>
      </di>

      <div class="board">
        <div class="header">
          <p>score</p>
          <p class="score">0</p>
        </div>
        <table class="stage_container" @keydown.right="keyRight()" >
          <tbody class="stage">
            <tr v-for="(y, index) in 20" :key="`y-${index}`" :id="`${y},${x}`" style="background-color: green;">
              <td v-for="(x, index) in 10" :key="`x-${index}`" :id="`${y},${x}`" style="background-color:green;">
              </td>
            </tr>
          </tbody>
        </table>
      </div>      
      <div class="next"></div>
    </div>
  </body>
</template>



<script>
import TetrixModel from './TetrixModel/TetrixModel';

const model = TetrixModel;

export default {
  components: {    
  },
  mounted() {
    this.init();    
  },
  data() {
    return {            
      now: [],
      max: [],
      count: 0,      
      stackData: [],
    };
  },
  methods: {
    init() {            
      
      this.makeBlock();
    },
    async makeBlock() {            

      if(this.count == 1) {
        return 0;
      }      
      
      let num = Math.floor(Math.random() * 5);               
      let block = ''; //eslint-disable-line no-unused-vars                  
      let color = this.colorSelect(num);            

      for(let i = 0; i < 4; i++) {                             
          block = model[0][num][0][i].toString();  
          this.now.push(model[0][num][0][i]);         
          document.getElementById(block).style.backgroundColor = color; 
      }                  
                  
      await this.autoMove(this.now, color);                
      this.count ++;           
            
            
      return this.makeBlock()

    },
    colorSelect(num) {
      let color = "black";

      if(num == 0) {
        color = "yellow";
      } else if(num == 1) {
        color = "orange";
      } else if(num == 2) {
        color = "red";
      } else if(num == 3) {
        color = "blue";          
      } 

      return color;
    },
    async autoMove(now, color) {          
      
      await this.delay(500);
      
      if(this.max.length > 0) {
        this.max = [];
        this.now = [];        
        return;
      }                        

      console.log("SDfsd : " + Math.max.apply(null, this.now().map((e)=> e[idx])));

      this.stackCheck(this.now.map(e=> [e[0] + 1, e[1]]));
      this.colorStack("green");
      this.now = this.now.map(e=> [e[0] + 1, e[1]]);      
      this.colorStack(color);
      this.max = this.now.filter(e => e[0] == 20);       
       
       return this.autoMove(this.now, color);
    },
    delay(ms) {
      return new Promise(r => setTimeout(r,ms));
    },    
    colorStack(color) {
      let block = '';      
      for(let i = 0; i < 4; i++) {                             
        block = this.now[i].toString();             
        document.getElementById(block).style.backgroundColor = color; 
      }   
    },    
    stackCheck(check) {      
      let block = '';      
      for(let i = 0; i < 4; i++) {                             
        block = check[i].toString();             
        let down = document.getElementById(block).style.backgroundColor;
        if(down == "green") {
          console.log(this.now);
          console.log(check);

        }
      }           
    },
    keyRight() {
      alert("오른쪽");
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
* {
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  justify-content: center;
  height: 100%;
  background-color: white;  
}

.container {
  background-color: skyblue;
  width: 700px;
  height: 700px;
  margin: 3em;
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.header {
  text-align: center;
  font-size: 28px;
  margin-bottom: 1rem;
}

.popup {
  display: none;
}
.stage {
  margin: 0 auto;
  outline: 2px solid white;
  background-color: rgba(134, 134, 134, 0.5);
}
.stage > tr {
  width: 100%;
  height: 25px;
}
.stage > tr > td {
  width: 25px;
  height: 25px;
  outline: 1px solid white;
}
</style>
