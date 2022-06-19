<script lang="ts">
import { onMount } from "svelte";
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
            isKing: false,
            isBlack: false,
            active: false,
            canAttack: false
        }
    }
});

const canAttack = (rowI: number, tileI: number) => {
    var highlights: [number, number][] = [];
    const diagonals = [[-1, 1], [-1, -1], [1, -1], [1, 1]];

    for (var i = 0; i < diagonals.length; i++) {
        const diagonal = diagonals[i];

        // row does not exist
        if (!board[rowI + diagonal[0] * 2]) {
            i++;
            continue;
        }

         // if column does no exist
        if (tileI + diagonal[1] * 2 > 9 || tileI + diagonal[1] * 2 < 0) {
            continue;
        }

        // if there is a piece to take and if there is place behind the piece
        if (typeof board[rowI + diagonal[0] * 2][tileI + diagonal[1] * 2] !== "undefined" || typeof board[rowI + diagonal[0]][tileI + diagonal[1]] === "undefined") {
            continue;
        }

        // if piece is of the other team
        if (board[rowI + diagonal[0]][tileI + diagonal[1]].isBlack == board[rowI][tileI].isBlack) {
            continue
        }

        // highlight attacking moves and lock to only attacking
        canAttackToggle = true;
        board[rowI][tileI].canAttack = true;
        highlights.push([rowI + diagonal[0] * 2, tileI + diagonal[1] * 2]);
    }
    return highlights;
}

const pieceClicked = (rowI: number, tileI: string) => {
    if (turn != board[rowI][tileI].isBlack) return;
    if (canAttackToggle) {
        if (!board[rowI][tileI].canAttack) return;
        const highlights = canAttack(rowI, parseInt(tileI));
        highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];

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
    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];

    board[rowI][tileI].active = true;
    currentActive = [rowI, tileI];

    if (board[rowI + (turn ? 1 : -1)]) {
        if (typeof board[rowI + (turn ? 1 : -1)][parseInt(tileI) + 1] === "undefined" && parseInt(tileI) + 1 <= 9) {
            highlight[rowI + (turn ? 1 : -1)][parseInt(tileI) + 1] = 1;
        }
        if (typeof board[rowI + (turn ? 1 : -1)][parseInt(tileI) - 1] === "undefined" && parseInt(tileI) - 1 >= 0) {
            highlight[rowI + (turn ? 1 : -1)][parseInt(tileI) - 1] = 1;
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

        if (turn) {
            blackScore += 1;
        } else {
            whiteScore += 1;
        }
    }

    const piece = board[currentActive[0]][currentActive[1]];
    piece.canAttack = false;
    delete board[currentActive[0]][currentActive[1]];
    highlight = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
    currentActive[0] = -1;

    piece.active = false;
    board[rowI][tileI.toString()] = piece;
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