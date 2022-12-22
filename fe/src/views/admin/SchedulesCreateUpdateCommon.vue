<script setup>
// library
import { onBeforeMount, onMounted, reactive } from 'vue';
import { useRoute, useRouter } from 'vue-router';

// custom
import CustomPageHeader from '@/components/CustomPageHeader.vue';
import CustomActionButton from '@/components/CustomActionButton.vue';

// data
import ScheduleStatus from '@/flag/scheduleStatus.json';

// mixin
import API from '@/mixin/api.js';
import MESSAGE from '@/mixin/message';
import CONSTANT from '@/mixin/constant';
import { LOG, LOGD } from '@/mixin/log.js';
import { NOW_DATE, NOW_TIME } from '@/mixin/datetime.js';

// store
import { useLoginStore } from '@/store/login.js';

// swal
import swal from 'sweetalert2';
import { LOGOUT } from '../../mixin/logout';

// env
const ENV_URL_BACKEND_HOME = import.meta.env.VITE_APP_BASE_URL_BACKEND_HOME;

// props
const props = defineProps({
  mode: {
    type: String,
  },
});

// route
const route = useRoute();
const router = useRouter();

// store
const loginStore = useLoginStore();

// state
const state = reactive({
  form: {
    title: '',
    content: '',
    status: '',
    regist_date: '',
    regist_time: '',
    limit_date: '',
    limit_time: '',
    regist_user: loginStore.userInfo.name + '/' + loginStore.userInfo.email,
    user_code: loginStore.userInfo.id,
    user_email: loginStore.userInfo.email,
    target_users: [],
    checked_users: [],
    all_user_origin: [],
  },
  errors: [],
  mode: '',
});

const goBack = () => {
  router.go(-1);
};

const goList = () => {
  router.push({ name: 'AdminSchedules' });
};

const setScheduleItem = () => {
  state.scheduleInfoOrigin = JSON.parse(route.query.scheduleInfo);
  state.form.schedule_code = state.scheduleInfoOrigin.schedule_code;
  state.form.regist_date = state.scheduleInfoOrigin.regist_date?.split(' ')[0];
  state.form.regist_time = state.scheduleInfoOrigin.regist_date?.split(' ')[1];
  state.form.limit_date = state.scheduleInfoOrigin.limit_date?.split(' ')[0];
  state.form.limit_time = state.scheduleInfoOrigin.limit_date?.split(' ')[1];
  state.form.status = state.scheduleInfoOrigin.status;
  state.form.completed_user = state.scheduleInfoOrigin.completed_user;
  state.form.title = state.scheduleInfoOrigin.title;
  state.form.content = state.scheduleInfoOrigin.content.replaceAll('<br>', '\n');
  state.form.all_user_origin = state.scheduleInfoOrigin.all_user_origin;
};

const setPrevChkUsers = () => {
  const chkUsers = document.querySelectorAll('[name=chkUsers]');
  const originUsers = JSON.parse(JSON.stringify(state.form.all_user_origin));

  chkUsers.forEach((chkUser, idx) => {
    if (idx !== 0) {
      const isEqual = originUsers.includes(chkUser.value);
      if (isEqual) {
        chkUser.checked = true;
        state.form.checked_users.push(chkUser.value);
      }
    }
  });

  if (state.form.checked_users.length === chkUsers.length - 1) {
    document.querySelector('#chkUsersAll').checked = true;
  } else {
    document.querySelector('#chkUsersAll').checked = false;
  }
};

