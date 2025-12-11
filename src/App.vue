<script setup lang="ts">
import { SidebarProvider, SidebarInset } from '@/components/ui/sidebar'
import AppSidebar from '@/components/AppSidebar.vue'
import SiteHeader from '@/components/SiteHeader.vue'
import SectionCards from '@/components/SectionCards.vue'
import ChartAreaInteractive from '@/components/ChartAreaInteractive.vue'
import PositionsDataCard from '@/components/PositionsDataCard.vue'
import PositionsList from '@/components/PositionsList.vue'
</script>

<template>
  <div class="min-h-screen bg-background">
    <SidebarProvider>
      <AppSidebar />
      <SidebarInset>
        <SiteHeader />
        <div class="flex flex-1 flex-col gap-4 p-4 pt-0">
          <!-- <SectionCards /> -->
          <div class="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
            <PositionsDataCard />
          </div>
          <div class="p-4 lg:p-6">
            <!-- ChartAreaInteractive  -->
            <PositionsList />
          </div>
        </div>
      </SidebarInset>
    </SidebarProvider>
  </div>
</template>
