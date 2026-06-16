<script setup lang="ts">
import productitem from './productitem.vue';
</script>

<template>
  <UDashboardPanel id="invoices">
    <template #header>
      <UDashboardNavbar title="Facturas" :ui="{ right: 'gap-3' }">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>


        <template #right>
          <UButton variant="outline" color="primary" size="sm">
            Agregar Facturas
          </UButton>
        </template>
      </UDashboardNavbar>
    </template>

    
    <template #body>
      <productitem />
    </template>
  </UDashboardPanel>
</template>


<style scoped>
</style>