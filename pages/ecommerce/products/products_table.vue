<script setup>
definePageMeta({
  layout: "dashboard",
});

import { ref, onMounted, computed } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const products = ref([]);
const editMode = ref(false);
const filters = ref({
  code: "",
  name: "",
});

const headers = [
  { text: "Code", value: "code" },
  { text: "Product ID", value: "product_id" },
  { text: "Name", value: "name" },
  { text: "SEO URL", value: "seo_url" },
  { text: "SEO Title", value: "seo_title" },
  { text: "SEO Description", value: "seo_description" },
  { text: "Actions", value: "actions" },
];

const fetchProducts = async () => {
  try {
    const response = await fetch("http://localhost:1880/get-all-products", {
      method: "GET",
      headers: {
        "Accept": "application/json",
        "Content-Type": "application/json"
      }
    });
    if (!response.ok) throw new Error("Error retrieving products");
    const data = await response.json();
    // Suponemos que la respuesta es un array de objetos
    products.value = Array.isArray(data) ? data : [];
  } catch (error) {
    console.error("Error fetching products:", error);
  }
};

const editProduct = async (product) => {
  try {
    const response = await fetch("http://localhost:1880/update-product", {
      method: "POST",
      headers: {
        "Accept": "application/json",
        "Content-Type": "application/json"
      },
      body: JSON.stringify(product)
    });
    if (!response.ok) throw new Error("Error updating product");
    const result = await response.json();
    console.log("Product updated:", result);
  } catch (error) {
    console.error("Error updating product:", error);
  }
};

const filteredProducts = computed(() => {
  return products.value.filter((product) => {
    return (
      (!filters.value.code ||
        product.code.toString().includes(filters.value.code)) &&
      (!filters.value.name ||
        product.name.toLowerCase().includes(filters.value.name.toLowerCase()))
    );
  });
});

const viewProduct = (product) => {
  router.push(`/ecommerce/products/${product.code}`);
};

onMounted(fetchProducts);
</script>


<template>
  <v-container fluid>
    <v-card elevation="10" class="pa-5">
      <v-card-title class="d-flex justify-space-between align-center">
        <span class="text-h5">Product List</span>
        <v-btn color="info" @click="$router.push('./config')">
          <i class="bi bi-gear"></i> API Configuration
        </v-btn>
      </v-card-title>

      <!-- Filtros -->
      <v-row class="mb-4">
        <v-col cols="12" md="3">
          <v-text-field v-model="filters.code" label="Filter by Code" clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field v-model="filters.name" label="Filter by Name" clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="3" class="d-flex align-center">
          <v-btn color="primary" class="mr-2" @click="fetchProducts">Refresh</v-btn>
          <v-btn color="secondary" @click="filters = { code: '', name: '' }">Reset</v-btn>
        </v-col>
      </v-row>

      <v-switch v-model="editMode" label="Edit Mode" color="primary" class="mb-3"></v-switch>

      <v-data-table
        :headers="headers"
        :items="filteredProducts"
        item-value="code"
        class="elevation-2"
        dense
        disable-pagination
      >
        <!-- Code -->
        <template v-slot:item.code="{ item }">
          {{ item.code }}
        </template>

        <!-- Product ID -->
        <template v-slot:item.product_id="{ item }">
          {{ item.product_id }}
        </template>

        <!-- Name -->
        <template v-slot:item.name="{ item }">
          <v-text-field
            v-if="editMode"
            v-model="item.name"
            dense
            solo
            hide-details
          ></v-text-field>
          <span v-else>{{ item.name }}</span>
        </template>

        <!-- SEO URL -->
        <template v-slot:item.seo_url="{ item }">
          <a :href="item.seo_url" target="_blank">{{ item.seo_url }}</a>
        </template>

        <!-- SEO Title -->
        <template v-slot:item.seo_title="{ item }">
          <v-text-field
            v-if="editMode"
            v-model="item.seo_title"
            dense
            solo
            hide-details
          ></v-text-field>
          <span v-else>{{ item.seo_title }}</span>
        </template>

        <!-- SEO Description -->
        <template v-slot:item.seo_description="{ item }">
          <v-text-field
            v-if="editMode"
            v-model="item.seo_description"
            dense
            solo
            hide-details
          ></v-text-field>
          <span v-else>{{ item.seo_description }}</span>
        </template>

        <!-- Acciones -->
        <template v-slot:item.actions="{ item }">
          <v-btn variant="outlined" color="primary" size="small" @click="viewProduct(item)">
            <i class="bi bi-eye"></i> View
          </v-btn>
          <v-btn
            v-if="editMode"
            color="success"
            size="small"
            class="ml-2"
            @click="editProduct(item)"
          >
            <i class="bi bi-pencil"></i> Save
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>
