<template>
  <main class="page">
    <h1>景点探索</h1>
    <div class="toolbar">
      <el-input v-model="spotStore.keyword" placeholder="搜索景点、标签" style="max-width: 260px" />
      <CategoryFilter v-model="spotStore.category" />
    </div>
    <EmptyState v-if="!spotStore.filteredSpots.length" title="没有景点" :description="messages.emptySpots" />
    <section class="grid">
      <SpotCard v-for="spot in spotStore.filteredSpots" :key="spot.id" :spot="spot" @favorite="spotStore.toggleFavorite" @add="openDialog" />
    </section>
    <AddToTripDialog ref="dialogRef" @confirm="handleConfirm" />
  </main>
</template>
<script setup lang="ts">
import { ref } from 'vue';
import { useDayPlanStore } from '../stores/dayPlanStore';
import { useSpotStore } from '../stores/spotStore';
import CategoryFilter from '../components/common/CategoryFilter.vue';
import SpotCard from '../components/common/SpotCard.vue';
import EmptyState from '../components/common/EmptyState.vue';
import AddToTripDialog from '../components/common/AddToTripDialog.vue';
import { messages } from '../constants/messages';
const dayPlanStore = useDayPlanStore();
const spotStore = useSpotStore();
const dialogRef = ref<InstanceType<typeof AddToTripDialog>>();
function openDialog(spotId: string) {
  dialogRef.value?.open(spotId);
}
function handleConfirm(tripId: string, spotId: string, dayIdx: number, date: string) {
  dayPlanStore.addSpot(tripId, spotId, dayIdx, date);
}
</script>
