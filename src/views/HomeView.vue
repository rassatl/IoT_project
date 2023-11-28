<script setup>
import { onMounted, ref } from 'vue';
import Chart from 'chart.js/auto';

const fetchData = async () => {
    try {
        // Récupère le data.json et transmet les données dans les composants correspondants
        const response = await fetch('data.json');
        const jsonData = await response.json();

        const responseReel = await fetch('dataReel.json');
        const jsonDataReel = await responseReel.json();
        displayChartsLifeAndRounds(jsonData['lifeAndRound']);
        displayLogs(jsonDataReel);
        displayLeftBarGoldScoreBoard(jsonDataReel['gold']);
        // console.log(jsonDataReel['enter_arena']);
    } catch (error) {
        console.error('Erreur lors de la récupération des données JSON :', error);
    }
};

let sortedGold = ref([]);

const displayLeftBarGoldScoreBoard = (gold) => {
    sortedGold.value = gold
        .flatMap(([timestamp, goldData]) => Object.entries(goldData).map(([playerId, playerGold]) => ({ playerId, playerGold })))
        .sort((a, b) => b.playerGold - a.playerGold);
};

// Creer un graphique pour afficher les points de vie en fonction des rounds
const displayChartsLifeAndRounds = (datas) => {
    ctx.value = document.getElementById('life');

    new Chart(ctx.value, {
        type: 'line',
        data: datas,
        options: {
            plugins: {
                title: {
                    display: true,
                    text: 'Life/Round',
                    font: {
                        size: 16
                    }
                }
            },
            scales: {
                x: {
                    title: {
                        display: true,
                        text: 'Rounds'
                    }
                },
                y: {
                    title: {
                        display: true,
                        text: 'Life Points'
                    }
                }
            }
        }
    });
}

// Créer une liste des logs
const displayLogs = (logs) => {
    const logsDiv = document.getElementById('logs');
    logsDiv.innerHTML = '<ul>';
    logs['enter_arena'].forEach((log) => {
        const logItem = document.createElement('li');
        logItem.textContent = `${log[1]['cid']} from Team ${log[1]['teamid']} as join the arena.`;
        logsDiv.appendChild(logItem);
    });
    logsDiv.innerHTML += '<p>----------</p>';
    logs['set_action'].forEach((log) => {
        const logItem = document.createElement('li');
        logItem.textContent = `action : ${log[1]['action']} as been given to ${log[1]['cid']} from the ${log[1]['teamid']} team.`;
        logsDiv.appendChild(logItem);
    });
    logsDiv.innerHTML += '<p>----------</p>';
    logs['set_target'].forEach((log) => {
        const logItem = document.createElement('li');
        logItem.textContent = `The player ${log[1]['cid']} from the ${log[1]['teamid']} team aimed at ${log[1]['target']}`;
        logsDiv.appendChild(logItem);
    });
    logsDiv.innerHTML += '<p>----------</p>';
    logsDiv.innerHTML += '</ul>';
};

const ctx = ref(null);

onMounted(() => {
    fetchData();
});

// const _ctx = ref(null);
// const statsData = ref({
//     labels: ['Eating', 'Drinking', 'Sleeping', 'Designing', 'Coding', 'Cycling', 'Running'],
//     datasets: [{
//         label: 'My First Dataset',
//         data: [65, 59, 90, 81, 56, 55, 40],
//         fill: true,
//         backgroundColor: 'rgba(255, 99, 132, 0.2)',
//         borderColor: 'rgb(255, 99, 132)',
//         pointBackgroundColor: 'rgb(255, 99, 132)',
//         pointBorderColor: '#fff',
//         pointHoverBackgroundColor: '#fff',
//         pointHoverBorderColor: 'rgb(255, 99, 132)'
//     }, {
//         label: 'My Second Dataset',
//         data: [28, 48, 40, 19, 96, 27, 100],
//         fill: true,
//         backgroundColor: 'rgba(54, 162, 235, 0.2)',
//         borderColor: 'rgb(54, 162, 235)',
//         pointBackgroundColor: 'rgb(54, 162, 235)',
//         pointBorderColor: '#fff',
//         pointHoverBackgroundColor: '#fff',
//         pointHoverBorderColor: 'rgb(54, 162, 235)'
//     }]
// });

// const statsConfig = ref({
//     type: 'radar',
//     data: statsData.value,
//     options: {
//         elements: {
//             line: {
//                 borderWidth: 3
//             }
//         }
//     }
// });

// onMounted(() => {
//     _ctx.value = document.getElementById('stats');
//     fetchData(statsData, _ctx, statsConfig);
// });
</script>

<template>
    <div class="p-5 h-screen flex flex-col gap-3">
        <div class="skeleton h-[10vh] w-full grid grid-cols-3 justify-center place-items-center gap-5">
            <div class="flex gap-3 w-full pl-5 pr-5">
                <select class="select select-primary w-full max-w-xs">
                    <option disabled selected>Server</option>
                    <option>All</option>
                    <option>http://127.0.0.0:4700</option>
                    <option>http://127.0.0.0:4600</option>
                    <option>http://127.0.0.0:4500</option>
                </select>

                <select class="select select-primary w-full max-w-xs">
                    <option disabled selected>Rounds</option>
                    <option>All</option>
                    <option>Rounds 1</option>
                    <option>Rounds 2</option>
                    <option>Rounds 3</option>
                </select>
            </div>

            <input type="text" placeholder="https://127.0.0.0:1256" class="input input-bordered w-full max-w-xs" disabled />

            <div class="flex gap-3 w-full justify-end p-5">
                <div class="indicator pr-5 pl-5">
                    <span class="indicator-item badge badge-secondary"></span>
                    <div class="bg-base-300 flex gap-2">
                        <span>Waiting</span>
                        <span class="loading loading-dots loading-xs"></span>
                    </div>
                </div>
                <label class="flex cursor-pointer gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
                        stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <circle cx="12" cy="12" r="5" />
                        <path
                            d="M12 1v2M12 21v2M4.2 4.2l1.4 1.4M18.4 18.4l1.4 1.4M1 12h2M21 12h2M4.2 19.8l1.4-1.4M18.4 5.6l1.4-1.4" />
                    </svg>
                    <input type="checkbox" value="dark" class="toggle theme-controller" />
                    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
                        stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
                    </svg>
                </label>
            </div>

        </div>
        <div class="flex grow h-full w-full gap-3">
            <div class="skeleton h-full w-[20%]" id="goldSB">
                <!-- <p>Gold Score Board</p> -->
                <table class="table w-full">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>Player ID</th>
                            <th>Gold</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(player, index) in sortedGold.slice(0, 3)" :key="player.playerId"
                            :class="{ 'hover:bg-gray-200': index < 3 }">
                            <th>{{ index + 1 }}</th>
                            <td>{{ player.playerId }}</td>
                            <td>{{ player.playerGold }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="skeleton h-full grow grid grid-cols-2 gap-3 p-3">
                <div class="mockup-code h-full w-full" id="logs" style="overflow-y:auto;">
                </div>
                <div class="h-full w-full grid grid-rows-2 gap-3">
                    <div class="w-full h-full flex justify-center">
                        <canvas id="life"></canvas>
                    </div>
                    <div class="w-full h-full">
                        <canvas id="stats"></canvas>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template> 