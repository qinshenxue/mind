
<template>
    <div>
        <div class="node" :class="{'is-root':data.isRoot}" :id="'node-'+data.id"
             @blur="onBlur" contenteditable :style="{
        top:data.y+'px',
        left:data.x+'px',
        minWidth:data.w+'px',
        minHeight:data.h+'px'
    }" @keydown.ctrl.enter.prevent="addChildNode"
             @keydown.ctrl.delete.prevent="$emit('ctrl-delete')"
             @keydown.alt.enter.prevent="$emit('alt-enter')" ref="node"
             @mousedown="dragStart" @paste.prevent="onPaste">

        </div>
        <template v-if="data.expand">
            <node @alt-enter="addBrotherNode(index)"
                  @ctrl-delete="delNode(index)" @change="onChange"
                  v-for="(item, index) in data.children" :key="index"
                  :data="item">
            </node>
        </template>
    </div>
</template>

<script>
export default {
    name: "node",
    inject: ["mind"],
    props: {
        data: Object,
        index: Number,
        x: Number,
        y: Number,
        w: Number,
        h: Number
    },

    data() {
        return {
            top: this.y,
            left: this.x,
            width: this.w,
            height: this.h,
            drag: {
                start: false,
                x1: 0,
                x2: 0,
                y1: 0,
                y2: 0
            }
        };
    },

    mounted() {
        document.addEventListener("mousemove", this.dragging);
        document.addEventListener("mouseup", this.dragEnd);
    },
    methods: {
        getUuid() {
            return Math.random()
                .toString(36)
                .slice(2);
        },
        addChildNode() {
            const id = this.getUuid();
            this.data.children.push({
                id,
                deep: this.data.deep + 1,
                x: this.data.x + 300,
                y: 0,
                startY: 0,
                w: 100,
                h: 80,
                gap: 40,
                childrenHeight: 0,
                edit: true,
                content: "",
                expand: true,
                children: []
            });

            this.onChange(id);
        },
        addBrotherNode(i) {
            const id = this.getUuid();
            this.data.children.splice(i + 1, 0, {
                id,
                deep: this.data.deep + 1,
                x: this.data.x + 300,
                y: 0,
                startY: 0,
                w: 100,
                h: 80,
                gap: 40,
                childrenHeight: 0,
                edit: true,
                content: "",
                expand: true,
                children: []
            });
            this.onChange(id);
        },
        delNode(i) {
            this.data.children.splice(i, 1);
            this.onChange();
        },

        onPaste(e) {
            var items = (e.clipboardData || window.clipboardData).items;
            var file = null;
            if (items && items.length) {
                // 搜索剪切板items
                for (var i = 0; i < items.length; i++) {
                    if (items[i].type.indexOf("image") !== -1) {
                        file = items[i].getAsFile();
                        break;
                    }
                }
            }
            if (file) {
                var reader = new FileReader();
                reader.onload = function(event) {
                    document.execCommand(
                        "insertImage",
                        false,
                        event.target.result
                    );
                };
                reader.readAsDataURL(file);
            } else {
                var clipboardData = e.clipboardData || window.clipboardData;
                var pastedData = clipboardData.getData("Text");
                document.execCommand("insertText", false, pastedData);
            }
        },

        onChange(id) {
            this.$emit("change", id);
        },

        onBlur(e) {
            if (!this.mind.focus) {
                this.data.h = 0;
                this.$nextTick(function() {
                    var rect = this.$refs.node.getBoundingClientRect();
                    this.data.w = Math.floor(rect.width);
                    this.data.h = Math.max(Math.floor(rect.height), 80);
                    this.data.content = e.target.textContent;
                    this.onChange();
                });
            }
        },
        dragStart(e) {
            if (this.data.isRoot) {
                window.clearTimeout(this.timeId);
                this.timeId = setTimeout(() => {
                    this.drag.start = true;
                    this.drag.x1 = e.pageX;
                    this.drag.y1 = e.pageY;
                    this.drag.x = this.data.x;
                    this.drag.y = this.data.startY;
                }, 200);
            }
        },
        dragging(e) {
            if (this.drag.start) {
                this.data.x = this.drag.x + e.pageX - this.drag.x1;
                this.data.startY = this.drag.y + e.pageY - this.drag.y1;
                this.$emit("change");
            }
        },
        dragEnd() {
            window.clearTimeout(this.timeId);
            if (this.drag.start) {
                this.drag.start = false;
            }
        }
    }
};
</script>

<style lang="less">
.node {
    box-sizing: border-box;
    border: 2px solid #333333;
    border-radius: 5px;
    max-width: 200px;
    padding: 15px;
    color: #333;
    font-size: 12px;

    position: absolute;
    outline: none;

    user-modify: read-write-plaintext-only;
    // -webkit-user-modify: read-write-plaintext-only;
    img {
        display: block;
        max-width: 100%;
    }
    &.is-root {
        user-select: none;
    }
    &:focus {
        border-color: #1396f4;
        box-shadow: fade(#1396f4, 30%) 0 0 0 4px;
        background-color: #fff;
        z-index: 2;
    }
}
</style>