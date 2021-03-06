<template>
  <div class="app-calendar-row position-relative"
       v-bind:name="inputCalendarSelector"
       v-bind:class="inputShowTitle ? 'app-calendar-row-title-bar' : ''">
    <b-form-group class="input-with-icon"
                  v-bind:label="inputTitle"
                  v-if="inputShowTitle">
      <b-input-group>
        <b-form-input type="text"
                      readonly
                      autocomplete="off"
                      v-model="calendar.text"
                      class="app-calendar-input bg-white rounded-0"
                      v-bind:horizontal="inputHorizontalView"
                      v-bind:placeholder="inputShowPlaceholder ? placeholder : ''"
                      v-on:click.native.prevent="toggleCalendar($event)">
        </b-form-input>
        <b-input-group-text slot="prepend"
                            class="large px-2 py-1 radius-right"
                            v-if="inputShowIcons">
          <i class="far fa-calendar-alt app-calendar-icon"></i>
        </b-input-group-text>
        <b-input-group-text slot="append"
                            class="large px-2 py-1 radius-left pointer"
                            v-on:click="removeDate()"
                            v-if="calendar.text && inputShowIcons && inputShowRemoveIcon">
          <i class="fa fa-times app-calendar-icon"></i>
        </b-input-group-text>
      </b-input-group>
    </b-form-group>
    <b-input-group v-else-if="calendar.text && inputShowIcons && inputShowRemoveIcon">
      <b-form-input readonly
                    type="text"
                    autocomplete="off"
                    v-model="calendar.text"
                    class="app-calendar-input bg-white radius-right"
                    v-bind:placeholder="inputShowPlaceholder ? placeholder : ''"
                    v-on:click.native.prevent="toggleCalendar($event)">
      </b-form-input>
      <b-input-group-text slot="append"
                          class="large px-2 py-1 radius-left pointer"
                          v-on:click="removeDate()">
        <i class="fa fa-times app-calendar-icon"></i>
      </b-input-group-text>
    </b-input-group>
    <b-form-input v-else
                  readonly
                  type="text"
                  autocomplete="off"
                  v-model="calendar.text"
                  class="app-calendar-input bg-white"
                  v-bind:placeholder="inputShowPlaceholder ? placeholder : ''"
                  v-on:click.native.prevent="toggleCalendar($event)">
    </b-form-input>

    <b-card no-body
            class="app-calendar shadow mb-3"
            v-bind:id="inputCalendarSelector"
            v-bind:ref="inputCalendarSelector"
            v-if="flags.showCalendar && flags.initialCalendar">
      <div class="card-body p-2">
        <div v-show="flags.showDates">
          <b-row>
            <b-col cols="2">
              <b-button block
                        size="lg"
                        variant="light"
                        class="bg-gray border-0 py-1 px-0 shadow-none text-white"
                        v-bind:disabled="date <= inputMinYear"
                        v-on:click="goToPreviousMonth()">
                <i class="fa fa-arrow-right pr-1 ml-1 app-calendar-icon"></i>
              </b-button>
            </b-col>
            <b-col cols="4"
                   class="p-0">
              <b-button block
                        size="lg"
                        variant="primary"
                        v-bind:style="{ backgroundColor: inputPrimaryColor}"
                        class="rounded-0 border-0 py-1 shadow-none radius-right"
                        v-on:click="showMonths(monthInfo.firstDayOfMonth)">
                {{monthInfo.title}}
              </b-button>
            </b-col>
            <b-col cols="4"
                   class="p-0">
              <b-button block
                        size="lg"
                        variant="primary"
                        v-bind:style="{ backgroundColor: inputPrimaryColor}"
                        class="rounded-0 border-0 py-1 shadow-none radius-left"
                        v-on:click="showYears()">
                {{yearInfo}}
              </b-button>
            </b-col>
            <b-col cols="2">
              <b-button block
                        size="lg"
                        variant="light"
                        class="bg-gray border-0 py-1 px-0 shadow-none text-white"
                        v-bind:disabled="date >= inputMaxYear"
                        v-on:click="goToNextMonth()">
                <i class="fa fa-arrow-left pl-1 mr-1 app-calendar-icon"></i>
              </b-button>
            </b-col>
          </b-row>
          <b-row class="justify-content-center my-2">
            <b-col cols="12">
              <ul class="list-unstyled show-weekdays float-right p-0">
                <li v-for="(day, index) in daysInPersian" v-bind:key="index">
                  {{day.title}}
                </li>
              </ul>
              <ul class="list-unstyled show-days float-right p-0">
                <li v-for="(day, index) in dayArray"
                    v-bind:key="index"
                    v-bind:title="day.isToday && inputHighlightToday ? 'امروز' : ''"
                    v-bind:class="[
                                        day.isHoliday ? 'day-isHoliday text-danger' : '',
                                        day.isToday && inputHighlightToday ? 'day-isToday' : '',
                                        inputMinDate && day.dateFormat < inputMinDate ? 'day-deactivate' : '',
                                        inputMaxDate && day.dateFormat > inputMaxDate ? 'day-deactivate' : '',
                                        inputMinYear && day.dateFormat < inputMinYear ? 'day-deactivate' : '',
                                        inputMaxYear && day.dateFormat > inputMaxYear ? 'day-deactivate' : '',
                                        day.isGrey && !inputShowNextMonth ? 'hide-other-month' : '',
                                        day.isGrey ? 'day-isGrey' : '',
                                        day.isSelected ? 'day-isSelected' : '']"
                    v-bind:style="{ backgroundColor: day.isSelected ? inputPrimaryColor : ''}"
                    v-on:click="checkSelectable(day) ? selectDay(day) : ''">
                  {{day.title}}
                </li>
              </ul>
            </b-col>
          </b-row>
          <b-row class="mb-2 position-relative"
                 v-if="inputShowTime">
            <b-col cols="6">
              <b-form-input min="0"
                            max="60"
                            type="text"
                            tabindex="2"
                            class="text-center"
                            autocomplete="off"
                            placeholder="دقیقه"
                            v-model="calendar.minute"
                            v-on:change="changeMinute()">
              </b-form-input>
            </b-col>
            <b-col cols="1"
                   class="position-absolute app-calendar-time-separator">
              :
            </b-col>
            <b-col cols="6">
              <b-form-input min="0"
                            max="24"
                            type="text"
                            tabindex="1"
                            class="text-center"
                            autocomplete="off"
                            placeholder="ساعت"
                            v-model="calendar.hour"
                            v-on:change="changeHour()">
              </b-form-input>
            </b-col>
          </b-row>
          <b-row>
            <template v-if="inputShowFooterButtons">
              <b-col v-if="checkSelectable(today)">
                <b-button block
                          variant="light"
                          v-on:click="goToToday()">
                  امروز
                </b-button>
              </b-col>
              <b-col v-bind:class="checkSelectable(today) ? 'p-0' : ''">
                <b-button block
                          variant="light"
                          v-on:click="goToCurrentMonth()">
                  ماه جــاری
                </b-button>
              </b-col>
            </template>
            <b-col>
              <b-button block
                        variant="light"
                        v-on:click="closeCalendar()"
                        tabindex="3">
                بستن
              </b-button>
            </b-col>
          </b-row>
        </div>
        <div v-show="flags.showMonths">
          <b-row>
            <b-col cols="12">
              <div class="bg-gray text-white rounded text-center mb-2 py-2">
                یک ماه را انتخاب کنید
              </div>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <ul class="list-unstyled mx-0 p-0 mb-2 w-100 float-right show-months">
                <li v-for="(month, index) in monthsInPersian"
                    v-bind:key="index"
                    v-on:click="selectMonth(month.firstDay)"
                    v-bind:style="{ backgroundColor: month.isSelected ? inputPrimaryColor : ''}"
                    v-bind:class="month.isSelected ? 'month-isSelected' : ''">
                  {{month.title}}
                </li>
              </ul>
            </b-col>
          </b-row>
          <b-row v-if="inputShowFooterButtons">
            <b-col cols="12"
                   v-if="checkSelectable(today)">
              <b-button block
                        variant="light"
                        v-on:click="goToToday()">
                امروز
              </b-button>
            </b-col>
          </b-row>
        </div>
        <div v-show="flags.showYears">
          <b-row>
            <b-col cols="12">
              <div class="bg-gray text-white rounded text-center mb-2 py-2">
                یک سال را انتخاب کنید
              </div>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <ul class="list-unstyled mx-0 mb-2 p-0 w-100 float-right show-year">
                <template v-for="(year, index) in yearsInPersian">
                  <li v-if="year.title"
                      v-bind:key="index"
                      v-on:click="selectYear(year.title)"
                      v-bind:style="{backgroundColor: year.isSelected ? inputPrimaryColor : ''}"
                      v-bind:class="year.isSelected ? 'year-isSelected' : ''">
                    {{year.title}}
                  </li>
                </template>
              </ul>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <b-button block
                        variant="light"
                        v-on:click="showYearGoUp()">
                <i class="fas fa-angle-up app-calendar-icon"></i>
              </b-button>
            </b-col>
            <b-col class="p-0"
                   v-if="inputShowFooterButtons && checkSelectable(today)">
              <b-button block
                        variant="light"
                        v-on:click="goToToday()">
                امروز
              </b-button>
            </b-col>
            <b-col>
              <b-button block
                        variant="light"
                        v-on:click="showYearGoDown()">
                <i class="fas fa-angle-down app-calendar-icon"></i>
              </b-button>
            </b-col>
          </b-row>
        </div>
      </div>
    </b-card>
  </div>
