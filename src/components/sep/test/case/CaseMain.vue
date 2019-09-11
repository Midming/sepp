<template>
<div id="case_main" class="case-main">
  <div class="crumbs">
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item><i class="iconfont icon-bug"></i> 测试管理</el-breadcrumb-item>
          <el-breadcrumb-item>测试用例</el-breadcrumb-item>
      </el-breadcrumb>
  </div>

    <el-dialog :close-on-click-modal="modal_close" :visible.sync="showUploadResult" width="1000px":fullscreen="maximize">
      <div slot="title">
        <span style="font-size:18px">测试用例上传</span>
        <button class="el-dialog__headerbtn" style="right:40px" @click="maximize=!maximize">
          <el-tooltip content="最大化" effect="dark" placement="left">
            <i v-show="maximize==false" class="el-icon-full-screen" style="font-size:14px"></i>
          </el-tooltip>
          <el-tooltip content="还原" effect="dark" placement="left">
            <i v-show="maximize==true" class="el-icon-bottom-left" style="font-size:14px"></i>
          </el-tooltip>
        </button>
      </div>
      <el-collapse v-model="activeCollapse" accordion class="case-upload-collapse">
        <el-collapse-item name="cases">
          <template slot="title">
            <i class="el-icon-document"></i>
            <span style="font-size:15px;font-weight:600"> 上传成功测试用例清单</span>
          </template>        
          <el-table :data="uploadCases" size="mini" stripe empty-text="没有上传成功的测试用例" max-height="300" :border="show_border">
            <el-table-column type="index" label="序号" width="50" align="center">
            </el-table-column>
            <el-table-column prop="case_id" label="用例编号" width="90" align="center">
            </el-table-column>
            <el-table-column prop="status_name" label="用例状态" width="90" align="center">
            </el-table-column>
            <el-table-column prop="designer" label="设计者" width="100" align="center">
            </el-table-column>
            <el-table-column prop="priority_name" label="优先级" width="100" align="center">
            </el-table-column>
            <el-table-column prop="prod_module_name" label="所属模块" width="100" align="center">
            </el-table-column>
            <el-table-column prop="test_type_name" label="测试类型" width="100" align="center">
            </el-table-column>
            <el-table-column prop="test_period_name" label="测试阶段" width="100" align="center">
            </el-table-column>
            <el-table-column prop="name" label="用例标题" header-align="center" :show-overflow-tooltip="true">
            </el-table-column>
          </el-table>
        </el-collapse-item>

        <el-collapse-item name="messages">
          <template slot="title">
            <i class="el-icon-info"></i>
            <span style="font-size:15px;font-weight:600"> 上传失败信息</span>
          </template>
          <el-table :data="uploadMessages" size="mini" stripe empty-text="没有上传失败的记录" max-height="300" :border="show_border">
            <el-table-column type="index" label="序号" width="50" align="center">
            </el-table-column>
            <el-table-column property="message" label="失败消息" header-align="center">
            </el-table-column>
          </el-table>
        </el-collapse-item>
      </el-collapse>
      <div slot="footer">
        <el-button v-no-more-click type="primary" icon="el-icon-close" @click="completeUpload()" size="mini">关闭</el-button>
      </div>
    </el-dialog>

  <el-dialog :close-on-click-modal="modal_close" title="测试用例上传" :visible.sync="showUpload" width="600px">
    <div v-loading.lock="uploading" 
      class="uploading"
      element-loading-text="上载解析中..." 
      element-loading-spinner="el-icon-loading" 
      element-loading-background="rgba(0, 0, 0, 0.8)">
      <el-upload 
        class="case-upload" 
        ref="upload" 
        drag 
        :limit="1"
        :on-exceed="fileExceeded" 
        :before-upload="beforeUpload"
        :on-success="uploadComplete"
        :on-preview="handlePreview" 
        :on-remove="handleRemove" 
        :file-list="fileList" 
        accept=".csv"
        action=""
        :http-request="uploadAction">
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或
          <em>点击上传</em>
        </div>
        <div class="el-upload__tip" slot="tip">仅支持文件编码格式为ANSI/ASCII的CSV文件上载，上载模板请从<span @click="fetchTemplate()">这里</span>下载</div>
        <div class="el-upload__tip" slot="tip" style="color:red">注意一：测试用例各字段中尽量避免使用英文半角逗号，必要时请使用【 /, 】来替代！</div>
        <div class="el-upload__tip" slot="tip" style="color:red">注意二：上传文件编码格式为固定 ANSI/ASCII 格式，请不要随意改动文件编码格式！</div>
      </el-upload>
    </div>
    <div slot="footer">
      <el-button v-no-more-click type="primary" icon="el-icon-circle-close" size="small" @click="showUpload=false">取消</el-button>
      <el-button v-no-more-click type="primary" icon="el-icon-circle-check" size="small" @click="commitUpload()" :disabled="prodDenied() || uploading">确定</el-button>
    </div>
  </el-dialog>

  <el-dialog :close-on-click-modal="modal_close" title="新建 文件夹 / 测试用例" :visible.sync="showNewBox" width="40%">
    <el-form :model="new_form" size="small">
      <el-form-item label="请选择类型" label-width="100px">
        <el-radio-group v-model="new_form.type" size="small">
          <el-radio-button label="folder" :disabled="currenType === 'case'">文件夹</el-radio-button>
          <el-tooltip content="根目录下不允许直接创建测试用例！" placement="right" effect="dark" :disabled="!parentRoot">
            <el-radio-button label="case" :disabled="parentRoot">测试用例</el-radio-button>
          </el-tooltip>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="请输入标题" label-width="100px">
        <el-input v-model="new_form.name" style="width: 95%" clearable></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer">
      <el-button v-no-more-click type="primary" icon="el-icon-circle-close" size="small" @click="onCancel">取消</el-button>
      <el-button v-no-more-click type="primary" icon="el-icon-circle-check" size="small" @click="saveNewNode" :disabled="prodDenied()">确定</el-button>
    </div>
  </el-dialog>

  <el-dialog :close-on-click-modal="modal_close" title="修改 文件夹 / 测试用例" :visible.sync="showModBox" width="40%">
    <el-form :model="mod_form" size="middium">
      <el-form-item label="请选择类型" label-width="100px">
        <el-radio-group v-model="mod_form.type" size="small">
          <el-radio-button label="folder" :disabled="currenType === 'case'">文件夹</el-radio-button>
          <el-radio-button label="case" :disabled="currenType === 'folder'">测试用例</el-radio-button>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="请输入标题" label-width="100px">
        <el-input v-model="mod_form.name"  clearable></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer">
      <el-button v-no-more-click type="primary" icon="el-icon-circle-close" size="small" @click="onCancel">取消</el-button>
      <el-button v-no-more-click type="primary" icon="el-icon-circle-check" size="small" @click="saveModNode" :disabled="prodDenied()">确定</el-button>
    </div>
  </el-dialog>

    <el-dialog :close-on-click-modal="modal_close" :visible.sync="showAddToSenario" width="900px">
      <div slot="title">
        <span style="font-size:18px">添加到测试集</span>
        <span style="font-size:18px;color:red">（须展开至待选目标节点方可有效勾选用例）</span>
      </div>
      <el-form :model="addto" size="small" :inline="true" label-width="80px" class="grant-form">
        <el-form-item label="版本号" required>
          <el-select v-model="addto.releases.selected" placeholder="请选择版本" @change="senarioQuery()" style="width:200px" filterable clearable>
            <el-option v-for="opt in addto.releases.opts" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="测试集" required>
          <el-select v-model="addto.senarios.selected" placeholder="请选择测试集" style="width:400px" filterable clearable>
            <el-option v-for="opt in addto.senarios.opts" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button v-no-more-click type="primary" icon="el-icon-circle-close" @click="showAddToSenario=false" size="small">取消</el-button>
        <el-button v-no-more-click type="primary" icon="el-icon-circle-check" @click="addToSenario()" size="small">确认添加</el-button>
      </div>
    </el-dialog>

  <div class="block" v-bind:style="{height: baseHeight + 'px'}" ref="cases">
    <v-contextmenu ref="contextmenu" v-show="menuDisplay">
      <v-contextmenu-item @click="senario" v-if="!prodDenied()">添加到测试集</v-contextmenu-item>
      <v-contextmenu-item @click="copy">复制 (Ctrl+C)</v-contextmenu-item>
      <v-contextmenu-item @click="paste" v-if="!prodDenied()">粘贴 (Ctrl+V)</v-contextmenu-item>
      <v-contextmenu-item @click="add" v-if="!prodDenied()">新建 (Ctrl+S)</v-contextmenu-item>
      <v-contextmenu-item @click="edit" v-if="!prodDenied()">修改 (Ctrl+E)</v-contextmenu-item>
      <v-contextmenu-item @click="del" v-if="!prodDenied()">删除 (Ctrl+D)</v-contextmenu-item>
    </v-contextmenu>
    <el-tree
      v-if="loadingTree"
      class="tree-view"
      :style="{ height: treeHeight + 'px' }"
      node-key="id"
      ref="folder"
      :key="nodeKey"
      @check="checkProduct"
      @node-expand="expandNode"
      @node-collapse="collapseNode"
      @keydown.ctrl.native ="hotKeyPress"
      highlight-current
      :show-checkbox="true"
      :auto-expand-parent="false"
      :expand-on-click-node="false"
      :props="defaultProps"
      lazy
      :load="loadNode"
      @node-click="nodeClickEvent"
      @current-change="initData"
      @node-contextmenu="showMenu"
      :default-expanded-keys="expanded"
      :render-content="renderContent">
    </el-tree>
    <el-alert type="info" title="提示：不支持批量拷贝用例，不可编辑其他项目用例" :closable="false" show-icon></el-alert>
  </div>

  <div ref="detail" v-show="showDetail" class="detail" v-bind:style="{height: baseHeight + 'px'}"> 
    <el-tabs v-model="activeName" class="sub-tabs" v-bind:style="{height: baseHeight - 4 + 'px'}">
        <el-tab-pane name="base">
          <span slot="label">基本信息</span>
          <v-base 
            class="base" 
            ref="vBase" 
            v-if="showComp('base')" 
            v-bind:style="{height: tableHeight + 'px'}" 
            :caseId="currentId" 
            :currentProd="currentProd" 
            :title="currentTitle"
            :readOnly="false">
          </v-base>
        </el-tab-pane>
        <el-tab-pane name="step">
          <span slot="label">测试步骤</span>
          <v-step v-if="showComp('step')" :caseId="currentId" :currentProd="currentProd"></v-step>
        </el-tab-pane>
        <el-tab-pane name="treq">
          <span slot="label">关联需求</span>
          <v-treq v-if="showComp('treq')" :caseId="currentId" :currentProd="currentProd"></v-treq>
        </el-tab-pane>
        <el-tab-pane name="tbug">
          <span slot="label">关联缺陷</span>
          <v-tbug v-if="showComp('tbug')" :caseId="currentId" :currentProd="currentProd"></v-tbug>
        </el-tab-pane>
    </el-tabs>
  </div>