const getMembers = async () => {
  const url = `${ENV_URL_BACKEND_HOME}/members`;
  const args = {};
  const header = {
    authorization: loginStore.accessToken,
  };

  const response = await API(CONSTANT.GET, url, args, header);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_200) {
    state.form.target_users = response.result.dataList;
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

const changeCheckedUsers = () => {
  state.form.checked_users = [];
  const chkUsers = document.querySelectorAll('[name=chkUsers]');
  chkUsers.forEach((chkUser) => {
    if (chkUser?.id !== 'chkUsersAll') {
      if (chkUser.checked) {
        state.form.checked_users.push(chkUser.value);
      }
    }
  });

  if (state.form.checked_users.length === chkUsers.length - 1) {
    document.querySelector('#chkUsersAll').checked = true;
  } else {
    document.querySelector('#chkUsersAll').checked = false;
  }
};

const changeCheckAll = () => {
  const chkUsers = document.querySelectorAll('[name=chkUsers]');

  // check or uncheck all of checkboxes
  let isCheckedAll = false;
  if (chkUsers[0].checked) {
    isCheckedAll = true;
  }
  chkUsers.forEach((chkUser) => {
    chkUser.checked = isCheckedAll;
  });

  // add or remove checkboxes to array
  if (chkUsers[0].checked) {
    state.form.checked_users = state.form.target_users.map((user) => user.user_email);
  } else {
    state.form.checked_users = [];
  }
};

const createSchedule = async () => {
  const url = `${ENV_URL_BACKEND_HOME}/schedules/create`;
  const args = state.form;
  const header = {
    authorization: loginStore.accessToken,
  };

  const response = await API(CONSTANT.POST, url, args, header);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_201) {
    swal.fire({
      icon: 'info',
      title: '스케줄 생성 완료',
      text: MESSAGE.MESSAGE_HTTP_STATUS_201,
    });

    goList();
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

const updateSchedule = async () => {
  const url = `${ENV_URL_BACKEND_HOME}/schedules/update`;
  const args = state.form;
  const header = {
    authorization: loginStore.accessToken,
  };

  const response = await API(CONSTANT.PATCH, url, args, header);

  if (response.code === MESSAGE.CODE_HTTP_STATUS_201) {
    swal.fire({
      icon: 'info',
      title: '스케줄 수정 완료',
      text: MESSAGE.MESSAGE_HTTP_STATUS_201,
    });

    goList();
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

const validation = {
  validateTitle: (limitLength = 30, title) => {
    let isValidate = true;
    if (!title) {
      isValidate = false;
      state.errors.push(`제목은 반드시 입력해야 합니다.`);
    }
    if (title?.length > limitLength) {
      isValidate = false;
      state.errors.push(`제목은 ${limitLength} 글자로 제한됩니다.`);
    }
    return isValidate;
  },
  validateContent: (limitLength = 4000, content) => {
    let isValidate = true;
    if (!content) {
      isValidate = false;
      state.errors.push(`내용은 반드시 입력해야 합니다.`);
    }
    if (content?.length > limitLength) {
      isValidate = false;
      state.errors.push(`내용은 ${limitLength} 글자로 제한됩니다.`);
    }
    return isValidate;
  },
  validateAll: () => {
    state.errors = [];
    if (!validation.validateTitle(30, state.form.title)) {
      return;
    }
    if (!validation.validateContent(4000, state.form.content)) {
      return;
    }
  },
};

const create = (paramForParent) => {
  validation.validateAll();

  if (state.errors.length === 0) {
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

          createSchedule();
        }

        swal.fire(resultMessage, '', confirmText);
      });
  }
};

const update = (paramForParent) => {
  validation.validateAll();

  if (state.errors.length === 0) {
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

          updateSchedule();
        }

        swal.fire(resultMessage, '', confirmText);
      });
  }
};

onBeforeMount(() => {
  LOGD(route.name);
});

onMounted(async () => {
  state.mode = props.mode;

  await getMembers();

  if (state.mode === 'CREATE') {
    state.form.status = '1';
    state.form.regist_date = NOW_DATE();
    state.form.regist_time = NOW_TIME();
    state.form.limit_date = NOW_DATE();
    state.form.limit_time = '23:59:59';
  } else {
    await setScheduleItem();

    await setPrevChkUsers();
  }
});
</script>

