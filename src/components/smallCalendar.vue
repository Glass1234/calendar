<template>
  <div
    class="calendar"
    :ref="(element) => (animation.calendarRef.value = element)"
  >
    <div class="calendar-bg">
      <div class="calendar-using">
        <div class="calendar-nav">
          <span
            class="calendar-nav-text"
            :ref="(element) => (animation.monthRef.value = element)"
            >{{ calendar.month }}</span
          >
          <div class="modal">
            <button
              class="btn calendar-btn"
              :ref="(element) => (animation.downBtn1.value = element)"
              @click="modal.openMonth"
            >
              <img src="../assets/icons/grayRow.svg" alt="" />
            </button>
            <div
              v-if="modal.isOpenMonth"
              class="modal-month"
              v-on-click-outside="foo1"
              :ref="(element) => (animation.monthsRef.value = element)"
            >
              <div
                class="date"
                :class="{ 'date-selected': item === calendar.month }"
                v-for="(item, index) in calendar.MONTH"
                :key="index"
                @click="selectMonth(item)"
              >
                {{ item }}
              </div>
            </div>
          </div>

          <span
            class="calendar-nav-text"
            :ref="(element) => (animation.yearRef.value = element)"
            >{{ calendar.year }}</span
          >
          <div class="modal">
            <button
              class="btn calendar-btn"
              :ref="(element) => (animation.downBtn2.value = element)"
              @click="modal.openYear"
            >
              <img src="../assets/icons/grayRow.svg" alt="" />
            </button>
            <div
              v-if="modal.isOpenYear"
              class="modal-year"
              v-on-click-outside="foo2"
              :ref="(element) => (animation.yearsRef.value = element)"
            >
              <div
                class="date"
                :class="{ 'date-selected': item === calendar.year }"
                v-for="(item, index) in calendar.YEARS"
                :key="index"
                @click="selectYear(item)"
              >
                {{ item }}
              </div>
            </div>
          </div>

          <div class="space" />
          <div class="month-nav">
            <button
              class="btn calendar-btn"
              @click="calendar.prevPage()"
              :ref="(element) => (animation.leftBtn.value = element)"
            >
              <img
                src="../assets/icons/rightRow.svg"
                alt=""
                style="rotate: 180deg"
              />
            </button>
            <button
              class="btn calendar-btn"
              @click="calendar.nextPage()"
              :ref="(element) => (animation.rigthBtn.value = element)"
            >
              <img src="../assets/icons/rightRow.svg" alt="" />
            </button>
          </div>
        </div>
        <div
          class="calendar-DaysWeek"
          :ref="(element) => (animation.weekRef.value = element)"
        >
          <div
            class="item item-week"
            v-for="(item, index) in calendar.WEEK"
            :key="index"
          >
            {{ item }}
          </div>
        </div>
        <div
          class="calendar-Days"
          :ref="(element) => (animation.daysRef.value = element)"
        >
          <div
            class="item item-day"
            :class="{
              red: item.isRed && item.isActual,
              'item-disabled': !item.isActual,
            }"
            v-for="(item, index) in calendar.days"
            :key="index"
          >
            {{ item.number }}
          </div>
        </div>
      </div>
      <img class="imgBG" src="../assets/ellipse.png" alt="" />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, nextTick } from "vue";
import { vOnClickOutside } from "@vueuse/components";
import { gsap } from "gsap";

