<template>
  <div class="border container mt-4 bg-secondary">
    <!-- Modal -->
    <div class="modal fade" id="addTaskModal" tabindex="-1" aria-labelledby="addTaskModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content bg-dark">
          <div class="modal-header bg-secondary">
            <h5 class="modal-title text-light" id="addTaskModalLabel">Add Task</h5>
            <button id="modalCloseButton" type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <div class="form-floating mb-3">
              <input type="text" class="form-control" id="floatingTaskName" v-model="task.data.attributes.taskName" placeholder="Task Name">
              <label for="floatingTaskName">Task Name</label>
            </div>
            <div class="form-floating">
              <input type="text" class="form-control" id="floatingTaskDescription" v-model="task.data.attributes.taskDescription" placeholder="Task Description">
              <label for="floatingTaskDescription">Task Description</label>
            </div>
          </div>
          <div class="modal-footer bg-secondary">
            <button type="button" class="btn btn-outline-light" data-bs-dismiss="modal">Close</button>
            <button type="button" class="btn btn-dark" @click="createTask">Save changes</button>
          </div>
        </div>
      </div>
    </div>
    <h1 class="text-light">Tasks</h1>
    <div v-if="tasks.length === 0" class="text-light">Click Add Task to start adding tasks</div>
    <div v-for="(task, index) in tasks" :key="task.attributes.taskName+index" class="row">
        <div class="col-8 border bg-dark p-2 m-2">
            <div class="form-check form-check-inline">
              <input class="form-check-input" type="checkbox" :id="'complete-id-'+index" @click="updateTaskCompletion(index)" v-model="tasks[index].attributes.complete" value="completed">
              <label class="form-check-label text-light h5" :for="'complete-id-'+index">{{task.attributes.taskName}}</label>
            </div>
            <div class="row ps-4 text-light">{{task.attributes.taskDescription}}</div>
        </div>
    </div>
    <div class="row mb-2 mt-2">
      <button type="button" class="btn btn-dark col-auto ms-2 me-4" data-bs-toggle="modal" data-bs-target="#addTaskModal">Add Task</button>
      <button type="button" class="btn btn-dark col-auto" @click="deleteAllTasks">Clear All</button>
    </div>
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  mounted() {
    this.$nextTick(() => {
      this.fetchTasks();
    });
  },
  data() {
    return {
      task: {
        data: {
          type: "task",
          attributes: {
            taskName: "",
            taskDescription: "",
            category: "",
            complete: false,
            status: ""
          }
        }
      },
      tasks: [
          {
            id: "31dddb5e155b4fd58cacf7a298417ed7",
            type: "task",
            links: {
                self: "http://localhost:8080/tasks/31dddb5e155b4fd58cacf7a298417ed7"
            },
            attributes: {
                taskName: "MyTask",
                taskDescription: "My task desc",
                category: "MyCategory",
                complete: false,
                status: "To Do"
            }
          },
          {
            id: "31dddb5e155b4fd58cacf7a298417ed7",
            type: "task",
            links: {
                self: "http://localhost:8080/tasks/31dddb5e155b4fd58cacf7a298417ed7"
            },
            attributes: {
                taskName: "Other Task",
                taskDescription: "BBBBBBBBBBBBBB",
                category: "MyCategoryB",
                complete: false,
                status: "Doing"
            }
          },
      ]
    }
  },
  methods: {
    fetchTasks(){
        const baseURI = 'https://axios-tasks.herokuapp.com/tasks';
        this.$http.get(baseURI)
        .then((result) => {
          this.tasks = result.data.data;
        });
    },
    patchTask(task){
      const baseURI = 'https://axios-tasks.herokuapp.com/tasks/'+task.data.id;
      const options = {
        headers: {
          'Content-Type':'application/vnd.api+json'
        }
      };
      this.$http.patch(baseURI, task, options)
      .then((result) => {
        console.log(result.status);
        this.fetchTasks();
      });
    },
    deleteTask(task){
      const baseURI = 'https://axios-tasks.herokuapp.com/tasks/'+task.id;
      this.$http.delete(baseURI)
      .then((result) => {
        console.log(result.status);
        this.fetchTasks();
      });
    },
    deleteAllTasks(){
      const baseURI = 'https://axios-tasks.herokuapp.com/deleteAll';
      this.$http.delete(baseURI)
      .then((result) => {
        console.log(result.status);
        this.fetchTasks();
      });  
    },
    resetTaskModel(){
      this.task = {
        data: {
          type: "task",
          attributes: {
            taskName: "",
            taskDescription: "",
            category: "",
            complete: false,
            status: ""
          }
        }
      };
    },
    postTask(task){
      console.log(task);
      const baseURI = 'https://axios-tasks.herokuapp.com/tasks/';
      const options = {
        headers: {
          'Content-Type':'application/vnd.api+json'
        }
      };
      this.$http.post(baseURI, task, options)
      .then((result) => {
        console.log(result);
        //after successful call refetch data from db
        this.fetchTasks();
      }, (error) => {
        console.log(error);
      });
    },
    createTask(){
      // clear validations
      if(document.getElementById("floatingTaskName").classList.contains("is-invalid")){
        document.getElementById("floatingTaskName").classList.remove("is-invalid");
      }
      if(document.getElementById("floatingTaskDescription").classList.contains("is-invalid")){
        document.getElementById("floatingTaskDescription").classList.remove("is-invalid");
      }

      if(this.task.data.attributes.taskName !== "" && this.task.data.attributes.taskDescription !== ""){
        //send task to api
        this.postTask(this.task);
        //reset model
        this.resetTaskModel();
        //hide modal
        document.getElementById("modalCloseButton").click();
      } else {
        //if invalid set invalid
        if(this.task.data.attributes.taskName === ""){
          document.getElementById("floatingTaskName").classList.add("is-invalid");
        }
        if(this.task.data.attributes.taskDescription === ""){
          document.getElementById("floatingTaskDescription").classList.add("is-invalid");
        }
      }
    },
    updateTaskCompletion(index){
      this.resetTaskModel();
      this.task.data = this.tasks[index];
      this.task.data.attributes.complete = document.querySelector('#complete-id-'+index).checked;
      this.patchTask(this.task);
      this.resetTaskModel();
    },
  },
}
</script>

<style scoped>

</style>
