<script lang="ts">
import { onMount } from "svelte";

import Board from "../components/Board.svelte";
import Piece from "../components/Piece.svelte";

var board: {
    [key: string]: {
        isKing: boolean,
        isBlack: boolean
    }
}[] = [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}];
onMount(() => {
    for (var i = 0; i < 10; i++) {
        board[0][(i).toString()] = {
            isKing: false,
            isBlack: true
        }
        board[1][(i).toString()] = {
            isKing: false,
            isBlack: true
        }

        board[8][(i).toString()] = {
            isKing: false,
            isBlack: false
        }
        board[9][(i).toString()] = {
            isKing: false,
            isBlack: false
        }
    }
});
</script>

<Board>
    {#each board as row, i}
        {#each Object.keys(row) as tileI}
            <Piece posX={parseInt(tileI)} posY={i} {...row[tileI]}/>
        {/each}
    {/each}
</Board>