<template>
    <div>
        <div class="row">
            <div class="col-12">
                <card class="xss-card-container">
                    <div class="row pl-4 pr-4 p-2" style="display: block;">
                        <div>
                            <h1><i class="fas fa-cogs"></i> Settings</h1>
                        </div>
                        <card>
                            <h4 class="card-title">Injection Correlation API Key</h4>
                            <h6 class="card-subtitle mb-2 text-muted">Use with an XSS Hunter compatible client to track which injection caused a payload fire.</h6>
                            <p class="card-text">
                                <base-input v-bind:value="correlation_api_key" type="text" placeholder="..." ></base-input>
                            </p>
                            <base-button class="mr-1" type="primary" v-clipboard:copy="correlation_api_key" >
                                <i class="far fa-copy"></i> Copy API Key
                            </base-button>
                            <base-button type="danger" v-on:click="generate_new_correlation_api_key">
                                <i class="fas fa-sync-alt"></i> Rotate API Key
                            </base-button>
                        </card>
                        <card>
                            <h4 class="card-title">Master Password</h4>
                            <h6 class="card-subtitle mb-2 text-muted">Change your login password for this XSS Hunter express instance.</h6>
                            <p class="card-text">
                                <base-input v-model="password" type="password" placeholder="*******************"></base-input>
                            </p>
                            <base-button type="primary" v-on:click="update_password">
                                <i class="fas fa-lock"></i> Update Password
                            </base-button>
                        </card>
                        <card>
                            <h4 class="card-title">Additional JavaScript to Chainload (URL)</h4>
                            <h6 class="card-subtitle mb-2 text-muted">Remote JavaScript to retrieve and evaluate after the original payload completes.</h6>
                            <p class="card-text">
                                <base-input v-model="chainload_uri" type="text" placeholder="https://example.com/remote.js"></base-input>
                            </p>
                            <base-button type="primary" v-on:click="update_chainload_uri">
                                <i class="far fa-save"></i> Save JavaScript URL
                            </base-button>
                        </card>
                        <card>
                            <h4 class="card-title">Pages to Collect on Payload Fire</h4>
                            <h6 class="card-subtitle mb-2 text-muted">List of relative paths to collect from a site when a payload fires.</h6>
                            <div class="form-row">
                                <div class="col">
                                    <base-input class="mt-1" type="text" placeholder="/robots.txt" v-model="new_page_to_collect" />
                                </div>
                                <div class="col">
                                    <base-button type="primary" v-on:click="add_new_page_to_collect">
                                        <i class="fas fa-plus"></i> Add New Path to Collect
                                    </base-button>
                                </div>
                            </div>
                            <p class="card-text">
                                <base-input >
                                    <select multiple class="form-control" v-model="selected_page_to_collect">
                                        <option v-for="page_to_collect in pages_to_collect">{{page_to_collect}}</option>
                                    </select>
                                </base-input>
                            </p>
                            <base-button type="danger" class="btn-sm mt-0 mb-3" v-on:click="delete_selected_pages_to_collect">
                                <i class="fas fa-trash-alt"></i> Delete Selected Path(s)
                            </base-button>
                            <br />
                            <base-button type="primary" v-on:click="update_pages_to_collect">
                                <i class="far fa-save"></i> Update Pages to Collect List
                            </base-button>
                        </card>
                        <card>
                            <h4 class="card-title">Miscellaneous Options</h4>
                            <div>
                                <base-button 
                                    :type="discord_notifications_enabled ? 'success' : 'danger'"
                                    v-on:click="toggle_discord_notifications"
                                >
                                    <i :class="discord_notifications_enabled ? 'far fa-bell' : 'far fa-bell-slash'"></i>
                                    Discord Notifications: {{ discord_notifications_enabled === true ? 'Enabled' : 'Disabled' }}
                                </base-button>
                                <h6 class="mt-2 text-muted">
                                    {{ discord_notifications_enabled === true ? 
                                        'XSS payload fire reports are being sent to Discord.' : 
                                        'XSS payload fire reports are not being sent to Discord.' 
                                    }}
                                </h6>
                            </div>
                            <hr />
                            <div class="mt-3">
                                <base-button type="primary" v-on:click="revoke_all_sessions">
                                    <i class="fas fa-sign-out-alt"></i> Revoke All Active Sessions
                                </base-button>
                                <h6 class="mt-2 text-muted">
                                    Revoke all active sessions, forcing all users to re-authenticate.
                                </h6>
                            </div>
                        </card>
                    </div>
                </card>
            </div>
        </div>
    </div>
</template>
<script>
import config from '@/config';
import Vue from "vue";
import api_request from '@/libs/api.js';
import router from "@/router/index";
import utils from '@/libs/utils';
import BaseAlert from '@/components/BaseAlert.vue';

