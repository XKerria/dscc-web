<template>
  <div class="admin-list">
    <div class="tbar">
      <a-input-search
        v-model:value="search"
        style="width: 400px"
        enter-button
        placeholder="关键字（姓名、手机号）"
        @search="onSearch"
      />
      <a-button type="primary" @click="$router.push({ name: 'admin-add' })">
        <ui-icon name="md-add-r" />
        <span>添加</span>
      </a-button>
    </div>

    <div class="table">
      <antd-table ref="table" :search="search" :columns="columns" :request="request" />
    </div>
  </div>
</template>

<script setup lang="jsx">
import { computed, onMounted, reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import AntdTable from '@/components/antd/Table.vue'
import adminApi from '@/api/admin'
import { message } from 'ant-design-vue'

const store = useStore()
const router = useRouter()
const table = ref(null)

const admin = computed(() => store?.state?.auth?.admin)

const columns = [
  { title: '姓名', dataIndex: 'name', width: 150, sorter: true, encoding: 'gbk' },
  { title: '手机号', dataIndex: 'phone', sorter: true },
  {
    dataIndex: 'id',
    align: 'right',
    customRender: ({ record }) => {
      return (
        <a-space size='middle'>
          <a-tooltip title='编辑'>
            <a-button
              type='primary'
              shape='circle'
              size='small'
              onClick={() => router.push({ name: 'admin-edit', params: { id: record.id } })}
            >
              <ui-icon name='md-edit-fr' />
            </a-button>
          </a-tooltip>
          {admin.value.id !== record.id && (
            <a-popconfirm title='该操作将无法恢复，确认要删除？' onConfirm={() => onDeleteConfirm(record)}>
              <a-tooltip title='删除'>
                <a-button type='primary' danger shape='circle' size='small'>
                  <ui-icon name='md-delete-fr' />
                </a-button>
              </a-tooltip>
            </a-popconfirm>
          )}
        </a-space>
      )
    }
  }
]

const search = ref('')

const request = (params) => {
  return adminApi.index({
    like: `name:${search.value}|phone:${search.value}`,
    ...params
  })
}

const state = reactive({ columns, search, request })

const onDeleteConfirm = (item) => {
  adminApi.destroy(item.id).then(({ data }) => {
    if (data === 1) {
      table.value.refresh()
      message.success('删除成功')
    } else {
      message.success('删除失败')
    }
  })
}

const onSearch = () => {
  table.value.refresh()
}
</script>

<style lang="less" scoped>
.admin-list {
  height: 100%;
  overflow: hidden;
  padding: 16px;
  display: flex;
  flex-direction: column;

  .tbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 8px;
  }

  .table {
    flex: 1;
    overflow: hidden;
  }
}
</style>