<template>
  <CustomPageHeader text="스케줄 생성" option3="txt-admin" v-if="state.mode === 'CREATE'" />
  <CustomPageHeader text="스케줄 수정" option3="txt-admin" v-else />
  <div class="container">
    <div class="row align-items-center py-1">
      <div class="col-md-10 mx-auto col-lg-10">
        <form class="p-4 p-md-4 border rounded-3 bg-light" @input="validation.validateAll">
          <h4 class="mb-3">기본 정보</h4>
          <div>
            <div class="form-floating mb-3">
              <input
                type="text"
                class="form-control"
                id="regist_user"
                placeholder="등록자"
                readonly
                tabindex="-1"
                v-model="state.form.regist_user"
              />
              <label for="regist_user">등록자</label>
            </div>
            <div class="d-flex justify-content-start">
              <div class="form-floating mb-3 w-50 me-2">
                <input type="date" class="form-control" id="regist_date" v-model="state.form.regist_date" />
                <label for="regist_date">등록일자</label>
              </div>
              <div class="form-floating mb-3 w-50">
                <input type="time" class="form-control" id="regist_time" v-model="state.form.regist_time" />
                <label for="regist_time">등록시간</label>
              </div>
            </div>
            <div class="d-flex justify-content-start">
              <div class="form-floating mb-3 w-50 me-2">
                <input type="date" class="form-control" id="limit_date" v-model="state.form.limit_date" />
                <label for="limit_date">마감일자</label>
              </div>
              <div class="form-floating mb-3 w-50">
                <input type="time" class="form-control" id="limit_time" v-model="state.form.limit_time" />
                <label for="limit_time">등록시간</label>
              </div>
            </div>
          </div>
          <div v-if="state.errors.length">
            <ul class="errorMessage">
              <li v-for="error in state.errors">{{ error }}</li>
            </ul>
          </div>
          <div class="form-floating mb-3">
            <input type="text" class="form-control" id="title" placeholder="제목" v-model="state.form.title" />
            <label for="title">제목</label>
          </div>
          <div class="form-floating mb-3">
            <textarea
              class="form-control h-r10"
              id="content"
              v-model="state.form.content"
              placeholder="내용"
            ></textarea>
            <label for="content">내용</label>
          </div>
          <h4 class="mb-3">상태</h4>
          <div class="d-flex justify-content-start">
            <div class="form-floating mb-3 me-2 w-50">
              <div class="form-check form-check-inline" v-for="column in ScheduleStatus">
                <input
                  class="form-check-input"
                  type="radio"
                  name="statusOptions"
                  :id="column.val"
                  :value="column.key"
                  v-model="state.form.status"
                />
                <label class="form-check-label" :for="column.val">{{ column.val }}</label>
              </div>
            </div>
          </div>
          <h4 class="mb-3">대상 선택</h4>
          <div class="input-group mb-1">
            <div class="input-group-prepend">
              <div class="input-group-text p-08">
                <input
                  type="checkbox"
                  id="chkUsersAll"
                  name="chkUsers"
                  aria-label="Checkbox"
                  @change="changeCheckAll()"
                />
              </div>
            </div>
            <input type="text" class="form-control text-primary fw-bold" aria-label="Text" value="전체" tabindex="-1" />
          </div>
          <div class="input-group mb-1" v-for="(user, index) in state.form.target_users">
            <div class="input-group-prepend">
              <div class="input-group-text p-08">
                <input
                  type="checkbox"
                  name="chkUsers"
                  aria-label="Checkbox"
                  :value="user.user_email"
                  @change="changeCheckedUsers()"
                />
              </div>
            </div>
            <input
              type="text"
              name="inpUsers"
              class="form-control"
              aria-label="Text"
              readonly
              tabindex="-1"
              v-model="user.user_name"
            />
          </div>
          <hr />
          <CustomActionButton
            text="생성 완료"
            command="create"
            @buttonClicked="create"
            option1="btn-admin"
            v-if="state.mode === 'CREATE'"
          />
          <CustomActionButton text="수정 완료" command="update" @buttonClicked="update" option1="btn-admin" v-else />
          <CustomActionButton text="이전" @click="goBack" v-if="state.mode === 'UPDATE'" />
          <CustomActionButton text="목록" @click="goList" />
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.errorMessage {
  color: #842029;
  background-color: #f8d7da;
}
.errorMessage li {
  list-style-type: square;
}
</style>
