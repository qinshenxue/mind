
<template>
    <div>
        <div class="node" :id="'node-'+data.id" @blur="onBlur" contenteditable
             :style="{
        top:data.y+'px',
        left:data.x+'px',
        minWidth:data.w+'px',
        minHeight:data.h+'px'
    }" @keydown.ctrl.enter.prevent="addChildNode"
             @keydown.ctrl.delete.prevent="$emit('ctrl-delete')"
             @keydown.alt.enter.prevent="$emit('alt-enter')" ref="node">
            {{data.content}}
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
        /*   document.addEventListener("mousemove", this.dragging);
        document.addEventListener("mouseup", this.dragEnd); */
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
        onChange(id) {
            this.$emit("change", id);
        },

        onBlur(e) {
            console.log(this.mind.focus);
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
        }
        // dragStart(e) {
        //     e.stopPropagation();
        //     this.drag.start = true;
        //     this.drag.x1 = e.pageX;
        //     this.drag.y1 = e.pageY;
        //     this.drag.top = this.top;
        //     this.drag.left = this.left;
        // },
        // dragging(e) {
        //     if (this.drag.start) {
        //         this.top = this.drag.top + e.pageY - this.drag.y1;
        //         //this.$emit("update:x", this.left);
        //         this.$emit("update:y", this.top);
        //     }
        // },
        // dragEnd() {
        //     if (this.drag.start) {
        //         this.drag.start = false;
        //     }
        // }
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
    &:focus {
        border-color: #1396f4;
        box-shadow: fade(#1396f4, 30%) 0 0 0 4px;
        background-color: #fff;
        z-index: 2;
    }
}
</style>