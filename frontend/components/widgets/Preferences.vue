<template>
    <!-- Offcanvas Preferences -->
    <div :data-bs-theme="isDarkMode ? 'dark' : 'light'" :class="[isMobile ? ' w-100' : '']"
        class="offcanvas offcanvas-start h-100 border-0 mt-5" tabindex="-1" id="offcanvasPreferences"
        aria-labelledby="offcanvasPreferencesLabel">
        <div class="offcanvas-header mt-3">
            <h5 class="offcanvas-title"><i class="bi bi-toggles"></i>&nbsp;&nbsp;{{
                t('nav.preferences.title') }}
            </h5>
            <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"></button>
        </div>
        <div class="offcanvas-body pt-0 m-2">
            <div class="preferences-tip">{{ t('nav.preferences.preferenceTips') }}</div>

            <!-- 语言设置 -->

            <div id="Pref_language">
                <div class="form-label col-12 preferences-title"><i class="bi bi-translate"></i> {{
                    t('nav.preferences.language') }}</div>
                <div class="btn-group-vertical col-auto w-50 mb-2" role="group" aria-label="Color Scheme">
                    <template v-for="lang in ['auto','zh', 'en', 'fr']">
                        <input type="radio" class="btn-check" :name="'language' + lang" :id="'language' + lang"
                            autocomplete="off" :value="lang" v-model="userPreferences.lang"
                            @change="prefLanguage(lang)">
                        <label class="btn jn-number text-start" :class="{
                            'btn-outline-dark': !isDarkMode,
                            'btn-outline-light': isDarkMode,
                            'active fw-bold': userPreferences.lang === lang
                        }" :for="'language' + lang">
                            <span v-if="lang === 'zh'"><i class="fi fi-cn"></i> 中文&nbsp;</span>
                            <span v-else-if="lang === 'en'"><i class="fi fi-us"></i> English&nbsp;</span>
                            <span v-else-if="lang === 'fr'"><i class="fi fi-fr"></i> Français&nbsp;</span>
                            <span v-else>{{ t('nav.preferences.systemAuto') }}&nbsp;</span>
                            <i class="bi bi-check2-circle" v-if="userPreferences.lang === lang"></i>
                        </label>
                    </template>
                </div>
                <div class="preferences-tip">{{ t('nav.preferences.languageTips') }}</div>
            </div>

            <!-- 主题方案 -->

            <div id="Pref_colorScheme">
                <div class="form-label col-12 preferences-title"><i class="bi bi-palette-fill"></i> {{
                    t('nav.preferences.colorScheme') }}</div>
                <div class="btn-group col-auto" role="group" aria-label="Color Scheme">
                    <template v-for="theme in ['auto', 'light', 'dark']">
                        <input type="radio" class="btn-check" :name="'darkMode' + theme" :id="'darkMode' + theme"
                            autocomplete="off" :value="theme" v-model="userPreferences.theme"
                            @change="prefTheme(theme)">
                        <label class="btn" :class="{
                            'btn-outline-dark': !isDarkMode,
                            'btn-outline-light': isDarkMode,
                            'active fw-bold': userPreferences.theme === theme
                        }" :for="'darkMode' + theme">
                            <span v-if="theme === 'light'"><i class="bi bi-brightness-high"></i> {{
                                t('nav.preferences.colorLight') }}</span>
                            <span v-else-if="theme === 'dark'"><i class="bi bi-moon-stars"></i> {{
                                t('nav.preferences.colorDark') }}</span>
                            <span v-else>{{ t('nav.preferences.systemAuto') }}</span>
                        </label>
                    </template>
                </div>
            </div>

            <!-- IP 源 -->

            <div id="Pref_ipCards">
                <div class="form-label col-12 preferences-title">
                    <i class="bi bi-ui-checks-grid"></i> {{ t('nav.preferences.ipSourcesToCheck') }}
                </div>
                <div class="btn-group col-auto w-50 mb-2" role="group" aria-label="ipCards">
                    <template v-for="num in [3, 6]">
                        <input v-model="userPreferences.ipCardsToShow" type="radio" class="btn-check"
                            :name="'ipCards_' + num" :id="'ipCards_' + num" autocomplete="off" :value=num
                            @change="prefipCards(num)">
                        <label class="btn jn-number" :class="{
                            'btn-outline-dark': !isDarkMode,
                            'btn-outline-light': isDarkMode,
                            'active fw-bold': userPreferences.ipCardsToShow === num
                        }" :for="'ipCards_' + num">{{ num
                            }}</label>
                    </template>
                </div>
                <div class="preferences-tip">{{ t('nav.preferences.ipSourcesToCheckTips') }}</div>
            </div>

            <!-- IP 地理位置数据库 -->

            <div id="Pref_ipGeoSource">
                <div class="form-label col-12 preferences-title">
                    <i class="bi bi-ui-checks-grid"></i> {{ t('nav.preferences.ipDB') }}
                </div>
                <div class="btn-group-vertical col-auto w-50 mb-2" role="group" aria-label="ipGeoSource">
                    <template v-for="ipdb in ipDBs">
                        <input v-model="userPreferences.ipGeoSource" type="radio" class="btn-check"
                            :name="'ipGeoSource_' + ipdb.text" :id="'ipGeoSource_' + ipdb.id" autocomplete="off"
                            :value=ipdb.id @change="prefipGeoSource(ipdb.id)">
                        <label class="btn jn-number text-start" :class="{
                            'btn-outline-dark': !isDarkMode,
                            'btn-outline-light': isDarkMode,
                            'active fw-bold': userPreferences.ipGeoSource === ipdb.id,
                            'jn-disabled-button': !ipdb.enabled
                        }" :for="'ipGeoSource_' + ipdb.id" :aria-disabled="!ipdb.enabled" :aria-label="ipdb.text">
                            <span :class="[ipdb.enabled ? '' : 'jn-disabled-text']">{{ ipdb.text }}&nbsp;</span>
                            <i class="bi bi-check2-circle" v-if="userPreferences.ipGeoSource === ipdb.id"></i>
                        </label>
                    </template>
                </div>
                <div class="preferences-tip">{{ t('nav.preferences.ipDBTips') }}</div>
            </div>

            <!-- 应用设置 -->

            <div id="Pref_appSettings">
                <div class="form-label col-12 preferences-title"><i class="bi bi-window-dock"></i> {{
                    t('nav.preferences.appSettings') }}</div>
                <ul class="list-group">
                    <li class="list-group-item d-flex justify-content-between align-items-start"
                        :class="[isDarkMode ? 'border-light' : 'border-dark']">
                        <div class="me-auto">
                            <div class="fw-bold"><label class="form-check-label" for="autoStart">{{
                                    t('nav.preferences.autoRun')
                                    }}</label>
                            </div>
                            <div class="preferences-tip">{{ t('nav.preferences.autoRunTips') }}</div>
                        </div>
                        <div class="form-check form-switch col-auto ">
                            <input class="form-check-input" :class="[isDarkMode ? 'jn-check-dark' : 'jn-check-light']"
                                type="checkbox" role="switch" id="autoStart" :checked="userPreferences.autoStart"
                                @change="prefAutoStart($event.target.checked)">
                        </div>
                    </li>

                    <li class="list-group-item d-flex justify-content-between align-items-start"
                        :class="[isDarkMode ? 'border-light' : 'border-dark']" v-if="userPreferences.autoStart">
                        <div class="me-auto">
                            <div class="fw-bold"><label class="form-check-label" for="ConnectivityRefresh">{{
                                    t('nav.preferences.connectivityAutoRefresh') }}</label></div>
                            <div class="preferences-tip">{{ t('nav.preferences.connectivityAutoRefreshTips') }}</div>
                        </div>
                        <div class="form-check form-switch col-auto ">
                            <input class="form-check-input" :class="[isDarkMode ? 'jn-check-dark' : 'jn-check-light']"
                                type="checkbox" role="switch" id="ConnectivityRefresh"
                                :checked="userPreferences.connectivityAutoRefresh"
                                @change="prefConnectivityRefresh($event.target.checked)">
                        </div>
                    </li>

                    <li class="list-group-item d-flex justify-content-between align-items-start"
                        :class="[isDarkMode ? 'border-light' : 'border-dark']" v-if="configs.map">
                        <div class="me-auto">
                            <div class="fw-bold"><label class="form-check-label" for="showMap">{{
                                    t('nav.preferences.showMap')
                                    }}</label>
                            </div>
                            <div class="preferences-tip">{{ t('nav.preferences.showMapTips') }}</div>
                        </div>
                        <div class="form-check form-switch col-auto ">
                            <input class="form-check-input" :class="[isDarkMode ? 'jn-check-dark' : 'jn-check-light']"
                                type="checkbox" role="switch" id="showMap" :checked="userPreferences.showMap"
                                @change="prefShowMap($event.target.checked)">
                        </div>
                    </li>

                    <li class="list-group-item d-flex justify-content-between align-items-start"
                        :class="[isDarkMode ? 'border-light' : 'border-dark']" v-if="isMobile">
                        <div class="me-auto">
                            <div class="fw-bold"><label class="form-check-label" for="simpleMode">{{
                                    t('nav.preferences.simpleMode')
                                    }}</label></div>
                            <div class="preferences-tip">{{ t('nav.preferences.simpleModeTips') }}</div>
                        </div>
                        <div class="form-check form-switch col-auto ">
                            <input class="form-check-input" :class="[isDarkMode ? 'jn-check-dark' : 'jn-check-light']"
                                type="checkbox" role="switch" id="simpleMode" :checked="userPreferences.simpleMode"
                                @change="prefSimpleMode($event.target.checked)">
                        </div>
                    </li>

                    <li class="list-group-item d-flex justify-content-between align-items-start"
                        :class="[isDarkMode ? 'border-light' : 'border-dark']">
                        <div class="me-auto">
                            <div class="fw-bold"><label class="form-check-label" for="ConnectivityNotifications">{{
                                    t('nav.preferences.popupConnectivityNotifications') }}</label>
                            </div>
                            <div class="preferences-tip">{{ t('nav.preferences.popupConnectivityNotificationsTips') }}
                            </div>
                        </div>
                        <div class="form-check form-switch col-auto ">
                            <input class="form-check-input" :class="[isDarkMode ? 'jn-check-dark' : 'jn-check-light']"
                                type="checkbox" role="switch" id="ConnectivityNotifications"
                                :checked="userPreferences.popupConnectivityNotifications"
                                @change="prefconnectivityShowNoti($event.target.checked)">
                        </div>
                    </li>

                </ul>
            </div>

            <div id="offcanvasPlaceholder mb-5" class="jn-placeholder mb-5">
            </div>

        </div>
    </div>

