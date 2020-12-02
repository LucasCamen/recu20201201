<template>

  <section class="src-components-formulario">

    <div class="jumbotron">
      <h2>Ingreso de Tareas</h2>
      <hr />
      <br>

      <form novalidate autocomplete="off" @submit.prevent="enviar()">
        <div class="form-group">
          <label for="descripcion">Descripción</label>
          <input
            type="text"
            id="descripcion"
            class="form-control"
            v-model="v.f.descripcion.$model"
          />
          <div
            v-if="v.f.descripcion.$error && v.f.descripcion.$dirty"
            class="alert alert-danger mt-1"
          >
            <div v-if="v.f.descripcion.required.$invalid">
              Este campo es requerido
            </div>
            <div v-if="v.f.descripcion.minLength.$invalid">
              Este campo debe tener al menos
              {{ v.f.descripcion.minLength.$params.min }} caracteres
            </div>
            <div v-if="v.f.descripcion.maxLength.$invalid">
              Este campo debe tener máximo
              {{ v.f.descripcion.maxLength.$params.max }} caracteres
            </div>
          </div>
        </div>

        <div class="form-group">
          <label for="nombre">Nombre</label>
          <input
            type="text"
            id="nombre"
            class="form-control"
            v-model="v.f.nombre.$model"
          />
          <div
            v-if="v.f.nombre.$error && v.f.nombre.$dirty"
            class="alert alert-danger mt-1"
          >
            <div v-if="v.f.nombre.required.$invalid">
              Este campo es requerido
            </div>
          </div>
        </div>
        <div class="form-group">
          <label for="email">Email</label>
          <input
            type="email"
            id="email"
            class="form-control"
            v-model="$v.f.email.$model"
          />

          <div
            v-if="$v.f.email.$error && $v.f.email.$dirty"
            class="alert alert-danger mt-1"
          >
            <div v-if="$v.f.email.required.$invalid">
              Este campo es requerido
            </div>
            <div v-if="$v.f.email.email.$invalid">
              Debe proveer un email válido
            </div>
          </div>
        </div>

        <div class="form-group">
          <input
            type="submit"
            :disabled="v.$invalid"
            class="btn btn-success mt-4"
            value="Enviar"
          />
        </div>
      </form>
    </div>

<h3 v-if="!tareas.length && !pidiendo" class="alert alert-warning">
      No se encontraron tareas
    </h3>
    <div v-if="tareas.length" class="alert alert-info">
      <h2>Detalle de Tareas</h2>
      <hr />
      <br>

      <div class="table-responsive">
        <table class="table table-dark">
          <tr :style="{color:'cornflowerblue'}">
            <th>DESCRIPCIÓN</th>
            <th>NOMBRE</th>
            <th>EMAIL</th>
            <th>FECHA</th>
          </tr>
          <tr v-for="(tarea,key) in tareas" :key="key">
            <td>{{tarea.descripcion}}</td>
            <td>{{tarea.nombre}}</td>
            <td>{{tarea.email}}</td>
            <td>{{fyhLocal(tarea.createdAt)}}</td>
          </tr>
        </table>
      </div>        
    </div>
<h2>Respuestas</h2>
      <hr />
      <br>
      <div class="table-responsive">
        <table class="table table-dark">
          <tr :style="{color:'cornflowerblue'}">
            <th>PREGUNTA</th>
            <th>RESPUESTA</th>
          </tr>
          <tr v-for="(pregunta,key) in preguntas" :key="key">
            <td>{{pregunta.pregunta}}</td>
            <td>{{pregunta.respuestas}}</td>
          </tr>
        </table>
      </div>
  </section>
</template>

<script>
import { required, minLength, maxLength } from "@vuelidate/validators";
import { useVuelidate } from "@vuelidate/core";
import filters from '../filters.js'

export default {
  name: "src-components-formulario",
  props: [],
  mixins : [filters],
  mounted () {
        this.getDatosFormAxios()
    },
  created() {
    const rules = {
      f: {
        nombre: {
          required,
        },
        descripcion: {
          required,
          minLength: minLength(10),
          maxLength: maxLength(50),
        },
        email: {
          required,
      },
      },
    };
    const f = this.f;
    this.v = useVuelidate(rules, { f });
  },
  data() {
    return {
      url: "https://5fc6ca88f3c77600165d790d.mockapi.io/tareas",
      f: {
        nombre: "",
        descripcion: "",
        email:"",
        pidiendo: true,
      },
      v: null,
      preguntas : [
            { pregunta: 1, respuestas: ['c'] },
            { pregunta: 2, respuestas: ['c'] },
            { pregunta: 3, respuestas: ['a'] },
            { pregunta: 4, respuestas: ['d'] },
            { pregunta: 5, respuestas: ['e'] },
          ]
    };
  },
  methods: {
    async sendDatosFormAxios(datos) {
      try {
        let res = await this.axios.post(this.url, datos, {
          "content-type": "application/json",
        });
        console.log(res.data);
      } catch (error) {
        console.log("HTTP POST ERROR", error);
      }
    },
    async enviar() {
      this.v.$touch();
      if (!this.v.$invalid) {
        let form = this.f;
        await this.sendDatosFormAxios(form);
        this.resetForm();
        this.v.$reset();
      }
    },
    resetForm() {
      this.v.f.nombre.$model = "";
      this.v.f.descripcion.$model = "";
      this.v.f.email.$model = "";
    },
    async getDatosFormAxios() {
            try {
              let res = await this.axios(this.url)
              console.log(res.data)
              this.tareas = res.data
            }
            catch(error) {
              console.log('HTTP GET ERROR', error)
            }
            finally {
              this.pidiendo = false
            }
        },
  },

  computed: {},
};
</script>

<style scoped lang="css">
.src-components-formulario {
}
</style>
