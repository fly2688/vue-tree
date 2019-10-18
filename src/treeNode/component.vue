<template>
  <!--
  [:draggable="{Boolean}"] 空节点不能被拖动，判断依据是是否存在 node.name
  [@dragover.prevent] 必须设置，否则浏览器默认是禁用拖动的 -->
  <section class="tree-node"
       :draggable="!!node.name && node.isDrag" 
       @dragover.prevent
       @dragstart.stop="handleDragStart($event)"
       @drop.stop="handleDrop"
       @dragenter.stop="handleDragEnter"
       @dragleave.stop="handleDragLeave"
       @dragend.prevent="handleDragEnd">
    <section v-if="node.name" class="tree-node-name borderL" :class="{ 'has-name': node.name, 'noBorder': node.level === 0 }">
      <span>
        <span @click.stop.parent="toggleChildren(node)" v-if="node.children && node.children.length > 0" :class="node.open ? 'tree-open' : 'tree-close'"></span>
        <section v-if="options.hasCheck" class="checkbox">
            <input @change="handlecheckedChange(node)" v-model="node.isChecked" type="checkbox" :id="node.selectFlag + node.id" />
            <label style="width: auto;" :for="node.selectFlag + node.id"></label>
        </section>
        <span class="el-tree-node__label pointer" :class="{on: node.active}" @click.stop.prevent="nodeClick(node)" >{{ node.name }}</span>
      </span>
    </section>
    <section class="tree-node-children borderL" v-show="node.open">
      <treenode v-for="(child, $index) in node.children" :depth='depth + 1' :key='child.id' @toggleChildren='toggleChildren'
          v-model="valueModel" :options='options' @nodeClick='nodeClick' @handlecheckedChange="handlecheckedChange" :node="child" :idx="$index">
      </treenode>
    </section>
  </section>
</template>

<script>
import Vue from 'vue';
export default {
    name: 'treenode',
    props: ['node', 'value', 'idx', 'options', 'depth'],
    data () {
      return {
        hideChildren: false,
        unwatchRootNode: () => {},
        tree: null
      }
    },
    created () { // 视图被创建
      if (this.depth < this.options.depth) {
        this.node.open = true;
      }
    },
    methods: {
      clearBgColor () { // 清理样式
        this.$el.style.backgroundColor = '';
      },
      handleDragStart (event) {
        this.valueModel = this; // 设置本组件为当前正在拖动的实例，此举将同步 sync 到所有 TreeNode 实例
        event.dataTransfer.setData('asdf', '');
        // this.$el.style.backgroundColor = 'silver';
      },
      handleDrop () {
        let _this = this; // 此时 this 为目的地节点，vm 才是被拖动节点
        let currentCompany = _this.value.node;
        let toCompany = _this.node;
        _this.clearBgColor();
        if (!this.isAllowToDrop) return;
        // 无论如何都直接删除被拖动节点
        let index = _this.value.$parent.node.children.indexOf(currentCompany);
        _this.value.$parent.node.children.splice(index, 1);
        // 情况 1：拖入空节点，成其兄弟（使用 splice 插入节点）
        if (!toCompany.name) {
          return _this.$parent.node.children.splice(_this.idx / 2, 0, currentCompany);
        }
        // 情况2：拖入普通节点，成为其子
        if (!toCompany.children) Vue.set(toCompany, 'children', []); // 须用 $set 引入双向绑定
        toCompany.children.push(currentCompany);
      },
      handleDragEnter () { // 允许拖放才会显示样式
        if (!this.isAllowToDrop) return;
        this.$el.style.backgroundColor = 'yellow';
      },
      handleDragLeave () {
        this.clearBgColor();
      },
      handleDragEnd () {
        this.clearBgColor();
        if (typeof this.idx === 'undefined') {
          this.$emit('on-node-change', this.node);
        } else {
          this.$parent && this.$parent.onDragEnd();
        }
      },
      onDragEnd () {
        if (typeof this.idx === 'undefined') {
          this.$emit('on-node-change', this.node);
        } else {
          this.$parent && this.$parent.onDragEnd();
        }
      },
      handlecheckedChange (item) {
        this.$emit('handlecheckedChange', item);
      },
      nodeClick (item) {
        if (this.options.hasClick) {
          this.$emit('nodeClick', item);
        } else {
          if (this.options.hasCheck && !this.options.isSync) this.$emit('nodeClick', item);
        }
      },
      toggleChildren (item) {
        this.$emit('toggleChildren', item);
      }
    },
    computed: {
      valueModel: {
        get: function () {
          return this.value;
        },
        set: function (val) {
          this.$emit('input', val);
        }
      },
      children () { // 为每个子节点前后都生成空节点，便于实现兄弟节点间的“插入排序”
        // 举例：原本是 [N1, N2, N3]
        let { children } = this.node;
        if (!children || !children.length) return [];
        if (children.isDrag) {
          let _children = [];
          children.forEach(child => _children.push({}, child));
          _children.push({});
          // 最后生成 [E1, N1, E2, N2, E3, N3, E4]（其中 N 表示节点，E 表示空节点）
          return _children;
        } else {
          return children;
        }
      },
      isParent () { // 拖放限制 1：判断“我”是否为被拖动节点的父节点
        return this === (this.value && this.value.$parent);
      },
      isNextToMe () { // 拖放限制 2：判断“我”是否为被拖动节点的相邻节点
        return this.$parent === this.value && this.value.$parent && Math.abs(this.idx - this.value.idx) === 1;
      },
      isMeOrMyAncestor () { // 拖放限制 3：判断被拖动节点是否为“我”自身或“我”的祖先
        var p = this;
        while (p) {
          if (this.value === p) return true;
          p = p.$parent;
        }
      },
      isAllowToDrop () { // 上述拖放限制条件的综合
        return !(this.isParent || this.isNextToMe || this.isMeOrMyAncestor);
      }
    },
    beforeDestroy () {
      let app = this;
      if (typeof app.idx === 'undefined') {
        app.unwatchRootNode();
      }
    }
  }
