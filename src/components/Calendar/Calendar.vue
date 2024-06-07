<template>
  <div class="subscription-date">
    <div class="form-control">
      <h2 class="form-control__title">When do you want me to remind you?</h2>
      <form class="form" @submit.prevent="handleSubmit">
        <label for="one-week">
          <div
            :class="[
              'form__item',
              { 'form__item--selected': postponeOption === 'one-week' },
            ]"
            @click="handlePostpone"
          >
            <input
              id="one-week"
              v-model="postponeOption"
              name="shipment-date"
              type="radio"
              value="one-week"
            />
            In one week
          </div>
        </label>
        <label for="two-weeks">
          <div
            :class="[
              'form__item',
              { 'form__item--selected': postponeOption === 'two-weeks' },
            ]"
            @click="handlePostpone"
          >
            <input
              id="two-weeks"
              v-model="postponeOption"
              name="shipment-date"
              type="radio"
              value="two-weeks"
            />
            In 2 weeks
          </div>
        </label>
        <label for="custom">
          <div
            :class="[
              'form__item',
              { 'form__item--selected': postponeOption === 'custom' },
            ]"
            @click="handleClickCustomDate"
          >
            <input
              id="custom"
              v-model="postponeOption"
              name="shipment-date"
              type="radio"
              value="custom"
            />
            Select a custom date
          </div>
        </label>
        <Transition name="show-selected-date">
          <div
            v-if="!!dateToBeSubmitted && !calendarIsOpen"
            class="selected-date-container"
          >
            <div class="selected-date-container__scheduled">
              <h4 class="selected-date-container__title">
                Current selected date
              </h4>
              <div class="selected-date-container__date">
                {{ scheduledDate }}
              </div>
            </div>
            <div class="selected-date-container__requested">
              <h4 class="selected-date-container__title">New date</h4>
              <div class="selected-date-container__date">
                {{ dateToBeSubmitted }}
              </div>
            </div>
          </div>
        </Transition>

        <Transition name="save-button">
          <button
            v-if="dateToBeSubmitted && !calendarIsOpen"
            :disabled="!dateToBeSubmitted"
          >
            Submit
          </button>
        </Transition>

        <Transition name="open-calendar">
          <div v-if="calendarIsOpen" class="calendar-container">
            <div class="date-container">
              <h4 class="date-container__title">Select and Save</h4>
              <div class="date-container__date">
                {{ selectedDateInCalendar }}
              </div>
              <div class="date-container__description">
                The date marked with
                <div class="date-container__description-square"></div>
                is the current selected date
              </div>
            </div>
            <div class="calendar">
              <!-- <button @click="toggleWeekStart">{{ firstDayOfTheWeek }}-start</button> -->
              <header class="calendar__header">
                <div class="calendar__wrapper">
                  <div class="left-chev chev">
                    <span @click="updateDate('prev-month')">&lt;</span>
                  </div>
                  <div class="calendar__info">
                    <span class="calendar__month">{{ current.month }}</span>
                    <span class="calendar__year">{{ currentYear }}</span>
                  </div>
                  <div class="right-chev chev">
                    <span @click="updateDate('next-month')">&gt;</span>
                  </div>
                </div>
              </header>

              <div class="calendar__grid">
                <div class="calendar__days-grid">
                  <div
                    v-for="(day, index) in days"
                    :key="index"
                    :class="[
                      'calendar__day',
                      {
                        'calendar__day--today': day === today.day && isToday,
                      },
                    ]"
                  >
                    {{ day }}
                  </div>
                </div>
                <div class="calendar__date-grid">
                  <div
                    v-for="dt in dateGrid"
                    :key="dt.id"
                    class="calendar__date"
                  >
                    <div
                      :class="[
                        'calendar__val',
                        {
                          'calendar__val--today':
                            dt.id < 99 && dt.dt === today.date && isToday,
                          'calendar__val--prev': dt.id > 99 && dt.id < 199,
                          'calendar__val--next': dt.id > 199,
                          'calendar__val--selected': dt.isSelected,
                          'calendar__val--scheduled':
                            dt.id < 99 &&
                            dt.dt === currentScheduledDate.date &&
                            isScheduled,
                        },
                      ]"
                      @click="handleSelectDate(dt.id, dt.month, dt.year)"
                    >
                      {{ dt.dt }}
                    </div>
                  </div>
                </div>
              </div>
              <Transition name="invalid">
                <div v-if="isInvalidDate" class="calendar__val--invalid">
                  Please select a future date!
                </div>
              </Transition>
            </div>
            <div class="button-container">
              <button
                @click="saveDate"
                :disabled="isInvalidDate || !selectedDate"
              >
                Save the date
              </button>
            </div>
          </div>
        </Transition>
      </form>
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent } from "vue";

