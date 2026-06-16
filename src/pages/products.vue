<script setup lang="ts">
import { computed, h, reactive, ref, resolveComponent, useTemplateRef, watch } from 'vue'
import { useStorage } from '@vueuse/core'
import { getPaginationRowModel, type Row } from '@tanstack/table-core'
import type { FormSubmitEvent, TableColumn } from '@nuxt/ui'
import * as z from 'zod'

// 1. Tipos e Interfaz basados en el requerimiento de la clase
type ProductoEstado = 'disponible' | 'agotado' | 'inactivo'

interface Producto {
  id: number
  nombre: string
  descripcion: string
  precio: number
  stock: number
  categoria: string
  imagen: string
  marca?: string
  estado: ProductoEstado
}

const UAvatar = resolveComponent('UAvatar')
const UButton = resolveComponent('UButton')
const UBadge = resolveComponent('UBadge')
const UDropdownMenu = resolveComponent('UDropdownMenu')
const UCheckbox = resolveComponent('UCheckbox')

// 2. Configuraciones de estados y colores
const estadoItems: Array<{ label: string, value: ProductoEstado }> = [
  { label: 'Disponible', value: 'disponible' },
  { label: 'Agotado', value: 'agotado' },
  { label: 'Inactivo', value: 'inactivo' }
]
const estadoFiltroItems: Array<{ label: string, value: ProductoEstado | 'todos' }> = [
  { label: 'Todos', value: 'todos' },
  ...estadoItems
]
const estadoColors: Record<ProductoEstado, 'success' | 'error' | 'neutral'> = {
  disponible: 'success',
  agotado: 'error',
  inactivo: 'neutral'
}

// 3. Datos iniciales quemados para pruebas
const productosIniciales: Producto[] = [
  {
    id: 5001,
    nombre: 'Laptop Gamer X',
    descripcion: 'Laptop potente con RTX 4060 y 16GB RAM',
    precio: 1299.99,
    stock: 15,
    categoria: 'Tecnología',
    imagen: 'https://images.unsplash.com/photo-1603302576837-37561b2e2302',
    marca: 'Asus',
    estado: 'disponible'
  },
  {
    id: 5002,
    nombre: 'Mouse Ergonómico Inalámbrico',
    descripcion: 'Mouse para oficina recargable de alta precisión',
    precio: 45.50,
    stock: 0,
    categoria: 'Accesorios',
    imagen: 'https://images.unsplash.com/photo-1615663245857-ac93bb7c39e7',
    marca: 'Logitech',
    estado: 'agotado'
  },
  {
    id: 5003,
    nombre: 'Teclado Mecánico RGB',
    descripcion: 'Teclado switch azul con distribución en español',
    precio: 89.90,
    stock: 8,
    categoria: 'Accesorios',
    marca: 'Redragon',
    imagen: 'https://images.unsplash.com/photo-1587829741301-dc798b83add3',
    estado: 'disponible'
  }
]

// 4. Esquema de validación Zod ajustado a Productos
const schema = z.object({
  nombre: z.string().min(2, 'Ingresa al menos 2 caracteres'),
  descripcion: z.string().min(5, 'Ingresa una descripción detallada'),
  precio: z.number().positive('El precio debe ser mayor a 0'),
  stock: z.number().int().nonnegative('El stock no puede ser negativo'),
  categoria: z.string().min(2, 'Ingresa la categoría'),
  imagen: z.string().url('Ingresa una URL de imagen válida'),
  marca: z.string().optional(),
  estado: z.enum(['disponible', 'agotado', 'inactivo'])
})

type Schema = z.output<typeof schema>

const toast = useToast()
const table = useTemplateRef('table')
const productos = useStorage<Producto[]>('productos-crud-v1', productosIniciales)

const search = ref('')
const estadoFiltro = ref<ProductoEstado | 'todos'>('todos')
const columnVisibility = ref()
const rowSelection = ref({})
const pagination = ref({
  pageIndex: 0,
  pageSize: 8
})

const formOpen = ref(false)
const editingId = ref<number | null>(null)
const state = reactive<Partial<Schema>>({
  nombre: '',
  descripcion: '',
  precio: 0,
  stock: 0,
  categoria: '',
  imagen: '',
  marca: '',
  estado: 'disponible'
})

const deleteOpen = ref(false)
const deleteIds = ref<number[]>([])

