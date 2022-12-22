<script setup>
// library
import { onBeforeMount, onMounted, reactive } from 'vue';
import { useRoute, useRouter } from 'vue-router';

// mixin
import API from '@/mixin/api.js';
import MESSAGE from '@/mixin/message';
import CONSTANT from '@/mixin/constant';
import { LOG, LOGD } from '@/mixin/log.js';
import { LOGOUT } from '@/mixin/logout.js';

// store
import { useLoginStore } from '@/store/login.js';

// env
const ENV_URL_BACKEND_HOME = import.meta.env.VITE_APP_BASE_URL_BACKEND_HOME;

// route
const route = useRoute();
const router = useRouter();

// store
const loginStore = useLoginStore();

// state
const state = reactive({
  msg: {},
});

const getInfo = async () => {
  const url = `${ENV_URL_BACKEND_HOME}/`;
  const args = {};

  const response = await API(CONSTANT.GET, url, args);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_200) {
    state.msg = response.result;
  } else if (response.code === MESSAGE.CODE_ERR_BAD_REQUEST || response.code === MESSAGE.CODE_HTTP_STATUS_419) {
    swal.fire({
      icon: 'error',
      title: '에러',
      text: MESSAGE.MESSAGE_HTTP_STATUS_419,
    });

    LOGOUT(router);
  } else {
    state.msg.userAgent = MESSAGE.MESSAGE_HTTP_STATUS_500;
    state.msg.hostname = MESSAGE.MESSAGE_HTTP_STATUS_500;
  }
};

// route from.fullPath after login by pinia
const routeLastUrl = () => {
  const nextUrl = loginStore.lastUrl;
  if (nextUrl) {
    loginStore.setLastUrl(null);
    router.push(nextUrl);
  }
};

onBeforeMount(() => {
  LOGD(route.name);
});

onMounted(() => {
  getInfo();

  routeLastUrl();
});
</script>

<template>
  <h1 class="txt-red text-center" v-if="loginStore.isLogin && loginStore.status !== '99'">
    <div>error code : {{ loginStore.status }}</div>
    <div>&nbsp;</div>
    <div>당신의 계정은 사용 허가가 필요합니다. 관리자에게 문의하세요.</div>
    <div>&nbsp;</div>
  </h1>
  <h5 class="tac">
    <img src="/assets/image/opinion.png" alt="소감" class="main_image" />
  </h5>
  <div class="dspn">
    <h5>
      <span>User-Agent : </span>
      <span> {{ state.msg.userAgent }}</span>
    </h5>
    <h5>
      <span>hostname : </span>
      <span>{{ state.msg.hostname }} </span>
    </h5>
  </div>
  <hr />
  <div class="container w-50">
    <ul class="list-group">
      <li class="list-group-item list-group-item-primary">Link</li>
      <li class="list-group-item list-group-item-info">
        <a
          href="https://bespinlearmingmate.notion.site/Top-ShareMate-Schedule-Checker-1710a43460824f67964991aaa0bc33e9"
          target="_blank"
          >Notion</a
        >
      </li>
      <li class="list-group-item list-group-item-info">
        <a href="https://github.com/KimYunBeom/schedule-checker" target="_blank">Github, Source</a>
      </li>
      <li class="list-group-item list-group-item-info">
        <a href="https://github.com/KimYunBeom/schedule-checker-k8s" target="_blank">Github, CD</a>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.main_image {
  width: 50%;
  height: 60%;
}

.dspn {
  display: none;
}

a:hover {
  font-weight: bold;
}
</style>
