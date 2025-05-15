<template>
    <div v-if="settings == null || settings.url == null" class="setup-container">
        <h1>Enter your API settings</h1>
        <div class="form-group">
            <label for="api_url">Domain:</label>
            <input type="text" name="api_url" id="api_url" v-model="url" placeholder="https://www.example.com" />
        </div>
        <div class="form-group">
            <label for="api_key">API Key:</label>
            <input type="text" name="api_key" id="api_key" v-model="key" placeholder="xyz123..." />
        </div>
        <div class="form-actions">
            <button v-on:click="setSettings">Save</button>
        </div>
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
<style>
.setup-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 2rem;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.setup-container h1 {
    margin-top: 0;
    margin-bottom: 1.5rem;
    color: #2c3e50;
    font-size: 1.5rem;
    text-align: center;
}

.form-group {
    margin-bottom: 1rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
    color: #546e7a;
}

.form-group input {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid #cfd8dc;
    border-radius: 4px;
    font-size: 1rem;
    box-sizing: border-box;
}

.form-actions {
    margin-top: 1.5rem;
    text-align: center;
}

.form-actions button {
    padding: 0.75rem 2rem;
    background-color: #0b7285;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s;
}

.form-actions button:hover {
    background-color: #095f6c;
}

@media (max-width: 600px) {
    .setup-container {
        padding: 1.5rem;
        border-radius: 0;
        box-shadow: none;
    }
}
</style>