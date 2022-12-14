<template>
  <div class="container mt-5">
   <div class="row">
     <div class="col-3 me-5">
      <h2>{{ isEdit ? 'Edit Product' : 'Create Product' }}</h2>
      <form @submit.prevent="isEdit ? update() : store()" enctype="multipart/form-data">
        <div class="form-group">
          <label for="">Name</label>
          <input type="text" class="form-control" v-model="product.name">
          <div v-if="errorMessage">
            <p v-for="error in errors.name" :key="error" class="alert alert-danger">{{ error }}</p>
          </div>
        </div>
        <div class="form-group">
          <label for="">Price</label>
          <input type="number" class="form-control" v-model="product.price">
          <div v-if="errorMessage">
            <p v-for="error in errors.price" :key="error" class="alert alert-danger">{{ error }}</p>
          </div>
        </div>
        <div class="form-group">
          <label for="">Image</label>
          <input type="file" class="form-control" name="image" @change="onChange" id="image">
          <img :src="imagePreview" v-show="showImage" width="100" height="100" alt="Image">
          <div v-if="errorMessage">
            <p v-for="error in errors.image" :key="error" class="alert alert-danger">{{ error }}</p>
          </div>
        </div>
        <button type="submit" class="btn btn-primary mt-3">{{ isEdit ? 'Edit' : 'Create' }}</button>
      </form>
    </div>
    <div class="col-8 ms-5">
      <div class="d-flex justify-content-between">
        <button class="btn btn-primary mb-3" @click="create">Create</button>
        <div class="me-5" @keyup="productList">
          <input type="search" class="form-control" v-model="search" placeholder="Search ... ">
        </div>
      </div>
      <table class="table" id="my-table">
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Price</th>
          <th>Image</th>
          <th>Action</th>
        </tr>
        <tr v-for="product in products.data" :key="product.id">
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>
            <img :src="product.image" alt="" width="70" height="70">
          </td>
          <td>
            <button class="btn btn-sm btn-secondary text-dark" @click="edit(product)">Edit</button>
            <button class="btn btn-sm btn-danger text-dark" @click="destroy(product.id)">Delete</button>
          </td>
        </tr>
      </table>
      <b-pagination
      v-model="currentPage"
      :total-rows="rows"
      :per-page="perPage"
      aria-controls="my-table"
      @input="paginate(currentPage)"
    ></b-pagination>
    </div>
   </div>
  </div>
</template>

<script>
export default {
  data(){
    return {
      isEdit: false,
      products: {},
      product: {
        id: '',
        name: '',
        price: '',
        image: '',
      },
      search: '',
      showImage: false,
      imagePreview: null,
      errors: [],
      errorMessage: false,
      perPage: null,
      currentPage: null,
      rows: null,
    }
  },
  methods: {
    productList(page = 1){
      this.$axios.get(`http://127.0.0.1:8000/api/products?page=${page}&search=${this.search}`)
      .then(response => {
        this.products = response.data
        this.currentPage = response.data.current_page
        this.rows = response.data.total
        this.perPage = response.data.per_page
      })
    },
    paginate(page){
      this.$axios.get('http://127.0.0.1:8000/api/products?page='+page)
      .then(response => {
        this.products = response.data 
      })
    },
    imageUrl(image){
      return `http://127.0.0.1:8000/storage/${image}`
    },
    create(){
      this.isEdit = false
      this.product.id = this.product.name = this.product.price = this.product.image = ''
      this.showImage = false
      this.imagePreview = null
      document.getElementById('image').value = ''
      this.errorMessage = false
    },
    store(){
      let formData = new FormData();
      formData.append('id', this.product.id)
      formData.append('name', this.product.name)
      formData.append('price', this.product.price)
      formData.append('image', this.product.image)
      this.$axios.post('http://127.0.0.1:8000/api/products', formData)
      .then(response => {
        this.productList()
        this.product = { id: '', name: '', price: '' }
        this.showImage = false
        this.imagePreview = null
        document.getElementById('image').value = ''
        this.errorMessage = false
      })
      .catch(error => {
        this.errors = error.response.data.errors
        this.errorMessage = true
      })
    },
    onChange(e){
      this.product.image = e.target.files[0];
      let reader = new FileReader();
      reader.addEventListener('load', function(){
        this.showImage = true
        this.imagePreview = reader.result
      }.bind(this), false)
      if(this.product.image){
        reader.readAsDataURL(this.product.image)
      }
    },
    edit(product){
      this.isEdit = true
      this.product.id = product.id
      this.product.name = product.name
      this.product.price = product.price
      this.product.image = product.image
      this.imagePreview = this.imageUrl(product.image)
      this.showImage = true
      this.errorMessage = false
    },
    update()
    {
      let formData = new FormData();
      formData.append('id', this.product.id)
      formData.append('name', this.product.name)
      formData.append('price', this.product.price)
      formData.append('image', this.product.image)
      this.$axios.post(`http://127.0.0.1:8000/api/product-update/${this.product.id}`, formData)
      .then(response => {
        this.productList();
        this.product = { id: '', name: '', price: '' }
        this.showImage = false
        this.imagePreview = null
        document.getElementById('image').value = ''
        this.isEdit = false
        this.errorMessage = false
      })
      .catch(error => {
        this.errors = error.response.data.errors
        this.errorMessage = true
      })
    },
    destroy(id){
      this.$axios.delete(`http://127.0.0.1:8000/api/products/${id}`)
      .then(response => this.productList())
    }
  },
  mounted(){
    this.productList();
  }
}
</script>

<style>

</style>
