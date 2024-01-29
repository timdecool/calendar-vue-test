<script setup>
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import interactionPlugin from '@fullcalendar/interaction'
import timeGridPlugin from '@fullcalendar/timegrid'
import frLocale from '@fullcalendar/core/locales/fr'

import axios from 'axios'
import SearchBar from '@/components/SearchBar.vue'
import { ref } from 'vue'

const posts = ref([])
// axios.get('http://127.0.0.1:8002/api/posts')
//   .then(function (response) {
//     console.log(response.data["hydra:member"])
//   })
//   .catch(function(error) {
//     console.log(error)
//   })

const availView = ref(false)

const calendarOptions = ref({
  plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
  initialView: 'timeGridWeek',
  titleFormat: {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  },
  scrollTime: Date.now(),
  locale: frLocale,
  weekends: true,
  firstDay: 1,
  editable: true,
  selectable: true,
  height: 650,
  expandRows: false,
  allDaySlot: false,
  nowIndicator: true,
  customButtons: {
    avail: {
      text: 'Disponibilités',
      click: function() {
        availView.value = !availView.value
        if(availView.value) {
          getHourlyAttendance()
        } else {
          emptyEvents()
        }
      }
    }

  },
  headerToolbar: {
    start: 'title',
    center: 'avail',
    end: 'today prev next'

  },
  events: [
  ],
  datesSet: (info) => {
    emptyEvents()
    if(availView.value) {
      getHourlyAttendance()
    }
  }
})

const calendar = ref(null);

const hours = ref({
  id: 1,
  name: 'Les zozos du dojo',
  members: [
    {
      id: 1,
      firstName: 'Miranda',
      lastName: 'Portique',
      role: 'Admin',
      availabilities : [
        { 
          id: 1,
          start: '2024-01-20 09:00:00',
          end: '2024-01-20 11:00:00',
        },
        {
          id: 2,
          start: '2024-01-20 15:00:00',
          end: '2024-01-20 21:00:00',
        },
        {
          id: 3,
          start: '2024-01-21 10:00:00',
          end: '2024-01-21 18:00:00',
        }
      ]
    },
    {
      id: 2,
      firstName: 'Michel',
      lastName: 'Boulon',
      role: 'User',
      availabilities : [
        { 
          id: 4,
          start: '2024-01-20 8:00:00',
          end: '2024-01-20 10:00:00',
        },
        {
          id: 5,
          start: '2024-01-20 16:00:00',
          end: '2024-01-20 21:00:00',
        },
        {
          id: 6,
          start: '2024-01-21 10:00:00',
          end: '2024-01-21 13:00:00',
        }
      ]
    },
    {
      id: 3,
      firstName: 'Martin',
      lastName: 'Matin',
      role: 'User',
      availabilities : [
        { 
          id: 7,
          start: '2024-01-20 14:00:00',
          end: '2024-01-20 18:00:00',
        },
        {
          id: 8,
          start: '2024-01-20 21:00:00',
          end: '2024-01-20 23:00:00',
        },
        {
          id: 9,
          start: '2024-01-21 08:00:00',
          end: '2024-01-21 11:00:00',
        },
        {
          id: 10,
          start: '2024-01-21 15:00:00',
          end: '2024-01-21 16:00:00',
        },
        { 
          id: 157,
          start: '2024-01-22 08:00:00',
          end: '2024-01-25 16:00:00'
        }
      ]
    },
    {
      id: 3,
      firstName: 'Yaël',
      lastName: 'Rougail',
      role: 'User',
      availabilities : [
        { 
          id: 11,
          start: '2024-01-20 06:00:00',
          end: '2024-01-20 18:00:00',
        },
        {
          id: 12,
          start: '2024-01-21 08:00:00',
          end: '2024-01-21 11:00:00',
        },
        {
          id: 13,
          start: '2024-01-21 15:00:00',
          end: '2024-01-21 16:00:00',
        },
        {
          id: 158,
          start: '2024-01-23 15:00:00',
          end: '2024-01-23 21:00:00',
        },
        {
          id: 159,
          start: '2024-01-25 08:00:00',
          end: '2024-01-25 14:00:00',
        }
      ]
    }


  ]
})

function emptyEvents() {
  calendarOptions.value.events = []
}

function getHourlyAttendance() {
  // Get today's date & set at midnight
  const startDate = getWeekStart()
  startDate.setHours(0, 0, 0, 0)
  let curWeek = true

  // loop for the whole day
  while(curWeek) {
    const endDate = new Date(startDate)
    endDate.setHours(startDate.getHours()+1, 0, 0, 0)
    let attendees = 0

    // evaluate the number of available people for the hour
    for(let member of hours.value.members) {

      for(let availability of member.availabilities) {
        const startAvail = new Date(availability.start).setMinutes(0, 0, 0)
        const endAvail = new Date(availability.end).setMinutes(0, 0, 0)
        
        if(startAvail <= startDate && endAvail >= endDate) {
          attendees++
          break
        }
      }
    }

    if(attendees > 0) {
      let ratio = attendees / hours.value.members.length * 100
      let color = "#8fb4ff"
      if(ratio === 100) {
        color = "#BB9F06"
      }
      else if(ratio >= 50) {
        color = "#5AAA95"
      }
        calendarOptions.value.events.push({
        start:  `${startDate.getFullYear()}-${startDate.getMonth()+1 > 9 ? '':'0'}${startDate.getMonth()+1}-${startDate.getDate() > 9 ? '':'0'}${startDate.getDate()} ${startDate.getHours() > 9 ? '':'0'}${startDate.getHours()}:${startDate.getMinutes() > 9 ? '':'0'}${startDate.getMinutes()}:${startDate.getSeconds() > 9 ? '':'0'}${startDate.getSeconds()}`,
        end: `${endDate.getFullYear()}-${endDate.getMonth()+1 > 9 ? '':'0'}${endDate.getMonth()+1}-${endDate.getDate() > 9 ? '':'0'}${endDate.getDate()} ${endDate.getHours() > 9 ? '':'0'}${endDate.getHours()}:${endDate.getMinutes() > 9 ? '':'0'}${endDate.getMinutes()}:${endDate.getSeconds() > 9 ? '':'0'}${endDate.getSeconds()}`,
        color: color,
        title: `${attendees} disponible${attendees > 1 ? 's':''}`,
        display: 'background'
      }) 
    }
  
    startDate.setHours(startDate.getHours()+1)
    if(startDate.getDay() === 1 && startDate.getHours() === 0) {
      curWeek = false
    }
  }
}
// getHourlyAttendance()

function getWeekStart() {
  let date = new Date()
  if(calendar.value !== null) {
    date = calendar.value.getApi().getDate()
  }
  while(date.getDay() !== 1) {
    date.setDate(date.getDate()-1)
  }
  return date;
}

</script>

<template>
  <div class="flex">
      <section id="calendar-view">
          <full-calendar ref="calendar" :options="calendarOptions"/>
      </section>
      <section>
        <search-bar />
      </section>
  </div>
</template>

<style scoped>

.flex {
  display: flex;
}

#calendar-view {
  padding: 20px;
  box-sizing: border-box;
  width: 60%;
}

@media screen and (max-width: 1060px) {
  #calendar-view {
    width: 100%;
  }
  
}

</style>
