<template>
  <page-container>
    <div class="ant-pro-table">
      <div class="ant-pro-table-search">
        <a-form layout="horizontal">
          <a-row :gutter="16" type="flex" justify="start">
            <a-col :xs="24" :sm="24" :md="12">
              <a-form-item label="权限名称">
                <a-input placeholder="please enter" />
              </a-form-item>
            </a-col>
            <a-col :xs="24" :sm="24" :md="12">
              <a-form-item label="描述">
                <a-input placeholder="please enter" />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <a-card :body-style="{ padding: 0 }" ref="elRef">
        <div class="ant-pro-table-list-toolbar">
          <div class="ant-pro-table-list-toolbar-container">
            <div class="ant-pro-table-list-toolbar-left">
              <div class="ant-pro-table-list-toolbar-title">权限列表</div>
            </div>
            <div class="ant-pro-table-list-toolbar-right">
              <a-space align="center">
                <a-button
                  type="primary"
                  @click="
                    () => {
                      editModal.model = null;
                      editModal.visible = true;
                    }
                  "
                >
                  <plus-outlined />
                  新增权限
                </a-button>
              </a-space>
              <div class="ant-pro-table-list-toolbar-divider">
                <a-divider type="vertical" />
              </div>
              <div class="ant-pro-table-list-toolbar-setting-item">
                <a-tooltip title="刷新">
                  <reload-outlined @click="() => handleTableChange({ current: 1, pageSize: 10 })" />
                </a-tooltip>
              </div>
              <div class="ant-pro-table-list-toolbar-setting-item">
                <a-tooltip title="密度">
                  <a-dropdown :trigger="['click']" placement="bottomRight">
                    <column-height-outlined />
                    <template #overlay>
                      <a-menu
                        style="width: 80px"
                        :selected-keys="[state.tableSize]"
                        @click="
                          ({ key }) => {
                            state.tableSize = key;
                          }
                        "
                      >
                        <a-menu-item key="default">
                          <a href="javascript:;">默认</a>
                        </a-menu-item>
                        <a-menu-item key="middle">
                          <a href="javascript:;">中等</a>
                        </a-menu-item>
                        <a-menu-item key="small">
                          <a href="javascript:;">紧凑</a>
                        </a-menu-item>
                      </a-menu>
                    </template>
                  </a-dropdown>
                </a-tooltip>
              </div>
              <div class="ant-pro-table-list-toolbar-setting-item">
                <a-popover
                  placement="bottomRight"
                  arrowPointAtCenter
                  trigger="click"
                  overlayClassName="ant-pro-table-column-setting-overlay"
                >
                  <template #title>
                    <div class="ant-pro-table-column-setting-title">
                      <a-checkbox
                        v-model:checked="columnState.checkAll"
                        :indeterminate="columnState.indeterminate"
                        @change="handleColumnAllClick"
                      >
                        列展示 / 排序
                      </a-checkbox>
                      <a @click="reset">重置</a>
                    </div>
                  </template>
                  <template #content>
                    <span class="ant-pro-table-column-setting-list">
                      <drag-container
                        lockAxis="y"
                        dragClass="ant-pro-table-drag-ghost"
                        dropClass="ant-pro-table-drop-ghost"
                        @drop="({ removedIndex, addedIndex }) => move(removedIndex, addedIndex)"
                      >
                        <draggable :key="column.key" v-for="column in dynamicColumnItems">
                          <div class="ant-pro-table-column-setting-list-item">
                            <drag-icon />
                            <a-checkbox
                              :checked="column.checked"
                              @change="handleColumnChange($event, column)"
                            >
                              {{ column.label }}
                            </a-checkbox>
                          </div>
                        </draggable>
                      </drag-container>
                    </span>
                  </template>
                  <a-tooltip title="列设置">
                    <setting-outlined />
                  </a-tooltip>
                </a-popover>
              </div>
              <div class="ant-pro-table-list-toolbar-setting-item">
                <a-tooltip :title="screenState ? '退出全屏' : '全屏'">
                  <fullscreen-outlined v-if="!screenState" @click="setFull" />
                  <fullscreen-exit-outlined v-else @click="exitFull" />
                </a-tooltip>
              </div>
            </div>
          </div>
        </div>
        <a-table
          row-key="id"
          :size="state.tableSize"
          :loading="state.loading"
          :columns="dynamicColumns"
          :data-source="state.dataSource"
          :pagination="{
            current: state.current,
            pageSize: state.pageSize,
            total: state.total,
          }"
          @change="handleTableChange"
        >
          <template #actions="{ record }">
            <a-tag v-for="action in record.actions" :key="action">{{ action }}</a-tag>
          </template>
          <template #action="{ text, record }">
            <a :title="text" @click="() => handleOpenEdit(record)">编辑</a>
          </template>
        </a-table>
      </a-card>
    </div>

    <permission-modal
      :model="editModal.model"
      :visible="editModal.visible"
      @cancel="
        () => {
          editModal.visible = false;
        }
      "
      @ok="
        () => {
          editModal.visible = false;
          reload();
        }
      "
    />
  </page-container>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue';
import {
  PlusOutlined,
  ReloadOutlined,
  ColumnHeightOutlined,
  SettingOutlined,
  FullscreenOutlined,
  FullscreenExitOutlined,
} from '@ant-design/icons-vue';
import { Container as DragContainer, Draggable } from '@/components/draggable';
import type { TableColumn } from '@/typing';
import { useFullscreen } from '@/utils/hooks/useFullscreen';
import { useTableDynamicColumns } from '@/utils/hooks/useTableColumn';
import DragIcon from '@/components/table/drag-icon.vue';

import PermissionModal from './permission-modal.vue';

const baseColumns: TableColumn[] = [
  {
    title: '#',
    dataIndex: 'id',
  },
  {
    title: '权限名称',
    dataIndex: 'name',
  },
  {
    title: 'Action',
    dataIndex: 'actions',
    slots: { customRender: 'actions' },
  },
  {
    title: '操作',
    dataIndex: 'action',
    slots: { customRender: 'action' },
  },
];
export default defineComponent({
  name: 'PermissionList',
  setup() {
    const {
      state: columnState,
      dynamicColumns,
      dynamicColumnItems,
      handleColumnAllClick,
      handleColumnChange,
      reset,
      move,
    } = useTableDynamicColumns(baseColumns, true);
    const [elRef, screenState, { setFull, exitFull }] = useFullscreen();


    // edit
    const editModal = reactive({
      visible: false,
      model: null,
    });
    const handleOpenEdit = (record: any) => {
      editModal.visible = true;
      editModal.model = {
        ...record,
      };
    };

    return {
      columnState,
      dynamicColumns,
      dynamicColumnItems,

      // fullscreen
      elRef,
      screenState,
      setFull,
      exitFull,

      handleColumnChange,
      handleColumnAllClick,
      reset,
      move,

      // edit
      editModal,
      handleOpenEdit,
    };
  },
  components: {
    DragIcon,
    PlusOutlined,
    ReloadOutlined,
    ColumnHeightOutlined,
    SettingOutlined,
    FullscreenOutlined,
    FullscreenExitOutlined,
    Draggable,
    DragContainer,
    PermissionModal,
  },
});
</script>
