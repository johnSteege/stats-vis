<script setup lang="ts">
import StatsHeader from '@/layout/StatsHeader.vue';
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
function processData() {
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
    });
}

const showSchedule = ref<boolean>(true);
</script>

<template>
    <StatsHeader style="position: relative"></StatsHeader>
    <div class="card">
        <Button text icon="pi pi-plus" label="Show Schedule" @click="showSchedule = true" v-if="!showSchedule" />
        <Button text icon="pi pi-minus" label="Hide Schedule" @click="showSchedule = false" v-if="showSchedule" />
        <DataTable :value="processedGameData" dataKey="id" showGridlines stripedRows paginator :rows="10" v-if="showSchedule">
            <template #header>
                <span class="text-900 font-medium text-xl">2024 Schedule</span>
            </template>
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
    </div>
</template>
