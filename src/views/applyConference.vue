<template>
    <div>


        <div class="warnning">
            <el-alert
                    title="提示"
                    type="error"
                    description="请尽量在会议开始前1小时之前进行申请，且每个会议之间间隔30分钟以供特殊情况时使用"
            style="border-radius: 30px"
            :closable="false">
            </el-alert>
        </div>

        <!--            按照条件搜索-->
        <div class="mConditions">
            <el-select  v-model="roomFloor" clearable placeholder="楼层" @change="change">
                <el-option
                        v-for="item in floors"
                        :key="item.roomFloor"
                        :label="item.roomFloor"
                        :value="item.roomFloor">
                </el-option>
            </el-select>

            <el-select
                    class="mselect"
                    v-model="roomType"
                    clearable
                    placeholder="类型"
                    @change="change">
                <el-option
                        v-for="item in types"
                        :key="item.roomType"
                        :label="item.roomType"
                        :value="item.roomType">
                </el-option>
            </el-select>

            <el-select
                    class="mselect"
                    v-model="roomSize"
                    clearable
                    placeholder="可容纳人数"
                    @change="change">
                <el-option
                        v-for="item in sizes"
                        :key="item.roomSize"
                        :label="item.roomSize"
                        :value="item.roomSize">
                </el-option>
            </el-select>

        </div>
        <el-table :data="conferenceRooms.filter(data => !this.search || data.roomNo.toLowerCase().includes(this.search.toLowerCase())||
                data.roomFloor.toLowerCase().includes(this.search.toLowerCase()) || data.roomType.toLowerCase().includes(this.search.toLowerCase()))"
                  style="width: 100%;text-align: center"
>
            <el-table-column
                    prop="roomNo"
                    label="门牌号"
                    width="150">
            </el-table-column>
            <el-table-column
                    prop="roomFloor"
                    label="楼层"
                    width="150">
            </el-table-column>
            <el-table-column
                    prop="roomType"
                    label="类型"
                    width="250">
            </el-table-column>
            <el-table-column
                    prop="roomSize"
                    label="可容纳人数"
                    width="180">
            </el-table-column>

            <el-table-column label="设备" width="180">
                <template slot-scope="scope">
                    <el-button type="text" @click="lookDevice(scope.row)">查看</el-button>
                </template>

            </el-table-column>


            <el-table-column
                    align="right" >
                <template slot="header" slot-scope="scope">
                    <el-input
                            v-model="search"
                            size="mid"
                            placeholder="输入楼层、会议室门牌号、会议室类型"/>
                </template>
                <template slot-scope="scope">
                    <el-button
                            size="medium"
                            @click="handleApply(scope.$index, scope.row)"

                            round
                            type="warning"
                            style="margin-right: 10px"
                    >申 请</el-button>

                </template>
            </el-table-column>
        </el-table>


<!--        申请会议室弹出框-->
        <el-dialog title="申请会议室" :visible.sync="dialogFormApply">
            <el-form :model="record.conferenceRecord"  label-position="right" label-width="80px" status-icon :rules="rules" ref="applyForm">
                <el-form-item label="会议主题" prop="theme">
                    <el-input v-model="record.conferenceRecord.theme" placeholder="请输入会议主题" style="width: 250px"></el-input>
                </el-form-item>
                <el-form-item label="会议介绍"  prop="digest">
                    <el-input  type="textarea" autosize v-model="record.conferenceRecord.digest" placeholder="请输入会议摘要" style="width: 250px">
                    </el-input>
                </el-form-item>
                <el-form-item label="会议人数" prop="personCount">
                    <el-input  @blur="examinePersonCount"  v-model.number="record.conferenceRecord.personCount"  placeholder="会议人数" style="width: 250px">
                    </el-input>
                </el-form-item>

                <el-form-item label="开始时间" prop="startTime" style="width: 250px">
                    <el-date-picker
                            v-model="record.conferenceRecord.startTime"
                            type="datetime"
                            placeholder="选择开始日期时间"
                            :picker-options="pickerOptions"
                            @change="searchTimeConflict"
                            :default-time="defaultStartTime"
                            value-format="yyyy-MM-dd HH:mm:ss">
                    </el-date-picker>
                </el-form-item>

                <el-form-item label="结束时间" prop="endTime" style="width: 250px">
                    <el-date-picker
                            v-model="record.conferenceRecord.endTime"
                            type="datetime"
                            placeholder="选择结束日期时间"
                            :picker-options="pickerOptions"
                             @change="searchTimeConflict"
                            :default-time="defaultEndTime"
                            value-format="yyyy-MM-dd HH:mm:ss">
                    </el-date-picker>
                </el-form-item>

                <el-form-item>
                    <el-button style="background-color:oldlace " @click="submitForm('applyForm')" >提交申请</el-button>
                    <el-button @click="resetForm('applyForm')">重置</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>

        <!--        查看设备的表格-->
        <el-dialog title="设备" :visible.sync="dialogTableVisible" style="text-align: center">
            <el-table
                    :data="devices"
                    style="text-align: center"
            >
                <el-table-column
                        prop="dname"
                        label="设备名称"
                       >
                </el-table-column>
                <el-table-column
                        prop="dnumber"
                        label="数量"

                >

                </el-table-column>



            </el-table>
        </el-dialog>
    </div>
