<script lang="ts">
    export let black = 0;
    export let white = 0;
    export let turn = false;
</script>

<div class="score">
    <div>
        Black: {black}<br />
        White: {white}
    </div>
    <div>
        <br />
        {turn ? 'Black' : 'White'}'s turn
    </div>
</div>

<style>
    div.score {
        width: 1000px;
        margin-left: auto;
        margin-right: auto;
        margin-top: 60px;
        font-size: 20px;
        display: flex;
    }

    div.score > div:last-child {
        text-align: right;
        flex-grow: 1;
    }
</style>