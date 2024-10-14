<template>
  <div class="calendar">
    <div class="calendar-header">
      <h1>{{ currentYear }}年{{ currentMonth }}月</h1>
      <div class="calendar-actions">
        <button class="icon-button" @click="changeMonth(-1)">
          <ChevronLeftIcon />
        </button>
        <button class="icon-button" @click="resetToToday">
          <MapPin />
          <!-- <RefreshCwIcon /> -->
        </button>
        <button class="icon-button" @click="changeMonth(1)">
          <ChevronRightIcon />
        </button>
        <button class="icon-button" @click="toggleView">
          {{ isWeekView ? 'Month' : 'Week' }}
        </button>
      </div>
    </div>
    <div class="calendar-weekdays">
      <div v-for="day in weekdays" :key="day" class="weekday">{{ day }}</div>
    </div>
    <div class="calendar-days">
      <div
        v-for="date in displayDates"
        :key="`${date.fullDate}-${viewKey}`"
        :class="[
          'date',
          {
            current: date.isToday,
            weekend: date.isWeekend,
            'different-month': !date.isCurrentMonth,
          },
        ]"
      >
        <span class="day-number">{{ date.day }}</span>
        <span class="day-name">{{ date.name }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import {
  startOfMonth,
  endOfMonth,
  startOfWeek,
  endOfWeek,
  eachDayOfInterval,
  format,
  isToday,
  isWeekend,
  isSameMonth,
  addMonths,
  addDays,
  getYear,
  getMonth,
} from 'date-fns';
import { zhCN } from 'date-fns/locale';
import {
  RefreshCwIcon,
  ChevronLeftIcon,
  MapPin,
  ChevronRightIcon,
} from 'lucide-vue-next';

const currentDate = ref(new Date());
const weekdays = ['日', '一', '二', '三', '四', '五', '六'];
const isWeekView = ref(false);
const viewKey = ref(0);

const currentYear = computed(() => getYear(currentDate.value));
const currentMonth = computed(() => getMonth(currentDate.value) + 1);

const displayDates = computed(() => {
  let start, end;
  if (isWeekView.value) {
    start = startOfWeek(currentDate.value);
    end = addDays(start, 6);
  } else {
    start = startOfWeek(startOfMonth(currentDate.value));
    end = endOfWeek(endOfMonth(currentDate.value));
  }
  return eachDayOfInterval({ start, end }).map((date) => ({
    fullDate: format(date, 'yyyy-MM-dd'),
    day: format(date, 'd'),
    name: format(date, 'EEE', { locale: zhCN }),
    isToday: isToday(date),
    isWeekend: isWeekend(date),
    isCurrentMonth: isSameMonth(date, currentDate.value),
  }));
});

const changeMonth = (amount) => {
  currentDate.value = addMonths(currentDate.value, amount);
  viewKey.value++; // Force re-render
};

const resetToToday = () => {
  currentDate.value = new Date();
  viewKey.value++; // Force re-render
};

const toggleView = () => {
  isWeekView.value = !isWeekView.value;
  viewKey.value++; // Force re-render
};
</script>

<style scoped>
.calendar {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica,
    Arial, sans-serif;
  width: 450px;
  margin: 0 auto;
  padding: 20px;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.calendar-header h1 {
  font-size: 20px;
  font-weight: bold;
}

.calendar-actions {
  display: flex;
  gap: 10px;
}

.icon-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 5px;
}

.calendar-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
  font-weight: bold;
  margin-bottom: 10px;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
}

.date {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 60px;
  border-radius: 6px;
  cursor: pointer;
}

.current {
  background-color: #ebf1ff;
  color: #0c53f7;
}

.weekend {
  color: #ff0000;
}

.different-month {
  opacity: 0.5;
}

.day-number {
  font-size: 18px;
  font-weight: bold;
}

.day-name {
  font-size: 12px;
  margin-top: 2px;
}
</style>
