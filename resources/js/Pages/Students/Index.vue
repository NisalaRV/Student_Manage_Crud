<template>
    <AppLayouts>
      <template #content>
        <br>
        <h1 h1 class="main-con">Student Details</h1>
        <br>

        <!-- Button to trigger the modal -->
        <button type="button" class="btn btn-primary mb-4 " data-bs-toggle="modal" data-bs-target="#addModal">
          Add
        </button>
        <!-- Edit Modal for adding student -->

        <!-- Modal for adding student -->
        <div class="modal fade" id="addModal" tabindex="-1" aria-labelledby="addModalLabel" aria-hidden="true">
          <div class="modal-dialog">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="addModalLabel"> Add </h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
              </div>
              <div class="modal-body">
                <form @submit.prevent="addStudent">
                  <div class="form-group">
                    <label for="name">Name</label>
                    <input type="text" class="form-control" v-model="newStudent.name" required>
                  </div>
                  <div class="form-group">
                    <label for="age">Age</label>
                    <input type="number" class="form-control" v-model="newStudent.age" required>
                  </div>
                  <div class="form-group">
                    <label for="image">Image </label>
                   <input type="file" class="form-control" id="image"   @change="handleImageChange" >
                  </div>
                <br>
                  <button type="submit" class="btn btn-primary">Save </button>
                </form>
              </div>
            </div>
          </div>
        </div>

        <!-- Table to display student information -->

        <table class="table table-hover">
          <!-- Table header -->
          <thead>
            <tr>
              <th scope="col">Id</th>
              <th scope="col">Name</th>
              <th scope="col">Age</th>
              <th scope="col">Image</th>
              <th scope="col">Status</th>
              <th scope="col">Action</th>
            </tr>
          </thead>
          <tbody>
            <!-- Table rows using v-for loop -->
            <tr v-for="item in student_list " :key="item.id">
              <th scope="row">{{ item.id }}</th>
              <td>{{ item.name }}</td>
              <td>{{ item.age }}</td>
              <td><img :src="image" alt="Student Image" class="img-thumbnail"></td>
              <td><span v-if="item.status==0" class="btn btn-danger">inactive </span>
                  <span v-else="item.status==1" class="btn btn-success">active </span>
              </td>

              <td><button class="btn btn-primary"   @click.prevent="editeStudent(item.id)" >Edit</button>
              <button @click.prevent="deleteStudent(item.id)" class="btn btn-danger">Delete</button>
              <button v-if="item.status==0"  @click.prevent="doneStudent(item.id)" class="btn btn-success">Active</button>
              <button v-if="item.status==1"  @click.prevent="doneStudent(item.id)" class="btn btn-warning">Inctive</button>
              </td>
           </tr>
          </tbody>
        </table>
      </template>




   // ------------------------------- Edit Modal for adding student------------------------------------->

        <div class="modal fade" id="editModal" tabindex="-1" aria-labelledby="addModalLabel" aria-hidden="true">
          <div class="modal-dialog">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="addModalLabel">Add Student</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
              </div>
              <div class="modal-body">
                <form @submit.prevent="updateStudent">
                  <div class="form-group">
                    <label for="name">Name</label>
                    <input type="text" class="form-control" v-model="updateStudent.name" required>
                  </div>
                  <div class="form-group">
                    <label for="age">Age</label>
                    <input type="number" class="form-control" v-model="updateStudent.age" required>
                  </div>
                  <div class="form-group">
                    <label for="image">Image </label>
                    <input type="text" class="form-control" v-model="updateStudent.image" required>
                  </div>
                <br>
                  <button type="submit" class="btn btn-primary">Add Student</button>
                </form>
              </div>
            </div>
          </div>
        </div>


  </AppLayouts>
  </template>



  <script>

  import AppLayouts from '@/Layouts/App.vue';
  import axios from 'axios';
  import { Modal } from 'bootstrap';


  export default {
    components: {
      AppLayouts
    },


    data() {
      return {
        newStudent: {
          name: '',
          age: '',
          image: '',
        },
        updateStudent: {
          name: '',
          age: '',
          image: '',
          status:'',
        },
        student_list: [] //Initializing an array  of student data
      };
    },
  beforeMount() {
    this.getStudent();
  },

    methods: {

      async getStudent() {
      const response = await axios.get(route('student.list')); // fetch student data.
      this.student_list = response.data;

  },



      async addStudent() {

        const { image, ...studentData } = this.newStudent;
        await axios.post(route('student.save'), studentData);

        this.getStudent();

           this.newStudent = {
          name: '',
          age: '',
          image: '',
        };

        },



    async deleteStudent(id) {

      await axios.delete(route('student.delete', id));
      this.getStudent();
      },



   async doneStudent(id) {
        await axios.get(route('student.done', id));
        this.getStudent();

  },



  async editeStudent(id) {

          const data = (await axios.get(route('student.show',id))).data
          this.updateStudent = data
          $('#editModal').modal('show')

  },



  async updateStudent() {

          await axios.post(route('student.update',this.updateStudent,id),this.updateStudent)
          this.getStudent()
          $('#editModal').modal('hide')

  },


   handleImageChange(event) {
      const file = event.target.files[0];
        this.newStudent.image = URL.createObjectURL(file); // Display the uploaded image locally
        this.uploadImage(file, 'newStudent');
    },



   uploadImage(file, studentType) {
    const formData = new FormData();
    formData.append('image', file);

    const uploadUrl = studentType === 'newStudent' ? '/students/upload-image' : `/students/${this.updateStudent.id}/upload-image`;

    axios.post(uploadUrl, formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
      .then(response => {
        if (studentType === 'newStudent') {
          this.newStudent.image = response.data.url;
        } else {
          this.updateStudent.image = response.data.url;
        }
      })
      .catch(error => {
        console.log(error);
      });
  }


  }

    }

  </script>

  <style scoped>


   .main-con{
    color: #4caf89;
    margin-left: 650px;
  }
.table-container {
  width: 100%;
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
  background-color: #343a40;
  color: #fff;

}

.table, th {
  border: 1px solid #454d55;
}

th, td {
  padding: 12px;
  text-align: left;
}

th {
  background-color: #495057;
}


.img-thumbnail {
  max-width: 50px;
  max-height: 50px;
}

.btn-danger, .btn-success, .btn-warning {
  padding: 6px 12px;
}

.table td button {
  margin-right: 5px;
}

  </style>
