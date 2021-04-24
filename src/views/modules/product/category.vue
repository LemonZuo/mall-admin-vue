<template>
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
            <el-button type="text" size="mini" v-show="data.catLevel != 3" @click="() => append(data)">

            </el-button>
            <el-button type="text" size="mini" v-show="data.catLevel == 3" @click="() => remove(node, data)">
            删除
            </el-button>
        </span>
      </span>
    </el-tree>
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
            defaultExpandedList: []
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
            // const newChild = { id: id++, label: 'testtest', children: [] };
            // if (!data.children) {
            //     this.$set(data, 'children', []);
            // }
            // data.children.push(newChild);
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
