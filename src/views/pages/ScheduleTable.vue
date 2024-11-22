<script setup lang="ts">
import StatsHeader from '@/layout/StatsHeader.vue';
import Checkbox from 'primevue/checkbox';
import DataTable from 'primevue/datatable';
import MultiSelect from 'primevue/multiselect';
import Panel from 'primevue/panel';

import { v4 as uuidv4 } from 'uuid';
import { computed, onMounted, ref } from 'vue';

type TeamName =
    | 'Arizona Cardinals'
    | 'Atlanta Falcons'
    | 'Baltimore Ravens'
    | 'Buffalo Bills'
    | 'Carolina Panthers'
    | 'Chicago Bears'
    | 'Cincinnati Bengals'
    | 'Cleveland Browns'
    | 'Dallas Cowboys'
    | 'Denver Broncos'
    | 'Detroit Lions'
    | 'Green Bay Packers'
    | 'Houston Texans'
    | 'Indianapolis Colts'
    | 'Jacksonville Jaguars'
    | 'Kansas City Chiefs'
    | 'Las Vegas Raiders'
    | 'Los Angeles Chargers'
    | 'Los Angeles Rams'
    | 'Miami Dolphins'
    | 'Minnesota Vikings'
    | 'New England Patriots'
    | 'New Orleans Saints'
    | 'New York Giants'
    | 'New York Jets'
    | 'Philadelphia Eagles'
    | 'Pittsburgh Steelers'
    | 'San Francisco 49ers'
    | 'Seattle Seahawks'
    | 'Tampa Bay Buccaneers'
    | 'Tennessee Titans'
    | 'Washington Commanders';
const TEAM_NAME_VALUES: TeamName[] = [
    'Arizona Cardinals',
    'Atlanta Falcons',
    'Baltimore Ravens',
    'Buffalo Bills',
    'Carolina Panthers',
    'Chicago Bears',
    'Cincinnati Bengals',
    'Cleveland Browns',
    'Dallas Cowboys',
    'Denver Broncos',
    'Detroit Lions',
    'Green Bay Packers',
    'Houston Texans',
    'Indianapolis Colts',
    'Jacksonville Jaguars',
    'Kansas City Chiefs',
    'Las Vegas Raiders',
    'Los Angeles Chargers',
    'Los Angeles Rams',
    'Miami Dolphins',
    'Minnesota Vikings',
    'New England Patriots',
    'New Orleans Saints',
    'New York Giants',
    'New York Jets',
    'Philadelphia Eagles',
    'Pittsburgh Steelers',
    'San Francisco 49ers',
    'Seattle Seahawks',
    'Tampa Bay Buccaneers',
    'Tennessee Titans',
    'Washington Commanders'
];

type GameResultRaw = {
    week: number;
    dayOfWeek: string;
    date: string;
    time: string;
    winner: TeamName;
    location: string;
    loser: TeamName;
    boxscore: string;
    pointsWinner: number;
    pointsLoser: number;
    yardsWinner: number;
    turnoversWinner: number;
    yardsLoser: number;
    turnoversLoser: number;
};
const emptyGRR: GameResultRaw = {
    week: 0,
    dayOfWeek: '',
    date: '',
    time: '',
    winner: 'Green Bay Packers',
    location: '',
    loser: 'Minnesota Vikings',
    boxscore: '',
    pointsWinner: 0,
    pointsLoser: 0,
    yardsWinner: 0,
    turnoversWinner: 0,
    yardsLoser: 0,
    turnoversLoser: 0
};
const keysGRR = Object.keys(emptyGRR);

const rawData = ref<GameResultRaw[]>([]);

onMounted(() => {
    fetch('/src/assets/data/results2024.csv')
        .then((response) => response.text())
        .then((csv) => {
            const rows: string[] = csv.split('\r\n');
            const headers: string[] = rows[0].split(',');
            const parsedData: GameResultRaw[] = rows.slice(1).map((row) => {
                const values: string[] = row.split(',');
                return headers.reduce<GameResultRaw>((obj, header, index) => {
                    obj[keysGRR[index]] = values[index];
                    return obj;
                }, {} as GameResultRaw);
            });
            rawData.value = parsedData;

            processData();
        });
});

type ProcessedGameResult = {
    id: string;
    week: number;
    gameIndex: number;
    winner: TeamName;
    pointsWinner: number;
    yardsWinner: number;
    turnoversWinner: number;
    loser: TeamName;
    pointsLoser: number;
    yardsLoser: number;
    turnoversLoser: number;
};
const processedGameData = ref<ProcessedGameResult[]>([]);

type TeamTotals = {
    name: TeamName;
    gameCount: number;
    wins: number;
    losses: number;
    pointsFor: number;
    pointsAgainst: number;
    pointsDiff: number;
    yardsFor: number;
    yardsAgainst: number;
    yardsDiff: number;
    turnoversFor: number;
    turnoversAgainst: number;
    turnoversDiff: number;
};
const teamTotalsData = ref<TeamTotals[]>([]);
const totalsPerGame = ref<TeamTotals[]>([]);

