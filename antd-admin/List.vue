<template>
  <div>
    <s-table-search :model="formState" @search-click="onSearchClick">
      <a-form-item label="公告标题" name="announceTitle">
        <a-input
          v-model:value="formState.announceTitle"
          :maxLength="50"
          placeholder="请输入..."
          type="text"
        />
      </a-form-item>

      <!-- 下拉选择 -->
      <a-form-item label="公告类型" name="announceType">
        <a-select
          v-model:value="formState.announceType"
          placeholder="请选择..."
          allowClear
        >
          <a-select-option
            :value="key"
            v-for="(val, key) in announceTypeOption"
            :key="key"
            >{{ val }}</a-select-option
          >
        </a-select>
      </a-form-item>
    </s-table-search>
    <s-table
      :scroll="{ x: 1000 }"
      :data="loadData"
      :columns="columns"
      @patchDelConfirm="onPatchDelConfirm"
      :row-selection="{
        selectedRowKeys: selectedRowKeys,
        onChange: onSelectChange,
      }"
      row-key="announceId"
      :show-export="false"
      :show-delete="true"
      ref="tableRef"
      v-model:loading="loading"
    >
      <template #operators>
        <a-button type="primary" @click="onOpenAnnounceInfo('add')">
          <PlusOutlined />
          新建
        </a-button>
      </template>
      <template #action="{ record }">
        <TooltipBtn title="编辑">
          <EditOutlined
            @click="onOpenAnnounceInfo('edit', record)"
            v-show="!isPublish(record)"
          />
        </TooltipBtn>
        <a-divider type="vertical" v-show="!isPublish(record)" />
        <TooltipBtn title="详情">
          <FileSearchOutlined @click="onOpenAnnounceInfo('detail', record)" />
        </TooltipBtn>
        <a-divider type="vertical" />
        <TooltipBtn title="发布">
          <UpSquareOutlined
            @click="editAnnounceInfo('2', record)"
            v-show="!isPublish(record)"
          />
        </TooltipBtn>
        <a-divider type="vertical" v-show="!isPublish(record)" />
        <TooltipBtn title="撤销">
          <Popconfirm
            title="确定撤销吗?"
            @confirm="() => editAnnounceInfo('1', record)"
          >
            <UndoOutlined v-show="isPublish(record)" />
            <!-- <UndoOutlined @click="editAnnounceInfo('1', record)" v-show="isPublish(record)" /> -->
          </Popconfirm>
        </TooltipBtn>
      </template>
    </s-table>
    <UseModal ref="editRef" @ok="onSearchClick"></UseModal>
  </div>
</template>

<script lang="ts" setup>
import { computed, reactive, ref, onMounted } from "vue";
import type {
  IColumnProps,
  IFuncDataPrams,
} from "@/components/Table/types/index";
import { getCodeMap } from "@/utils/util";
import {
  getAnnounceList,
  removeAnnounce,
  editAnnounce,
} from "@/api/appManage/announce";
import { showError, showSuccess } from "@/hooks/useMessage";
import { notification, Popconfirm } from "ant-design-vue";
import UseModal from "./UseModal.vue";
import { encryptBySGsm2 } from "@/utils/encrypt/index";

const loading = ref(false);
type statusType = "1" | "2";
const test = computed(() => {
  return function (a: any) {
    return a;
  };
});
const isPublish = computed(() => {
  return function (record: any) {
    return record.status === "2"; // 1未发布 2已发布
  };
});

const announceTypeOption = computed(() => {
  return getCodeMap("ANNOUNCE_TYPE");
});

const columns: IColumnProps[] = [
  { title: "公告id", dataIndex: "announceId", align: "center" },
  {
    title: "公告标题",
    dataIndex: "announceTitle",
    align: "center",
    ellipsis: true,
  },
  {
    title: "公告类型",
    dataIndex: "announceType",
    align: "center",
    $codeFormat: "ANNOUNCE_TYPE",
  },
  {
    title: "公告状态", // 1未发布 2已发布
    dataIndex: "status",
    align: "center",
    $codeFormat: "ANNOUNCE_STATUS",
    $codeTag: {
      "1": "red",
      "2": "green",
    },
  },
  {
    title: "阅读量",
    dataIndex: "readCount",
    align: "center",
    customRender: ({ text }) => {
      return typeof text === "number" ? text : "--";
    },
  },
  {
    title: "点赞量",
    dataIndex: "likeCount",
    align: "center",
    customRender: ({ text }) => {
      return typeof text === "number" ? text : "--";
    },
  },
  {
    title: "操作",
    dataIndex: "action",
    slots: { customRender: "action" },
    align: "center",
    fixed: "right",
  },
];
const editRef = ref();
const tableRef = ref();
const formState = reactive({
  announceTitle: "",
  announceType: "",
  status: "",
});

let selectedRowKeys = ref<any>([]);
const onSelectChange = (rowKeys: []) => {
  selectedRowKeys.value = rowKeys;
};

const loadData = async (params: IFuncDataPrams) => {
  const form = Object.assign({}, params, formState);
  const [err, data] = await getAnnounceList(form);
  if (err) return null;
  return {
    pageSize: data.size,
    pageNo: data.current,
    totalCount: data.total,
    totalPage: data.pages,
    data: data.records,
  };
};
const onSearchClick = () => {
  tableRef.value.refresh(true);
};
const onOpenAnnounceInfo = (type: TModalForm.Type, data = undefined) => {
  editRef.value?.open(type, data);
};

const editAnnounceInfo = async (status: statusType, record: any) => {
  let form = Object.assign({}, record, { status: status });
  if (form.announceContent) {
    form.announceContent = encryptBySGsm2(record.announceContent);
  }
  loading.value = true;
  const [err] = await editAnnounce(form);
  loading.value = false;
  if (err) {
    return;
  }
  notification.success({
    message: "提示",
    description: "操作成功",
  });
  record.status = status;
};

const onPatchDelConfirm = () => {
  // console.log(selectedRowKeys.value);
  if (selectedRowKeys.value.length > 0) {
    removeAnnounce(selectedRowKeys.value.join()).then((res: any[]) => {
      let [err] = res;
      if (!err) {
        onSearchClick();
        showSuccess("操作成功");
      }
    });
  } else {
    showError("暂无选中的数据");
  }
};
</script>
