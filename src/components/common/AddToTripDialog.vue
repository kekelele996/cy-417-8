<template>
  <el-dialog v-model="visible" title="加入行程" width="420px" @close="reset">
    <div v-if="tripStore.trips.length" class="form-block">
      <label>选择旅行</label>
      <el-select v-model="tripId" placeholder="请选择旅行" style="width: 100%">
        <el-option v-for="trip in tripStore.trips" :key="trip.id" :label="trip.title" :value="trip.id">
          <span>{{ trip.title }}</span>
          <span class="muted" style="margin-left: 8px">{{ trip.destination }} · {{ formatDate(trip.start_date) }} ~ {{ formatDate(trip.end_date) }}</span>
        </el-option>
      </el-select>
    </div>
    <EmptyState v-else title="暂无旅行" description="请先创建一趟旅行" />
    <div v-if="dayOptions.length" class="form-block">
      <label>选择日期</label>
      <el-select v-model="dayIndex" placeholder="请选择日期" style="width: 100%">
        <el-option v-for="d in dayOptions" :key="d.index" :label="'第 ' + d.index + ' 天 · ' + d.date" :value="d.index" />
      </el-select>
    </div>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" :disabled="!tripId || !dayIndex" @click="confirm">确认加入</el-button>
    </template>
  </el-dialog>
</template>
<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import dayjs from 'dayjs';
import { useTripStore } from '../../stores/tripStore';
import EmptyState from './EmptyState.vue';
import { formatDate } from '../../utils/formatters';

const tripStore = useTripStore();

const visible = ref(false);
const tripId = ref('');
const dayIndex = ref(1);
const spotId = ref('');

const emit = defineEmits<{ confirm: [tripId: string, spotId: string, dayIndex: number, date: string] }>();

const dayOptions = computed(() => {
  const trip = tripStore.trips.find((t) => t.id === tripId.value);
  if (!trip) return [];
  const start = dayjs(trip.start_date);
  const end = dayjs(trip.end_date);
  const days = end.diff(start, 'day') + 1;
  const result: { index: number; date: string }[] = [];
  for (let i = 0; i < days; i++) {
    result.push({ index: i + 1, date: start.add(i, 'day').format('YYYY-MM-DD') });
  }
  return result;
});

const selectedDate = computed(() => dayOptions.value.find((d) => d.index === dayIndex.value)?.date || '');

watch(tripId, () => {
  dayIndex.value = 1;
});

function open(spot: string) {
  spotId.value = spot;
  tripId.value = tripStore.trips[0]?.id || '';
  visible.value = true;
}

function confirm() {
  emit('confirm', tripId.value, spotId.value, dayIndex.value, selectedDate.value);
  visible.value = false;
}

function reset() {
  tripId.value = '';
  dayIndex.value = 1;
  spotId.value = '';
}

defineExpose({ open });
</script>
<style scoped>
.form-block { margin-bottom: 16px; }
.form-block label { display: block; font-weight: 600; margin-bottom: 6px; }
</style>