class Animation {
  constructor() {
    this.calendarRef = ref(null);
    this.monthRef = ref(null);
    this.yearRef = ref(null);
    this.downBtn1 = ref(null);
    this.downBtn2 = ref(null);
    this.leftBtn = ref(null);
    this.rigthBtn = ref(null);
    this.weekRef = ref(null);
    this.daysRef = ref(null);
    this.monthsRef = ref(null);
    this.yearsRef = ref(null);
  }
  moveCalendar() {
    const downBtn = () => {
      const g = gsap.timeline({ defaults: { duration: 0.2 } });
      g.from(this.downBtn1.value, { opacity: 0, y: -20, stagger: 0 }, 0).from(
        this.downBtn2.value,
        { opacity: 0, y: -20, stagger: 0 },
        0
      );
      return g;
    };
    const moveBtn = () => {
      const g = gsap.timeline({ defaults: { duration: 0.2 } });
      g.from(this.rigthBtn.value, { opacity: 0, y: -20, x: 20 }).from(
        this.leftBtn.value,
        { opacity: 0, x: -100 }
      );
      return g;
    };
    const hiddenWeek = () => {
      const daysOfWeek = Array.from(this.weekRef.value.children);
      const firstThreeDays = daysOfWeek.slice(0, 3);
      const lastThreeDays = daysOfWeek.slice(-3);
      const g = gsap.timeline({ defaults: { duration: 0.1 } });
      firstThreeDays.forEach((day, index) => {
        g.from(day, {
          opacity: 0,
          x: -20,
          delay: index * 0.001,
        });
      });
      lastThreeDays.reverse().forEach((day, index) => {
        g.from(day, {
          opacity: 0,
          x: 20,
          delay: index * 0.001,
        });
      });
      g.from(daysOfWeek[3], {
        opacity: 0,
        y: -30,
        delay: 4 * 0.001,
      });
      return g;
    };
    const hiddenDay = () => {
      const daysOfWeek = Array.from(this.daysRef.value.children);
      const randomOfWeek = daysOfWeek.sort(() => Math.random() - 0.5);
      const g = gsap.timeline({ defaults: { duration: 0.001 } });
      randomOfWeek.forEach((day) => {
        g.from(day, {
          opacity: 0,
          delay: 0.04,
        });
      });
    };
    const tl = gsap.timeline({ defaults: { duration: 0.1 } });
    tl.from(this.monthRef.value, { opacity: 0, y: -20, x: -20 })
      .from(this.yearRef.value, { opacity: 0, y: -20 })
      .add(downBtn())
      .add(moveBtn())
      .add(hiddenWeek())
      .add(hiddenDay());
  }
  slidLeft() {
    gsap.fromTo(
      this.monthRef.value,
      {
        opacity: 0,
        x: -5,
      },
      {
        opacity: 1,
        x: 0,
        duration: 0.7,
        delay: 0,
      }
    );
    gsap.fromTo(
      this.yearRef.value,
      {
        opacity: 0,
        x: -5,
      },
      {
        opacity: 1,
        x: 0,
        duration: 0.7,
        delay: 0,
      }
    );
  }
  slidRight() {
    gsap.fromTo(
      this.monthRef.value,
      {
        opacity: 0,
        x: 5,
      },
      {
        opacity: 1,
        x: 0,
        duration: 0.7,
        delay: 0,
      }
    );
    gsap.fromTo(
      this.yearRef.value,
      {
        opacity: 0,
        x: 5,
      },
      {
        opacity: 1,
        x: 0,
        duration: 0.7,
        delay: 0,
      }
    );
  }
  slideDayRight() {
    const moveDays = () => {
      const chunkArray = (arr, size) => {
        var result = [];
        for (var i = 0; i < arr.length; i += size) {
          result.push(arr.slice(i, i + size));
        }
        return result;
      };
      const transpose = (a) => a[0].map((_, i) => a.map((row) => row[i]));
      const weeks = Array.from(this.weekRef.value.children);
      const days = Array.from(this.daysRef.value.children);
      const g = gsap.timeline({ defaults: { duration: 0.05 } });
      let union = [...weeks, ...days];
      union = transpose(chunkArray(union.reverse(), 7));
      union.forEach((day) => {
        g.fromTo(
          day,
          {
            opacity: 0,
            x: 20,
          },
          {
            opacity: 1,
            x: 0,
          }
        );
      });
      return g;
    };
    const tl = gsap.timeline({ defaults: { duration: 0.1 } });
    tl.add(moveDays());
  }
  slideDayLeft() {
    const moveDays = () => {
      const chunkArray = (arr, size) => {
        var result = [];
        for (var i = 0; i < arr.length; i += size) {
          result.push(arr.slice(i, i + size));
        }
        return result;
      };
      const transpose = (a) => a[0].map((_, i) => a.map((row) => row[i]));
      const weeks = Array.from(this.weekRef.value.children);
      const days = Array.from(this.daysRef.value.children);
      const g = gsap.timeline({ defaults: { duration: 0.05 } });
      let union = [...weeks, ...days];
      union = transpose(chunkArray(union, 7));
      union.forEach((day) => {
        g.fromTo(
          day,
          {
            opacity: 0,
            x: -20,
          },
          {
            opacity: 1,
            x: 0,
          }
        );
      });
      return g;
    };
    const tl = gsap.timeline({ defaults: { duration: 0.1 } });
    tl.add(moveDays());
  }
  dropMonths() {
    const drop = () => {
      const g = gsap.timeline({ defaults: { duration: 0.05 } });
      const months = Array.from(this.monthsRef.value.children);
      months.forEach((month) => {
        g.fromTo(
          month,
          {
            opacity: 0,
            y: -20,
          },
          {
            opacity: 1,
            y: 0,
          }
        );
      });
      return g;
    };
    const tl = gsap.timeline({ defaults: { duration: 0.1 } });
    tl.add(drop());
  }
  dropYears() {
    const drop = () => {
      const g = gsap.timeline({ defaults: { duration: 0.05 } });
      const years = Array.from(this.yearsRef.value.children);
      years.forEach((year) => {
        g.fromTo(
          year,
          {
            opacity: 0,
            y: -20,
          },
          {
            opacity: 1,
            y: 0,
          }
        );
      });
      return g;
    };
    const tl = gsap.timeline({ defaults: { duration: 0.1 } });
    tl.add(drop());
  }
  dropMonth() {
    const month = this.monthRef.value;
    gsap.fromTo(
      month,
      {
        opacity: 0,
        y: -20,
      },
      {
        opacity: 1,
        y: 0,
      }
    );
  }
  dropYear() {
    const month = this.yearRef.value;
    gsap.fromTo(
      month,
      {
        opacity: 0,
        y: -20,
      },
      {
        opacity: 1,
        y: 0,
      }
    );
  }
  dropDays() {
    const drop = () => {
      const days = Array.from(this.daysRef.value.children).sort(
        () => Math.random() - 0.5
      );
      const g = gsap.timeline({ defaults: { duration: 1.5, delay: 0 } });
      g.fromTo(
        days,
        {
          opacity: 0,
          y: () => Math.floor(-100 + Math.random() * (100 + 1 - -100)),
          x: () => Math.floor(-100 + Math.random() * (100 + 1 - -100)),
          rotate: () => Math.floor(-360 + Math.random() * (360 + 1 - -360)),
          rotateX: () => Math.floor(-360 + Math.random() * (360 + 1 - -360)),
          rotateY: () => Math.floor(-360 + Math.random() * (360 + 1 - -360)),
          scale: () => Math.floor(-3 + Math.random() * (3 + 1 - -3)),
        },
        { opacity: 1, y: 0, x: 0, rotate: 0, rotateX: 0, rotateY: 0, scale: 1 }
      );
      return g;
    };
    const tl = gsap.timeline({ defaults: { duration: 0 } });
    tl.add(drop());
  }
}

