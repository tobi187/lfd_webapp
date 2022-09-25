<template>
    <div>
        <form>
            <div class="center-me go-under">
                <select v-model="selected" class="default-alignment">
                    <option v-for="job in options" v-bind:value="{val: job.value, nam: job.name}">
                        {{ job.name }}
                    </option>
                </select>
                <div class="default-alignment">
                    <input type="file" @change="uploadFile">
                </div>
                <div v-if="files.length > 1">
                    Upload: 
                </div>
                <button @click="sendPost" class="default-alignment">
                    Senden
                </button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const selected = ref({})
const files = ref([])
const options = ref([
    {"name": "Systemintegration", "value": "systemIntegration"},
    {"name": "Anwendungsentwicklung", "value": "coding"},
    {"name": "Lager", "value": "lager"}
])

function uploadFile(e) {
    var uFiles = e.target.files || e.dataTransfer.files;
    if (!uFiles.length)
        return;
    if (files.value.length > 0)
        files.value = []
    files.value.push(uFiles[0])
    console.log(files.value)
}

function sendPost() {
    console.log(selected.value.val)
    const requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ job: selected.value.val })
    };
    fetch("http://localhost:5000/api/v1/init", requestOptions)
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

.go-under {
    display: flex;
    flex-direction: column;
} 

.default-alignment {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 15px 5px;
}

</style>