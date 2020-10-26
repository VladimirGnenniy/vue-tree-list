<template>
  <div class="vtl">
    <div v-if="model.name !== 'root'">
      <div
        class="vtl-border vtl-up"
        :class="{'vtl-active': isDragEnterUp}"
        @drop="dropBefore"
        @dragenter="dragEnterUp"
        @dragover="dragOverUp"
        @dragleave="dragLeaveUp"
      ></div>
      <div
        :id="model.id"
        :class="treeNodeClass"
        :draggable="!model.dragDisabled"
        @dragstart="dragStart"
        @dragover="dragOver"
        @dragenter="dragEnter"
        @dragleave="dragLeave"
        @drop="drop"
        @dragend="dragEnd"
        @mouseover="mouseOver"
        @mouseout="mouseOut"
        @click.stop="click"
      >
        <span class="vtl-caret vtl-is-small" v-if="model.children && model.children.length > 0">
          <i class="vtl-icon" :class="caretClass" @click.prevent.stop="toggle"></i>
        </span>

        <span v-if="model.isLeaf">
          <slot name="leafNodeIcon">
            <i class="far fa-file mr-1 text-info" :title="model.title"></i>
          </slot>
        </span>
        <span v-else>
          <slot name="treeNodeIcon">
            <i class="fas fa-folder mr-1 text-info" :title="model.title"></i>
          </slot>
        </span>

        <div class="vtl-node-content text-info" :title="model.title" v-if="!editable">{{model.name}}</div>
        <input
          v-else
          class="vtl-input"
          type="text"
          ref="nodeInput"
          :value="model.name"
          @input="updateName"
          v-on:keyup.enter="setUnEditable"
          @blur="setUnEditable"
        />
        <div class="vtl-operation" v-show="isHover">
          <span
            title="Add"
            @click.stop.prevent="addChild(false)"
            v-if="!model.isLeaf && !model.addTreeNodeDisabled"
          >
            <slot name="addTreeNode">
              <i class="fas fa-folder-plus text-info"></i>
            </slot>
          </span>
          <span
            title="add leaf node"
            @click.stop.prevent="addChild(true)"
            v-if="!model.isLeaf && !model.addLeafNodeDisabled"
          >
            <slot name="addLeafNode">
              <i class="fas fa-folder-plus text-info"></i>
            </slot>
          </span>
          <span title="Edit" @click.stop.prevent="setEditable" v-if="!model.editNodeDisabled">
            <slot name="editNode">
              <i class="far fa-edit text-info"></i>
            </slot>
          </span>
          <span title="Delete" @click.stop.prevent="delNode" v-if="!model.delNodeDisabled">
            <slot name="delNode">
              <i class="far fa-trash-alt text-info"></i>
            </slot>
          </span>
          <span :title="model.customActionTitle" @click.stop.prevent="onCustomeAction" v-if="model.customActionAvailable">
            <slot name="customAction"></slot>          
          </span>
        </div>
      </div>

      <div
        v-if="model.children && model.children.length > 0 && expanded"
        class="vtl-border vtl-bottom"
        :class="{'vtl-active': isDragEnterBottom}"
        @drop="dropAfter"
        @dragenter="dragEnterBottom"
        @dragover="dragOverBottom"
        @dragleave="dragLeaveBottom"
      ></div>
    </div>

    <div
      :class="{'vtl-tree-margin': model.name !== 'root'}"
      v-show="model.name === 'root' || expanded"
      v-if="isFolder"
    >
      <item
        v-bind:selectedItem="localSelectedItem"
        v-for="model in model.children"
        :default-tree-node-name="defaultTreeNodeName"
        :default-leaf-node-name="defaultLeafNodeName"
        v-bind:default-expanded="defaultExpanded"
        :model="model"
        :key="model.id"
        :class="{[defaultActiveTreeNodeClass]: localSelectedItem && model.id === localSelectedItem.id}"
        :default-active-tree-node-class="defaultActiveTreeNodeClass"
      >
        <slot name="addTreeNode" slot="addTreeNode" />
        <slot name="addLeafNode" slot="addLeafNode" />
        <slot name="editNode" slot="editNode" />
        <slot name="delNode" slot="delNode" />
        <slot name="customAction" slot="customAction" />
      </item>
    </div>
  </div>
</template>

<script>
import { Tree, TreeNode } from "./Tree.js";
import { addHandler, removeHandler } from "./tools.js";