</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useMainStore } from '@/store';
import { useI18n } from 'vue-i18n';
import { trackEvent } from '@/utils/use-analytics';

const { t } = useI18n();

const store = useMainStore();
const isDarkMode = computed(() => store.isDarkMode);
const isMobile = computed(() => store.isMobile);
const configs = computed(() => store.configs);
const userPreferences = computed(() => store.userPreferences);
const ipDBs = computed(() => store.ipDBs);
const isSignedIn = computed(() => store.isSignedIn);

const prefersDarkMode = ref(window.matchMedia('(prefers-color-scheme: dark)').matches);
const mediaQueryList = window.matchMedia('(prefers-color-scheme: dark)');

const handleThemeChange = (event) => {
    prefersDarkMode.value = event.matches;
    const theme = userPreferences.value.theme;
    if (theme === 'auto') {
        store.setDarkMode(prefersDarkMode.value);
    } else if (theme === 'light') {
        store.setDarkMode(false);
    } else if (theme === 'dark') {
        store.setDarkMode(true);
    }
    updateBodyClass();
    PWAColor();
};

const updateBodyClass = () => {
    document.body.classList.toggle("body-dark-mode", isDarkMode.value);
};

const PWAColor = () => {
    const themeColor = document.querySelector('meta[name="theme-color"]');
    const backgroundColor = document.querySelector('meta[name="background-color"]');
    const color = isDarkMode.value ? "#171a1d" : "#f8f9fa";
    const bgColor = isDarkMode.value ? "#212529" : "#ffffff";
    themeColor.setAttribute("content", color);
    backgroundColor.setAttribute("content", bgColor);
};

