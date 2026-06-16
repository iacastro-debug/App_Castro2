<script setup lang="ts">
import type { Producto } from '../types';

defineProps<{
  products: Producto[];
}>();

const emit = defineEmits<{
  'add-to-cart': [product: Producto];
}>();

const addToCart = (product: Producto) => {
  emit('add-to-cart', product);
};
</script>

<template>
  <div class="grid gap-6 p-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
    <UCard
      v-for="product in products"
      :key="product.id"
      class="overflow-hidden transition hover:shadow-lg"
    >
      <img :src="product.image" alt="Product Image" class="h-48 object-cover">
      <UCardBody class="flex flex-col gap-2">
        <UCardTitle>{{ product.title }}</UCardTitle>
        <p class="text-sm text-gray-600">
          {{ product.description }}
        </p>
        <p class="text-sm text-gray-600">
          {{ product.category }}
        </p>
      </UCardBody>
      <UCardFooter class="flex items-center justify-between mt-4">
        <UBadge class="text-lg font-bold">
          ${{ product.price }}
        </UBadge>
        <UButton
          variant="outline"
          color="primary"
          size="sm"
          @click="addToCart(product)"
        >
          <UIcon name="i-ri-add-line" :size="32" />
        </UButton>
      </UCardFooter>
    </UCard>
  </div>
</template>

<style scoped>
</style>