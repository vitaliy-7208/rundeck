<template>
    <modal
        :title="$t('filter.jobs')"
        size="lg"
        v-model="visible"
        v-if="jobPageStore"
    >
        <div class="form-horizontal">
            <input
                type="hidden"
                name="project"
                v-model="jobPageStore.query['project']"
            />
            <input type="hidden" name="max" value="-1" />
            <input type="hidden" name="offset" value="0" />

            <div class="form-group" v-if="jobPageStore.query['idlist']">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}idlist"
                    >{{ $t("jobquery.title.idlist") }}</label
                >
                <div class="col-sm-10">
                    <input
                        type="text"
                        name="idlist"
                        id="${rkey}idlist"
                        v-model="jobPageStore.query['idlist']"
                        class="form-control"
                    />
                </div>
            </div>

            <div class="form-group">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}jobFilter"
                    >{{ $t("jobquery.title.jobFilter") }}</label
                >
                <div class="col-sm-10">
                    <input
                        type="text"
                        name="jobFilter"
                        id="${rkey}jobFilter"
                        v-model="jobPageStore.query['jobFilter']"
                        class="form-control"
                    />
                </div>
            </div>
            <div class="form-group">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}groupPath"
                    >{{ $t("jobquery.title.groupPath") }}</label
                >
                <div class="col-sm-10">
                    <div class="input-group">
                        <span class="input-group-addon"
                            ><i class="glyphicon glyphicon-folder-open"></i
                        ></span>
                        <input
                            type="text"
                            name="groupPath"
                            id="${rkey}groupPath"
                            v-model="jobPageStore.query['groupPath']"
                            class="form-control"
                        />
                    </div>
                </div>
            </div>
            <div class="form-group">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}descFilter"
                    >{{ $t("jobquery.title.descFilter") }}</label
                >
                <div class="col-sm-10">
                    <input
                        type="text"
                        name="descFilter"
                        id="${rkey}descFilter"
                        v-model="jobPageStore.query['descFilter']"
                        class="form-control"
                    />
                </div>
            </div>
            <div class="form-group">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}scheduledFilter"
                    >{{ $t("jobquery.title.scheduledFilter") }}</label
                >
                <div class="col-sm-10">
                    <label class="radio-inline">
                        <input
                            type="radio"
                            name="scheduledFilter"
                            value="true"
                            v-model="scheduledFilter"
                        />
                        {{ $t("yes") }}
                    </label>
                    <label class="radio-inline">
                        <input
                            type="radio"
                            name="scheduledFilter"
                            value="false"
                            v-model="scheduledFilter"
                        />
                        {{ $t("no") }}
                    </label>
                    <label class="radio-inline">
                        <input
                            type="radio"
                            name="scheduledFilter"
                            value=""
                            v-model="scheduledFilter"
                        />
                        {{ $t("all") }}
                    </label>
                </div>
            </div>
            <!--            <g:if test="${clusterModeEnabled}">-->
            <div class="form-group">
                <label
                    class="col-sm-2 control-label"
                    for="${enc(attr:rkey)}serverNodeUUIDFilter"
                    >{{ $t("jobquery.title.serverNodeUUIDFilter") }}</label
                >
                <div class="col-sm-10">
                    <input
                        type="text"
                        name="serverNodeUUIDFilter"
                        id="${rkey}serverUuid"
                        v-model="jobPageStore.query['serverNodeUUIDFilter']"
                        class="form-control"
                    />
                </div>
            </div>
            <!--            </g:if>-->

            <!--            <g:if test="${jobQueryComponents}">-->
            <!--              <g:each in="${jobQueryComponents}" var="component">-->
            <!--                <g:if test="${component.value.queryProperties}">-->
            <!--                  <g:each in="${component.value.queryProperties}" var="properties">-->
            <!--                    <g:render template="/framework/pluginConfigPropertiesInputs"-->
            <!--                              model="${[-->
            <!--                              properties         : properties,-->
            <!--                              report             : null,-->
            <!--                              prefix             : '',-->
            <!--                              values             : params,-->
            <!--                              fieldnamePrefix    : '',-->
            <!--                              origfieldnamePrefix: 'orig.' ,-->
            <!--                              messagePrefix       :'',-->
            <!--                              messagesType       : 'job.query'-->
            <!--                            ]}"/>-->
            <!--                  </g:each>-->
            <!--                </g:if>-->
            <!--              </g:each>-->
            <!--            </g:if>-->
            <ui-socket location="fields" section="job-search-modal"/>
        </div>
        <template #footer>
            <btn type="button" @click="doClose">
                {{ $t("cancel") }}
            </btn>
            <btn type="button" @click="doClear">
                {{ $t("job.filter.clear.button.title") }}
            </btn>
            <btn type="primary" @click="doSearch">
                {{ $t("job.filter.apply.button.title") }}
            </btn>
        </template>
    </modal>
</template>

<script lang="ts">
import UiSocket from '@/library/components/utils/UiSocket.vue'
import {
    JobPageStore,
    JobPageStoreInjectionKey,
} from "@/library/stores/JobPageStore";
import { defineComponent, inject, ref } from "vue";

export default defineComponent({
    name: "JobSearchModal",
    components: { UiSocket },
    emits: ["close", "search", "clear", "update:modelValue"],
    props: {
        modelValue: {
            type: Boolean,
            default: false,
        },
    },
    setup(props) {
        const jobPageStore: JobPageStore = inject(
            JobPageStoreInjectionKey
        ) as JobPageStore;
        return {
            jobPageStore,
            visible: ref(props.modelValue),
        };
    },
    computed: {
        scheduledFilter: {
            get() {
                if (this.jobPageStore.query["scheduledFilter"]) {
                    return this.jobPageStore.query["scheduledFilter"];
                }
                return "";
            },
            set(value) {
                this.jobPageStore.query["scheduledFilter"] = value;
            },
        },
    },
    watch: {
        visible(value) {
            this.$emit("update:modelValue", value);
        },
        modelValue(value) {
            this.visible = value;
        },
    },
    methods: {
        doClose() {
            this.$emit("close");
        },
        doClear() {
            this.$emit("clear");
        },
        doSearch() {
            this.$emit("search");
        },
    },
});
</script>

<style scoped lang="scss"></style>