</template>

<script>
    import Cookies from 'js-cookie';
    export default {
        name: "applyConference",
        data(){
            return{
                devices:[
                    {"did":'',
                        "dname":'',
                        "dnumber":'',
                        "roomId":'',

                    }
                ],

                dialogTableVisible:false,


                pickerOptions: {
                    disabledDate(time) {
                        return time.getTime() < Date.now() - 8.64e7;
                    },
                },

                //日期选择器里只选日期时  默认带出的时刻
                //打开弹窗时按"当前时间往后推"重新算一次  避免默认成10点这种已经过去的时间
                defaultStartTime: '10:00:00',
                defaultEndTime: '10:30:00',

                //申请弹出框
                dialogFormApply:false,

                //查出没有重复的条件 进行筛选 后台数据用list嵌套map传到前端
                floors:[{
                    roomFloor:'1'
                }],
                types:[{
                    roomType:'多媒体'
                }],
                sizes:[{
                    roomSize:100
                }],

                //与下拉选框双向绑定
                roomFloor:'',
                roomSize:'',
                roomType:'',

                //搜索框
                search:'',

                //表格绑定
                conferenceRooms: [{
                    roomId:2,
                    roomNo: "003",
                    roomFloor: 11,
                    roomType: '大多媒体会议室',
                    roomSize: '300',
                    roomState: 1
                }],

                record:{
                    "apply": {
                        "roomSize":'', //申请会议室时 会议人数要比较
                        "depId": '',
                        "roomId": ''
                    },
                    "conferenceRecord": {
                        "startTime": null,
                        "endTime": null,
                        "personCount":'',
                        "theme": "",
                        "digest": "",
                    }
                },

                rules: {
                    theme: [
                        {required: true, message: '请输入会议主题', trigger: 'blur'},
                        {min: 5, max: 50, message: '长度在 5 到 50 个字符', trigger: 'blur'}
                    ],
                    digest: [
                        {required: true, message: '请输入会议介绍', trigger: 'blur'},
                        {min: 5, max: 50, message: '长度在 5 到 50 个字符', trigger: 'blur'}
                    ],
                    personCount: [
                        {required: true, message: '请输入参与会议人数', trigger: 'blur'},
                        { type: 'number', message: '必须为数字', trigger: 'blur' }
                    ],
                    startTime: [
                        {required: true, message: '请选择会议开始时间', trigger: 'blur'},
                    ],
                    endTime: [
                        {required: true, message: '请选择会议结束时间', trigger: 'blur'},
                    ],

                },
            }
        },

        methods:{
            lookDevice(row){
                let roomId = row.roomId;
                let _this = this;
                _this.axios.get("/device/listbyapply/"+roomId,{
                    headers:{
                        "Authorization":localStorage.getItem("token")
                    }
                }).then(res=>{
                    _this.devices = res.data.data;
                    this.dialogTableVisible = true;
                })

            },

            //检查参与会议人数是否小于等于会议室可容纳人数 并且不能为0
            examinePersonCount(){
                    if(this.record.conferenceRecord.personCount!==null&&this.record.conferenceRecord.personCount!==0){
                        if(this.record.conferenceRecord.personCount>this.record.apply.roomSize){
                            this.$message({
                                message: '人数超过会议室可容纳人数',
                                type:'error',
                                center: true
                            });

                            this.record.conferenceRecord.personCount = '';
                        }

                    }

                    if(this.record.conferenceRecord.personCount === 0){
                        this.$message({
                            message: '参与会议人数不能为0',
                            type:'error',
                            center: true
                        });
                        this.record.conferenceRecord.personCount = '';
                    }
            },

            //打开申请弹窗时  按当前时间重算选择器只选日期时带出的默认时刻
            //开始时间取当前时间往后推30分钟  结束时间取开始时间往后推1小时
            //这样即使选的是今天  也不会默认成10点这种已经过去的时间
            setDefaultTime(){
                let pad = n => (n < 10 ? '0' + n : '' + n);
                let fmt = d => pad(d.getHours()) + ':' + pad(d.getMinutes()) + ':00';

                let start = new Date(new Date().getTime() + 30*60*1000);
                start.setSeconds(0,0);
                //会议室10点之后才能使用  算出来早于10点就默认成10点
                if(start.getHours() < 10){
                    start.setHours(10,0,0,0);
                }
                let end = new Date(start.getTime() + 60*60*1000);

                this.defaultStartTime = fmt(start);
                this.defaultEndTime = fmt(end);
            },

            //把 "yyyy-MM-dd HH:mm:ss" 转成时间戳  兼容部分浏览器不支持该格式的情况
            parseTime(timeStr){
                return new Date(String(timeStr).replace(/-/g,'/')).getTime();
            },

            //本地时间规则校验  只做判断不发请求
            //校验通过返回 null  不通过返回错误提示  两个时间还没选全时也返回 null（交给表单的必填校验）
            //规则：结束晚于开始、会议最短10分钟、开始时间不能早于当前时间、使用时间10-23点
            validateTime(){
                let startTime = this.record.conferenceRecord.startTime;
                let endTime = this.record.conferenceRecord.endTime;
                if(!startTime || !endTime){
                    return null;
                }
                let start = this.parseTime(startTime);
                let end = this.parseTime(endTime);
                if(isNaN(start) || isNaN(end)){
                    return '时间格式有误，请重新选择';
                }
                if(start >= end){
                    return '结束时间必须晚于开始时间';
                }
                if(end - start < 10*60*1000){
                    return '会议时间最短为10分钟';
                }
                //开始时间不能早于当前时间  留10分钟容错（与后端一致）  这样选择"此刻"、或者填完表停留一会儿也能正常提交
                if(start < new Date().getTime() - 10*60*1000){
                    return '开始时间不能早于当前时间，请重新选择';
                }
                //会议室的使用时间只能是早上10点到晚上23点
                if(new Date(start).getHours() < 10 || new Date(end).getHours() >= 23){
                    return '会议室的使用时间为每天的10:00-23:00';
                }
                return null;
            },

            //选择时间后触发  先做本地校验 再请求后台判断该时间段是否已被占用
            //注意：校验不通过只提示 不再清空用户已经选好的时间  避免用户反复重选
            searchTimeConflict(){
                //只选了一个时间 先不校验 等两个时间都选好
                if(!this.record.conferenceRecord.startTime || !this.record.conferenceRecord.endTime){
                    return Promise.resolve(false);
                }
                let error = this.validateTime();
                if(error){
                    this.$message({
                        message: error,
                        type:'error',
                        center: true
                    });
                    return Promise.resolve(false);
                }
                return this.checkTimeConflict(true);
            },

            //请求后台 查询所选会议室在所选时间段内是否与审核通过的会议冲突
            //showSuccess为true时  校验通过会提示"时间允可"（用户手动选时间时）
            checkTimeConflict(showSuccess){
                let _this = this;
                let startTime = this.record.conferenceRecord.startTime;
                let endTime = this.record.conferenceRecord.endTime;
                let roomId = this.record.apply.roomId;
                return new Promise(resolve=>{
                    _this.axios.get("/apply/searchtimeconflict/" + roomId + "/" + startTime
                        + "/" + endTime, {
                        headers: {
                            "Authorization": localStorage.getItem("token")
                        }
                    }).then(res => {
                        if (res.data.data === '1') {
                            if(showSuccess){
                                _this.$message({
                                    message: '时间允可',
                                    type: 'success',
                                    center: true
                                });
                            }
                            resolve(true);
                        } else {
                            _this.$message({
                                message: '时间冲突，请选择其他时间或者会议室',
                                type: 'error',
                                center: true
                            });
                            resolve(false);
                        }
                    }).catch(()=>{
                        resolve(false);
                    })
                })
            },

            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (!valid) {
                        console.log('error submit!!');
                        return false;
                    }

                    //表单必填校验通过后 再完整校验一次时间
                    //因为有的浏览器不一定触发change 只靠选时间时的校验可能会漏掉
                    let startTime = this.record.conferenceRecord.startTime;
                    let endTime = this.record.conferenceRecord.endTime;
                    if (!startTime || !endTime) {
                        this.$message({
                            message: '请选择会议的开始时间和结束时间',
                            type: 'error',
                            center: true
                        });
                        return false;
                    }
                    let error = this.validateTime();
                    if (error) {
                        this.$message({
                            message: error,
                            type: 'error',
                            center: true
                        });
                        return false;
                    }

                    let _this = this;
                    //提交前再跟后台确认一次时间是否冲突 防止页面停留太久期间被别人占用
                    _this.checkTimeConflict(false).then(ok => {
                        if (!ok) {
                            return;
                        }
                        _this.axios.post("/apply/add", _this.record, {
                            headers: {
                                "Authorization": localStorage.getItem("token")
                            }
                        }).then(res => {
                            _this.$message({
                                message: '申请成功,等待管理员审核',
                                type: 'success',
                                center: true
                            });
                            _this.dialogFormApply = false;
                            _this.resetRecord();
                        })
                    })
                })
            },

            //提交成功后清空表单 并把校验状态一起清掉 避免下次打开还带着红字提示
            resetRecord(){
                this.record.conferenceRecord = {
                    "startTime": null,
                    "endTime": null,
                    "personCount": '',
                    "theme": "",
                    "digest": ""
                };
                if(this.$refs['applyForm']){
                    this.$refs['applyForm'].clearValidate();
                }
            },


            resetForm(formName) {
                this.$refs[formName].resetFields();
            },

            change(){
                //这样才能避免为空的时候也能传递数据  因为后台restFul风格下传递的参数不能不传 如果直接写this.xxx
                //当属性为空时  传递不了
                let floor = JSON.stringify(this.roomFloor);
                let type1  = JSON.stringify(this.roomType);
                let size = JSON.stringify(this.roomSize);
                let url = "/conference-room/listbyonstate/"+floor+"/"+type1+"/"+size;

                let _this = this;
                this.axios.get(url,{
                    headers:{
                        "Authorization":localStorage.getItem("token")
                    }
                }).then(res=>{
                    _this.conferenceRooms = res.data.data;
                })
            },

            getConditions(){
                let _this = this;
                this.axios.get("/conference-room/getconditionsonstate",{
                    headers:{
                        "Authorization":localStorage.getItem("token")
                    }
                }).then(res=>{
                    _this.floors = res.data.data.floors;
                    _this.types = res.data.data.types;
                    _this.sizes = res.data.data.sizes;
                })
            },
            getAllConferenceRoom(){
                let _this = this;
                _this.axios.get("/conference-room/listallonstate",{
                    headers:{
                        "Authorization":localStorage.getItem("token")
                    }
                }).then(res=>{
                    _this.conferenceRooms = res.data.data;
                })
            },
            handleApply(index, row) {
                this.dialogFormApply = true;
                this.setDefaultTime();
                this.record.apply.roomId = row.roomId;
                this.record.apply.roomSize = row.roomSize;

                //员工还是部门用户
                let isEmployee = JSON.parse(Cookies.get("userInfo")).depId;
               if( isEmployee === undefined){
                   this.record.apply.depId = JSON.parse(Cookies.get("userInfo")).id;
               }else{
                   this.record.apply.depId = isEmployee
               }

            },

            tableRowClassName({row, rowIndex}) {
                if (rowIndex === 1) {
                    return 'warning-row';
                } else if (rowIndex === 3) {
                    return 'success-row';
                } else if (rowIndex == 5) {
                    return 'warning-row';
                } else if (rowIndex === 7) {
                    return 'success-row'
                }
                return '';
            }

        },
        created() {
            this.getConditions();
            this.getAllConferenceRoom();
        }
    }
</script>

<style scoped>

    .warnning{
        margin-top: 30px;
    }
</style>