<script setup lang="ts">
import productitem from './productitem.vue';
import {ref} from "vue";
import type { Producto } from '../types';
import productDetails from './productDetails.vue';


const getProducts = async (): Promise<Producto[]>=>{
    try{
        const response = await fetch('https://fakestoreapi.com/products');
        if (!response.ok){
            throw new Error (`HTTP error! status: ${response.status}`);
        }
       return await response.json();   
    }  catch (error) {
        console.error('Error fetching products:', error);
        return [];
    }
    
};

const carrito = ref<Producto[]>([]);

const products = await getProducts();

const addToCart= (product: Producto) =>{
    carrito.value.push(product);
    console.log("Producto agregado al carrito", product);
};

const showCart = ref(false);

</script>

<template>
  <UDashboardPanel id="products">
    <template #header>
      <UDashboardNavbar title="Productos" :ui="{ right: 'gap-3' }">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>
        <template #right>
          <UBadge 
            color="primary"
            variant="outline" 
            class="cursor-pointer"
            role="button" 
            tabindex="0"
            @click="showCart = true"
            @keydown.enter="showCart = true"
            @keydown.space.prevent="showCart = true"
          >
          
            <UIcon name="i-ri-shopping-cart-line" :size="32"/>{{ carrito.length }}
             
          </UBadge>
        </template>

      </UDashboardNavbar>
    </template>

    
    <template #body>
      <productitem :products="products" @add-to-cart="addToCart"></productitem>
      
    </template>
  </UDashboardPanel>

  <USlideover
  v-model:open="showCart"
  title="Carrito de Compras"
  description="Aqui puedes revisar los productos que has agregado a tu carrito">
  <template #body>
    <ProductDetails :products="carrito"/>
  </template>
  </USlideover>
</template>

 

<style scoped>
</style>

