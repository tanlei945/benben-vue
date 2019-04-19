<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="用户名">
          <a-input placeholder="请输入用户名" v-decorator="['username', validatorRules.username ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="真实姓名">
          <a-input placeholder="请输入真实姓名" v-decorator="['realname', validatorRules.realname ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="昵称">
          <a-input placeholder="请输入昵称" v-decorator="['nickname', validatorRules.nickname ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="密码">
          <a-input placeholder="请输入密码" v-decorator="['password', validatorRules.password ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="电子邮箱">
          <a-input placeholder="请输入电子邮箱" v-decorator="['email', validatorRules.email ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="手机号">
          <a-input placeholder="请输入手机号" v-decorator="['mobile', validatorRules.mobile ]" />
        </a-form-item>
<!--        <a-form-item-->
<!--          :labelCol="labelCol"-->
<!--          :wrapperCol="wrapperCol"-->
<!--          label="头像">-->
<!--          <a-input placeholder="请输入头像" v-decorator="['avatar', validatorRules.avatar ]" />-->
<!--        </a-form-item>-->

        <a-form-item label="头像" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
          >
            <img v-if="validatorRules.avatar" :src="validatorRules.src" alt="头像" style="height:104px;max-width:300px"/>
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
        </a-form-item>

        <a-form-item label="性别（1：男 2：女）" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-select v-decorator="[ 'sex', {}]" placeholder="请选择性别">
            <a-select-option :value="1">男</a-select-option>
            <a-select-option :value="2">女</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="生日">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'birthday', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="格言">
          <a-input placeholder="请输入格言" v-decorator="['bio', validatorRules.bio ]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import {doMian} from '@/api/api'
  import pick from 'lodash.pick'
  import moment from "moment"
  import { ACCESS_TOKEN } from "@/store/mutation-types"
  import Vue from 'vue'

  export default {
    name: "UserModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        username:{rules: [{ required: true, message: '请输入用户名!' }]},
        realname:{rules: [{ required: true, message: '请输入真实姓名!' }]},
        nickname:{rules: [{ required: true, message: '请输入昵称!' }]},
        password:{rules: [{ required: true, message: '请输入密码!' }]},
        email:{rules: [{ required: true, message: '请输入电子邮箱!' }]},
        mobile:{rules: [{ required: true, message: '请输入手机号!' }]},
        avatar:{rules: [{ required: true, message: '请输入头像!' }]},
        sex:{rules: [{ required: true, message: '请输入性别（1：男 2：女）!' }]},
        bio:{rules: [{ required: true, message: '请输入格言!' }]},
          src:"",
        },
        headers:{},
        url: {
          add: "/user/add",
          edit: "/user/edit",
          fileUpload:doMian+"sys/common/upload",
        },
      }
    },
    created () {
      const token = Vue.ls.get(ACCESS_TOKEN);
      this.headers = {"X-Access-Token":token}
    },

    computed:{
      uploadAction:function () {
        return this.url.fileUpload;
      }
    },

    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.validatorRules.src=record.avatar;
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'username','realname','nickname','password','email','mobile','avatar','sex','bio'))
		  //时间格式化
          this.form.setFieldsValue({birthday:this.model.birthday?moment(this.model.birthday):null})
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.birthday = formData.birthday?formData.birthday.format('YYYY-MM-DD HH:mm:ss'):null;

            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },
      beforeUpload: function(file){
        var fileType = file.type;
        if(fileType.indexOf('image')<0){
          this.$message.warning('请上传图片');
          return false;
        }
        //TODO 验证文件大小
      },
      handleChange (info) {
        console.log(info);
        if (info.file.status === 'uploading') {
          this.uploadLoading = true
          return
        }
        if (info.file.status === 'done') {
          var response = info.file.response;
          this.uploadLoading = false;
          console.log(response);
          if(response.success){
            this.validatorRules.src = response.message;
            this.model.avatar = response.message;
          }else{
            this.$message.warning(response.message);
          }
        }
      },
    }
  }
</script>

<style scoped>

</style>