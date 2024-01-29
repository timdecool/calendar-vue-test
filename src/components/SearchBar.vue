<script setup>
import { ref } from 'vue'
import axios from 'axios' 

const query = ref("")
const results = ref([])

function searchUsers() {
    if(query.value !== "") {
        axios.get(`http://127.0.0.1:8000/api/user/search?q=${query.value}`)
            .then(function (response) {
                results.value = response.data
            })
            .catch(function(error) {
                console.log(error)
            })
    }
    else
    {
        results.value = []
    }
}
</script>

<template>
    <input type="text" v-model="query" @input="searchUsers">
    <ul>
        <li v-for="result in results">
            <span v-if="result.firstName">
                {{ result.firstName }} {{ result.lastName }}
            </span>
            <span v-else>
                {{ result.email }}
            </span>
        </li>
    </ul>


</template>