// 5. Filtro de búsqueda
const productosFiltrados = computed(() => {
  const term = search.value.trim().toLowerCase()

  return productos.value.filter((producto) => {
    const matchesSearch = !term || [
      producto.nombre,
      producto.descripcion,
      producto.categoria,
      producto.marca || ''
    ].some(value => value.toLowerCase().includes(term))
    const matchesEstado = estadoFiltro.value === 'todos' || producto.estado === estadoFiltro.value

    return matchesSearch && matchesEstado
  })
})

// 6. Tarjetas de resumen
const resumen = computed(() => [{
  label: 'Total productos',
  value: productos.value.length,
  icon: 'i-lucide-box',
  leadingClass: 'p-2.5 rounded-full bg-primary/10 ring ring-inset ring-primary/25'
}, {
  label: 'Disponibles',
  value: productos.value.filter(p => p.estado === 'disponible').length,
  icon: 'i-lucide-check-circle',
  leadingClass: 'p-2.5 rounded-full bg-success/10 ring ring-inset ring-success/25'
}, {
  label: 'Agotados',
  value: productos.value.filter(p => p.estado === 'agotado').length,
  icon: 'i-lucide-alert-triangle',
  leadingClass: 'p-2.5 rounded-full bg-error/10 ring ring-inset ring-error/25'
}, {
  label: 'Inactivos',
  value: productos.value.filter(p => p.estado === 'inactivo').length,
  icon: 'i-lucide-ban',
  leadingClass: 'p-2.5 rounded-full bg-neutral/10 ring ring-inset ring-neutral/25'
}])

const modalTitle = computed(() => editingId.value === null ? 'Nuevo producto' : 'Editar producto')
const submitLabel = computed(() => editingId.value === null ? 'Crear producto' : 'Guardar cambios')
const deleteCount = computed(() => deleteIds.value.length)
const deleteTitle = computed(() => deleteCount.value === 1 ? 'Eliminar producto' : `Eliminar ${deleteCount.value} productos`)
const deleteDescription = computed(() => {
  if (deleteCount.value === 1) {
    const producto = productos.value.find(item => item.id === deleteIds.value[0])
    return `Se eliminará "${producto?.nombre || 'este producto'}" de la lista local.`
  }
  return 'Se eliminarán los productos seleccionados de la lista local.'
})

watch([search, estadoFiltro], () => {
  pagination.value = { ...pagination.value, pageIndex: 0 }
  rowSelection.value = {}
})

function getEstadoLabel(estado: ProductoEstado): string {
  return estadoItems.find(item => item.value === estado)?.label || estado
}

function getColumnLabel(columnId: string): string {
  return {
    id: 'ID',
    nombre: 'Producto',
    precio: 'Precio',
    stock: 'Stock',
    categoria: 'Categoría',
    estado: 'Estado'
  }[columnId] || columnId
}

function resetForm() {
  Object.assign(state, {
    nombre: '',
    descripcion: '',
    precio: 0,
    stock: 0,
    categoria: '',
    imagen: '',
    marca: '',
    estado: 'disponible' as ProductoEstado
  })
}

function openCreateModal() {
  editingId.value = null
  resetForm()
  formOpen.value = true
}

function openEditModal(producto: Producto) {
  editingId.value = producto.id
  Object.assign(state, {
    nombre: producto.nombre,
    descripcion: producto.descripcion,
    precio: producto.precio,
    stock: producto.stock,
    categoria: producto.categoria,
    imagen: producto.imagen,
    marca: producto.marca || '',
    estado: producto.estado
  })
  formOpen.value = true
}

// 7. Formulario Submit (Crear / Editar)
async function onSubmit(event: FormSubmitEvent<Schema>) {
  const producto = event.data

  if (editingId.value !== null) {
    productos.value = productos.value.map(item => item.id === editingId.value
      ? { ...item, ...producto }
      : item)
    toast.add({ title: 'Producto actualizado', description: `${producto.nombre} fue actualizado.`, color: 'success' })
  } else {
    const nextId = Math.max(5000, ...productos.value.map(item => item.id)) + 1

    productos.value = [{
      id: nextId,
      ...producto
    }, ...productos.value]
    toast.add({ title: 'Producto creado', description: `${producto.nombre} fue agregado.`, color: 'success' })
  }

  formOpen.value = false
  resetForm()
}

function selectedProductoIds() {
  return table.value?.tableApi?.getFilteredSelectedRowModel().rows.map((row: Row<Producto>) => row.original.id) ?? []
}

function openDeleteSelected() {
  const ids = selectedProductoIds()
  if (!ids.length) return
  deleteIds.value = ids
  deleteOpen.value = true
}

function openDeleteProducto(producto: Producto) {
  deleteIds.value = [producto.id]
  deleteOpen.value = true
}

