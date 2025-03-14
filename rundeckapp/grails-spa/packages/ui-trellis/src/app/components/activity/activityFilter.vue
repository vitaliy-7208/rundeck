<template>
  <span>
    <dropdown v-if="query.recentFilter!=='-' && displayOpts.showRecentFilter">
      <span class="dropdown-toggle text-info cursor-pointer mr-2">
        {{ $t(`period.label.${period.name}`)}}
        <span class="caret"></span>
      </span>
      <template v-slot:dropdown>
        <li v-for="perobj in periods" :key="perobj.name">
          <a role="button" @click="changePeriod(perobj)">
              {{ $t(`period.label.${perobj.name}`)}}
            <span v-if="period.name===perobj.name">√</span>
          </a>
        </li>
      </template>
    </dropdown>

    <btn
      @click="filterOpen=true"
      size="xs"
      :class="hasQuery?'btn-queried btn-info':'btn-default'"
      v-tooltip="hasQuery?$t('Click to edit Search Query'):''"
      v-if="displayOpts.showFilter"
    >
      <span v-if="hasQuery" class="query-params-summary">
        <ul class="list-inline">
          <li v-for="qname in queryParamsList" :key="qname">
              {{ $t(`jobquery.title.${qname}`) }}:
            <code class="queryval">{{query[qname]}}</code>
          </li>
        </ul>
      </span>
      <span v-else>{{$t('search.ellipsis')}}</span>
    </btn>

    <saved-filters
      :query="modelValue"
      :has-query="hasQuery"
      @select_filter="selectFilter($event)"
      v-if="modelValue && displayOpts.showSavedFilters"
      :event-bus="eventBus"
    ></saved-filters>

    <modal
      id="activityFilter"
      v-model="filterOpen"
      :title="$t('Search Activity')"
      size="lg"
      @hide="closing"
      append-to-body
    >
      <div>
        <div class="base-filters">
          <div class="row">
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="jobIdFilter" class="sr-only">
                    {{ $t('jobquery.title.jobFilter') }}
                </label>
                <input
                  type="text"
                  name="jobFilter"
                  v-model="query.jobFilter"
                  autofocus="true"
                  class="form-control"
                  :placeholder="$t('jobquery.title.jobFilter')"
                />
              </div>

              <div class="form-group" v-if="query.jobIdFilter">
                <label for="jobIdFilter" class="sr-only">
                  {{ $t("jobquery.title.jobIdFilter") }}
                </label>
                <input
                  type="text"
                  name="jobIdFilter"
                  v-model="query.jobIdFilter"
                  class="form-control"
                  :placeholder="$t( 'jobquery.title.jobIdFilter')"
                />
              </div>
            </div>
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="userFilter" class="sr-only">
                    {{ $t("jobquery.title.userFilter") }}
                </label>
                <input
                  type="text"
                  name="userFilter"
                  v-model="query.userFilter"
                  class="form-control"
                  :placeholder="$t( 'jobquery.title.userFilter')"
                />
              </div>
            </div>
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="execnodeFilter" class="sr-only">
                    {{ $t("jobquery.title.filter") }}
                </label>
                <input
                  type="text"
                  name="execnodeFilter"
                  v-model="query.execnodeFilter"
                  class="form-control"
                  :placeholder="$t( 'jobquery.title.filter')"
                />
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="titleFilter" class="sr-only">
                    {{ $t("jobquery.title.titleFilter") }}
                </label>
                <input
                  type="text"
                  name="titleFilter"
                  v-model="query.titleFilter"
                  class="form-control"
                  :placeholder="$t( 'jobquery.title.titleFilter')"
                />
              </div>
            </div>
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="statFilter" class="sr-only">
                    {{ $t("jobquery.title.statFilter") }}
                </label>
                <select
                  name="statFilter"
                  v-model="query.statFilter"
                  noSelection="['': 'Any']"
                  valueMessagePrefix="status.label"
                  class="form-control"
                >
                  <option value>Any</option>
                  <option>succeed</option>
                  <option>fail</option>
                  <option>cancel</option>
                  <option>missed</option>
                </select>
              </div>
            </div>
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <label for="recentFilter" class="sr-only">
                    {{ $t("jobquery.title.recentFilter") }}
                </label>
                <span class="radiolist">
                  <select name="recentFilter" v-model="query.recentFilter" class="form-control">
                    <option value>Any Time</option>
                    <option :value="val" v-for="(key,val) in recentDateFilters" :key="key">{{key}}</option>
                    <option value="-">Other...</option>
                  </select>
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="date-filters panel panel-default" v-if="query.recentFilter==='-'">
          <div class="panel-body form-horizontal">
            <div v-for="df in DateFilters" :key="df.name" class="container-fluid">
              <date-filter v-model="df.filter">{{$t('jobquery.title.'+df.name)}}</date-filter>
            </div>
          </div>
        </div>
      </div>
      <template v-slot:footer>
        <btn @click="filterOpen=false">{{$t('cancel')}}</btn>
        <btn @click="search" type="primary" class="btn btn-primary">{{$t('search')}}</btn>
        <btn @click="saveFilter" type="default" class="btn-default pull-right">
          <i class="glyphicon glyphicon-plus"></i>
          {{$t('Save as a Filter...')}}
        </btn>
      </template>
    </modal>
  </span>
</template>
<script>
import { defineComponent } from 'vue'
import DateTimePicker from "./dateTimePicker.vue";
import DateFilter from "./dateFilter.vue";
import SavedFilters from "./savedFilters.vue";

