
<template>
    <div>
        <div class="node" @blur="onBlur" contenteditable @mousedown="dragStart"
             :style="{
        top:data.y+'px',
        left:data.x+'px',
        minWidth:data.w+'px',
        minHeight:data.h+'px'
    }" @keydown.ctrl.enter.prevent="addChildNode" @keydown.alt.enter.prevent="$emit('alt-enter')"  ref="node">
            {{data.id}}<br>
            <!-- {{data.x}}<br>
            id:{{data.id}} -->
        </div>

        <template v-if="data.expand">
            <node @alt-enter="addBrotherNode(index)"
                  @change="onChange" v-for="(item, index) in data.children"
                  :key="index" :data="item">
            </node>
        </template>

    </div>

</template>

<script>
var id = 1;
export default {
    name: "node",
    props: {
        data: Object,
        index: Number,
        x: Number,
        y: Number,
        w: Number,
        h: Number
    },

    // watch: {
    //     x(v) {
    //         this.left = v;
    //     },
    //     y(v) {
    //         this.top = v;
    //     },
    //     w(v) {
    //         this.width = v;
    //     },
    //     h(v) {
    //         this.height = v;
    //     }
    // },

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
        addChildNode() {
            this.data.children.push({
                id: ++id,
                deep: this.data.deep + 1,
                x: this.data.x + 300,
                y: 0,
                w: 100,
                h: 80,
                gap: 40,
                childrenHeight: 0,
                content: "",
                expand: true,
                children: []
            });

            this.onChange();
        },
        addBrotherNode(i) {
            this.data.children.splice(i+1, 0, {
                id: ++id,
                deep: this.data.deep + 1,
                x: this.data.x + 300,
                y: 0,
                w: 100,
                h: 80,
                gap: 40,
                childrenHeight: 0,
                content: "",
                expand: true,
                children: []
            });
            this.onChange();
        },
        onChange() {
            // var h = 0;

            // this.data.children.forEach(item => {

            //     h += Math.max(
            //         this.data.childrenHeight +
            //             this.data.children.length * item.gap,
            //         item.h + item.gap
            //     );
            // });
            // this.data.childrenHeight = h;
            this.$emit("change");
        },

        onBlur(e) {
            this.data.h = 0;
            this.$nextTick(function() {
                var rect = this.$refs.node.getBoundingClientRect();
                this.data.w = rect.width;
                this.data.h = Math.max(rect.height, 80);
                this.data.content = e.data;
                this.onChange();
            });

            /* this.$emit("update:w", rect.width);
            this.$emit("update:h", rect.height); */
        },
        dragStart(e) {
            e.stopPropagation();
            this.drag.start = true;
            this.drag.x1 = e.pageX;
            this.drag.y1 = e.pageY;
            this.drag.top = this.top;
            this.drag.left = this.left;
        },
        dragging(e) {
            if (this.drag.start) {
                this.top = this.drag.top + e.pageY - this.drag.y1;
                //this.$emit("update:x", this.left);
                this.$emit("update:y", this.top);
            }
        },
        dragEnd() {
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
    &:focus {
        border-color: #1396f4;
        box-shadow: fade(#1396f4, 30%) 0 0 0 4px;
        background-color: #fff;
        z-index: 2;
    }
}
</style>