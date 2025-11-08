<template>
  <ul class="file-tree">
    <li v-for="node in nodes" :key="node.path" :class="{ 'excluded-node': node.excluded }">
      <div class="node-item" :style="{ 'padding-left': depth * 20 + 'px' }">
        <span v-if="node.isDir" @click="toggleExpand(node)" class="toggler" :title="node.lazyLoadChildren && !node.childrenLoaded ? 'Load folder contents' : 'Toggle folder'">
          <span v-if="node.loadingChildren" class="loader" aria-label="Loading">⟳</span>
          <span v-else>{{ node.expanded ? '▼' : '▶' }}</span>
        </span>
        <span v-else class="item-spacer"></span>
        
        <input 
          type="checkbox" 
          :checked="!node.excluded" 
          @change="handleCheckboxChange(node)"
          class="exclude-checkbox"
        />
        <span @click="node.isDir ? toggleExpand(node) : null" :class="{ 'folder-name': node.isDir }">
          {{ node.name }}
        </span>
      </div>
      <FileTree 
        v-if="node.isDir && node.expanded && node.children" 
        :nodes="node.children" 
        :project-root="projectRoot"
        :depth="depth + 1"
        @toggle-exclude="emitToggleExclude"
        @load-children="emitLoadChildren"
      />
    </li>
  </ul>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  nodes: Array,
  projectRoot: String,
  depth: {
    type: Number,
    default: 0
  },
  parentExcluded: { // Whether an ancestor is excluded
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['toggle-exclude', 'load-children']);

function toggleExpand(node) {
  if (!node.isDir) return;
  if (node.lazyLoadChildren && !node.childrenLoaded) {
    if (!node.loadingChildren) {
      emitLoadChildren(node);
    }
    return;
  }
  node.expanded = !node.expanded;
}

function handleCheckboxChange(node) {
  // Emit an event with the node to toggle its exclusion status in the parent (App.vue)
  emit('toggle-exclude', node);
}

function emitToggleExclude(node) {
    emit('toggle-exclude', node); // Bubble up the event
}

function emitLoadChildren(node) {
    emit('load-children', node);
}

// A node is effectively excluded if one of its PARENTS is.
// This is mainly for UI state (e.g., disabling checkbox), backend handles true exclusion.
function isEffectivelyExcludedByParent(node) {
    let current = node.parent; 
    while(current) {
        if (current.excluded) return true;
        current = current.parent;
    }
    return false;
}

</script>

<style scoped>
.file-tree {
  list-style-type: none;
  padding-left: 0; /* Remove default ul padding */
}
.file-tree li {
  margin: 2px 0;
}
.node-item {
  display: flex;
  align-items: center;
  cursor: default;
}
.node-item:hover {
  background-color: #f0f0f0;
}
.toggler {
  cursor: pointer;
  width: 20px;
  display: inline-block;
  text-align: center;
}
.item-spacer {
  width: 20px; /* Space for non-folders to align with folder togglers */
  display: inline-block;
}
.folder-name {
  cursor: pointer; /* To indicate it's clickable for expanding */
  font-weight: bold;
}
.exclude-checkbox {
  margin-right: 5px;
  cursor: pointer;
}
.excluded-node > .node-item > span:not(.toggler) {
  text-decoration: line-through;
  color: #999;
}
/* Style for checkbox of an effectively excluded item (e.g. parent excluded) */
.exclude-checkbox:disabled + span {
    color: #bbb; /* Lighter text for items under an excluded parent */
}
.exclude-checkbox:disabled {
    cursor: not-allowed;
}

.loader {
  display: inline-block;
  width: 20px;
  text-align: center;
  animation: spin 0.9s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

</style>
