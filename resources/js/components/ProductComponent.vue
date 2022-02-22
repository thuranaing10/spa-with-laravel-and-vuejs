<template>
    <div class="container my-5">
        <div class="row justify-content-end mb-4">
            <div class="col-4">
                <button class="btn btn-primary" @click="create">
                    <i class="fas fa-plus-circle mr-1"></i> Create
                </button>
            </div>
            <div class="col-4">
                <form @submit.prevent="view">
                    <div class="input-group">
                        <input
                            type="text"
                            class="form-control"
                            placeholder="search..."
                            v-model="search"
                        />
                        <div class="input-group-append">
                            <button type="submit" class="btn btn-primary">
                                <i class="fas fa-search"></i>
                            </button>
                        </div>
                    </div>
                </form>
            </div>
        </div>
        <div class="row">
            <div class="col-4">
                <div class="card">
                    <h4 class="card-header">
                        {{ isEditMode ? "Edit" : "Create" }}
                    </h4>
                    <div class="card-body">
                        <alert-error
                            :form="product"
                            :message="errorMessage"
                        ></alert-error>

                        <form @submit.prevent="isEditMode ? update() : store()">
                            <div class="form-group">
                                <label for="name">Name:</label>
                                <input
                                    type="text"
                                    id="name"
                                    class="form-control"
                                    v-model="product.name"
                                    :class="{
                                        'is-invalid': product.errors.has('name')
                                    }"
                                />
                                <has-error
                                    :form="product"
                                    field="name"
                                ></has-error>
                            </div>
                            <div class="form-group">
                                <label for="price">Price:</label>
                                <input
                                    type="number"
                                    id="price"
                                    class="form-control"
                                    v-model="product.price"
                                    :class="{
                                        'is-invalid': product.errors.has(
                                            'price'
                                        )
                                    }"
                                />
                                <has-error
                                    :form="product"
                                    field="price"
                                ></has-error>
                            </div>
                            <button class="btn btn-primary" type="submit">
                                <i class="fas fa-save mr-1"></i>
                                {{ isEditMode ? "Update" : "Save" }}
                            </button>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col-8">
                <!-- Table Start -->
                <table class="table">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Name</th>
                            <th>Price</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="product in products.data" :key="product.id">
                            <td>{{ product.id }}</td>
                            <td>{{ product.name }}</td>
                            <td>{{ product.price }}</td>
                            <td>
                                <button
                                    class="btn btn-success btn-sm"
                                    @click="edit(product)"
                                >
                                    <i class="fas fa-edit mr-1"></i> Edit
                                </button>
                                <button
                                    class="btn btn-danger btn-sm"
                                    @click="destroy(product.id, $event)"
                                >
                                    <i class="fas fa-trash-alt mr-1"></i> Delete
                                </button>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <!-- Table End -->

                <!-- Pagination Start -->
                <pagination
                    :data="products"
                    @pagination-change-page="view"
                ></pagination>
                <!-- Pagination End -->

                <!-- Loading Start -->
                <!-- <loading
                    :active.sync="isLoading"
                    :is-full-page="true"
                ></loading> -->
                <!-- Loading End -->
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "ProductComponent",
    data() {
        return {
            products: {},
            product: new Form({
                id: "",
                name: "",
                price: ""
            }),

            isEditMode: false,
            search: "",
            errorMessage: ""
        };
    },
    created() {
        this.view();
    },
    methods: {
        view(page = 1) {
            this.$Progress.start();
            let loader = this.$loading.show();
            axios
                .get(`/api/products?page=${page}&search=${this.search}`)
                .then(response => {
                    this.products = response.data;
                    loader.hide();
                    this.$Progress.finish();
                })
                .catch(error => {
                    this.$Progress.fail();
                });
        },
        create() {
            this.product.clear();
            this.isEditMode = false;

            // this.product.name = "";
            // this.product.price = "";

            this.product.reset();
        },
        store() {
            this.product
                .post("/api/products")
                .then(response => {
                    this.view();
                    // this.product.name = "";
                    // this.product.price = "";

                    this.product.reset();
                    Toast.fire({
                        icon: "success",
                        title: "Created successfully"
                    });
                })
                .catch(error => {
                    this.errorMessage = error.response.data.message;
                });
        },
        edit(product) {
            this.product.clear();
            this.isEditMode = true;
            // this.product.id = product.id;
            // this.product.name = product.name;
            // this.product.price = product.price;

            this.product.fill(product);
        },
        update() {
            this.product
                .put(`/api/products/${this.product.id}`)
                .then(response => {
                    this.view();
                    // this.product.name = "";
                    // this.product.price = "";
                    this.isEditMode = false;
                    this.product.reset();
                    Toast.fire({
                        icon: "success",
                        title: "Updated successfully"
                    });
                })
                .catch(error => {
                    this.errorMessage = error.response.data.message;
                });
        },
        destroy(id) {
            Swal.fire({
                title: "Are you sure?",
                icon: "warning",
                showCancelButton: true,
                confirmButtonColor: "#3085d6",
                cancelButtonColor: "#d33",
                confirmButtonText: "Yes, delete it!"
            }).then(result => {
                if (result.isConfirmed) {
                    axios.delete(`/api/products/${id}`).then(response => {
                        this.view();
                        Swal.fire({ title: "Deleted!", icon: "success" });
                        Toast.fire({
                            icon: "success",
                            title: "Deleted successfully"
                        });
                    });
                }
            });
        }
    }
};
</script>