export default {
    data() {
        return {
            base_api_path: '',
            rate_limit_options: [
                {
                    'text': 'No rate limiting',
                    'seconds': -1,
                },
                {
                    'text': 'One payload fire a minute',
                    'seconds': 60,
                },
                {
                    'text': 'Two payload fires a minute',
                    'seconds': 30,
                },
                {
                    'text': 'Five payload fires a minute',
                    'seconds': 12,
                },
                {
                    'text': 'One payload fire every five minutes',
                    'seconds': 300,
                },
                {
                    'text': 'One payload fire every ten minutes',
                    'seconds': 600,
                },
                {
                    'text': 'One payload fire every half-hour',
                    'seconds': 1800,
                },
                {
                    'text': 'One payload fire every hour',
                    'seconds': 3600,
                },
                {
                    'text': 'One payload fire every two hours',
                    'seconds': 7200,
                },
                {
                    'text': 'One payload fire every five hours',
                    'seconds': 18000,
                },
                {
                    'text': 'One payload fire every ten hours',
                    'seconds': 36000,
                },
                {
                    'text': 'One payload fire every day',
                    'seconds': 86400,
                },
            ],
            chainload_uri: '',
            correlation_api_key: '',
            pages_to_collect: [],
            selected_page_to_collect: [],
            new_page_to_collect: '',
            rate_limit: -1,
            password: '',
            discord_notifications_enabled: true,
        }
    },
    watch: {},
    methods: {
        toggle_discord_notifications: async function() {
            try {
                const response = await api_request.set_discord_notifications(!this.discord_notifications_enabled);
                if (response.success) {
                    await this.pull_latest_settings();
                    const status = this.discord_notifications_enabled ? 'enabled' : 'disabled';
                    toastr.success(`Discord notifications have been ${status}`, 'Success');
                } else {
                    toastr.error('Failed to update Discord notification settings', 'Error');
                }
            } catch (error) {
                console.error('Error toggling Discord notifications:', error);
                toastr.error('Failed to update Discord notification settings', 'Error');
            }
        },
        update_password: async function() {
            const password = this.password;
            if(password === '') {
                alert('Password is empty, please provide a valid password to continue.');
                return
            }
            await api_request.update_password(this.password);
            this.password = '';
            toastr.success('Your instance password has been updated.', 'Password Updated')
        },
        generate_new_correlation_api_key: async function() {
            await api_request.generate_new_correlation_api_key();
            await this.pull_latest_settings();
            toastr.success('Your correlated injection API key has been rotated.', 'Correlated Injection API Key Rotated')
        },
        pull_latest_settings: async function() {
            try {
                const response = await api_request.get_settings();
                if (response.success) {
                    this.discord_notifications_enabled = response.result.discord_notifications_enabled === 'true';
                    const settings_keys = [
                        'chainload_uri',
                        'correlation_api_key',
                        'pages_to_collect',
                        'rate_limit',
                    ];

                    // Pull settings
                    const settings = response.result;
                    settings_keys.map(settings_key => {
                        this[settings_key] = settings[settings_key];
                    });
                }
            } catch (error) {
                console.error('Error fetching settings:', error);
                toastr.error('Failed to fetch settings', 'Error');
            }
        },
        update_chainload_uri: async function() {
            await api_request.set_chainload_uri(this.chainload_uri);
            await this.pull_latest_settings();
            toastr.success('Your chainload URI has been updated.', 'Chainload URI Updated')
        },
        revoke_all_sessions: async function() {
            await api_request.revoke_all_sessions();
            toastr.success('All active sessions have been revoked.', 'All Active Sessions Revoked')
            window.location.reload();
        },
        add_new_page_to_collect: async function() {
            var new_page = this.new_page_to_collect;
            if(!new_page.startsWith('/')) {
                new_page = '/' + new_page;
            }
            this.pages_to_collect.push(new_page);
            this.new_page_to_collect = '';
        },
        delete_selected_pages_to_collect: async function() {
            this.pages_to_collect = this.pages_to_collect.filter(page_to_collect => {
                return !this.selected_page_to_collect.includes(page_to_collect);
            });
        },
        update_pages_to_collect: async function() {
            await api_request.update_pages_to_collect(this.pages_to_collect);
            await this.pull_latest_settings();
            toastr.success('Your pages to collect have been updated.', 'Pages to Collect Updated')
        }
    },
    computed: {
        rate_limit_text: function() {
            const matching_options = this.rate_limit_options.filter(option_data => {
                return option_data.seconds === this.rate_limit;
            });
            if(matching_options.length === 0) {
                return 'No rate limiting';
            }
            return matching_options[0].text;
        }
    },
    components: {
        BaseAlert,
    },
    async mounted() {
        // For debugging
        window.app = this;

        // For rendering
        this.base_api_path = api_request.BASE_API_PATH;

        await this.pull_latest_settings();
    },
    beforeDestroy() {}
};
</script>
<style>
.dropdown-item {
    font-size: 16px !important;
}
</style>