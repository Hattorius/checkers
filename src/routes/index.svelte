<script lang="ts">
import { onMount } from "svelte";

import Board from "../components/Board.svelte";
import Piece from "../components/Piece.svelte";

var board: {
    [key: string]: {
        isKing: boolean,
        isBlack: boolean,
        active: boolean
    }
}[] = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
var turn = false;

onMount(() => {
    for (var i = 0; i < 10; i++) {
        board[0][(i).toString()] = {
            isKing: false,
            isBlack: true,
            active: false
        }
        board[1][(i).toString()] = {
            isKing: false,
            isBlack: true,
            active: false
        }

        board[8][(i).toString()] = {
            isKing: false,
            isBlack: false,
            active: false
        }
        board[9][(i).toString()] = {
            isKing: false,
            isBlack: false,
            active: false
        }
    }
});

var currentActive: [
    number,
    string
] = [-1, ''];
var highlight: {
    [key: string]: number
}[] = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
const pieceClicked = (rowI: number, tileI: string) => {
    if (turn != board[rowI][tileI].isBlack) return;

    if (currentActive[0] != -1) {
        board[currentActive[0]][currentActive[1]].active = false;
    }
    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];

    board[rowI][tileI].active = true;
    currentActive = [rowI, tileI];

    if (board[rowI + (turn ? 1 : -1)]) {
        if (board[rowI + (turn ? 2 : -2)]) {
            var found = false;
            if (parseInt(tileI) - 2 >= 0) {
                if (board[rowI + (turn ? 1 : -1)][parseInt(tileI) - 1]) {
                    if (board[rowI + (turn ? 1 : -1)][parseInt(tileI) - 1].isBlack != turn) {
                        highlight[rowI + (turn ? 2 : -2)][parseInt(tileI) - 2] = 1;
                        found = true;
                    }
                }
            }
            if (parseInt(tileI) + 2 <= 9) {
                // check if someone there
                if (board[rowI + (turn ? 1 : -1)][parseInt(tileI) + 1]) {
                    if (board[rowI + (turn ? 1 : -1)][parseInt(tileI) + 1].isBlack != turn) {
                        highlight[rowI + (turn ? 2 : -2)][parseInt(tileI) + 2] = 1;
                        found = true;
                    }
                }
            }
            if (found) return;
        }


        if (typeof board[rowI + (turn ? 1 : -1)][tileI] === 'undefined') {
            highlight[rowI + (turn ? 1 : -1)][tileI] = 1;

            if (board[rowI + (turn ? 2 : -2)]) {
                if (typeof board[rowI + (turn ? 2 : -2)][tileI] === 'undefined') {
                    highlight[rowI + (turn ? 2 : -2)][tileI] = 1;
                }
            }
        }
    }
}

const movePieceTo = (rowI: number, tileI: number) => {
    if (tileI.toString() != currentActive[1]) {
        var tileToRemove = tileI - parseInt(currentActive[1]);
        if (tileToRemove > 0) tileToRemove--;
        else if (tileToRemove < 0) tileToRemove++;
        var rowToRemove = rowI - currentActive[0];
        if (rowToRemove > 0) rowToRemove--;
        else if (rowToRemove < 0) rowToRemove++;
        delete board[rowI - rowToRemove][(tileI - tileToRemove).toString()];
    }

    const piece = board[currentActive[0]][currentActive[1]];
    delete board[currentActive[0]][currentActive[1]];
    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
    currentActive[0] = -1;

    piece.active = false;
    board[rowI][tileI.toString()] = piece;
    turn = !turn;
}
</script>

<Board active={highlight} onClick={movePieceTo}>
    {#each board as row, i}
        {#each Object.keys(row) as tileI}
            <Piece onClick={() => {pieceClicked(i, tileI)}} posX={parseInt(tileI)} posY={i} {...row[tileI]}/>
        {/each}
    {/each}
</Board>