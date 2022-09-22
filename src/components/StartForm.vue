<template>
    <div>
        <div class="center-me">
            <select v-model="selected">
                <option v-for="job in options" v-bind:value="{val: job.value, nam: job.name}">
                    {{ job.name }}
                </option>
            </select>
            <button @click="sendPost">
                Senden
            </button>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const selected = ref({})
const options = ref([
    {"name": "Systemintegration", "value": "systemintegration"},
    {"name": "Anwendungsentwicklung", "value": "coding"},
    {"name": "Lager", "value": "lager"}
])

function sendPost() {
    console.log(selected.value.val)
    const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ job: selected.value.val })
    };
    fetch("http://127.0.0.1:5000/api/v1/init", requestOptions)
        .then(res => res.json())
        .then(res => console.log(res))
}

</script>

<style scoped>

.center-me {
    display: flex;
    justify-items: center;
    align-items: center;
}

</style>