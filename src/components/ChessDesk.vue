<script lang="ts">
import {defineComponent, ref} from 'vue';
import {cloneDeep} from 'lodash';

const KING = 1; // Король
const QUEEN = 2; // Королева
const ROOK = 3; // Ладья
const BISHOP = 4; // Слон
const KNIGHT = 5; // Конь
const PAWN = 6; // Пешка

const WHITE_TEAM = 0;
const BLACK_TEAM = 1;

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
type Coords = [number, number];

const INITIAL_BASE_ROW = [ROOK, KNIGHT, BISHOP, QUEEN, KING, BISHOP, KNIGHT, ROOK];
const KNIGHT_MOVES: Coords[] = [ [2,1], [2,-1], [-2,1], [-2,-1], [1,2], [-1,2], [1,-2], [-1,-2] ];
const ROOK_MOVES: Coords[] = [ [1,0], [0,1], [-1,0], [0,-1] ];
const BISHOP_MOVES: Coords[] = [ [1,-1], [1,1], [-1,1], [-1,-1] ];
const KING_MOVES: Coords[] = [ ...ROOK_MOVES, ...BISHOP_MOVES ];
const QUEEN_MOVES: Coords[] = [ ...ROOK_MOVES, ...BISHOP_MOVES ];

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
                case 0: row.push(createChessFigure(BLACK_TEAM, PAWN)); break;
                case 5: row.push(createChessFigure(WHITE_TEAM, PAWN)); break;
                default: row.push(0);
            }
        }
        desk.push(row);
    }
    return [
        INITIAL_BASE_ROW.map((type) => createChessFigure(BLACK_TEAM, type)),
        ...desk,
        INITIAL_BASE_ROW.map((type) => createChessFigure(WHITE_TEAM, type)),
    ];
};

const generateDirections = (moves: Coords[], steps: number, team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    let directions: Coords[] = [];
    moves.forEach(([xm, ym]) => {
        let newX = x;
        let newY = y;
        for(let i = 0; i<steps; ++i){
            newX += xm;
            newY += ym;

            if (newX >= 0 && newX < 8 && newY >= 0 && newY < 8){
                const position = matrix[newY][newX];
                if (position === 0){
                    directions.push([newX, newY]);
                } else {
                    if (position.team !== team){
                        directions.push([newX, newY]);
                    }
                    break;
                }
            } else {
                break;
            }
        }
    });
    return directions;
}

const pawnDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    let directions: Coords[] = [];
    const yDir = team === WHITE_TEAM ? -1 : 1;
    const forwardMoveY = y + yDir;
    const forwardMoveCheck = forwardMoveY !== 8 && forwardMoveY !== -1
    if (forwardMoveCheck && matrix[forwardMoveY][x] === 0){
        directions.push([x, forwardMoveY]);

        if (team === WHITE_TEAM && y === 6 && matrix[y-2][x] === 0){
            directions.push([x, y-2]);
        } else if (team === BLACK_TEAM && y === 1 && matrix[y+2][x] === 0){
            directions.push([x, y+2]);
        }
    }
    if (x > 0 && forwardMoveCheck){
        const enemy = matrix[forwardMoveY][x-1];
        if (enemy !== 0 && enemy.team !== team){
            directions.push([x-1, forwardMoveY]);
        }
    }
    if (x < 7 && forwardMoveCheck){
        const enemy = matrix[forwardMoveY][x+1];
        if (enemy !== 0 && enemy.team !== team){
            directions.push([x+1, forwardMoveY]);
        }
    }

    return directions;
};

const knightDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    return generateDirections( KNIGHT_MOVES, 1, team, matrix, x, y);
};

const rookDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    return generateDirections(ROOK_MOVES, 7, team, matrix, x, y);
};

const bishopDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    return generateDirections(BISHOP_MOVES, 7, team, matrix, x, y);
};

const queenDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    return generateDirections(QUEEN_MOVES, 7, team, matrix, x, y);
};

const kingDirections = (team: 0|1, matrix: ChessDeskMatrix, x: number, y: number): Coords[] => {
    return generateDirections(KING_MOVES, 1, team, matrix, x, y);
};

