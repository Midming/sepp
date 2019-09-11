<template>
  <div>
    <div v-if="hasRelease" class="relate-query">
      <div class="crumbs">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item>关联缺陷</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-form :inline="true" size="mini" label-width="60px">
        <el-form-item label="版本">
          <el-select v-model="release.selected" style="width:150px" placeholder="请选择" @change="loadRelease()">
            <el-option v-for="opt in release.opts" :label="opt.label" :key="opt.value" :value="opt.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="缺陷">
          <el-select v-model="defect.selected" style="width:400px" multiple placeholder="请选择" collapse-tags @change="loadDefect()">
            <el-option v-for="opt in defect.opts" :label="opt.label" :key="opt.value" :value="opt.value" :disabled="opt.disabled"></el-option>
          </el-select>
        </el-form-item>
        <br>
        <el-form-item style="float:right;">
          <el-button v-no-more-click type="primary" icon="el-icon-refresh" @click="relatedBUGQuery()">刷新</el-button>
          <el-button v-no-more-click type="primary" icon="el-icon-check" @click="relate()" :disabled="btnDisabled || prodForbbiden()">关联</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div v-if="!hasRelease">
        <el-alert type="warning" title="没有在开发、测试计划中的版本，请联系版本经理" :closable="false"></el-alert>
    </div>
    <div class="related-table" v-bind:style="{height: tableHeight + 'px'}">
      <div class="crumbs">
          <el-breadcrumb separator-class="el-icon-arrow-right">
              <el-breadcrumb-item>已关联缺陷</el-breadcrumb-item>
          </el-breadcrumb>
      </div>
      <el-table :data="filteredBugs" style="width: 100%;" :max-height="tableHeight - 30" stripe size="small" :border="show_border">
        <el-table-column prop="rel_code" label="版本号" width="120" align="center">
        </el-table-column>
        <el-table-column prop="id" label="缺陷编号" width="100" align="center" sortable>
        </el-table-column>
        <el-table-column prop="status_name" label="缺陷状态" width="120" align="center" sortable>
        </el-table-column>
        <el-table-column prop="summary" label="缺陷摘要" header-align="center">
        </el-table-column>
        <el-table-column  width="80" align="center" label="操作">
            <template slot-scope="scope">
                <el-button v-no-more-click type="text" size="small" @click="unrelate(scope.row.id)" :disabled="prodForbbiden()">解除关联</el-button>
            </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
let origin_bugs = [];
export default {
  data: function() {
    return {
      show_border: localStorage.show_border && localStorage.show_border == 'true',
      filteredBugs: [],
      hasRelease: true,
      relates: [], //sort过后的版本和defect信息查询结果
      related_bugs: [], //已关联的defect
      btnDisabled: true,
      release: {
        selected: "",
        opts: []
      },
      defect: {
        selected: [],
        opts: []
      },
      tableHeight: ""
    };
  },

  watch: {
    related_bugs: function(val) {
      this.filteredBugs = this.dataFilter(val);
    }
  },

  created() {
    this.tableHeight = bodyAviHeightNTab - 170;
    this.openBUGQuery();
  },

  methods: {
    dataFilter(val) {
      return origin_bugs.filter(function(d) {
        if (!val) {
          return null;
        }
        return val.indexOf(d.id) > -1;
      });
    },

    relate() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/case_relate_save",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: _self.caseId,
            ids: _self.defect.selected.toString(),
            relate_type: 1
          }
        })
        .then(function(res) {
          if (res.data > 0) {
            _self.$message.success("关联成功");
            for (let i = 0; i < _self.defect.selected.length; i++) {
              _self.related_bugs.push(_self.defect.selected[i]);
            }
            _self.defect.selected.splice(0, _self.defect.selected.length);
            _self.release.selected = "";
          } else {
            _self.$message.info("关联失败");
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    unrelate(id) {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/case_relate_delete",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: _self.caseId,
            id: id,
            relate_type: 1
          }
        })
        .then(function(res) {
          if (res.data === 1) {
            _self.$message.success("解除关联成功");
          } else {
            _self.$message.info("解除关联失败");
          }
          _self.related_bugs.splice(_self.related_bugs.indexOf(id), 1);
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    prodForbbiden(){
      return parseInt(this.currentProd) != parseInt(sessionStorage.product_id);
    },

    loadDefect() {
      this.btnDisabled = this.defect.selected.length < 1;
    },

    loadRelease() {
      let _self = this;
      let relChoice = [];
      relChoice = _self.relates.filter(function(d) {
        return d.rel_id === _self.release.selected;
      });
      _self.defect.selected = [];
      _self.defect.opts.splice(0, _self.defect.opts.length);
      for (let i = 0; i < relChoice[0].children.length; i++) {
        _self.defect.opts.push({
          value: relChoice[0].children[i].id,
          label:
            relChoice[0].children[i].id +
            " - " +
            relChoice[0].children[i].summary,
          disabled: _self.related_bugs.indexOf(relChoice[0].children[i].id) > -1
        });
      }
    },

    relatedBUGQuery() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/case_relate_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            case_id: _self.caseId,
            relate_type: 1
          }
        })
        .then(function(res) {
          let json = eval(res.data);
          for (let i = 0; i < json.length; i++) {
            _self.related_bugs.push(json[i].relate_id);
          }
          _self.filteredBugs = _self.dataFilter(_self.related_bugs);
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    openBUGQuery() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/testing/open_defect_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            current_product: _self.currentProd
          }
        })
        .then(function(res) {
          let json = eval(res.data);
          origin_bugs.splice(0, origin_bugs.length);
          for (let z = 0; z < json.length; z++) {
            origin_bugs.push(json[z]);
          }
          _self.relates = _self.sortData(json, "rel_id", "rel_code", "children");
          _self.release.opts.splice(0, _self.release.opts.length);
          for (let i = 0; i < _self.relates.length; i++) {
            _self.release.opts.push({
              value: _self.relates[i].rel_id,
              label: _self.relates[i].rel_code
            });
          }
          if (_self.release.opts.length > 0) {
            _self.hasRelease = true;
          } else {
            _self.hasRelease = false;
          }
          _self.$nextTick(function() {
            _self.relatedBUGQuery();
          });
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    sortData(json, idKey, labelKey, childKey) {
      let temp = [];
      let result = [];
      for (let i = 0; i < json.length; i++) {
        temp.push(json[i][idKey]);
      }
      temp = temp.filter(function(element, index, array) {
        return array.indexOf(element) === index;
      });

      for (let k = 0; k < temp.length; k++) {
        let children = json.filter(function(d) {
          return d[idKey] === temp[k];
        });
        result.push({
          [idKey]: temp[k],
          [labelKey]: children[0][labelKey],
          [childKey]: children
        });
      }
      return result;
    }
  },

  props: ["caseId", "currentProd"]
};
</script>

<style>
.relate-query {
  margin-top: 10px;
  height: 110px;
}

.related-table {
  margin-top: 10px;
}
</style>