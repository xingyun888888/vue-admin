<template>
    <section class="opertation-container">
        <div class="table-content">
            <el-form :inline="true" size="small" :model="searchForm" class="demo-form-inline">
                <el-form-item label="商户名称">
                    <el-date-picker
                            v-model="dataRange"
                            type="daterange"
                            range-separator="-"
                            start-placeholder="开始日期"
                            end-placeholder="结束日期">
                    </el-date-picker>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit">查询</el-button>
                </el-form-item>
            </el-form>
            <el-table style="background:#fff;" :data="devices" highlight-current-row v-loading="listLoading" @selection-change="selsChange" >
                <el-table-column type="selection" >
                </el-table-column>
                <el-table-column type="index" width="90px;" label="设备Id">
                </el-table-column>
                <el-table-column prop="name" label="商铺名称" :formatter="formatSex">
                </el-table-column>
                <el-table-column prop="addr" label="商铺地址" >
                </el-table-column>
                <el-table-column prop="orderCode" label="租赁订单号" >
                </el-table-column>
                <el-table-column prop="leaseName" label="租借人" width="90px;">
                </el-table-column>
                <el-table-column prop="orderMoney" width="120px;" label="产生订单金额" >
                </el-table-column>
                <el-table-column prop="leaseDate" label="租借时间" >
                </el-table-column>
                <el-table-column prop="returnDate" label="归还时间" >
                </el-table-column>
                <el-table-column prop="leasePlace" label="租借地点" >
                </el-table-column>
                <el-table-column prop="returnPlace" label="归还地点" >
                </el-table-column>
                <el-table-column prop="leaseId" width="120px;" label="租借充电宝ID" >
                </el-table-column>
                <el-table-column prop="singlePrice" label="客单价" >
                </el-table-column>
                <el-table-column prop="leaseStatus" label="租借状态" >
                    <template slot-scope="scope">
                        <el-button type="text"  >{{scope.leaseStatus}}</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                    class="pagination-class"
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="currentPage"
                    :page-sizes="[10, 20, 30, 40]"
                    :page-size="100"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="devices.length">
            </el-pagination>
        </div>
    </section>
</template>
<script>
    import util from '../../common/js/util'
    //import NProgress from 'nprogress'
    import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

    export default {
        data() {
            return {
                searchForm: {
                    user: '',
                    region: ''
                },
                dataRange:"",
                currentPage:1,
                filters: {
                    name: ''
                },
                devices: [
                    {
                        index:1,
                        name:"名字",
                        addr:"地址",
                        orderCode:"1",
                        leaseName:"2",
                        orderMoney:"2",
                        leaseDate:"2",
                        returnDate:"2",
                        leasePlace:"2",
                        returnPlace:"2",
                        leaseId:1,
                        singlePrice:1000,
                        leaseStatus:"未归还"
                    },
                ],
                total: 0,
                page: 1,
                listLoading: false,
                sels: [],//列表选中列

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                editForm: {
                    id: 0,
                    name: '',
                    sex: -1,
                    age: 0,
                    birth: '',
                    addr: ''
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ]
                },
                //新增界面数据
                addForm: {
                    name: '',
                    sex: -1,
                    age: 0,
                    birth: '',
                    addr: ''
                }

            }
        },
        methods: {
            onSubmit() {
                console.log('submit!');
            },
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
            },
            handleCurrentChange(val) {
                console.log(`当前页: ${val}`);
            },
            //性别显示转换
            formatSex: function (row, column) {
                return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getUsers();
            },
            //获取用户列表
            getUsers() {
                let para = {
                    page: this.page,
                    name: this.filters.name
                };
                this.listLoading = true;
                //NProgress.start();
                getUserListPage(para).then((res) => {
                    this.total = res.data.total;
                    this.users = res.data.users;
                    this.listLoading = false;
                    //NProgress.done();
                });
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { id: row.id };
                    removeUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getUsers();
                    });
                }).catch(() => {

                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                this.addForm = {
                    name: '',
                    sex: -1,
                    age: 0,
                    birth: '',
                    addr: ''
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
                            editUser(para).then((res) => {
                                this.editLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.editFormVisible = false;
                                this.getUsers();
                            });
                        });
                    }
                });
            },
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.addForm);
                            para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
                            addUser(para).then((res) => {
                                this.addLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['addForm'].resetFields();
                                this.addFormVisible = false;
                                this.getUsers();
                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { ids: ids };
                    batchRemoveUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getUsers();
                    });
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getUsers();
        }
    }

</script>
<style lang="scss">
    .el-table,.el-table__expanded-cell{
        background:#f0f2f5;
    }
    .el-pager li{
        border:1px solid #d1d3d6 !important;
        border-radius:5px;
        margin:0 3px;
        box-shadow:none;
        box-sizing:border-box;
        min-width:32px;
    }
    .table-content{
        background:#fff;
        padding:20px 40px;
        width:1350px;
        margin:0 auto;
    }
    .pagination-class{
        box-sizing:border-box;
        text-align:right;
        padding:20px 10px;
        background:#fff;
    }
    .el-table tr{
        background:#fff;
    }
    .el-table th{
        background:#f8f8f8;
    }
    .device-container{
        background:#f0f2f5;
        padding-top:50px;
    }
</style>