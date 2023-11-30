<template>
  <div >
  
    <h1 v-if="isGameOver" class='gameTitle'> You Won! Good Job</h1>
    <h1 v-if="!isGameOver" class='gameTitle'>Guess a Capital game:You have {{lives}} lives left!</h1>
    <h1 v-if="isGameOver || lives === 0" >You time is {{ handleTimeElapsedCheck()}} seconds!</h1>
      <button class="resetButton" id="glow-on-hover" v-bind:style="{ width: isClicked ? '200px' : '120px' }" v-on:click="resetTasks()">{{resetMessage}}</button>
    <p v-if="lives === 0 || isGameOver"> 
         <button
            class='gameLostButton'
            v-on:click="handleLostGame()"
             >{{lives === 0 ? 'You lost try again' : 'Good game, restart!'}}</button> 
    </p>
    <p v-else> 
    </p>
        <div class='gameContainer'>
            <div class='gameButtonsContainer'>
              <button 
               v-show="item.isChecked"
               v-for="(item, index) in gameData"
               v-bind:key="index"
               class='gameButton'
               v-bind:class="item.state"
               v-on:click="handleButtonClick(item.value)"
               >
               
               
               {{item.value}}</button>
            </div>
            <div class='gameCheckboxContainer'>
                 <label class='selectAllCheckBox'>
                    <input  type="checkbox" v-bind:checked="isCheckedAll" v-on:change="handleIsCheckedAll()">
                    Select All
                </label>
            <li  >
             
                <li v-for="(item, index) in gameData" v-bind:key="index">
                    <label>
                        <input type="checkbox" v-bind:value="item.value" v-bind:checked="item.isChecked" v-on:change="handleIsCheckedButton(item.value)">
                        {{item.value}}
                    </label>
                </li>
            </li>
             </div>
           
        </div>       
  </div>
</template>

<script>
export default {
data() {
    return {
      initialState : { 
          "Germany": "Berlin",
          "Russian":"Moscow",
          "USA":"Washington",
          "New Zealand":"Wellington",
          "Canada":"Ottawa",
          "Spain":"Madrid",
          "Japan":"Tokyo",
          "France":"Paris",
          "Austalia":"Canberra",
          "Mexico":"Mexico City",
          "Brazil":"Brasillia",
      },
       gameData:[],
       lives: 10,
       selected:null,
       isGameOver:false,
       resetMessage:'Reset tasks',
       isClicked:false,
       isCheckedAll:false,
       startTime: null,
       endTime: null,
       timeElapsed: 0,

  
    };
  },
  watch:{
   
  },
  mounted(){
  let keys = Object.keys(this.initialState);
  let values = Object.values(this.initialState);

  this.gameData = [...this.gameData, ...keys, ...values].map((value)=>({
          value,
          state:'DEFAULT'
          })).sort(()=>Math.random() - 0.5)
  

      
  // console.log(this.gameData);
  

  if (localStorage.lives) {
    this.lives = localStorage.lives;
  }
  if (localStorage.isCheckedAll) {
    this.isCheckedAll = localStorage.isCheckedAll;
  }
  if(localStorage.isSelected){
    
       this.selected = localStorage.isSelected;
  }
  if (localStorage.getItem('gameData')) {
  
   
    //  console.log(localStorage.getItem('gameData'))
     this.gameData =  (JSON.parse(localStorage.getItem('gameData'))).map((opt) => {
            return opt.value === localStorage.isSelected ? {
                   ...opt,
                   state:'SELECTED'
               } : opt; 
           
           })

    
       
    }
  

  },
  computed: {
    // getGameTime: function(){
    //   return   this.timeElapsed = this.endTime - this.startTime;
    // }
    },
  methods: {
    handleTimeElapsedCheck(){
            this.endTime = new Date(); 
            
            console.log(this.endTime); 
            console.log(this.startTime - this.endTime);
            return this.timeElapsed = (Math.abs(this.endTime- this.startTime) / 1000).toFixed(2);
    },
    handleIsCheckedAll()  {

      this.isCheckedAll = !this.isCheckedAll;
        this.gameData = [...this.gameData].map(data=>
          ({...data,
              isChecked: !data.isChecked})
      )
      localStorage.isCheckedAll = true;
      localStorage.setItem('gameData', JSON.stringify(this.gameData));  
    },

    handleIsCheckedButton(value) {

    this.gameData = [...this.gameData].map(data => 
       value === data.value ? 
         ({...data,
            isChecked: !data.isChecked}) : {...data})
    console.log(this.gameData)
    localStorage.setItem('gameData', JSON.stringify(this.gameData));  
    },

    resetTasks(){
        if (!this.isClicked) {
          localStorage.removeItem('gameData');
          localStorage.removeItem('lives');
          localStorage.removeItem('isSelected');
          localStorage.removeItem('isCheckedAll');
          this.resetMessage = 'App will restart is 2 seconds';
          this.isClicked = !this.isClicked;
          setTimeout(function(){
          window.location.reload();
          }, 2000)
      }
      },

  handleButtonClick(data){
   

    if (!this.selected){
        this.selected = data;
        localStorage.isSelected = this.selected;
        this.gameData = this.gameData.map((opt) => {
            return opt.value === data ? {
                   ...opt,
                   state:'SELECTED'
               } : opt; 
               
           })
     if(this.startTime === null){
      this.startTime = new Date();
     
     }   
 

    } else if ((this.selected === this.initialState[data]) || (this.initialState[this.selected] === data))
     {
   
        this.gameData = this.gameData.filter((opt)=>{
         return !(opt.value === this.selected || opt.value === data)
        }).map((opt) => {
            return  {
                   ...opt,
                   state:'DEFAULT'
               } 
               
           })
             console.log(`data after one right choice ${this.gameData}`)
            localStorage.setItem('gameData', JSON.stringify(this.gameData));
            if (this.gameData.find((data)=>!data.isChecked)){
            this.endTime = new Date(); 
         
       
            this.isGameOver = true;

          }
            this.selected = null;
            localStorage.removeItem('isSelected');
           
      } 
        else
       {
            if (this.lives >= 1){
    
            this.lives -= 1;
            localStorage.lives = this.lives;
            this.gameData =  this.gameData.map((opt) => {
              
                return opt.value === data ? {
                      ...opt,
                      state:'WRONG'
                  } : opt;       
                
            }) 
        }
    
      }
 },

 handleLostGame() {

  
    this.lives = 10;
    let keys = Object.keys(this.initialState);
    let values = Object.values(this.initialState);
    this.gameData = this.gameData = [ ...keys, ...values].map((value)=>({
          value,
          state:'DEFAULT'
          })).sort(()=>Math.random() - 0.5)
      this.selected = null;
      this.isGameOver = false;
      localStorage.removeItem('gameData');
      localStorage.removeItem('lives');
      localStorage.removeItem('isSelected');
      localStorage.removeItem('isCheckedAll');
      this.isCheckedAll = false;
      this.startTime = null;
      this.endTime = null;
      this.timeElapsed = null;
}
 
 }
     
};
</script>

