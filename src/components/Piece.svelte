<script lang="ts">
    export let posX = 0;
    export let posY = 0;
    export let isKing = false;
    export let isBlack = false;
    export let active = false;

    export let onClick = () => {};

    var color: string;
    if (isBlack) {
        color = '#292929';
    } else {
        color = '#e0e0e0';
    }
</script>

<div on:click={onClick} style="--theme-color: {color}; left: {posX * 100 + 10}px; top: {posY * 100 + 10}px; {active ? 'box-shadow: 0px 0px 15px 5px #EDCB0C;' : ''} {isKing ? 'background: linear-gradient(135deg, #edcb0c 50%, ' + color + ' 50%);' : ''}"/>

<style>
    div {
        width: 80px;
        height: 80px;
        border-radius: 50%;
        background: var(--theme-color);
        position: absolute;
        cursor: pointer;
    }
</style>