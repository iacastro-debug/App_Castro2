<script setup lang="ts">
import { ref } from "vue";
import type { NavigationMenuItem } from "@nuxt/ui";

const open = ref(false);

const links = [
  { label: "Inicio", icon: "i-lucide-house", to: "/", onSelect: () => { open.value = false } },
  { label: "Clientes", icon: "i-lucide-users", to: "/customers", onSelect: () => { open.value = false } },
  { label: "Productos", icon: "i-lucide-box", to: "/products", onSelect: () => { open.value = false } },
  { label: "Facturas", icon: "i-lucide-receipt", to: "/invoices", onSelect: () => { open.value = false } }
] satisfies NavigationMenuItem[];


</script>

<template>
  <UDashboardGroup unit="rem" storage="local">
    <UDashboardSidebar
      id="default"
      v-model:open="open"
      collapsible
      resizable
      class="bg-elevated/25"
      :ui="{ footer: 'lg:border-t lg:border-default' }"
    >
      <template #header="{ collapsed }">
        <UButton
          color="neutral"
          variant="ghost"
          block
          :square="collapsed"
          :label="collapsed ? undefined : 'Gestor de clientes'"
          icon="i-lucide-users-round"
          class="data-[state=open]:bg-elevated"
        />
      </template>

      <template #default="{ collapsed }">
        <UNavigationMenu
          :collapsed="collapsed"
          :items="links"
          orientation="vertical"
          tooltip
        />
      </template>

      <template #footer="{ collapsed }">
        <UserMenu :collapsed="collapsed" />
      </template>
    </UDashboardSidebar>

    <RouterView />
  </UDashboardGroup>
</template>
