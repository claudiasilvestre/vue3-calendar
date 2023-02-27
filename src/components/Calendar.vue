<script setup>
import { ref } from 'vue'
import axios from 'axios'
import Modal from './Modal.vue'

const date = ref(new Date());
const currYear = ref(date.value.getFullYear());
const currMonth = ref(date.value.getMonth());
const firstDayofMonth = ref(0);
const lastDateofMonth = ref(null);
const lastDayofMonth = ref(0);
const lastDateofLastMonth = ref(null);
const days = ref([]);
const isToday = ref([]);
const daysOfCurrMonth = ref([]);
const holidays = ref([]);
const holidaysFetch = ref([]);
const holiday = ref(false);
const holidaysList = ref([])
const holidayObject = ref({});
const showModal = ref(false);
const modalObject = ref({})
const holidaysFetchCopy = ref([]);

const shortDaysWeek = ["SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"];

const months = ["January", "February", "March", "April", "May", "June", "July",
                "August", "September", "October", "November", "December"];

const currentDate = ref(`${months[currMonth.value]} ${currYear.value}`);

axios.get('https://cloudmanlabs.api.stdlib.com/challenge/holidays/')
    .then(response => {
        Object.assign(holidaysFetch.value, response.data);
    })
    .catch(error => { console.log(error) });

const renderCalendar = () => {
    days.value = []
    isToday.value = []
    daysOfCurrMonth.value = []
    holidays.value = []
    firstDayofMonth.value = new Date(currYear.value, currMonth.value, 1).getDay();
    lastDateofMonth.value = new Date(currYear.value, currMonth.value + 1, 0).getDate();
    lastDayofMonth.value = new Date(currYear.value, currMonth.value, lastDateofMonth.value).getDay();
    lastDateofLastMonth.value = new Date(currYear.value, currMonth.value, 0).getDate();

    // to show holidays list
    holidaysFetch.value.forEach(hol => {
        if (currYear.value === new Date(hol.day).getFullYear) {
            holidayObject.value = {...hol};
            holidayObject.value.day = new Date(holidayObject.value.day)
                                        .toLocaleDateString("es-ES", { day: 'numeric', month: 'long' });
            holidaysList.value.push(holidayObject.value);
        }
    });

    //creating previous month last days
    for (let i = firstDayofMonth.value; i > 0; i--) { 
        days.value.push(lastDateofLastMonth.value - i + 1);
        isToday.value.push(false);
        daysOfCurrMonth.value.push(false);
        holidays.value.push(false);
    }

    // creating all days of current month
    for (let i = 1; i <= lastDateofMonth.value; i++) {
        // adding active class if the current day, month and year matched
        isToday.value.push(i === date.value.getDate() && currMonth.value === new Date().getMonth()
                        && currYear.value === new Date().getFullYear());

        daysOfCurrMonth.value.push(true);

        // first check satudays and sundays, later holidays
        holiday.value = false

        if (new Date(currYear.value, currMonth.value, i).getDay() === 0 
                            || new Date(currYear.value, currMonth.value, i).getDay() === 6) {
            holidays.value.push(true);
        } else {
            holidaysFetch.value.forEach(hol => {
                if (new Date(hol.day).getDate() === i && new Date(hol.day).getMonth() === currMonth.value
                    && new Date(hol.day).getFullYear() === currYear.value) {
                        holiday.value = true;
                }
            });

            holidays.value.push(holiday.value);
        }

        days.value.push(i);
    }

    // creating next month first days
    for (let i = lastDayofMonth.value; i < 6; i++) { 
        days.value.push(i - lastDayofMonth.value + 1);
        isToday.value.push(false);
        daysOfCurrMonth.value.push(false);
        holidays.value.push(false);
    }
}

renderCalendar();

const prevNext = (type) => {
    currMonth.value = type === "prev" ? currMonth.value - 1 : currMonth.value + 1;

    if (currMonth.value < 0 || currMonth.value > 11) {
        date.value = new Date(currYear.value, currMonth.value);
        currYear.value = date.value.getFullYear();
        currMonth.value = date.value.getMonth();

        // to show holidays list
        holidaysList.value = []
        holidaysFetch.value.forEach(hol => {
            if (currYear.value === new Date(hol.day).getFullYear()) {
                holidayObject.value = {...hol};
                holidayObject.value.day = new Date(holidayObject.value.day)
                                            .toLocaleDateString("es-ES", { day: 'numeric', month: 'long' });
                holidaysList.value.push(holidayObject.value);
            }
        });

    } else {
        date.value = new Date()
    }

    currentDate.value = `${months[currMonth.value]} ${currYear.value}`;

    renderCalendar();
};

function formatDate(date) {
    var month = '' + (date.getMonth() + 1),
    day = '' + date.getDate(),
    year = date.getFullYear();

    if (month.length < 2) 
        month = '0' + month;
    if (day.length < 2) 
        day = '0' + day;

    return [year, month, day].join('-');
}

const holidayModal = (daySelected, index) => {
    if (holidays.value[index] && new Date(currYear.value, currMonth.value, daySelected).getDay() !== 0 
        && new Date(currYear.value, currMonth.value, daySelected).getDay() !== 6) {
        
        holidaysFetchCopy.value = JSON.parse(JSON.stringify(holidaysFetch.value));
        modalObject.value = holidaysFetchCopy.value
                            .find(({ day }) => day === formatDate(new Date(currYear.value, currMonth.value, daySelected)));
        modalObject.value.day = new Date(modalObject.value.day)
                                .toLocaleDateString("es-ES", { day: 'numeric', month: 'long', year: 'numeric' });

        showModal.value = true;
    }
}
</script>

<template>
    <div class="calendar-holidays">
        <div class="wrapper-calendar">
            <div>
                <h1>Calendario {{ currYear }}</h1>
                <div class="wrapper">
                    <div class="wrapper-header">
                        <p class="current-date">{{currentDate}}</p>
                        <div class="icons">
                            <span @click="prevNext('prev')" class="material-symbols-rounded">chevron_left</span>
                            <span @click="prevNext('next')" class="material-symbols-rounded">chevron_right</span>
                        </div>
                    </div>
                    <div class="calendar">
                        <ul class="weeks">
                            <li v-for="dayWeek in shortDaysWeek">
                                {{ dayWeek }}
                            </li>
                        </ul>
                        <ul class="days">
                            <li @click="holidayModal(day, index)" v-for="(day, index) in days" :class="{ active: isToday[index], 
                                                                inactive: !daysOfCurrMonth[index] ,
                                                                holidays: holidays[index]}">
                                {{day}}
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <div v-if="holidaysList.length > 0" class="holidays">
            <h3>Festivos {{ currYear }}</h3>
            <ul>
                <li v-for="holiday in holidaysList">
                    <div class="flex-column list-item">
                        <span><span class="bold">{{holiday.day}}</span> - {{holiday.description}}</span>
                        <span class="italic">{{holiday.type}}</span>
                    </div>
                </li>
            </ul>
        </div>
    </div>

    <Modal :show="showModal" @close="showModal = false">
        <div class="flex-column">
            <span class="bold">{{modalObject.day}}</span>
            <span>{{modalObject.description}}</span>
            <span class="italic">{{modalObject.type}}</span>
        </div>
    </Modal>
</template>
