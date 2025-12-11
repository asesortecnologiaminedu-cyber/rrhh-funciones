<script setup lang="ts">
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Separator } from '@/components/ui/separator'

// Helper function to check if text is valid/corrupted
const isValidText = (text: string): boolean => {
  if (!text || typeof text !== 'string') return false
  // Filter out corrupted text patterns
  const corruptedPatterns = [
    /\u0000/, // null characters
    /DIRE\/ION/, // corrupted pattern
    /STRATIvn/, // corrupted pattern
    /[^\w\s\u00C0-\u017F.,;:!?()\-'"]{3,}/, // too many special characters
  ]
  return !corruptedPatterns.some(pattern => pattern.test(text))
}

// Helper function to clean text
const cleanText = (text: string): string => {
  if (!text || typeof text !== 'string') return ''
  // Remove null characters and other control characters
  return text.replace(/\u0000/g, '').trim()
}

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

const props = defineProps<{
  position: Position
  code: string
}>()
</script>

<template>
  <Card class="position-card">
    <CardHeader>
      <div class="flex items-start justify-between mb-2">
        <div>
          <CardTitle class="text-xl mb-1">{{ position.identificacion.denominacion_puesto }}</CardTitle>
          <CardDescription class="text-xs">
            Código: {{ position.identificacion.codigo }} | 
            {{ position.header.ministry }}
          </CardDescription>
        </div>
        <div class="flex gap-2">
          <Badge variant="outline">{{ position.identificacion.categoria }}</Badge>
          <Badge variant="secondary">{{ position.identificacion.nivel }}</Badge>
        </div>
      </div>
      <div class="text-xs text-muted-foreground space-y-1">
        <p><strong>Formulario:</strong> {{ position.header.form_number }} | <strong>Versión:</strong> {{ position.header.version }}</p>
        <p><strong>SAP:</strong> {{ position.header.sap_code }}</p>
        <p>{{ position.header.plan_type }}</p>
      </div>
    </CardHeader>
    
    <CardContent class="space-y-4">
      <!-- IDENTIFICACIÓN -->
      <div>
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">IDENTIFICACIÓN</h4>
        <Separator class="mb-3" />
        <div class="space-y-2 text-sm">
          <div>
            <span class="font-medium text-muted-foreground">Unidad Organizacional:</span>
            <p class="mt-1">{{ position.identificacion.unidad_organizacional }}</p>
          </div>
          <div v-if="position.identificacion.jefe_inmediato_superior">
            <span class="font-medium text-muted-foreground">Jefe Inmediato Superior:</span>
            <p class="mt-1">{{ position.identificacion.jefe_inmediato_superior }}</p>
          </div>
          <div v-if="position.identificacion.superior_jerarquico">
            <span class="font-medium text-muted-foreground">Superior Jerárquico:</span>
            <p class="mt-1">{{ position.identificacion.superior_jerarquico }}</p>
          </div>
          <div v-if="position.identificacion.supervisa_a && position.identificacion.supervisa_a.length > 0">
            <span class="font-medium text-muted-foreground">Supervisa a:</span>
            <ul class="mt-1 list-disc list-inside">
              <li v-for="(item, idx) in position.identificacion.supervisa_a" :key="idx">{{ item }}</li>
            </ul>
          </div>
        </div>
      </div>

      <!-- OBJETIVO DEL PUESTO -->
      <div v-if="position.descripcion.objetivo_del_puesto">
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">OBJETIVO DEL PUESTO</h4>
        <Separator class="mb-3" />
        <p class="text-sm">{{ position.descripcion.objetivo_del_puesto }}</p>
      </div>

      <!-- FUNCIONES -->
      <div v-if="position.descripcion.funciones && position.descripcion.funciones.length > 0">
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">FUNCIONES</h4>
        <Separator class="mb-3" />
        <ul class="space-y-2 text-sm">
          <li v-for="(funcion, idx) in position.descripcion.funciones" :key="idx" class="flex items-start">
            <span class="mr-2 text-muted-foreground">{{ typeof funcion === 'object' && funcion.numero ? funcion.numero : idx + 1 }}.</span>
            <span>{{ 
              typeof funcion === 'string' 
                ? funcion 
                : funcion.descripcion || funcion.text || (typeof funcion === 'object' ? JSON.stringify(funcion) : funcion)
            }}</span>
          </li>
        </ul>
      </div>

      <!-- RESULTADOS -->
      <div v-if="position.descripcion.resultados && position.descripcion.resultados.length > 0">
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">RESULTADOS</h4>
        <Separator class="mb-3" />
        <ul class="space-y-2 text-sm">
          <li v-for="(resultado, idx) in position.descripcion.resultados" :key="idx" class="flex items-start">
            <span class="mr-2 text-muted-foreground">{{ typeof resultado === 'object' && resultado.numero ? resultado.numero : idx + 1 }}.</span>
            <span>{{ 
              typeof resultado === 'string' 
                ? resultado 
                : resultado.descripcion || resultado.text || (typeof resultado === 'object' ? JSON.stringify(resultado) : resultado)
            }}</span>
          </li>
        </ul>
      </div>

      <!-- CUALIDADES REQUERIDAS -->
      <div v-if="position.descripcion.cualidades_requeridas && Object.keys(position.descripcion.cualidades_requeridas).length > 0">
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">CUALIDADES REQUERIDAS</h4>
        <Separator class="mb-3" />
        <div class="space-y-3 text-sm">
          <div v-for="(values, key) in position.descripcion.cualidades_requeridas" :key="key">
            <template v-if="(Array.isArray(values) ? values : [values]).some(v => {
              const text = typeof v === 'string' ? v : JSON.stringify(v)
              return isValidText(text) && cleanText(text).length > 0
            })">
              <p class="font-medium mb-1">{{ key }}:</p>
              <ul class="list-disc list-inside space-y-1 ml-2">
                <li v-for="(value, idx) in (Array.isArray(values) ? values : [values]).filter(v => {
                  const text = typeof v === 'string' ? v : JSON.stringify(v)
                  return isValidText(text) && cleanText(text).length > 0
                })" :key="idx">
                  {{ cleanText(typeof value === 'string' ? value : JSON.stringify(value)) }}
                </li>
              </ul>
            </template>
          </div>
        </div>
      </div>

      <!-- NORMAS A CUMPLIR -->
      <div v-if="position.descripcion.normas_a_cumplir && position.descripcion.normas_a_cumplir.length > 0">
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">NORMAS A CUMPLIR</h4>
        <Separator class="mb-3" />
        <ul class="space-y-2 text-sm">
          <li v-for="(norma, idx) in position.descripcion.normas_a_cumplir" :key="idx" class="flex items-start">
            <span class="mr-2 text-muted-foreground">{{ typeof norma === 'object' && norma.numero ? norma.numero : idx + 1 }}.</span>
            <span>{{ 
              typeof norma === 'string' 
                ? norma 
                : norma.descripcion || norma.text || (typeof norma === 'object' ? JSON.stringify(norma) : norma)
            }}</span>
          </li>
        </ul>
      </div>

      <!-- FORMACIÓN Y EXPERIENCIA -->
      <div>
        <h4 class="text-sm font-semibold mb-2 uppercase tracking-wide">REQUISITOS</h4>
        <Separator class="mb-3" />
        <div class="space-y-3 text-sm">
          <div v-if="position.descripcion.formacion_academica">
            <span class="font-medium text-muted-foreground">Formación Académica:</span>
            <p class="mt-1">{{ position.descripcion.formacion_academica }}</p>
          </div>
          <div v-if="position.descripcion.formacion_adicional && position.descripcion.formacion_adicional.length > 0">
            <span class="font-medium text-muted-foreground">Formación Adicional:</span>
            <ul class="mt-1 list-disc list-inside">
              <li v-for="(formacion, idx) in position.descripcion.formacion_adicional" :key="idx">
                {{ typeof formacion === 'string' ? formacion : JSON.stringify(formacion) }}
              </li>
            </ul>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div v-if="position.descripcion.experiencia_general">
              <span class="font-medium text-muted-foreground">Experiencia General:</span>
              <p class="mt-1">{{ position.descripcion.experiencia_general }}</p>
            </div>
            <div v-if="position.descripcion.experiencia_especifica">
              <span class="font-medium text-muted-foreground">Experiencia Específica:</span>
              <p class="mt-1">{{ position.descripcion.experiencia_especifica }}</p>
            </div>
          </div>
        </div>
      </div>
    </CardContent>
  </Card>
</template>

<style scoped>
.position-card {
  transition: transform 0.2s, box-shadow 0.2s;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.position-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>

