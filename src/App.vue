<template>
    <div id="app">
        <div><h3>Blockly test</h3></div>
        <v-blockly toolbox="toolbox.xml" workspace="workspace.xml" style="width: 100%; flex:1;"></v-blockly>
    </div>
</template>

<script>
    import VBlockly from "./components/VBlockly";

    export default {
        name: 'app',
        components: {
            VBlockly
        }
    }
</script>

<style lang="scss">
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
    }

    html, body {
        height: 100%;
        width: 100%;
        margin: 0;
    }
</style>