let compInOperation = null;

export default {
  data: function() {
    return {
      isHover: false,
      editable: false,
      isDragEnterUp: false,
      isDragEnterBottom: false,
      isDragEnterNode: false,
      expanded: this.defaultExpanded,
      oldName: "",
      localSelectedItem: this.selectedItem
    };
  },
  props: {
    model: {
      type: Object
    },
    defaultLeafNodeName: {
      type: String,
      default: "New leaf node"
    },
    defaultTreeNodeName: {
      type: String,
      default: "New tree node"
    },
    defaultExpanded: {
      type: Boolean,
      default: true
    },
    title: {
      type: String,
      default: ""
    },
    customActionTitle: {
      type: String,
      default: ""
    },
    selectedItem: {
      type: Object,
      default: null
    },
    defaultActiveTreeNodeClass: {
      type: String,
      default: null
    }
  },
  watch: {
    selectedItem: function (value) {
      this.localSelectedItem = value;
    }
  },
  computed: {
    itemIconClass() {
      return this.model.isLeaf ? "vtl-icon-file" : "vtl-icon-folder";
    },

    caretClass() {
      return this.expanded ? "vtl-icon-caret-down" : "vtl-icon-caret-right";
    },

    isFolder() {
      return this.model.children && this.model.children.length;
    },

    treeNodeClass() {
      const {
        model: { dragDisabled, disabled },
        isDragEnterNode
      } = this;

      return {
        "vtl-tree-node": true,
        "vtl-active": isDragEnterNode,
        "vtl-drag-disabled": dragDisabled,
        "vtl-disabled": disabled
      };
    }
  },
  mounted() {
    /*const vm = this
      addHandler(window, 'keyup', function (e) {
        // click enter
        if (e.keyCode === 13 && vm.editable) {
          vm.editable = false
        }
      })*/
  },
  beforeDestroy() {
    removeHandler(window, "keyup");
  },
  methods: {
    updateName(e) {
      this.oldName = this.model.name;
      this.model.changeName(e.target.value);
    },
    emitChangeName(e) {
      if (e.target.value) {
        var node = this.getRootNode();
        node.$emit("change-name", {
          id: this.model.id,
          oldName: this.oldName,
          newName: e.target.value
        });
      }
    },

    delNode() {
      var node = this.getRootNode();
      node.$emit("delete-node", this.model);
    },

    onCustomeAction() {
      var node = this.getRootNode();
      node.$emit("custom-action", this.model);
    },

    setEditable() {
      this.oldName = this.model.name;
      if (this.model.isLeaf) {
        var node = this.getRootNode();
        node.$emit("change-name", {
          id: this.model.id,
          oldName: this.oldName,
          newName: this.oldName
        });
      } else {
        this.editable = true;
        this.$nextTick(() => {
          const $input = this.$refs.nodeInput;
          $input.focus();
          $input.setSelectionRange(0, $input.value.length);
        });
      }
    },

    setUnEditable(e) {
      if (e.target.value) {
        this.editable = false;
        this.emitChangeName(e);
      }
    },

    toggle() {
      if (this.isFolder) {
        this.expanded = !this.expanded;
      }
    },

    mouseOver(e) {
      if (this.model.disabled) return;
      this.isHover = true;
    },

    mouseOut(e) {
      this.isHover = false;
    },

    click() {
      var node = this.getRootNode();
      node.localSelectedItem = this.model;
      node.$emit("click", this.model);
    },

    addChild(isLeaf) {
      const name = isLeaf ? this.defaultLeafNodeName : this.defaultTreeNodeName;
      this.expanded = true;
      var node = new TreeNode({ name, isLeaf });
      this.model.addChildren(node, true);
      var root = this.getRootNode();
      root.$emit("add-node", node);
    },

    dragStart(e) {
      if (!(this.model.dragDisabled || this.model.disabled)) {
        compInOperation = this;
        // for firefox
        e.dataTransfer.setData("data", "data");
        e.dataTransfer.effectAllowed = "move";
        return true;
      }
      return false;
    },
    dragEnd(e) {
      compInOperation = null;
    },
    dragOver(e) {
      e.preventDefault();
      return true;
    },
    dragEnter(e) {
      if (!compInOperation) return;
      if (this.model.isLeaf) return;
      this.isDragEnterNode = true;
    },
    dragLeave(e) {
      this.isDragEnterNode = false;
    },
    drop(e) {
      if (!compInOperation) return;
      const oldParent = compInOperation.model.parent;
      compInOperation.model.moveInto(this.model);
      this.isDragEnterNode = false;
      var node = this.getRootNode();
      node.$emit("drop", {
        target: this.model,
        node: compInOperation.model,
        src: oldParent
      });
    },

    dragEnterUp() {
      if (!compInOperation) return;
      this.isDragEnterUp = true;
    },
    dragOverUp(e) {
      e.preventDefault();
      return true;
    },
    dragLeaveUp() {
      if (!compInOperation) return;
      this.isDragEnterUp = false;
    },
    dropBefore() {
      if (!compInOperation) return;
      const oldParent = compInOperation.model.parent;
      compInOperation.model.insertBefore(this.model);
      this.isDragEnterUp = false;
      var node = this.getRootNode();
      node.$emit("drop-before", {
        target: this.model,
        node: compInOperation.model,
        src: oldParent
      });
    },

    dragEnterBottom() {
      if (!compInOperation) return;
      this.isDragEnterBottom = true;
    },
    dragOverBottom(e) {
      e.preventDefault();
      return true;
    },
    dragLeaveBottom() {
      if (!compInOperation) return;
      this.isDragEnterBottom = false;
    },
    dropAfter() {
      if (!compInOperation) return;
      const oldParent = compInOperation.model.parent;
      compInOperation.model.insertAfter(this.model);
      this.isDragEnterBottom = false;
      var node = this.getRootNode();
      node.$emit("drop-after", {
        target: this.model,
        node: compInOperation.model,
        src: oldParent
      });
    },
    getRootNode() {
      var node = this.$parent;
      while (node._props.model.name !== "root") {
        node = node.$parent;
      }
      return node;
    }
  },
  beforeCreate() {
    this.$options.components.item = require("./VueTreeList.vue");
  }
};
</script>

