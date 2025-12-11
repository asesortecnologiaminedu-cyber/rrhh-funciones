<script setup lang="ts">
import AppSidebar from '@/components/AppSidebar.vue'
import ChartAreaInteractive from '@/components/ChartAreaInteractive.vue'
import SectionCards from '@/components/SectionCards.vue'
import SiteHeader from '@/components/SiteHeader.vue'
import { SidebarInset, SidebarProvider } from '@/components/ui/sidebar'
</script>

<template>
  <SidebarProvider>
    <AppSidebar />
    <SidebarInset>
      <SiteHeader />
      <div class="flex flex-1 flex-col gap-4 p-4 pt-0">
        <div class="grid auto-rows-min gap-4 md:grid-cols-3">
          <div class="aspect-video rounded-xl bg-muted/50" />
          <div class="aspect-video rounded-xl bg-muted/50" />
          <div class="aspect-video rounded-xl bg-muted/50" />
        </div>
        <div class="min-h-[100vh] flex-1 rounded-xl bg-muted/50 md:min-h-min">
          <SectionCards />
          <div class="p-4 lg:p-6">
            <ChartAreaInteractive />
          </div>
        </div>
      </div>
    </SidebarInset>
  </SidebarProvider>
</template>