</script>

<style lang="scss" scoped="scoped">
/*根字体*/
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
}
html, body {
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  background-color: #efefef;
  font-family: 'Microsoft Yahei', Helvetica, Arial, sans-serif;
  color: #1c2438;
}
.tree-node.empty-node { /* 空节点 */
    height: 8px;
    list-style-type: none;
}
.tree-node{
    font-size: 14px;
    text-align: left;
    .tree-node-children { /* 层次缩进 */
        margin-left: 2.5em;
        width: 100%;
    }
    .tree-node-name .tree-node-action{
        margin-left: 15px;
        font-size: 14px;
        display: none;
    }
    .tree-node-name.has-name{
        height: 33px;
        line-height: 33px;
        display: inline-block;
    }
    .tree-node-name.has-name:hover{
        background: #eee;
        
    }
    .tree-node-name:hover .tree-node-action{
        display: inline-block;
        height: 14px;
    }
    .borderL {
        position: relative;
    }
    .borderL:after,
    .borderL:before {
        content: '';
        left: -21px;
        position: absolute;
        right: auto;
        border-width: 1px
    }
    .borderL:before {
        border-left: 1px dashed #999;
        bottom: 50px;
        height: 100%;
        top: -8px;
        width: 1px;
    }
    .borderL:after {
        border-top: 1px dashed #999;
        height: 20px;
        top: 17px;
        width: 22px
    }
    .borderL.tree-node-children:after {
        border-top: none;
        height: 0;
        top:0;
        width:0;
    }
    .borderL.noBorder:after,
    .borderL.noBorder:before {
        border: none;
    }
    .checkbox label::after{
        top: -8px;
    }
    .checkbox label{
        padding-left: 0;
    }
    .el-tree-node__label.on{
        background-color: #F04055;
        color: #fff;
        padding: 0 4px;
        border-radius: 4px;
    }
    .tree-open,.tree-close {
        display: inline-block;
        width:14px;
        height:14px;
        text-align: center;
        line-height: 10px;
        border: 1px solid orange;
        border-radius: 2px;
        background: orange;
        color: #fff;
    }
    .tree-open {
        line-height: 13px;
    }
    .tree-close:after {
        content: "+";
        font-style: normal;
    }
    .tree-open:after {
        content: "\2013";
        font-style: normal;
    }
    .pointer{
      cursor: pointer;
    }
    .checkbox {
        display: inline-block;
        margin-top: 0;
        margin-bottom: 0;
        vertical-align: middle;
    }
    .checkbox label {
      min-height: 20px;
      padding-left: 0;
      margin-bottom: 0;
      font-weight: normal;
      cursor: pointer;
    }
    .checkbox input[type="checkbox"] {
      position: absolute;
      margin-top: 4px \9;
      margin-left: -20px;
    }
    .checkbox + .checkbox {
      margin-top: 0;
    }
    input[type="checkbox"][disabled],
    input[type="checkbox"].disabled{
      cursor: not-allowed;
    }
    .checkbox {
      padding-left: 20px;
    }
    .checkbox label {
      display: inline-block;
      vertical-align: middle;
      position: relative;
      padding-left: 2px;
    }
    .checkbox label::before {
      content: "";
      display: inline-block;
      position: absolute;
      width: 17px;
      height: 17px;
      left: 0;
      margin-left: -20px;
      border: 1px solid #cccccc;
      border-radius: 3px;
      background-color: #fff;
      -webkit-transition: border 0.15s ease-in-out, color 0.15s ease-in-out;
      -o-transition: border 0.15s ease-in-out, color 0.15s ease-in-out;
      transition: border 0.15s ease-in-out, color 0.15s ease-in-out;
    }
    .checkbox label::after {
      display: inline-block;
      position: absolute;
      width: 16px;
      height: 16px;
      left: 0;
      top: -3px;
      margin-left: -22px;
      padding-left: 3px;
      padding-top: 1px;
      font-size: 18px;
      color: #FFFFFF;
    }
    .checkbox input[type="checkbox"],
    .checkbox input[type="radio"] {
      opacity: 0;
      z-index: 1;
      cursor: pointer;
    }
    .checkbox input[type="checkbox"]:checked + label::before{
        background-image: url(../assets/gou.png);
        border: 1px solid #F04055;
        outline: thin dotted;
        outline: 5px auto -webkit-focus-ring-color;
        outline-offset: -2px;
    }
    .checkbox input[type="checkbox"]:checked + label::after {
      content: " ";
    }
    .checkbox input[type="checkbox"]:disabled + label {
      opacity: 0.65;
    }
    .checkbox input[type="checkbox"]:disabled + label::before{
      background-color: #eeeeee;
      border-color: #cccccc;
      cursor: not-allowed;
    }
    .checkbox.checkbox-circle label::before {
      border-radius: 50%;
    }
    input[type="checkbox"].styled:checked + label:after{
      font-family: 'iconfont';
      content: "\e633";
    }
    input[type="checkbox"] .styled:checked + label::before{
      color: #fff;
    }
    input[type="checkbox"] .styled:checked + label::after{
      color: #fff;
    }
    .checkbox input[type="checkbox"]:disabled + label::after{
      cursor: not-allowed;
      color: #333;
    }
}

</style>