const updateIPDBs = () => {
    if (configs.value && Object.keys(configs.value).length > 0) {
        store.updateIPDBs({ id: 0, enabled: configs.value.ipChecking });
        store.updateIPDBs({ id: 1, enabled: configs.value.ipInfo });
        store.updateIPDBs({ id: 3, enabled: configs.value.ipapiis });
        store.updateIPDBs({ id: 4, enabled: configs.value.ip2location });
    }
};

const prefTheme = (value) => {
    switch (value) {
        case 'light':
            store.setDarkMode(false);
            break;
        case 'dark':
            store.setDarkMode(true);
            break;
        case 'auto':
            handleThemeChange({ matches: mediaQueryList.matches });
            break;
    }
    updateBodyClass();
    PWAColor();
    store.updatePreference('theme', value);
    trackEvent('Nav', 'PreferenceClick', 'Theme');
};

const prefLanguage = (value) => {
    store.updatePreference('lang', value);
    trackEvent('Nav', 'PrefereceClick', 'LanguageChange');
};

const prefConnectivityRefresh = (value) => {
    store.updatePreference('connectivityAutoRefresh', value);
    if (isSignedIn.value && !store.userAchievements.ResourceHog.achieved) {
        store.setTriggerUpdateAchievements('ResourceHog');
    }
    trackEvent('Nav', 'PrefereceClick', 'ConnectivityRefresh');
};

