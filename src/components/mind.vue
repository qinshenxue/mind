<template>
    <div class="mind" :style="{
      width:width,
      height:height,
    }">

        <svg :style="{
      width:width,
      height:height,
    }" version="1.1" xmlns="http://www.w3.org/2000/svg">
            <path @click="expandChild(item)" v-for="(item, index) in links"
                  :key="index" :d="item.d" stroke="#333" fill="transparent"
                  stroke-width="2px">
            </path>
        </svg>

        <Node @change="onChange" v-for="(item, index) in nodes" :key="index"
              :data="item">
        </Node>
    </div>
</template>

<script>
import Node from "./node";
export default {
    name: "mind",
    components: {
        Node
    },
    props: {
        msg: String
    },
    data() {
        return {
            links: [],
            width: 0,
            height: 0,
            nodes: [
                {
                    id: 1,
                    deep: 0,
                    isRoot: true,
                    x: Math.floor(window.innerWidth / 2 - 50),
                    y: Math.floor(window.innerHeight / 2 - 40),
                    startY: Math.floor(window.innerHeight / 2 - 60),
                    w: 100,
                    h: 80,
                    gap: 40,
                    content: "中心主题",
                    expand: true,
                    childrenHeight: 0,
                    children: []
                }
            ]
        };
    },
    created() {
        this.onChange();
    },
    computed: {},
    methods: {
        onChange(id) {
            this.calcChildrenHeight();
            this.calcRoot();
            this.calcPos();
            this.calcLinks();

            if (id) {
                this.$nextTick(function() {
                    const el = document.getElementById(`node-${id}`);

                    if (el) {
                        const rect = el.getBoundingClientRect();
                        if (rect.bottom > window.innerHeight) {
                            document.documentElement.scrollTop +=
                                rect.bottom - window.innerHeight;
                        } else if (rect.top < 0) {
                            document.documentElement.scrollTop += rect.top;
                        }
                    }
                });
            }
        },

        expandChild(data) {
            if (data.type == "expand") {
                data.node.expand = !data.node.expand;
                this.onChange(data.node.id);
            }
        },

        calcChildrenHeight() {
            var deep = 0;
            var calc = node => {
                if (node.children.length && node.expand) {
                    var h = 0;
                    node.children.forEach(item => {
                        h += calc(item);
                    });
                    node.childrenHeight = Math.max(h, node.h + node.gap);
                    return h;
                } else {
                    node.childrenHeight = node.h + node.gap;
                    deep = Math.max(deep, node.deep);
                    return node.childrenHeight;
                }
            };
            calc(this.nodes[0]);

            var startY = obj => {
                if (obj.children.length) {
                    [
                        {
                            startY: obj.startY,
                            childrenHeight: 0
                        },
                        ...obj.children
                    ].reduce((a, b) => {
                        b.startY = a.startY + a.childrenHeight;
                        startY(b);
                        return b;
                    });
                }
            };
            startY(this.nodes[0]);
            this.width = this.nodes[0].x + (deep + 1) * 300 + 10 + "px";
        },

        calcRoot() {
            this.nodes.forEach(item => {
                this.height = item.y + item.childrenHeight + "px";
                item.y = item.childrenHeight / 2 - item.h / 2;
            });
        },
        calcPos() {
            var calc = obj => {
                if (obj.expand && obj.children.length) {
                    obj.children.forEach(child => {
                        calc(child);
                    });

                    var first = obj.children[0];
                    var last = obj.children[obj.children.length - 1];
                    var y1 =
                        first.y +
                        first.h / 2 +
                        (last.y + last.h / 2 - first.y - first.h / 2) / 2 -
                        obj.h / 2;

                    obj.y = y1;
                } else {
                    obj.y = obj.startY + obj.gap / 2;
                }
            };
            calc(this.nodes[0]);
        },
        calcLinks() {
            var links = [];

            var calc = arr => {
                arr.forEach(item => {
                    var p1 = [item.x + item.w, item.y + Math.floor(item.h / 2)];
                    var p2 = [p1[0] + 20, p1[1]];
                    var p3 = [p2[0] + 16, p1[1]];

                    var circle = `M${p2.join(" ")} A 8 8, 0, 1, 0, ${p3.join(
                        " "
                    )} A 8 8, 0, 1, 0, ${p2.join(" ")}`;

                    var minus = `M${p2[0] + 4} ${p1[1]} L${p3[0] - 4} ${p1[1]}`;
                    var vline = `M${p2[0] + (p3[0] - p2[0]) / 2} ${p1[1] -
                        4} L${p2[0] + (p3[0] - p2[0]) / 2} ${p1[1] + 4}`;

                    if (item.children.length) {
                        links.push({
                            d: `M${p1.join(" ")} L${p2.join(" ")}`
                        });

                        if (item.expand) {
                            links.push({
                                type: "expand",
                                node: item,
                                d: `${circle} ${minus}`
                            });
                            item.children.forEach(child => {
                                var childPoint = [
                                    child.x,
                                    child.y + Math.floor(child.h / 2)
                                ];
                                var qpoint = [
                                    Math.floor((p3[0] + child.x) / 2) - 20,
                                    childPoint[1]
                                ];
                                links.push({
                                    d: `M${p3.join(" ")} Q ${qpoint.join(
                                        " "
                                    )} ${childPoint.join(" ")} `
                                });
                            });
                            calc(item.children);
                        } else {
                            var plus = ``;
                            links.push({
                                type: "expand",
                                node: item,
                                d: `${circle} ${minus} ${vline}`
                            });
                        }
                    }
                });
            };

            calc(this.nodes);
            this.links = links;
        }
    }
};
</script>

<style lang="less">
.mind {
    position: relative;
}
</style>
