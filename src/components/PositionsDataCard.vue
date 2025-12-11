<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'

interface PositionData {
  metadata: {
    source_file: string
    source_type: string
    total_positions: number
    generated_date: string
    position_codes: number[]
  }
  positions: Record<string, any>
}

const data = ref<PositionData | null>(null)
const loading = ref(true)
const error = ref<string | null>(null)

onMounted(async () => {
  try {
    const response = await fetch('/output-processed.json')
    if (!response.ok) {
      throw new Error('Failed to load positions data')
    }
    data.value = await response.json()
    loading.value = false
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unknown error occurred'
    loading.value = false
  }
})

const formatDate = (dateString: string) => {
  try {
    const date = new Date(dateString)
    return date.toLocaleDateString('es-ES', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  } catch {
    return dateString
  }
}
</script>

<template>
  <Card>
    <CardHeader>
      <CardTitle>Manual de Puestos</CardTitle>
      <CardDescription>Datos del archivo output-processed.json</CardDescription>
    </CardHeader>
    <CardContent>
      <div v-if="loading" class="text-center py-4">
        <p class="text-muted-foreground">Cargando datos...</p>
      </div>
      
      <div v-else-if="error" class="text-center py-4">
        <p class="text-destructive">Error: {{ error }}</p>
      </div>
      
      <div v-else-if="data" class="space-y-4">
        <div class="grid grid-cols-2 gap-4">
          <div>
            <p class="text-sm text-muted-foreground">Total de Puestos</p>
            <p class="text-2xl font-bold">{{ data.metadata.total_positions }}</p>
          </div>
          <div>
            <p class="text-sm text-muted-foreground">Archivo Fuente</p>
            <p class="text-sm font-medium truncate">{{ data.metadata.source_file }}</p>
          </div>
        </div>
        
        <div>
          <p class="text-sm text-muted-foreground mb-2">Fecha de Generación</p>
          <p class="text-sm">{{ formatDate(data.metadata.generated_date) }}</p>
        </div>
        
        <div>
          <p class="text-sm text-muted-foreground mb-2">Tipo de Fuente</p>
          <Badge variant="outline">{{ data.metadata.source_type.toUpperCase() }}</Badge>
        </div>
        
        <div>
          <p class="text-sm text-muted-foreground mb-2">Códigos de Puestos</p>
          <p class="text-xs text-muted-foreground">
            {{ data.metadata.position_codes.length }} códigos disponibles
          </p>
          <div class="mt-2 max-h-32 overflow-y-auto">
            <div class="flex flex-wrap gap-1">
              <Badge 
                v-for="code in data.metadata.position_codes.slice(0, 20)" 
                :key="code"
                variant="secondary"
                class="text-xs"
              >
                {{ code }}
              </Badge>
              <span v-if="data.metadata.position_codes.length > 20" class="text-xs text-muted-foreground">
                +{{ data.metadata.position_codes.length - 20 }} más...
              </span>
            </div>
          </div>
        </div>
      </div>
    </CardContent>
  </Card>
</template>

