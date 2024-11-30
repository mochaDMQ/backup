<script setup>
import { ref, reactive, getCurrentInstance, nextTick, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import md5 from "js-md5";
const { proxy } = getCurrentInstance();
const router = useRouter();
const route = useRoute();
const api = {
  sendMailCode: "/sendEmailCode",
  register: "/register",
  login: "/login",
  resetPwd: "/resetPwd",
};

const formData = ref({});
const formDataRef = ref();
const opType=ref(1) // 0:注册 1:登录
const showPanel=(type)=>{
  opType.value = type;
  resetForm();
};

//重置表单
const resetForm = () => {
  nextTick(() => {
    formDataRef.value.resetFields();
    formData.value = {};
    if (opType.value == 1) {     //登录
      const cookieLoginInfo = proxy.VueCookies.get("loginInfo");
      if (cookieLoginInfo) {
        formData.value = cookieLoginInfo;
      }
    }
  });
};

// 表单校验规则
const rules = {
  // email: [
  //   { required: true, message: "请输入邮箱" },
  //   { validator: proxy.Verify.email, message: "请输入正确的邮箱" },
  // ],
  // password: [{ required: true, message: "请输入密码" }],
  // emailCode: [{ required: true, message: "请输入邮箱验证码" }],
  // nickName: [{ required: true, message: "请输入昵称" }],
  // registerPassword: [
  //   { required: true, message: "请输入密码" },
  //   {
  //     validator: proxy.Verify.password,
  //     message: "密码只能是数字，字母，特殊字符 8-18位",
  //   },
  // ],
  // reRegisterPassword: [
  //   { required: true, message: "请再次输入密码" },
  //   {
  //     validator: checkRePassword,
  //     message: "两次输入的密码不一致",
  //   },
  // ],
  // checkCode: [{ required: true, message: "请输入图片验证码" }],
};
</script>

<template>
  <div class="login-body">
    <div class="login-panel"> <!--登录框-->
      <el-form
          class="login-register"
          :model="formData"
          :rules="rules"
          ref="formDataRef"
          @submit.prevent
      >
        <div class="login-title">数据备份</div>
        <!--账号：注册/登录都需要-->
        <el-form-item prop="email"> <!--prop用于校验-->
          <el-input
              size="large"
              clearable
              placeholder="请输入邮箱"
              v-model.trim="formData.email"
              maxLength="150"
            >
            <template #prefix>
              <span class="iconfont icon-account"></span>
            </template>
          </el-input>
        </el-form-item>

        <!--登录：密码-->
        <el-form-item prop="password" v-if="opType==1">
          <el-input
              type="password"
              size="large"
              placeholder="请输入密码"
              v-model.trim="formData.password"
              show-password
          >
            <template #prefix>
              <span class="iconfont icon-password"></span>
            </template>
          </el-input>
        </el-form-item>

        <!-- 登录：是否记住密码 -->
        <el-form-item v-if="opType==1">
          <div class="remember-panel">
            <el-checkbox v-model="formData.remember">记住密码</el-checkbox>
          </div>
          <div class="no-account">
            <!-- 2→重置，0→注册 -->
<!--            <a href="javascript:void(0)" class="a-link" @click="showPanel(2)">重置密码</a>-->
            <a href="javascript:void(0)" class="a-link" @click="showPanel(0)">注册账号</a>
          </div>
        </el-form-item>

        <!--  注册/重置  -->
        <div v-if="opType == 0">
          <el-form-item prop="emailCode">
            <!--获取邮箱验证码-->
            <div class="send-email-panel">
              <el-input
                  size="large"
                  placeholder="请输入邮箱验证码"
                  v-model.trim="formData.emailCode"
              >
                <template #prefix>
                  <span class="iconfont icon-checkcode"></span>
                </template>
              </el-input>
              <el-button
                  class="send-mail-btn"
                  type="primary"
                  size="large"
                  @click="getEmailCode">获取验证码</el-button
              >
            </div>
          <!-- 注册：昵称 -->
          </el-form-item>
          <el-form-item prop="nickName" v-if="opType == 0">
            <el-input
                size="large"
                clearable
                placeholder="请输入昵称"
                v-model.trim="formData.nickName"
                maxLength="20"
            >
              <template #prefix>
                <span class="iconfont icon-account"></span>
              </template>
            </el-input>
          </el-form-item>
          <!-- 注册/重置：密码 -->
          <el-form-item prop="registerPassword">
            <el-input
                type="password"
                size="large"
                placeholder="请输入密码"
                v-model.trim="formData.registerPassword"
                show-password
            >
              <template #prefix>
                <span class="iconfont icon-password"></span>
              </template>
            </el-input>
          </el-form-item>
          <!-- 注册/重置：再次输入密码 -->
          <el-form-item prop="reRegisterPassword">
            <el-input
                type="password"
                size="large"
                placeholder="请再次输入密码"
                v-model.trim="formData.reRegisterPassword"
                show-password
            >
              <template #prefix>
                <span class="iconfont icon-password"></span>
              </template>
            </el-input>
          </el-form-item>
        </div>

        <!--  按钮  -->
        <el-form-item>
          <el-button
              type="primary"
              class="op-btn"
              @click="doSubmit"
              size="large"
          >
            <span v-if="opType == 0">注册</span>
            <span v-if="opType == 1">登录</span>
<!--            <span v-if="opType == 2">重置密码</span>-->
          </el-button>
        </el-form-item>
        <el-form-item v-if="opType == 0"> <!--|2-->
          <a href="javascript:void(0)" class="a-link" @click="showPanel(1)">转到登录界面</a>
        </el-form-item>
      </el-form>
    </div>

  </div>
</template>

<style lang="scss" scoped>
.login-body {
  height: calc(100vh);
  background-size: cover; // 默认使用cover
  background: url("../assets/login_img.jpg");
  display: flex;
  justify-content: center; // 水平居中
  align-items: center; // 垂直居中
  .login-panel {
    width: 430px;
    margin-top: calc((100vh - 500px) / 2);
    .login-register {
      padding: 25px;
      background: #fff;
      border-radius: 5px;
      .login-title {
        text-align: center;
        font-size: 20px;
        font-weight: bold;
        margin-bottom: 20px;
      }
      .send-email-panel {
        display: flex;
        width: 100%;
        justify-content: space-between;
        .send-mail-btn {
          margin-left: 5px;
        }
      }
      .remember-panel {
        width: 80%;
      }
      .no-account {
        width: 100%;
        display: flex;
        justify-content: space-between;
      }
      .op-btn {
        width: 100%;
      }
    }
  }
}
</style>