</template>

<script type="text/javascript">
  export default {
    name: 'CalendarComponent',
    props: {
      inputMaxYear: {
        type: Date,
        required: false
      },
      inputMinYear: {
        type: Date,
        required: false
      },
      inputMinDate: {
        type: Date,
        required: false
      },
      inputMaxDate: {
        type: Date,
        required: false
      },
      inputShowFooterButtons: {
        type: Boolean,
        default: true,
        required: false
      },
      inputHorizontalView: {
        type: Boolean,
        default: false,
        required: false
      },
      inputHighlightToday: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowNextMonth: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowTime: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowIcons: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowRemoveIcon: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowPlaceholder: {
        type: Boolean,
        default: true,
        required: false
      },
      inputAllowEmpty: {
        type: Boolean,
        default: true,
        required: false
      },
      inputShowTitle: {
        type: Boolean,
        default: true,
        required: false
      },
      inputPlaceholder: {
        type: String,
        required: false
      },
      inputTitle: {
        type: String,
        required: false
      },
      inputSelectedDate: {
        type: Date,
        required: false
      },
      inputPrimaryColor: {
        type: String,
        default: '#007bff',
        required: false
      },
      inputCalendarSelector: {
        type: String,
        required: false,
        default: 'app-calendar'
      },
      timeZone: {
        type: String,
        required: false,
        default: 'UTC'
      }
    },
    computed: {
      placeholder() {
        return this.inputPlaceholder || 'تاریخ مورد نظر خود را انتخاب کنید';
      }
    },
    data() {
      return {
        date: new Date(),
        today: new Date(),
        dayArray: [],
        yearToShow: 0,
        yearInfo: null,
        monthInfo: null,
        yearsInPersian: null,
        monthsInPersian: [
          { index: 0, title: 'فروردین', days: 31, isSelected: false, firstDay: '' },
          { index: 1, title: 'اردیبهشت', days: 31, isSelected: false, firstDay: '' },
          { index: 2, title: 'خرداد', days: 31, isSelected: false, firstDay: '' },
          { index: 3, title: 'تیر', days: 31, isSelected: false, firstDay: '' },
          { index: 4, title: 'مرداد', days: 31, isSelected: false, firstDay: '' },
          { index: 5, title: 'شهریور', days: 31, isSelected: false, firstDay: '' },
          { index: 6, title: 'مهر', days: 30, isSelected: false, firstDay: '' },
          { index: 7, title: 'آبان', days: 30, isSelected: false, firstDay: '' },
          { index: 8, title: 'آذر', days: 30, isSelected: false, firstDay: '' },
          { index: 9, title: 'دی', days: 30, isSelected: false, firstDay: '' },
          { index: 10, title: 'بهمن', days: 30, isSelected: false, firstDay: '' },
          { index: 11, title: 'اسفند', days: 29, isSelected: false, firstDay: '' }
        ],
        daysInPersian: [
          { index: 0, title: 'شــنــبــــــه', searchable: 'شنبه' },
          { index: 1, title: 'یــکشـنبــه', searchable: 'یکشنبه' },
          { index: 2, title: 'دوشـنبــــه', searchable: 'دوشنبه' },
          { index: 3, title: 'سه‌شنبـــه', searchable: 'سه‌شنبه' },
          { index: 4, title: 'چهارشنبـه', searchable: 'چهارشنبه' },
          { index: 5, title: 'پنجـشنبــه', searchable: 'پنجشنبه' },
          { index: 6, title: 'جـمــعـــــــه', searchable: 'جمعه' }
        ],
        calendar: {
          show: false,
          text: '',
          hour: 0,
          minute: 0
        },
        flags: {
          showDates: true,
          showYears: false,
          showMonths: false,
          initialCalendar: false,
          showCalendar: false,
          clickOut: false
        }
      };
    },
    watch: {
      'inputSelectedDate': function () {
        this.fillData();
      }
    },
    mounted() {
      this.fillData();
    },
    methods: {
      toggleCalendar(e) {
        e.preventDefault();
        this.flags.showCalendar = !this.flags.showCalendar;
        this.flags.clickOut = false;

        if (this.flags.showCalendar) {
          document.addEventListener('click', this.outsideClickListener);
        } else {
          document.removeEventListener('click', this.outsideClickListener);
        }
      },
      outsideClickListener(event) {
        if (this.clickOutside(event) && this.flags.clickOut) {
          this.flags.showCalendar = false;
          this.flags.clickOut = false;
          return;
        }

        this.flags.clickOut = true;
      },
      clickOutside(event) {
        const reference = this.$refs[String(this.inputCalendarSelector)];
        const condition = reference && reference.contains(event.target);
        return !condition;
      },

      /**
       * close all 'app-calendar' panels
       */
      closeCalendar() {
        this.flags.showDates = true;
        this.flags.showYears = false;
        this.flags.showMonths = false;
        this.flags.showCalendar = false;
      },
      fillData() {
        /**
         * Decide to fill input
         */
        if (this.inputSelectedDate) {
          if (this.checkSelectable(this.inputSelectedDate)) {
            this.date = this.convertToDate(String(this.inputSelectedDate));
            this.calendar.text = this.convertToPersianDate(this.date, 'combo');
          } else {
            this.$emit('error', 'تاریخ انتخابی در بازه تعیین شده نمی‌باشد!');
          }
        } else {
          this.calendar.text = '';

          /**
           * Calculate First Day Of Year
           */
          if (this.inputMaxYear) {
            const persianYear = Number(this.getPersianYear(new Date()));
            const persianMaxYear = Number(this.getPersianYear(this.inputMaxYear));
            const abs = persianYear - persianMaxYear;
            const newDate = new Date();
            newDate.setFullYear(newDate.getFullYear() - abs);

            this.date = this.getFirstDayOfYear(newDate);
          } else if (this.inputMaxDate) {
            this.date = this.inputMaxDate;
          }
        }
        this.createCalendar();
        this.checkLeapYear();
      },
      removeDate() {
        this.calendar.text = '';
        this.$emit('changeDate', null);
      },
      selectDay(day) {
        /**
         * reset all selected highlights
         */
        this.dayArray.forEach(item => {
          item.isSelected = false;
        });

        /**
         * highlight the selected day
         */
        day.isSelected = true;

        /**
         * update date
         */
        const date = this.convertToDate(day.dateFormat);
        this.date = date;

        /**
         * update models
         */
        this.updateModel(date);

        /**
         * update parent
         */
        this.$emit('changeDate', date);

        /**
         * close calendar
         */
        this.closeCalendar();
      },
      checkSelectable(day) {
        if (this.inputMaxYear && ((day.dateFormat || day) > this.inputMaxYear)) {
          return false;
        }
        if (this.inputMinYear && ((day.dateFormat || day) < this.inputMinYear)) {
          return false;
        }

        if (this.inputMinDate && ((day.dateFormat || day) < this.inputMinDate)) {
          return false;
        }

        return !(this.inputMaxDate && ((day.dateFormat || day) > this.inputMaxDate));
      },
      goToToday() {
        if (this.checkSelectable(this.today)) {
          this.selectDay({ dateFormat: this.today });
        }
      },
      goToNextMonth() {
        this.checkLeapYear();
        this.date.setDate(this.date.getDate() + Number(this.monthInfo.days));

        /**
         * update calendar's items
         */
        this.createCalendar();
      },
      goToCurrentMonth() {
        this.date = new Date();
        this.checkLeapYear();
        this.date.setDate(this.date.getDate());

        /**
         * update calendar's items
         */
        this.createCalendar();
      },
      goToPreviousMonth() {
        this.checkLeapYear();
        this.date.setDate(this.date.getDate() - Number(this.monthInfo.previousMonthAmount));

        /**
         * update calendar's items
         */
        this.createCalendar();
      },
      createCalendar(date) {
        let firstDayOfMonth;
        let persianWeekday;
        let persianWeekdayIndex;
        let counter = 1;

        if (!date) {
          date = this.date ? this.date : new Date();
        }

        /**
         * reset model
         */
        this.resetData();

        firstDayOfMonth = this.getFirstDayOfMonth(date);
        persianWeekday = this.getPersianWeekday(firstDayOfMonth);
        this.yearInfo = this.getPersianYear(firstDayOfMonth); // !! should be above of month
        this.monthInfo = this.fillMonthInfo(firstDayOfMonth);

        /**
         * create weekday
         */
        this.daysInPersian.map((item, index) => {
          if (item.searchable === persianWeekday) {
            persianWeekdayIndex = index + 1;
          }
        });

        /**
         * create column 1 (previous month)
         */
        for (let row = 1; row < persianWeekdayIndex; row++) {
          const column = 1;
          const formula = ((row - 1) * 6) + column - 1;
          const dataArray = this.dayArray[formula];
          const dateFormat = this.convertToDate(firstDayOfMonth);

          if (formula > 42 || counter > this.monthInfo.previousMonthAmount) {
            break;
          }
          dataArray.isGrey = true;
          dataArray.title = this.monthInfo.previousMonthAmount - persianWeekdayIndex + row + 1;
          dataArray.dateFormat = this.convertToDate(dateFormat.setDate(firstDayOfMonth.getDate() - persianWeekdayIndex + row));
        }

        /**
         * create column 2 (this month)
         */
        for (let row = persianWeekdayIndex; row <= 7; row++) {
          const column = 1;
          const formula = ((row - 1) * 6) + column - 1;
          const dataArray = this.dayArray[formula];
          const dateFormat = this.convertToDate(firstDayOfMonth);

          if (formula > 42 || counter > this.monthInfo.days) {
            break;
          }

          dataArray.title = counter;
          dataArray.dateFormat = this.convertToDate(dateFormat.setDate(firstDayOfMonth.getDate() + counter - 1));

          if (row === 7) {
            dataArray.isHoliday = true;
          }
          if (this.isCounterSelected(counter)) {
            dataArray.isSelected = true;
          }
          if (this.isCounterTodayDate(counter)) {
            dataArray.isToday = true;
          }
          counter++;
        }

        /**
         * create column 2 to 7 (this month and next month)
         */
        for (let column = 2, nextMonthCounter = 1; column <= 6; column++) {
          for (let row = 1; row <= 7; row++) {
            const formula = ((row - 1) * 6) + column - 1;
            const dataArray = this.dayArray[formula];
            const dateFormat = this.convertToDate(firstDayOfMonth);

            if (formula > 42) {
              break;
            }
            if (counter > this.monthInfo.days) {
              const nexMonthDateFormat = this.convertToDate(firstDayOfMonth);
              if (row === 7) {
                dataArray.isHoliday = true;
              }
              dataArray.title = nextMonthCounter;
              dataArray.isGrey = true;
              dataArray.dateFormat = this.convertToDate(nexMonthDateFormat.setDate(firstDayOfMonth.getDate() + nextMonthCounter + this.monthInfo.days - 1));

              nextMonthCounter++;
              continue;
            }

            dataArray.title = counter;
            dataArray.dateFormat = this.convertToDate(dateFormat.setDate(firstDayOfMonth.getDate() + counter - 1));

            if (row === 7) {
              dataArray.isHoliday = true;
            }
            if (this.isCounterTodayDate(counter)) {
              dataArray.isToday = true;
            }
            if (this.isCounterSelected(counter)) {
              dataArray.isSelected = true;
            }
            counter++;
          }
        }

        this.flags.initialCalendar = true;
      },
      updateModel(date) {
        this.calendar.hour = this.getHour(date);
        this.calendar.minute = this.getMinute(date);
        this.calendar.text = this.convertToPersianDate(date, 'combo');
      },
      checkLeapYear() {
        const date = this.date;
        if (this.isLeapYear(this.getPersianYear(date))) {
          this.monthsInPersian[11].days = 30;
        } else {
          this.monthsInPersian[11].days = 29;
        }
      },
      isCounterSelected(counter) {
        if (!this.inputSelectedDate) {
          return false;
        }

        const dateDay = this.getPersianDay(this.inputSelectedDate);
        const dateMonth = this.getPersianMonth(this.inputSelectedDate);
        const dateYear = this.getPersianYear(this.inputSelectedDate);
        counter = String(counter);

        return (counter === dateDay && this.monthInfo.title === dateMonth && this.yearInfo === dateYear);
      },
      isCounterTodayDate(counter) {
        const dateDay = this.getPersianDay(new Date());
        const dateMonth = this.getPersianMonth(new Date());
        const dateYear = this.getPersianYear(new Date());
        counter = String(counter);

        return (counter === dateDay && this.monthInfo.title === dateMonth && this.yearInfo === dateYear);
      },
      resetData() {
        this.dayArray = [];
        for (let i = 0; i < 42; i++) {
          this.dayArray.push({
            title: null,
            isHoliday: false,
            isGrey: false,
            isToday: false,
            isSelected: false,
            dateFormat: null
          });
        }
      },
      getFirstDayOfMonth(date) {
        const firstDay = this.convertToDate(date);
        const pastDays = Number(this.getPersianDay(date));
        firstDay.setDate(firstDay.getDate() - pastDays + 1);

        return firstDay;
      },
      getFirstDayOfYear(date) {
        let pastDays = 0;

        date = this.getFirstDayOfMonth(date);
        const getMonth = this.getPersianMonth(this.convertToDate(date), '2-digit');
        const getMonthIndex = Number(this.convertToEnglishDigit(getMonth)) - 1;

        for (let index = 0; index < getMonthIndex; index++) {
          pastDays += Number(this.monthsInPersian[index].days);
        }

        return this.convertToDate(date.setDate(date.getDate() - pastDays));
      },
      fillMonthInfo(date) {
        let persianDate,
          monthIndex,
          daysAmount,
          monthName,
          nexMonthAmount,
          previousMonthAmount;

        persianDate = this.getPersianMonth(date);

        this.monthsInPersian.map((item, index) => {
          if (this.monthsInPersian[index].title === persianDate) {
            monthIndex = index;
            daysAmount = item.days;
            monthName = item.title;
            nexMonthAmount = this.monthsInPersian[index + 1 > 11 ? 0 : index + 1].days;
            previousMonthAmount = this.monthsInPersian[index - 1 < 0 ? 11 : index - 1].days;
          }
        });

        if (this.isLeapYear(Number(this.yearInfo) - 1) && Number(monthIndex) === 0) {
          previousMonthAmount = 30;
        }

        if (this.isLeapYear(this.yearInfo) && Number(monthIndex) === 11) {
          daysAmount = 30;
        }

        if (this.isLeapYear(this.yearInfo) && Number(monthIndex) === 10) {
          nexMonthAmount = 30;
        }

        return {
          index: monthIndex,
          number: Number(monthIndex) + 1,
          title: monthName,
          days: daysAmount,
          nexMonthAmount: nexMonthAmount,
          previousMonthAmount: previousMonthAmount,
          firstDayOfMonth: date
        };
      },
      isLeapYear(year) {
        return ((((((year - ((year > 0) ? 474 : 473)) % 2820) + 474) + 38) * 682) % 2816) < 682;
      },
      getPersianDay(date, opt) {
        const options = { day: opt || 'numeric', timeZone: this.timeZone };
        return this.convertToEnglishDigit(date.toLocaleDateString('fa-persian', options));
      },
      getPersianWeekday(date, opt) {
        const options = { weekday: opt || 'long', timeZone: this.timeZone };
        return date.toLocaleDateString('fa-persian', options);
      },
      getPersianMonth(date, opt) {
        const options = { month: opt || 'long', timeZone: this.timeZone };
        return date.toLocaleDateString('fa-persian', options);
      },
      getPersianYear(date, opt) {
        const options = { year: opt || 'numeric', timeZone: this.timeZone };
        return this.convertToEnglishDigit(date.toLocaleDateString('fa-persian', options));
      },
      getHour(date) {
        const options = { hour: '2-digit', timeZone: this.timeZone };
        const output = date.toLocaleString('fa-persian', options);
        if (output.length <= 1) {
          return '0' + output;
        }
        return output;
      },
      getMinute(date) {
        const options = { minute: '2-digit', timeZone: this.timeZone };
        const output = date.toLocaleString('fa-persian', options);
        if (output.length <= 1) {
          return '0' + output;
        }
        return output;
      },
      convertToPersianDate(date, type) {
        if (!date) {
          return;
        }
        if (!type) {
          type = 'combo';
        }
        const year = this.convertToEnglishDigit(this.getPersianYear(date, 'numeric'));
        const weekday = this.getPersianWeekday(date, 'long');
        const hour = this.getHour(date);
        const minute = this.getMinute(date);

        let month = this.convertToEnglishDigit(this.getPersianMonth(date, '2-digit'));
        let day = this.getPersianDay(date, '2-digit');

        if (day.length <= 1) {
          day = '0' + day;
        }
        if (month.length <= 1) {
          month = '0' + month;
        }
        if (type === 'combo') {
          let output;
          month = this.getPersianMonth(date, 'long');
          output = `${weekday} ${day} ${month} ${year}`;
          if (this.inputShowTime) {
            output += ` - ${hour}:${minute}`;
          }
          return output;
        }
        return `${year}/${month}/${day}`;
      },
      convertToEnglishDigit(string) {
        return string.replace(/[\u0660-\u0669]/g, (c) => {
          return c.charCodeAt(0) - 0x0660;
        }).replace(/[\u06f0-\u06f9]/g, (c) => {
          return c.charCodeAt(0) - 0x06f0;
        });
      },
      convertToDate(input) {
        const date = new Date(input);
        date.toDateString();
        date.toGMTString();

        return date;
      },

      /**
       * Selecting a month and a year together
       */
      showMonths(firstDayOfMonth) {
        /**
         * Month Index Selected
         */
        let differenceYear;
        let firstDayOfYear;
        let increasingDay = 0;

        if (!firstDayOfMonth) {
          firstDayOfMonth = this.monthInfo.firstDayOfMonth;
        }

        /**
         * Reset is selected
         */
        this.monthsInPersian.forEach(item => {
          item.isSelected = false;
        });

        /**
         * Find and select month
         */
        for (const i in this.monthsInPersian) {
          if (this.monthsInPersian.hasOwnProperty(i)) {
            if (this.monthsInPersian[i].title === this.getPersianMonth(firstDayOfMonth)) {
              this.monthsInPersian[i].isSelected = true;
              break;
            }
          }
        }

        /**
         * Calculate days after changing year
         */
        firstDayOfMonth = this.monthInfo.firstDayOfMonth;
        firstDayOfYear = this.convertToDate(this.getFirstDayOfYear(firstDayOfMonth));

        if (this.yearInfo !== Number(this.getPersianYear(firstDayOfYear))) {
          differenceYear = this.yearInfo - Number(this.getPersianYear(firstDayOfYear));
          this.date = this.convertToDate(firstDayOfYear.setFullYear(firstDayOfYear.getFullYear() + differenceYear));
          this.yearInfo = Number(this.getPersianYear(this.date));
        }

        /**
         * Find and select month
         */
        this.monthsInPersian.forEach(item => {
          item.firstDay = this.convertToDate(firstDayOfYear.setDate(firstDayOfYear.getDate() + increasingDay));
          increasingDay = item.days;
        });

        /**
         * show other panel
         */
        this.flags.showDates = false;
        this.flags.showYears = false;
        this.flags.showMonths = true;
      },

      /**
       *
       * @param firstDayOfMonth
       */
      selectMonth(firstDayOfMonth) {
        /**
         * Update date and month
         */
        this.date = this.convertToDate(firstDayOfMonth);
        this.monthInfo = this.fillMonthInfo(this.date);

        /**
         * Show other panel
         */
        this.flags.showMonths = false;
        this.flags.showYears = false;
        this.flags.showDates = true;

        /**
         * Create calendar
         */
        this.createCalendar();
      },

      /**
       *
       */
      showYears() {
        /**
         * Create year array
         */
        this.yearsInPersian = [];
        for (let i = (this.yearToShow - 7); i <= (this.yearToShow + 7); i++) {
          if (this.inputMaxYear || this.inputMinYear) {
            if (Number(this.yearInfo) + i > this.getPersianYear(this.inputMaxYear) || Number(this.yearInfo) + i < this.getPersianYear(this.inputMinYear)) {
              continue;
            }
          }

          const generatedYear = Number(this.yearInfo) + i;
          this.yearsInPersian.push({
            title: generatedYear,
            isSelected: false
          });
        }

        /**
         * Find and select year
         */
        for (const i in this.yearsInPersian) {
          if (this.yearsInPersian.hasOwnProperty(i)) {
            if (String(this.yearsInPersian[i].title) === this.yearInfo) {
              this.yearsInPersian[i].isSelected = true;
              break;
            }
          }
        }

        /**
         * Show other panel
         */
        this.flags.showDates = false;
        this.flags.showMonths = false;
        this.flags.showYears = true;
      },

      /**
       *
       * @param year
       */
      selectYear(year) {
        /**
         * show other panel
         */
        this.flags.showDates = false;
        this.flags.showYears = false;
        this.flags.showMonths = true;

        /**
         * update year info
         */
        this.yearInfo = year;

        /**
         * create month panel
         */
        this.showMonths();
      },

      /**
       *
       */
      showYearGoUp() {
        /**
         * check inputMinYear
         * @type {number}
         */
        if (this.inputMinYear && this.yearsInPersian[0].title <= this.getPersianYear(this.inputMinYear)) {
          return;
        }

        this.yearToShow -= 6;
        this.showYears(this.yearToShow);
      },

      /**
       *
       */
      showYearGoDown() {
        /**
         * check inputMaxYear
         * @type {number}
         */
        const index = this.yearsInPersian.length - 1;
        if (this.inputMaxYear && this.yearsInPersian[index].title >= this.getPersianYear(this.inputMaxYear)) {
          return;
        }

        this.yearToShow += 6;
        this.showYears(this.yearToShow);
      },

      /**
       * selecting an hour and minute
       */
      changeHour() {
        const selectedDate = this.inputSelectedDate;
        const newHour = this.convertToEnglishDigit(this.calendar.hour);
        const date = this.convertToDate(selectedDate.setHours(newHour));
        this.monthInfo = this.fillMonthInfo(date);

        /**
         * update parents
         */
        this.$emit('changeDate', date);

        /**
         * update models
         */
        this.updateModel(date);

        /**
         * update calendar's items
         */
        this.createCalendar(date);
        this.date = date;
      },

      /**
       *
       */
      changeMinute() {
        const selectedDate = this.inputSelectedDate;
        const newMinute = this.convertToEnglishDigit(this.calendar.minute);
        const date = this.convertToDate(selectedDate.setMinutes(newMinute));
        this.monthInfo = this.fillMonthInfo(date);

        /**
         * update parents
         */
        this.$emit('changeDate', date);

        /**
         * update models
         */
        this.updateModel(date);

        /**
         * update calendar's items
         */
        this.createCalendar(date);
        this.date = date;
      }
    }
  };
</script>
