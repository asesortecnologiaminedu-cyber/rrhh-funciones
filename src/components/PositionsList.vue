<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import PositionCard from './PositionCard.vue'
import { Input } from '@/components/ui/input'

interface Position {
  header: {
    form_number: string
    title: string
    ministry: string
    plan_type: string
    sap_code: string
    version: string
  }
  identificacion: {
    codigo: string
    categoria: string
    nivel: string
    denominacion_puesto: string
    unidad_organizacional: string
    jefe_inmediato_superior: string
    superior_jerarquico: string
    supervisa_a: string[]
  }
  descripcion: {
    objetivo_del_puesto: string
    funciones: any[]
    resultados: any[]
    normas_a_cumplir: any[]
    cualidades_requeridas: Record<string, any>
    formacion_academica: string
    formacion_adicional: any[]
    experiencia_general: string
    experiencia_especifica: string
  }
}

interface PositionData {
  metadata: {
    source_file: string
    source_type: string
    total_positions: number
    generated_date: string
    position_codes: number[]
  }
  positions: Record<string, Position>
}

const positions = ref<Array<{ code: string; data: Position }>>([])
const loading = ref(true)
const error = ref<string | null>(null)
const searchQuery = ref('')

const normalizeText = (text: string) => {
  return text.normalize("NFD").replace(/[\u0300-\u036f]/g, "").toLowerCase();
}

const filteredPositions = computed(() => {
  if (!searchQuery.value.trim()) return positions.value

  const query = normalizeText(searchQuery.value)
  return positions.value.filter(({ code, data }) => {
    return (
      normalizeText(code).includes(query) ||
      normalizeText(data.identificacion.denominacion_puesto).includes(query) ||
      normalizeText(data.identificacion.unidad_organizacional).includes(query) ||
      normalizeText(data.header.form_number).includes(query)
    )
  })
})

onMounted(async () => {
  try {
    const response = await fetch('/output-processed.json')
    if (!response.ok) {
      throw new Error('Failed to load positions data')
    }
    const data: PositionData = await response.json()
    
    // Convert positions object to array
    positions.value = Object.entries(data.positions).map(([code, position]) => ({
      code,
      data: position
    }))
    
    loading.value = false
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unknown error occurred'
    loading.value = false
  }
})
</script>

<template>
  <div class="positions-container flex flex-col gap-4">
    <div class="flex items-center gap-2 max-w-md">
      <Input
        v-model="searchQuery"
        placeholder="Buscar por puesto, código, unidad..."
        class="h-9"
      />
    </div>

    <div v-if="loading" class="loading-state">
      <p class="text-muted-foreground">Cargando puestos...</p>
    </div>

    <div v-else-if="error" class="error-state">
      <p class="text-destructive">Error: {{ error }}</p>
    </div>

    <div v-else class="positions-grid">
      <PositionCard
        v-for="position in filteredPositions"
        :key="position.code"
        :position="position.data"
        :code="position.code"
      />
      <div v-if="filteredPositions.length === 0" class="text-center text-muted-foreground py-8">
        No se encontraron resultados para "{{ searchQuery }}"
      </div>
    </div>
  </div>
</template>

<style scoped>
.positions-container {
  width: 100%;
}

.loading-state,
.error-state {
  text-align: center;
  padding: 3rem;
}

.positions-grid {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  max-width: 100%;
}
</style>

