<!--
  - This file is part of LiquidBounce (https://github.com/CCBlueX/LiquidBounce)
  -
  - Copyright (c) 2024 CCBlueX
  -
  - LiquidBounce is free software: you can redistribute it and/or modify
  - it under the terms of the GNU General Public License as published by
  - the Free Software Foundation, either version 3 of the License, or
  - (at your option) any later version.
  -
  - LiquidBounce is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU General Public License for more details.
  -
  - You should have received a copy of the GNU General Public License
  - along with LiquidBounce. If not, see <https://www.gnu.org/licenses/>.
  -
  -
  -->

<script>
    import {connectServer, getProtocol, getProtocols, getServers, setProtocol} from "../../client/api.svelte";
    import {listen} from "../../client/ws.svelte"

    let servers = [];
    let selectedServer = null;

    let protocols = [];
    let selectedProtocol = null;

    function updateServers() {
        servers = [];

        getServers().then((data) => {
            /**
             *         addProperty("name", name)
             *         addProperty("address", address)
             *         addProperty("online", online)
             *         add("playerList", protocolGson.toJsonTree(playerListSummary))
             *         add("label", protocolGson.toJsonTree(label))
             *         add("playerCountLabel", protocolGson.toJsonTree(playerCountLabel))
             *         add("version", protocolGson.toJsonTree(version))
             *         addProperty("protocolVersion", protocolVersion)
             *         add("players", JsonObject().apply {
             *             addProperty("max", players?.max)
             *             addProperty("online", players?.online)
             *         })
             *
             *         favicon?.let {
             *             addProperty("icon", Base64.getEncoder().encodeToString(it))
             *         }
             */

            servers = data;
        });
    }

    function refreshProtocols() {
        getProtocols().then((data) => {
            // data [{ name: string, version: int }, ...]
            protocols = data;

            getProtocol().then((data) => {
                selectedProtocol = data.version;
            });
        });
    }

    function updateProtocol() {
        setProtocol(selectedProtocol);
    }

    function connect() {
        if (selectedServer === null) {
            console.warn("No server selected");
            return;
        }

        // Connect to the selected server
        connectServer(selectedServer);
    }

    listen("serverPinged", (event) => {
        const server = event.server;

        for (let i = 0; i < servers.length; i++) {
            if (servers[i].address === server.address) {
                servers[i] = server;
                break;
            }
        }
    });

    refreshProtocols();
    updateServers();
</script>

<main>
    <h1>Server</h1>
    <p>Choose a server to connect to.</p>
    <select bind:value={selectedServer}>
        {#each servers as server}
            <option value={server.address}>
                {server.name} ({server.address})

                {server.label}

                {#if server.online}
                    - {server.players.online}/{server.players.max}
                {/if}

                {#if server.icon}
                    <img src="data:image/png;base64,{server.icon}" alt="Server icon" />
                {/if}
            </option>
        {/each}
    </select>
    <button on:click={updateServers}>Refresh</button>
    <button on:click={connect}>Connect</button>

    <h1>Protocol</h1>
    <p>Choose a protocol to use.</p>
    <select bind:value={selectedProtocol} on:change={updateProtocol}>
        {#each protocols as protocol}
            <option value={protocol.version}>{protocol.name} ({protocol.version})</option>
        {/each}
    </select>
</main>
