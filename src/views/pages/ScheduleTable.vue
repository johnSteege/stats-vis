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

type TeamDiff = {
    name: TeamName;
    gameCount: number;
    points: number;
    yards: number;
    turnovers: number;
};
const teamDiffData = ref<TeamDiff[]>([]);
const teamDiffPerGame = ref<TeamDiff[]>([]);

function processData() {
    TEAM_NAME_VALUES.forEach((name) => {
        teamDiffData.value.push({
            name: name,
            gameCount: 0,
            points: 0,
            yards: 0,
            turnovers: 0
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

        const winnerDiff = teamDiffData.value.find((team) => team.name === game.winner);
        winnerDiff.gameCount++;
        winnerDiff.points += game.pointsWinner - game.pointsLoser;
        winnerDiff.yards += game.yardsWinner - game.yardsLoser;
        winnerDiff.turnovers += game.turnoversWinner - game.turnoversLoser;

        const loserDiff = teamDiffData.value.find((team) => team.name === game.loser);
        loserDiff.gameCount++;
        loserDiff.points += game.pointsLoser - game.pointsWinner;
        loserDiff.yards += game.yardsLoser - game.yardsWinner;
        loserDiff.turnovers += game.turnoversLoser - game.turnoversWinner;
    });

    TEAM_NAME_VALUES.forEach((name) => {
        const teamDiff: TeamDiff = teamDiffData.value.find((team) => team.name === name);
        teamDiffPerGame.value.push({
            name: name,
            gameCount: 1,
            points: teamDiff.points / teamDiff.gameCount,
            yards: teamDiff.yards / teamDiff.gameCount,
            turnovers: teamDiff.turnovers / teamDiff.gameCount
        });
    });
}

const showSchedule = ref<boolean>(true);
const showDiffs = ref<boolean>(true);
const diffsPerGame = ref<boolean>(false);
</script>

<template>
    <StatsHeader style="position: relative"></StatsHeader>
    <div>
        <Panel>
            <template #header>
                <div class="flex items-center gap-2">
                    <span class="text-900 font-medium text-xl">2024 Diffs</span>
                    <Button text icon="pi pi-plus" label="Show Diffs" @click="showDiffs = true" v-if="!showDiffs" />
                    <Button text icon="pi pi-minus" label="Hide Diffs" @click="showDiffs = false" v-if="showDiffs" />
                    <Checkbox label="Per Game" inputId="diffsPerGame" v-model="diffsPerGame" binary /><label for="diffsPerGame">Per Game</label>
                </div>
            </template>
            <DataTable :value="diffsPerGame ? teamDiffPerGame : teamDiffData" dataKey="name" showGridlines stripedRows v-if="showDiffs" removableSort>
                <Column field="name" header="Team"></Column>
                <Column v-if="!diffsPerGame" field="gameCount" header="Games"></Column>
                <Column field="points" header="Points" style="width: 10%" sortable
                    ><template #body="{ data }"> {{ data.points.toFixed(diffsPerGame ? 1 : 0) }} </template></Column
                ><Column field="yards" header="Yards" style="width: 10%" sortable
                    ><template #body="{ data }"> {{ data.yards.toFixed(diffsPerGame ? 1 : 0) }} </template></Column
                ><Column field="turnovers" header="Turnovers" style="width: 10%" sortable
                    ><template #body="{ data }"> {{ data.turnovers.toFixed(diffsPerGame ? 2 : 0) }} </template></Column
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
