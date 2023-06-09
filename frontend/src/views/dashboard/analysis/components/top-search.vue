<template>
  <a-card
    style="height: 100%"
    :loading="loading"
    :bordered="false"
    :title="t('dashboard.analysis.online-top-search')"
  >
    <template #extra>
      <slot name="dropdownGroup" />
    </template>
    <a-row :gutter="68" type="flex">
      <a-col :sm="12" :xs="24" style="margin-bottom: 24px">
        <number-info :gap="8" :total="numeral(12321).format('0,0')" status="up" :sub-total="17.1">
          <template #subTitle>
            <span>
              {{ t('dashboard.analysis.search-users') }}
              <a-tooltip :title="t('dashboard.analysis.introduce')">
                <info-circle-outlined style="margin-left: 8px" />
              </a-tooltip>
            </span>
          </template>
        </number-info>
      </a-col>
      <a-col :sm="12" :xs="24" style="margin-bottom: 24px">
        <number-info :total="2.7" status="down" :sub-total="26.2" :gap="8">
          <template #subTitle>
            <span>
              {{ t('dashboard.analysis.per-capita-search') }}
              <a-tooltip :title="t('dashboard.analysis.introduce')">
                <info-circle-outlined style="margin-left: 8px" />
              </a-tooltip>
            </span>
          </template>
        </number-info>
      </a-col>
    </a-row>
    <a-table
      size="small"
      :row-key="record => record.index"
      :columns="state.columns"
      :pagination="{
        style: { marginBottom: 0 },
        pageSize: 5,
      }"
    >
      <template #keywordRender="{ text }">
        <a href="/">{{ text }}</a>
      </template>

      <template #rangeRender="{ text, record }">
        <trend :flag="record.status === 1 ? 'down' : 'up'">
          <span style="margin-right: 4px">{{ text }}%</span>
        </trend>
      </template>
    </a-table>
  </a-card>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue';
import { InfoCircleOutlined } from '@ant-design/icons-vue';
import { useI18n } from 'vue-i18n';
import NumberInfo from './number-info/index.vue';
import Trend from './trend/index.vue';

import numeral from 'numeral';

export default defineComponent({
  props: {
    loading: {
      type: Boolean,
      default: () => false,
    },
  },
  setup() {
    const { t } = useI18n();

    const columns = [
      {
        title: t('dashboard.analysis.table.rank'),
        dataIndex: 'index',
        key: 'index',
      },
      {
        title: t('dashboard.analysis.table.search-keyword'),
        dataIndex: 'keyword',
        key: 'keyword',
        scopedSlots: { customRender: 'keywordRender' },
      },
      {
        title: t('dashboard.analysis.table.users'),
        dataIndex: 'count',
        key: 'count',
        sorter: (a: any, b: any) => a.count.length - b.count.length,
        class: 'alignRight',
      },
      {
        title: t('dashboard.analysis.table.weekly-range'),
        dataIndex: 'range',
        key: 'range',
        sorter: (a: Record<string, number>, b: Record<string, number>) => a.range - b.range,
        scopedSlots: { customRender: 'rangeRender' },
      },
    ];
    const state = reactive({
      columns,
    });
    return {
      t,
      numeral,

      state,
    };
  },
  components: {
    NumberInfo,
    Trend,

    // icons
    InfoCircleOutlined,
  },
});
</script>
<style lang="less" scoped>
@import '../style.less';
</style>
