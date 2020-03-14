
 
<template>
<!-- 只是记录封装 -->
    <div class="treeTmp">
      <el-tree
        ref="tree"
        :props="lableKey"
        :load="loadNode"
        lazy
        :highlight-current="true"
        :show-checkbox="showCheckbox"
        @check-change="handleCheckChange"
        @current-change="chang"
        @node-click="handleClick"
        node-key="id"
      >
      </el-tree>
    </div>
  </template>
  
  <script>
  // import system from "@/api/system.js"
  import axios from 'axios';
  export default {
    name: "zElTree",
    props: {
      // 是否复选框
      showCheckbox: {
        type: Boolean,
        default: false
      },
      limitAssetType:{
        type:String,
        default:'floor'
      },
      reUploadId:String,
      // 请求数据接口
      // fetchData: {
      //   type: Function,
      //   // default: system.getTreeData
      //   default:axios.post('/os_kernel_assetctr/app/asset/getAssetTree')
      // },
      // 对应名称
      lableKey: {
        type: Object,
        default:() => {
          return {
            label: "assetSimpleName",
            children: "zones"
          }
        }
      },
      // 首层Id
      fisrtId:{
        type:Number,
        default:0
      },
      // 首层tree
      firstTree: {
        type: Array,
        default: () => [
          { assetSimpleName: "优托邦", id: "0bd285d0e00611e99de100155d1b3d06",assetType:'hq',status:1 }
        ]
      }
    },
    data() {
      return {
        props: {
          label: "assetSimpleName",
          children: "zones"
        },
        resolve:null,
        nodeId:null,
        assetType:null,
        count: 1,
        treeData: [],
        node:null
      }
    },
    watch: {
      reUploadId: function(newVal, oldVal) {
  
        this.$nextTick(() => {
          if(this.assetType == this.limitAssetType ){
            // this.treeData = []
            // setTimeout(() => {
            //   this.resolve(this.treeData)
            // }, 600)
          }else{
            this.fetchTreeData(this.nodeId,this.resolve,this.node,'val')
          }
  
        });
      }
    },
    mounted() {
      // this.fetchTreeData()
    },
    methods: {
      handleClick(data,node,item){
        this.node = node
        this.assetType = node.data.assetType
        console.log('nodeclick',node)
  
      },
      fetchTreeData(id,resolve,node,val) {
        const parmasId = id || 0
        // this.fetchData({ id: parmasId })
        axios.post('/os_kernel_assetctr/app/asset/getAssetTree',{
          id: parmasId
        })
        .then(res => {
          if (res.data.code === 0) {
            this.treeData = res.data.result || []
          } else {
            this.treeData = []
          }
          if(val){
            var theChildren =node.childNodes
            theChildren.splice(0, theChildren.length)
            node.doCreateChildren(this.treeData)
          }else{
             resolve(this.treeData)
          }
        })
      },
      handleCheckChange(data, checked, indeterminate) {
  
        const nodes = this.$refs.tree.getCheckedNodes()
        const checkIds = this.$refs.tree.getCheckedKeys()
        console.log('nodes',nodes)
        console.log('checkIds',checkIds)
        this.$emit("getCheckedNodes", nodes)
        this.$emit("getCheckedKeys", checkIds)
      },
      handleNodeClick(data) {
        console.log(data)
      },
      // 编写递归获取父节点信息
      findTopNode(node,assetCodeDto){
  
         if (node && node.level == 5) {
            assetCodeDto.floorCode = node.data.assetSimpleName
         }
         if (node && node.level == 4) {
            assetCodeDto.buildingCode = node.data.assetSimpleName
         }
         if (node && node.level == 3) {
            assetCodeDto.areaCode = node.data.assetSimpleName
         }
         if (node && node.level == 2) {
            assetCodeDto.projectCode = node.data.assetSimpleName
         }
         if (node && node.level == 1) {
           return false
         }
         let parentNode = node.parent
         this.findTopNode(parentNode, assetCodeDto);
      },
      chang(data,node, indeterminate) {
        let condition = {}
        var assetCodeDto = {};
        this.findTopNode(node, assetCodeDto);
        condition.assetCodeDto = assetCodeDto
        condition.data = data
        this.nodeId = data.id
        console.log('node',node)
        this.$emit("currentNode", condition)
        // this.$emit("currentNode", data)
      },
      loadNode(node, resolve) {
        // this.nodeId = ""
        this.resolve = resolve
  
        if (node.level === 0) {
          // this.nodeId = this.fisrtId
          this.fetchTreeData(this.fisrtId,resolve,node)
        }
        if (node.level > 0) {
  
          if( node.data.assetType == this.limitAssetType ){
            this.treeData = []
            setTimeout(() => {
              resolve(this.treeData)
            }, 600)
          }else{
            // console.log('获取子级')
            this.fetchTreeData(node.data.id,resolve,node)
  
          }
        }
        // if( node.data.assetType == this.limitAssetType )
        // setTimeout(() => {
        //   resolve(this.treeData)
        // }, 600)
      }
    }
  }
  </script>
  <style lang="scss">
  .treeTmp {
    .el-tree {
      min-height: 20rem;
      max-height: 38rem;
      overflow: auto;
    }
  }
  .el-tree-node:focus>.el-tree-node__content{
    background: #ccc;
  }
  </style>
  