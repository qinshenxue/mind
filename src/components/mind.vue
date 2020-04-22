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
    name: "HelloWorld",
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
                    x: 10,
                    y: 0,
                    startY: 0,
                    w: 100,
                    h: 80,
                    gap: 40,
                    content: "",
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
        onChange() {
            this.calcChildrenHeight();
            this.calcRoot();
            this.calcPos();
            this.calcLinks();
        },

        expandChild(data) {
            // console.log(data)
            if (data.type == "expand") {
                data.node.expand = !data.node.expand;
                this.onChange();
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
            this.width = (deep + 1) * 300 + 10 + "px";
        },

        calcRoot() {
            this.nodes.forEach(item => {
                this.height = item.childrenHeight + "px";
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
                    obj.y =
                        first.y +
                        first.h / 2 +
                        (last.y + last.h / 2 - first.y - first.h / 2) / 2 -
                        obj.h / 2;
                } else {
                    obj.y = obj.startY + obj.gap / 2;
                }
            };
            calc(this.nodes[0]);
        },
        calcLinks() {
            var links = [];

            var dg = arr => {
                arr.forEach(item => {
                    var p1 = [item.x + item.w + 40, item.y + item.h / 2];

                    if (item.children.length) {
                        links.push({
                            d: `M${p1[0] - 40} ${p1[1]} L${p1[0] - 16} ${p1[1]}`
                        });

                        if (item.expand) {
                            links.push({
                                type: "expand",
                                node: item,
                                d: `M${p1[0] - 16} ${p1[1]} A 8 8, 0, 1, 0, ${
                                    p1[0]
                                } ${p1[1]} A 8 8, 0, 1, 0, ${p1[0] - 16} ${
                                    p1[1]
                                } M${p1[0] - 12} ${p1[1]} L${p1[0] - 4} ${
                                    p1[1]
                                }`
                            });
                            item.children.forEach(child => {
                                var p3 = [child.x, child.y + child.h / 2];
                                var p2 = [
                                    (p1[0] + child.x) / 2,
                                    child.y + child.h / 2
                                ];
                                links.push({
                                    d: `M${p1.join(" ")} Q ${p2.join(
                                        " "
                                    )} ${p3.join(" ")} `
                                });
                            });
                            dg(item.children);
                        } else {
                            links.push({
                                type: "expand",
                                node: item,
                                d: `M${p1[0] - 16} ${p1[1]} A 8 8, 0, 1, 0, ${
                                    p1[0]
                                } ${p1[1]} A 8 8, 0, 1, 0, ${p1[0] - 16} ${
                                    p1[1]
                                } M${p1[0] - 12} ${p1[1]} L${p1[0] - 4} ${
                                    p1[1]
                                } M${p1[0] - 8} ${p1[1] - 4} L${p1[0] -
                                    8} ${p1[1] + 4}`
                            });
                        }
                    }
                });
            };

            dg(this.nodes);

            this.links = links;
        }
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less">
.mind {
    position: relative;
    svg {
    }
}
</style>
