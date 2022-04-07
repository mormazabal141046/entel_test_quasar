<template>
  <q-page class="container flex flex-center column full-width	">
    <div class="row flex justify-between q-px-lg full-width	items-center content-center">
      <div class="col-3">
        <q-select outlined v-model="category" :options="categories" label="Categorias" @input="searchEntriesByCategory" class="" />
      </div>
      <div class="col-3 flex justify-end">
          <q-btn type="a" color="red" icon="las la-plus-circle"  label="Nuevo" rounded @click="registerForm = !registerForm" />
      </div>
    </div>
    <div class="q-pa-lg row full-width flex justify-center">
      <q-table title="Entries" :data="entries" :columns="columns" row-key="link" :pagination="initialPagination" :filter="filter" class="full-width" :loading="loading" rows-per-page-label="Registros por página: " no-results-label="No se han encontrado registros">
        <template v-slot:top-right>
          <q-input borderless dense debounce="300" v-model="filter" placeholder="Buscar">
            <q-icon slot="append" name="search" />
          </q-input>
        </template>
         <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="category" :props="props">
            {{ props.row.Category }}
          </q-td>
          <q-td key="api" :props="props">
            {{ props.row.API }}
          </q-td>
          <q-td key="description" :props="props">
            {{ props.row.Description }}
          </q-td>
          <q-td key="link" :props="props">
            <q-btn type="a" color="red" icon="las la-globe"  label="Visitar" outline rounded :href="props.row.Link" />
          </q-td>
        </q-tr>
      </template>
      </q-table>
    </div>
    <q-dialog v-model="registerForm" persistent transition-show="flip-down" transition-hide="flip-up">
      <q-card class="bg-primary text-white" style="width: 700px; max-width: 80vw;">
        <q-bar>
          <q-space />
          <q-btn dense flat icon="close" v-close-popup>
            <q-tooltip class="bg-white text-primary">Close</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section>
          <div class="text-h6">Nuevo Registro</div>
        </q-card-section>

        <q-card-section class="q-pt-none bg-white text-grey-10">
          <div class="q-gutter-sm">
            <q-radio v-model="choice_register" val="category" label="Categoria" />
            <q-radio v-model="choice_register" val="entry" label="Entrada" />
          </div>
          <div v-if="choice_register == 'category'">
            <h6 class="q-ma-none q-mb-md">Nueva Categoria</h6>
            <q-input filled v-model="new_category" label="Categoria" stack-label dense />
          </div>
          <div v-if="choice_register == 'entry'">
            <h6 class="q-ma-none q-mb-md">Nueva Entrada</h6>
            <q-input class="q-mb-md" filled v-model="new_entry.API" label="API" stack-label dense />
            <q-input class="q-mb-md" filled v-model="new_entry.Description" label="Description" stack-label dense />
            <q-input class="q-mb-md" filled v-model="new_entry.Auth" label="Auth" stack-label dense />
            <q-toggle class="q-mb-md" v-model="new_entry.HTTPS" color="red" label="HTTPS" left-label />
            <q-input class="q-mb-md" filled v-model="new_entry.Cors" label="Cors" stack-label dense />
            <q-input class="q-mb-md" filled v-model="new_entry.Link" label="Link" stack-label dense />
            <q-select v-model="new_entry.Category" :options="categories" label="Categorias" filled  dense />
          </div>
        </q-card-section>
        <q-card-actions align="right" class="bg-white text-grey-10">
          <q-btn flat color="grey-10" label="Cancelar" v-close-popup />
          <q-btn v-if="choice_register != ''" label="Guardar" @click="saveRegister(choice_register)" color="red" :unelevated="true" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
// import { Notify } from 'quasar';

export default {
  name: 'PageIndex',
  data(){
    return{
      loading: false,
      choice_register: "",
      category: "Seleccione",
      registerForm: false,
      new_category:'',
      new_entry: {
        API: "",
        Description: "",
        Auth: "",
        HTTPS: true,
        Cors: "",
        Link: "",
        Category: ""
		  },
      entries: [],
      categories:[],
      filter: '',
      initialPagination: {
        sortBy: 'desc',
        descending: false,
        page: 1,
        rowsPerPage: 10
      },
      columns: [
        { name: 'category', label: 'Categoria', field: 'Category', sortable: true, align:'center'},
        { name: 'api', label: 'API', field: 'API', sortable: true, align:'center'},
        { name: 'description', label: 'Descripción', field: 'Description', sortable: true, align:'center' },
        { name: 'link', label: 'Link', field: 'Link', sortable: true, align:'center' },
      ]
    }
  },
  async mounted(){
    await this.getCategories();
    await this.getEntries();
    this.loadStorage();
  },
  methods:{
    async getCategories(){
      let url = "https://api.publicapis.org/categories";
      await fetch(url).then( resp => {
        return resp.json();
      }).then( data =>{
        this.categories = data.categories;
      }).catch( error =>{
        console.log("Error");
      })
    },
    async getEntries(category=null){
      this.loading = true;
      let url = "https://api.publicapis.org/entries";
      if(category != null){
        url = `${url}?category=${category}`
      }
      await fetch(url).then( resp => {
        return resp.json();
      }).then( data =>{
        if(data){
          this.entries = data.entries;
        }
        this.loading = false;
      }).catch( error =>{
        this.loading = false;
        console.log("Error");
      })
    },
    searchEntriesByCategory(){
      this.getEntries(this.category);
    },
    saveRegister(type){
      if(type == 'category'){
        if(this.$q.localStorage.getItem('categories')){
          let storage_categories = this.$q.localStorage.getItem('categories');
          storage_categories.push(this.new_category);
          this.categories.push(this.new_category);
          this.$q.localStorage.set('categories', storage_categories)
        }else{
          this.$q.localStorage.set('categories', [this.new_category])
        }
      }
      if(type == 'entry'){
        if(this.$q.localStorage.getItem('entries')){
          let storage_entries = this.$q.localStorage.getItem('entries');
          storage_entries.push(this.new_entry);
          this.entries.push(this.new_entry);
          this.$q.localStorage.set('entries', storage_entries)
        }else{
          this.$q.localStorage.set('entries', [this.new_entry])
        }
      }

      // Notificacion Final
      let type_message = (type == 'category' ? 'Categoria' : 'Entrada')
      this.$q.notify({
        message: `${type_message} Ingresada correctamente!`,
        color: 'green-4',
        icon: 'check_circle'
      })
      this.registerForm = false;
    },
    loadStorage(){
      if(this.$q.localStorage.getItem('categories')){
        let storage_categories = this.$q.localStorage.getItem('categories');
        storage_categories.forEach(item =>{
          this.categories.push(item);
        })
      }
      if(this.$q.localStorage.getItem('entries')){
        let storage_entries = this.$q.localStorage.getItem('entries');
        storage_entries.forEach(item =>{
          this.entries.push(item);
        })
      }
    },
    clearInputs(){
      this.new_entry = {
        API: "",
        Description: "",
        Auth: "",
        HTTPS: true,
        Cors: "",
        Link: "",
        Category: ""
		  };
      this.new_category="";
      this.choice_register="";
    }
  }
}
</script>

<style>

</style>
