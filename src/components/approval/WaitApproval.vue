<template>
    <div id="list">
        <h1 class="title">物资申请列表：</h1>
        <el-divider></el-divider>
        <el-card class="filter-bar" shadow="never">
            <el-form :inline="true" :model="filterCondition"  ref="approvalFilter">
                <el-form-item label="申请等级">
                    <el-select v-model="filterCondition.level" placeholder="申请等级">
                        <el-option label="低" value="低"></el-option>
                        <el-option label="中" value="中"></el-option>
                        <el-option label="高" value="高"></el-option>
                        <el-option label="紧急" value="紧急"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="审批待处理人">
                        <el-select v-model="filterCondition.approvalUserId" placeholder="审批待处理人" clearable :style="{width: '100%'}">
                            <el-option v-for="(item, index) in userList" :key="index" :label="item.label" :value="item.value" :disabled="item.disabled"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <el-button type="info" size="medium" @click="resetForm('approvalFilter', resetFilter)">清 空</el-button>
                    <el-button type="primary" size="medium" @click="submitForm('approvalFilter', loadList)">筛选申请</el-button>
                </el-form-item>
            </el-form>
        </el-card>
        <el-table :data="list" border style="width: 100%" :highlight-current-row="true" :stripe="true">
            <el-table-column fixed prop="id" label="ID" width="180" align="center"></el-table-column>
            <el-table-column  prop="title" label="申请标题" width="180" align="center"></el-table-column>
            <el-table-column  prop="content" label="详细内容" width="200" align="center"></el-table-column>
            <el-table-column  prop="createTime" label="创建时间" width="150" align="center"></el-table-column>
            <el-table-column  prop="lastUpdateTime" label="最后更新时间" width="150" align="center"></el-table-column>
            <el-table-column  prop="level" label="申请等级" width="100" align="center"></el-table-column>
            <el-table-column prop="status" label="申请状态" width="100" align="center"></el-table-column>
            <el-table-column prop="approvalUserId" label="关联审批人ID" width="150" align="center"></el-table-column>
            <el-table-column prop="eventId" label="关联事件ID" width="150" align="center"></el-table-column>
            <el-table-column prop="createUserId" label="创建人ID" width="150" align="center"></el-table-column>
            <el-table-column fixed="right" label="操作" width="200" align="center">
                <template slot-scope="scope">
                    <el-button type="primary" round size="mini" @click="showDetail(scope.row)">详 情</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination @current-change="loadList" background
                       :current-page.sync="filterCondition.currentPage"
                       :page-size="filterCondition.pageSize"
                       layout="total, prev, pager, next"
                       :total="filterCondition.total">
        </el-pagination>
        <el-drawer title="物资申请详情：" :visible.sync="showMaterialApplyDetail" direction="ltr">
            <div>
                <el-card class="detail detail-form" shadow="never">
                    <p><span>申请ID：</span>{{detailRecord.id}}</p>
                    <p><span>申请人：</span>{{detailRecord.createUserId}}</p>
                    <p><span>创建时间：</span>{{detailRecord.createTime}}</p>
                    <p><span>关联事件ID：</span>{{detailRecord.eventId}}</p>
                    <p><span>事件影响等级：</span>{{detailRecord.level}}</p>
                    <p><span>状态：</span>{{detailRecord.status}}</p>
                    <el-divider></el-divider>
                    <p>申请列表：</p>
                    <span v-for="(i , index) in materialApplyList" :key="index + Date.now()">
                        <span style="padding-left: 15px">
                            <b>{{i.name}} * {{i.num}}</b>
                        </span>
                         <el-divider direction="vertical"></el-divider>
                    </span>
                    <el-divider></el-divider>
                    <p>详细说明：</p>
                    <el-input v-model="detailRecord.content" readonly type="textarea" :autosize="{minRows: 4, maxRows: 8}"></el-input>
                    <el-divider></el-divider>
                    <div>
                        <el-button type="danger" round @click="rejectApproval">驳 回 </el-button>
                        <el-button type="success" round @click="passApproval">通 过</el-button>
                    </div>
                </el-card>
            </div>
        </el-drawer>
    </div>
</template>

<script>
    import {resetForm, submitForm} from "@/base/ele-base";
    import { approvalApi, userApi } from '@/axios/api'
    import entity from '@/base/entity'
    import {mergeBean} from '@/base/utils'
    export default {
        name: "WaitApproval",
        data(){
            return{
                entity,
                showMaterialApplyDetail: false,
                detailRecord:{},
                materialApplyList:[],
                //筛选条件
                list: [],
                filterCondition:{

                },
                userList:[]
            }
        },
        methods:{
            mergeBean, resetForm,submitForm,
            showDetail(approval){
                this.materialApplyList = [];
                console.log("显示物资详情:");
                console.log(approval);
                this.detailRecord = approval;
                const arr = approval.materialApply.split("@");
                for(const a in arr){
                    if (arr[a] !== undefined && arr[a] !== "") {
                        const aa = arr[a].split("#");
                        this.materialApplyList.push({
                            name: aa[0],
                            num: aa[1]
                        });
                    }
                }
                this.showMaterialApplyDetail = true;
            },
            rejectApproval(){
                console.log("拒绝审批, ID = " + this.detailRecord.id);
                approvalApi.updateApproval({id: this.detailRecord.id, status:'已拒绝'}).then(res=>{
                    this.$message({message:'该审批已经流转至拒绝状态!', type:'success'});
                });
            },
            passApproval(){
                console.log("通过审批, ID = " + this.detailRecord.id);
                approvalApi.updateApproval({id: this.detailRecord.id, status:'待配送'}).then(res=>{
                    this.$message({message:'该审批已经流转至待配送状态!', type:'success'});
                });
            },
            loadList(){
                this.filterCondition.status = "待审核";
                approvalApi.listApproval(this.filterCondition).then(res=>{
                    this.list = res.data.list;
                    this.filterCondition = this.mergeBean(res.data.pagination, this.filterCondition);
                });
            },
            resetFilter(){
                this.filterCondition.status = undefined;
                this.filterCondition.level = undefined;
                this.filterCondition.approvalUserId = undefined;
                this.loadList();
            },
            async loadUser(){
                let res = await userApi.listUser({currentPage: 1, pageSize: 100, identityName:'审批人员'});
                if(res.status){
                    const list = res.data.list;
                    this.$message({message:"加载本部门下应急审批人员完成, 数量："+list.length , type: 'success'});
                    list.forEach((v)=>{
                        this.userList.push({
                            'label': v.nickName,
                            'value': v.id
                        });
                    });
                }
            }
        },
        mounted(){
            console.log("挂载待审批列表");
            this.filterCondition = this.mergeBean(this.entity.pagination, this.filterCondition);
            this.loadList();
            this.loadUser();
        }
    }
</script>

<style scoped>
    .title{
        text-align: left;
        padding: 3px 5px;
        border-left: rgba(44, 62, 80, 0.87) 5px solid;
    }
    .el-pagination{
        margin: 10px auto;
    }
    .filter-bar{
        margin: 10px auto;
        text-align: left;
    }
    .detail{
        margin: 10px;
        padding: 0 10px;
    }
    .detail div{
        display: inline-block;
        margin: 10px;
    }
    .detail-form p{
        margin: 10px 0;
        padding-left: 10px;
    }
    .detail-form p span{
        display: inline-block;
        width: 200px;
    }
</style>