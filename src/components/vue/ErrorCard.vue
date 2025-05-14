<template>
    <div class="error-card" v-for="error in errors"></div>
</template>
<script>
export default {
    data() {
        return {
            errors: [],
        };
    },
};
</script>
<style>
.error-card {
    background-color: #eee;
}
</style>
