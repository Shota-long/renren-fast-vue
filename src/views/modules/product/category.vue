<!--  -->
<template>
  <div>
    <el-row>
      <el-button type="danger" icon="el-icon-delete" circle style="align:right"></el-button>
    </el-row>
    <el-tree
      :data="data"
      :props="defaultProps"
      show-checkbox
      @node-click="handleNodeClick"
      node-key="catId"
      :expand-on-click-node="false"
      :default-expanded-keys="this.expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
           <el-button
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            edit
          </el-button>
          <el-button
            v-if="data.child == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog id="append" title="添加" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="菜单名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="() => appendCateGory()"
          >确 定</el-button
        >
      </div>
    </el-dialog>
     <el-dialog id="edit" title="编辑" :visible.sync="dialogFormVisible_edit" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="菜单名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible_edit = false">取 消</el-button>
        <el-button type="primary" @click="() => editCateGory()"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      category: {catId: null ,name: '', parentCid: null, catLevel: null, showStatus: 1, sort: 0, icon: null, productUnit: null, productCount: 0},
      dialogFormVisible: false,
      dialogFormVisible_edit: false,
      expandedKey: [],
      data: [],
      defaultProps: {
        children: "child",
        label: "name",
      },
    };
  },
  methods: {
    handleNodeClick(data) {
    },
    //获取菜单信息
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"), //从后台uri请求三级分类数据
        method: "get",
      }).then(({ data }) => {
        this.data = data.data;
      });
    },
    //添加菜单
    append(data) {
      this.dialogFormVisible = true;
      this.category.catId = null;
      this.category.name = null;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel*1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.icon = null;
      this.category.productUnit = null;
      this.category.productCount = 0;
    },
    //删除菜单
    remove(node, data) {
      var id = [data.catId];
      console.log("catId" + id);
      this.$confirm(`此操作将删除${data.name}, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"), //从后台uri请求三级分类数据
            method: "post",
            data: this.$http.adornData(id, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!",
            }),
              this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    //提交新增菜单
    appendCateGory(){
      console.log("菜单",this.category);
      this.$http({
            url: this.$http.adornUrl("/product/category/save"), //从后台uri请求三级分类数据
            method: "post",
            data: this.$http.adornData(this.category, false),
          }).then(({ data }) => {
            this.dialogFormVisible = false,
            this.$message({
              type: "success",
              message: "添加成功!",
            }),
            this.getMenus();
            this.expandedKey = [this.category.parentCid];
          });
    },
    //编辑菜单
    edit(data){
        this.$http({
            url: this.$http.adornUrl(`/product/category/info/${data.catId}`), //从后台uri请求三级分类数据
            method: "get",
          }).then(({ data }) => {
            //this.category.catId = data.catId;
            //this.category.parentCid = data.parentCid;
            //this.category.name = data.category.name;
            //this.category.icon = data.category.icon;
            //this.category.productUnit = data.category.productUnit;
              this.category = data.category;
            //console.log("data",data)
            this.dialogFormVisible_edit = true;
            });
    },
    //提交菜单修改数据
    editCateGory(){
      this.$http({
            url: this.$http.adornUrl("/product/category/update"), //从后台uri请求三级分类数据
            method: "post",
            data: this.$http.adornData(this.category, false),
          }).then(({ data }) => {
            this.dialogFormVisible_edit = false,
            this.$message({
              type: "success",
              message: "修改成功!",
            }),
            this.getMenus();
            this.expandedKey = [this.category.parentCid];
          });
    }
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {
    this.getMenus();
  }, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>