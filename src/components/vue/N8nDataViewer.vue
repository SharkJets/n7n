<template>
    <div class="n8n-data-viewer">
        <div v-if="isLoading" class="loading">
            Loading n8n data...
        </div>

        <div v-else-if="error" class="error-card">
            <h2>Error</h2>
            <p>{{ error }}</p>
        </div>

        <div v-else-if="n8nData" class="data-container">
            <div class="heading">
                <div class="nav">
                    <button v-on:click="refresh">Refresh</button>
                    <button v-on:click="logout">Logout</button>
                </div>
                <div class="data-summary">
                    <div class="data-controls">
                        <label for="limit-select">Show:</label>
                        <select id="limit-select" v-model="itemLimit" @change="fetchData">
                            <option value="10">10 entries</option>
                            <option value="20">20 entries</option>
                            <option value="50">50 entries</option>
                            <option value="100">100 entries</option>
                        </select>
                    </div>
                    <div class="controls">
                        <div class="mode-filter">
                            <label for="mode-select">Mode:</label>
                            <select id="mode-select" v-model="selectedMode" @change="applyModeFilter">
                                <option value="all">All Modes</option>
                                <option v-for="mode in availableModes" :key="mode" :value="mode">{{ mode }}</option>
                            </select>
                        </div>
                    </div>
                </div>                
            </div>

            <div class="data-list">
                <div v-for="(item, index) in filteredData" :key="index" class="data-item">
                    <div class="data-header" @click="toggleExpanded(index)">
                        <div class="header-left">
                            <span class="expand-icon">{{ expandedItems[index] ? '▼' : '►' }}</span>
                            <h3>{{ item.workflowData.name }}</h3>
                            <div>{{ new Date(item.startedAt).toISOString().slice(0, 16).replace('T', ' ') }}</div>
                            <div v-if="item.data.resultData.runData['Error Trigger']">
                                <div><span class="label">Workflow:</span></div>
                                <div>{{  item.data.resultData.runData["Error Trigger"][0].data.main[0][0].json.workflow.name  }}</div>
                                <div><span class="label">Error:</span></div>
                                <div>{{  item.data.resultData.runData["Error Trigger"][0].data.main[0][0].json.execution.error.message  }}</div>
                                <div><span class="label">Last Node:</span></div>
                                <div>{{  item.data.resultData.runData["Error Trigger"][0].data.main[0][0].json.execution.lastNodeExecuted  }}</div>
                            </div>
                        </div>
                        <span class="status" :class="{ 'finished': item.finished, 'pending': !item.finished }">
                            {{ item.mode }}
                        </span>
                    </div>

                    <div class="data-details" :class="{ 'expanded': expandedItems[index] }">
                        <div class="basic-details">
                            <p><strong>ID:</strong> {{ item.id }}</p>
                            <p><strong>Mode:</strong> {{ item.mode }}</p>
                            <p><strong>Started:</strong> {{ new Date(item.startedAt).toLocaleString() }}</p>
                            <p><strong>Stopped:</strong> {{ item.stoppedAt ? new Date(item.stoppedAt).toLocaleString() : 'N/A' }}</p>
                        </div>

                        <div v-if="expandedItems[index]" class="extended-details">
                            <h4>All Properties</h4>
                            <div class="json-viewer">
                                <JsonViewer :data="item" />
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-else class="no-data">
            <p>No data available</p>
        </div>
    </div>
</template>
<script>
import JsonViewer from './JsonViewer.vue';
import { Preferences } from "@capacitor/preferences";

