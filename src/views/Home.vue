<template>
  <div class="main">
    <div id="container">
      <div id="container_left"></div>
      <div class="container_right">
        <div class="container_right_header">
          <span @click.stop="saveData">保存</span>
        </div>
        <div id="container_body"></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
// import _ from 'lodash'
import { Graph, Addon, Shape } from "@antv/x6";

const DATA = {
  // 节点
  nodes: [
    {
      id: "node1", // String，可选，节点的唯一标识
      x: 40, // Number，必选，节点位置的 x 值
      y: 40, // Number，必选，节点位置的 y 值
      width: 80, // Number，可选，节点大小的 width 值
      height: 40, // Number，可选，节点大小的 height 值
      label: "hello", // String，节点标签
    },
    {
      id: "node2", // String，节点的唯一标识
      x: 160, // Number，必选，节点位置的 x 值
      y: 180, // Number，必选，节点位置的 y 值
      width: 80, // Number，可选，节点大小的 width 值
      height: 40, // Number，可选，节点大小的 height 值
      label: "world", // String，节点标签
    },
  ],
  // 边
  edges: [
    {
      source: "node1", // String，必须，起始节点 id
      target: "node2", // String，必须，目标节点 id
    },
  ],
};
const SCALE_STEP = 0.1;
@Component({})
export default class Home extends Vue {
  graph: any;

  mounted(): void {
    // 初始化画布
    this.initGraph();
    // 自定义默认node配置
    this.initMyRectNode();
    // 初始化左侧 预置node
    this.initStencil();
    // 初始化画布数据
    this.drawGraph();
  }
  beforeDestroy() {
    this.graph.dispose();
  }
  initGraph(): void {
    this.graph = new Graph({
      container: document.getElementById("container_body")!,
      background: {
        color: "#fffbe6", // 设置画布背景颜色
      },
      grid: {
        size: 10, // 网格大小 10px
        visible: true, // 渲染网格背景
      },
      panning: {
        enabled: true, // 画布平移
      },
      translating: {
        restrict: true, // 节点不能移动超出画布区域
      },
      snapline: true,
      connecting: {
        snap: {
          radius: 30, // 连线的过程中距离节点或者连接桩 30px 时会触发自动吸附
        },
        allowBlank: false, // 是否允许连接到画布空白位置的点
        // highlight: true,
        validateConnection({ sourceCell, targetCell }) {
          console.log(sourceCell?.data.type);
          console.log(targetCell?.data.type);
          return true;
        },
        router: {
          name: "manhattan",
        },
      },
    });

    this.graph.on("cell:click", (res: any) => {
      let {cell, e} = res
      console.log(res);

    });

    this.graph.on("cell:dblclick", (res: any) => {
      let {cell, e} = res
      console.log(2,cell, e);

      const isNode = cell.isNode();
      const isEdge = cell.isEdge();
      console.log(isNode, isEdge);

    });
  }
  initStencil() {
    const stencil = new Addon.Stencil({
      target: this.graph,
      // validateNode(node, options) {
      //   console.log(node, options);
      //   return true;
      // },
      stencilGraphWidth: 300,
      stencilGraphHeight: 300,
      groups: [{ name: "basic", title: "基础节点" }],
    });
    document.getElementById("container_left")!.appendChild(stencil.container);
    stencil.load(
      [
        this._getGraphNode({
          nodeType: "my-rect",
          label: "卡片",
          type: "card",
        }),
        this._getGraphNode({
          nodeType: "my-rect",
          label: "指标",
          type: "indicator",
        }),
        this._getGraphNode({
          nodeType: "my-rect",
          label: "OLAP",
          type: "olap",
        }),
        this._getGraphNode({
          nodeType: "my-rect",
          label: "数据源",
          type: "datasource",
        }),
      ],
      "basic"
    );
  }
  initMyRectNode() {
    Shape.Rect.define({
      shape: "my-rect",
      width: 180,
      height: 80,
      ports: {
        groups: {
          top: {
            position: "top",
            label: {
              position: "top",
            },
            attrs: {
              circle: {
                r: 3,
                magnet: true,
                stroke: "#31d0c6",
                strokeWidth: 2,
                fill: "#fff",
              },
            },
          },
          bottom: {
            position: "bottom",
            label: {
              position: "bottom",
            },
            attrs: {
              circle: {
                r: 3,
                magnet: true,
                stroke: "#31d0c6",
                strokeWidth: 2,
                fill: "#fff",
              },
            },
          },
        },
      },
    });
  }
  drawGraph(): void {
    if (sessionStorage.getItem("hahaha")) {
      this.graph.fromJSON(JSON.parse(sessionStorage.getItem("hahaha")!));
    }
    // this.graph.fromJSON(DATA);
  }
  _getGraphNode(attr: { nodeType: string; label: string; type: string }) {
    return this.graph.createNode({
      shape: attr.nodeType,
      x: 40, // Number，必选，节点位置的 x 值
      y: 40, // Number，必选，节点位置的 y 值
      width: 80, // Number，可选，节点大小的 width 值
      height: 40, // Number，可选，节点大小的 height 值
      label: attr.label, // String，节点标签
      ports: [
        {
          group: "top", // 指定分组名称
        },
        {
          group: "top", // 指定分组名称
        },
        {
          group: "bottom", // 指定分组名称
        },
      ],
      data: { type: attr.type },
    });

    // return this.graph.createNode({
    //   shape: attr.nodeType,
    //   x: 40, // Number，必选，节点位置的 x 值
    //   y: 40, // Number，必选，节点位置的 y 值
    //   width: 80, // Number，可选，节点大小的 width 值
    //   height: 40, // Number，可选，节点大小的 height 值
    //   label: "hello", // String，节点标签
    // });
  }

  // 画布缩放
  zoomGraph(flag: string): void {
    this.graph.zoom(flag === "up" ? SCALE_STEP : -SCALE_STEP);
  }

  saveData() {
    console.log(this.graph.toJSON());
    sessionStorage.setItem("hahaha", JSON.stringify(this.graph.toJSON()));
  }
}
</script>
<style lang="scss" scope>
.x6-port-body {
  opacity: 0;
}
.x6-node:hover .x6-port-body {
  opacity: 1;
}
.main {
  height: 100%;
  #container {
    display: flex;
    height: 100%;
    #container_left {
      width: 300px;
      height: 100%;
      position: relative;
    }
    .container_right {
      width: calc(100% - 300px);
      height: 100%;
      .container_right_header {
        height: 50px;
      }
      #container_body {
        height: calc(100% - 50px);
      }
    }
  }
}
</style>
