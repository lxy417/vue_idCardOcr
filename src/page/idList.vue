<template>
    <div class="fillcontain">
        <head-top></head-top>
        <div class="table_container">
            <el-table
                :data="tableData"
                highlight-current-row
                style="width: 100%">
                <el-table-column
                  type="index"
                  width="100">
                </el-table-column>
                <el-table-column
                  property="username"
                  label="姓名"
                  width="220">
                </el-table-column>
                <el-table-column
                  property="idCard"
                  label="身份证号"
                  width="220">
                </el-table-column>
                <el-table-column
                  property="isReport"
                  label="是否报道">
                </el-table-column>
            </el-table>
            <div class="Pagination" style="text-align: left;margin-top: 10px;">
                <el-pagination
                  @size-change="handleSizeChange"
                  @current-change="handleCurrentChange"
                  :current-page="currentPage"
                  :page-size="20"
                  layout="total, prev, pager, next"
                  :total="count">
                </el-pagination>
            </div>
        </div>
    </div>
</template>

<script>
    import axios  from 'axios'
    import headTop from '../components/headTop'
    import {getUserList, getUserCount} from '@/api/getData'
    export default {
        data(){
            return {
                tableData: [{
                  idCard: '11111111111111111111',
                  username: '刘星宇',
                  isReport: "true"
                }, {
                  idCard: '11111111111111111111',
                  username: '刘星宇',
                  isReport: "true"
                }, {
                  idCard: '11111111111111111111',
                  username: '刘星宇',
                  isReport: "true"
                }, {
                  idCard: '11111111111111111111',
                  username: '刘星宇',
                  isReport: "true"
                }],
                currentRow: null,
                offset: 0,
                limit: 20,
                count: 0,
                currentPage: 1,
            }
        },
    	components: {
    		headTop,
    	},
        created(){
            this.initData();
        },
        methods: {
            async initData(){
                await axios.get("http://127.0.0.1:5000/getall")
                .then((response) => {
                    // console.log("111111111111",response.data);
                    const countData = response.data
                    if (countData.status == 1) {
                        this.count = countData.count;
                        this.process(countData.data)
                    }else{
                        throw new Error('获取数据失败');
                    }
                });  
            },
            process(data){
                this.tableData=[]
                for(let i=0;i<this.count;i++){
                    this.tableData.push({
                        idCard: data[i][1],
                        username: data[i][0],
                        isReport: data[i][2]?"是":"否"
                    })
                }
            },
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
            },
            handleCurrentChange(val) {
                this.currentPage = val;
                this.offset = (val - 1)*this.limit;
                this.getUsers()
            },
        },
    }
</script>

<style lang="less">
	@import '../style/mixin';
    .table_container{
        padding: 20px;
    }
</style>