</div>
</template>

<script>
import vBase from "./CaseInfo.vue";
import vStep from "./CaseStep.vue";
import vTreq from "./CaseReq.vue";
import vTbug from "./CaseBug.vue";
import contextmenu from 'v-contextmenu/src/components/Contextmenu';
import contextmenuitem from 'v-contextmenu/src/components/ContextmenuItem';

let id = 0;
const root_parent = 0;
const root_id = 1;
export default {
  components: {
    vBase,
    vStep,
    vTreq,
    vTbug,
    vContextmenu: contextmenu,
    vContextmenuItem: contextmenuitem
  },

  data: function() {
    return {
      show_border: localStorage.show_border && localStorage.show_border == 'true',
      modal_close: localStorage.modal_close && localStorage.modal_close == 'true' ? true : false,
      currentId: "",
      currentTitle: "",
      maximize: false,
      parentRoot: false,
      currenType: "",
      currentProd: "",
      loadingTree: true,
      firstLoad: true,
      showUpload: false,
      uploading: false,
      showUploadResult: false,
      uploadMessages: [],
      uploadCases: [],
      activeCollapse: "cases",
      modules: [],
      filePath: "",
      fileList: [],
      nodeKey: "",
      expanded: [],
      defaultProps: {
        children: 'children',
        label: "name",
        isLeaf: 'leaf',
        product_id: parseInt(sessionStorage.product_id)
      },
      showAddToSenario: false,
      addto: {
        releases: {
          selected: "",
          opts: []
        },
        senarios: {
          selected: "",
          opts: []
        }
      },
      activeName: "",
      showNewBox: false,
      showModBox: false,
      new_form: {
        name: "",
        type: "case"
      },
      mod_form: {
        name: "",
        type: "",
        id: "",
        parent_id: ""
      },
      showDetail: false,
      baseHeight: "",
      treeHeight: "",
      tableHeight: "",
      tempData: [],
      tempNode: {},
      menuDisplay: false,
      index: 0,
      currnetMenu: {},
      copiedMenus: []
    };
  },

  created() {
    this.expanded.splice(0, this.expanded.length);
    this.expanded.push(root_id);
  },

  mounted() {
    this.baseHeight = bodyAviHeightNTab + 45;
    this.treeHeight = bodyAviHeightNTab;
    this.tableHeight = bodyAviHeightNTab - 25;
  },

  methods: {
    loadNode(node, resolve) {
        if (node.level === 0) {
          this.folderQuery(0, resolve);
        }
        
        if (node.level >= 1){
          this.folderQuery(node.data.id, resolve);
        }
    },

    expandNode(data, node, self){
      this.$refs.folder.setCurrentNode(data);
      this.expanded.push(data.id);
    },

    collapseNode(data, node, self){
      let index = this.expanded.indexOf(data.id);
      if (index > -1) {
        this.expanded.splice(index, 1);
      }
    },

    checkProduct(cNodes, cKeys, halfCNodes, halfCKeys){
      let deniedOper = 0;
      for (let i = 0; i < cKeys.checkedNodes.length; i ++) {
        if (cKeys.checkedNodes[i].product_id != parseInt(sessionStorage.product_id)) {
          this.$refs.folder.setChecked(cKeys.checkedKeys[i], false, false);
          deniedOper ++;
        }
      }
      if (deniedOper > 0) {
        this.$message.warning("请不要选择其他项目的用例！");
      }
    },

    prodDenied(){
      return this.currentProd != -1 && this.currentProd != parseInt(sessionStorage.product_id);
    },

    folderQuery(parent_id, resolve) {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/tree_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            "parent_id" : parent_id
          }
        })
        .then(function(res) {
          let json = eval(res.data);
          if (parent_id > 0) {
            let node = _self.$refs.folder.getNode(parent_id);
            _self.$set(node.data, "children", []);
            json.forEach(item => {
              item.leaf = (item.type === 'case');
              node.data.children.push(item);
            });
          }
          if (typeof(resolve) == 'function') {
            resolve(json);
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    showComp(tabName) {
      return tabName == this.activeName;
    },

    initData(){
      this.menuDisplay = false; 
      this.index = 0;
    },

    hotKeyPress(e){
      e.preventDefault();
      let crtData = this.$refs.folder.getCurrentNode();
      let crtNode = this.$refs.folder.getNode(crtData.id);
      this.currnetMenu = {
        data: crtData,
        event : e,
        node : crtNode
      };

      if (e.keyCode === 67) {
        this.copy();
      } else if (e.keyCode === 68 && !this.prodDenied()) {
        this.del();
      } else if (e.keyCode === 69 && !this.prodDenied()) {
        this.edit();
      } else if (e.keyCode === 83 && !this.prodDenied()) {
        this.add();
      } else if (e.keyCode === 86 && !this.prodDenied()) {
        this.paste();
      }
    },

    copy(vm, event){
      this.itemClicked();
      let arr = [];
      arr.push(this.currnetMenu);
      this.copiedMenus.splice(0, this.copiedMenus.length);
      this.copiedMenus = arr.concat();
    },

    senario(vm, event){
      this.itemClicked();
      if (this.currnetMenu.data.product_id != parseInt(sessionStorage.product_id)) {
        this.$message.warning("请勿跨产品操作编辑功能！");
        return;
      }
      let checkedNodes = this.$refs.folder.getCheckedNodes(true);
      if (!checkedNodes || checkedNodes.length === 0) {
        this.$message.warning("未有效选择测试用例，请展开文件夹！");
        return;
      }
      this.showAddToSenario = true;
      this.releaseQuery();
      this.addto.releases.selected = "";
      this.addto.senarios.selected = "";
    },

    paste(vm, event){
      this.itemClicked();
      if (this.currnetMenu.data.product_id != parseInt(sessionStorage.product_id)) {
        this.$message.warning("请勿跨产品操作编辑功能！");
        return;
      }
      this.index ++;
      this.caseInfoPaste(this.copiedMenus[0].node, this.copiedMenus[0].data);
    },

    add(vm, event){
      this.itemClicked();
      if (this.currnetMenu.data.product_id != parseInt(sessionStorage.product_id)) {
        this.$message.warning("请勿跨产品操作编辑功能！");
        return;
      }
      this.append(this.currnetMenu.data);
    },

    edit(vm, event){
      this.itemClicked();
      if (this.currnetMenu.data.product_id != parseInt(sessionStorage.product_id)) {
        this.$message.warning("请勿跨产品操作编辑功能！");
        return;
      }
      this.modify(this.currnetMenu.node, this.currnetMenu.data)
    },

    del(vm, event){
      this.itemClicked();
      if (this.currnetMenu.data.product_id != parseInt(sessionStorage.product_id)) {
        this.$message.warning("请勿跨产品操作编辑功能！");
        return;
      }
      this.remove(this.currnetMenu.node, this.currnetMenu.data)
    },

    itemClicked(){
      this.menuDisplay = false;
      document.body.removeEventListener('click', this.bodyClick);
    },

    showMenu(e, d, n, s){
      this.menuDisplay = true;
      this.$refs.contextmenu.style="position:absolute; left: " + e.pageX + "px;top: " + e.pageY + "px";
      document.body.addEventListener('click', this.bodyClick);
      this.currnetMenu = {
        data: d,
        event : e,
        node : n
      };
    },

    bodyClick (e){
      this.menuDisplay = false;
    },

    onCancel() {
      this.showNewBox = false;
      this.showModBox = false;
    },

    expandDetail(){
      this.showDetail = true;
      this.$refs.cases.style.width = '35%';
      this.$refs.detail.style.width = '64%';
    },

    closeDetail(){
      this.showDetail = false;
      this.$refs.cases.style.width = '100%';
    },

    nodeClickEvent(data, node, _self){
      this.menuDisplay = false;
      this.currentId = data.id;
      this.currentTitle = data.name;
      this.currenType = data.type;
      this.currentProd = data.product_id;
      if (data.type === 'case') {
        this.expandDetail();
        if (this.activeName != "base") {
          this.firstLoad = true;
        }
        this.activeName = "base";
        if (!this.firstLoad && this.showDetail) {
            this.$refs.vBase.queryCaseInfo(data.id);
        }
        this.firstLoad = false;
      } else {
        this.closeDetail();
      }
      this.$refs.folder.setCurrentNode(data);
    },

    caseInfoPaste(node, data) {
      let _self = this;
      let nData = _self.$refs.folder.getCurrentNode();
      let actNode = (nData.type != "folder") ? _self.$refs.folder.getNode(nData).parent.data : nData;
      let name = data.name + " - " + _self.index;
      let type = data.type;
      
      _self
        .$axios({
          method: "post",
          url: "/testing/case_node_paste",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            type : type,
            name : name,
            parent_id : actNode.id,
            origin_id: data.id,
            designer: sessionStorage.user_id
          }
        })
        .then(function(res) {
          if (res.data >= 1) {
            _self.$message.success("测试用例复制成功");
            const newChild = { //刷新之前只在前端将新节点添加，不需要与后端强制同步，节省开销
              id: res.data, 
              name: name, 
              product_id: parseInt(sessionStorage.product_id),
              children: [], 
              type: type
            };
            if (!actNode.children) {
              _self.$set(actNode, "children", []);
            }
            actNode.children.push(newChild);
            _self.expanded.push(newChild.id);
            _self.$refs.folder.setCurrentNode(actNode);
          } else {
            _self.$message.warning("测试用例复制失败");
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    import(data) {
      let _self = this;
      _self.showUpload = true;
      _self.fileList = [];
    },

    commitUpload(){
      let _self = this;
      _self.uploading = true;
      _self.$axios({
        method: "post",
        url: "/testing/case_upload",
        headers: {
          "Content-type": "application/x-www-form-urlencoded"
        },
        params: {
          file_path: _self.filePath
        }
      })
      .then(function(res) {
        _self.uploadMessages = eval(res.data.messages);
        _self.uploadCases = eval(res.data.cases);
        _self.$nextTick(() => {
          _self.uploading = false;
          _self.showUpload = false;
          _self.showUploadResult = true;
          _self.moduleQuery();
        });
      })
      .catch(function(response) {
        _self.$notify.error("用例上载解析过程中发生错误！");
        _self.uploading = false;
        console.log(response);
      });

    },

    completeUpload(){
      let _self = this;
      _self.showUploadResult = false;
      if (_self.uploadCases.length == 0) {
        return;
      }
      _self.nodeKey = +new Date();
    },

    moduleQuery() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/requirements/module_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          }
        })
        .then(function(res) {
          _self.modules = eval(res.data);
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    append(data) {
      let node = this.$refs.folder.getNode(data.id);
      if (node.level > 5 && data.type === "folder") {
        this.$message.warning("最多只能有5级目录，请重新组织");
        return;
      }
      if (data.id <= 1) {
        this.parentRoot = true;
        this.new_form.type = "folder";
      } else {
        this.parentRoot = false;
      }
      if (data.type != "folder") {
        this.tempData = node.parent.data;
      } else {
        this.tempData = data;
      }
      this.showNewBox = true;
    },

    saveNewNode(d) {
      let _self = this;
      let data = _self.tempData;
      let name = _self.new_form.name;
      const data_type = _self.new_form.type;

      if (null == name || '' == name) {
        _self.$message.warning("标题不能为空！");
        return;
      }
      if (name.length > 20) {
        _self.$message.warning("标题长度不能超过20字符！");
        return;
      }
      
      if (data.children) {
        for (let i = 0; i < data.children.length; i ++){
          if (data.children[i].name === name && data.children[i].type === data_type) {
            _self.$message.warning("存在相同标题的 " + data_type);
            return;
          }
        }
      }
      
      _self.$axios({
        method: "post",
        url: "/testing/case_node_save",
        headers: {
          "Content-type": "application/x-www-form-urlencoded"
        },
        params: {
          type : data_type,
          name : name,
          parent_id : data.id,
          designer: sessionStorage.user_id
        }
      })
      .then(function(res) {
        _self.showNewBox = false;
        _self.nodeKey = +new Date();
        _self.$message.success("新增保存成功");
      })
      .catch(function(response) {
        _self.$notify.error("发生错误");
        console.log(response);
      });
    },

    remove(node) {
      let _self = this;
      let type_cn = (node.data.type === 'folder') ? '文件夹' : '测试用例'; 
      _self.$confirm("确认删除 [" + type_cn + "]  [" + node.data.name + "] 吗?", "操作确认", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
      .then(() => {
        _self.$axios({
          method: "post",
          url: "/testing/case_node_delete",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            "id" : node.data.id,
            "type" : node.data.type
          }
        })
        .then(function(res) {
          if (res.data === 1) {
            const parent = node.parent;
            _self.nodeKey = +new Date();
            _self.$message.success("删除成功");
            setTimeout(() => {
            _self.$refs.folder.setCurrentNode(parent);
            }, 500);
          } else {
            _self.$notify.error("删除失败，请刷新本页数据！");
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
      })
      .catch(() => {});
    },

    modify(node, data) {
      this.showModBox = true;
      this.tempNode = node;
      this.tempData = data;

      this.mod_form.type = data.type;
      this.mod_form.name = data.name;
      this.mod_form.id = data.id;
      this.mod_form.parent_id = data.parent_id;
    },

    saveModNode() {
      let _self = this;
      let data = _self.tempData;
      let node = _self.tempNode;
      if (null == _self.mod_form.name || '' == _self.mod_form.name) {
        _self.$notify.error("标题不能为空！");
        return;
      }
      if (_self.mod_form.name.length > 20) {
        _self.$notify.error("标题长度不能超过20字符！");
        return;
      }
      
      const parent = node.parent;
      const children = parent.data.children || parent.data;

      for (let i = 0; i < children.length; i ++){
        if (children[i].name === _self.mod_form.name && children[i].type === _self.mod_form.type && data.name != _self.mod_form.name) {
          _self.$message.warning("存在相同标题的 " + _self.mod_form.type);
          return;
        }
      }
      _self.showModBox = false;

      data.type = _self.mod_form.type;
      data.name = _self.mod_form.name;
      data.id = _self.mod_form.id;
      data.parent_id = _self.mod_form.parent_id;

      _self.$axios({
        method: "post",
        url: "/testing/case_node_update",
        headers: {
          "Content-type": "application/x-www-form-urlencoded"
        },
        params: {
          "id" : data.id,
          "type" : data.type,
          "name" : data.name,
          "parent_id" : data.parent_id
        }
      })
      .then(function(res) {
        if (res.data === 1) {
          const index = children.findIndex(d => d.id === data.id);
          children.splice(index, 1, data);
          _self.$message.success("修改成功");
        } else {
          _self.$notify.error("修改失败，没有记录更新");
        }
      })
      .catch(function(response) {
        _self.$notify.error("发生错误");
        console.log(response);
      });
    },

    addToSenario() {
      let _self = this;
      let checkedNodes = _self.$refs.folder.getCheckedNodes(true);
      if (!checkedNodes || checkedNodes.length === 0) {
        this.$message.warning("请选择测试用例");
        return;
      }
      let checkedCases = [];
      for (let k = 0; k < checkedNodes.length; k++) {
        if (checkedNodes[k].type === "case" && checkedNodes[k].product_id === parseInt(sessionStorage.product_id)) {
          checkedCases.push(checkedNodes[k].id + "");
        }
      }
      let selectedSenario = {};
      for (let i = 0; i < _self.addto.senarios.opts.length; i++) {
        if (_self.addto.senarios.opts[i].value === _self.addto.senarios.selected) {
          selectedSenario = _self.addto.senarios.opts[i];
        }
      }

      let originCases = selectedSenario.cases ? selectedSenario.cases.split(",") : [];
      let newCases = [];
      if (originCases.length === 0) {
        newCases = checkedCases;
      } else {
        let restCases = checkedCases.filter(function(d) {
          return originCases.indexOf(d) === -1;
        });
        newCases = originCases.concat(restCases);
      }

      if (newCases.length <= originCases.length) {
        _self.$message.info("所选用例已全部在目标测试集中");
        _self.showAddToSenario = false;
        return;
      }

      _self
        .$axios({
          method: "post",
          url: "/testing/test_senario_update",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            id: _self.addto.senarios.selected,
            name: selectedSenario.label,
            cases: newCases.toString()
          }
        })
        .then(function(res) {
          if (res.data > 0) {
            _self.$message.success("成功加入测试集！");
          } else {
            _self.$message.warning("加入测试集失败！");
          }
          _self.showAddToSenario = false;
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    releaseQuery(row, event, column) {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/requirements/release_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          }
        })
        .then(function(res) {
          let json = eval(res.data.list);
          _self.addto.releases.opts.splice(0, _self.addto.releases.opts.length);
          for (let i = json.length - 1; i >= 0; i--) {
            _self.addto.releases.opts.push({
              value: json[i].rel_id,
              label: "[" + json[i].branchName + "] " + json[i].rel_code
            });
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    senarioQuery() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/test_senario_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            rel_id: _self.addto.releases.selected
          }
        })
        .then(function(res) {
          let json = eval(res.data);
          _self.addto.senarios.opts.splice(0, _self.addto.senarios.opts.length);
          for (let i = json.length - 1; i >= 0; i--) {
            _self.addto.senarios.opts.push({
              value: json[i].id,
              label: json[i].name,
              cases: json[i].cases
            });
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    editDenied(product_id){
      return parseInt(product_id) != parseInt(sessionStorage.product_id);
    },

    renderContent(h, { node, data, store }) {
      if (node.level === 1) {
        return (
          <span class="folder-tree">
            <span>
              <i class="iconfont icon-folder" style="margin-right: 4px"/>
              {node.label}
            </span>
            <span>
              <el-button v-no-more-click size="small" type="text" on-click={() => this.import(data)} >
                <i class="el-icon-upload2"></i>
              </el-button>
              <el-button v-no-more-click size="small" type="text" on-click={() => this.append(data)} >
                <i class="iconfont icon-addition"></i>
              </el-button>
            </span>
          </span>
        );
      }
      if (this.editDenied(data.product_id)) {
        if (data.type === 'folder') {
          return (
            <span class="folder-tree">
              <span>
                <i class="iconfont icon-folder" style="margin-right: 4px"/>
                {node.label}
              </span>
            </span>
          );
        } else {
          return (
            <span class="folder-tree">
              <span>
                <i class="el-icon-document" style="margin-right: 4px" />
                {node.label}
              </span>
            </span>
          );
        }
      } else {
        if (data.type === 'folder') {
          if (node.isLeaf) {
            return (
              <span class="folder-tree">
                <span>
                  <i class="iconfont icon-folder" style="margin-right: 4px"/>
                  {node.label}
                </span>
                <span>
                  <el-button v-no-more-click size="small" type="text" on-click={() => this.append(data)} >
                    <i class="iconfont icon-addition"></i>
                  </el-button>
                  <el-button v-no-more-click size="small" type="text" on-click={() => this.modify(node, data)} >
                    <i class="el-icon-edit-outline"></i>
                  </el-button>
                  <el-button v-no-more-click size="small" type="text" on-click={() => this.remove(node, data)} >
                    <i class="el-icon-delete"></i>
                  </el-button>
                </span>
              </span>
            );
          } else { //非叶子节点的文件夹不允许删除
            return (
              <span class="folder-tree">
                <span>
                  <i class="iconfont icon-folder" style="margin-right: 4px" />
                  {node.label}
                </span>
                <span>
                  <el-button v-no-more-click size="small" type="text" on-click={() => this.append(data)} >
                    <i class="iconfont icon-addition"></i>
                  </el-button>
                  <el-button v-no-more-click size="small" type="text" on-click={() => this.modify(node, data)} >
                    <i class="el-icon-edit-outline"></i>
                  </el-button>
                </span>
              </span>
            );
          }
        } else {
          return (
            <span class="folder-tree">
              <span>
                <i class="el-icon-document" style="margin-right: 4px" />
                {node.label}
              </span>
              <span>
                <el-button v-no-more-click size="small" type="text" on-click={() => this.append(data)} >
                  <i class="iconfont icon-addition"></i>
                </el-button>
                <el-button v-no-more-click size="small" type="text" on-click={() => this.modify(node, data)} >
                  <i class="el-icon-edit-outline"></i>
                </el-button>
                <el-button v-no-more-click size="small" type="text" on-click={() => this.remove(node, data)} >
                  <i class="el-icon-delete"></i>
                </el-button>
              </span>
            </span>
          );
        }
      }
    },
    
    beforeUpload(file){
      if (file.size / 1024 / 1024 > 2) {
        this.$message.info("上传文件不能超过 2MB！");
        return false;
      }
      if (file.name == "") {
        this.$message.info("文件名不能为空，请重命名！");
        return false;
      }
      const fileType = file.name.split(".");
      if (fileType[fileType.length - 1].toUpperCase() != "CSV") {
        this.$message.info("仅支持CSV格式的文件上传！");
        return false;
      }
    },

    fileExceeded(files, fileList) {
      this.$message.info("每次只能上载一个文件！");
      return;
    },

    uploadComplete(res, file, fileList) {
      let _self = this;
      _self.fileList = fileList;
    },

    handleRemove(file, fileList) {
      let _self = this;
      let fileId = null;
      if (!file || (!file.id && !file.response)) {
        return; 
      } else {
        fileId = file.id ? file.id : file.response[0].id
      }
      
      _self
        .$axios({
          method: "post",
          url: "/file/file_delete",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            id: fileId
          }
        })
        .then(function(res) {
          if (res.data > 0) {
            _self.fileList = fileList;
            _self.$message.success("文件删除成功！");
          } else {
            _self.$message.info("文件删除失败");
            console.log(res);
          }
        })
        .catch(function(response) {
          _self.$notify.error("文件删除时发生程序错误！");
          console.log(response);
        });
    },

    handlePreview(file) {
      commonQuery.fileDownload(file);
    },

    fetchTemplate(){
      window.open("/static/files/测试用例上载模板.csv");
    },

    uploadAction(params){
      let _self = this;
      let file = params.file;
      let fileList = _self.fileList || [];
      
      let formData = new FormData();
      formData.append("file", file);

      _self
        .$axios({
          method: "post",
          url: "/file/file_upload",
          headers: {
            "Content-type": "multipart/form-data"
          },
          params: {
            useage: "testcase",
            file_name:file.name

          },
          data: formData
        })
        .then(function(res) {
          if (res.data.length > 0 && res.data[0].id > 0) {
            _self.$message.success("文件上传成功！");
            fileList.push(res.data[0]);
            _self.fileList = fileList;
            _self.filePath = res.data[0].url;
          } else {
            _self.$message.info("文件上传失败！");
            console.log(res);
          }
        })
        .catch(function(response) {
          _self.$notify.error("文件上传时发生程序错误");
          console.log(response);
        });
    }
  }
};
</script>

<style>
@import '~v-contextmenu/dist/index.css';
.folder-tree {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  font-family: 微软雅黑;
  padding-right: 15px;
}

.block {
  width: 100%;
  float: left;
  border: 1px solid #e4edf3db;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  border-radius: 2px;
}

.block .el-alert {
  border-bottom: 1px solid #e4edf3db;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  border-bottom-left-radius: 3px;
  border-bottom-right-radius: 3px;
}

.tree-view {
  padding: 5px 0;
  overflow-y: auto;
  font-size: 13px;
  width: 100%;
  background-color: #f0f8ff40;
}

.tree-view .el-button+.el-button {
  margin-left: 5px !important;
}

.tree-view .iconfont {
  font-size: 17px;
}

.tree-view i {
  font-size: 16px;
}

.tree-view .el-tree-node__expand-icon {
  font-size: 16px;
  font-weight: 600;
}

.detail {
  width: 61%;
  float: right;
  border: 1px solid #e4edf3db;
  background-color: #f0f8ff40;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  border-radius: 2px;
  padding: 0 15px 10px 15px;
}

.sub-tabs {
  border-radius: 2px;
}

.sub-tabs .el-tabs__content {
  padding: 0;
  overflow-y: auto;
}

.v-contextmenu{
  padding: 5px;
}

.v-contextmenu .v-contextmenu-item {
  padding: 8px 15px;
}

.case-upload,
.case-upload input {
  width: 560px !important;
}

.case-upload .el-upload--text {
  width: 100%;
  height: 100px;
}

.case-upload .el-upload-dragger {
  width: 100%;
  height: 100px;
  padding: 10px 0;
}

.case-upload .el-upload-dragger .el-icon-upload {
  margin: 0;
  font-size: 50px;
}

.case-upload .el-upload__text {
  font-size: 15px;
}

.case-upload .el-upload__tip {
  font-size: 13px;
  text-align: center;
}

.case-upload .el-upload__tip span {
  color:#3AB4D7;
  cursor:pointer;
  font-weight:600;
}

.case-upload-collapse i {
  font-size: 15px;
}

.case-upload-collapse .el-collapse-item__arrow {
  font-weight: 600;
  line-height: 45px;
}
</style>
