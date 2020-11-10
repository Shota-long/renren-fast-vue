<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="120px"
    >
      <el-form-item label="品牌名" prop="name">
        <el-input v-model="dataForm.name" placeholder="品牌名"></el-input>
      </el-form-item>
      <el-form-item label="品牌logo地址" prop="logo">
        <!-- <el-input v-model="dataForm.logo" placeholder="品牌logo地址"></el-input> -->
        <SingleUpload v-model="dataForm.logo"></SingleUpload>
      </el-form-item>
      <el-form-item label="介绍" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="介绍"></el-input>
      </el-form-item>
      <el-form-item label="显示状态" prop="showStatus">
        <el-switch
          v-model="dataForm.showStatus"
          active-color="#13ce66"
          inactive-color="#ff4949"
          :active-value="1"
          :inactive-value="0"
          @change="
            () => {
              changeStatus(scope.row);
            }
          "
        >
        </el-switch>
      </el-form-item>
      <el-form-item label="检索首字母" prop="firstLetter">
        <el-input
          v-model="dataForm.firstLetter"
          placeholder="检索首字母"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model.number="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template> 

<script>
import SingleUpload from "@/components/upload/singleUpload.vue";
export default {
  components: { SingleUpload: SingleUpload }, //组件名等于导入的组件
  data() {
    var validateFirstLetter = (rule, value, callback) => {
      if (value == "") {
        callback(new Error("首字母不能为空"));
      } else if (!/^[a-zA-Z]$/.test(value)) {
        callback(new Error("首字母必须在a-z或A-Z之间"));
      } else {
        callback();
      }
    };
    var validateSort = (rule, value, callback) => {
     
      if (!value) {
        callback(new Error("排序数字不能为空"));
      }
      setTimeout(() => {
        if (!Number.isInteger(value) || value < 0) {
          callback(new Error("数字必须是一个非负整数"));
        } else {
          callback();
        }
      }, 300);
    };
    return {
      visible: false,
      dataForm: {
        brandId: 0,
        name: "",
        logo: "",
        descript: "",
        showStatus: "",
        firstLetter: "",
        sort: "",
      },
      dataRule: {
        name: [{ required: true, message: "品牌名不能为空", trigger: "blur" }],
        logo: [
          { required: true, message: "品牌logo地址不能为空", trigger: "blur" },
        ],
        descript: [
          { required: true, message: "介绍不能为空", trigger: "blur" },
        ],
        showStatus: [
          {
            required: true,
            message: "显示状态[0-不显示；1-显示]不能为空",
            trigger: "blur",
          },
        ],
        firstLetter: [{ validator: validateFirstLetter, trigger: "blur" }],
        sort: [{ validator: validateSort, trigger: "blur" }],
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.brandId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.brandId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/brand/info/${this.dataForm.brandId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.name = data.brand.name;
              this.dataForm.logo = data.brand.logo;
              this.dataForm.descript = data.brand.descript;
              this.dataForm.showStatus = data.brand.showStatus;
              this.dataForm.firstLetter = data.brand.firstLetter;
              this.dataForm.sort = data.brand.sort;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/brand/${!this.dataForm.brandId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              brandId: this.dataForm.brandId || undefined,
              name: this.dataForm.name,
              logo: this.dataForm.logo,
              descript: this.dataForm.descript,
              showStatus: this.dataForm.showStatus,
              firstLetter: this.dataForm.firstLetter,
              sort: this.dataForm.sort,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
    changeStatus(data) {
      // let {brandId,showStatus} = data;
      // //更改状态
      // this.$http({
      //     url: this.$http.adornUrl("/product/brand/update"), //从后台uri请求三级分类数据
      //     method: "post",
      //     data: this.$http.adornData(data, false),
      //   }).then(({ data }) => {
      //     this.$message({
      //       type: "success",
      //       message: "修改成功!",
      //     })
      //     this.getDataList();
      // });
      console.log(data);
    },
  },
};
</script>