export default {
    components: {
        JsonViewer
    },
    data() {
        return {
            settings: null,
            itemLimit: 10,
            n8nData: null,
            isLoading: true,
            error: null,
            expandedItems: {},
            allExpanded: false,
            selectedMode: 'all',
            availableModes: []
        };
    },
    computed: {
        filteredData() {
            if (!this.n8nData || !this.n8nData.data) return [];
            
            if (this.selectedMode === 'all') {
                return this.n8nData.data;
            }
            
            return this.n8nData.data.filter(item => item.mode === this.selectedMode);
        }
    },
    methods: {
        async fetchData() {
            try {
                this.isLoading = true;
                this.error = null;
                this.expandedItems = {};
                this.allExpanded = false;
                
                // Fetch the n8n data with the selected limit
                const response = await fetch(`${this.settings.url}/api/v1/executions?includeData=true&limit=${this.itemLimit}`, {
                    headers: {
                        "X-N8N-API-KEY": this.settings.key
                    }
                });
                
                if (!response.ok) {
                    throw new Error(`Failed to load data: ${response.status} ${response.statusText}`);
                }
                
                this.n8nData = await response.json();
                
                // Extract unique modes from data
                if (this.n8nData && this.n8nData.data) {
                    const modes = new Set();
                    this.n8nData.data.forEach(item => {
                        if (item.mode) {
                            modes.add(item.mode);
                        }
                    });
                    this.availableModes = Array.from(modes).sort();
                    
                    // Initialize expandedItems with all items collapsed
                    const initialExpandedItems = {};
                    this.n8nData.data.forEach((_, index) => {
                        initialExpandedItems[index] = false;
                    });
                    this.expandedItems = initialExpandedItems;
                }
                
                this.isLoading = false;
            } catch (err) {
                this.error = err.message;
                this.isLoading = false;
                console.error('Error loading n8n data:', err);
            }
        },
        applyModeFilter() {
            // Reset expanded items when filter changes
            this.expandedItems = {};
            this.allExpanded = false;
        },
        toggleExpanded(index) {
            // Using direct assignment instead of $set
            this.expandedItems = {
                ...this.expandedItems,
                [index]: !this.expandedItems[index]
            };
            this.updateAllExpandedState();
        },
        toggleAllExpanded() {
            const newState = !this.allExpanded;
            this.allExpanded = newState;

            if (this.n8nData && this.n8nData.data) {
                const newExpandedItems = {};
                this.n8nData.data.forEach((_, index) => {
                    newExpandedItems[index] = newState;
                });
                // Replace the entire object for reactivity
                this.expandedItems = newExpandedItems;
            }
        },
        updateAllExpandedState() {
            if (!this.n8nData || !this.n8nData.data || this.n8nData.data.length === 0) {
                this.allExpanded = false;
                return;
            }

            const expandedCount = Object.values(this.expandedItems).filter(Boolean).length;
            this.allExpanded = expandedCount === this.n8nData.data.length;
        },
        async logout(){
            console.log('logout');
            await Preferences.remove({key: "settings"});
            this.settings = null;
            document.location = '/';
        }, 
        async refresh(){
            await this.fetchData();
        }
    },
    async mounted() {
        // Load data on component mount
        let result = await Preferences.get({ key: "settings" });
        this.settings = JSON.parse(result.value);
        await this.fetchData();
    },
};
</script>
<style>
.nav{
    display:flex;
    justify-content: space-between;
}

.label{
    font-weight: bold;
}

.n8n-data-viewer {
    font-family: system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
    max-width: 1200px;
    margin: 0 auto;
}

.loading, .error-card, .no-data {
    text-align: center;
    padding: 2rem;
    background: #f8f9fa;
    border-radius: 8px;
    margin: 1rem 0;
}

.error-card {
    background: #fff2f2;
    color: #d32f2f;
}

.data-container {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

.data-summary {
    background: #f5f7f9;
    padding: 0.75rem 1.5rem;
    border-bottom: 1px solid #e0e4e8;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.controls {
    display: flex;
    gap: 1rem;
    align-items: center;
}

.data-controls {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.data-controls label,
.mode-filter label {
    font-size: 0.875rem;
    font-weight: 500;
}

.data-controls select,
.mode-filter select {
    padding: 0.375rem 0.75rem;
    border: 1px solid #cfd8dc;
    border-radius: 4px;
    background: #fff;
    font-size: 0.875rem;
    cursor: pointer;
}

.mode-filter {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.toggle-all-btn {
    background: #fff;
    border: 1px solid #cfd8dc;
    border-radius: 4px;
    padding: 0.375rem 0.75rem;
    font-size: 0.875rem;
    cursor: pointer;
    transition: all 0.2s;
}

.toggle-all-btn:hover {
    background: #f5f7f9;
    border-color: #b0bec5;
}

.data-item {
    border-bottom: 1px solid #e0e4e8;
}

.data-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 1.5rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

.data-header:hover {
    background-color: #f5f7f9;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 0.5rem;
}

.expand-icon {
    font-size: 0.75rem;
    color: #546e7a;
}

.data-header h3 {
    margin: 0;
    font-size: 1.2rem;
}

.status {
    padding: 0.25rem 0.75rem;
    border-radius: 16px;
    font-size: 0.875rem;
    font-weight: 500;
}

.status.finished {
    background: #e8f5e9;
    color: #2e7d32;
}

.status.pending {
    background: #fff8e1;
    color: #f57c00;
}

.data-details {
    padding: 0 1.5rem;
    max-height: 0;
    overflow: hidden;
    transition: all 0.3s ease-in-out;
}

.data-details.expanded {
    max-height: 2000px;
    padding: 0 1.5rem 1.5rem;
}

.basic-details {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 0.75rem;
    margin-bottom: 1rem;
}

.basic-details p {
    margin: 0;
}

.extended-details {
    margin-top: 1.5rem;
    padding-top: 1.5rem;
    border-top: 1px dashed #e0e4e8;
}

.extended-details h4 {
    margin-top: 0;
    margin-bottom: 1rem;
    color: #546e7a;
    font-size: 1rem;
    padding-left: 0.5rem;
}

.json-viewer {
    background: #ffffff;
    border-radius: 8px;
    padding: 0;
    overflow: auto;
    max-height: 600px;
    width: 100%;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}
</style>