export default defineComponent({
    name: 'ChessDesk',
    setup(){
        const chessDeskMatrix = ref<ChessDeskMatrix>(generateStartChessDesk());
        const backBuffer = ref<ChessDeskMatrix[]>([]);
        const nextBuffer = ref<ChessDeskMatrix[]>([]);
        let teamStep = WHITE_TEAM;
        let clickedPosition = ref<false|Coords>(false);
        let possibleDirections = ref<Coords[]>([]);

        const setPosition = (x: number, y: number) => {
            clickedPosition.value = false;
            possibleDirections.value = [];
            const figure = chessDeskMatrix.value[y][x];
            if (figure !== 0 && figure.team === teamStep){
                clickedPosition.value = [x, y];
                let calcDirection = null;
                switch (figure.type){
                    case PAWN: calcDirection = pawnDirections; break;
                    case BISHOP: calcDirection = bishopDirections; break;
                    case ROOK: calcDirection = rookDirections; break;
                    case KNIGHT: calcDirection = knightDirections; break;
                    case QUEEN: calcDirection = queenDirections; break;
                    case KING: calcDirection = kingDirections; break;
                }
                if (calcDirection){
                    possibleDirections.value = calcDirection(teamStep, chessDeskMatrix.value, x, y);
                }
            }
        }

        const move = (x: number, y: number) => {
            if (clickedPosition.value){
                backBuffer.value.push(cloneDeep(chessDeskMatrix.value));
                nextBuffer.value = [];
                const [cx, cy] = clickedPosition.value;
                const figure = chessDeskMatrix.value[cy][cx];
                chessDeskMatrix.value[cy][cx] = 0;
                chessDeskMatrix.value[y][x] = figure;

                teamStep = teamStep === WHITE_TEAM ? BLACK_TEAM : WHITE_TEAM;
                clickedPosition.value = false;
                possibleDirections.value = [];
            }
        };

        const backMove = () => {
            if (backBuffer.value.length > 0){
                teamStep = teamStep === WHITE_TEAM ? BLACK_TEAM : WHITE_TEAM;
                const popped = backBuffer.value.pop() as ChessDeskMatrix;
                nextBuffer.value.push(cloneDeep(chessDeskMatrix.value));
                chessDeskMatrix.value = cloneDeep(popped);
            }
        };

        const nextMove = () => {
            if (nextBuffer.value.length > 0){
                teamStep = teamStep === WHITE_TEAM ? BLACK_TEAM : WHITE_TEAM;
                const popped = nextBuffer.value.pop() as ChessDeskMatrix;
                backBuffer.value.push(cloneDeep(chessDeskMatrix.value));
                chessDeskMatrix.value = cloneDeep(popped);
            }
        };

        const clickBy = (x: number, y: number) => {
            if (!clickedPosition.value){
                setPosition(x, y);
            } else {
                const checkMove = possibleDirections.value.some((dir) => x === dir[0] && y === dir[1]);
                if (checkMove){
                    move(x, y);
                } else {
                    setPosition(x, y);
                }
            }
        };

        return {
            clickBy,
            chessDeskMatrix,
            backBuffer,
            nextBuffer,
            backMove,
            nextMove,
            FIGURE_LABELS,
            WHITE_TEAM,
            BLACK_TEAM,
            possibleDirections,
            clickedPosition,
        };
    },
});
</script>

<template>
    <div class="desk-wrap">
        <div class="controls">
            <button :disabled="backBuffer.length === 0" @click="backMove">Back</button>
            <button :disabled="nextBuffer.length === 0" @click="nextMove">Next</button>
        </div>
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
                    @click="clickBy(colIndex, rowIndex)"
                >
                    <div
                        v-if="possibleDirections.some((dir) => colIndex === dir[0] && rowIndex === dir[1])"
                        class="possible-move"
                    />
                    <span
                        v-if="cell !== 0"
                        class="figure"
                        :class="{
                        'active': clickedPosition && clickedPosition[0] === colIndex && clickedPosition[1] === rowIndex,
                    }"
                    >
                    {{ FIGURE_LABELS[cell.team][cell.type] }}
                </span>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
    .desk-wrap{
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }

    .controls{
        display: flex;
        margin-bottom: 15px;

        button{
            margin-right: 10px;
            background: #3737e7;
            color:white;
            border:0;
            padding: 4px 20px;
            border-radius: 5px;
            cursor: pointer;

            &:hover{
                background: #2c2cb6;
            }
            &:disabled{
                background: #55709c;
                color: #a6b4cb;
                cursor: default;
            }
        }
    }

    .desk{
        display: flex;
        flex-direction: column;
    }
    .row{
        display: flex;
    }
    .cell{
        position: relative;
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
    .possible-move{
        position: absolute;
        left:50%;
        top:50%;
        width:10px;
        height:10px;
        border-radius: 50%;
        transform: translateX(-50%) translateY(-50%);
        background: #25ac0d;
        opacity: 50%;
        z-index: 2;
    }
    .figure{
        position: relative;
        z-index: 1;
        user-select: none;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 50%;
        width:30px;
        height:30px;
        background: #dbeae7;
        border:2px solid #b6c5c0;

        &.active{
            border-color: #25ac0d;
        }
    }
</style>