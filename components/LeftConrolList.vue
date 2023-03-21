<template>
  <div class="list-data">
    <ul>
      <li v-for="itemDay in list" :key="itemDay.id">
        <div class="accordion">{{ itemDay.day }} {{ itemDay.date }}</div>
        <ul class="time-list panel">
          <li v-for="itemTime in itemDay.listTime" :key="itemTime.id">
            <span :class="timeItemClass(itemDay.date, itemTime.id)">
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
  props: {
    list: {
      type: Array,
      defautl: [],
    },
  },
  mounted(){
    this.openTimeList() 
  },
  methods: {
    timeItemClass(itemDay, itemTime) {
      let Data = new Date();
      let className = "time";
      if (itemDay === Data.getDate().toString()) {
        if (Data.getHours() < 6 && itemTime === 4) {
          className = "time now";
        }
        if (Data.getHours() >= 6 && Data.getHours() < 12 && itemTime === 3) {
          className = "time now";
        }
        if (Data.getHours() >= 12 && Data.getHours() < 18 && itemTime === 2) {
          className = "time now";
        }

        if (Data.getHours() >= 18 && Data.getHours() < 24 && itemTime === 3) {
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
  font-weight: 550;
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
