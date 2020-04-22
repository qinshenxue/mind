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
        data: Array
    },
    data() {
        return {
            links: [],
            width: 0,
            height: 0,
            nodes: this.data || [
                {
                    id: 1,
                    deep: 0,
                    isRoot: true,
                    x: 10,
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
            let deep = 0;
            const calc = node => {
                if (node.children.length && node.expand) {
                    var h = 0;
                    node.children.forEach(item => {
                        h += calc(item);
                    });
                    node.childRealHeight = h;
                    node.childrenHeight = Math.max(h, node.h + node.gap);
                    return h;
                } else {
                    node.childrenHeight = node.h + node.gap;
                    deep = Math.max(deep, node.deep);
                    return node.childrenHeight;
                }
            };
            calc(this.nodes[0]);

            const calcStartY = obj => {
                if (obj.children.length) {
                    let startY = obj.startY;

                    if (obj.childRealHeight < obj.h + obj.gap) {
                        startY =
                            obj.startY +
                            (obj.h + obj.gap - obj.childRealHeight) / 2;
                    }
                    [
                        {
                            startY,
                            childrenHeight: 0
                        },
                        ...obj.children
                    ].reduce((a, b) => {
                        b.startY = a.startY + a.childrenHeight;
                        calcStartY(b);
                        return b;
                    });
                }
            };
            calcStartY(this.nodes[0]);
            this.width = this.nodes[0].x + (deep + 1) * 300 + 10 + "px";
        },

        calcRoot() {
            this.nodes.forEach(item => {
                this.height = item.y + item.childrenHeight + "px";
                item.y = item.childrenHeight / 2 - item.h / 2;
            });
        },
        calcPos() {
            const calc = obj => {
                if (obj.expand && obj.children.length) {
                    obj.children.forEach(child => {
                        calc(child);
                    });

                    const first = obj.children[0];
                    const last = obj.children[obj.children.length - 1];
                    const y1 =
                        first.y +
                        first.h / 2 +
                        (last.y + last.h / 2 - first.y - first.h / 2) / 2 -
                        obj.h / 2;

                    // obj.y =
                    //     obj.childrenHeight <= obj.h + obj.gap
                    //         ? (obj.startY + obj.gap / 2)
                    //         : y1;
                    obj.y = y1;
                } else {
                    obj.y = obj.startY + obj.gap / 2;
                }
            };
            calc(this.nodes[0]);
        },
        calcLinks() {
            const links = [];

            const calc = arr => {
                arr.forEach(item => {
                    const p1 = [
                        item.x + item.w,
                        item.y + Math.floor(item.h / 2)
                    ];
                    const p2 = [p1[0] + 20, p1[1]];
                    const p3 = [p2[0] + 16, p1[1]];

                    const circle = `M${p2.join(" ")} A 8 8, 0, 1, 0, ${p3.join(
                        " "
                    )} A 8 8, 0, 1, 0, ${p2.join(" ")}`;

                    const minus = `M${p2[0] + 4} ${p1[1]} L${p3[0] - 4} ${
                        p1[1]
                    }`;
                    const vline = `M${p2[0] + (p3[0] - p2[0]) / 2} ${p1[1] -
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
                                const childPoint = [
                                    child.x,
                                    child.y + Math.floor(child.h / 2)
                                ];
                                const qpoint = [
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
