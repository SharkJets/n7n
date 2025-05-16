<template>
  <div class="json-viewer-component">
    <template v-if="isObject(data)">
      <div class="json-object-container" :class="{ 'root-item': level === 0 }">
        <div class="json-object-header" @click="toggleExpand"  v-if="!isEmpty(data)">
          <div class="header-left-inner">
            <span class="toggle-icon" :class="isExpanded ? '' : 'spin'">▼</span>
            <span class="json-key" v-if="propKey">{{ propKey }}</span>
            <span class="json-preview">{{ getObjectPreview(data) }}</span>
          </div>
        </div>
        
        <div v-if="isExpanded" class="json-object-content">
          <div v-for="(value, key) in data" :key="key" class="json-item">
            <JsonViewer :data="value" :prop-key="key" :level="level + 1"  />
          </div>
        </div>
      </div>
    </template>
    
    <template v-else>
      <div class="json-property">
        <div class="json-key-container" v-if="propKey">{{ propKey }}</div>
        <div class="json-value-container" :class="getValueType(data)">{{ formatValue(data) }}</div>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  name: 'JsonViewer',
  props: {
    data: {
      required: true
    },
    propKey: {
      type: String,
      default: ''
    },
    level: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      isExpanded: this.level < 1  // Auto-expand first level
    };
  },
  methods: {
    isObject(val) {
      return val !== null && typeof val === 'object';
    },
    isArray(val) {
      return Array.isArray(val);
    },
    getValueType(val) {
      if (val === null) return 'null';
      if (val === undefined) return 'undefined';
      return typeof val;
    },
    formatValue(val) {
      if (val === null) return 'null';
      if (val === undefined) return 'undefined';
      if (typeof val === 'string') return `"${val}"`;
      if (typeof val === 'boolean' || typeof val === 'number') return val.toString();
      return val;
    },
    toggleExpand() {
      this.isExpanded = !this.isExpanded;
    },
    isEmpty(obj){
      const count = Object.keys(obj).length;
      if (count === 0) return true;
    },
    getObjectPreview(obj) {
      const isArray = Array.isArray(obj);
      const count = Object.keys(obj).length;
      if (count === 0) return isArray ? '(empty array)' : '(empty object)';
      
      if (isArray) {
        return `(${count} item${count === 1 ? '' : 's'})`;
      } else {
        return `(${count} propert${count === 1 ? 'y' : 'ies'})`;
      }
    }
  }
};
</script>

<style>
.json-viewer-component {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 14px;
  line-height: 1.4;
  width: 100%;
  box-sizing: border-box;
}

.json-object-container {
  margin-bottom: 0.25rem;
  border-radius: 0;
  overflow: hidden;
  width: 100%;
  box-shadow: none;
}

.root-item {
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  margin-bottom: 0.75rem;
}

.json-object-header {
  display: flex;
  align-items: center;
  padding: 0.75rem;
  background-color: #f5f7fa;
  cursor: pointer;
  user-select: none;
  border-bottom: 1px solid #eaeaea;
  justify-content: space-between;
}

.header-left-inner {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
}

.json-object-header:hover {
  background-color: #eef1f6;
}

.json-object-content {
  padding: 0;
  background-color: #ffffff;
}

.toggle-icon {
  font-size: 0.8rem;
  color: #666;
  margin-right: 0.5rem;
  width: 12px;
  text-align: center;
}

.json-key {
  color: #0b7285;
  font-weight: bold;
  margin-right: 0.5rem;
}

.json-type {
  color: #909399;
  background-color: #f0f2f5;
  padding: 0.2rem 0.5rem;
  border-radius: 12px;
  font-size: 0.75rem;
  margin-right: 0.5rem;
}

.json-preview {
  color: #777;
  font-style: italic;
}

.json-item {
  border-bottom: 1px solid #f5f5f5;
}

.json-item:last-child {
  border-bottom: none;
}

.json-property {
  display: flex;
  flex-direction: column;
  padding: 0;
  background-color: #fff;
  width: 100%;
  border-bottom: 1px solid #f5f5f5;
}

.json-key-container {
  color: #0b7285;
  font-weight: bold;
  padding: 0.75rem 0.75rem 0.25rem 0.75rem;
}

.json-value-container {
  padding: 0.25rem 0.75rem 0.75rem 0.75rem;
  word-break: break-word;
}

.json-value-container.string {
  color: #10ac84;
}

.json-value-container.number {
  color: #2e86de;
}

.json-value-container.boolean {
  color: #ee5253;
  font-weight: bold;
}

.json-value-container.null, .json-value-container.undefined {
  color: #ff9f43;
  font-style: italic;
}
</style>