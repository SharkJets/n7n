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
    margin: 2rem auto;
    padding: 2.5rem;
    background-color: #eee;
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.25);
}

.setup-container h1 {
    margin-top: 0;
    margin-bottom: 2rem;
    color: #3b4252;
    font-size: 1.8rem;
    text-align: center;
    font-weight: 600;
}

.form-group {
    margin-bottom: 1.5rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 600;
    color: #4a5568;
    font-size: 0.95rem;
}

.form-group input {
    width: 100%;
    padding: 0.9rem 1rem;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    font-size: 1rem;
    box-sizing: border-box;
    transition: all 0.3s ease;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.02);
}

.form-group input:focus {
    outline: none;
    border-color: #4c6ef5;
    box-shadow: 0 0 0 3px rgba(76, 110, 245, 0.15);
}

.form-actions {
    margin-top: 2rem;
    text-align: center;
}

.form-actions button {
    padding: 0.9rem 2.5rem;
    background: linear-gradient(to right, #4c6ef5, #3b5bdb);
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 4px 12px rgba(76, 110, 245, 0.25);
}

.form-actions button:hover {
    background: linear-gradient(to right, #3b5bdb, #364fc7);
    transform: translateY(-2px);
    box-shadow: 0 6px 16px rgba(76, 110, 245, 0.3);
}

@media (max-width: 600px) {
    .setup-container {
        margin: 0 auto;
        padding: 1.5rem;
        /* border-radius: 0; */
        /* box-shadow: none; */
    }
}
</style>