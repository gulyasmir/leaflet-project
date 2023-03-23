<template>
  <div class="list-data">
    <ul>
      <li v-for="itemDay in weekList" :key="itemDay.id">

        <div @click="selectTime(itemDay.id, 'day')" class="accordion">{{ itemDay.date }}</div>

        <ul class="time-list panel">
          <li v-for="itemTime in itemDay.listTime" :key="itemTime.id">
            <span @click="selectTime(itemDay.id, itemTime.timeInfo)"
              :class="timeItemClass(itemDay.id, itemTime.id)">
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

  methods: {
    selectTime(dayIndex, timeInfo) {
      this.$emit('selectControlWeekButtons', {
        dayIndex: dayIndex,
        timeInfo: timeInfo
      })
    },
    timeItemClass(itemDayId, itemTime) {
      let Data = new Date();
      let className = "time";
      if (itemDayId === 0) {
        if (Data.getHours() < 12 && itemTime === 1) {
          className = "time active";
        }
        if (Data.getHours() > 12 && itemTime === 2) {
          className = "time active";
        }
      }
      return className;
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
    getNewData(yaer, month, date,  weekDay) {
      var newDate = date
      var checkdate
      if ((month <= 7 && (month % 2 != 0)) || ((month > 7) && (month % 2 == 0))) {  // 31 день в месяце
        checkdate = 31
      } else if (month === 2) {
        checkdate = checkdate = yaer == '2024' ? 29 : 28
      } else { // 30 дней  в месяце
        checkdate = 30
      }
      if (newDate > checkdate) {
        newDate = newDate - checkdate
        month++
      }
     
      if ((newDate > checkdate) && month == 12) {
        newDate = newDate - checkdate
        month = 1
        yaer++
      }
    let monthText = month < 10 ? '0' + month : month
    let weekDayName = this.getweekDay(weekDay) 
      return weekDayName + ' ' +  newDate // + '.' + monthText
    },
  },
  computed: {
    weekList() {
      let nowDate = new Date()
      let yaer = nowDate.getFullYear()
      let month = nowDate.getMonth() + 1
      let date = nowDate.getDate() - 1
      let weekDay

      let array = []
       for ( let i = 0; i < 8; i++) {
         weekDay =  new Date(new Date().getTime() + i * (24 * 60 * 60 * 1000)).getDay()
         date = date + 1
         let newData =  this.getNewData(yaer, month, date, weekDay)
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
  