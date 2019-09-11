<template>
  <div>
    <div class="relate-query">
      <div class="crumbs">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item>关联开发需求</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-form :inline="true" size="mini" label-width="60px">
        <el-form-item label="版本">
          <el-select v-model="release.selected" style="width:150px" placeholder="请选择" @change="reqQuery()">
            <el-option v-for="opt in release.opts" :label="opt.label" :key="opt.value" :value="opt.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="需求">
          <el-select v-model="req.selected" style="width:400px" placeholder="请选择" multiple collapse-tags>
            <el-option v-for="opt in req.opts" :label="opt.label" :key="opt.value" :value="opt.value" :disabled="opt.disabled"></el-option>
          </el-select>
        </el-form-item>
        <br>
        <el-form-item style="float:right;">
          <el-button v-no-more-click type="primary" icon="el-icon-refresh" @click="reqBatchQuery()">刷新</el-button>
          <el-button v-no-more-click type="primary" icon="el-icon-check" @click="relate()" :disabled="prodForbbiden() || req.selected.length == 0">关联</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="related-table" v-bind:style="{height: tableHeight + 'px'}">
      <div class="crumbs">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item>已关联开发需求</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-table :data="req_info" style="width: 100%;" :max-height="tableHeight - 30" stripe size="mini" :border="show_border">
        <el-table-column prop="req_id" label="产品需求号" width="90" align="center" sortable>
        </el-table-column>
        <el-table-column prop="status_name" label="需求状态" width="90" align="center" sortable>
        </el-table-column>
        <el-table-column prop="summary" label="需求摘要" header-align="center">
        </el-table-column>
        <el-table-column  width="90" align="center" label="操作">
          <template slot-scope="scope">
            <el-button v-no-more-click type="text" size="mini" @click="unrelate(scope.row.req_id)" :disabled="prodForbbiden()">解除关联</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      show_border: localStorage.show_border && localStorage.show_border == 'true',
      related_req: [], //已关联的REQ
      req_info: [],
      release: {
        selected: "",
        opts: []
      },
      req: {
        selected: [],
        opts: []
      },
      tableHeight: ""
    };
  },

  created() {
    this.tableHeight = bodyAviHeightNTab - 170;
    this.releaseQuery();
    this.relatedSRQuery(this.reqBatchQuery);
  },

  methods: {
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
            ids: _self.req.selected.toString(),
            relate_type: 2
          }
        })
        .then(function(res) {
          let selected = _self.req.selected;
          if (res.data >= selected.length) {
            _self.$message.success("关联成功！");
            for (let i = 0; i < selected.length; i++) {
              _self.related_req.push(selected[i]);
            }
            _self.req.selected.splice(0, _self.req.selected.length);
            _self.release.selected = "";
            _self.reqBatchQuery();
          } else {
            _self.$message.info("关联失败！");
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    unrelate(req_id) {
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
            id: req_id,
            relate_type: 2
          }
        })
        .then(function(res) {
          if (res.data > 0) {
            _self.$message.success("解除关联成功");
          } else {
            _self.$message.info("解除关联失败");
          }
          _self.related_req.splice(_self.related_req.indexOf(req_id), 1);
          _self.reqBatchQuery();
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    prodForbbiden(){
      return parseInt(this.currentProd) != parseInt(sessionStorage.product_id);
    },

    relatedSRQuery(callback) {
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
            relate_type: 2
          }
        })
        .then(function(res) {
          let json = eval(res.data);
          _self.related_req.splice(0, _self.related_req.length);
          json.forEach(item => {
            _self.related_req.push(item.relate_id);
          });
          if (typeof callback == "function") {
            callback();
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },
    
    reqBatchQuery() {
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/requirements/req_batch_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            req_ids: _self.related_req.toString(),
          }
        })
        .then(function(res) {
          _self.req_info =  eval(res.data);
        })
        .catch(function(response) {
          console.log(response);
        });
    },

    releaseQuery(){
      let _self = this;
      _self
        .$axios({
          method: "post",
          url: "/requirements/open_rel_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          }
        })
        .then(function(res) {
          var releases = eval(res.data);
          _self.release.opts.splice(0, _self.release.opts.length);
          for (var i = releases.length - 1; i >= 0; i--) {
            _self.release.opts.push({
              label: "[" + releases[i].branchName + "] " + releases[i].rel_code,
              value: releases[i].rel_id
            });
          }
        })
        .catch(function(response) {
          _self.$notify.error("发生错误");
          console.log(response);
        });
    },

    reqQuery() {
      let _self = this;
      _self.req.selected = [];
      _self
        .$axios({
          method: "post",
          url: "/requirements/rel_req_query",
          headers: {
            "Content-type": "application/x-www-form-urlencoded"
          },
          params: {
            rel_id: _self.release.selected
          }
        })
        .then(function(res) {
          let json = eval(res.data.list);
          if (json.length == 0) {
            _self.$message.warning("该版本尚未纳入开发任务！");
            return;
          }
          _self.req.opts.splice(0, _self.req.opts.length);
          for (let i = 0; i < json.length; i++) {
            _self.req.opts.push({
              value: json[i].req_id,
              label: json[i].req_id + " - " + json[i].summary,
              disabled: _self.related_req.indexOf(json[i].req_id) > -1
            });
          }
        })
        .catch(function(response) {
          _self.$notify.error("查询版本下需求时发生程序错误！");
          console.log(response);
        });
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