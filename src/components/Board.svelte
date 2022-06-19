<script lang="ts">
    export let active: { [key: string]: number }[] = [];
    export let onClick = (i: number, ii: number) => {}
</script>

<div class="wrapper">
    <div class="board">
        {#each [...Array(10).keys()] as i}
            {#each [...Array(10).keys()] as ii}
                <div on:click={() => {active[i] ? active[i][ii.toString()] ? onClick(i, ii) : '' : ''}} class="tile" style="{i % 2 == ((ii + 1) % 2) ? 'background: #80ed9d;' : ''} {active[i] ? active[i][ii.toString()] ? 'box-shadow: inset 0px 0px 15px 5px #EDCB0C; cursor: pointer;' : '' : ''}"/>
            {/each} 
        {/each}

        <slot/>
    </div>
</div>

<style>

div.wrapper {   
    width: 1000px;
    height: 1000px;
    position: relative;
    margin-left: auto;
    margin-right: auto;
    margin-top: 20px;
}

div.wrapper > div.board {
    border: #e0a212 2px solid;
    background: white;
    width: 1000px;
    height: 1000px;
    display: flex;
    flex-wrap: wrap;
    box-shadow: 10px 10px 5px 5px #80ED9D;
    position: relative;
}

div.wrapper > div.board > div.tile {
    width: 100px;
    height: 100px;
}

</style>