const prefShowMap = (value) => {
    store.updatePreference('showMap', value);
    trackEvent('Nav', 'PrefereceClick', 'ShowMap');
};

const prefSimpleMode = (value) => {
    store.updatePreference('simpleMode', value);
    trackEvent('Nav', 'PrefereceClick', 'SimpleMode');
};

const prefAutoStart = (value) => {
    store.updatePreference('autoStart', value);
    trackEvent('Nav', 'PrefereceClick', 'AutoStart');
    if (isSignedIn.value && !value && !store.userAchievements.EnergySaver.achieved) {
        store.setTriggerUpdateAchievements('EnergySaver');
    }
};

const prefconnectivityShowNoti = (value) => {
    store.updatePreference('popupConnectivityNotifications', value);
    trackEvent('Nav', 'PrefereceClick', 'ConnectivityNotifications');
};

const prefipCards = (value) => {
    store.updatePreference('ipCardsToShow', value);
    trackEvent('Nav', 'PrefereceClick', 'ipCards');
};

const prefipGeoSource = (value) => {
    store.updatePreference('ipGeoSource', value);
    trackEvent('Nav', 'PrefereceClick', 'ipGeoSource');
    trackEvent('IPCheck', 'SelectSource', ipDBs.value.find(x => x.id === value).text);
};

const toggleMaps = () => {
    store.updatePreference('showMap', !userPreferences.value.showMap);
    trackEvent('Nav', 'ToggleClick', 'ShowMap');
};

onMounted(() => {
    mediaQueryList.addEventListener('change', handleThemeChange);
    handleThemeChange({ matches: mediaQueryList.matches });
    setTimeout(updateIPDBs, 4000);
});

defineExpose({
    toggleMaps,
});
</script>

<style scoped>
.preferences-title {
    margin-top: 12pt;
    font-weight: 500;
    margin-bottom: 12pt;
}

.preferences-tip {
    font-size: smaller;
    opacity: 0.7;
    margin-top: 3pt;
}

.jn-number {
    min-width: 40pt;
}

.jn-margin {
    margin-top: 42pt;
}

.jn-placeholder {
    height: 20pt;
}

.jn-disabled-text {
    opacity: 0.5;
    text-decoration: line-through;
}

.jn-disabled-button {
    pointer-events: none;
}

#offcanvasPreferences {
    z-index: 1053;
}
</style>