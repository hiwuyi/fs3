<template>
    <div class="login" v-loading="loginLoad">
        <div class="formContent">
            <el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" class="demo-ruleForm">
                <el-form-item prop="Accesskey">
                    <div class="title">Access Key</div>
                    <el-input type="text" v-model="ruleForm.Accesskey" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item prop="Secretkey">
                    <div class="title">Secret Key</div>
                    <el-input type="password" v-model="ruleForm.Secretkey" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button icon="iconfont icon-denglutongji" @click="submitForm('ruleForm')"></el-button>
                </el-form-item>
            </el-form>
        </div>
        <div class="login_bot">
            <div class="left">{{location}}</div>
            <router-link to="/fs3/login" class="right"><img :src="login_img" /></router-link>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import Moment from 'moment'
export default {
    data() {
        var validateAccesskey = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('Please fill in this field.'));
            } else {
                callback();
            }
        };
        var validateSecretkey = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('Please fill in this field.'));
            } else if (value.length < 8) {
                callback(new Error('Length cannot be less than 8.'));
            } else {
                callback();
            }
        };
        return {
            login_img: require('@/assets/images/logo.png'),
            ruleForm: {
                Accesskey: '',
                Secretkey: ''
            },
            rules: {
                Accesskey: [
                    { validator: validateAccesskey, trigger: 'blur' }
                ],
                Secretkey: [
                    { validator: validateSecretkey, trigger: 'blur' }
                ]
            },
            location: '',
            loginLoad: false
        };
    },
    computed: {
        email() {
            return this.$store.state.user.email
        },
    },
    watch: {
        $route: function (to, from) {
            if(this.bodyWidth){

            }
        }
    },
    methods: {
        submitForm(formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    let _this = this;
                    let currentUiVersion = 'MINIO_UI_VERSION';
                    _this.loginLoad = true;

                    let p = window.location.pathname
                    const minioBrowserPrefix = p.slice(0, p.indexOf("/", 1))
                    let postUrl = _this.data_api + `/minio/webrpc`
                    let dataLogin = {
                        id: 1,
                        jsonrpc: "2.0",
                        method: "web.Login",
                        params:{
                            username: _this.ruleForm.Accesskey,
                            password: _this.ruleForm.Secretkey
                        }
                    }

                    axios.post(postUrl, dataLogin,{headers: {}}).then((response) => {
                        let json = response.data
                        let error = json.error
                        let result = json.result
                        if (error) {
                            _this.$message.error(error.message);
                            _this.loginLoad=false
                            return false
                        }

                        if (!Moment(result.uiVersion).isValid()) {
                            _this.$message.error("Invalid UI version in the JSON-RPC response");
                        }
                        if (result.uiVersion !== currentUiVersion && currentUiVersion !== 'MINIO_UI_VERSION') {
                            storage.setItem('newlyUpdated', true)
                            location.reload()
                        }

                        _this.loginLoad=false
                        sessionStorage.fs3MinioLoginAccessToken = result.token
                        _this.$store.state.user.accessToken = result.token
                        console.log('login token: ', _this.$store.state.user.accessToken)
                        _this.$router.replace({ name: 'fs3' })

                        //Storage account
                        localStorage.removeItem('MinioAccountNumber')
                        localStorage.setItem('MinioAccountNumber', JSON.stringify(_this.ruleForm))

                    }).catch(function (error) {
                        console.log(error);
                        _this.loginLoad=false
                    });
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });
        },
        resetForm(formName) {
            this.$refs[formName].resetFields();
        }
    },
    mounted() {
        this.location = window.location.host;
    }
};
</script>
<style  lang="scss" scoped>
.login{
    height: 100%;
    min-height: 500px;
    background: #002a37;
    text-align: center;
    .formContent{
        width: 80%;
        max-width: 500px;
        height: calc(100% - 100px);
        margin: auto;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;
        .el-form /deep/{
            width: 100%;
            margin: auto;
            .el-form-item{
                width: 100%;
                .title{
                    color: #8e8e8e;
                    line-height: 1;
                }
                .el-input{
                    .el-input__inner{
                        background-color: transparent;
                        box-shadow: none;
                        border: 0;
                        border-bottom: 1px solid rgba(255, 255, 255, 0.1);
                        color: #fff;
                        text-align: center;
                    }
                }
                .el-button{
                    width: 0.55rem;
                    height: 0.55rem;
                    padding: 0;
                    margin: 0.15rem auto 0;
                    background-color: transparent;
                    border: 1px solid #fff;
                    border-radius: 50%;
                    opacity: 0.3;
                    font-size: 0.2rem;
                    font-family: inherit;
                    color: #fff;
                    i{
                        font-size: 0.22rem;
                        font-weight: bold;
                    }
                    &:hover{
                        opacity: 0.8;
                    }
                    @media screen and (max-width:600px){
                        font-size: 22px;
                        width: 40px;
                        height: 40px;
                        i{
                            font-size: 22px;
                        }
                    }
                }
            }
        }
    }
    .login_bot{
        height: 100px;
        padding: 0 0.5rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
        .left{
            font-size: 0.2rem;
            color: rgba(255, 255, 255, 0.4);
            line-height: 0.8rem;
            @media screen and (max-width:600px){
                font-size: 14px;
            }
        }
        .right{
            img{
                height: 0.4rem;
                margin: 0.2rem 0;
                @media screen and (max-width:600px){
                    height: 30px;
                    margin: 15px 0 0;
                }
            }
        }
    }
}
@media screen and (max-width:1024px){

}
@media screen and (max-width: 600px){

}
</style>
