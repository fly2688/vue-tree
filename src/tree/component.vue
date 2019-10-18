<template>
    <section class="halo-tree">
        <treeNode v-if='treeData !== ""' :depth='0' v-model="vm" @handlecheckedChange='handlecheckedChange' @toggleChildren='toggleChildren' @nodeClick='nodeClick' :node="treeData.root" :options='treeOptions' @on-node-change="onNodeChange"></treeNode>
    </section>
</template>

<script>
import treeNode from '../treeNode';
import TreeData from '../utils/TreeUtil';
export default {
    name: 'vTree',
    props: ['options'],
    data () {
      return {
        treeData: '',
        vm: null,
        treeOptions: {},
        isDrag: false
      }
    },
    components: ({treeNode}),
    created () { // 视图被创建
      this.setOptions();
    },
    methods: {
      setOptions () {
        let temp = this.options || {};
        let data = temp.treeData || [];
        this.isDrag = typeof temp.isDrag === 'boolean' ? temp.isDrag : false;
        this.vm = temp.vm || null;
        let selectFlag = temp.selectFlag || '';
        this.onSelectedNode = typeof temp.onSelectedNode === 'function' ? temp.onSelectedNode : null;
        this.onGetCurrentItem = typeof temp.onGetCurrentItem === 'function' ? temp.onGetCurrentItem : null;
        this.treeOptions = {
          hasClick: (typeof temp.hasClick === 'boolean' ? temp.hasClick : false),
          isSync: (typeof temp.isSync === 'boolean' ? temp.isSync : false),
          hasCheck: (typeof temp.hasCheck === 'boolean' ? temp.hasCheck : false),
          depth: (temp.openDepth || 0)
        }
        this.treeData = (data.length > 0) ? new TreeData(data, this.isDrag, selectFlag) : '';
      },
      onNodeChange (rootNode) {
        this.treeData.root = JSON.parse(JSON.stringify(rootNode));
      },
      handlecheckedChange (curNode) {
        let temp = this.treeData.changeCheckHalfStatus(curNode);
        let selectedNodes = {};
        if (this.treeOptions.isSync && curNode.parentId) {
          temp = this.treeData.changeCurrentFaCheck(curNode.path);
          selectedNodes = this.getSelectedNodes(temp);
        } else {
          selectedNodes = this.getSelectedNodes(temp);
        }
        selectedNodes.selectAll = false; // 标记选择所有企业与否
        if (curNode.parentId === '' && curNode.isChecked) selectedNodes.selectAll = true;
        typeof this.onSelectedNode === 'function' && this.onSelectedNode(selectedNodes);
        this.treeData.root = JSON.parse(JSON.stringify(temp));
      },
      nodeClick (curItem) {
        let temp = null;
        if (this.hasClick) {
          temp = this.treeData.clickActive(curItem);
          typeof this.onGetCurrentItem === 'function' && this.onGetCurrentItem(curItem);
        }
        if (this.hasCheck & !this.isSync) {
          temp = this.treeData.changeCurChecked(curItem.id);
          let selectedNodes = this.getSelectedNodes(temp);
          typeof this.onSelectedNode === 'function' && this.onSelectedNode(selectedNodes);
        }
        this.treeData.root = JSON.parse(JSON.stringify(temp));
      },
      getSelectedNodes (tempRoot) {
        let selectedNodes = {};
        selectedNodes.selectedId = [];
        selectedNodes.selectedName = [];
        selectedNodes.selected = [];
        const _traverseDown = (tempRoot) => {
          if (tempRoot.isChecked) {
            selectedNodes.selectedId.push(tempRoot.id);
            selectedNodes.selectedName.push(tempRoot.name);
            selectedNodes.selected.push(tempRoot);
          }
          for (var i in tempRoot.children) {
            _traverseDown(tempRoot.children[i]);
          }
        };
        _traverseDown(tempRoot);
        return selectedNodes;
      },
      toggleChildren (curItem) {
        let temp = this.treeData.toggleChilds(curItem.id);
        this.treeData.root = JSON.parse(JSON.stringify(temp));
      }
    },
    watch: {
      options () {
        this.setOptions();
      }
    }
  }
</script>
