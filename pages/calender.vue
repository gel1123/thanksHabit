<script setup>
import { DateTime } from "luxon";
import { getDocs, collection } from 'firebase/firestore'

const { $firebaseDB } = useNuxtApp();

// モーダル
const showModal = ref(false)
const modalContent = ref('')

const openModal = (count) => {
  //　実際は日付を渡してデータを取得して表示
  modalContent.value = count
  showModal.value = true
}

const closeModal = () => showModal.value = false

// カレンダー関係
const currentDate = ref(DateTime.local());
const displayDate = computed(() => currentDate.value.toFormat('yyyy年M月'))
const currentMonth = computed(() => currentDate.value.toFormat('yyyy-MM'))
const youbi = ["日", "月", "火", "水", "木", "金", "土"]

const startDate = computed(() => {
  const newDt = currentDate.value.startOf("month")
  const youbiNum = newDt.weekday
  return newDt.minus({
    days: youbiNum
  })
});

const endDate = computed(() => {
  const endDt = currentDate.value.endOf("month")
  let youbiNum = endDt.weekday
  youbiNum == 7 ? youbiNum = 0 : youbiNum = youbiNum
  
  return endDt.plus({
    days: 6 - youbiNum
  })
});

const calenders = computed(() => {
  const weekNumber = Math.ceil(endDate.value.diff(startDate.value, 'days').days / 7);
  let date = startDate.value

  let calendars = [];
  for (let week = 0; week < weekNumber; week++) {
    let weekRow = [];
    for (let day = 0; day < 7; day++) {
      const thanksCount = getDayThanksFirebase(date.toFormat('yyyy-MM'))
      weekRow.push({
        date: date.day,
        month: date.toFormat('yyyy-MM'),
        count: thanksCount
      });
      date = date.plus({days: 1})
    }
    calendars.push(weekRow)
  }
  // console.log(calendars);
  return calendars;
});

// const events = [
//   { date: "2022-08-02", count: 12},
//   { date: "2022-08-03", count: 3},
//   { date: "2022-08-04", count: 4},
// ]

// const getDayThanks = (date) => {
//   return events.find( event => {
//     return date.toFormat('yyyy-MM-dd') == event.date
//   })
// }

const getDayThanksFirebase = (date) => {
  const collectionRef = collection($firebaseDB, date)
  //console.log(collectionRef)
  getDocs(collectionRef).then((snap) => {
    return snap.size
  })
}

const prevMonth = () => currentDate.value = currentDate.value.minus({ months: 1})
const nextMonth = () => currentDate.value = currentDate.value.plus({ months: 1})
</script>

<template>
<div>
  <modal 
    :show-content="showModal"
    @closeModal="closeModal"  
  >
    {{ modalContent }}
  </modal>

  <h2>カレンダー 
    {{ displayDate }}
  </h2>
  <button @click="prevMonth">前の月</button>
  <button @click="nextMonth">次の月</button>

  <div 
    style="margin:auto;max-width:900px;"
    class="border-t border-gray-300 border-solid"
  >
    <div class="flex text-center border-l border-gray-300 border-solid">
      <div 
        v-for="n in 7" 
        :key="n"
        style="flex:1"
        class="border-r border-gray-300 border-solid"
      >
        {{ youbi[n-1] }}
      </div>
    </div>
    <div 
      v-for="(week,index) in calenders" 
      :key="index" 
      class="flex border-l border-gray-300 border-solid"
    >
      <div 
        v-for="(day, index) in week" 
        :key="index"
        style="flex:1;min-height:100px;"
        :class="`${ 
          currentMonth != day.month ? 'bg-gray-100' : ''
        } border-r border-b border-gray-300 border-solid text-center`"
      >
        {{ day.date }}
        <p 
          class="mt-2 text-6xl font-bold text-red-300"
          @click="openModal(day.count)"
        >
          {{ day.count }}
        </p>

      </div>
    </div>
  </div>
</div>
</template>