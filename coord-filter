(async function () {
    'use strict';

    if (!/screen=map/.test(location.href)) {
        const world = window.location.href.match(/https:\/\/(.*?\.tribalwars\.[^\/]+)/);
        if (world) {
            window.location.href = `https://${world[1]}/game.php?screen=map`;
        } else {
            UI.ErrorMessage("This script can only be used on the map screen!");
        }
        return;
    }

    window.scriptConfig = {
        scriptData: {
            prefix: 'coordFilter',
            name: 'Coord Filter',
            version: '2.0',
            author: 'Giovanni Livio',
            authorUrl: 'https://github.com/glivio21',
            helpLink: 'https://forum.tribalwars.net/',
        },
        allowedScreens: ['map'],
        isDebug: false,
        enableCountApi: true,
    };

    const en = {
        coordinatesFilter: 'Filtrar Coordenadas',
        clansLabel: 'Tribes (use ;):',
        playersLabel: 'Players (use ;):',
        xMinLabel: 'X min:', xMaxLabel: 'X max:',
        yMinLabel: 'Y min:', yMaxLabel: 'Y max:',
        runButton: 'Start Filtro',
        separatorLabel: 'Formato:',
        playersNotFound: 'Players not found:',
        tribesNotFound: 'Tribos not found:',
        clansPlaceholder: 'Tribo1;Tribo2',
        playersPlaceholder: 'Player1;Player2'
    };

    const L = en;

    const loadTwSDK = () => new Promise((resolve, reject) => {
        if (window.twSDK) return resolve();
        const script = document.createElement('script');
        script.src = 'https://twscripts.dev/scripts/twSDK.js';
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
    });

    const parseList = input => input.split(/[;\n]/).map(s => s.trim()).filter(Boolean);

    const filterCoords = (villages, xmin, xmax, ymin, ymax, sep) =>
        villages.map(v => `${v[2]}${sep}${v[3]}`)
                .filter(c => {
                    const [x, y] = c.split(sep).map(Number);
                    return x >= xmin && x <= xmax && y >= ymin && y <= ymax;
                });

    const createUI = () => {
        const container = document.querySelector('#map_config') || document.body;
        const tbl = document.createElement('table');
        tbl.className = 'vis';
        tbl.style.width = '100%';
        tbl.innerHTML = `
<tr><th colspan="4">${L.coordinatesFilter}</th></tr>
<tr><td colspan="4"><label>${L.clansLabel}</label><br><input id="coordFilterTribes" type="text" style="width: 98%;" placeholder="${L.clansPlaceholder}"></td></tr>
<tr><td colspan="4"><label>${L.playersLabel}</label><br><input id="coordFilterPlayers" type="text" style="width: 98%;" placeholder="${L.playersPlaceholder}"></td></tr>
<tr>
  <td><label>${L.xMinLabel}</label><br><input id="coordFilterXmin" type="number" style="width: 95%;"></td>
  <td><label>${L.xMaxLabel}</label><br><input id="coordFilterXmax" type="number" style="width: 95%;"></td>
  <td><label>${L.yMinLabel}</label><br><input id="coordFilterYmin" type="number" style="width: 93%;"></td>
  <td><label>${L.yMaxLabel}</label><br><input id="coordFilterYmax" type="number" style="width: 93%;"></td>
</tr>
<tr>
  <td colspan="2"><label>${L.separatorLabel}</label><br>
    <select id="coordFilterSeparator" style="width: 100%;">
      <option value="|">xxx|yyy</option>
      <option value="," >xxx,yyy</option>
    </select>
  </td>
  <td colspan="2">
    <button id="coordFilterRun" class="btn float_right" style="margin-top: 5px; width: 100%;">${L.runButton}</button>
  </td>
</tr>
<tr>
  <td colspan="4">
    <textarea id="coordFilterResult" rows="6" style="width:98%; margin-top: 5px; margin-bottom: 2px;" readonly placeholder="xxx|yyy xxx|yyy..."></textarea>
    <p style="text-align:right; margin: 2px 0 0;"><small>by <span style="color: purple;"><b>Giovanni Livio</b></span></small></p>
  </td>
</tr>`;

        container.appendChild(tbl);
        document.getElementById('coordFilterRun').addEventListener('click', runFilter);
    };

    const runFilter = async () => {
        const tribesInput = parseList(document.getElementById('coordFilterTribes').value);
        const playersInput = parseList(document.getElementById('coordFilterPlayers').value);
        const separator = document.getElementById('coordFilterSeparator').value;

        const getVal = (id, def) => {
            const v = document.getElementById(id).value.trim();
            return v === '' ? def : parseInt(v, 10);
        };

        const xmin = getVal('coordFilterXmin', -Infinity);
        const xmax = getVal('coordFilterXmax', Infinity);
        const ymin = getVal('coordFilterYmin', -Infinity);
        const ymax = getVal('coordFilterYmax', Infinity);

        const [villages, players, tribes] = await Promise.all([
            twSDK.worldDataAPI('village'),
            twSDK.worldDataAPI('player'),
            twSDK.worldDataAPI('ally')
        ]);

        const missingTribes = tribesInput.filter(name => !tribes.some(t => t[2]?.toLowerCase() === name.toLowerCase()));
        const missingPlayers = playersInput.filter(name => !players.some(p => p[1]?.toLowerCase() === name.toLowerCase()));

        if (missingTribes.length > 0 || missingPlayers.length > 0) {
            document.getElementById('coordFilterResult').value = '';
            let msgParts = [];
            if (missingTribes.length > 0) msgParts.push(`${L.tribesNotFound} ${missingTribes.join(', ')}`);
            if (missingPlayers.length > 0) msgParts.push(`${L.playersNotFound} ${missingPlayers.join(', ')}`);
            UI.ErrorMessage(msgParts.join(' | '));
            return;
        }

        const playerIds = twSDK.getEntityIdsByArrayIndex(playersInput, players, 1);
        const tribeIds = twSDK.getEntityIdsByArrayIndex(tribesInput, tribes, 2);
        const tribePlayerIds = twSDK.getTribeMembersById(tribeIds, players);
        const allPlayerIds = [...new Set([...playerIds, ...tribePlayerIds])];

        const matchingVillages = villages.filter(v => allPlayerIds.includes(parseInt(v[4])));
        const filteredCoords = filterCoords(matchingVillages, xmin, xmax, ymin, ymax, separator);

        document.getElementById('coordFilterResult').value = filteredCoords.length > 0 ? filteredCoords.join(' ') : '';
    };

    await loadTwSDK();
    await window.twSDK.init(window.scriptConfig);
    createUI();
})();
