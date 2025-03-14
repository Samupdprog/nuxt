<script setup>
definePageMeta({
  layout: "dashboard",
});

import { ref, onMounted, computed } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const orders = ref([]);
const editMode = ref(false);
const filters = ref({
  id: "",
  email: "",
  status: "",
});

const headers = [
  { text: "Order Number", value: "order_number" },
  { text: "Customer Email", value: "customer_email" },
  { text: "Status", value: "status" },
  { text: "Total Price", value: "order_total" },
  { text: "Products", value: "products" },
  { text: "Actions", value: "actions" },
];

const fetchOrders = async () => {
  try {
    const response = await fetch("http://localhost:1880/get-all-orders");
    if (!response.ok) throw new Error("Failed to fetch orders");
    const data = await response.json();
    orders.value = data.orders || [];
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

const filteredOrders = computed(() => {
  return orders.value.filter((order) => {
    return (
      (!filters.value.id || order.order_number.includes(filters.value.id)) &&
      (!filters.value.email || order.customer_email.includes(filters.value.email)) &&
      (!filters.value.status || order.status.includes(filters.value.status))
    );
  });
});

const viewOrder = (order) => {
  router.push(`/ecommerce/orders/${order.order_number}`);
};

const editOrder = (order) => {
  console.log("Saving order:", order);
};

const formatCurrency = (value, currency) => {
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: currency === "CZK" ? "CZK" : "EUR",
  }).format(value);
};

onMounted(fetchOrders);
</script>

<template>
  <v-container fluid>
    <v-card elevation="10" class="pa-5">
      <v-card-title class="d-flex justify-space-between align-center">
        <span class="text-h5">Order List</span>
        <v-btn color="info" @click="$router.push('./config')">
          <i class="bi bi-gear"></i> API Configuration
        </v-btn>
      </v-card-title>

      <!-- Filtros -->
      <v-row class="mb-4">
        <v-col cols="12" md="3">
          <v-text-field v-model="filters.id" label="Filter by Order Number" clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field v-model="filters.email" label="Filter by Customer Email" clearable></v-text-field>
        </v-col>
        <v-col cols="12" md="3">
          <v-select
            v-model="filters.status"
            :items="['Pending', 'Confirmed', 'Shipped', 'Storno']"
            label="Filter by Status"
            clearable
          ></v-select>
        </v-col>
        <v-col cols="12" md="3" class="d-flex">
          <v-btn color="primary" class="mr-2" @click="fetchOrders">Refresh</v-btn>
          <v-btn color="secondary" @click="filters = { id: '', email: '', status: '' }">Reset</v-btn>
        </v-col>
      </v-row>

      <v-switch v-model="editMode" label="Edit Mode" color="primary" class="mb-3"></v-switch>

      <v-data-table
        :headers="headers"
        :items="filteredOrders"
        item-value="order_number"
        class="elevation-2"
        dense
        disable-pagination
      >
        <!-- Order Number -->
        <template v-slot:item.order_number="{ item }">
          {{ item.order_number }}
        </template>

        <!-- Customer Email -->
        <template v-slot:item.customer_email="{ item }">
          <v-text-field v-if="editMode" v-model="item.customer_email" dense solo hide-details></v-text-field>
          <span v-else>{{ item.customer_email }}</span>
        </template>

        <!-- Status -->
        <template v-slot:item.status="{ item }">
          <v-select
            v-if="editMode"
            v-model="item.status"
            :items="['Pending', 'Confirmed', 'Shipped', 'Storno']"
            dense solo hide-details
          ></v-select>
          <span v-else>{{ item.status }}</span>
        </template>

        <!-- Total Price -->
        <template v-slot:item.order_total="{ item }">
          <span v-if="!editMode">{{ formatCurrency(item.order_total, item.currency) }}</span>
        </template>

        <!-- Products -->
        <template v-slot:item.products="{ item }">
          <div v-for="product in item.products" :key="product.title" class="d-flex align-center">
            <v-img :src="product.image_url" max-width="50" class="mr-2"></v-img>
            <span>{{ product.title }}</span>
          </div>
        </template>

        <!-- Actions -->
        <template v-slot:item.actions="{ item }">
          <v-btn variant="outlined" color="primary" size="small" @click="viewOrder(item)">
            <i class="bi bi-eye"></i> View
          </v-btn>
          <v-btn v-if="editMode" color="success" size="small" class="ml-2" @click="editOrder(item)">
            <i class="bi bi-pencil"></i> Save
          </v-btn>
        </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>
