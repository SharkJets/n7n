<template>
    <div v-if="settings == null || settings.url == null">
        <h1>Enter your API settings..</h1>
        <div>Domain: <input type="text" name="api_url" id="api_url" v-model="url" placeholder="https://www.example.com" /></div>
        <div>API Key: <input type="text" name="api_key" id="api_key" v-model="key" placeholder="xyz123..." /></div>
        <div><button v-on:click="setSettings">Save</button></div>
    </div>
    <div v-if="settings != null && settings.url != null ">
        <slot></slot>
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
    async mounted() {
        await this.getSettings();
    },
    methods: {
        async getSettings() {
            let result  = await Preferences.get({ key: "settings" });
            this.settings = JSON.parse(result.value);
        },
        async setSettings() {
            if(this.url.trim() == '' || this.key.trim() == '') {
                return;
            }
            await Preferences.set({
                key: "settings",
                value: JSON.stringify({
                    url: this.url,
                    key: this.key,
                }),
            });
            await this.getSettings();
        },
    },
};
</script>
<style></style>
