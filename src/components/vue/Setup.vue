<template>
    <div>
        <h1>user: {{ user.value }}</h1>
        <h1>Please enter your API settings..</h1>
        <div>Domain: <input type="text" name="api_url" id="api_url" /></div>
        <div>API Key: <input type="text" name="api_key" id="api_key" /></div>
    </div>
</template>
<script>
import { Preferences } from "@capacitor/preferences";
export default {
    data() {
        return {
            user: null,
        };
    },
    mounted() {
        console.log('hi');
        this.getUser();
        if(this.user == null){
            this.setUser();
        }
    },
    methods: {
        async getUser() {
            this.user = await Preferences.get({ key: "user" });
            console.log(`set: ${this.user}`);
        },
        async setUser() {
            await Preferences.set({
                key: "user",
                value: "joe"
            });
            this.getUser();
        },
    },
};
</script>
<style></style>
