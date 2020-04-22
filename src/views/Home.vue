<template>
    <div class="home">

        <div class="shortcut-key">
            <div>
                <kbd>Ctrl</kbd> + <kbd>Enter</kbd> 插入子节点
            </div>
            <div>
                <kbd>Alt</kbd> + <kbd>Enter</kbd> 插入兄弟节点
            </div>
            <div>
                <kbd>Ctrl</kbd> + <kbd>Delete</kbd> 删除子节点
            </div>

        </div>
        <mind msg="Welcome to Your Vue.js App" />

    </div>
</template>

<script>
// @ is an alias to /src
import mind from "@/components/mind.vue";

export default {
    name: "Home",
    components: {
        mind
    }
};
</script>


<style lang="less">
kbd {
    margin: 0px 0.1em;
    padding: 0.1em 0.6em;
    border-radius: 3px;
    border: 1px solid rgb(204, 204, 204);
    color: rgb(51, 51, 51);
    line-height: 1.4;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 10px;
    display: inline-block;
    box-shadow: 0px 1px 0px rgba(0, 0, 0, 0.2), inset 0px 0px 0px 2px #ffffff;
    background-color: rgb(247, 247, 247);
    -moz-box-shadow: 0 1px 0px rgba(0, 0, 0, 0.2), 0 0 0 2px #ffffff inset;
    -webkit-box-shadow: 0 1px 0px rgba(0, 0, 0, 0.2), 0 0 0 2px #ffffff inset;
    -moz-border-radius: 3px;
    -webkit-border-radius: 3px;
    text-shadow: 0 1px 0 #fff;
}
.shortcut-key {
    font-size: 12px;
    line-height: 30px;
}
</style>