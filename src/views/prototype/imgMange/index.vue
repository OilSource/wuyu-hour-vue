<template>
  <div class="main">
    <!-- <h1>docManage</h1> -->
    <el-dialog title="效果图管理" :visible.sync="dialogFormVisible">
      <el-form ref="form" class="cusForm">
        <el-form-item label="效果图管理:" v-hasPermi="['pr:sketch:upload']">
          <br /><el-upload
            class="upload-demo"
            ref="uploadDoc"
            :limit="10"
            :data="dataDoc"
            :headers="Headers"
            :action="fileUrl"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :before-upload="handleBeforeDoc"
            :on-success="handleSuccess"
            :file-list="fileListDOC"
            :auto-upload="false"
          >
            <el-button
              style="padding:6px"
              slot="trigger"
              size="small"
              type="primary"
              >选取文件</el-button
            >
            <el-button
              v-show="false"
              style="margin-left: 10px;height:26px"
            ></el-button>
            <div slot="tip" class="el-upload__tip">
              文件格式仅限包含JPG/IMG/SVG/等文件的ZIP压缩包，并且只能上传一个。
            </div>
          </el-upload>
        </el-form-item>
        <el-form-item>
          <el-button
            style="width:100%;padding:6px;float:right;"
            size="small"
            type="success"
            @click="onSubmit"
            v-hasPermi="['pr:sketch:upload']"
            >立即上传</el-button
          >
          <!-- <el-button>取消</el-button> -->
        </el-form-item>
        <el-form-item label="上传记录：">
          <template>
            <el-table :data="tableData" style="width: 100%">
              <el-table-column prop="fileName" label="效果图">
              </el-table-column>
              <el-table-column prop="createTime" label="更新时间" width="150">
              </el-table-column>

              <el-table-column prop="createUserName" label="操作" width="100">
              </el-table-column>
              <el-table-column label="效果图访问">
                <template slot-scope="scope">
                  <a v-if="!scope.row.id">
                    <el-button type="text" size="small" :disabled="true"
                      >暂不可见</el-button
                    >
                  </a>
                  <a v-else @click="goShowImg(scope.row)">
                    <el-button
                      type="text"
                      size="small"
                      style="text-decoration:underline"
                      >立即访问</el-button
                    >
                  </a>
                </template></el-table-column
              >
              <el-table-column label="管理" width="160">
                <template slot-scope="scope">
                  <!-- <el-button
                    size="mini"
                    @click="download(scope.$index, scope.row)"
                    >下载</el-button
                  > -->
                  <el-button
                    size="mini"
                    type="danger"
                    @click="handleDelete(scope.$index, scope.row)"
                    v-hasPermi="['pr:doc:delete']"
                    >删除</el-button
                  >
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
import { listSketchdelete, listProtoSketch } from "@/api/prototype/prototype";
import { getToken } from "@/utils/auth";

export default {
  data() {
    return {
      dialogFormVisible: false,
      tempselectid: null,
      id: null,
      arrIndex: null,
      protoId: null,
      params: { pageNum: 1, pageSize: 10 },
      tableData: [],
      fileListDOC: [],
      Headers: {
        Authorization: "Bearer " + getToken()
      },
      dataDoc: {
        prototypeId: null
      },
      fileUrl: process.env.VUE_APP_BASE_API + "/pr/sketch/upload"
    };
  },
  created() {
    this.id = this.$route.query.id;
    // this.arrIndex = this.$route.query.arrIndex;
    // this.protoId = this.$route.query.protoId;
    this.params.projectId = this.id;

    // this.init();
  },
  methods: {
    open() {
      // this.dataDoc.prototypeId = this.protoId;
      this.fileListDOC = [];
      this.dialogFormVisible = true;
      // this.protoId=this.dataDoc.projectId
      listProtoSketch(this.protoId).then(res => {
        // console.log("queryDoc", res);
        if (res.code == 200) {
          this.tableData = res.data;
          // console.log("this.tableData", this.tableData);
        }
      });
    },
    close() {
      this.dialogFormVisible = false;
      this.fileListDOC = [];
    },
    onSubmit() {
      // console.log("submit!");
      this.$refs.uploadDoc.submit();
    },
    goShowImg(row) {
      console.log("goShowImg", row);

      let newpage = this.$router.resolve({
        name: "ShowImg",
        query: {
          id: row.id
        }
      });
      window.open(newpage.href, "_blank");
    },
    handleBeforeDoc(file) {
      // console.log("handleBefore", file);
      var FileExt = file.name.replace(/.+\./, "");
      if (["rar", "zip", "gz", "bz2"].indexOf(FileExt.toLowerCase()) === -1) {
        this.$message({
          type: "warning",
          message:
            "文件格式仅限包含JPG/IMG/SVG/等文件的ZIP压缩包，并且只能上传一个。"
        });
        return false;
      }
    },
    handleSuccess(res, file, fileList) {
      // console.log("handleSuccess", res, file, fileList);
      if (res.code == 200) {
        this.$message.success("上传成功");
        this.close();
        // this.init();
        // this.$parent.init();
        this.$parent.value4 = parseInt(this.tempselectid);
        this.$parent.getList(this.tempselectid);
      } else {
        this.$message.error("上传失败！");
      }
    },
    handleRemove(file, fileList) {
      // console.log("handleRemove", file, fileList);
    },
    handlePreview(file) {
      // console.log("file", file);
    },
    download(index, row) {
      // console.log(index, row);
      window.open(process.env.VUE_APP_BASE_API + row.docUrl);
    },
    handleDelete(index, row) {
      // console.log(index, row);
      this.$confirm("确定要删除该条内容？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(_ => {
          listSketchdelete(row.id).then(res => {
            if (res.code == 200) {
              this.$message.success("删除成功！");
              this.tableData.splice(index, 1);
            }
          });
        })
        .catch(_ => {
          this.$message.warning("取消删除！");
        });
    }
  }
};
</script>

<style lang="scss" scoped>
.cusForm {
  width: 90%;
  margin: 0 auto;
  ::v-deep .el-upload--text {
    float: right;
  }
  ::v-deep .el-upload-list--text {
    border: 1px solid #d0d3db;
    height: 100px;
    overflow: auto;
  }
}
.doc {
  /* height: 86px; */
  text-align: center;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
::v-deep .el-upload__tip {
  height: 36px;
}
</style>
