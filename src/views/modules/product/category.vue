<template>
    <div>
        <el-tree show-checkbox
                 :data="categoryMenuList"
                 :props="defaultProps"
                 :expand-on-click-node="false"
                 :default-expanded-keys="defaultExpandedList"
                 :draggable="true"
                 :allow-drop="allowDrop"
                 node-key="catId"
                 @node-drop="handleDrop">
              <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
            <!-- 针对一二级分类 -->
            <el-button type="text" size="mini" v-show="data.catLevel >0 && data.catLevel <=2"
                       @click="() => append(data)">
            新 增
            </el-button>
            <!-- 所有分类 -->
            <el-button type="text" size="mini" @click="() => edit(data)">
            修改
            </el-button>
            <!-- 最后一级分类 -->
            <el-button type="text" size="mini" v-show="data.children == null|| data.children.length === 0"
                       @click="() => remove(node, data)">
            删 除
            </el-button>
        </span>
      </span>
        </el-tree>
        <!-- 新增子集分类 -->
        <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="分类图标">
                    <el-input v-model="category.icon" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="分类计量单位">
                    <el-input v-model="category.productUnit" autocomplete="off"></el-input>
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
                catId: '',
                name: '',
                parentCid: '',
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                icon: '',
                productUnit: ''
            },
            dialogTitle: '',
            waitUpdateNodes: []
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
            this.dialogTitle = '新增商品分类';

            this.category.parentCid = data.catId;
            this.category.catLevel = parseInt(data.catLevel) + 1;

            this.defaultExpandedList = [data.catId];
        },
        edit: function (data) {
            this.dialogFormVisible = true;
            this.dialogType = 'edit';
            this.dialogTitle = '修改商品分类';
            // 获取最新分类数据
            this.$http({
                method: 'GET',
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`)
            }).then(({data}) => {
                if (data && data.code === 0) {
                    this.category.catId = data.data.catId;
                    this.category.name = data.data.name;
                    this.category.catLevel = data.data.catLevel;
                    this.category.parentCid = data.data.parentCid;
                    this.category.icon = data.data.icon;
                    this.category.productUnit = data.data.productUnit;
                    this.category.showStatus = data.data.showStatus;
                    this.category.sort = data.data.sort;

                    this.defaultExpandedList = [data.data.catId];
                } else {

                }
            }).catch(() => {

            })
        },
        submitData() {
            if (this.dialogType === 'add') {
                this.addCategory();
            } else if (this.dialogType === 'edit') {
                this.updateCategory();
            } else {

            }
        },
        // 判断是否允许拖拽
        allowDrop(draggingNode, dropNode, type) {
            if (type === 'inner') {
                console.log("inner", dropNode.level);
                return dropNode.level < 3;
            } else {
                console.log("else", dropNode.level);
                return dropNode.level <= 3;
            }
        },
        // 拖拽成功触发事件
        handleDrop(draggingNode, dropNode, dropType, event) {
            // 父节点编号
            let parentCatId = 0;
            // 当前节点同级元素
            let siblingNodes = null;
            if (dropType === 'inner') {
                parentCatId = dropNode.data.catId;
                siblingNodes = dropNode.childNodes;
            } else {
                parentCatId = dropNode.parent.data.catId === undefined ? parentCatId : dropNode.parent.data.catId;
                siblingNodes = dropNode.parent.childNodes;
            }
            this.defaultExpandedList = [parentCatId];
            // 对同级元素排序
            for (let i = 0; i < siblingNodes.length; i++) {
                if (siblingNodes[i].data.catId === dropNode.data.catId) {
                    // 当前拖动节点

                    let catLevel = dropNode.level;
                    if (siblingNodes[i].level !== dropNode.level) {
                        // 修改当前节点层级
                        catLevel = siblingNodes[i].level;
                        // 修改子子节点层级
                        this.updateChildNodeLevel(siblingNodes[i])
                        this.waitUpdateNodes.push({
                            catId: siblingNodes[i].data.catId,
                            sort: i,
                            parentCid: parentCatId,
                            catLevel: catLevel
                        })
                    } else {
                        this.waitUpdateNodes.push({
                            catId: siblingNodes[i].data.catId,
                            sort: i,
                            parentCid: parentCatId
                        })
                    }
                } else {
                    // 同级节点
                    this.waitUpdateNodes.push({catId: siblingNodes[i].data.catId, sort: i})
                }
            }
            this.$http({
                url: this.$http.adornUrl('/product/category/updateSort'),
                method: 'post',
                data: this.$http.adornData(this.waitUpdateNodes, false)
            }).then(({data}) => {
                // this.category = {
                //     catId: '',
                //     name: '',
                //     parentCid: '',
                //     catLevel: 0,
                //     showStatus: 1,
                //     sort: 0,
                //     icon: '',
                //     productUnit: ''
                // };
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
        updateChildNodeLevel(node) {
            let childrenNodeList = node.childNodes;
            if (childrenNodeList.length > 0) {
                for (let i = 0; i < childrenNodeList.length; i++) {
                    this.waitUpdateNodes.push({
                        catId: childrenNodeList[i].data.catId,
                        catLevel: childrenNodeList[i].level
                    });
                    this.updateChildNodeLevel(childrenNodeList[i]);
                }
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
                    catId: '',
                    name: '',
                    parentCid: '',
                    catLevel: 0,
                    showStatus: 1,
                    sort: 0,
                    icon: '',
                    productUnit: ''
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
            let {catId, name, icon, productUnit} = this.category;
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData({catId, name, icon, productUnit}, false)
            }).then(({data}) => {
                this.dialogFormVisible = false;
                this.category = {
                    catId: '',
                    name: '',
                    parentCid: '',
                    catLevel: 0,
                    showStatus: 1,
                    sort: 0,
                    icon: '',
                    productUnit: ''
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
