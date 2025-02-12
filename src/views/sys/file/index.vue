<template>
  <div>
    <BasicTable @register="registerTable">
      <template #toolbar>
        <BasicUpload
          :maxSize="20"
          :maxNumber="10"
          :api="uploadApi"
          class="my-5"
          :emptyHidePreview="true"
          v-if="hasPermission('sys:file:upload')"
        />
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'action'">
          <TableAction
            :actions="[
              {
                icon: 'clarity:eye-line',
                tooltip: t('common.previewText'),
                onClick: handlePreview.bind(null, record),
                ifShow: hasPermission('sys:file:download'),
              },
              {
                icon: 'clarity:download-line',
                tooltip: t('common.downloadText'),
                onClick: handleDownload.bind(null, record),
                ifShow: hasPermission('sys:file:download'),
              },
              {
                icon: 'ant-design:delete-outlined',
                color: 'error',
                tooltip: t('common.delText'),
                ifShow: hasPermission('sys:file:delete'),
                popConfirm: {
                  title: t('common.delTip'),
                  confirm: handleDelete.bind(null, record),
                },
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
  </div>
</template>
<script lang="ts">
  import { defineComponent } from 'vue';

  import { BasicTable, useTable, TableAction } from '/@/components/Table';
  import { BasicUpload } from '/@/components/Upload';
  import { uploadApi, deleteFile, downloadFile, getFileList } from '/@/api/sys/file';
  import { useI18n } from '/@/hooks/web/useI18n';
  import { columns, searchFormSchema } from './file.data';
  import { downloadByData } from '/@/utils/file/download';
  import { createImgPreview } from '/@/components/Preview';
  import { usePermission } from '/@/hooks/web/usePermission';

  export default defineComponent({
    name: 'SysFile',
    components: { BasicTable, TableAction, BasicUpload },
    setup() {
      const { t } = useI18n();
      const { hasPermission } = usePermission();
      const [registerTable, { reload }] = useTable({
        api: getFileList,
        columns,
        formConfig: {
          labelWidth: 120,
          schemas: searchFormSchema,
        },
        pagination: true,
        striped: false,
        useSearchForm: true,
        showTableSetting: true,
        bordered: true,
        showIndexColumn: false,
        canResize: false,
        actionColumn: {
          width: 100,
          title: t('common.operateText'),
          dataIndex: 'action',
        },
      });

      function handleDelete(record: Recordable) {
        deleteFile(record.id).then(() => reload());
      }

      function handlePreview(record: Recordable) {
        createImgPreview({ imageList: [record.url] });
      }

      function handleDownload(record: Recordable) {
        downloadFile(record.id).then((res) => {
          downloadByData(
            res.data,
            decodeURI(
              res?.headers['content-disposition']?.substring('attachment;filename='.length),
            ),
          );
        });
      }

      function handleSuccess() {
        reload();
      }

      return {
        t,
        hasPermission,
        uploadApi,
        registerTable,
        handleDownload,
        handlePreview,
        handleDelete,
        handleSuccess,
      };
    },
  });
</script>
