<template>
    <div v-if="loadedTheme" class="container mt-3">
        <!-- Sidebar for edit mode -->
        <div v-if="enableEditMode" class="sidebar">
            <div class="my-3">
                <label for="slug" class="form-label">{{ $t("Slug") }}</label>
                <div class="input-group">
                    <span id="basic-addon3" class="input-group-text">/status/</span>
                    <input id="slug" v-model="config.slug" type="text" class="form-control">
                </div>
            </div>

            <div class="my-3">
                <label for="title" class="form-label">{{ $t("Title") }}</label>
                <input id="title" v-model="config.title" type="text" class="form-control">
            </div>

            <div class="my-3">
                <label for="description" class="form-label">{{ $t("Description") }}</label>
                <textarea id="description" v-model="config.description" class="form-control"></textarea>
            </div>

            <div class="my-3 form-check form-switch">
                <input id="switch-theme" v-model="config.theme" class="form-check-input" type="checkbox" true-value="dark" false-value="light">
                <label class="form-check-label" for="switch-theme">{{ $t("Switch to Dark Theme") }}</label>
            </div>

            <div class="my-3 form-check form-switch">
                <input id="showTags" v-model="config.showTags" class="form-check-input" type="checkbox">
                <label class="form-check-label" for="showTags">{{ $t("Show Tags") }}</label>
            </div>

            <div v-if="false" class="my-3">
                <label for="password" class="form-label">{{ $t("Password") }} <sup>Coming Soon</sup></label>
                <input id="password" v-model="config.password" disabled type="password" autocomplete="new-password" class="form-control">
            </div>

            <div v-if="false" class="my-3">
                <label for="cname" class="form-label">Domain Names <sup>Coming Soon</sup></label>
                <textarea id="cname" v-model="config.domanNames" rows="3" disabled class="form-control" :placeholder="domainNamesPlaceholder"></textarea>
            </div>

            <div class="danger-zone">
                <button class="btn btn-danger me-2" @click="deleteDialog">
                    <font-awesome-icon icon="trash" />
                    {{ $t("Delete") }}
                </button>
            </div>

            <!-- Sidebar Footer -->
            <div class="sidebar-footer">
                <button class="btn btn-success me-2" @click="save">
                    <font-awesome-icon icon="save" />
                    {{ $t("Save") }}
                </button>

                <button class="btn btn-danger me-2" @click="discard">
                    <font-awesome-icon icon="save" />
                    {{ $t("Discard") }}
                </button>
            </div>
        </div>

        <!-- Main Status Page -->
        <div :class="{ edit: enableEditMode}" class="main">
            <!-- Logo & Title -->
            <h1 class="mb-4 title-flex">
                <!-- Logo -->
                <span class="logo-wrapper" @click="showImageCropUploadMethod">
                    <img :src="logoURL" alt class="logo me-2" :class="logoClass" />
                    <font-awesome-icon v-if="enableEditMode" class="icon-upload" icon="upload" />
                </span>

                <!-- Uploader -->
                <!--    url="/api/status-page/upload-logo" -->
                <ImageCropUpload v-model="showImageCropUpload"
                                 field="img"
                                 :width="128"
                                 :height="128"
                                 :langType="$i18n.locale"
                                 img-format="png"
                                 :noCircle="true"
                                 :noSquare="false"
                                 @crop-success="cropSuccess"
                />

                <!-- Title -->
                <Editable v-model="config.title" tag="span" :contenteditable="editMode" :noNL="true" />
            </h1>

            <!-- Admin functions -->
            <div v-if="hasToken" class="mb-4">
                <div v-if="!enableEditMode">
                    <button class="btn btn-info me-2" @click="edit">
                        <font-awesome-icon icon="edit" />
                        {{ $t("Edit Status Page") }}
                    </button>

                    <a href="/manage-status-page" class="btn btn-info">
                        <font-awesome-icon icon="tachometer-alt" />
                        {{ $t("Go to Dashboard") }}
                    </a>
                </div>

                <div v-else>
                    <button class="btn btn-primary btn-add-group me-2" @click="createIncident">
                        <font-awesome-icon icon="bullhorn" />
                        {{ $t("Create Incident") }}
                    </button>
                </div>
            </div>

            <!-- Incident -->
            <div v-if="incident !== null" class="shadow-box alert mb-4 p-4 incident" role="alert" :class="incidentClass">
                <strong v-if="editIncidentMode">{{ $t("Title") }}:</strong>
                <Editable v-model="incident.title" tag="h4" :contenteditable="editIncidentMode" :noNL="true" class="alert-heading" />

                <strong v-if="editIncidentMode">{{ $t("Content") }}:</strong>
                <Editable v-model="incident.content" tag="div" :contenteditable="editIncidentMode" class="content" />

                <!-- Incident Date -->
                <div class="date mt-3">
                    {{ $t("Created") }}: {{ $root.datetime(incident.createdDate) }} ({{ dateFromNow(incident.createdDate) }})<br />
                    <span v-if="incident.lastUpdatedDate">
                        {{ $t("Last Updated") }}: {{ $root.datetime(incident.lastUpdatedDate) }} ({{ dateFromNow(incident.lastUpdatedDate) }})
                    </span>
                </div>

                <div v-if="editMode" class="mt-3">
                    <button v-if="editIncidentMode" class="btn btn-light me-2" @click="postIncident">
                        <font-awesome-icon icon="bullhorn" />
                        {{ $t("Post") }}
                    </button>

                    <button v-if="!editIncidentMode && incident.id" class="btn btn-light me-2" @click="editIncident">
                        <font-awesome-icon icon="edit" />
                        {{ $t("Edit") }}
                    </button>

                    <button v-if="editIncidentMode" class="btn btn-light me-2" @click="cancelIncident">
                        <font-awesome-icon icon="times" />
                        {{ $t("Cancel") }}
                    </button>

                    <div v-if="editIncidentMode" class="dropdown d-inline-block me-2">
                        <button id="dropdownMenuButton1" class="btn btn-secondary dropdown-toggle" type="button" data-bs-toggle="dropdown" aria-expanded="false">
                            {{ $t("Style") }}: {{ $t(incident.style) }}
                        </button>
                        <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'info'">{{ $t("info") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'warning'">{{ $t("warning") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'danger'">{{ $t("danger") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'primary'">{{ $t("primary") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'light'">{{ $t("light") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'dark'">{{ $t("dark") }}</a></li>
                        </ul>
                    </div>

                    <button v-if="!editIncidentMode && incident.id" class="btn btn-light me-2" @click="unpinIncident">
                        <font-awesome-icon icon="unlink" />
                        {{ $t("Unpin") }}
                    </button>
                </div>
            </div>

            <!-- Overall Status -->
            <div class="shadow-box list  p-4 overall-status mb-4">
                <div v-if="Object.keys($root.publicMonitorList).length === 0 && loadedData">
                    <font-awesome-icon icon="question-circle" class="ok" />
                    {{ $t("No Services") }}
                </div>

                <template v-else>
                    <div v-if="allUp">
                        <font-awesome-icon icon="check-circle" class="ok" />
                        {{ $t("All Systems Operational") }}
                    </div>

                    <div v-else-if="partialDown">
                        <font-awesome-icon icon="exclamation-circle" class="warning" />
                        {{ $t("Partially Degraded Service") }}
                    </div>

                    <div v-else-if="allDown">
                        <font-awesome-icon icon="times-circle" class="danger" />
                        {{ $t("Degraded Service") }}
                    </div>

                    <div v-else>
                        <font-awesome-icon icon="question-circle" style="color: #efefef;" />
                    </div>
                </template>
            </div>

            <!-- Description -->
            <strong v-if="editMode">{{ $t("Description") }}:</strong>
            <Editable v-model="config.description" :contenteditable="editMode" tag="div" class="mb-4 description" />

            <div v-if="editMode" class="mb-4">
                <div>
                    <button class="btn btn-primary btn-add-group me-2" @click="addGroup">
                        <font-awesome-icon icon="plus" />
                        {{ $t("Add Group") }}
                    </button>
                </div>

                <div class="mt-3">
                    <div v-if="allMonitorList.length > 0 && loadedData">
                        <label>{{ $t("Add a monitor") }}:</label>
                        <select v-model="selectedMonitor" class="form-control">
                            <option v-for="monitor in allMonitorList" :key="monitor.id" :value="monitor">{{ monitor.name }}</option>
                        </select>
                    </div>
                    <div v-else class="text-center">
                        {{ $t("No monitors available.") }}  <router-link to="/add">{{ $t("Add one") }}</router-link>
                    </div>
                </div>
            </div>

            <div class="mb-4">
                <div v-if="$root.publicGroupList.length === 0 && loadedData" class="text-center">
                    <!-- 👀 Nothing here, please add a group or a monitor. -->
                    👀 {{ $t("statusPageNothing") }}
                </div>

                <PublicGroupList :edit-mode="enableEditMode" :show-tags="config.showTags" />
            </div>

            <footer class="mt-5 mb-4">
                {{ $t("Powered by") }} <a target="_blank" href="https://github.com/louislam/uptime-kuma">{{ $t("Uptime Kuma" ) }}</a>
            </footer>
        </div>

        <Confirm ref="confirmDelete" btn-style="btn-danger" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="deleteStatusPage">
            {{ $t("deleteStatusPageMsg") }}
        </Confirm>
    </div>
</template>

<script>
import axios from "axios";
import PublicGroupList from "../components/PublicGroupList.vue";
import ImageCropUpload from "vue-image-crop-upload";
import { STATUS_PAGE_ALL_DOWN, STATUS_PAGE_ALL_UP, STATUS_PAGE_PARTIAL_DOWN, UP } from "../util.ts";
import { useToast } from "vue-toastification";
import dayjs from "dayjs";
import Favico from "favico.js";
import { getResBaseURL } from "../util-frontend";
import Confirm from "../components/Confirm.vue";

const toast = useToast();

const leavePageMsg = "Do you really want to leave? you have unsaved changes!";

let feedInterval;

const favicon = new Favico({
    animation: "none"
});

export default {
    components: {
        PublicGroupList,
        ImageCropUpload,
        Confirm,
    },

    // Leave Page for vue route change
    beforeRouteLeave(to, from, next) {
        if (this.editMode) {
            const answer = window.confirm(leavePageMsg);
            if (answer) {
                next();
            } else {
                next(false);
            }
        }
        next();
    },

    data() {
        return {
            slug: null,
            enableEditMode: false,
            enableEditIncidentMode: false,
            hasToken: false,
            config: {},
            selectedMonitor: null,
            incident: null,
            previousIncident: null,
            showImageCropUpload: false,
            imgDataUrl: "/icon.svg",
            loadedTheme: false,
            loadedData: false,
            baseURL: "",
            clickedEditButton: false,
            domainNamesPlaceholder: "domain1.com\ndomain2.com\n..."
        };
    },
    computed: {

        logoURL() {
            if (this.imgDataUrl.startsWith("data:")) {
                return this.imgDataUrl;
            } else {
                return this.baseURL + this.imgDataUrl;
            }
        },

        /**
         * If the monitor is added to public list, which will not be in this list.
         */
        allMonitorList() {
            let result = [];

            for (let id in this.$root.monitorList) {
                if (this.$root.monitorList[id] && ! (id in this.$root.publicMonitorList)) {
                    let monitor = this.$root.monitorList[id];
                    result.push(monitor);
                }
            }

            return result;
        },

        editMode() {
            return this.enableEditMode && this.$root.socket.connected;
        },

        editIncidentMode() {
            return this.enableEditIncidentMode;
        },

        isPublished() {
            return this.config.published;
        },

        logoClass() {
            if (this.editMode) {
                return {
                    "edit-mode": true,
                };
            }
            return {};
        },

        incidentClass() {
            return "bg-" + this.incident.style;
        },

        overallStatus() {

            if (Object.keys(this.$root.publicLastHeartbeatList).length === 0) {
                return -1;
            }

            let status = STATUS_PAGE_ALL_UP;
            let hasUp = false;

            for (let id in this.$root.publicLastHeartbeatList) {
                let beat = this.$root.publicLastHeartbeatList[id];

                if (beat.status === UP) {
                    hasUp = true;
                } else {
                    status = STATUS_PAGE_PARTIAL_DOWN;
                }
            }

            if (! hasUp) {
                status = STATUS_PAGE_ALL_DOWN;
            }

            return status;
        },

        allUp() {
            return this.overallStatus === STATUS_PAGE_ALL_UP;
        },

        partialDown() {
            return this.overallStatus === STATUS_PAGE_PARTIAL_DOWN;
        },

        allDown() {
            return this.overallStatus === STATUS_PAGE_ALL_DOWN;
        },

    },
    watch: {

        /**
         * Selected a monitor and add to the list.
         */
        selectedMonitor(monitor) {
            if (monitor) {
                if (this.$root.publicGroupList.length === 0) {
                    this.addGroup();
                }

                const firstGroup = this.$root.publicGroupList[0];

                firstGroup.monitorList.push(monitor);
                this.selectedMonitor = null;
            }
        },

        // Set Theme
        "config.theme"() {
            this.$root.statusPageTheme = this.config.theme;
            this.loadedTheme = true;
        },

        "config.title"(title) {
            document.title = title;
        },

        "$root.monitorList"() {
            let count = Object.keys(this.$root.monitorList).length;

            // Since publicGroupList is getting from public rest api, monitors' tags may not present if showTags = false
            if (count > 0) {
                for (let group of this.$root.publicGroupList) {
                    for (let monitor of group.monitorList) {
                        if (monitor.tags === undefined && this.$root.monitorList[monitor.id]) {
                            monitor.tags = this.$root.monitorList[monitor.id].tags;
                        }
                    }
                }
            }
        }

    },
    async created() {
        this.hasToken = ("token" in this.$root.storage());

        // Browser change page
        // https://stackoverflow.com/questions/7317273/warn-user-before-leaving-web-page-with-unsaved-changes
        window.addEventListener("beforeunload", (e) => {
            if (this.editMode) {
                (e || window.event).returnValue = leavePageMsg;
                return leavePageMsg;
            } else {
                return null;
            }
        });

        // Special handle for dev
        this.baseURL = getResBaseURL();
    },
    async mounted() {
        this.slug = this.$route.params.slug;

        if (!this.slug) {
            this.slug = "default";
        }

        axios.get("/api/status-page/" + this.slug).then((res) => {
            this.config = res.data.config;

            if (this.config.icon) {
                this.imgDataUrl = this.config.icon;
            }

            this.incident = res.data.incident;
            this.$root.publicGroupList = res.data.publicGroupList;
        });

        // 5mins a loop
        this.updateHeartbeatList();
        feedInterval = setInterval(() => {
            this.updateHeartbeatList();
        }, (300 + 10) * 1000);

        // Go to edit page if ?edit present
        // null means ?edit present, but no value
        if (this.$route.query.edit || this.$route.query.edit === null) {
            this.edit();
        }
    },
    methods: {

        updateHeartbeatList() {
            // If editMode, it will use the data from websocket.
            if (! this.editMode) {
                axios.get("/api/status-page/heartbeat/" + this.slug).then((res) => {
                    const { heartbeatList, uptimeList } = res.data;

                    this.$root.heartbeatList = heartbeatList;
                    this.$root.uptimeList = uptimeList;

                    const heartbeatIds = Object.keys(heartbeatList);
                    const downMonitors = heartbeatIds.reduce((downMonitorsAmount, currentId) => {
                        const monitorHeartbeats = heartbeatList[currentId];
                        const lastHeartbeat = monitorHeartbeats.at(-1);

                        if (lastHeartbeat) {
                            return lastHeartbeat.status === 0 ? downMonitorsAmount + 1 : downMonitorsAmount;
                        } else {
                            return downMonitorsAmount;
                        }
                    }, 0);

                    favicon.badge(downMonitors);

                    this.loadedData = true;
                });
            }
        },

        edit() {
            if (this.hasToken) {
                this.$root.initSocketIO(true);
                this.enableEditMode = true;
                this.clickedEditButton = true;
            }
        },

        save() {
            let startTime = new Date();

            this.$root.getSocket().emit("saveStatusPage", this.slug, this.config, this.imgDataUrl, this.$root.publicGroupList, (res) => {
                if (res.ok) {
                    this.enableEditMode = false;
                    this.$root.publicGroupList = res.publicGroupList;

                    // Add some delay, so that the side menu animation would be better
                    let endTime = new Date();
                    let time = 100 - (endTime - startTime) / 1000;

                    if (time < 0) {
                        time = 0;
                    }

                    setTimeout(() => {
                        location.href = "/status/" + this.config.slug;
                    }, time);

                } else {
                    toast.error(res.msg);
                }
            });
        },

        deleteDialog() {
            this.$refs.confirmDelete.show();
        },

        deleteStatusPage() {
            this.$root.getSocket().emit("deleteStatusPage", this.slug, (res) => {
                if (res.ok) {
                    this.enableEditMode = false;
                    location.href = "/manage-status-page";
                } else {
                    toast.error(res.msg);
                }
            });
        },

        monitorSelectorLabel(monitor) {
            return `${monitor.name}`;
        },

        addGroup() {
            let groupName = this.$t("Untitled Group");

            if (this.$root.publicGroupList.length === 0) {
                groupName = this.$t("Services");
            }

            this.$root.publicGroupList.unshift({
                name: groupName,
                monitorList: [],
            });
        },

        discard() {
            location.href = "/status/" + this.slug;
        },

        /**
         * Crop Success
         */
        cropSuccess(imgDataUrl) {
            this.imgDataUrl = imgDataUrl;
        },

        showImageCropUploadMethod() {
            if (this.editMode) {
                this.showImageCropUpload = true;
            }
        },

        createIncident() {
            this.enableEditIncidentMode = true;

            if (this.incident) {
                this.previousIncident = this.incident;
            }

            this.incident = {
                title: "",
                content: "",
                style: "primary",
            };
        },

        postIncident() {
            if (this.incident.title == "" || this.incident.content == "") {
                toast.error(this.$t("Please input title and content"));
                return;
            }

            this.$root.getSocket().emit("postIncident", this.slug, this.incident, (res) => {

                if (res.ok) {
                    this.enableEditIncidentMode = false;
                    this.incident = res.incident;
                } else {
                    toast.error(res.msg);
                }

            });

        },

        /**
         * Click Edit Button
         */
        editIncident() {
            this.enableEditIncidentMode = true;
            this.previousIncident = Object.assign({}, this.incident);
        },

        cancelIncident() {
            this.enableEditIncidentMode = false;

            if (this.previousIncident) {
                this.incident = this.previousIncident;
                this.previousIncident = null;
            }
        },

        unpinIncident() {
            this.$root.getSocket().emit("unpinIncident", this.slug, () => {
                this.incident = null;
            });
        },

        dateFromNow(date) {
            return dayjs.utc(date).fromNow();
        },

    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.overall-status {
    font-weight: bold;
    font-size: 25px;

    .ok {
        color: $primary;
    }

    .warning {
        color: $warning;
    }

    .danger {
        color: $danger;
    }
}

h1 {
    font-size: 30px;

    img {
        vertical-align: middle;
        height: 60px;
        width: 60px;
    }
}

.main {
    transition: all ease-in-out 0.1s;

    &.edit {
        margin-left: 300px;
    }
}

.sidebar {
    position: fixed;
    left: 0;
    top: 0;
    width: 300px;
    height: 100vh;
    padding: 15px 15px 68px 15px;
    overflow-x: hidden;
    overflow-y: auto;
    border-right: 1px solid #ededed;

    .danger-zone {
        border-top: 1px solid #ededed;
        padding-top: 15px;
    }

    .sidebar-footer {
        width: 100%;
        bottom: 0;
        left: 0;
        padding: 15px;
        position: absolute;
        border-top: 1px solid #ededed;
    }
}

footer {
    text-align: center;
    font-size: 14px;
}

.description span {
    min-width: 50px;
}

.title-flex {
    display: flex;
    align-items: center;
    gap: 10px;
}

.logo-wrapper {
    display: inline-block;
    position: relative;

    &:hover {
        .icon-upload {
            transform: scale(1.2);
        }
    }

    .icon-upload {
        transition: all $easing-in 0.2s;
        position: absolute;
        bottom: 6px;
        font-size: 20px;
        left: -14px;
        background-color: white;
        padding: 5px;
        border-radius: 10px;
        cursor: pointer;
        box-shadow: 0 15px 70px rgba(0, 0, 0, 0.9);
    }
}

.logo {
    transition: all $easing-in 0.2s;

    &.edit-mode {
        cursor: pointer;

        &:hover {
            transform: scale(1.2);
        }
    }
}

.incident {
    .content {
        &[contenteditable=true] {
            min-height: 60px;
        }
    }

    .date {
        font-size: 12px;
    }
}

.mobile {
    h1 {
        font-size: 22px;
    }

    .overall-status {
        font-size: 20px;
    }
}

.dark {
    .sidebar {
        background-color: $dark-header-bg;
        border-right-color: $dark-border-color;

        .danger-zone {
            border-top-color: $dark-border-color;
        }

        .sidebar-footer {
            border-top-color: $dark-border-color;
        }
    }
}

</style>
