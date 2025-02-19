<template>
  <n-card :bordered="false" class="proCard">
    <BasicTable
      title="表格列表"
      titleTooltip="这是一个提示"
      :columns="columns"
      :request="loadDataTable"
      :row-key="(row) => row.id"
      ref="actionRef"
      :actionColumn="actionColumn"
      @update:checked-row-keys="onCheckedRow"
    >
      <template #toolbar>
        <n-button type="primary" @click="reloadTable">刷新数据</n-button>
      </template>
    </BasicTable>
  </n-card>
</template>

<script lang="ts">
  import { defineComponent, reactive, toRefs, ref, h } from 'vue';
  import { BasicTable, TableAction } from '@/components/Table';
  import { getTableList } from '@/api/table/list';
  import { columns } from './basicColumns';
  import { useDialog, useMessage } from 'naive-ui';

  export default defineComponent({
    components: { BasicTable },
    setup() {
      const message = useMessage();
      const dialog = useDialog();
      const actionRef = ref();
      const state = reactive({
        params: {
          pageSize: 5,
          name: 'xiaoMa',
        },
        actionColumn: {
          width: 150,
          title: '操作',
          dataIndex: 'action',
          fixed: 'right',
          align: 'center',
          render(record) {
            return h(TableAction, {
              style: 'button',
              actions: createActions(record),
            });
          },
        },
      });

      function createActions(record) {
        return [
          {
            label: '删除',
            icon: 'ic:outline-delete-outline',
            onClick: handleDelete.bind(null, record),
            // 根据业务控制是否显示 isShow 和 auth 是并且关系
            ifShow: () => {
              return true;
            },
            // 根据权限控制是否显示: 有权限，会显示，支持多个
            auth: ['basic_list'],
          },
          {
            label: '编辑',
            onClick: handleEdit.bind(null, record),
            ifShow: () => {
              return true;
            },
            auth: ['basic_list'],
          },
        ];
      }

      const loadDataTable = async (params) => {
        const data = await getTableList(params);
        return data;
      };

      function onCheckedRow(rowKeys) {
        console.log(rowKeys);
      }

      function reloadTable() {
        actionRef.value.reload();
      }

      function handleDelete(record) {
        console.log(record);
        dialog.info({
          title: '提示',
          content: `您想删除${record.name}`,
          positiveText: '确定',
          negativeText: '取消',
          onPositiveClick: () => {
            message.success('删除成功');
          },
          onNegativeClick: () => {},
        });
      }

      function handleEdit(record) {
        console.log(record);
        message.success('您点击了编辑按钮');
      }

      return {
        ...toRefs(state),
        columns,
        actionRef,
        loadDataTable,
        onCheckedRow,
        reloadTable,
      };
    },
  });
</script>

<style lang="less" scoped></style>
