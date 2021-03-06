<template>
    <div
        :class="{'kiwi-serverselector--custom': presetServer === 'custom'}"
        class="kiwi-serverselector"
    >
        <div v-if="usePreset && presetNetworks.length > 0" class="kiwi-serverselector-presets">
            <label>
                <span>{{ $t('server') }}</span>
                <select v-model="presetServer">
                    <option value="custom">Custom Server</option>
                    <option disabled>-----------------</option>
                    <option
                        v-for="s in presetNetworks"
                        :key="s.name"
                        :value="toUri(s)"
                    >{{ s.name }}</option>
                </select>
            </label>
        </div>

        <template v-if="showCustom || presetNetworks.length === 0 || !usePreset">
            <input-text
                v-focus
                :label="$t('server')"
                v-model="connection.server"
                class="kiwi-networksettings-connection-address"
            />

            <input-text
                :label="$t('settings_port')"
                v-model="connection.port"
                type="number"
                class="kiwi-networksettings-connection-port"
            >
                <span
                    :class="{ 'kiwi-customserver-tls--enabled' : connection.tls }"
                    class="fa-stack fa-lg kiwi-customserver-tls"
                    @click="toggleTls"
                >
                    <i class="fa fa-lock fa-stack-1x kiwi-customserver-tls-lock"/>
                    <i
                        v-if="!connection.tls"
                        class="fa fa-unlock fa-stack-1x kiwi-customserver-tls-minus"
                    />
                </span>
            </input-text>
        </template>
    </div>
</template>

<script>

'kiwi public';

import _ from 'lodash';

export default {
    props: {
        usePreset: {
            type: Boolean,
            default: true,
        },
        networkList: {
            type: Array,
            default: () => [],
        },
        connection: {
            type: Object,
            default: () => {},
        },
    },
    data: function data() {
        return {
            name: '',
            presetNetworks: [],
            showCustom: true,
            willEmit: false,
        };
    },
    computed: {
        presetServer: {
            set(newVal) {
                if (newVal === 'custom') {
                    this.name = '';
                    this.connection.server = '';
                    this.connection.port = 6667;
                    this.connection.tls = false;

                    this.showCustom = true;
                } else {
                    let addr = this.parseFormatted(newVal);
                    this.name = addr.name;
                    this.connection.server = addr.server;
                    this.connection.port = addr.port;
                    this.connection.tls = addr.tls;

                    this.showCustom = false;
                }
            },
            get() {
                return this.showCustom ?
                    'custom' :
                    this.toUri(this);
            },
        },
    },
    created() {
        if (this.networkList) {
            this.importUris(this.networkList);
        }

        // If the given network is in the preset server list, select it
        let con = this.connection;
        if (_.find(this.presetNetworks, (s) => {
            let match = s.server === con.server && s.port === con.port && s.tls === con.tls;
            return match;
        })) {
            this.showCustom = false;
        }
    },
    methods: {
        toUri(s) {
            return `${s.server}:${s.tls ? '+' : ''}${s.port}`;
        },
        toggleTls() {
            this.connection.tls = !this.connection.tls;

            // Switching the port only if were currently using the most common TLS/plain text ports
            if (this.connection.tls && this.connection.port === 6667) {
                this.connection.port = 6697;
            } else if (!this.connection.tls && this.connection.port === 6697) {
                this.connection.port = 6667;
            }
        },
        // parseFormatted - Parse freenode|irc.freenode.net:+6697 links
        parseFormatted(input) {
            let ret = {
                name: '',
                server: '',
                port: 6667,
                tls: false,
            };

            let val = input;

            let pipePos = val.indexOf('|');
            if (pipePos > -1) {
                ret.name = val.substr(0, pipePos);
                val = val.substr(pipePos + 1);
            }

            let colonPos = val.indexOf(':');
            if (colonPos === -1) {
                ret.server = val;
                val = '';
            } else {
                ret.server = val.substr(0, colonPos);
                val = val.substr(colonPos + 1);
            }

            if (val[0] === '+') {
                ret.tls = true;
                val = val.substr(1);
            }

            if (val.length > 0) {
                ret.port = parseInt(val, 10);
                val = '';
            }

            if (!ret.name) {
                ret.name = ret.server;
            }

            return ret;
        },
        importUris(serverList) {
            // [ 'freenode|irc.freenode.net:+6697', 'irc.snoonet.org:6667' ]
            let servers = serverList.map(s => this.parseFormatted(s));
            this.$set(this, 'presetNetworks', servers);
        },
    },
};
</script>

<style>
.kiwi-serverselector-presets {
    margin-bottom: 1em;
}

.kiwi-serverselector-presets label span {
    margin-left: 0;
    transition: opacity 0.2s, width 0.2s;
}

.kiwi-serverselector--custom .kiwi-serverselector-presets label span {
    max-width: 0;
    opacity: 0;
}
</style>