class Calendar {
  WEEK = ["пн", "вт", "ср", "чт", "пт", "сб", "вс"];
  CONVERTWEEK = ["вс", "пн", "вт", "ср", "чт", "пт", "сб"];
  MONTH = [
    "Январь",
    "Февраль",
    "Март",
    "Апрель",
    "Май",
    "Июнь",
    "Июль",
    "Август",
    "Сентябрь",
    "Октябрь",
    "Ноябрь",
    "Декабырь",
  ];
  YEARS = [];
  year = null;
  month = null;
  days = [];
  constructor() {
    const date = new Date();
    this.days = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
    this.setActualDays();
    this.setTopData(date);
    this.addPrevDays();
    this.addNextDays();
    this.setNotActualDays();
    this.addYears();
    this.reduceYears();
  }
  getArrDays(year, month) {
    const date = new Date(year, month, 1);
    const firstDay = new Date(date);
    firstDay.setDate(1);
    const count = new Date(year, month, 0).getDate();
    let arr = [];
    for (let i = 1; i <= count; i++) {
      const dayDate = new Date(year, month - 1, i);
      const dayOfWeek = dayDate.getDay();
      const dayOfWeekString = this.CONVERTWEEK[dayOfWeek];
      arr.push({
        number: i,
        week: dayOfWeekString,
        isRed: dayOfWeek > 5 || dayOfWeek === 0,
      });
    }
    return arr;
  }
  setTopData(date) {
    this.year = date.getFullYear();
    this.month = this.MONTH[date.getMonth()];
  }
  addPrevDays() {
    const leftDay = this.days[0];
    const needDays = this.WEEK.indexOf(leftDay.week);
    if (needDays > 0) {
      const date = new Date(this.year, this.MONTH.indexOf(this.month) - 1);
      let leftArr = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
      leftArr = leftArr.slice(-needDays);
      this.days = [...leftArr, ...this.days];
    }
  }
  addNextDays() {
    const rightDay = this.days.at(-1);
    const needDays = this.WEEK.length - this.WEEK.indexOf(rightDay.week) - 1;
    if (needDays > 0) {
      const date = new Date(this.year, this.MONTH.indexOf(this.month) + 1);
      let rightArr = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
      rightArr = rightArr.slice(0, needDays);
      this.days = [...this.days, ...rightArr];
    }
  }
  async nextPage() {
    const date = new Date(this.year, this.MONTH.indexOf(this.month) + 1);
    this.days = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
    this.setActualDays();
    this.setTopData(date);
    this.addPrevDays();
    this.addNextDays();
    this.setNotActualDays();
    await nextTick();
    animation.slidLeft();
    animation.slideDayLeft();
  }
  async prevPage() {
    const date = new Date(this.year, this.MONTH.indexOf(this.month) - 1);
    this.days = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
    this.setActualDays();
    this.setTopData(date);
    this.addPrevDays();
    this.addNextDays();
    this.setNotActualDays();
    await nextTick();
    animation.slidRight();
    animation.slideDayRight();
  }
  setActualDays() {
    this.days.forEach((item) => {
      item["isActual"] = true;
    });
  }
  setNotActualDays() {
    this.days.forEach((item) => {
      if ("isActual" in item === false) {
        item["isActual"] = false;
      }
    });
  }
  selectMonth(month) {
    this.month = month;
    const date = new Date(this.year, this.MONTH.indexOf(this.month));
    this.days = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
    this.setActualDays();
    this.setTopData(date);
    this.addPrevDays();
    this.addNextDays();
    this.setNotActualDays();
  }
  selectYear(year) {
    this.year = year;
    const date = new Date(this.year, this.MONTH.indexOf(this.month));
    this.days = this.getArrDays(date.getFullYear(), date.getMonth() + 1);
    this.setActualDays();
    this.setTopData(date);
    this.addPrevDays();
    this.addNextDays();
    this.setNotActualDays();
  }
  addYears() {
    if (this.YEARS.length === 0) {
      this.YEARS.push(this.year);
    }
    for (let i = 0; i < 10; i++) {
      this.YEARS.push(this.YEARS.at(-1) + 1);
    }
  }
  reduceYears() {
    if (this.YEARS.length === 0) {
      this.YEARS.push(this.year);
    }
    const tmp = [this.YEARS[0] - 1];
    for (let i = 0; i < 9; i++) {
      tmp.push(tmp.at(-1) - 1);
    }
    this.YEARS = [...tmp.reverse(), ...this.YEARS];
  }
}