function deleteProductos() {
  const ids = new Set(deleteIds.value)
  const total = ids.size

  productos.value = productos.value.filter(p => !ids.has(p.id))
  deleteIds.value = []
  rowSelection.value = {}
  deleteOpen.value = false
  toast.add({
    title: total === 1 ? 'Producto eliminado' : 'Productos eliminados',
    description: total === 1 ? 'El registro fue eliminado.' : `${total} registros fueron eliminados.`,
    color: 'success'
  })
}

function getRowItems(row: Row<Producto>) {
  return [
    { type: 'label' as const, label: row.original.nombre },
    { label: 'Editar', icon: 'i-lucide-pencil', onSelect() { openEditModal(row.original) } },
    { type: 'separator' as const },
    { label: 'Eliminar', icon: 'i-lucide-trash', color: 'error' as const, onSelect() { openDeleteProducto(row.original) } }
  ]
}

// 8. Definición de Columnas de la Tabla
const columns: TableColumn<Producto>[] = [
  {
    id: 'select',
    header: ({ table }) => h(UCheckbox, {
      'modelValue': table.getIsSomePageRowsSelected() ? 'indeterminate' : table.getIsAllPageRowsSelected(),
      'onUpdate:modelValue': (value: boolean | 'indeterminate') => table.toggleAllPageRowsSelected(!!value),
      'ariaLabel': 'Seleccionar todos'
    }),
    cell: ({ row }) => h(UCheckbox, {
      'modelValue': row.getIsSelected(),
      'onUpdate:modelValue': (value: boolean | 'indeterminate') => row.toggleSelected(!!value),
      'ariaLabel': 'Seleccionar producto'
    })
  },
  { accessorKey: 'id', header: 'ID' },
  {
    accessorKey: 'nombre',
    header: ({ column }) => {
      const isSorted = column.getIsSorted()
      return h(UButton, {
        color: 'neutral',
        variant: 'ghost',
        label: 'Producto',
        icon: isSorted ? (isSorted === 'asc' ? 'i-lucide-arrow-up-narrow-wide' : 'i-lucide-arrow-down-wide-narrow') : 'i-lucide-arrow-up-down',
        class: '-mx-2.5',
        onClick: () => column.toggleSorting(column.getIsSorted() === 'asc')
      })
    },
    cell: ({ row }) => h('div', { class: 'flex items-center gap-3' }, [
      h(UAvatar, { src: row.original.imagen, alt: row.original.nombre, size: 'lg', icon: 'i-lucide-box' }),
      h('div', undefined, [
        h('p', { class: 'font-medium text-highlighted' }, row.original.nombre),
        h('p', { class: 'text-sm text-muted max-w-xs truncate' }, row.original.descripcion)
      ])
    ])
  },
  { accessorKey: 'categoria', header: 'Categoría' },
  {
    accessorKey: 'precio',
    header: 'Precio',
    cell: ({ row }) => `$${row.original.precio.toFixed(2)}`
  },
  { accessorKey: 'stock', header: 'Stock' },
  {
    accessorKey: 'estado',
    header: 'Estado',
    cell: ({ row }) => h(UBadge, {
      variant: 'subtle',
      color: estadoColors[row.original.estado]
    }, () => getEstadoLabel(row.original.estado))
  },
  {
    id: 'actions',
    cell: ({ row }) => h('div', { class: 'text-right' }, h(UDropdownMenu, {
      content: { align: 'end' },
      items: getRowItems(row)
    }, () => h(UButton, { icon: 'i-lucide-ellipsis-vertical', color: 'neutral', variant: 'ghost', class: 'ml-auto' })))
  }
]
</script>

