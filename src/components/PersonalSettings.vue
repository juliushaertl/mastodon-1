<template>
    <div id="mastodon_prefs" class="section">
            <h2>
                <a class="icon icon-mastodon"></a>
                {{ t('mastodon', 'Mastodon') }}
            </h2>
            <div class="mastodon-grid-form">
                <label for="mastodon-url">
                    <a class="icon icon-link"></a>
                    {{ t('mastodon', 'Mastodon instance address') }}
                </label>
                <input id="mastodon-url" type="text" v-model="state.url" @input="onInput"
                    :placeholder="t('mastodon', 'Mastodon instance URL')"/>
                <button id="mastodon-oauth" @click="onOAuthClick">
                    <span class="icon icon-external"/>
                    {{ t('mastodon', 'Get access with OAuth') }}
                </button>
                <label for="mastodon-token">
                    <a class="icon icon-category-auth"></a>
                    {{ t('mastodon', 'Mastodon access token') }}
                </label>
                <input id="mastodon-token" type="password" v-model="state.token" @input="onInput"
                    :placeholder="t('mastodon', 'Authenticate with OAuth')"/>
            </div>
    </div>
</template>

<script>
import { loadState } from '@nextcloud/initial-state'
import { generateUrl, imagePath } from '@nextcloud/router'
import axios from '@nextcloud/axios'
import { delay } from '../utils'
import { showSuccess, showError } from '@nextcloud/dialogs'

export default {
    name: 'PersonalSettings',

    props: [],
    components: {
    },

    mounted() {
        const paramString = window.location.search.substr(1)
        const urlParams = new URLSearchParams(paramString)
        const mToken = urlParams.get('mastodonToken')
        if (mToken === 'success') {
            showSuccess(t('mastodon', 'Mastodon OAuth access token successfully retrieved!'))
        } else if (mToken === 'error') {
            showError(t('mastodon', 'Mastodon OAuth access token could not be obtained:') + ' ' + urlParams.get('message'))
        }
    },

    data() {
        return {
            state: loadState('mastodon', 'user-config'),
        }
    },

    watch: {
    },

    methods: {
        onInput() {
            const that = this
            delay(function() {
                that.saveOptions()
            }, 2000)()
        },
        saveOptions() {
            if (!this.state.url.startsWith('https://')) {
                if (this.state.url.startsWith('http://')) {
                    this.state.url = this.state.url.replace('http://', 'https://')
                } else {
                    this.state.url = 'https://' + this.state.url
                }
            }
            const req = {
                values: {
                    token: this.state.token,
                    url: this.state.url
                }
            }
            const url = generateUrl('/apps/mastodon/config')
            axios.put(url, req)
                .then(function (response) {
                    showSuccess(t('mastodon', 'Mastodon options saved.'))
                })
                .catch(function (error) {
                    showError(t('mastodon', 'Failed to save Mastodon options') +
                        ': ' + error.response.request.responseText
                    )
                })
                .then(function () {
                })
        },
        onOAuthClick() {
            // first we need to add an app to the target instance
            // so we get client_id and client_secret
            const redirect_endpoint = generateUrl('/apps/mastodon/oauth-redirect')
            const redirect_uri = OC.getProtocol() + '://' + OC.getHostName() + redirect_endpoint
            //const redirect_uri = OC.getProtocol() + '://' + OC.getHostName()
            const req = {
                redirect_uris: redirect_uri,
            }
            const url = generateUrl('/apps/mastodon/oauth-app')
            axios.post(url, req)
                .then((response) => {
                    this.oAuthStep1(response.data.client_id)
                })
                .catch((error) => {
                    showError(t('mastodon', 'Failed to add Mastodon OAuth app') +
                        ': ' + error.response.request.responseText
                    )
                })
                .then(function () {
                })
        },
        oAuthStep1(client_id) {
            // redirect to '/oauth/auhorize' api endpoint to get a code
            const redirect_endpoint = generateUrl('/apps/mastodon/oauth-redirect')
            const redirect_uri = OC.getProtocol() + '://' + OC.getHostName() + redirect_endpoint
            //const redirect_uri = OC.getProtocol() + '://' + OC.getHostName()
            const request_url = this.state.url + '/oauth/authorize?client_id=' + encodeURIComponent(client_id) +
                '&redirect_uri=' + encodeURIComponent(redirect_uri) +
                '&response_type=code' +
                '&scope=' + encodeURIComponent('read write follow')
            window.location.replace(request_url)
        }
    }
}
</script>

<style scoped lang="scss">
.mastodon-grid-form label {
    line-height: 38px;
}
.mastodon-grid-form input {
    width: 100%;
}
.mastodon-grid-form {
    width: 700px;
    display: grid;
    grid-template: 1fr / 233px 233px 300px;
    margin-left: 30px;
    button .icon {
        margin-bottom: -1px;
    }
}
#mastodon_prefs .icon {
    display: inline-block;
    width: 32px;
}
#mastodon_prefs .grid-form .icon {
    margin-bottom: -3px;
}
.icon-mastodon {
    background-image: url(./../../img/app-dark.svg);
    background-size: 23px 23px;
    height: 23px;
    margin-bottom: -4px;
}
body.dark .icon-mastodon {
    background-image: url(./../../img/app.svg);
}
</style>
