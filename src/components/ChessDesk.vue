<script lang="ts">
import {defineComponent, ref} from 'vue';

const KING = 1; // Король
const QUEEN = 2; // Королева
const ROOK = 3; // Ладья
const BISHOP = 4; // Слон
const KNIGHT = 5; // Конь
const PAWN = 6; // Пешка

const FIGURE_LABELS = {
    [0]: {
        [KING]: '♔',
        [QUEEN]: '♕',
        [PAWN]: '♙',
        [BISHOP]: '♗',
        [ROOK]: '♖',
        [KNIGHT]: '♘'
    },
    [1]: {
        [KING]: '♚',
        [QUEEN]: '♛',
        [PAWN]: '♟',
        [BISHOP]: '♝',
        [ROOK]: '♜',
        [KNIGHT]: '♞'
    }
}

type ChessFigure = 0 | {
    type: number;
    team: 0|1;
};

const INITIAL_BASE_ROW = [ROOK, KNIGHT, BISHOP, QUEEN, KING, BISHOP, KNIGHT, ROOK];
type ChessDeskMatrix = Array<Array<ChessFigure>>;
const createChessFigure = (team: 0|1, type: number) => {
    return {
        type,
        team
    } as ChessFigure
};

const generateStartChessDesk = (): ChessDeskMatrix => {
    let desk:ChessDeskMatrix = [];
    for(let i=0; i<6; ++i){
        const row: Array<ChessFigure> = [];
        for (let j=0; j<8; ++j){
            switch (i){
                case 0: row.push(createChessFigure(1, PAWN)); break;
                case 5: row.push(createChessFigure(0, PAWN)); break;
                default: row.push(0);
            }
        }
        desk.push(row);
    }
    return [
        INITIAL_BASE_ROW.map((type) => createChessFigure(1, type)),
        ...desk,
        INITIAL_BASE_ROW.map((type) => createChessFigure(0, type)),
    ];
}

export default defineComponent({
    name: 'ChessDesk',
    setup(){
        const chessDeskMatrix = ref<ChessDeskMatrix>(generateStartChessDesk());

        return {
            chessDeskMatrix,
            FIGURE_LABELS,
        };
    },
});
</script>

<template>
    <div class="desk">
        <div
            v-for="(row, rowIndex) in chessDeskMatrix"
            :key="rowIndex"
            class="row"
        >
            <div
                v-for="(cell, colIndex) in row"
                :key="colIndex"
                class="cell"
                :class="{
                    'paired': (rowIndex + colIndex) % 2 === 1,
                    'unpaired': (rowIndex + colIndex) % 2 === 0,
                }"
            >
                <span
                    v-if="cell !== 0"
                    class="figure"
                    :class="cell.team === 0 ? 'white' : 'black'"
                >
                    {{ FIGURE_LABELS[cell.team][cell.type] }}
                </span>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
    .desk{
        display: flex;
        flex-direction: column;
    }
    .row{
        display: flex;
    }
    .cell{
        width: 50px;
        height: 50px;
        display: flex;
        justify-content: center;
        align-items: center;

        &.paired{
            background: black;
        }
        &.unpaired{
            background: white;
        }
    }
    .figure{
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 50%;
        width:30px;
        height:30px;
        background: #dbeae7;
        border:1px solid #b6c5c0;
    }
</style>