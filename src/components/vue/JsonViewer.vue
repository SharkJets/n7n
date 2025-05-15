<template>
  <div class="json-viewer-component">
    <template v-if="isObject(data)">
      <div class="json-object" :class="{ expanded: isExpanded }">
        <div class="json-toggle" @click="toggleExpand">
          <span class="toggle-icon">{{ isExpanded ? '▼' : '►' }}</span>
          <span class="json-key" v-if="propKey">{{ propKey }}:</span>
          <span class="json-braces">{{ isArray(data) ? '[' : '{' }}</span>
          <span v-if="!isExpanded" class="json-preview">{{ getObjectPreview(data) }}</span>
          <span v-if="!isExpanded" class="json-braces">{{ isArray(data) ? ']' : '}' }}</span>
        </div>
        
        <div v-if="isExpanded" class="json-children">
          <div v-for="(value, key) in data" :key="key" class="json-property">
            <JsonViewer 
              :data="value" 
              :prop-key="key" 
              :level="level + 1" 
            />
          </div>
          <div class="json-braces end-braces">{{ isArray(data) ? ']' : '}' }}</div>
        </div>
      </div>
    </template>
    
    <template v-else>
      <div class="json-property">
        <span v-if="propKey" class="json-key">{{ propKey }}:</span>
        <span class="json-value" :class="getValueType(data)">{{ formatValue(data) }}</span>
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
    getObjectPreview(obj) {
      const isArray = Array.isArray(obj);
      const count = Object.keys(obj).length;
      if (count === 0) return isArray ? '' : '';
      
      if (isArray) {
        return `${count} item${count === 1 ? '' : 's'}`;
      } else {
        return `${count} propert${count === 1 ? 'y' : 'ies'}`;
      }
    }
  }
};
</script>

<style>
.json-viewer-component {
  font-family: monospace;
  font-size: 14px;
  line-height: 1.4;
}

.json-object {
  padding-left: 1.5rem;
  position: relative;
}

.json-toggle {
  cursor: pointer;
  user-select: none;
}

.json-toggle:hover {
  text-decoration: underline;
}

.toggle-icon {
  font-size: 0.75rem;
  color: #555;
  margin-right: 0.25rem;
}

.json-key {
  color: #0b7285;
  margin-right: 0.25rem;
}

.json-braces {
  color: #555;
}

.end-braces {
  padding-left: 0.5rem;
}

.json-preview {
  color: #777;
  font-style: italic;
  margin: 0 0.25rem;
}

.json-value {
  margin-left: 0.25rem;
}

.json-value.string {
  color: #218c74;
}

.json-value.number {
  color: #6a89cc;
}

.json-value.boolean {
  color: #e67e22;
}

.json-value.null, .json-value.undefined {
  color: #d35400;
  font-style: italic;
}

.json-children {
  padding-left: 0.5rem;
  border-left: 1px dotted #ccc;
  margin-left: 0.25rem;
}

.json-property {
  line-height: 1.6;
}
</style>