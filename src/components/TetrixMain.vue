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
      flag: true,
      down: false,
      time: 1000,
      status: {
        color: "",
        inedx: 0,
        change: 1,
      },      
    };
  },
  methods: {
    init() {                  
      this.makeBlock();
    },
    // 블럭 생성
    async makeBlock() {            
      this.down = false;

      // let num = Math.floor(Math.random() * 7);               
      let num = 2;
      let block = ''; //eslint-disable-line no-unused-vars                  
      this.status.color = this.colorSelect(num);   
      this.status.inedx = num;       
      this.status.change = 1; 

      //랜덤으로 블럭 생성
      for(let i = 0; i < 4; i++) {                             
          block = model[0][num][0][i].toString();  
          this.now.push(model[0][num][0][i]);         
          document.getElementById(block).style.backgroundColor = this.status.color; 
      }         
      
      // 밑에 블럭 있는지 체크
      let check = this.blockLimitArr(this.now.map(e=> [e[0] + 1, e[1]]));
      let result = this.stackCheck(check);

      if(result == "return") {        
        return ;
      }
                  
      this.flag = true;      

      while(this.flag == true) {
        await this.autoMove();                     
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
    async autoMove() {          
            
      // 떨어지는 시간 동기처리
      await this.delay(this.time);      
                                                                                            
      const stackMax = Math.max(...this.now.map(e => e[0]).flat());       
      
      if(stackMax == 20) {          
        this.flag = false;
        return;
      }    

      //밑에 블럭이 있는지 계산
      let check = this.blockLimitArr(this.now.map(e=> [e[0] + 1, e[1]]));
      let result = this.stackCheck(check);      
                        
      // 밑에 블럭이 있을때
      if(result == "return") {
        this.now = this.now.map(e=> [e[0] + 1, e[1]]);                      
        return;
      }

      // 블럭 초기화
      this.colorStack("green");
      this.now = this.now.map(e=> [e[0] + 1, e[1]]);      
      // 블럭 쌓기
      this.colorStack(this.status.color);
    },
    // 떨어지는 속도
    delay(ms) {      
      if(this.down) {
        ms = 0;        
      }
      
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
    outCheck(check) {  
      let newCheck = [];
      check.filter((e=> {
        if(e[0] == 0) {
          newCheck = check.map(e=> [e[0] + 1, e[1]]);
        } else if(e[0] == 20) {
          newCheck = check.map(e=> [e[0] - 1, e[1]]);
        } else if(e[1] == 0) {
          newCheck = check.map(e=> [e[0], e[1] + 1]);
        } else if(e[1] == 10) {
          newCheck = check.map(e=> [e[0] , e[1] - 1]);
        }                    
      }));
      console.log(check);
      console.log(newCheck);


      if(check.length > 0) {
        return newCheck;
      }

      return check;
    },
    // 블럭있는지 체크
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
          this.blockChange();

        // alert("윗키");
      } else if(event.keyCode == 37) {        
        const stackMin = Math.min(...this.now.map(e => e[1]).flat());              
        if(stackMin - 1 == 0) {
            return;
        }                
        
        let result = this.stackCheck(this.blockLimitArr(this.now.map(e=> [e[0], e[1] - 1])));

        if(result == "return") {
          return;
        }                

        this.colorStack("green");
        this.now = this.now.map(e => [e[0], e[1] - 1]);                
        this.colorStack(this.status.color);        
      } else if(event.keyCode == 39) {        
        const stackMax = Math.max(...this.now.map(e => e[1]).flat());              
        if(stackMax == 10) {
            return;
        }                
        
        let result = this.stackCheck(this.blockLimitArr(this.now.map(e=> [e[0], e[1] + 1])));

        if(result == "return") {
          return;
        }            
        this.colorStack("green");
        this.now = this.now.map(e => [e[0], e[1] + 1]);   
        this.colorStack(this.status.color);                    
      } else if(event.keyCode == 40) {                
        this.down = true;
      }  
    },
    // 
    blockLimitArr(arr) {      
      let arr2 = this.now.map(e=> [e[0], e[1]]);      
      let compare = [];
      
      for(let i = 0; i < arr.length; i++) {
        for(let j = 0; j < arr.length; j++) {
          if(JSON.stringify(arr[i]) == JSON.stringify(arr2[j])) {            
            break;
          }

          if(j == 3) {
            compare.push(arr[i]);
          }          
        }
      }           
      return compare;
    },
    blockChange() {      
      const index = this.status.inedx;

      // ㄴ자
      if(index == 1) {
        this.change1();
      // ㅣ자
      } else if(index == 2) {
        this.change2();
      // ㅗ자
      } else if(index == 3) {
        this.change3();
      // 반대 ㄴ자
      } else if(index == 4) {
        this.change4();
      // z자
      } else if(index == 5) {      
        this.change5();
      // z 반대  
      } else if(index == 6) {
        this.change6();
      }

                     
    },
    change1() {            
      let idx = this.status.change;

      if(idx%4 == 1) {        
        let changeArr = [];
        
        const topMin = Math.min(...this.now.map(e => e[0]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
          if(e[1] == rightMax) {              
            changeArr.push([e[0] + 1, e[1] - 1]);
          } else if(e[0] == topMin){
            changeArr.push([e[0], e[1] + 1]);
          } else if(e[1] == leftMin){
            changeArr.push([e[0] + 1, e[1]]);
          } else {
            changeArr.push([e[0], e[1]]);
          }      
        });                           

        this.colorStack("green");
        this.now = changeArr  
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;        
      } else if(idx%4 == 2) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
          if(e[1] == leftMin) {              
            changeArr.push([e[0] - 2, e[1]]);
          } else if(e[0] == topMax && e[1] == rightMax){
            changeArr.push([e[0] - 1, e[1] + 1]);
          } else if(e[0] == topMax - 1 && e[1] == rightMax) {
            changeArr.push([e[0] - 1, e[1] + 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });                      

        this.colorStack("green");
        this.now = changeArr  
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 3) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax) {              
            changeArr.push([e[0], e[1] - 2]);
          } else if(e[1] == rightMax){
            changeArr.push([e[0] + 2, e[1] - 2]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });                      

        this.colorStack("green");
        this.now = changeArr  
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 0) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax) {              
            changeArr.push([e[0] - 1, e[1] + 2]);
          } else if(e[1] == rightMax){
            changeArr.push([e[0] + 1, e[1]]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });                      

        this.colorStack("green");
        this.now = changeArr  
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } 
    },
    change2() {      
      let idx = this.status.change;

      if(idx%2 == 1) {        
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax) {              
            changeArr.push([e[0] - 3, e[1] - 1]);
          } else if(e[0] == topMax - 1){
            changeArr.push([e[0] - 2, e[1] + 1]);
          } else if (e[0] == topMax - 2){
            changeArr.push([e[0] - 1, e[1] + 2]);
          } else {
            changeArr.push([e[0], e[1]]);
          }           
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else  if(idx%2 == 0) {                
        let changeArr = [];
        
        const leftmin = Math.min(...this.now.map(e => e[1]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());        

        this.now.filter((e) => {          
          if(e[1] == leftmin) {              
            changeArr.push([e[0] + 1, e[1] + 1]);
          } else if(e[1] == rightMax){
            changeArr.push([e[0] + 2, e[1] - 2]);
          } else if (e[1] == rightMax - 1){
            changeArr.push([e[0] + 3, e[1] - 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }           
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      }
    },
    change3() {                           
      let idx = this.status.change;
      if(idx%4 == 1) {        
        let changeArr = [];
        
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());
        this.now.filter((e) => {          
        if(e[1] == rightMax) {              
          changeArr.push([e[0] + 1, e[1] - 1]);
        } else {
          changeArr.push([e[0], e[1]]);
        }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 2) {        
        let changeArr = [];

        const stackMin = Math.min(...this.now.map(e => e[0]).flat());
        this.now.filter((e) => {          
        if(e[0] == stackMin) {              
          changeArr.push([e[0] + 1, e[1] + 1]);
        } else {
          changeArr.push([e[0], e[1]]);
        }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 3) {                
        let changeArr = [];
        
        const stackMin = Math.min(...this.now.map(e => e[1]).flat());
        this.now.filter((e) => {          
          if(e[1] == stackMin) {              
            changeArr.push([e[0] - 1, e[1] + 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      

        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 0){        
        let changeArr = [];
        
        const stackMax = Math.max(...this.now.map(e => e[0]).flat());
        this.now.filter((e) => {          
          if(e[0] == stackMax) {              
            changeArr.push([e[0] - 1, e[1] - 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });     

        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color); 
        this.status.change = this.status.change + 1;
      }              
    },
    change4() {               
      let idx = this.status.change;
      if(idx%4 == 1) {                
        let changeArr = [];
        
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());
        const topMin = Math.min(...this.now.map(e => e[0]).flat());

        this.now.filter((e) => {          
        if(e[0] == topMin) {              
          changeArr.push([e[0], e[1] - 1]);
        } else if(e[1] == rightMax){
          changeArr.push([e[0] + 1, e[1]]);
        } else if(e[1] == leftMin) {
          changeArr.push([e[0] + 1, e[1] + 1]);
        } else {
          changeArr.push([e[0], e[1]]);
        }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 2) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax && e[1] == rightMax -1) {              
            changeArr.push([e[0] - 1, e[1] - 1]);
          } else if(e[1] == rightMax){
            changeArr.push([e[0] - 2, e[1]]);
          } else if(e[0] == topMax - 1) { 
            changeArr.push([e[0] - 1, e[1] - 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 3) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
        if(e[0] == topMax) {              
          changeArr.push([e[0] , e[1] + 2]);
        } else if(e[1] == leftMin){
          changeArr.push([e[0] + 2, e[1] + 2]);
        } else {
          changeArr.push([e[0], e[1]]);
        }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      } else if(idx%4 == 0) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());

        this.now.filter((e) => {          
        if(e[0] == topMax) {              
          changeArr.push([e[0] - 1, e[1] - 1]);
        } else if(e[1] == leftMin){
          changeArr.push([e[0] + 1, e[1] - 1]);
        } else {
          changeArr.push([e[0], e[1]]);
        }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;
      }
    },
    change5() {            
      let idx = this.status.change;

      if(idx%2 == 1) {                
        let changeArr = [];
        

        const topMin = Math.min(...this.now.map(e => e[0]).flat());
        const leftMin = Math.min(...this.now.map(e => e[1]).flat());        

        this.now.filter((e) => {          
          if(e[0] == topMin && e[1] == leftMin) {    
            changeArr.push([e[0] + 2, e[1] + 1]);
          } else if(e[0] == topMin && e[1] == leftMin + 1) {
            changeArr.push([e[0], e[1] + 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;   
      } else  if(idx%2 == 0) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const topMin = Math.min(...this.now.map(e => e[0]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax) {              
            changeArr.push([e[0] - 2, e[1] - 1]);            
          } else if(e[0] == topMin){
            changeArr.push([e[0] , e[1] - 1]);            
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;   
      }
    },
    change6() {            
      let idx = this.status.change;

      if(idx%2 == 1) {                
        let changeArr = [];
        
        const topMin = Math.min(...this.now.map(e => e[0]).flat());
        const rightMax = Math.max(...this.now.map(e => e[1]).flat());        

        this.now.filter((e) => {          
          if(e[0] == topMin && e[1] == rightMax) {    
            changeArr.push([e[0] + 2, e[1] - 1]);
          } else if(e[0] == topMin && e[1] == rightMax - 1) {
            changeArr.push([e[0], e[1] - 1]);
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;   
      } else  if(idx%2 == 0) {                
        let changeArr = [];
        
        const topMax = Math.max(...this.now.map(e => e[0]).flat());
        const topMin = Math.min(...this.now.map(e => e[0]).flat());

        this.now.filter((e) => {          
          if(e[0] == topMax) {              
            changeArr.push([e[0] - 2, e[1] + 1]);            
          } else if(e[0] == topMin){
            changeArr.push([e[0] , e[1] + 1]);            
          } else {
            changeArr.push([e[0], e[1]]);
          }            
        });      
        
        this.colorStack("green");
        this.now = changeArr;    
        this.colorStack(this.status.color);
        this.status.change = this.status.change + 1;   
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