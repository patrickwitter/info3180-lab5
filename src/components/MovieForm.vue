<script setup>
    import { ref, onMounted } from "vue";
    var success = ref("");
    var hasErrors = ref("");
    var errors = ref([]);
    let csrf_token = ref("");
    function getCsrfToken() {
        fetch('/api/v1/csrf-token')
        .then((response) => response.json())
        .then((data) => {
            console.log(data);
            csrf_token.value = data.csrf_token;
        })
    }
    function saveMovie(){
        let movieForm = document.getElementById('movieForm');
        let form_data = new FormData(movieForm);
        fetch("/api/v1/movies", {
            method: 'POST',
            body: form_data,
            headers: {
                'X-CSRFToken': csrf_token.value
            }
        })
        .then(function (response) {
            return response.json();
        })
        .then(function (data) {
            if (data.data) {
                success.value = true;
                hasErrors.value = false;
                console.log(data.data);
            }
            if (data.errors) {
                hasErrors.value = true;
                success.value = false;
                errors.value = data.errors;
                console.log(data.errors);
                clearForm();
            }
        })
        .catch(function (error) {
            console.log(error);
        });
    }
    function clearForm(){
        var inputs = document.querySelectorAll('input');
        var textArea = document.querySelectorAll('textarea');
        inputs.forEach(input =>  input.value = '');
        textArea.forEach(input =>  input.value = '');
    }
    onMounted(() => {
        clearForm();
        getCsrfToken();
    });
</script>

<template>
    <div class="form-container">
    <h2>Upload Form</h2>
    <div v-if="success" class="alert alert-success">File Upload Successful</div>
    <div v-if="hasErrors" class="alert alert-danger">
        <ul>
            <li v-for="err in errors">{{ err }}</li>
        </ul>    
    </div>
    <form @submit.prevent="saveMovie" enctype="multipart/form-data" id="movieForm">
        <div class="form-group mb-3">
        <label for="title" class="form-label">Movie Title</label>
        <input type="text" name="title" class="form-control" />
        </div>
        <div class="form-group mb-3">
        <label for="description" class="form-label">Description</label>
        <textarea name="description" class="form-control" rows="3"></textarea>
        </div>
        <div class="form-group mb-3">
        <label for="poster" class="form-label">Photo Upload</label>
        <input type="file" name="poster" class="form-control" />
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    </div>
</template>

<style scoped>
    div.form-container {
        width: auto;
        margin-left: 50px;
        margin-right: 50px;
    }
</style>