function processData() {
    TEAM_NAME_VALUES.forEach((name) => {
        teamTotalsData.value.push({
            name: name,
            gameCount: 0,
            wins: 0,
            losses: 0,
            pointsFor: 0,
            pointsAgainst: 0,
            pointsDiff: 0,
            yardsFor: 0,
            yardsAgainst: 0,
            yardsDiff: 0,
            turnoversFor: 0,
            turnoversAgainst: 0,
            turnoversDiff: 0
        });
    });

    rawData.value.forEach((game, index) => {
        processedGameData.value.push({
            id: uuidv4(),
            week: game.week,
            gameIndex: index,
            winner: game.winner,
            pointsWinner: game.pointsWinner,
            yardsWinner: game.yardsWinner,
            turnoversWinner: game.turnoversWinner,
            loser: game.loser,
            pointsLoser: game.pointsLoser,
            yardsLoser: game.yardsLoser,
            turnoversLoser: game.turnoversLoser
        });

        const winnerTotals: TeamTotals = teamTotalsData.value.find((team) => team.name === game.winner);
        winnerTotals.gameCount++;
        winnerTotals.wins++;
        winnerTotals.pointsFor += Number(game.pointsWinner);
        winnerTotals.pointsAgainst += Number(game.pointsLoser);
        winnerTotals.pointsDiff += Number(game.pointsWinner - game.pointsLoser);
        winnerTotals.yardsFor += Number(game.yardsWinner);
        winnerTotals.yardsAgainst += Number(game.yardsLoser);
        winnerTotals.yardsDiff += Number(game.yardsWinner - game.yardsLoser);
        winnerTotals.turnoversFor += Number(game.turnoversWinner);
        winnerTotals.turnoversAgainst += Number(game.turnoversLoser);
        winnerTotals.turnoversDiff += Number(game.turnoversWinner - game.turnoversLoser);

        const loserTotals: TeamTotals = teamTotalsData.value.find((team) => team.name === game.loser);
        loserTotals.gameCount++;
        loserTotals.losses++;
        loserTotals.pointsFor += Number(game.pointsLoser);
        loserTotals.pointsAgainst += Number(game.pointsWinner);
        loserTotals.pointsDiff += Number(game.pointsLoser - game.pointsWinner);
        loserTotals.yardsFor += Number(game.yardsLoser);
        loserTotals.yardsAgainst += Number(game.yardsWinner);
        loserTotals.yardsDiff += Number(game.yardsLoser - game.yardsWinner);
        loserTotals.turnoversFor += Number(game.turnoversLoser);
        loserTotals.turnoversAgainst += Number(game.turnoversWinner);
        loserTotals.turnoversDiff += Number(game.turnoversLoser - game.turnoversWinner);
    });

    TEAM_NAME_VALUES.forEach((name) => {
        const curTotals: TeamTotals = teamTotalsData.value.find((team) => team.name === name);
        totalsPerGame.value.push({
            name: name,
            gameCount: curTotals.gameCount,
            wins: curTotals.wins,
            losses: curTotals.losses,
            pointsFor: curTotals.pointsFor / curTotals.gameCount,
            pointsAgainst: curTotals.pointsAgainst / curTotals.gameCount,
            pointsDiff: curTotals.pointsDiff / curTotals.gameCount,
            yardsFor: curTotals.yardsFor / curTotals.gameCount,
            yardsAgainst: curTotals.yardsAgainst / curTotals.gameCount,
            yardsDiff: curTotals.yardsDiff / curTotals.gameCount,
            turnoversFor: curTotals.turnoversFor / curTotals.gameCount,
            turnoversAgainst: curTotals.turnoversAgainst / curTotals.gameCount,
            turnoversDiff: curTotals.turnoversDiff / curTotals.gameCount
        });
    });
}

const showSchedule = ref<boolean>(true);
const showTotalsTable = ref<boolean>(true);
const showTotalsPerGame = ref<boolean>(false);
const totalsColumns = ref([
    { field: 'record', header: 'Record' },
    { field: 'points', header: 'Points' },
    { field: 'yards', header: 'Yards' },
    { field: 'turnovers', header: 'Turnovers' }
]);
const selectedTotals = ref(totalsColumns.value);
const onToggleTotals = (val) => {
    selectedTotals.value = totalsColumns.value.filter((col) => val.includes(col));
};
const showTotalRecord = computed(() => {
    return selectedTotals.value.some((col) => col.field === 'record');
});
const showTotalPoints = computed(() => {
    return selectedTotals.value.some((col) => col.field === 'points');
});
const showTotalYards = computed(() => {
    return selectedTotals.value.some((col) => col.field === 'yards');
});
const showTotalTurnovers = computed(() => {
    return selectedTotals.value.some((col) => col.field === 'turnovers');
});
</script>

