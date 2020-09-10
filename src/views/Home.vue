<template>
  <div class = "container" >
      <div class = "section" id = "todolist-title">
        <div class = "b-navbar">
          <h3 class="page-heading">My Todo List</h3>
          <b-navbar toggleable="lg" type="dark">
            <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
            <b-collapse id="nav-collapse" is-nav>
              <b-navbar-nav>
                <button type="button" class="btn btn-primary col-md-12" id="btn-addTodo" @click="showTodoModal=true">Add New ToDo</button>
              </b-navbar-nav>
                <div class="search-area">
                  <label for="Search" class="mr-2 mt-2">Search</label>
                  <input class="form-control  mr-2" type="number" v-model="search" id="search" name="search" placeholder="Type the ID here">
                  <button type="button" class="btn btn-primary" @click="getTodoById()">Go</button>
                </div>
            </b-collapse>
          </b-navbar>
        </div>
      </div>

      <div class="section">
        <ul class = "list-heading">
          <li class = "list-items-heading">ID</li>
          <li class = "list-items-heading">Title</li>
          <li class = "list-items-heading">Date</li>
          <li class = "list-items-heading">Completed</li>
          <li class = "list-items-heading">Action</li>
        </ul>
        <div class = "section">
          <ul v-for="(task,index) in tasks" v-bind:key="task.id" class="list-details">
            <li class = "list-items-data">{{task.id}}</li>
            <li class = "list-items-data">{{task.title}}</li>
            <li class = "list-items-data">{{task.date}}</li>
            <li class = "list-items-data" v-if="task.completed==true">Yes</li>
            <li class = "list-items-data" v-else>No</li>
            <li class = "list-items-data">
              <button type ="button" class="btn btn-warning btn-action mr-2" @click="loadTodo(task.id)">Edit</button>
              <button type ="button" class="btn btn-danger btn-action" @click="confirmDelete(task.id, index)">Delete</button>
            </li>
          </ul>
        </div>
      </div>

      <!-- Add Todo Modal -->
        <div v-if="showTodoModal">
          <transition name="modal">
            <div class="modal-mask">
              <div class="modal-wrapper">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                      <div class="modal-header">
                          <h5 class="modal-title">Add a New Todo</h5>
                          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                          <span aria-hidden="true" @click="showTodoModal=false">&times;</span>
                          </button>
                      </div>
                      <p id = "required-message">{{message}}</p>
                      <div class="modal-body">
                        <div class="form-group">
                          <label for="title" class= "field-label">Title <span class = "required-fields">*</span></label>
                          <input type = "text" class="form-control" id = "title" v-model="title" name="title" required />
                          <p class = "err_message">{{err_message[0]}}</p>
                        </div>
                        <div class = "form-group">
                          <label for="date" class= "field-label">Date<span class = "required-fields">*</span></label>
                          <input type = "date" class="form-control" id = "date" v-model="date" name="date" @load="getDateToday()" required/>
                          <p class = "err_message">{{err_message[1]}}</p>
                        </div>
                        <div class = "form-group">
                           <label for="completed" class= "field-label">Completed<span class = "required-fields">*</span></label>
                              <select class="form-control" id="completed" v-model="completed" required>
                                  <option disabled value="">Select One</option>
                                  <option>Yes</option>
                                  <option>No</option>
                              </select>
                          <p class = "err_message">{{err_message[2]}}</p>
                        </div>
                      </div>
                      <div class="modal-footer">
                        <div class form-group>
                          <button type="button" class="btn btn-primary mr-3" @click="addNewTodo()">Save</button>
                          <button type="button" class="btn btn-secondary" @click="showTodoModal=false">Close</button>
                        </div>
                      </div>
                    </div>
                </div>
              </div>
            </div>
          </transition>
        </div>
        <!--End of Add Todo Modal-->

        <!-- Delete Todo -->
        <div v-if="showDeleteTodoModal">
          <transition name="modal">
            <div class="modal-mask">
              <div class="modal-wrapper">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                      <div class="modal-header">
                          <h5 class="modal-title">Delete Todo</h5>
                          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true" @click="showDeleteTodoModal=false">&times;</span>
                          </button>
                      </div>
                      <div class="modal-body">
                        <div class = "form-group">
                          <p>Do you really want to delete this todo?</p>
                        </div>
                      </div>
                      <div class="modal-footer">
                        <div class form-group>
                          <button type="button" class="btn btn-danger mr-3" @click="deleteTodo(task_id, task_index)">Yes</button>
                          <button type="button" class="btn btn-secondary" @click="showDeleteTodoModal=false">No</button>
                        </div>
                      </div>
                    </div>
                </div>
              </div>
            </div>
          </transition>
        </div>
        <!--End of Delete Todo-->

        <!-- Edit Todo Modal -->
        <div v-if="showEditTodoModal">
          <transition name="modal">
            <div class="modal-mask">
              <div class="modal-wrapper">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                      <div class="modal-header">
                          <h5 class="modal-title">Edit Todo</h5>
                          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true" @click="showEditTodoModal=false">&times;</span>
                          </button>
                      </div>
                      <div class="modal-body">
                        <div class = "form-group">
                          <label for="id" class= "field-label">ID</label>
                          <input type = "text" class="form-control" id = "edit-id" v-model="task_to_edit[0].id" name="edit-id" />
                          <p class = "err_message"></p>
                        </div>
                        <div class = "form-group">
                          <label for="title" class= "field-label">Title</label>
                          <input type = "text" class="form-control" id = "edit-title" v-model="task_to_edit[0].title" name="edit-title" />
                          <p class = "err_message"></p>
                        </div>
                        <div class = "form-group">
                          <label for="date" class= "field-label">Date</label>
                          <p style="display:none">{{todo_date=formatDate(task_to_edit[0].date)}}</p>
                          <input type = "date" class="form-control" id = "edit-date" v-model="todo_date" name="edit-date"/>
                          <p class = "err_message"></p>
                        </div>
                        <div class = "form-group">
                           <label for="completed" class= "field-label">Completed</label>
                              <p v-if="task_to_edit[0].completed==true" style="display: none">{{todo_comp="Yes"}}</p>
                              <p v-else style="display: none">{{todo_comp="No"}}</p>
                              <select class="form-control" id="edit-completed" v-model="todo_comp">
                                  <option disabled value="">Select One</option>
                                  <option>Yes</option>
                                  <option>No</option>
                              </select>
                          <p class = "err_message"></p>
                        </div>
                      </div>
                      <div class="modal-footer">
                        <div class form-group>
                          <button type="button" class="btn btn-primary mr-3" @click="updateTodo()">Save</button>
                          <button type="button" class="btn btn-secondary" @click="showEditTodoModal=false">Close</button>
                        </div>
                      </div>
                    </div>
                </div>
              </div>
            </div>
          </transition>
        </div>
        <!--End of Edit Todo Modal-->

        <div v-if="showUpdateTodoSuccessModal">
          <transition name="modal">
            <div class="modal-mask">
              <div class="modal-wrapper">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                      <div class="modal-header">
                          <h5 class="modal-title">Update Todo</h5>
                          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true" @click="showUpdateTodoSuccessModal=false">&times;</span>
                          </button>
                      </div>
                      <div class="modal-body">
                        <div class = "form-group">
                          <p>Todo details successfully updated.</p>
                        </div>
                      </div>
                      <div class="modal-footer">
                        <div class form-group>
                          <button type="button" class="btn btn-secondary" @click="showUpdateTodoSuccessModal=false">OK</button>
                        </div>
                      </div>
                    </div>
                </div>
              </div>
            </div>
          </transition>
        </div>
        <!--End of Delete Todo-->

  </div>
