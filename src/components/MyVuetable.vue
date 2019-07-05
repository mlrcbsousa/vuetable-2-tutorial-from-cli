// MyVuetable.vue

<template>
  <div class="ui container">
    <filter-bar></filter-bar>

    <!-- <div class="vuetable-pagination ui basic segment grid">
      <vuetable-pagination-info ref="paginationInfoTop"></vuetable-pagination-info>

      <vuetable-pagination ref="paginationTop"
        @vuetable-pagination:change-page="onChangePage"
      ></vuetable-pagination>
    </div> -->

    <vuetable ref="vuetable"
      :api-url="apiUrl"
      :fields="fields"
      :per-page="10"
      :multi-sort="true"
      :sort-order="sortOrder"
      :appendParams="appendParams"
      detail-row-component="detailRowComponent"
      pagination-path=""
      multi-sort-key="ctrl"
      @vuetable:pagination-data="onPaginationData"
      @vuetable:cell-clicked="onCellClicked"
    >

      <template slot="actions" slot-scope="props">
        <div class="custom-actions">
          <button
            class="ui basic button"
            @click="onAction('view-item', props.rowData, props.rowIndex)"
          >
            <i class="zoom icon"></i>
          </button>
          <button
            class="ui basic button"
            @click="onAction('edit-item', props.rowData, props.rowIndex)"
          >
            <i class="edit icon"></i>
          </button>
          <button
            class="ui basic button"
            @click="onAction('delete-item', props.rowData, props.rowIndex)"
          >
            <i class="delete icon"></i>
          </button>
        </div>
      </template>

    </vuetable>
      <!-- :css="css" -->

    <div class="vuetable-pagination ui basic segment grid">
      <vuetable-pagination-info ref="paginationInfo"></vuetable-pagination-info>

      <vuetable-pagination ref="pagination"
        @vuetable-pagination:change-page="onChangePage"
      ></vuetable-pagination>
    </div>
  </div>
</template>

<script>
import Vuetable from 'vuetable-2/src/components/Vuetable';
import VuetablePagination from 'vuetable-2/src/components/VuetablePagination';
import VuetablePaginationInfo from 'vuetable-2/src/components/VuetablePaginationInfo';
import accounting from 'accounting';
import moment from 'moment';
import Vue from 'vue';
import VueEvents from 'vue-events';
import CustomActions from './CustomActions';
import FilterBar from './FilterBar';

Vue.use(VueEvents);
Vue.component('filter-bar', FilterBar);
Vue.component('custom-actions', CustomActions);

export default {
  name: 'my-vuetable',
  components: {
    Vuetable,
    VuetablePagination,
    VuetablePaginationInfo,
  },
  props: {
    apiUrl: { type: String, required: true },
    fields: { type: Array, required: true },
    sortOrder: { type: Array, default() { return []; } },
    appendParams: { type: Object, default() { return {}; } },
    detailRowComponent: { type: String },
  },
  data() {
    return {};
  },
  mounted() {
    this.$events.$on('filter-set', eventData => this.onFilterSet(eventData));
    this.$events.$on('filter-reset', () => this.onFilterReset());
  },
  methods: {
    allcap(value) {
      return value.toUpperCase();
    },
    genderLabel(value) {
      return value === 'M'
        ? '<span class="ui teal label"><i class="large man icon"></i>Male</span>'
        : '<span class="ui pink label"><i class="large woman icon"></i>Female</span>';
    },
    formatNumber(value) {
      return accounting.formatNumber(value, 2);
    },
    formatDate(value, fmt = 'DD MM YYYY') {
      return (value == null)
        ? ''
        : moment(value, 'YYYY-MM-DD').format(fmt);
    },
    onPaginationData(paginationData) {
      // this.$refs.paginationTop.setPaginationData(paginationData);
      // this.$refs.paginationInfoTop.setPaginationData(paginationData);

      this.$refs.pagination.setPaginationData(paginationData);
      this.$refs.paginationInfo.setPaginationData(paginationData);
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page);
    },
    onAction(action, data, index) {
      console.log(`slot) action: ${action}, ${data.name}, ${index}`);
    },
    onCellClicked(data, field) {
      console.log(`cellClicked: ${field.name}`);
      this.$refs.vuetable.toggleDetailRow(data.id);
    },
    onFilterSet(filterText) {
      this.appendParams.filter = filterText;
      Vue.nextTick(() => this.$refs.vuetable.refresh());
    },
    onFilterReset() {
      delete this.appendParams.filter;
      Vue.nextTick(() => this.$refs.vuetable.refresh());
    },
  },
};
</script>