<template>
    <StatsHeader style="position: relative"></StatsHeader>
    <div>
        <Panel>
            <template #header>
                <div class="flex items-center gap-2">
                    <span class="text-900 font-medium text-xl">2024 Totals</span>
                    <Button text icon="pi pi-plus" label="Show Totals" @click="showTotalsTable = true" v-if="!showTotalsTable" />
                    <Button text icon="pi pi-minus" label="Hide Totals" @click="showTotalsTable = false" v-if="showTotalsTable" />
                </div>
            </template>
            <DataTable :value="showTotalsPerGame ? totalsPerGame : teamTotalsData" dataKey="name" size="small" scrollable showGridlines stripedRows v-if="showTotalsTable" removableSort>
                <template #header>
                    <div class="flex items-center gap-2" style="text-align: left">
                        <MultiSelect :modelValue="selectedTotals" :options="totalsColumns" optionLabel="header" @update:modelValue="onToggleTotals" display="chip" placeholder="Select Columns" />
                        <Checkbox label="Per Game" inputId="showTotalsPerGame" v-model="showTotalsPerGame" binary /><label for="showTotalsPerGame">Per Game</label>
                    </div>
                </template>
                <Column field="name" header="Team"></Column>
                <Column v-if="!showTotalsPerGame" field="gameCount" header="Games"></Column>
                <Column v-if="!showTotalsPerGame && showTotalRecord" field="wins" sortable
                    ><template #header> <div v-tooltip.top="'Wins'">W</div></template>
                </Column>
                <Column v-if="!showTotalsPerGame && showTotalRecord" field="losses" sortable
                    ><template #header> <div v-tooltip.top="'Losses'">L</div></template>
                </Column>
                <Column v-if="showTotalPoints" field="pointsFor" sortable>
                    <template #header> <div v-tooltip.top="'Points For'">PF</div></template> <template #body="{ data }"> {{ data.pointsFor.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalPoints" field="pointsAgainst" sortable>
                    <template #header> <div v-tooltip.top="'Points Against'">PA</div></template><template #body="{ data }"> {{ data.pointsAgainst.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalPoints" field="pointsDiff" sortable>
                    <template #header> <div v-tooltip.top="'Point Differential'">PDif</div></template><template #body="{ data }"> {{ data.pointsDiff.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                >
                <Column v-if="showTotalYards" field="yardsFor" sortable>
                    <template #header> <div v-tooltip.top="'Yards For'">YF</div></template><template #body="{ data }"> {{ data.yardsFor.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalYards" field="yardsAgainst" sortable>
                    <template #header> <div v-tooltip.top="'Yards Against'">YA</div></template><template #body="{ data }"> {{ data.yardsAgainst.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalYards" field="yardsDiff" sortable>
                    <template #header> <div v-tooltip.top="'Yard Differential'">YDif</div></template><template #body="{ data }"> {{ data.yardsDiff.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                >
                <Column v-if="showTotalTurnovers" field="turnoversFor" sortable>
                    <template #header> <div v-tooltip.top="'Turnovers For'">ToF</div></template><template #body="{ data }"> {{ data.turnoversFor.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalTurnovers" field="turnoversAgainst" sortable>
                    <template #header> <div v-tooltip.top="'Turnovers Against'">ToA</div></template><template #body="{ data }"> {{ data.turnoversAgainst.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column v-if="showTotalTurnovers" field="turnoversDiff" sortable>
                    <template #header> <div v-tooltip.top="'Turnover Differential'">ToDif</div></template><template #body="{ data }"> {{ data.turnoversDiff.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                >
            </DataTable>
        </Panel>
        <Panel>
            <template #header>
                <div class="flex items-center gap-2">
                    <span class="text-900 font-medium text-xl">2024 Schedule</span>
                    <Button text icon="pi pi-plus" label="Show Schedule" @click="showSchedule = true" v-if="!showSchedule" />
                    <Button text icon="pi pi-minus" label="Hide Schedule" @click="showSchedule = false" v-if="showSchedule" />
                </div>
            </template>
            <DataTable :value="processedGameData" dataKey="id" size="small" scrollable showGridlines stripedRows paginator :rows="10" v-if="showSchedule">
                <Column field="week" header="Week"></Column>
                <Column field="winner" header="Winner"></Column>
                <Column field="pointsWinner" header="Points"></Column>
                <Column field="yardsWinner" header="Yards"></Column>
                <Column field="turnoversWinner" header="Turnovers"></Column>
                <Column field="loser" header="Loser"></Column>
                <Column field="pointsLoser" header="Points"></Column>
                <Column field="yardsLoser" header="Yards"></Column>
                <Column field="turnoversLoser" header="Turnovers"></Column>
            </DataTable>
        </Panel>
    </div>
</template>
