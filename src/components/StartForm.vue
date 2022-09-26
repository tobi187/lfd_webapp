<template>
    <div>
        <div v-if="!isDataUploaded">
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
        </div>
        <div v-if="isDataUploaded">
            <div>
                Sätze:
                <ul>
                    <li v-for="point in berichtData.points">
                        <input type="text" v-model="point.lfd" />
                        {{ point.value }}
                    </li>
                </ul>
            </div>
            <div>
                <p style="white-space: pre-wrap;">
                    {{ berichtData.lfd_data }}
                </p>
            </div>
            <div>
                <button @click="sendPoints">Senden</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';

const selected = ref({})
const files = ref([])
const isDataUploaded = ref(false)

const berichtData = ref({
    lfd_data: {},
    points: [],
    fileName: ""
})

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

function sendPoints() {
    const requestOptions = {
        method: "POST", 
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            "data": berichtData.value.points, 
            "fileName": berichtData.value.fileName
        })
    };

    fetch("http://localhost:5000/api/v1/append", requestOptions)
        .then(res => res.json())
        .then(res => {
            const link = document.createElement("a")
            link.href = "http://localhost:5000/api/v1/dl/" + res["name"]
            link.download = "bheft.docx"
            link.click()
        })
}

async function fileToBase64(file) {
    
    function getBase64(file) {
        const reader = new FileReader()
        return new Promise(resolve => {
            reader.onload = ev => {
                const res = 
                    ev.target.result
                    .replace('data:', '')
                    .replace(/^.+,/, '') 
                resolve(res)
            }
            reader.readAsDataURL(file)
        })
    }
    
    const promises = files.value.map(el => getBase64(el))

    return await Promise.all(promises)
}

async function sendPost() {
    const requestOptions = {
        method: "POST", 
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            "job": selected.value.val,
            "files": await fileToBase64()
        })
    };
    fetch("http://localhost:5000/api/v1/init", requestOptions)
        .then(res => res.json())
        .then(res => {
            if (res["status"] === 200) {
                berichtData.value.points = res["points"]
                berichtData.value.lfd_data = res["data"]
                berichtData.value.fileName = res["file_name"]
                isDataUploaded.value = true
            }
            else {
                console.log(res)
            }
        })
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