</template>

<script>
// @ is an alias to /src
import axios from 'axios'
import moment from 'moment'
export default {
  name: 'Home',
  components: {
  },
  data () {
    return {
      tasks: [],
      errors: [],
      task_to_edit: [],
      baseURI: 'https://my-json-server.typicode.com/wsh-startup/mock-api',
      showTodoModal: false,
      showDeleteTodoModal: false,
      showEditTodoModal: false,
      showUpdateTodoSuccessModal: false,
      title: '',
      date: '',
      completed: '',
      isCompleted: false,
      search: 0,
      task_id: null,
      task_index: null,
      edit_title: '',
      edit_date: '',
      edit_completed: '',
      err_message: {},
      message: '',
      todo_date: null,
      todo_comp: ''
    }
  },
  methods: {
    formatDate (date) {
      return moment(date, 'MM/DD/YYYY').format('DD/MM/YYYY')
    },
    validateInput (a, b) {
      if (a === '' && b === 0) {
        this.err_message[b] = 'Please enter the title.'
      } else if (a === '' && b === 1) {
        this.err_message[b] = 'Please enter the date.'
      } else if (a === '' && b === 2) {
        this.err_message[b] = 'Please indicate if it is completed.'
      }
    },
    getAllTodo () {
      axios.get(this.baseURI + '/tasks')
        .then((result) => {
          this.tasks = result.data
        })
    },
    getTodoById () {
      const id = this.search
      if (id === 0 || id === '') {
        this.getAllTodo()
      } else {
        axios.get(this.baseURI + '/tasks/', { params: { id: id } })
          .then((result) => {
            this.tasks = result.data
          })
      }
    },
    addNewTodo () {
      if (this.title === '' ||
        this.date === '' ||
        this.completed === '') {
        this.message = 'Please provide input on the required fields (*).'
      } else {
        const axiosConfig = {
          headers: {
            'Content-Type': 'application/json;charset=UTF-8',
            'Access-Control-Allow-Origin': '*'
          }
        }
        if (this.completed === 'Yes') {
          this.isCompleted = true
        }
        const params = {
          title: this.title,
          date: this.date,
          completed: this.isCompleted
        }
        axios.post(this.baseURI + '/tasks', params, axiosConfig)
          .then(response => {
            alert('Todo successfully added...')
            this.showTodoModal = false
            this.getAllTodo()
            console.log(response)
          })
          .catch(e => {
            this.errors.push(e)
          })
      }
    },
    loadTodo (taskId) {
      const id = taskId
      axios.get(this.baseURI + '/tasks/', { params: { id: id } })
        .then((result) => {
          this.task_to_edit = result.data
          console.log(result.data)
        })
      this.showEditTodoModal = true
    },
    updateTodo (id) {
      const axiosConfig = {
        headers: {
          'Content-Type': 'application/json;charset=UTF-8',
          'Access-Control-Allow-Origin': '*'
        }
      }
      const params = {
        id: document.getElementById('edit-id').value,
        title: document.getElementById('edit-title').value,
        date: document.getElementById('edit-date').value,
        completed: document.getElementById('edit-completed')
      }
      axios.put(this.baseURI + '/tasks/', params, axiosConfig)
        .then((result) => {
          this.task_to_edit = result.data
          console.log(result.data)
        })
      this.showEditTodoModal = false
      this.showUpdateTodoSuccessModal = true
    },
    confirmDelete (id, index) {
      this.showDeleteTodoModal = true
      this.task_id = id
      this.task_index = index
    },
    deleteTodo (id, index) {
      axios.delete(this.baseURI + '/tasks/', { params: { id: id } })
        .then(response => {
          alert('Todo successfully deleted' + ' ' + id)
        })
      this.showDeleteTodoModal = false
      this.tasks.splice(index, 1)
    },
    getDateToday () {
      let today = new Date()
      let dd = today.getDate()
      let mm = today.getMonth() + 1 // January is 0!
      const yyyy = today.getFullYear()

      if (dd < 10) {
        dd = '0' + dd
      }
      if (mm < 10) {
        mm = '0' + mm
      }
      today = dd + '/' + mm + '/' + yyyy
      document.getElementById('date').value = today
    }
  },
  mounted () {
    this.getAllTodo()
  }
}
</script>

<style>
.container{
  border: 1px solid gray;
  border-radius: 15px;
  padding: 10px;
  box-shadow: 0px 3px 5px 3px gray;
}
.list-heading, .list-details{
  padding: 10px 0px 0px 10px;
  list-style-type:none;
  display: flex;
}
.list-items-heading, .list-items-data{
  width: 400px;
  display: table-cell;
  padding-left: 20px;
  border-bottom: 1px solid gray;
  white-space: normal;
  height: 50px;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}
.list-heading .list-items-heading{
  font-weight: bold;
}
#todolist-title{
  border-bottom: 1px solid black;
}
#btn-addTodo{
  font-size: 12pt;
}
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5);
  display: table;
  transition: opacity .3s ease;
  z-index: 1;
}
.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}
.modal-title, .modal-body {
  color:#042331;
}
.search-area{
  display: flex;
  position: absolute;
  right: 0;
}
.btn-action{
  height: 35px;
  width: 70px;
}
.page-heading{
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
.field-label{
  display: flex;
  color: #042331;
  font-weight: 600;
}
.required-fields{
  color: red;
}
 #required-message{
   color: red;
   font-style: italic;
 }
</style>
