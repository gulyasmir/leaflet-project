<template>
  <div class="list-data">
    <ul>
      <li v-for="itemDay in weekList" :key="itemDay.id">

        <div @click="selectTime(itemDay.id, 'day')" class="accordion">{{ itemDay.date }}</div>

        <ul class="time-list panel">
          <li v-for="itemTime in itemDay.listTime" :key="itemTime.id">
            <span @click="selectTime(itemDay.id, itemTime.timeInfo)"
              class="time">
              {{ itemTime.title }}
            </span>
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  name: "LeftConrolWeekList",

  mounted() {
   /* document.addEventListener('click',el => {
      console.log(el.target)
      el.target.classList.add('active')
    } )*/
    //
    //  this.openTimeList()
  },
  methods: {
    selectTime(dayIndex, timeInfo) {
      this.$emit('selectControlWeekButtons', {
        dayIndex: dayIndex,
        timeInfo: timeInfo
      })
    },
    getweekDay(weekDay) {
      switch(weekDay) {
        case 0: return 'Воскресенье'; break;
        case 1: return 'Понедельник'; break;
        case 2: return 'Вторник'; break;
        case 3: return 'Среда'; break;
        case 4: return 'Четверг'; break;
        case 5: return 'Пятница'; break;
        case 6: return 'Суббота'; break;
        default: return ''
      }
    },
    getNewData(yaer, month, date, i, weekDay) {
      let newDate = date + i
      let checkdate
      if ((month <= 7 && (month % 2 != 0)) || ((month > 7) && (month % 2 == 0))) {  // 31 день в месяце
        checkdate = 31
      } else if (month === 2) {
        checkdate = checkdate = yaer == '2024' ? 29 : 28
      } else { // 30 дней  в месяце
        checkdate = 30
      }

      if (newDate > checkdate) {
        newDate = 1
        month++
      }
      if ((newDate > checkdate) && month == 12) {
        newDate = 1
        month = 1
        yaer++
      }
    let weekDayName = this.getweekDay(weekDay) 

      return weekDayName + ' ' +  date 
    },
  },
  computed: {
    weekList() {
      let nowDate = new Date()
      let yaer = nowDate.getFullYear()
      let month = nowDate.getMonth()
      let date = nowDate.getDate()
      let weekDay

      let array = []
       for ( let i = 0; i < 8; i++) {
         weekDay =  new Date(new Date().getTime() + i * (24 * 60 * 60 * 1000)).getDay()
         date = date + i
         let newData =  this.getNewData(yaer, month, date, i, weekDay)
         array.push({
           id: i,
           date: newData,
           listTime: [
             {
               id: 1,
               title: "День",
               timeInfo: 'day'
             },
             {
               id: 2,
               title: "Ночь",
               timeInfo: 'night'
             }
         ]
         })
       }
      return array
    }
  }
}

</script>
  
<style scoped>
.list-data {
  background: #fff;
  border-radius: 3px;
  padding: 10px;
}

.list-data ul {
  padding-left: 0;
}

.list-data li {
  list-style: none;
  color: #003661;
  font-size: 13px;
  font-weight: 600;
}

.time-list {
  padding: 5px;
}

.time {
  font-size: 13px;
  font-weight: 500;
  padding-left: 5px;
}

.time.now {
  color: #db0084;
  font-weight: 550;
}

.accordion {
  cursor: pointer;
  transition: 0.4s;
}
.active {
  color: #db0084;

}
.panel {
  display: none;
  overflow: hidden;
  cursor: pointer;
}
</style>
  