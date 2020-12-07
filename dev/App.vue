<template>
    <div>
        <button @click="addNode">Add Node</button>
        <vue-tree-list
          @click="onClick"
          @change-name="onChangeName"
          @delete-node="onDel"
          @add-node="onAddNode"
          @drop="drop"
          @drop-before="dropBefore"
          @drop-after="dropAfter"
          :model="data"
          :edit-actions="['set-editable', onEdit]"
          :prevent-leaves-in-root="true"
          default-tree-node-name="new node"
          default-leaf-node-name="new leaf"
          :default-expanded="false"
          default-active-node-class="active-node"
          default-active-tree-node-class="active-tree-node"
          default-active-leaf-node-class="active-leaf-node"
          allow-selection-of="both"
        >
          <span class="icon" slot="addTreeNode">addTreeNode</span>
          <span class="icon" slot="addLeafNode">addLeafNode</span>
          <span class="icon" slot="editNode">editNode</span>
          <span class="icon" slot="delNode">delNode</span>
        </vue-tree-list>
        <button @click="getNewTree">Get new tree</button>
        <pre>
          {{newTree}}
        </pre>
    </div>
</template>
<script>
  import { VueTreeList, Tree, TreeNode } from '../src'
  export default {
    components: {
      VueTreeList
    },
    data () {
      return {
        newTree: {},
        data: new Tree([
          {
            name: 'Node 1',
            iconClasses: 'custom-icon',
            id: 1,
            pid: 0,
            dragDisabled: true,
            addTreeNodeDisabled: true,
            addLeafNodeDisabled: true,
            editNodeDisabled: true,
            delNodeDisabled: true,
            children: [
              {
                name: 'Node 1-2',
                id: 1,
                isLeaf: true,
                pid: 1
              }
            ]
          },
          {
            name: 'Node 2',
            id: 3,
            pid: 0,
            disabled: true
          },
          {
            name: 'Node 3',
            id: 4,
            pid: 0
          }
        ])
      }
    },
    methods: {
      onDel (node) {
        console.log(node)
        node.remove()
      },

      onChangeName (params) {
        console.log(params)
      },

      onAddNode (params) {
        console.log(params)
      },

      onClick (params) {
        console.log(params)
      },

      drop: function ({node, src, target}) {
        console.log('drop', node, src, target)
      },

      dropBefore: function ({node, src, target}) {
        console.log('drop-before', node, src, target)
      },

      dropAfter: function ({node, src, target}) {
        console.log('drop-after', node, src, target)
      },

      addNode () {
        var node = new TreeNode({ name: 'new node', isLeaf: false })
        if (!this.data.children) this.data.children = []
        this.data.addChildren(node)
      },

      onEdit(item) {
        console.log(item);
      },

      getNewTree () {
        var vm = this
        function _dfs (oldNode) {
          var newNode = {}

          for (var k in oldNode) {
            if (k !== 'children' && k !== 'parent') {
              newNode[k] = oldNode[k]
            }
          }

          if (oldNode.children && oldNode.children.length > 0) {
            newNode.children = []
            for (var i = 0, len = oldNode.children.length; i < len; i++) {
              newNode.children.push(_dfs(oldNode.children[i]))
            }
          }
          return newNode
        }

        vm.newTree = _dfs(vm.data)
      },

      onClick(model) {
        console.log(model)
      }
    }
  }
</script>
<style lang="less" rel="stylesheet/less">
  .active-node {
    border: 1px solid yellow;
  }

  .active-tree-node {
    color: red;
  }
  
  .active-leaf-node {
    color: green;
  }

  .custom-icon::before {
    content: 'x';
    font-style:  normal;
    margin-right: 0.5em
  }

  .vtl {
    .vtl-drag-disabled {
      background-color: #d0cfcf;
      &:hover {
        background-color: #d0cfcf;
      }
    }
    .vtl-disabled {
      background-color: #d0cfcf;
    }
  }
</style>

<style lang="less" rel="stylesheet/less" scoped>
  .icon {
    &:hover {
      cursor: pointer;
    }
  }
</style>
