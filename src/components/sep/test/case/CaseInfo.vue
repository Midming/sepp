<template>
  <div class="case-info">
    <el-form :model="form" :inline="true" size="mini" :rules="baseInfoRules" ref="baseInfoForm">
      <el-form-item class="autocase">
        <template slot="label">
          <el-tooltip content="点击用例标题查看更多测试用例信息" placement="left" effect="dark">
            <span>用例ID-标题</span>
          </el-tooltip>
        </template>
        <el-popover
          placement="bottom-start"
          width="380"
          trigger="click">
          <el-form label-position="right" size="mini" :inline="false" label-width="150px">
            <el-form-item label="被执行次数：">
              <el-input v-model="form.executed_times" disabled></el-input>
            </el-form-item>
            <el-form-item label="关联缺陷个数：">
              <el-input v-model="form.related_bug" disabled></el-input>
            </el-form-item>
            <el-form-item label="覆盖产品需求个数：">
              <el-input v-model="form.related_req" disabled></el-input>
            </el-form-item>
            <el-form-item label="用例设计时间：">
              <el-input v-model="form.design_date" disabled></el-input>
            </el-form-item>
            <el-form-item label="最后执行时间：">
              <el-input v-model="form.last_executed" disabled></el-input>
            </el-form-item>
            <el-form-item label="最新执行结果：">
              <el-select v-model="form.test_result" disabled>
                <el-option v-for="opt in test_results" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <el-input v-model="caseTitle" disabled class="read-content-tips" slot="reference">{{caseTitle}}</el-input>
        </el-popover>
      </el-form-item>
      <el-form-item class="button-item">
        <el-tooltip content="您没有本产品权限，无法操作！" placement="left" effect="dark" :disabled="!edit_forbidden">
          <el-button v-no-more-click size="mini" type="primary" icon="el-icon-circle-check" @click="checkSaveCaseInfo('baseInfoForm')" :disabled="edit_forbidden">保存</el-button>
        </el-tooltip>
      </el-form-item>
      <br>
      <el-form-item label="产品模块" prop="prod_module" v-if="!edit_forbidden">
        <el-select v-model="form.prod_module" placeholder="请选择（必选）" clearable filterable :disabled="readOnly">
          <el-option v-for="opt in modules" :value="opt.module_id" :key="opt.module_id" :label="opt.module_name"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="产品模块" v-else>
        <el-input v-model="form.prod_module_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="优先级" prop="priority" required v-if="!edit_forbidden">
        <el-select v-model="form.priority" placeholder="请选择" filterable :disabled="readOnly">
          <el-option v-for="opt in priorities" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="优先级" v-else>
        <el-input v-model="form.priority_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="用例状态" required v-if="!edit_forbidden">
        <el-select v-model="form.case_status" placeholder="请选择" :disabled="readOnly">
          <el-option v-for="opt in case_statuss" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="用例状态" v-else>
        <el-input v-model="form.status_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="测试阶段" required v-if="!edit_forbidden">
        <el-select v-model="form.test_period" placeholder="请选择" filterable :disabled="readOnly">
          <el-option v-for="opt in test_periods" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="测试阶段" v-else>
        <el-input v-model="form.test_period_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="测试类型" required v-if="!edit_forbidden">
        <el-select v-model="form.test_type" placeholder="请选择" :disabled="readOnly">
          <el-option v-for="opt in test_types" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="测试类型" v-else>
        <el-input v-model="form.test_type_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="设计者">
        <el-input v-model="form.designer_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="回归标识" v-if="!edit_forbidden">
        <el-select v-model="form.regress_mark.selected" placeholder="请选择" filterable :disabled="readOnly">
          <el-option v-for="opt in form.regress_mark.opts" :value="opt.value" :key="opt.value" :label="opt.label"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="回归标识" v-else>
        <el-input v-model="form.regress_mark_name" disabled></el-input>
      </el-form-item>
      <el-form-item label="测试脚本" class="autocase" prop="auto_path">
        <el-input v-model="form.auto_path" placeholder="例如：class:com.xx.TestABC 或 testng:~/tests/suiteEFG.xml" spellcheck="false" clearable :disabled="readOnly || edit_forbidden"></el-input>
      </el-form-item>
      <br>
      <el-form-item label="前置条件" class="case-desc" prop="pre_condition">
        <el-input v-model="form.pre_condition" type="textarea" :rows="4" placeholder="请输入前置条件" resize="none" :maxlength="2000" show-word-limit clearable :disabled="readOnly || edit_forbidden">前置条件</el-input>
      </el-form-item>
      <el-form-item label="用例描述" class="case-desc" prop="summary">
        <simple-mde v-model="form.summary" :configs="configs" ref="caseDetail" v-if="loadMde"></simple-mde>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { dateFormat } from "@/util/date.js";
