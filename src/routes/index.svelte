<script lang="ts">
import { afterUpdate, onMount } from "svelte";
import Score from "../components/Score.svelte";
import Board from "../components/Board.svelte";
import Piece from "../components/Piece.svelte";

var board: {
    [key: string]: {
        isKing: boolean, // not implemented
        isBlack: boolean,
        active: boolean,
        canAttack: boolean
    }
}[] = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
var turn = false;
var currentActive: [
    number,
    string
] = [-1, ''];
var highlight: {
    [key: string]: number
}[] = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
var canAttackToggle = false;

var blackScore = 0;
var whiteScore = 0;

onMount(() => {
    for (var i = 0; i < 20; i++) {
        board[i % 4][i % 5 * 2 + (i + 1) % 2] = {
            isKing: false,
            isBlack: true,
            active: false,
            canAttack: false
        }
        board[i % 4 + 6][i % 5 * 2 + (i + 1) % 2] = {
            isKing: i % 7 == 0 ? true : false,
            isBlack: false,
            active: false,
            canAttack: false
        }
    }
    delete board[1][6];
    delete board[2][5];
});

const canAttack = (rowI: number, tileI: number) => {
    var highlights: [number, number][] = [];
    var diagonals = [[-1, 1], [-1, -1], [1, -1], [1, 1]];

    for (var i = 0; i < diagonals.length; i++) {
        const diagonal = diagonals[i];
        var iteration = 1;
        while (true) {
            iteration += 1;
            if (iteration == 3 && !board[rowI][tileI].isKing) break;
            // row does not exist
            if (!board[rowI + diagonal[0] * iteration]) {
                break;
            }

            // if column does not exist
            if (tileI + diagonal[1] * iteration > 9 || tileI + diagonal[1] * iteration < 0) {
                break;
            }

            // if there is a piece to take
            if (typeof board[rowI + diagonal[0] * (iteration - 1)][tileI + diagonal[1] * (iteration - 1)] === "undefined") {
                continue;
            }

            // if there is place behind the piece
            if (typeof board[rowI + diagonal[0] * iteration][tileI + diagonal[1] * iteration] !== "undefined") {
                break;
            }

            if (board[rowI + diagonal[0] * (iteration - 1)][tileI + diagonal[1] * (iteration - 1)].isBlack == board[rowI][tileI].isBlack) {
                break
            }

            canAttackToggle = true;
            board[rowI][tileI].canAttack = true;
            while (true) {
                if (typeof board[rowI + diagonal[0] * iteration][tileI + diagonal[1] * iteration] === "undefined") {
                    highlights.push([rowI + diagonal[0] * iteration, tileI + diagonal[1] * iteration]);
                    if (!board[rowI][tileI].isKing) break;
                    iteration++;
                } else {
                    break;
                }
            }
            break;
        }
    }

    return highlights;
}

const pieceClicked = (rowI: number, tileI: string) => {
    if (turn != board[rowI][tileI].isBlack) return;

    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];

    if (canAttackToggle) {
        if (!board[rowI][tileI].canAttack) return;
        const highlights = canAttack(rowI, parseInt(tileI));

        board[rowI][tileI].active = true;
        if (currentActive[0] != -1) {
            board[currentActive[0]][currentActive[1]].active = false;
        }
        currentActive = [rowI, tileI];

        for (var i = 0; i < highlights.length; i++) {
            highlight[highlights[i][0]][highlights[i][1]] = 1;
        }

        return;
    };

    if (currentActive[0] != -1) {
        board[currentActive[0]][currentActive[1]].active = false;
    }

    board[rowI][tileI].active = true;
    currentActive = [rowI, tileI];

    var rowStep = turn ? 1 : -1;
    if (rowStep) {
        var iteration = 0;
        while (true) {
            iteration++;
            if (typeof board[rowI + rowStep * iteration][parseInt(tileI) + iteration] === "undefined" && parseInt(tileI) + 1 <= 9) {
                highlight[rowI + rowStep * iteration][parseInt(tileI) + iteration] = 1;
            } else {
                break;
            }
            if (!board[rowI][tileI].isKing) break;
        }

        iteration = 0;
        while (true) {
            iteration++;
            if (typeof board[rowI + rowStep * iteration][parseInt(tileI) - iteration] === "undefined" && parseInt(tileI) - 1 >= 0) {
                highlight[rowI + rowStep * iteration][parseInt(tileI) - iteration] = 1;
            } else {
                break;
            }
            if (!board[rowI][tileI].isKing) break;
        }
    }
}

const movePieceTo = (rowI: number, tileI: number) => {
    const tileDifference = tileI - parseInt(currentActive[1]);
    const rowDifference = rowI - currentActive[0];
    if (tileDifference > 1 || tileDifference < -1) {
        const tileStep = tileDifference < -1 ? -1 : 1;
        const rowStep = rowDifference < -1 ? -1 : 1;
        for (var i = 1; i < (tileDifference < -1 ? tileDifference * -1 : tileDifference); i++) {
            delete board[rowI - (rowStep * i)][(tileI - (tileStep * i)).toString()];
            if (turn) blackScore += 1;
            else whiteScore += 1;
        }
    }

    const piece = board[currentActive[0]][currentActive[1]];
    piece.canAttack = false;
    delete board[currentActive[0]][currentActive[1]];
    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
    currentActive[0] = -1;

    piece.active = false;
    board[rowI][tileI.toString()] = piece;

    if (canAttack(rowI, tileI).length > 0) {
        return;
    }

    turn = !turn;

    canAttackToggle = false;
    for (var i = 0; i < board.length; i++) {
        Object.keys(board[i]).forEach((tile) => {
            if (board[i][tile].isBlack == turn) {
                canAttack(i, parseInt(tile));
            }
        })
    }
}
</script>

<Score black={blackScore} white={whiteScore} turn={turn}/>
<Board active={highlight} onClick={movePieceTo}>
    {#each board as row, i}
        {#each Object.keys(row) as tileI}
            <Piece onClick={() => {pieceClicked(i, tileI)}} posX={parseInt(tileI)} posY={i} {...row[tileI]}/>
        {/each}
    {/each}
</Board>