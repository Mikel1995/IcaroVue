<template>
  <div>
    <h2>{{ msg }}</h2>
    <div v-if="isLoading" class="spinner spinner-lg"></div>
    <div v-if="(user.account_type == 'admin') || (user.account_type == 'reseller') && !isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <label v-if="!isLoading" class="col-sm-2 control-label" for="textInput-markup">Hotspot</label>
      <div v-if="!isLoading" class="col-sm-4">
        <select v-on:change="getAll()" v-model="hotspotSearchId" class="form-control">
          <option value="0">-</option>
          <option v-for="hotspot in hotspots" v-bind:key="hotspot.id" v-bind:value="hotspot.id">
            {{ hotspot.name }} - {{ hotspot.description}}
          </option>
        </select>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <label class="col-sm-2 control-label" for="textInput-markup">{{$t('session.user')}}</label>
      <div class="col-sm-4">
        <select v-on:change="getAll()" v-model="hotspotUserId" class="form-control">
          <option value="0">-</option>
          <option v-for="user in users" v-bind:key="user.id" v-bind:value="user.id">
            {{ user.name }}
          </option>
        </select>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <label class="col-sm-2 control-label" for="textInput-markup">{{$t('session.unit')}}</label>
      <div class="col-sm-4">
        <select v-on:change="getAll()" v-model="hotspotUnitId" class="form-control">
          <option value="0">-</option>
          <option v-for="unit in units" v-bind:key="unit.id" v-bind:value="unit.id">
            {{ unit.name }} - {{ unit.description }}
          </option>
        </select>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <label class="col-sm-2 control-label" for="textInput-markup">{{$t('session.from')}}</label>
      <div class="col-sm-4">
        <datepicker :format="dateFormatter" @input="getAll()" v-model="hotspotDateFrom" :language="locale"></datepicker>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <label class="col-sm-2 control-label" for="textInput-markup">{{$t('session.to')}}</label>
      <div class="col-sm-4">
        <datepicker :format="dateFormatter" @input="getAll()" v-model="hotspotDateTo" :language="locale"></datepicker>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <div class="col-sm-3">
        <a class="link" @click="clearFilters()">{{$t('session.clear_filters')}}</a>
      </div>
    </div>
    <div v-if="!isLoading" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
      <div class="col-xs-3 col-sm-3 col-md-3 col-lg-3">
        <button class="btn btn-primary" @click="getAll(true)">{{$t('session.refresh')}}</button>
      </div>
    </div>

    <ul v-if="!isLoading" class="nav nav-tabs nav-tabs-pf adjust-tabs" id="myTab" role="tablist">
      <li class="nav-item">
        <a @click="handleTab('active')" class="nav-link active" id="active-tab-parent" data-toggle="tab" href="#active-tab" role="tab"
          aria-controls="active" aria-selected="true">{{$t('session.active')}}</a>
      </li>
      <li class="nav-item">
        <a @click="handleTab('history')" class="nav-link" id="history-tab-parent" data-toggle="tab" href="#history-tab" role="tab"
          aria-controls="history" aria-selected="false">{{$t('session.history')}}</a>
      </li>
    </ul>
    <div class="tab-pane fade" id="active-tab" role="tabpanel" aria-labelledby="active-tab">
      <div v-if="!isLoading && activeTab == 'active'" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <div class="col-sm-12">
          <button @click="exportCSVActive()" class="btn btn-primary export-btn">{{$t('session.export_csv')}}</button>
        </div>
        <div class="result-list adjust-results">{{rows_active.length}} {{rows_active.length == 1 ? $t('result') : $t('results')}}</div>
      </div>
      <vue-good-table v-if="!isLoading && activeTab == 'active'" @perPageChanged="handlePerPage" :customRowsPerPageDropdown="[25,50,100]"
        :perPage="hotspotPerPage" :columns="columns_active" :rows="rows_active" :lineNumbers="false" :defaultSortBy="{field: 'duration', type: 'asc'}"
        :globalSearch="true" :globalSearchFn="searchFn" :paginate="true" styleClass="table" :nextText="tableLangsTexts.nextText"
        :prevText="tableLangsTexts.prevText" :rowsPerPageText="tableLangsTexts.rowsPerPageText" :globalSearchPlaceholder="tableLangsTexts.globalSearchPlaceholder"
        :ofText="tableLangsTexts.ofText">
        <template slot="table-row" slot-scope="props">
          <td class="fancy">
            <a :href="'#/units/'+ props.row.unit_id">{{ extractUnit(props.row.unit_id) && extractUnit(props.row.unit_id).name || '-'}}</a>
          </td>
          <td class="fancy">
            <a :href="'#/sessions/'+ props.row.id">{{ props.row.username || '-'}}</a>
          </td>
          <td class="fancy">{{ props.row.bytes_down | byteFormat}}</td>
          <td class="fancy">{{ props.row.bytes_up | byteFormat }}</td>
          <td class="fancy">{{ props.row.duration | secondsInHour }}</td>
          <td class="fancy">{{ props.row.start_time | formatDate }}</td>
          <td class="fancy">{{ props.row.update_time | formatDate }}</td>
          <td>
            <a :href="'#/sessions/'+ props.row.id">
              <span class="fa fa-angle-right details-arrow"></span>
            </a>
          </td>
        </template>
      </vue-good-table>
    </div>
    <div class="tab-pane fade" id="history-tab" role="tabpanel" aria-labelledby="history-tab">
      <div v-if="!isLoading && activeTab == 'history'" class="form-group select-search col-xs-12 col-sm-12 col-md-12 col-lg-12">
        <div class="col-sm-12">
          <button @click="exportCSVHistory()" class="btn btn-primary export-btn">{{$t('session.export_csv')}}</button>
        </div>
        <div class="result-list adjust-results">{{rows_history.length}} {{rows_history.length == 1 ? $t('result') : $t('results')}}</div>
      </div>
      <vue-good-table v-if="!isLoading && activeTab == 'history'" @perPageChanged="handlePerPage" :customRowsPerPageDropdown="[25,50,100]"
        :perPage="hotspotPerPage" :columns="columns_history" :rows="rows_history" :lineNumbers="false" :defaultSortBy="{field: 'duration', type: 'asc'}"
        :globalSearch="true" :globalSearchFn="searchFn" :paginate="true" styleClass="table" :nextText="tableLangsTexts.nextText"
        :prevText="tableLangsTexts.prevText" :rowsPerPageText="tableLangsTexts.rowsPerPageText" :globalSearchPlaceholder="tableLangsTexts.globalSearchPlaceholder"
        :ofText="tableLangsTexts.ofText">
        <template slot="table-row" slot-scope="props">
          <td class="fancy">
            <a :href="'#/units/'+ props.row.unit_id">{{ extractUnit(props.row.unit_id) && extractUnit(props.row.unit_id).name || '-'}}</a>
          </td>
          <td class="fancy">
            <a :href="'#/sessions/'+ props.row.id">{{ props.row.username || '-'}}</a>
          </td>
          <td class="fancy">{{ props.row.bytes_down | byteFormat}}</td>
          <td class="fancy">{{ props.row.bytes_up | byteFormat }}</td>
          <td class="fancy">{{ props.row.duration | secondsInHour }}</td>
          <td class="fancy">{{ props.row.start_time | formatDate }}</td>
          <td class="fancy">{{ props.row.stop_time | formatDate }}</td>
          <td>
            <a :href="'#/sessions/'+ props.row.id">
              <span class="fa fa-angle-right details-arrow"></span>
            </a>
          </td>
        </template>
      </vue-good-table>
    </div>
  </div>