<style scoped>

.gameContainer{
  display: flex;
  position: relative;
  flex-direction: row;
  align-content: flex-start;
  padding: 0 3rem;
  width: 95vw;
  margin-top: 25px;
  height: 600px;
  gap:50px;


  @media (max-width:698px){
    padding: 0 1.5rem;
  }
  @media (max-width:965px){
 
   height: 700px;
  }

}
.gameTitle{
  padding: 0 3rem;
  margin-top: 10px;
  @media (max-width:698px){
    padding: 0 1.5rem;
  }
}
.gameLostButton{
  position: relative;
  background-color: black;
  color: white;
  margin: 25px auto;
  &:before{
    content: '';
    background: linear-gradient(45deg, #ff0000, #ff7300, #fffb00, #48ff00, #00ffd5, #002bff, #7a00ff, #ff00c8, #ff0000);
    position: absolute;
    top: -2px;
    left:-2px;
    background-size: 400%;
    z-index: -1;
    filter: blur(5px);
    width: calc(100% + 6px);
    height: calc(100% + 6px);
    animation: animate 20s linear infinite;
    opacity: 1;
    transition:  1s ease-in-out;
    border-radius: 10px;
  }
  &:after{
    z-index: -1;
    content: '';
    position: absolute;
    width: 100%;
    height: 100%;
    background: #111;
    left: 0;
    top: 0;
    border-radius: 10px;
  }
  &:active{
    color: azure;
  }

}

.gameCheckboxContainer * input{

    width: auto;
  
}
.gameCheckboxContainer li{

  text-align: left;
  display: flex;
  flex-direction: column;
  

}
.gameButtonsContainer{
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  width: 700px;
  height: 640px;
  max-width: 62vw;
    &:has(.gameButton:hover) > .gameButton:not(:hover){
    opacity: 0.5;
    scale: .92;
   
  }
}
.gameCheckboxContainer{
  display: flex;
  flex-direction: column;
  overflow-y: scroll;
  height: 540px;
  width: 350px;
  max-width: 35vw;
}
.selectAllCheckBox{
  position: relative;
  background-image: linear-gradient(34deg, rgba(2,0,36,1) 0%, rgba(250,0,0,1) 35%, rgba(29,0,250,1) 100%);
  background-size: 100% 3px;
  background-repeat: no-repeat;
  background-position: left bottom;
  animation: animateCheckBox 5s infinite ;
  text-decoration: none;

  
  
}
.gameButton{
  border-radius: 8px;
  width: 105px;
  height:65px;
  cursor: pointer;
  transition: opacity .2s ease-in,scale .3s ease-in;
  &:hover{
  border: 2px solid purple;
 }
}
.WRONG{
 background-color: red;
}
.SELECTED{
  background-color: #28aae1;
}
@keyframes animateCheckBox {
  0% {
    background-size: 100% 3px;
  }

  50% {
    background-size: 600% 3px;
  }

  100% {
    background-size: 100% 3px;
  }
  }
</style>