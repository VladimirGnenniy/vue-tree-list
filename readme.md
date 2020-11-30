# vue-tree-list
A vue component for tree structure. Support adding treenode/leafnode, editing node's name and dragging.

![vue-tree-demo.gif](https://raw.githubusercontent.com/ParadeTo/vue-tree-list/master/img/demo.gif)

[Live Demo](http://paradeto.com/vue-tree-list/)

# use
``npm install vue-tree-list``

```html
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
          v-bind:prevent-leaves-in-root="true"
          default-tree-node-name="new node"
          default-leaf-node-name="new leaf"
          v-bind:default-expanded="false"
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
                id: 2,
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

```

# props
## props of vue-tree-list
| name | type | default | description |
|:-----:|:-------:|:------------:|:----:|
model | TreeNode | - | You can use `const head = new Tree([])` to generate a tree with the head of `TreeNode` type
default-tree-node-name | string | New node node | Default name for new treenode
default-leaf-node-name | string | New leaf node | Default name for new leafnode
default-expanded | boolean | true | Tree is expanded or not
allow-selection-of | null, 'tree', 'leaf', 'both'| null | Which type of nodes to select
default-active-node-class | string | null | Class to apply for selected items
default-active-tree-node-class | string | null | Class to apply for selected tree items
default-active-leaf-node-class | string | null | Class to apply for selected leaf items
selected-item | object | null | Node to preselect
prevent-leaves-in-root | boolean | false | Whether to prevent dropping leaves on to the root level
edit-tree-node-actions | (function (`TreeNode`) , 'set-editable')[] | ['set-editable'] | What to do when edit icon is clicked on a tree node
edit-leaf-actions | (function (`TreeNode`) , 'set-editable')[] | ['set-editable'] | What to do when edit icon is clicked on a leaf
click-tree-node-actions | (function (`TreeNode`) , 'select')[] | ['select'] | What to do when a tree node is clicked
click-leaf-node-actions | (function (`TreeNode`) , 'select')[] | ['select'] | What to do when a leaf is clicked


## props of TreeNode
### attributes
| name | type | default | description |
|:-----:|:-------:|:------------:|:----:|
id | string, number | current timestamp | The node's id
isLeaf | boolean | false | The node is leaf or not
dragDisabled | boolean | false | Forbid dragging tree node
addTreeNodeDisabled | boolean | false | Show `addTreeNode` button or not
addLeafNodeDisabled | boolean | false | Show `addLeafNode` button or not
editNodeDisabled | boolean | false | Show `editNode` button or not
delNodeDisabled | boolean | false | Show `delNode` button or not
children | array | null | The children of node

### methods
| name | params | description |
|:-----:|:-------:|:----:|
changeName | name | Change node's name
addChildren | children: object, array | Add children to node
remove | - | Remove node from the tree
moveInto | target: TreeNode | Move node into another node
insertBefore | target: TreeNode | Move node before another node
insertAfter | target: TreeNode | Move node after another node

# events
| name | params | description |
|:-----:|:-------:|:----:|
click | TreeNode | Trigger when clicking a tree node
change-name | {'id', 'oldName', 'newName'} | Trigger after changing a node's name
delete-node | TreeNode | Trigger when clicking `delNode` button. You can call `remove` of `TreeNode` to remove the node.
add-node | TreeNode | Trigger after adding a new node
drop | {node, src, target} | Trigger after dropping a node into another. node: the draggable node, src: the draggable node's parent, target: the node that draggable node will drop into
drop-before | {node, src, target} | Trigger after dropping a node before another. node: the draggable node, src: the draggable node's parent, target: the node that draggable node will drop before
drop-after | {node, src, target} | Trigger after dropping a node after another. node: the draggable node, src: the draggable node's parent, target: the node that draggable node will drop after
select | TreeNode | Trigger when selecting a tree node

# customize operation icons
The component has default icons for `addTreeNode`, `addLeafNode`, `editNode`, `delNode` button, but you can also customize them:

```html
<span class="icon" slot="addTreeNode">addTreeNode</span>
<span class="icon" slot="addLeafNode">addLeafNode</span>
<span class="icon" slot="editNode">editNode</span>
<span class="icon" slot="delNode">delNode</span>
```
