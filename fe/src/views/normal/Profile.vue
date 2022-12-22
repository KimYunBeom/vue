<script setup>
// library
import { onBeforeMount, onMounted, reactive } from 'vue';
import { useRoute, useRouter } from 'vue-router';

// custom
import CustomPageHeader from '@/components/CustomPageHeader.vue';
import CustomActionButton from '@/components/CustomActionButton.vue';

// mixin
import API from '@/mixin/api.js';
import MESSAGE from '@/mixin/message';
import CONSTANT from '@/mixin/constant';
import { LOG, LOGD } from '@/mixin/log.js';
import { LOGOUT } from '@/mixin/logout.js';

// store
import { useLoginStore } from '@/store/login.js';

// swal
import swal from 'sweetalert2';

// env
const ENV_URL_BACKEND_HOME = import.meta.env.VITE_APP_BASE_URL_BACKEND_HOME;

// route
const route = useRoute();
const router = useRouter();

// store
const loginStore = useLoginStore();

// state
const state = reactive({
  userInfo: {
    user_code: null,
    user_email: null,
    user_name: null,
    grade: null,
    status: null,
    regist_date: null,
    grade_val: null,
    status_val: null,
  },
});

const getUserInfo = async () => {
  const {
    userInfo: { email: user_email },
  } = loginStore;

  const url = `${ENV_URL_BACKEND_HOME}/members/${user_email}`;
  const args = {};
  const header = {
    authorization: loginStore.accessToken,
  };

  const response = await API(CONSTANT.GET, url, args, header);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_200) {
    state.userInfo = response.result;
  } else if (response.code === MESSAGE.CODE_ERR_BAD_REQUEST || response.code === MESSAGE.CODE_HTTP_STATUS_419) {
    swal.fire({
      icon: 'error',
      title: '에러',
      text: MESSAGE.MESSAGE_HTTP_STATUS_419,
    });

    LOGOUT(router);
  } else {
    LOGD(response.code);
  }
};

const deleteMember = async () => {
  const {
    userInfo: { email: user_email },
  } = loginStore;

  const url = `${ENV_URL_BACKEND_HOME}/members/${user_email}`;
  const args = {};
  const header = {
    authorization: loginStore.accessToken,
  };

  const response = await API(CONSTANT.DELETE, url, args, header);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_204) {
    swal.fire({
      icon: 'info',
      title: '탈퇴 완료',
      text: MESSAGE.MESSAGE_HTTP_STATUS_200,
    });

    LOGOUT(router);
  } else if (response.code === MESSAGE.CODE_ERR_BAD_REQUEST || response.code === MESSAGE.CODE_HTTP_STATUS_419) {
    swal.fire({
      icon: 'error',
      title: '에러',
      text: MESSAGE.MESSAGE_HTTP_STATUS_419,
    });

    LOGOUT(router);
  } else {
    LOGD(response.code);
  }
};

const leave = (paramForParent) => {
  const { title, showDenyButton, confirmButtonText, denyButtonText, resultMessageY, resultMessageN } = paramForParent;

  swal
    .fire({
      title,
      showDenyButton,
      confirmButtonText,
      denyButtonText,
    })
    .then((result) => {
      let resultMessage = resultMessageN;
      let confirmText = 'info';

      if (result.isConfirmed) {
        resultMessage = resultMessageY;
        confirmText = 'success';

        deleteMember();
      }

      swal.fire(resultMessage, '', confirmText);
    });
};

onBeforeMount(() => {
  LOGD(route.name);
});

onMounted(() => {
  getUserInfo();
});
</script>

<template>
  <CustomPageHeader text="프로필" />
  <div class="container">
    <div class="row align-items-center py-1">
      <div class="col-md-10 mx-auto col-lg-10">
        <div class="p-4 p-md-4 border rounded-3 bg-light">
          <h4 class="mb-3">프로필</h4>
          <div class="form-floating mb-3">
            <h5 class="text-muted">이름</h5>
            <div class="mb-3">{{ state.userInfo.user_name }}</div>
            <h5 class="text-muted">이메일</h5>
            <div class="mb-3">{{ state.userInfo.user_email }}</div>
            <h5 class="text-muted">권한</h5>
            <div class="mb-3">{{ state.userInfo.grade_val }}</div>
            <h5 class="text-muted">상태</h5>
            <div class="mb-3">{{ state.userInfo.status_val }}</div>
            <h5 class="text-muted">가입일시</h5>
            <div class="mb-3">{{ state.userInfo.regist_date }}</div>
          </div>
          <hr />
          <CustomActionButton text="회원 탈퇴" command="memberLeave" option1="btn-danger" @buttonClicked="leave" />
        </div>
      </div>
    </div>
  </div>
</template>
