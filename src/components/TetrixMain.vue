<template>
  <div class="title">
    DongTrix
  </div>
  <body>
    <div class="container">
      <div class="popup" v-if="endflag">
        게임종료
        <button class="restart">다시시작</button>
      </div>       

      <div class="board">
        <div class="header">          
          <p class="level">{{level}}</p>
        </div>        
        <table class="stage_container">
          <tbody class="stage">
            <tr v-for="(y, index) in 20" :key="`y-${index}`" style="background-color: green;">
              <td v-for="(x, index) in 10" :key="`x-${index}`" :id="`${y},${x}`" style="background-color:green;">
              </td>
            </tr>
          </tbody>
        </table>
      </div>      
      <div class="next_header">Next
        <table class="next_container" style="height: 10px; margin-top: 1em;">
          <tbody class="next">
            <tr v-for="(y, index) in 5" :key="`nextY-${index}`">
              <td v-for="(x, index) in 5" :key="`nextX-${index}`" :id="`n-${y},${x}`"></td>
            </tr>              
          </tbody>
        </table>
        <div class="score">
          점수 : {{ score }}
        </div>
      </div>        
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
      next: [],
      total: [],
      flag: true,      
      time: 1000,
      status: {
        color: "",
        inedx: 0,
        change: 1,
      },   
      score: 0, 
      level: "Level 1",
      blockNum : [],
      blockIdx : 0,
      endflag: false,
    };
  },
  methods: {
    init() {    
      for(let i = 0; i < 100000; i++) {
        this.blockNum.push(Math.floor(Math.random() * 7));
      }
      
      this.makeBlock();
    },
    // 블럭 생성
    async makeBlock() {                  
              
      this.nextBlock(this.blockNum[this.blockIdx + 1]);
      
      let num = this.blockNum[this.blockIdx];
      let block = ''; 
      this.status.color = this.colorSelect(num);   
      this.status.inedx = num;       
      this.status.change = 1; 
      this.now = [];  

      //랜덤으로 블럭 생성
      for(let i = 0; i < 4; i++) {                             
          block = model[0][num][0][i].toString();  
          this.now.push(model[0][num][0][i]);         
          document.getElementById(block).style.backgroundColor = this.status.color; 
      }         
      
      // 밑에 블럭 있는지 체크
      let check = this.blockLimitArr(this.now.map(e=> [e[0] + 1, e[1]]));
      let result = this.stackCheck(check);                    
      this.flag = true;      

      while(this.flag == true) {
        await this.autoMove();                     
      }                                                

      this.now.forEach((e) =>{      
        this.total.push([e[0], e[1], this.status.color]);
      });                  

      if(result == "return") {            
        window.removeEventListener('keydown', this.keydownEvent);        
        if(confirm('다시 시작하겠습니까?')) {      
          window.location.reload();
        } else {
          return;
        }
      }

      await this.scoreCheck();                              

      this.blockIdx += 1;      
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
        this.firstCheck = false;
        return;
      }    

      //밑에 블럭이 있는지 계산
      let check = this.blockLimitArr(this.now.map(e=> [e[0] + 1, e[1]]));
      let result = this.stackCheck(check);      
                        
      // 밑에 블럭이 있을때
      if(result == "return") {                 
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
        if(e[0] > 20) {
          let cal = e[0] - 20;
          newCheck = check.map(e=> [e[0] - cal, e[1]]);
        } else if(e[1] > 10) {
          let cal = e[1] - 10;          
          newCheck = check.map(e=> [e[0], e[1] - cal]);
        } else if(e[1] == 0)  {                    
          newCheck = check.map(e=> [e[0], e[1] + 1]);
        }
      }));      

      if(newCheck.length > 0) {
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
        const stackMax = Math.max(...this.now.map(e => e[0]).flat());       
      
        if(stackMax == 20) {          
          this.flag = false;
          return;
        }    
        
        const check = this.blockLimitArr(this.now.map(e=> [e[0] + 1, e[1]]));
        const result = this.stackCheck(check);   

        if(result == "return") {
          return;
        }

        this.colorStack("green");
        this.now = this.now.map(e => [e[0] + 1, e[1]]);   
        this.colorStack(this.status.color);                                          
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

        this.changeStack(changeArr);
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

        this.changeStack(changeArr);
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

        this.changeStack(changeArr);
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

        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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

        this.changeStack(changeArr);
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

        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr);
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
        
        this.changeStack(changeArr); 
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
        
        this.changeStack(changeArr);
      }
    },
    changeStack(changeArr) {      

      this.colorStack("green");
      this.now = this.outCheck(changeArr);
      this.colorStack(this.status.color);
      this.status.change = this.status.change + 1;      
    },
    async scoreCheck() {             
      let total = this.total.map(e => [e[0], e[1], e[2]]);                  
      let length = this.now.map(e=> e[0]).length;           
                    
      for(let i = 0; i < length; i++) {        
        for(let j = 20; j >= 1; j--) {
          for(let n = 1; n <= 10; n++) {
            let block = j + "," + n;            
            let flag = document.getElementById(block).style.backgroundColor == "green" ? true : false;     

            if(flag) {
              break;
            } else {
              if(n == 10) {                
                for(let a = 1; a <= 10; a++) {
                  block = j + "," + a;
                  document.getElementById(block).style.backgroundColor = "green";
                }
                
                let reMake = [];
                total.filter((e) => {
                  if(e[0] > j) {
                    reMake.push([e[0], e[1], e[2]]);
                  } else if(e[0] < j) {                    
                    reMake.push([e[0] + 1, e[1], e[2]]);
                  }
                });
                
                total = reMake;                                               

                for(let b = 0; b < total.length; b++) {
                  block = (total[b][0] - 1) + "," + total[b][1];
                  document.getElementById(block).style.backgroundColor = "green";
                }

                for(let c = 0; c < total.length; c++) {
                  block = total[c][0] + "," + total[c][1];
                  document.getElementById(block).style.backgroundColor = total[c][2];
                }
  
                this.score += 10;

              }
            }  
          }
        }
      }
      this.levelUp();
      this.total = total;                   
    },                   
    levelUp()  {
      let score = this.score;
      let cal = Math.floor((score / 100)) ;

      this.level = "Level " + (cal + 1);
      this.time = 1000 - (cal * 120);
     
    },
    nextBlock(num) {     
      for(let i = 0; i < this.next.length; i++) {
        document.getElementById(this.next[i]).style.backgroundColor = "skyblue";
      }      

      this.next = [];

      for(let i = 0; i < 4; i++) {                             
        let block = model[0][num][0][i];
        block = "n-" + (block[0] + 1) + "," + (block[1] - 3);        
        document.getElementById(block).style.backgroundColor = this.colorSelect(num); 
        this.next.push(block);
      }         
    },
  },  
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.title {
  font-style: oblique;
  font-size: 35px;
  background-color: black;
  color: white;
}

* {
  margin: 0;
  padding: 0;
}

body {
  display: flex;
  justify-content: center;
  height: 100%;
  background-color:black;  
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

/* .popup {
  display: none;
} */
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
.next_header {
  font-weight: bold;
  margin-top: 4em;
  margin-left: 4em;
  font-size: 20px;    
}
.next {    
  margin: 0 auto;
  outline: 2px solid white;
  background-color: skyblue
}
.next > tr {
  width: 100%;
  height: 25px;
}
.next > tr > td {
  width: 25px;
  height: 25px;
  /* outline: 1px solid white; */
}

.level {
  margin-top: 1em;
  font-weight: bold;
}

.score {
  height: 100px;
  margin-top: 5em;
}
</style>
