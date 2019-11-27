<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 课件管理
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
				<el-button
                    type="primary"
                    icon="el-icon-circle-plus-outline"
                    class="primary"
                    @click="handleAdd"
                >添加</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column prop="objectId" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="course_subject" label="科目"></el-table-column>
                <el-table-column prop="course_desc" label="描述"></el-table-column>
                <el-table-column label="日期" align="center">
				    <template slot-scope="scope">                    
                        <span style="margin-left: 10px">
						    {{scope.row.course_date ? (scope.row.course_date.iso ? scope.row.course_date.iso.substring(0, 10) : "") : ""}}
						</span>
                    </template>
				</el-table-column>
                <el-table-column prop="course_url" label="链接"></el-table-column>
				<el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="课件信息" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" :rules="rules" label-width="70px">
                <el-input type="hidden" v-model="form.objectId"></el-input>
				<el-form-item label="科目" prop="course_subject">
                    <el-input v-model="form.course_subject"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="course_desc">
                    <el-input v-model="form.course_desc"></el-input>
                </el-form-item>
				<el-form-item label="链接" prop="course_url">
                    <el-input v-model="form.course_url"></el-input>
                </el-form-item>
				<el-form-item label="日期" prop="course_date_iso">
                        <el-date-picker
                                type="date"
                                placeholder="选择日期"
                                v-model="form.course_date_iso"
                                value-format="yyyy-MM-dd"
                                style="width: 100%;"
                        ></el-date-picker>
                </el-form-item>
				
				<!--
				<el-upload
  			        class="upload-demo"
  			        ref="upload"
					action=""
					:http-request="uploadFile"
  			        :auto-upload="false">
                </el-upload>
				-->
				
				<el-form-item>
                    <el-button @click="editVisible = false">取 消</el-button>
                    <el-button type="primary" @click="saveEdit()">确 定</el-button>
                </el-form-item>
            </el-form>            
        </el-dialog>
    </div>
</template>

<script>
import { fetchData } from '../../api/index';
import request from '../../utils/request';

export default {
    name: 'basetable',
    data() {
        return {
            query: {
                course_subject: '',
                course_desc: '',
                pageIndex: 1,
                pageSize: 10
            },
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            pageTotal: 0,
            form: {},
            objectId: -1,
			fileList:[],
			rules: {
                course_subject: [
                    { required: true, message: '请输入科目', trigger: 'blur' }
                ],
                course_desc: [
                    { required: true, message: '请输入描述', trigger: 'change' }
                ],
			    course_date_iso: [
                    { required: true, message: '请输入日期', trigger: 'change' }
                ]
            }
        };
    },
    created() {
        this.getData();
    },
    methods: {
        getData() {
			request({
                url: 'https://api2.bmob.cn/1/classes/course?order=-course_date',
                method: 'get',
				headers : {'X-Bmob-Application-Id':'3bea17a55823d07e2487d6db68a04ba0', 'X-Bmob-REST-API-Key':'c8069787cb4ff2c10d99dae927667233', 'Content-Type':'application/json'},
                params: JSON.stringify(this.query)
            }).then(res => {
                this.tableData = res.results;
            });
        },
        // 触发搜索按钮
        handleSearch() {
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
        },
        // 删除操作
        handleDelete(index, row) {
		    var id = row.objectId;
			
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            }).then(() => {
			    request({
                    url: 'https://api2.bmob.cn/1/classes/course/' + id,
                    method: 'delete',
				    headers : {'X-Bmob-Application-Id':'3bea17a55823d07e2487d6db68a04ba0', 'X-Bmob-REST-API-Key':'c8069787cb4ff2c10d99dae927667233', 'Content-Type':'application/json'},
                    data: {}
                }).then(res => {
				    this.$message.success('删除成功');
				    this.handleSearch();       
                });
				
				             
            })
            .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].name + ' ';
            }
            this.$message.error(`删除了${str}`);
            this.multipleSelection = [];
        },
		// 添加操作
        handleAdd() {
		    this.form={};
			this.objectId = null;
            this.editVisible = true;
        },
        // 编辑操作
        handleEdit(index, row) {
            this.objectId = row.objectId;
			var course_date = row.course_date;
			var iso = "";
			if (course_date) {
			    iso = course_date.iso;
			}
			
			this.form={course_subject:row.course_subject, course_desc:row.course_desc, course_url:row.course_url, course_date_iso:iso};
			this.editVisible = true;
        },
		
		/*
		uploadFile(params) {
            const _file = params.file;
            const isLt10M = _file.size / 1024 / 1024 < 10;

            if (!isLt10M) {
                alert("请上传10M以下的文件");
                return false;
            }
			
			var formData = new FormData();
            formData.append("file", _file);
			
			request({
                url: 'https://api2.bmob.cn/2/files/' + _file.name,
                method: 'post',
				data: formData,
				headers : {'X-Bmob-Application-Id':'3bea17a55823d07e2487d6db68a04ba0', 'X-Bmob-REST-API-Key':'c8069787cb4ff2c10d99dae927667233', 'Content-Type':'application/octet-stream'}
            }).then(res => {				    
				alert(JSON.stringify(res));
            });
        },
		*/
		
        // 保存编辑
        saveEdit() {
		    var flag = false;
		    this.$refs["form"].validate((valid) => {
                if (valid) {                    
					flag = true;
                } else {
                    alert('error submit!');
                    return false;
                }
            });
			
			if (!flag) {
			    return;
			}
			
		    var id = this.objectId;
			
			var iso = this.form.course_date_iso;				
			if (iso) {
			    if (iso.length < 19) {
			        iso = iso + " 00:00:00";
				}
				this.form.course_date = {"__type": "Date", "iso": iso};
			}
			
			if (id && id.length > 0) {			    
			    request({
                    url: 'https://api2.bmob.cn/1/classes/course/' + id,
                    method: 'put',
					responseType: 'json',
				    headers : {'X-Bmob-Application-Id':'3bea17a55823d07e2487d6db68a04ba0', 'X-Bmob-REST-API-Key':'c8069787cb4ff2c10d99dae927667233', 'Content-Type':'application/json'},
                    data: JSON.stringify(this.form)
                }).then(res => {
				    this.editVisible = false;
                    this.handleSearch();
                });
				
				
			} else {
			    request({
                    url: 'https://api2.bmob.cn/1/classes/course/',
                    method: 'post',
				    headers : {'X-Bmob-Application-Id':'3bea17a55823d07e2487d6db68a04ba0', 'X-Bmob-REST-API-Key':'c8069787cb4ff2c10d99dae927667233', 'Content-Type':'application/json'},
                    data: JSON.stringify(this.form)
                }).then(res => {				    
					this.editVisible = false;
				    this.handleSearch();
                });
			}
            
        },
        // 分页导航
        handlePageChange(val) {
            this.$set(this.query, 'pageIndex', val);
            this.getData();
        }
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
