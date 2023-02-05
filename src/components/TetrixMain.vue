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
        <table class="stage_container">
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
    window.addEventListener('keydown', this.keydownEvent);
  },
  data() {
    return {            
      now: [],      
      count: 0,      
      stackData: [],
      blocks: [4, 4, 2, 2, 0 ],
      flag: true,
    };
  },
  methods: {
    init() {                  
      this.makeBlock();
    },
    // 블럭 생성
    async makeBlock() {            

      // if(this.count == 5) {
      //   return 0;
      // }      
      
      let num = Math.floor(Math.random() * 7);               
      // let num = this.blocks[this.count];
      let block = ''; //eslint-disable-line no-unused-vars                  
      let color = this.colorSelect(num);            

      //랜덤으로 블럭 생성
      for(let i = 0; i < 4; i++) {                             
          block = model[0][num][0][i].toString();  
          this.now.push(model[0][num][0][i]);         
          document.getElementById(block).style.backgroundColor = color; 
      }         
      

      const stackMax = Math.max(...this.now.map(e => e[0]).flat());      
      let result = this.stackCheck(this.now.filter(e => e[0] == stackMax).map(e => [e[0] + 1, e[1]]));

      if(result == "return") {        
        return ;
      }
                  
      this.flag = true;      

      while(this.flag == true) {
        await this.autoMove(this.now, color);                     
      }
                  
      this.now = [];                                                

      return this.makeBlock()

    },
    // 색깔 지정
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
      } else if(num == 4) {
        color = "purple";
      } else if(num == 5) {
        color = "pink";
      }

      return color;
    },
    // 자동으로 움직이는함수
    async autoMove(now, color) {          
      
      // 떨어지는 시간 동기처리
      await this.delay(10);
                                                                                            
      const stackMax = Math.max(...this.now.map(e => e[0]).flat());       
      
      if(stackMax == 20) {          
        this.flag = false;
        return;
      }


      let plus = this.now.map(e=> [e[0] + 1, e[1]]);
      let dd = this.now.map(e=> [e[0], e[1]]);
      // console.log(dd[1]);              
      // console.log(plus[0]);    
      // console.log(dd[1].equals(plus[0]));
      console.log(JSON.stringify(plus[0] == JSON.stringify(dd[1])));
      let ss = [];

      // console.log(plus[0].includes(this.now[0]));

      for(let i = 0; i < plus.length; i++) {
        for(let j = 0; j < plus.length; j++) {
          if(plus[i] == dd[j]) {
            alert("sdfsd");
            break;
          }

          if(j == 3) {
            ss.push("sdfdsf : " + plus[i]);
          }          
        }
      }
      console.log(ss);
      ss = [];

      //밑에 블럭이 있는지 계산
      let result = this.stackCheck(this.now.filter(e => e[0] == stackMax).map(e => [e[0] + 1, e[1]]));
            
      // 밑에 블럭이 있을때
      if(result == "return") {
        this.now = this.now.map(e=> [e[0] + 1, e[1]]);                      
        return;
      }

      // 블럭 초기화
      this.colorStack("green");
      this.now = this.now.map(e=> [e[0] + 1, e[1]]);      
      // 블럭 쌓기
      this.colorStack(color);
    },
    // 떨어지는 속도
    delay(ms) {
      return new Promise(r => setTimeout(r,ms));
    }, 
    // 블럭 쌓기   
    colorStack(color) {
      let block = '';      
      for(let i = 0; i < 4; i++) {                             
        block = this.now[i].toString();             
        document.getElementById(block).style.backgroundColor = color; 
      }   
    },    
    // 밑에 블럭있는지 체크
    stackCheck(check) {            
      let block = '';                  
      for(let i = 0; i < check.length; i++) {                             
        block = check[i].toString();             
        let down = document.getElementById(block).style.backgroundColor;        //eslint-disable-line no-unused-vars                   
        if(down != "green") {            
          this.flag = false;       
          return "return";
        }
      }           
      return "";
    },
    keydownEvent(event) {
      if(event.keyCode == 38) {
        // alert("윗키");
      } else if(event.keyCode == 37) {
        if(this.now.filter(e=> e[1] == 1).length > 0) {
            return;
        }                
        this.colorStack("green");
        this.now = this.now.map(e => [e[0], e[1] - 1]);        
      } else if(event.keyCode == 39) {
        if(this.now.filter(e=> e[1] == 10).length > 0) {
            return;
        }                
        this.colorStack("green");
        this.now = this.now.map(e => [e[0], e[1] + 1]);               
      } else if(event.keyCode == 40) {
        // alert("아랫키");
      }  
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