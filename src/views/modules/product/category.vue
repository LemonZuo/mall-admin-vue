<template>
    <div>
        <el-tree show-checkbox
                 :data="categoryMenuList"
                 :props="defaultProps"
                 :expand-on-click-node="false"
                 :default-expanded-keys="defaultExpandedList"
                 node-key="catId"
                 @node-click="">
              <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
            <!-- 针对一二级分类 -->
            <el-button type="text" size="mini" v-show="data.catLevel >0 && data.catLevel <=2" @click="() => append(data)">
            新 增
            </el-button>
            <!-- 所有分类 -->
            <el-button type="text" size="mini" @click="() => edit(data)">
            修改
            </el-button>
            <!-- 最后一级分类 -->
            <el-button type="text" size="mini" v-show="data.children == null|| data.children.length === 0" @click="() => remove(node, data)">
            删 除
            </el-button>
        </span>
      </span>
        </el-tree>
        <!-- 新增子集分类 -->
        <el-dialog title="新增商品分类" :visible.sync="dialogFormVisible">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="submitData">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script type="text/javascript">
export default {
    name: "category",
    data() {
        return {
            defaultProps: {
                children: 'children',
                label: 'name',
            },
            categoryMenuList: [],
            defaultExpandedList: [],
            dialogFormVisible: false,
            dialogType: '',
            category: {
                name: '',
                parentCid: '',
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                catId: ''
            }
        }
    },
    mounted() {
        this.getCategoryMenuList();
    },
    methods: {
        // 加载商品分类树
        getCategoryMenuList() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({data}) => {
                this.categoryMenuList = data.data;
            })
        },
        append(data) {
            this.dialogFormVisible = true;
            this.dialogType = 'add';
            this.category.parentCid = data.catId;
            this.category.catLevel = parseInt(data.catLevel) + 1;
            this.defaultExpandedList = [data.catId];
        },
        edit(data) {
            this.dialogFormVisible = true;
            this.dialogType = 'edit';
            this.category.catId = data.catId;
            this.category.parentCid = data.parentCid;
            this.category.catLevel = data.catLevel;
            this.category.name = data.name;
            this.category.showStatus = data.showStatus;
            this.category.sort = data.sort;
            this.defaultExpandedList = [data.catId];
        },
        submitData() {
            if (this.dialogType === 'add') {
                this.addCategory();
            } else if (this.dialogType === 'edit') {
                this.updateCategory();
            } else {

            }
        },
        // 保存分类数据
        addCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({data}) => {
                this.dialogFormVisible = false;
                this.category = {
                    name: '',
                    parentCid: '',
                    catLevel: 0,
                    showStatus: 1,
                    sort: 0,
                    catId: null
                };
                if (data && data.code === 0) {
                    this.$message({
                        message: '操作成功',
                        type: 'success',
                        duration: 1500,
                        onClose: () => {
                            // 重新获取分类数据
                            this.getCategoryMenuList();
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
            })
        },
        // 修改数据
        updateCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData(this.category, false)
            }).then(({data}) => {
                this.dialogFormVisible = false;
                this.category = {
                    name: '',
                    parentCid: '',
                    catLevel: 0,
                    showStatus: 1,
                    sort: 0,
                    catId: ''
                };
                if (data && data.code === 0) {
                    this.$message({
                        message: '操作成功',
                        type: 'success',
                        duration: 1500,
                        onClose: () => {
                            // 重新获取分类数据
                            this.getCategoryMenuList();
                        }
                    })
                } else {
                    this.$message.error(data.msg)
                }
            })
        },
        remove(node, data) {
            this.$confirm(`是否删除【${data.name}】分类?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                let ids = [data.catId];
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.$message({
                            message: '操作成功',
                            type: 'success',
                            duration: 1500,
                            onClose: () => {
                                // 重新获取分类数据
                                this.getCategoryMenuList();
                                // 展开父级分类数据
                                this.defaultExpandedList = [node.parent.data.catId];
                            }
                        })
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
        },
    }
}
</script>

<style>

</style>
