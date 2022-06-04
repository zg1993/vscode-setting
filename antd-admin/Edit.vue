<template>
  <FormContainer>
    <a-form
      :labelCol="{ style: { width: '100px' } }"
      ref="formRef"
      :model="formState"
      :rules="rules"
      v-widget="editType === 'detail'"
    >
      <a-form-item label="公告标题" name="announceTitle">
        <a-input v-model:value="formState.announceTitle" :maxLength="50" placeholder="请输入..." type="text" />
      </a-form-item>

      <a-form-item label="公告类型" name="announceType">
        <a-select v-model:value="formState.announceType" placeholder="请选择..." allowClear>
          <a-select-option :value="key" v-for="(val, key) in announceTypeOptions" :key="key">{{ val }}</a-select-option>
        </a-select>
      </a-form-item>

      <a-form-item label="图片" name="announcePic">
        <UploadFile
          ref="uploadRef"
          :disabled="editType === 'detail'"
          v-model:url="formState.announcePic"
          :amount="1"
          list-type="picture-card"
          :limit-size="2000"
        ></UploadFile>
      </a-form-item>

      <!-- <a-form-item label="开屏时间(s)" name="extension">
        <a-input v-model:value="formState.extension" :maxLength="50" placeholder="请输入..." type="text" />
      </a-form-item>

      <a-form-item label="跳转路径" name="jumpUrl">
        <a-input v-model:value="formState.jumpUrl" :maxLength="50" placeholder="请输入..." type="text" />
      </a-form-item> -->

      <a-form-item label="公告内容" name="announceContent">
        <WangEditor v-model:value="formState.announceContent" />
      </a-form-item>
    </a-form>
  </FormContainer>
  <FormButtonContainer>
    <a-button key="submit" type="primary" :loading="loading" @click="confirmOk" v-show="props.editType !== 'detail'">
      确定
    </a-button>
    <a-button key="back" @click="close">取消</a-button>
  </FormButtonContainer>
</template>
<script lang="ts" setup>
  import { reactive, ref, computed, onBeforeMount, onMounted, createVNode } from 'vue';
  import { getValidationRules, getCodeMap } from '@/utils/util';
  import { notification } from 'ant-design-vue';
  import UploadFile from '@/components/UploadFile/index.vue';
  import WangEditor from '@/components/WangEditor/index.vue';
  import { editAnnounce, addAnnounce } from '@/api/appManage/announce';
  import { encryptBySGsm2 } from '@/utils/encrypt/index';
  import { Modal } from 'ant-design-vue';
  import { ExclamationCircleOutlined } from '@ant-design/icons-vue';

  const announceTypeOptions = computed(() => {
    return getCodeMap('ANNOUNCE_TYPE');
  });

  const formRef = ref();
  const uploadRef = ref();
  const formState = reactive({
    announceId: '',
    announceContent: '',
    announcePic: '',
    announceTitle: '',
    announceType: '',
    // extension: '',
    // jumpUrl: '',
  });
  const loading = ref(false);
  const props = defineProps({
    editType: {
      type: String,
      default: 'add',
    },
    data: {
      type: Object,
      default: function () {
        return {};
      },
    },
  });
  onBeforeMount(() => {
    console.log('onBeforeMount');
  });
  onMounted(() => {
    console.log('onMounted');
    if (props.data) {
      Object.keys(formState).forEach((key) => {
        formState[key as keyof typeof formState] = props.data[key];
      });
      props.data.announcePic &&
        uploadRef.value.addDefaultFileList([
          {
            uid: '1',
            type: 'image',
            // name: 'file',
            url: props.data.announcePic,
          },
        ]);
    }
  });
  const emit = defineEmits(['ok', 'cancel']);

  const rules = getValidationRules('');

  const close = () => emit('cancel');

  const confirmOk = () => {
    if (props.editType == 'edit') {
      Modal.confirm({
        title: () => '确认',
        icon: () => createVNode(ExclamationCircleOutlined),
        content: () => '是否修改公告内容',
        onOk: handleOk,
      });
    } else handleOk();
  };
  // 提交
  const handleOk = () => {
    formRef.value.validate().then(async () => {
      loading.value = true;
      let method = props.editType === 'add' ? addAnnounce : editAnnounce;
      let formData: any = Object.assign({}, formState);
      // console.log(formData);
      if (formData.announceContent) {
        formData.announceContent = encryptBySGsm2(formData.announceContent);
      }
      // console.log(formData);
      const [err] = await method(formData);
      loading.value = false;
      if (err) return;
      notification.success({
        message: '提示',
        description: '操作成功',
      });
      close();
      // emit('cancel');
    });
  };
</script>
<style lang="less">
  .ant-calendar-picker {
    width: 100%;
  }
</style>
