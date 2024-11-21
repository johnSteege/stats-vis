<script setup lang="ts">
import StatsHeader from '@/layout/StatsHeader.vue';
import Checkbox from 'primevue/checkbox';
import DataTable from 'primevue/datatable';
import Panel from 'primevue/panel';
import { v4 as uuidv4 } from 'uuid';
import { onMounted, ref } from 'vue';

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
const showTotals = ref<boolean>(true);
const showTotalsPerGame = ref<boolean>(false);
</script>

<template>
    <StatsHeader style="position: relative"></StatsHeader>
    <div>
        <Panel>
            <template #header>
                <div class="flex items-center gap-2">
                    <span class="text-900 font-medium text-xl">2024 Totals</span>
                    <Button text icon="pi pi-plus" label="Show Totals" @click="showTotals = true" v-if="!showTotals" />
                    <Button text icon="pi pi-minus" label="Hide Totals" @click="showTotals = false" v-if="showTotals" />
                    <Checkbox label="Per Game" inputId="showTotalsPerGame" v-model="showTotalsPerGame" binary /><label for="showTotalsPerGame">Per Game</label>
                </div>
            </template>
            <DataTable :value="showTotalsPerGame ? totalsPerGame : teamTotalsData" dataKey="name" showGridlines stripedRows v-if="showTotals" removableSort>
                <Column field="name" header="Team"></Column>
                <Column v-if="!showTotalsPerGame" field="gameCount" header="Games"></Column>
                <Column v-if="!showTotalsPerGame" field="wins" header="Wins" sortable></Column>
                <Column v-if="!showTotalsPerGame" field="losses" header="Losses" sortable></Column>
                <Column field="pointsFor" header="Points For" sortable
                    ><template #body="{ data }"> {{ data.pointsFor.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column field="pointsAgainst" header="Points Against" sortable
                    ><template #body="{ data }"> {{ data.pointsAgainst.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
                ><Column field="pointsDiff" header="Points Diff" sortable
                    ><template #body="{ data }"> {{ data.pointsDiff.toFixed(showTotalsPerGame ? 1 : 0) }} </template></Column
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
            <DataTable :value="processedGameData" dataKey="id" showGridlines stripedRows paginator :rows="10" v-if="showSchedule">
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