import simpleMde from "vue-simplemde/src/markdown-editor";
let id = 0;
const root_parent = 0;
const root_id = 1;
const test_patern = /^(class:|test:).*(\.java|\.py|\.sql)$/;
const suite_patern = /^(suite:|testng:).*\.xml$/;
export default {
  data: function() {
    return {
      test_results: [],
      test_types: [],
      priorities: [],
      test_periods: [],
      case_statuss: [],
      form: {
        tips: "请完成测试用例信息编辑之后保存修改，否则信息将丢失",
        edit_forbidden: false,
        product: {
          selected: sessionStorage.product_id,
          opts: [
            {
              value: sessionStorage.product_id,
              label: sessionStorage.product_name
            }
          ]
        },
        case_id: "",
        case_status: 1,
        designer: parseInt(sessionStorage.user_id),
        priority: 2,
        test_type: 1,
        prod_module: "",
        test_period: 4,
        regress_mark: {
          selected: "N",
          opts: [
            {
              value: "Y",
              label: "是"
            },
            {
              value: "N",
              label: "否"
            }
          ]
        },
        auto_path: "",
        summary: "",
        pre_condition: "",
        last_executed: "",
        test_result: "",
        executed_times: "",
        design_date: "",
        related_req: "",
        related_bug: ""
      },
      modules: [],
      members: [],
      baseInfoRules: {
        'prod_module': [{ required: true, message: '请选择产品模块', trigger: 'change' }]
      },
      loadMde: true,
      caseTitle: "",
      designerDisabled: true,
      configs: {
        status: false, // 禁用底部状态栏
        autofocus: false, //激活容易影响键盘快捷键事件监听
        spellChecker: false, // 禁用拼写检查,
        showIcons: [
          "code",
          "table",
          "strikethrough"
        ],
        hideIcons: ["fullscreen", "side-by-side"],
        placeholder: "此处可通过粘贴板或者拖拽进行快捷图片上传，仅限图片",
        renderingConfig: {
          codeSyntaxHighlighting: true,
          highlightingTheme: "atom-one-light"
        }
      }
    };
  },

  components: {
    simpleMde
  },

  watch: {
    caseId: function() {
      this.caseTitle = this.caseId + " - " + this.title;
      this.edit_forbidden = parseInt(this.currentProd) != parseInt(sessionStorage.product_id);
    }
  },

  created() {
    let _self = this;
    _self.test_results.splice(0, _self.test_results.length);
    let testResultStatus = localStorage.getItem("testResultStatus");
    eval(testResultStatus).forEach(item => {
      _self.test_results.push({
        value: item.status_id,
        label: item.status_name
      });
    });

    _self.test_types.splice(0, _self.test_types.length);
    let testType = localStorage.getItem("testType");
    eval(testType).forEach(item => {
      _self.test_types.push({
        value: item.type_id,
        label: item.type_name
      });
    });

    _self.priorities.splice(0, _self.priorities.length);
    let testPriority = localStorage.getItem("testPriority");
    eval(testPriority).forEach(item => {
      _self.priorities.push({
        value: item.priority_id,
        label: item.priority_name
      });
    });

    _self.test_periods.splice(0, _self.test_periods.length);
    let testPeriod = localStorage.getItem("testPeriod");
    eval(testPeriod).forEach(item => {
      _self.test_periods.push({
        value: item.period_id,
        label: item.period_name
      });
    });

    _self.case_statuss.splice(0, _self.case_statuss.length);
    let testStatus = localStorage.getItem("testStatus");
    eval(testStatus).forEach(item => {
      _self.case_statuss.push({
        value: item.status_id,
        label: item.status_name
      });
    });

    _self.caseTitle = _self.caseId + " - " + _self.title;
    _self.edit_forbidden = parseInt(_self.currentProd) != parseInt(sessionStorage.product_id);
    _self.queryCaseInfo(_self.caseId);
    _self.moduleQuery();
    if (_self.readOnly) {
      if (!_self.configs.toolbar) {
        _self.$set(_self.configs,"toolbar", false);
      } else {
        _self.configs.toolbar = false;
      }
    } else {
      if (_self.configs.toolbar) {
        delete _self.configs[toolbar];
      }
    }
  },

  methods: {
    checkSaveCaseInfo(formName) {
      let _self = this;
      _self.$refs[formName].validate(valid => {
        if (!valid) {
          _self.$notify.error("表单校验不通过，无法提交");
          return;
        } else {
          if (!_self.form.prod_module || _self.form.prod_module == 0 || _self.form.prod_module == '未定义') {
            _self.$message.warning("请选择正确的所属模块！");
            return;
          }
          if (
            _self.form.auto_path &&
            !test_patern.test(_self.form.auto_path.toLowerCase()) &&
            !suite_patern.test(_self.form.auto_path.toLowerCase())
          ) {
            _self.$message.warning("测试脚本路径格式校验不通过！");
            return;
          }
          _self.saveCaseInfo();
        }
      });
    },

    saveCaseInfo() {
      let _self = this;
      let dataParam = new URLSearchParams();
      dataParam.append("summary", null != _self.form.summary && _self.form.summary != "" ? _self.form.summary.replace(/^\s+|\s+$/g,'') + "\n" : "");
      dataParam.append("pre_condition", null != _self.form.pre_condition && _self.form.pre_condition != "" ? _self.form.pre_condition : "");
      _self
        .$axios({
          method: "post",
          url: "/testing/case_info_save",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: _self.caseId,
            status: _self.form.case_status,
            product_id: _self.form.product.selected,
            designer: _self.form.designer,
            priority: _self.form.priority,
            test_period: _self.form.test_period,
            test_type: _self.form.test_type,
            prod_module: _self.form.prod_module,
            regress_mark: _self.form.regress_mark.selected ? _self.form.regress_mark.selected : "N",
            auto_path: _self.form.auto_path
          },
          data: dataParam
        })
        .then(function(res) {
          if (res.data >= 1) {
            _self.queryCaseInfo(_self.caseId);
            _self.$message.success("保存成功");
          } else {
            _self.$notify.error("保存失败，没有记录更新");
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    queryCaseInfo(case_id) {
      let _self = this;
      _self.loadMde = false;
      this.$nextTick(_ => {
        if (_self.$refs.baseInfoForm) {
          _self.$refs.baseInfoForm.clearValidate();
        }
      });

      _self
        .$axios({
          method: "post",
          url: "/testing/case_info_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: case_id
          }
        })
        .then(function(res) {
          let info = eval(res.data);
          if (res.data && eval(res.data).length > 0) {
            _self.caseReadQuery(case_id);
            _self.form.designer = info[0].designer;
            _self.form.designer_name = info[0].designer_name;
            _self.form.case_status = info[0].status;
            _self.form.status_name = info[0].status_name;
            _self.form.priority = info[0].priority;
            _self.form.priority_name = info[0].priority_name;
            _self.form.test_type = info[0].test_type;
            _self.form.test_type_name = info[0].test_type_name;
            if (info[0].prod_module == 0) {
              _self.form.prod_module = "";
            } else {
              _self.form.prod_module = info[0].prod_module;
            }
            _self.form.prod_module_name = info[0].prod_module_name;
            _self.form.test_period = info[0].test_period;
            _self.form.test_period_name = info[0].test_period_name;
            _self.form.regress_mark.selected = info[0].regress_mark ? info[0].regress_mark : "N";
            _self.form.regress_mark_name = _self.form.regress_mark.opts.find(item => {return item.value == _self.form.regress_mark.selected}).label;
            _self.form.auto_path = info[0].auto_path;
            _self.form.summary = info[0].summary;
            _self.form.pre_condition = info[0].pre_condition;
            _self.form.last_executed = info[0].last_executed;
            _self.form.executed_times = info[0].executed_times;
            _self.form.design_date = info[0].design_date;
            _self.form.related_req = info[0].related_req;
            _self.form.related_bug = info[0].related_bug;

            let textHeight = (_self.readOnly == true) ? bodyAviHeightTab - 420 : bodyAviHeightTab - 380;
            _self.loadMde = true;
            _self.mdeIMGHandler();

            _self.$nextTick(function(){
              _self.$el.querySelector(".CodeMirror").style.height = textHeight + "px";
            });
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    caseReadQuery(case_id) {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/case_read_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: case_id
          }
        })
        .then(function(res) {
          let result = res.data;
          _self.form.design_date = result.design_date;
          _self.form.executed_times = result.executed_times;
          _self.form.last_executed = !result.last_executed ? "尚未执行" : result.last_executed;
          _self.form.test_result = !result.test_result ? 5 : result.test_result;
          _self.form.related_req = !result.related_req ? 0 : result.related_req;
          _self.form.related_bug = !result.related_bug ? 0 : result.related_bug;
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
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

    mdeIMGHandler() {
      this.$nextTick(() => {
        [this.$refs.caseDetail].map(({ simplemde }) => {
          if (this.readOnly || this.edit_forbidden) {
            simplemde.togglePreview();
            return;
          }
          simplemde.codemirror.on("drop", (editor, e) => {
            if (!(e.dataTransfer && e.dataTransfer.files)) {
              this.$notify.error("该浏览器不支持操作");
              return;
            }
            let dataList = e.dataTransfer.files;
            let imageFiles = [];
            for (let i = 0; i < dataList.length; i++) {
              if (dataList[i].type.indexOf("image") === -1) {
                this.$message.info("仅支持图片拖拽");
                continue;
              }
              imageFiles.push(dataList[i]);
            }
            this.mdeIMGUpload(simplemde.codemirror, imageFiles);
            e.preventDefault();
          });

          simplemde.codemirror.on("paste", (editor, e) => {
            if (!(e.clipboardData && e.clipboardData.items)) {
              this.$notify.error("该浏览器不支持操作");
              return;
            }
            try {
              let dataList = e.clipboardData.items;
              if (
                dataList[0].kind === "file" &&
                dataList[0].getAsFile().type.indexOf("image") !== -1
              ) {
                this.mdeIMGUpload(simplemde.codemirror, [
                  dataList[0].getAsFile()
                ]);
              }
            } catch (e) {
              this.$notify.error("只能通过粘贴板上传，而不是文件");
            }
          });
        });
      });
    },

    mdeIMGUpload(simplemde, files) {
      let params = files.map(file => {
        let param = new FormData();
        param.append("file", file, '【' + sessionStorage.user_name + '】测试用例附件图片.png');
        return param;
      });
      let makeRequest = params => {
        return this.$axios.post("/vuemde/img_upload", params, {
          headers: {
            "Content-Type": "multipart/form-data"
          }
        });
      };
      let requests = params.map(makeRequest);
      this.$axios.spread = callback => {
        return arr => {
          return callback.apply(null, arr);
        };
      };
      this.$axios.all(requests).then(
        this.$axios.spread((...resps) => {
          for (let i = 0; i < resps.length; i++) {
            let result = resps[i].data;
            let url = `![](${ result.url})`; // 拼接成markdown语法
            let content = simplemde.getValue();
            simplemde.setValue(content + url + "\n"); // 和编辑框之前的内容进行拼接
          }
          this.$message.success("文件上传成功!");
        })
      );
    }
  },

  props: ["caseId", "currentProd", "readOnly", "title"]
};
</script>

<style>
@import "~simplemde/dist/simplemde.min.css";
.case-info .el-form {
  width: 100%;
}

.case-info .el-form-item {
  width: 32.95%;
}

.case-info .el-form-item {
  margin-bottom: 12px;
}

.case-info .el-form-item .el-select,
.case-info .el-form-item .el-input {
  width: 100%;
}

.case-info .el-form-item .el-form-item__label {
  width: 40%;
}

.case-info .el-form-item .el-form-item__content {
  width: 59%;
}

.case-info .autocase {
  width: 66% !important;
}

.case-info .autocase .el-form-item__label {
  width: 20%;
}

.case-info .autocase .el-form-item__content {
  width: 80% !important;
}

.case-info .autocase input {
  width: 100% !important;
}

.case-info .el-input__inner {
  padding: 0 10px;
}

.case-desc {
  width: 99% !important;
  border-radius: 2px;
}

.case-desc textarea,
.case-desc .markdown-editor {
  width: 100%;
}

.case-desc .el-form-item__label {
  width: 13.05% !important;
}

.case-desc .el-form-item__content {
  width: 86.95% !important;
}

.case-desc .CodeMirror,
.case-desc .CodeMirror-scroll {
  min-height: 65%;
}

.button-item, 
.button-item .el-form-item__content,
.button-item .el-button {
  float: right;
}

.autocase .read-content-tips input{
  color:#3AB4D7 !important;
  cursor: pointer !important;
}

.autocase .read-content-tips input:hover{
  color: #61c3df !important;
}
</style>