export default defineComponent({
  name: 'ActivityFilter',
  components: {
    DateTimePicker,
    DateFilter,
    SavedFilters
  },
  props: ["eventBus", "modelValue", "eventBus", "opts"],
  emit: ["update:modelValue"],
  data() {
    return {
      displayOpts: {
        showRecentFilter: true,
        showFilter: true,
        showSavedFilters: true
      },
      filterOpen: false,
      DateQueryNames: [
        "startafterFilter",
        "startbeforeFilter",
        "endafterFilter",
        "endbeforeFilter"
      ],
      QueryNames: [
        "jobFilter",
        "jobIdFilter",
        "userFilter",
        "execnodeFilter",
        "titleFilter",
        "statFilter",
        "startafterFilter",
        "startbeforeFilter",
        "endafterFilter",
        "endbeforeFilter"
      ],
      DateFilters: [
        {
          name: "startafterFilter",
          filter: {
            enabled: false,
            datetime: ""
          }
        },

        {
          name: "startbeforeFilter",
          filter: {
            enabled: false,
            datetime: ""
          }
        },
        {
          name: "endafterFilter",
          filter: {
            enabled: false,
            datetime: ""
          }
        },
        {
          name: "endbeforeFilter",
          filter: {
            enabled: false,
            datetime: ""
          }
        }
      ],
      query: {
        jobFilter: "",
        jobIdFilter: "",
        userFilter: "",
        execnodeFilter: "",
        titleFilter: "",
        statFilter: "",
        recentFilter: "",
        startafterFilter: "",
        startbeforeFilter: "",
        endafterFilter: "",
        endbeforeFilter: ""
      },
      hasQuery: false,
      showDateFilters: false,
      recentDateFilters: {
        "1h": "1 Hour",
        "1d": "1 Day",
        "1w": "1 Week",
        "1m": "1 Month"
      },
      didSearch: false,

      period: { name: "All", params: {} },
      periods: [
        { name: "All", params: { recentFilter: "" } },
        { name: "Hour", params: { recentFilter: "1h" } },
        { name: "Day", params: { recentFilter: "1d" } },
        { name: "Week", params: { recentFilter: "1w" } },
        { name: "Month", params: { recentFilter: "1m" } }
      ]
    };
  },
  methods: {
    checkQueryIsPresent() {
      let isquery = this.QueryNames.findIndex(q => this.query[q]) >= 0;

      this.hasQuery = isquery;
    },
    updated() {
      this.$emit("update:modelValue", this.query);
    },
    search() {
      this.checkQueryIsPresent();
      this.query.filterName = "";
      this.didSearch = true;
      this.filterOpen = false;
      this.updated();
    },
    selectFilter(filter) {
      this.QueryNames.forEach(v => (this.query[v] = filter.query[v]));
      this.DateQueryNames.forEach(v => {
        if (filter.query[v]) {
          this.query["do" + v] = "true";
        }
      });
      if (filter.query.recentFilter) {
        this.query.recentFilter = filter.query.recentFilter;
      }

      this.query.filterName = filter.filterName;
      this.checkQueryIsPresent();
      this.updateSelectedPeriod();
      this.updated();
    },
    cancel() {
      this.reset();
      this.filterOpen = false;
    },
    reset() {
      this.query = Object.assign({}, this.modelValue);
      this.DateFilters.forEach(element => {
        element.filter.datetime = this.query[element.name];
        element.filter.enabled = !!element.filter.datetime;
      });
      this.checkQueryIsPresent();
    },
    closing() {
      if (this.didSearch) {
        this.didSearch = false;
      } else {
        this.reset();
      }
    },
    saveFilter() {
      this.search();
      // I added a timeout to the popping of the save modal
      // this allows for the filter to be applied before
      // opening the save modal which resolves an issues we experienced
      // where the saving of a filter (giving it a name) didn't actually
      // save the filter itself, just the name
      setTimeout(() => {
        this.eventBus.emit("invoke-save-filter");
        this.didSearch = true;
        this.filterOpen = false;
      }, 500);
    },
    changePeriod(period) {
      this.period = period;
      this.query.recentFilter = period.params.recentFilter;
      this.updated();
    },
    updateSelectedPeriod() {
      if (
        this.query.recentFilter &&
        this.period &&
        this.query.recentFilter !== this.period.params.recentFilter
      ) {
        const p = this.periods.find(
          v => v.params.recentFilter === this.query.recentFilter
        );
        if (p && p !== this.period) {
          this.period = p;
        }
      }
    }
  },
  watch: {
    query: {
      handler(newValue, oldValue) {
        this.updateSelectedPeriod();
      }
    },
    modelValue: {
      handler(newValue, oldValue) {
        this.reset();
      },
      deep: true
    },
    DateFilters: {
      handler(newValue, oldVale) {
        newValue.forEach(element => {
          if (element.filter.enabled) {
            this.query["do" + element.name] = "true";
            this.query[element.name] = element.filter.datetime;
          } else {
            this.query["do" + element.name] = "false";
            this.query[element.name] = "";
          }
        });
      },
      deep: true
    }
  },
  computed: {
    queryParamsList() {
      return this.QueryNames.filter(s => !!this.query[s]);
    }
  },
  mounted() {
    Object.assign(this.displayOpts, this.opts);
    this.reset();
  }
})
</script>
<style lang="scss" scoped>
.query-params-summary {
  ul.list-inline {
    display: inline-block;
    margin: 0;
  }
}
.btn-queried {
  border-style: dotted;
}
.btn-primary {
  color: var(--font-fill-color);
  background-color: var(--primary-color);
  &:hover {
  color: var(--font-fill-color);
  background-color: var(--primary-states-color);
  }
}
</style>