<template>
  <UDashboardPanel id="productos">
    <template #header>
      <UDashboardNavbar title="Productos" :ui="{ right: 'gap-3' }">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>
        <template #right>
          <UButton label="Nuevo producto" icon="i-lucide-plus" @click="openCreateModal" />
        </template>
      </UDashboardNavbar>
    </template>

    <template #body>
      <UPageGrid class="lg:grid-cols-4 gap-4">
        <UPageCard
          v-for="item in resumen"
          :key="item.label"
          :icon="item.icon"
          :title="item.label"
          variant="subtle"
          :ui="{ leading: item.leadingClass, title: 'font-normal text-muted text-xs uppercase' }"
        >
          <span class="text-2xl font-semibold text-highlighted">{{ item.value }}</span>
        </UPageCard>
      </UPageGrid>

      <div class="flex flex-col gap-3 lg:flex-row lg:items-center lg:justify-between">
        <UInput
          v-model="search"
          icon="i-lucide-search"
          placeholder="Buscar por nombre, descripción o categoría..."
          class="w-full lg:max-w-md"
        />

        <div class="flex flex-wrap items-center gap-1.5">
          <UButton
            v-if="table?.tableApi?.getFilteredSelectedRowModel().rows.length"
            color="error"
            variant="subtle"
            icon="i-lucide-trash"
            :label="`Eliminar (${table?.tableApi?.getFilteredSelectedRowModel().rows.length})`"
            @click="openDeleteSelected"
          />

          <USelect
            v-model="estadoFiltro"
            :items="estadoFiltroItems"
            class="min-w-36"
          />
        </div>
      </div>

      <UTable
        ref="table"
        v-model:column-visibility="columnVisibility"
        v-model:row-selection="rowSelection"
        v-model:pagination="pagination"
        :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
        :data="productosFiltrados"
        :columns="columns"
        class="shrink-0"
        :ui="{
          base: 'table-fixed border-separate border-spacing-0',
          thead: '[&>tr]:bg-elevated/50 [&>tr]:after:content-none',
          tbody: '[&>tr]:last:[&>td]:border-b-0',
          th: 'py-2 first:rounded-l-lg last:rounded-r-lg border-y border-default first:border-l last:border-r',
          td: 'border-b border-default'
        }"
      />

      <UAlert
        v-if="!productosFiltrados.length"
        color="neutral"
        variant="subtle"
        icon="i-lucide-search-x"
        title="No se encontraron productos"
        description="Ajusta la búsqueda o agrega un nuevo producto."
      />

      <div class="flex flex-col gap-3 border-t border-default pt-4 mt-auto sm:flex-row sm:items-center sm:justify-between">
        <div class="text-sm text-muted">
          {{ table?.tableApi?.getFilteredSelectedRowModel().rows.length || 0 }} seleccionados de
          {{ productosFiltrados.length }} producto{{ productosFiltrados.length === 1 ? '' : 's' }}.
        </div>

        <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="productosFiltrados.length"
          @update:page="(p: number) => table?.tableApi?.setPageIndex(p - 1)"
        />
      </div>
    </template>
  </UDashboardPanel>

  <UModal v-model:open="formOpen" :title="modalTitle" description="Gestiona la información del catálogo de productos.">
    <template #body>
      <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        
        <UFormField label="Nombre del Producto" name="nombre">
          <UInput v-model="state.nombre" class="w-full" placeholder="Ej. Monitor Pro 27'" />
        </UFormField>

        <UFormField label="Descripción" name="descripcion">
          <UInput v-model="state.descripcion" class="w-full" placeholder="Breve descripción del producto" />
        </UFormField>

        <div class="grid gap-4 sm:grid-cols-2">
          <UFormField label="Precio ($)" name="precio">
            <UInput v-model.number="state.precio" type="number" step="0.01" class="w-full" />
          </UFormField>

          <UFormField label="Stock disponible" name="stock">
            <UInput v-model.number="state.stock" type="number" class="w-full" />
          </UFormField>
        </div>

        <div class="grid gap-4 sm:grid-cols-2">
          <UFormField label="Categoría" name="categoria">
            <UInput v-model="state.categoria" class="w-full" placeholder="Ej. Electrónica" />
          </UFormField>

          <UFormField label="Marca (Opcional)" name="marca">
            <UInput v-model="state.marca" class="w-full" placeholder="Ej. Sony" />
          </UFormField>
        </div>

        <UFormField label="URL de la Imagen" name="imagen">
          <UInput v-model="state.imagen" class="w-full" placeholder="https://ejemplo.com/imagen.jpg" />
        </UFormField>

        <UFormField label="Estado del Stock" name="estado">
          <USelect v-model="state.estado" :items="estadoItems" class="w-full" />
        </UFormField>

        <div class="flex justify-end gap-2 pt-2">
          <UButton label="Cancelar" color="neutral" variant="subtle" type="button" @click="formOpen = false" />
          <UButton :label="submitLabel" color="primary" variant="solid" type="submit" />
        </div>
      </UForm>
    </template>
  </UModal>

  <UModal v-model:open="deleteOpen" :title="deleteTitle" :description="deleteDescription">
    <template #body>
      <div class="flex justify-end gap-2">
        <UButton label="Cancelar" color="neutral" variant="subtle" @click="deleteOpen = false" />
        <UButton label="Eliminar" color="error" variant="solid" @click="deleteProductos" />
      </div>
    </template>
  </UModal>
</template>

