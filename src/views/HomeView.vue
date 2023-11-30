<template>
  <div class="taskListBox">
 <h1> My task List for this week</h1>
  <hr>
  <input class="inputField" type="text" v-model="newTask" v-on:keypress.enter="addTasks()"  placeholder="Enter the task and press enter" :maxlength="max"> 
  <ol class="grid">
    <transition-group name="listItemTransition" class="grid">
    <li class="grid-list" v-for="(task, index) in tasks" :key="index"  v-on:click="removeTasks(index)"> {{task.name}} (click to remove): </li>
    <button  class="grid-button" v-for="(task, index) in tasks" :key="index" v-on:click="changeStatus(index)" v-bind:class="{doneClass: tasks[index].done === 'Task is done'}">{{task.done}}</button>
    </transition-group>
  </ol>
  <div class="filterChoice" v-on:click="isCompleted()">Number of done tasks (click to filter all currently done tasks): {{numberOfDone}}</div>
  <div class="filterChoice" v-on:click="isPending()">Number of pending tasks (click to filter all currently pending tasks): {{numberOfPending}}</div>
  <button class="resetButton" id="glow-on-hover" v-bind:style="{ width: isClicked ? '200px' : '120px' }" v-on:click="resetTasks()">{{resetMessage}}</button>
  </div>
  
  <div class="filterDiv" >
    <table>
      <thead align="center">
        <tr>
          <th>Description</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <transition-group name="filterItemTransition">
        <tr
          v-for="task in getTasks"
          v-bind:key="task.id"
          v-bind:class="{ isPending: !showDone }"
        >
          <td>{{task.name}}: "{{task.done}}"</td>
        </tr>
        </transition-group>
      </tbody>
    </table>
  </div>
    
</template>



<script>
export default {
  data() {
    return {
      newTask: '',
      tasks: [],
      availableStatusTypes: ['Task is pending', 'In progress', 'Task is done'],
      numberOfDone: 0,
      numberOfPending: 0,
      action:"",
      tasksSelection:[],
      id:0,
      showDone: false,
      resetMessage:'Reset tasks',
      isClicked:false,
      max:18
    };
  },
  watch:{
      numberOfDone(newValDone){
        localStorage.numberOfDone = newValDone;
        console.log(localStorage.numberOfDone);
      },
       numberOfPending(newVal){
        localStorage.numberOfPending = newVal;
        console.log(localStorage.numberOfPending);
      }
  },
  mounted(){
     if (localStorage.getItem('tasks')) {
      try {
        this.tasks= JSON.parse(localStorage.getItem('tasks'));
      } catch(e) {
        localStorage.removeItem('tasks');
      }
    }
  
     this.numberOfDone = Number(localStorage.numberOfDone);
     this.numberOfPending = Number(localStorage.numberOfPending);
  },
  computed: {
      getTasks() {
        return this.tasks.filter(task => task.isDone === this.showDone);
      }
    },
  methods: {
       addTasks() {
          if(this.newTask.trim() != '')
            {
            this.numberOfPending++;
            this.tasks.push({
                id: this.id++,
                name: this.newTask.trim(),
                done:'Task is pending',
                isDone:false
            });
            this.newTask ='';
            this.saveTasks();
          }
            else 
            alert("U've enter the empty tasks.");
        },

        removeTasks(index){
          
            if (this.tasks[index].done == 'Task is pending' || this.tasks[index].done == 'In progress'){
              this.tasks.splice(index, 1);
              this.saveTasks();
              return this.numberOfPending--;
           
            }
            else {
              this.tasks.splice(index, 1);
              console.log("removed");
              this.saveTasks();
            }
        },

         changeStatus(index){
        
           let newIndex = this.availableStatusTypes.indexOf(this.tasks[index].done);
    
        if (this.tasks[index].done != 'Task is done') {
         newIndex++;
        this.tasks[index].done = this.availableStatusTypes[newIndex];
        this.saveTasks();
        console.log(newIndex);
         if (this.tasks[index].done === 'Task is done' && !this.tasks[index].isDone){
          this.tasks[index].isDone = true;
          this.numberOfPending--;
          this.numberOfDone++;
            this.saveTasks();
         }
        }  
      },

      saveTasks(){
        const parsed = JSON.stringify(this.tasks);
        localStorage.setItem('tasks', parsed);
        console.log(localStorage);
      },

      resetTasks(){
        if (!this.isClicked) {
         localStorage.removeItem('tasks');
          localStorage.numberOfDone = 0;
          localStorage.numberOfPending = 0;
          this.resetMessage = 'App will restart is 2 seconds';
          this.isClicked = !this.isClicked;
          setTimeout(function(){
          window.location.reload();
          }, 2000)
      }
      },
      isCompleted() {
        this.showDone = true;
        console.log("clicked done");
      },
      isPending() {
        this.showDone = false;
        console.log("clicked pending");
      }
  }
};
</script>

<style>

  
</style>
