<template>
  <div class="list-data">
    <ul>
      <li v-for="itemDay in list" :key="itemDay.id">
        <div @click="selectTime(itemDay.dayInfo, 'day')"  :class="getDayClass(itemDay.id)">{{ itemDay.day }}</div>
        <ul class="time-list panel" :style="getDayStyle(itemDay.id)">
          <li v-for="itemTime in itemDay.listTime" :key="itemTime.id">
            <span @click="selectTime(itemDay.dayInfo,itemTime.timeInfo, $event)" :class="timeItemClass(itemDay.dayInfo, itemTime.id)">
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
  name: "LeftConrolList",
  data(){
    return{
     list:  [
        {
          id: 0,
          day: "Сегодня",
          dayInfo: "today",
          listTime: [
            {
              id: 1,
              title: "День",
              timeInfo: "day",
            },
            {
              id: 2,
              title: "Ночь",
              timeInfo: "night",
            },
          ],
        },
        {
          id: 1,
          day: "Завтра",
          dayInfo: "tomorrow",
          listTime: [
            {
              id: 1,
              title: "День",
              timeInfo: "day",
            },
            {
              id: 2,
              title: "Ночь",
              timeInfo: "night",
            },
          ],
        },
      ]
    }
  },
  mounted() {
    this.openTimeList()

  },
  methods: {
    getDayClass(id) {
      return id === 0 ? "accordion active" : "accordion";
    },
    getDayStyle(id) {
      if (id === 0) {
        return "display:block";
      }
    },
    removeClassNow() {
      let elements = document.getElementsByClassName("time");
      if (elements) {
        Array.from(elements).forEach((element) => {
          element.classList.remove("now");
        });
      }
    },
    selectTime(dayInfo, timeInfo, event){
      if (event) {
        this.removeClassNow();
        event.target.className += " now";
      }
      this.$emit('selectControlButtons', {
              dayInfo: dayInfo,
              timeInfo: timeInfo
            })
    },
    timeItemClass(itemDay, itemTime) {
      let Data = new Date();
      let className = "time";
      if (itemDay === 'today') {
        if (Data.getHours() > 9 && Data.getHours() < 21 && itemTime === 1) {
          className = "time now";
        }
        if (Data.getHours() < 9 && Data.getHours() > 21 && itemTime === 2) {
          className = "time now";
        }
      }
      return className;
    },

    openTimeList() {
      let acc = document.getElementsByClassName("accordion");
      let i;
      acc[0].nextElementSibling.style.display = "block"
      for (i = 0; i < acc.length; i++) {
        acc[i].addEventListener("click", function () {
          this.classList.toggle("active")
          var panel = this.nextElementSibling
          if (panel.style.display === "block") {
            panel.style.display = "none"
          } else {
            panel.style.display = "block"
          }
        });
      }
    },
  },
};
</script>

<style>
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
  font-size: 14px;
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
}
.active {
  color: #db0084;
}
.accordion {
  cursor: pointer;
  transition: 0.4s;
}

.panel {
  display: none;
  overflow: hidden;
  cursor: pointer;
}
</style>
