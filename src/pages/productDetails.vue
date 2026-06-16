<script setup lang="ts">
import type { Producto } from '../types';
defineProps<{
    products: Producto[];
}>();


</script>

<template>
  <div>
    <div 
      v-if="products.length === 0"
      class="flex flex-col min-h-64 items-center justify-center rounded-lg border p-8 text-center"
    >
      <UIcon name="i-ri-error-warning-line" :size="48" class="mb-4 text-red-500" />
      <h3>No hay productos disponibles</h3>
      <p class="text-gray-600">
        Intenta agregar productos a tu carrito para verlo aqui.
      </p>
    </div>
    
    <div v-else>
      <Ucard v-for="product in products" :key="product.id">
        <div class="flex gap-4">
          <img :src="product.image" alt="Imagen del producto" class="w-32 h-32 object-cover rounded">
          
          <div>
            <h3 class="font-semibold">
              {{ product.title }}
            </h3>
            <p class="text-gray-600">
              {{ product.description }}
            </p>
            <p class="text-green-600 font-bold mt-2">
              ${{ product.price }}
            </p>
          </div>
        </div>
      </Ucard>
    </div>
  </div>
</template>


<style scoped>
</style>