<template>
  <div class="page-container">
    <md-app md-waterfall md-mode="overlap">


      <md-app-toolbar class="md-primary">
        <span class="md-title">Trabalho 3 - Gustavo Westarb</span>
      </md-app-toolbar>

      <md-app-content>
        <div class="md-layout">
                
          <md-card class="md-layout-item md-size-25 md-small-size-100" style="max-height: 344px">
            <form novalidate class="md-layout" @submit.prevent="callAjax">
              <md-card-header>
                <div class="md-title">Call Rest</div>
              </md-card-header>

            <md-card-content>
              <div class="md-layout md-gutter">
                <div class="md-layout-item md-small-size-100" >
                  <md-radio v-model="radio" value="get" v-on:change="onChangeRadio">Get</md-radio>
                  <md-radio v-model="radio" value="post" v-on:change="onChangeRadio">Post</md-radio>
                  <md-radio v-model="radio" value="put" v-on:change="onChangeRadio">Put</md-radio>
                  <md-radio v-model="radio" value="delete" v-on:change="onChangeRadio">Delete</md-radio>
                </div>
              </div>

              <div class="md-layout md-gutter">
                <div class="md-layout-item md-small-size-100">
                  <md-field>
                    <label for="identifier">Indentificador</label>
                    <md-input type="number" id="identifier" name="identifier" v-model="form.id" :disabled="disableId"/>
                  </md-field>
                </div>

                <div class="md-layout-item md-small-size-100">
                  <md-field>
                    <label for="name">Nome</label>
                    <md-input type="text" id="name" name="name" v-model="form.name" :disabled="disableName"/>
                  </md-field>
                </div>
              </div>

              <div class="md-layout md-gutter">
                <div class="md-layout-item md-small-size-100">
                  <md-field>
                    <label for="age">Idade</label>
                    <md-input type="number" id="age" name="age" autocomplete="age" v-model="form.age" :disabled="disableAge"/>
                  </md-field>
                </div>

                <div class="md-layout-item md-small-size-100">
                  <md-field>
                    <label for="salary">Salário</label>
                    <md-input type="number" id="salary" name="salary" v-model="form.salary" :disabled="disableSalary"/>
                  </md-field>
                </div>
              </div>
            </md-card-content>

            <md-progress-bar md-mode="indeterminate" />

            <md-card-actions>
              <md-button type="submit" class="md-primary" :disabled="sending">Send Request</md-button>
            </md-card-actions>
          </form>
          
          </md-card>

          <md-table v-model="employeesData" md-card class="md-layout-item md-size-70 md-small-size-100" ref="table">
            <md-table-toolbar>
              <h1 class="md-title">Employees</h1>
            </md-table-toolbar>

            <md-table-row slot="md-table-row" slot-scope="{ item }">
              <md-table-cell md-label="ID" md-sort-by="id" md-numeric>{{ item.id }}</md-table-cell>
              <md-table-cell md-label="Nome" md-sort-by="name">{{ item.name }}</md-table-cell>
              <md-table-cell md-label="Salário" md-sort-by="salary">{{ item.salary }}</md-table-cell>
              <md-table-cell md-label="Idade" md-sort-by="age">{{ item.age }}</md-table-cell>
            </md-table-row>
          </md-table>
        </div>

      <md-dialog-confirm
            :md-active.sync="active" 
            md-title="Deletar o registro" 
            md-content="Deseja realmente deletar o registro selecionado?"
            md-confirm-text="Sim"
            md-cancel-text="Não"
            @md-cancel="onCancel"
            @md-confirm="onConfirm" />
    </md-app-content>

  </md-app>
  </div>
</template>

<script>

let urlRest = "http://dummy.restapiexample.com/api/v1"

import { validationMixin } from 'vuelidate'
import axios from "axios";
import {
  required,
  email,
  minLength,
  maxLength
} from 'vuelidate/lib/validators'