</template>

<script>
  import StorageService from '../services/storage';
  import UtilService from '../services/util';
  import HotspotService from '../services/hotspot';
  import SessionService from '../services/session';
  import HistoryService from '../services/history';
  import UserService from '../services/user';
  import UnitService from '../services/unit';

  import Datepicker from 'vuejs-datepicker';
  import moment from 'moment'

  export default {
    name: 'Reports',
    mixins: [SessionService, StorageService, UtilService, HotspotService, UserService, UnitService, HistoryService],
    components: {
      Datepicker
    },
    data() {
      var hsId = this.get('sessions_hotspot_id') || 0
      if (this.$parent.user.info.type == 'customer' || this.$parent.user.info.type == 'desk') {
        hsId = this.$parent.user.info.hotspot_id
      }
      var activeTab = this.get('sessions_active_tab') || 'active'
      setTimeout(function () {
        window.$('#' + activeTab + '-tab-parent').click()
      }, 500)

      return {
        msg: this.$i18n.t("menu.sessions"),
        isLoading: true,
        locale: this.$root.$options.currentLocale,
        columns_active: [{
            label: this.$i18n.t('session.unit'),
            field: 'unit_id',
            filterable: true,
            sortable: false
          }, {
            label: this.$i18n.t('session.user'),
            field: 'username',
            filterable: true,
            sortable: false
          }, {
            label: this.$i18n.t('session.bytes_down'),
            field: 'bytes_down',
            type: 'number',
            filterable: true,
          }, {
            label: this.$i18n.t('session.bytes_up'),
            field: 'bytes_up',
            type: 'number',
            filterable: true,
          }, {
            label: this.$i18n.t('session.duration'),
            field: 'duration',
            type: 'number',
            filterable: true,
          },
          {
            label: this.$i18n.t('session.start_time'),
            field: 'start_time',
            filterable: true,
          },
          {
            label: this.$i18n.t('session.update_time'),
            field: 'update_time',
            filterable: true,
            sortable: false
          },
          {
            label: '',
            field: '',
            sortable: false
          },
        ],
        columns_history: [{
            label: this.$i18n.t('session.unit'),
            field: 'unit_id',
            filterable: true,
            sortable: false
          }, {
            label: this.$i18n.t('session.user'),
            field: 'username',
            filterable: true,
            sortable: false
          }, {
            label: this.$i18n.t('session.bytes_down'),
            field: 'bytes_down',
            type: 'number',
            filterable: true,
          }, {
            label: this.$i18n.t('session.bytes_up'),
            field: 'bytes_up',
            type: 'number',
            filterable: true,
          }, {
            label: this.$i18n.t('session.duration'),
            field: 'duration',
            type: 'number',
            filterable: true,
          },
          {
            label: this.$i18n.t('session.start_time'),
            field: 'start_time',
            filterable: true,
          },
          {
            label: this.$i18n.t('session.stop_time'),
            field: 'stop_time',
            filterable: true,
            sortable: false
          },
          {
            label: '',
            field: '',
            sortable: false
          },
        ],
        rows_active: [],
        rows_history: [],
        activeTab: activeTab,
        tableLangsTexts: this.tableLangs(),
        hotspots: [],
        hotspotSearchId: hsId,
        hotspotUserId: this.get('sessions_user_id') || 0,
        hotspotUnitId: this.get('sessions_unit_id') || 0,
        hotspotDateFrom: this.get('sessions_date_from') || moment(Date.now() - 12096e5).utc().startOf('day').toISOString(),
        hotspotDateTo: this.get('sessions_date_to') || moment(Date.now()).endOf('day').utc().toISOString(),
        hotspotPerPage: this.get('sessions_per_page') || 25,
        user: this.get('loggedUser') || null,
        users: [],
        units: []
      }
    },
    mounted() {
      if (this.$route.params.hotspotId !== undefined) {
        this.hotspotSearchId = this.$route.params.hotspotId;
      }
      // get session list
      this.getAll(true)
      this.getAllHotspots()
      this.getAllUsers()
      this.getAllUnits()
    },
    methods: {
      handleTab(tab) {
        this.activeTab = tab
        this.getAll()
      },
      handlePerPage(evt) {
        this.set('sessions_per_page', evt.currentPerPage)
      },
      searchFn(row, col, cellValue, searchTerm) {
        var value = cellValue.toString().toLowerCase()
        if (col.field == 'unit_id') {
          value = this.extractUnit(cellValue).description.toLowerCase()
        }

        if (col.field == 'user_id') {
          value = this.extractUser(cellValue).name.toLowerCase()
        }
        return value.includes(searchTerm.toLowerCase())
      },
      dateFormatter(date) {
        return moment(date).format('DD MMMM YYYY');
      },
      getAllHotspots() {
        this.hotspotGetAll(success => {
          this.hotspots = success.body
          $('[data-toggle="tooltip"]').tooltip()
          this.isLoading = false;
        }, error => {
          console.log(error)
        })
      },
      getAll(refresh) {
        // save to storage
        this.set('sessions_active_tab', this.activeTab || this.get('sessions_active_tab') || 0)
        this.set('sessions_hotspot_id', this.hotspotSearchId || this.get('sessions_hotspot_id') || 0)
        this.set('sessions_user_id', this.hotspotUserId || this.get('sessions_user_id') || 0)
        this.set('sessions_unit_id', this.hotspotUnitId || this.get('sessions_unit_id') || 0)
        this.set('sessions_date_from', this.hotspotDateFrom || this.get('sessions_date_from') || moment(Date.now() - 12096e5).utc().startOf('day').toISOString())
        this.set('sessions_date_to', this.hotspotDateTo || this.get('sessions_date_to') || moment(Date.now()).endOf('day').utc().toISOString())

        // preload users and units
        this.getAllUsers()
        this.getAllUnits()

        if (refresh) {
          if (this.hotspotDateTo === moment().utc().endOf('day').toISOString()) {
            this.hotspotDateTo = moment(Date.now()).endOf('day').utc().toISOString()
          }
        }

        // get all sessions
        if (this.activeTab == 'active') {
          this.sessionGetAll(this.hotspotSearchId, this.hotspotUserId, this.hotspotUnitId, new Date(this.hotspotDateFrom)
            .toISOString(), new Date(this.hotspotDateTo).toISOString(), success => {
              this.rows_active = success.body
              this.isLoading = false
            }, error => {
              this.isLoading = false
              this.rows_active = []
              console.log(error)
            })
        } else {
          this.historiesGetAll(this.hotspotSearchId, this.hotspotUserId, this.hotspotUnitId, new Date(this.hotspotDateFrom)
            .toISOString(), new Date(this.hotspotDateTo).toISOString(), success => {
              this.rows_history = success.body
              this.isLoading = false
            }, error => {
              this.isLoading = false
              this.rows_history = []
              console.log(error)
            })
        }
      },
      getAllUsers() {
        this.userGetAll(this.hotspotSearchId, null, success => {
          this.users = success.body
        }, error => {
          console.log(error)
          this.users = {}
        })
      },
      getAllUnits() {
        this.unitGetAll(this.hotspotSearchId, success => {
          this.units = success.body
        }, error => {
          console.log(error)
          this.units = {}
        })
      },
      extractUnit(id) {
        for (var u in this.units) {
          if (this.units[u].id == id) {
            return this.units[u]
            break;
          }
        }
      },
      extractUser(id) {
        for (var u in this.users) {
          if (this.users[u].id == id) {
            return this.users[u]
            break;
          }
        }
      },
      clearFilters() {
        this.hotspotSearchId = 0
        this.hotspotUserId = 0
        this.hotspotUnitId = 0
        this.hotspotDateFrom = moment(Date.now() - 12096e5).utc().startOf('day').toISOString()
        this.hotspotDateTo = moment(Date.now()).endOf('day').utc().toISOString()
        this.set('sessions_hotspot_id', 0)
        this.set('sessions_user_id', 0)
        this.set('sessions_unit_id', 0)
        this.set('sessions_date_from', moment(Date.now() - 12096e5).utc().startOf('day').toISOString())
        this.set('sessions_date_to', moment(Date.now()).endOf('day').utc().toISOString())
        this.getAll()
      },
      exportCSVActive() {
        var newRows = JSON.parse(JSON.stringify(this.rows_active))
        for (var r in newRows) {
          newRows[r].unit_id = this.extractUnit(newRows[r].unit_id) && this.extractUnit(newRows[r].unit_id).description ||
            "-"
          newRows[r].user_id = this.extractUser(newRows[r].user_id) && this.extractUser(newRows[r].user_id).name || "-"
          newRows[r].bytes_up = this.$options.filters['byteFormat'](newRows[r].bytes_up)
          newRows[r].bytes_down = this.$options.filters['byteFormat'](newRows[r].bytes_down)
          newRows[r].duration = this.$options.filters['secondsInHour'](newRows[r].duration)
          newRows[r].start_time = this.$options.filters['formatDate'](newRows[r].start_time)
          newRows[r].update_time = this.$options.filters['formatDate'](newRows[r].update_time)
        }
        var csv = this.createCSV(this.columns_active, newRows)
        this.downloadCSV(csv.cols, csv.rows, 'sessions_active')
      },
      exportCSVHistory() {
        var newRows = JSON.parse(JSON.stringify(this.rows_history))
        for (var r in newRows) {
          newRows[r].unit_id = this.extractUnit(newRows[r].unit_id) && this.extractUnit(newRows[r].unit_id).description ||
            "-"
          newRows[r].user_id = this.extractUser(newRows[r].user_id) && this.extractUser(newRows[r].user_id).name || "-"
          newRows[r].bytes_up = this.$options.filters['byteFormat'](newRows[r].bytes_up)
          newRows[r].bytes_down = this.$options.filters['byteFormat'](newRows[r].bytes_down)
          newRows[r].duration = this.$options.filters['secondsInHour'](newRows[r].duration)
          newRows[r].start_time = this.$options.filters['formatDate'](newRows[r].start_time)
          newRows[r].stop_time = this.$options.filters['formatDate'](newRows[r].stop_time)
        }
        var csv = this.createCSV(this.columns_history, newRows)
        this.downloadCSV(csv.cols, csv.rows, 'sessions_history')
      }
    }
  }

</script>
<style scoped>
  .adjust-tabs {
    margin-left: 15px !important;
    margin-right: 15px !important;
    border-bottom: 1px solid #cccccc !important;
  }

  .adjust-results {
    right: 45px !important;
    top: 0px;
  }

</style>