class Modal {
  isOpenMonth = false;
  isOpenYear = false;
  closeMonth() {
    this.isOpenMonth = false;
  }
  async openMonth() {
    this.isOpenMonth = !this.isOpenMonth;
    await nextTick();
    animation.dropMonths();
  }
  closeYear() {
    this.isOpenYear = false;
  }
  async openYear() {
    this.isOpenYear = !this.isOpenYear;
    await nextTick();
    animation.dropYears();
  }
}

const calendar = reactive(new Calendar());
const animation = new Animation();
const modal = reactive(new Modal());

const selectMonth = async (month) => {
  if (month !== calendar.month) {
    modal.closeMonth();
    calendar.selectMonth(month);
    await nextTick();
    animation.dropDays();
    animation.dropMonth();
  }
};

const selectYear = async (year) => {
  if (year !== calendar.year) {
    modal.closeYear();
    calendar.selectYear(year);
    await nextTick();
    animation.dropDays();
    animation.dropYear();
  }
};

const foo1 = () => {
  modal.closeMonth();
};

const foo2 = () => {
  modal.closeYear();
};

onMounted(() => {
  animation.moveCalendar();
});
</script>

<style scoped lang="scss">
.calendar {
  width: fit-content;
  &-bg {
    padding: 24px;
    border-radius: 12px;
    background-color: #1b1a1a;
    position: relative;
  }
  &-using {
    position: sticky;
    z-index: 10;
  }
  &-nav {
    display: flex;
    align-items: center;
    gap: 0 12px;
    &-text {
      color: white;
      font-size: 24px;
      font-weight: 500;
    }
  }
  &-DaysWeek {
    margin-top: 20px;
    display: flex;
    gap: 0 8px;
  }
  &-Days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    grid-column-gap: 8px;
    grid-row-gap: 8px;
  }
}
.month-nav {
  display: flex;
  align-items: center;
  gap: 0 7px;
}
.item {
  font-size: 16px;
  width: 40px;
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  &-day {
    color: white;
    background-color: #ffffff1a;
    border-radius: 4px;
  }
  &-week {
    color: #8a8a8a;
  }
  &-disabled {
    color: #737373;
    cursor: default;
  }
}
.red {
  color: #ff365e;
}
.btn {
  background: transparent;
  padding: 0;
  border-width: 0;
  cursor: pointer;
  height: fit-content;
}
.space {
  width: 100%;
}
.imgBG {
  position: absolute;
  bottom: 0;
  left: 0;
  border-radius: 12px;
  width: -webkit-fill-available;
}
.modal {
  position: relative;
  &-month {
    position: absolute;
    right: 0;
    top: 30px;
    z-index: 10;
    background: #1b1a1a;
    border: 1px solid #ffffff1f;
    border-radius: 9px;
    padding: 6px;
  }
  &-year {
    position: absolute;
    right: 0;
    top: 30px;
    z-index: 10;
    background: #1b1a1a;
    border: 1px solid #ffffff1f;
    border-radius: 9px;
    padding: 6px;
    max-height: 150px;
    overflow: auto;
  }
}
.date {
  color: white;
  font-size: 12px;
  padding: 3px 7px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.13s;
  &:hover {
    background-color: #ffffff1a;
  }
  &-selected {
    background-color: #495bff;
    &:hover {
      background-color: darken(#495bff, 10%);
    }
  }
}
*::-webkit-scrollbar,
*::-webkit-scrollbar-thumb {
  width: 6px;
  border-radius: 13px;
  background-clip: padding-box;
  border: 2px solid transparent;
}

*::-webkit-scrollbar-thumb {
  box-shadow: inset 0 0 0 10px #495bff;
}
</style>
