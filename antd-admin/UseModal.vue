<template>
  <a-modal
    :title="title"
    width="800px"
    v-model:visible="visible"
    destroy-on-close
    :footer="null"
    :bodyStyle="{ padding: '0', overflow: 'none' }"
  >
    <Edit :editType="editType" :data="data" @cancel="close"></Edit>
  </a-modal>
</template>

<script lang="ts" setup>
  import { useModal } from '@/hooks/useModal';
  import { ref } from 'vue';
  import Edit from './Edit.vue';
  const emit = defineEmits(['ok', 'cancel']);
  const data = ref<any>();
  const { title, editType, visible, show } = useModal({ title: '公告' });
  const open = (type: ModalFormType, record: any) => {
    data.value = record;
    // console.log(data.value);
    show(type);
  };
  const close = () => {
    visible.value = false;
    editType.value !== 'detail' && emit('ok');
  };
  defineExpose({ open });
</script>
