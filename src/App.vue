<template>
  <section id="app">
  <treeBar :options="menuOptions"></treeBar>
  </section>
</template>

<script>
import treeBar from './tree';
import demoData from './config/demoCompanys.json';
export default {
  name: 'app',
  data () {
    return {
      menuOptions: {},
      comData: [],
      treeComData: [],
      isClick: false,
      selected: null,
      selectComId: []
    }
  },
  created () { // 视图被创建
    let self = this;
    self.comData = demoData.result.list || [];
    self.comData.forEach(val => {
      let index = self.selectComId.indexOf(val.id);
      let obj = {
        id: val.id,
        name: val.name,
        parentId: val.parentId,
        level: ''
      };
      if (!self.isClick && (self.selectComId.indexOf('ALL') === 0 || index >= 0)) {
        obj.isChecked = true;
      } else if (self.isClick && index >= 0) {
        obj.active = true;
      }
      self.treeComData.push(obj);
    });
  },
  methods: {
    alerts () {
      this.alertOptions = {
        content: '多选日期',
        isShow: true
      };
    },
    dialogCs () {
      this.dialogOptions = {
        txt: '多选日期?',
        isShow: true,
        success: () => {
          console.log('sdfsd');
        }
      };
    }
  },
  watch: {
    treeComData (val) {
      let self = this;
      if (val.length === self.comData.length && self.comData.length !== 0 && val.length > 0) {
        let temp = val;
        self.menuOptions = {
          hasCheck: !self.isClick,
          hasClick: self.isClick,
          treeData: temp,
          isSync: true,
          onGetCurrentItem: (item) => {
            typeof self.selected === 'function' && self.selected(item);
          },
          onSelectedNode: (item) => {
            typeof self.selected === 'function' && self.selected(item);
          }
        };
      }
    }
  },
  components: {
    treeBar
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.btn {
  transition: all 0.2s ease-out;
  cursor: pointer;
  display: inline-block;
  text-align: center;
  -webkit-transition: -webkit-box-shadow 0.2s ease-out;
  -moz-transition: -moz-box-shadow 0.2s ease-out;
  -o-transition: box-shadow 0.2s ease-out;
  color: white;
  border-radius: 4px;
  padding: 6px 12px;
  font-size: 14px;
  font-weight: 400;
  font-family: 'Microsoft Yahei', Helvetica, Arial, sans-serif;
}
.btn-mg{
  margin-bottom: 3px;
  margin-right: 3px;
}
.btn-danger {
  background-color: #F04055;
  border-color: #F04055;
  &:hover{
    background-color: #db3549;
  }
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
section{
  display: block;
}
</style>
