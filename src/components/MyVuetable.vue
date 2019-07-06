// MyVuetable.vue

<script>
import Vuetable from 'vuetable-2/src/components/Vuetable';
// import VuetablePagination from 'vuetable-2/src/components/VuetablePagination';
import VuetablePaginationBootstrap from './VuetablePaginationBootstrap';
import VuetablePaginationInfo from 'vuetable-2/src/components/VuetablePaginationInfo';
import accounting from 'accounting';
import moment from 'moment';
import Vue from 'vue';
import VueEvents from 'vue-events';
import CustomActions from './CustomActions';
import FilterBar from './FilterBar';
import CssConfig from './VuetableCssConfig';

Vue.use(VueEvents);
Vue.component('filter-bar', FilterBar);
Vue.component('custom-actions', CustomActions);

export default {
  name: 'my-vuetable',
  render(h) {
    return h(
      'div',
      {
        class: { ui: true, container: true }
      },
      [
        h('filter-bar'),
        this.renderVuetable(h),
        this.renderPagination(h)
      ]
    )
  },
  components: {
    Vuetable,
    VuetablePaginationBootstrap,
    // VuetablePagination,
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
    return {
      css: CssConfig
    };
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
    renderVuetable(h) {
      return h(
        'vuetable',
        {
          ref: 'vuetable',
          props: {
            apiUrl: this.apiUrl,
            fields: this.fields,
            paginationPath: "",
            perPage: 10,
            multiSort: true,
            sortOrder: this.sortOrder,
            appendParams: this.appendParams,
            detailRowComponent: this.detailRowComponent,
            css: this.css.table,
          },
          on: {
            'vuetable:cell-clicked': this.onCellClicked,
            'vuetable:pagination-data': this.onPaginationData,
          },
          scopedSlots: this.$vnode.data.scopedSlots
        }
      )
    },
    renderPagination(h) {
      return h(
        'div',
        { class: { 'vuetable-pagination': true, 'ui': true, 'basic': true, 'segment': true, 'grid': true } },
        [
          h('vuetable-pagination-info', { ref: 'paginationInfo' }),
          h('vuetable-pagination-bootstrap', {
            ref: 'pagination',
            class: { 'pull-right': true },
            // props: { css: this.css.pagination },
            on: {
              'vuetable-pagination:change-page': this.onChangePage
            }
          })
        ]
      )
    },
  },
};
</script>