export default {
  name: 'FormValidation',
  mixins: [validationMixin],
  data: () => ({
    radio: "get",
    active: false,
    form: {
      name: null,
      salary: null,
      age: null,
      id: null
    },
    employeesData: [],
    userSaved: false,
    sending: false,
    disableId: false,
    disableName: true,
    disableSalary: true,
    disableAge: true,
    lastUser: null
  }),
  methods: {
    onCancel () {
      alert("Cancelado")
    },
    onConfirm (){
      this.deleteEmploye(this)
    },
    callAjax () {
      this.sending = true

      if(this.radio == "get"){
        this.form.age = null
        this.form.salary = null
        this.getEmploye(this)
      }else if(this.radio == "post"){
        this.postEmployee(this)
      }else if(this.radio == "put"){
        this.putEmployee(this)
      }
      else{
        this.active = true
      }
    },
    getEmploye(args){

      var urlGet;
      var withId = args.form.id !== null

      if(withId){
        urlGet = "http://dummy.restapiexample.com/api/v1/employee/" + args.form.id
      }
      else{
        urlGet = "http://dummy.restapiexample.com/api/v1/employees"
      }

      axios.get(urlGet)
      .then(response => {

        if(withId){
          args.employeesData = [];
          args.employeesData.push({
            id: response.data.id,
            name: response.data.employee_name,
            salary: response.data.employee_salary,
            age: response.data.employee_age
          })
        }else{
          args.employeesData = [];
          for (let i = 0; i < 10; i++) {
            args.employeesData.push({
              id: response.data[i].id,
              name: response.data[i].employee_name,
              salary: response.data[i].employee_salary,
              age: response.data[i].employee_age
            })
          }
        }
      })
      .catch(error => {
        console.log(error)
      })

      args.sending = false
    },
    postEmployee(args){

      axios.post("http://dummy.restapiexample.com/api/v1/create",
        '{"name":"'+ args.form.name +'", "salary":"'+ args.form.salary +'","age":"'+ args.form.age +'"}'
      ).then(response => {
        args.employeesData.push({
          id: response.data.id,
          name: response.data.name,
          salary: response.data.salary,
          age: response.data.age
        })
      }).catch(error => {
        console.log(error)
      })

      args.sending = false
    },
    putEmployee(args){
      let urlPut = "http://dummy.restapiexample.com/api/v1/update/" + args.form.id 

      let positionEmployee = args.employeesData.findIndex(x => x.id == args.form.id)
      let name = args.form.name === null ? args.employeesData[positionEmployee].name : args.form.name
      let age = args.form.age === null ? args.employeesData[positionEmployee].age : args.form.age
      let salary = args.form.salary === null ? args.employeesData[positionEmployee].salary : args.form.salary

      axios.put(urlPut,
        '{"name":"'+ name +'", "salary":"'+ age +'","age":"'+ salary +'"}'
      ).then(response => {
        args.employeesData[positionEmployee] = JSON.parse('{"id":"' + args.form.id + '","name":"'+ response.data.name + '","salary":"'+ response.data.salary+ '","age":"' + response.data.age+'"}')
        args.radio = "get"
        args.radio = "put"
      }).catch(error => {
        console.log(error)
      })

      args.sending = false
    },
    deleteEmploye(args){
      let urlDelete = "http://dummy.restapiexample.com/api/v1/delete/"
      let idDelete = 0;

      if(args.form.name !== null){
        let positionEmployee = args.employeesData.findIndex(x => x.name == args.form.name)
        idDelete = args.employeesData[positionEmployee].id
      }else{
        idDelete = args.form.id
      }

      urlDelete = urlDelete + idDelete
      axios.delete(urlDelete)
        .then(response => {
          let positionEmployee = args.employeesData.findIndex(x => x.id == idDelete)
          args.employeesData.splice(positionEmployee, 1)
        })
        .catch(error => {
          console.log(error)
        })

      args.sending = false
    },
    onChangeRadio(){
      if (this.radio == "put" || this.radio == "post") {
        this.disableId = this.radio == "post"
        this.disableName = false
        this.disableSalary = false
        this.disableAge = false
      }else if( this.radio == "delete"){
        this.disableId = false;
        this.disableName = false
        this.disableSalary = true
        this.disableAge = true
      }
      else{
        this.disableId = false
        this.disableName = true
        this.disableSalary = true
        this.disableAge = true
      }

      this.form.name = null;
      this.form.id = null;
      this.form.age = null;
      this.form.salary = null;
    }
  }
}
</script>

<style>

  .md-app {
    height: 100%;
    border: 1px solid rgba(#000, .12);
  }
</style>