<style lang="less" rel="stylesheet/less">
@font-face {
  font-family: "icomoon";
  src: url("fonts/icomoon.eot?ui1hbx");
  src: url("fonts/icomoon.eot?ui1hbx#iefix") format("embedded-opentype"),
    url("fonts/icomoon.ttf?ui1hbx") format("truetype"),
    url("fonts/icomoon.woff?ui1hbx") format("woff"),
    url("fonts/icomoon.svg?ui1hbx#icomoon") format("svg");
  font-weight: normal;
  font-style: normal;
}

.vtl-icon {
  /* use !important to prevent issues with browser extensions that change fonts */
  font-family: "icomoon" !important;
  speak: none;
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  line-height: 1;

  /* Better Font Rendering =========== */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  &.vtl-menu-icon {
    margin-right: 4px;
    &:hover {
      color: inherit;
    }
  }
  /*&:hover {
      color: blue;
    }*/
}

.vtl-icon-file:before {
  content: "\e906";
}
.vtl-icon-folder:before {
  content: "\e907";
}
.vtl-icon-caret-down:before {
  content: "\e901";
}
.vtl-icon-caret-right:before {
  content: "\e900";
}
.vtl-icon-edit:before {
  content: "\e902";
}
.vtl-icon-folder-plus-e:before {
  content: "\e903";
}
.vtl-icon-plus:before {
  content: "\e904";
}
.vtl-icon-trash:before {
  content: "\e905";
}

.vtl-border {
  height: 5px;
  &.vtl-up {
    margin-top: -5px;
    background-color: transparent;
  }
  &.vtl-bottom {
    background-color: transparent;
  }
  &.vtl-active {
    border-bottom: 3px dashed blue;
    /*background-color: blue;*/
  }
}

.vtl-tree-node {
  display: flex;
  align-items: center;
  padding: 5px 0 5px 1rem;
  .vtl-input {
    border: none;
    max-width: 150px;
    /*border-bottom: 1px solid blue;*/
  }
  &:hover {
    background-color: #f0f0f0;
  }
  &.vtl-active {
    outline: 2px dashed pink;
  }
  .vtl-caret {
    margin-left: -1rem;
    cursor: pointer;
  }
  .vtl-operation {
    margin-left: 2rem;
    letter-spacing: 1px;
    cursor: pointer;
  }
}

.vtl-item {
  cursor: pointer;
}
.vtl-tree-margin {
  margin-left: 2em;
}
</style>
