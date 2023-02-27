<script setup>
import { ref } from 'vue'
import axios from 'axios'

const user = ref({});
const isOpen = ref(true);

axios.get('https://cloudmanlabs.api.stdlib.com/challenge/user/')
    .then(response => {
        Object.assign(user.value, response.data);
    })
    .catch(error => { console.log(error) });

const openMenu = () => {
    isOpen.value = !isOpen.value;
}
</script>

<template>
    <div class="btn" :class="{btnClick: isOpen}">
        <span @click="openMenu" class="material-symbols-outlined">
            menu
        </span>
    </div>
    <nav class="sidebar flex-space-between" :class="{sidebarShow: isOpen}">
        <div>
            <div class="flex-align-center pb-30">
                <img alt="User icon" src="../assets/img/user.png" class="user-photo" width="35" height="35" />
                <div><h3>{{ user.name }} {{ user.surname }}</h3></div>
            </div>
            <div class="pb-30 flex-column">
                <span class="bold">Cargo</span>
                <span>{{ user.position }}</span>
            </div>
            <div class="pb-30 flex-column">
                <span class="bold">E-mail</span>
                <span>{{ user.email }}</span>
            </div>
            <div class="pb-30 flex-column">
                <span class="bold">Teléfono</span>
                <span>{{ user.phone }}</span>
            </div>
            <div class="pb-30 mb-10 flex-column">
                <span class="bold">Días de vacaciones restantes</span>
                <span>{{ user.vacationDays }}</span>
            </div>
            <a href="mailto:soporte@cloudmanlabs.com" class="errBtn">
                Informar de un error
            </a>
        </div>
        <div class="sidebarEnd">
            <img alt="Cloud Man Labs logo" src="../assets/img/logo_web.svg" width="110" height="110" />
        </div>
    </nav>
</template>