type MonthObject = {
  [key: string]: string;
};

interface DateItemObject {
  id: number;
  dt: number;
  day: string;
  month: number;
  year: number;
  isSelected: boolean;
}

export default defineComponent({
  data() {
    return {
      date: new Date(),
      months: {
        Jan: "Januari",
        Feb: "Februari",
        Mar: "Mars",
        Apr: "April",
        May: "Maj",
        Jun: "Juni",
        Jul: "Juli",
        Aug: "Augusti",
        Sep: "September",
        Oct: "Oktober",
        Nov: "November",
        Dec: "December",
      } as MonthObject,
      firstDayOfTheWeek: "Monday",
      selectedDateId: null as number | null,
      selectedMonth: null as number | null,
      selectedYear: null as number | null,
      selectedDate: null as Date | null,
      dateClicked: false,
      visiblePrevMonthDays: [] as DateItemObject[],
      isInvalidDate: false,
      postponeOption: null,
      selectedCustomDate: "",
      calendarIsOpen: false,
      // scheduledDate: "Wed Apr 10 2024", // temp
      // scheduledDate: "2024-04-10", // temp
      scheduledDate: "",
    };
  },
  computed: {
    days() {
      if (this.firstDayOfTheWeek === "Monday") {
        return ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];
      }
      return ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
    },
    dateString() {
      return this.date.toDateString();
    },
    today() {
      const dayIndex = new Date().getDay();
      const day =
        this.firstDayOfTheWeek === "Monday"
          ? this.days[dayIndex - 1] // Adjust output index for Monday-start
          : this.days[dayIndex];
      const date = new Date().getDate();
      const month = new Date().getMonth() + 1;
      const year = new Date().getFullYear();

      return { day, date, month, year };
    },
    currentMonthNum() {
      return this.date.getMonth() + 1;
    },
    currentYear() {
      return this.date.getFullYear();
    },
    current() {
      const [_, monthStr, _2, yearStr] = this.dateString.split(" ");
      return {
        month: this.months[monthStr],
        // year: yearStr,
        monthKey: monthStr,
      };
    },
    dateGrid() {
      return this.getDateGrid(this.currentMonthDates);
    },
    isToday() {
      return (
        this.date.getMonth() + 1 === this.today.month &&
        this.currentYear === this.today.year
      );
    },
    isLeapYear() {
      const firstCondition =
        this.currentYear % 4 === 0 && this.currentYear % 100 !== 0;
      const secondCondition =
        this.currentYear % 4 === 0 &&
        this.currentYear % 100 === 0 &&
        this.currentYear % 400 === 0;
      if (firstCondition || secondCondition) return true;
      return false;
    },
    prevMonth() {
      return this.getMonthBeside("prev");
    },
    nextMonth() {
      return this.getMonthBeside("next");
    },
    currentMonthDates() {
      return this.getMonthInfoArr(
        this.current.monthKey,
        this.currentMonthNum,
        this.currentYear
      );
    },
    selectedDateInCalendar() {
      if (this.selectedDate === null) return "Please select a date";
      if (this.isInvalidDate) return "Invalid date";
      const date = this.selectedDate.toLocaleDateString();
      return date;
    },
    dateToBeSubmitted() {
      if (
        this.postponeOption === "one-week" ||
        this.postponeOption === "two-weeks"
      ) {
        return this.getNewShipmentDate();
      } else {
        return this.selectedCustomDate;
      }
    },
    currentScheduledDate() {
      const date = new Date(this.scheduledDate).getDate();
      const month = new Date(this.scheduledDate).getMonth() + 1;
      const year = new Date(this.scheduledDate).getFullYear();
      return { date, month, year };
    },
    isScheduled() {
      return (
        this.date.getMonth() + 1 === this.currentScheduledDate.month &&
        this.currentYear === this.currentScheduledDate.year
      );
    },
  },
  mounted() {
    this.setScheduledDate();
  },
  methods: {
    increaseMonth() {
      if (this.currentMonthNum !== 12) {
        return new Date(`${this.currentYear}-${this.currentMonthNum + 1}`);
      } else {
        return new Date(`${this.currentYear + 1}-01`);
      }
    },
    decreaseMonth() {
      if (this.currentMonthNum !== 1) {
        return new Date(`${this.currentYear}-${this.currentMonthNum - 1}`);
      } else {
        return new Date(`${this.currentYear - 1}-12`);
      }
    },
    updateDate(direction: string) {
      this.isInvalidDate = false;
      if (direction === "next-month") {
        this.date = this.increaseMonth();
      } else {
        this.date = this.decreaseMonth();
      }
    },
    getNumberOfDays(month: string) {
      const thirtyOneDays = ["Jan", "Mar", "May", "Jul", "Aug", "Oct", "Dec"];
      if (thirtyOneDays.includes(month)) {
        return 31;
      } else if (month === "Feb") {
        return this.isLeapYear ? 29 : 28;
      } else {
        return 30;
      }
    },
    getMonthBeside(direction: string) {
      const target =
        direction === "prev" ? this.decreaseMonth() : this.increaseMonth();
      const monthNum = target.getMonth() + 1;
      const [_, monthKey, _2, year] = target.toDateString().split(" ");
      const monthInfo = this.getMonthInfoArr(monthKey, monthNum, +year);
      return monthInfo;
    },
    getDateGrid(currentMonthDates: DateItemObject[]) {
      const monthDates =
        this.selectedDateId !== null &&
        this.selectedMonth !== null &&
        this.selectedYear !== null
          ? currentMonthDates.map((date) =>
              date.id === this.selectedDateId &&
              date.month === this.selectedMonth &&
              date.year === this.selectedYear
                ? { ...date, isSelected: true }
                : date
            )
          : currentMonthDates;

      const firstDayIndex = this.days.indexOf(monthDates[0].day);
      const lastDayIndex = this.days.indexOf(
        monthDates[monthDates.length - 1].day
      );

      const firstVisiblePrevMonthDayIndex =
        this.prevMonth && this.prevMonth.length - firstDayIndex;

      // Prevent from splicing when only date is selected
      if (!this.dateClicked && firstVisiblePrevMonthDayIndex) {
        this.visiblePrevMonthDays = this.prevMonth?.splice(
          firstVisiblePrevMonthDayIndex,
          this.prevMonth.length
        );
      }

      const visibleNextMonthDays = this.nextMonth.slice(0, 6 - lastDayIndex);

      let newDateGrid: DateItemObject[] = [];
      if (this.visiblePrevMonthDays && this.visiblePrevMonthDays.length > 0) {
        let n = 0;
        for (let i = 0; i <= firstDayIndex; i++) {
          if (i < firstDayIndex) {
            // Add dates from prev month first if grid space is available
            newDateGrid = [
              ...newDateGrid,
              {
                id: +this.visiblePrevMonthDays[n].dt + 100,
                dt: this.visiblePrevMonthDays[n].dt,
                day: this.days[i],
                month:
                  this.currentMonthNum === 1 ? 12 : this.currentMonthNum - 1,
                year:
                  this.currentMonthNum === 1
                    ? this.currentYear - 1
                    : this.currentYear,
                isSelected: false,
              },
            ];
            n++;
          } else {
            // Add current month dates
            newDateGrid = [...newDateGrid, ...monthDates];
          }
        }
      } else {
        newDateGrid = [...newDateGrid, ...monthDates];
      }

      if (visibleNextMonthDays && visibleNextMonthDays.length > 0) {
        // Add dates from prev month if grid space is available
        for (let i = 0; i < visibleNextMonthDays.length; i++) {
          newDateGrid = [
            ...newDateGrid,
            {
              id: +visibleNextMonthDays[i].dt + 200,
              dt: +visibleNextMonthDays[i].dt,
              day: this.days[i],
              month: this.currentMonthNum === 12 ? 1 : this.currentMonthNum + 1,
              year:
                this.currentMonthNum === 12
                  ? this.currentYear + 1
                  : this.currentYear,
              isSelected: false,
            },
          ];
        }
      }
      this.dateClicked = false;
      return newDateGrid;
    },
    getMonthInfoArr(monthKey: string, monthNum: number, year: number) {
      const monthInfoArr = [];

      for (let i = 0; i < this.getNumberOfDays(monthKey); i++) {
        const item = {} as DateItemObject;
        const day =
          this.firstDayOfTheWeek === "Monday"
            ? new Date(`${year}-${monthNum}-${i + 1}`).getDay() - 1
            : new Date(`${year}-${monthNum}-${i + 1}`).getDay();
        item.id = i + 1;
        item.dt = i + 1;
        item.day =
          day === -1 && this.firstDayOfTheWeek === "Monday"
            ? (item.day = this.days[day + 7])
            : (item.day = this.days[day]);
        item.month = monthNum;
        item.year = year;
        item.isSelected = false;

        monthInfoArr.push(item);
      }
      return monthInfoArr;
    },
    handleSelectDate(dateId: number, month: number, year: number) {
      if (dateId > 31) return;

      this.selectedDate = new Date(`${year}-${month}-${dateId}`);
      const today = new Date(
        `${this.today.year}-${this.today.month}-${this.today.date}`
      );

      if (this.selectedDate < today) {
        this.isInvalidDate = true;
        this.selectedDateId = null;
        this.dateClicked = true;
        return;
      }

      this.selectedDateId = dateId;
      this.selectedMonth = month;
      this.selectedYear = year;

      this.isInvalidDate = false;
      this.dateClicked = true;
    },
    saveDate() {
      if (!this.selectedDate) return;
      // this.selectedCustomDate = this.selectedDate?.toDateString();
      this.selectedCustomDate = this.selectedDate?.toLocaleDateString();
      this.postponeOption = null;
      this.calendarIsOpen = false;
    },

    getLocaleDate(date: string) {
      const event = new Date(date);
      const options = {
        weekday: "long",
        year: "numeric",
        month: "long",
        day: "numeric",
      };
      console.log(event.toLocaleDateString(date));
    },

    handleClickCustomDate() {
      this.calendarIsOpen = true;
    },

    closeCalendar() {
      this.calendarIsOpen = false;
    },

    getNewShipmentDate() {
      const fullDate = new Date(this.scheduledDate);
      this.postponeOption === "one-week"
        ? fullDate.setDate(fullDate.getDate() + 7)
        : fullDate.setDate(fullDate.getDate() + 14);
      return fullDate.toLocaleDateString();
    },
    handlePostpone() {
      this.closeCalendar();
    },

    handleSubmit(e: any) {
      e.preventDefault();
      console.log(this.dateToBeSubmitted);
    },

    setScheduledDate() {
      const date = new Date(
        `${this.today.year}-${this.today.month}-${this.today.date}`
      );
      this.scheduledDate = date.toLocaleDateString();
    },

    toggleWeekStart() {
      if (this.firstDayOfTheWeek === "Monday") {
        this.firstDayOfTheWeek = "Sunday";
      } else {
        this.firstDayOfTheWeek = "Monday";
      }
    },
  },
});
</script>

<style lang="scss">
@import "./Calendar.scss";
</style>
