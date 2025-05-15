<template>
    <div>
        <h1>Enter your API settings..</h1>
        <div>Domain: <input type="text" name="api_url" id="api_url" v-model="url" placeholder="https://www.example.com" /></div>
        <div>API Key: <input type="text" name="api_key" id="api_key" v-model="key" placeholder="xyz123..." /></div>
    </div>
</template>
<script>
import { Preferences } from "@capacitor/preferences";
export default {
    data() {
        return {
            url: '',
            key: '',
            settings: null,
        };
    },
    mounted() {
        this.getSettings();
        if (this.settings == null) {
            this.SetSettings();
        }
    },
    methods: {
        async getSettings() {
            this.settings = await Preferences.get({ key: "settings" });
            console.log(`set: ${this.settings}`);
        },
        async SetSettings() {
            await Preferences.set({
                key: "settings",
                value: JSON.stringify({
                    url: this.url,
                    key: this.key,
                }),
            });
            this.getSettings();
        },
    },
};
</script>
<style></style>
