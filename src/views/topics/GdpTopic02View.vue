<template>
  <TopicMeta :topic="topic" />
  <ATypographyParagraph> </ATypographyParagraph>
  <div id="ceva-theorem-wrapper">
    <ATypographyTitle :level="4">Animated Ceva Theorem</ATypographyTitle>
    <canvas id="ceva-theorem-canvas" width="500" height="500" />
  </div>

  <ATypographyParagraph> </ATypographyParagraph>
  <div id="menelaus-theorem-wrapper">
    <ATypographyTitle :level="4">Animated Menelaus Theorems</ATypographyTitle>
    <canvas id="menelaus-theorem-canvas" width="500" height="500" />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { indexTopicMap } from "@/data";
import { Topic } from "@/types";
import { fabric } from "fabric";
import { makeCircle, makeLabel, makeLine, makeMovablePoint } from "@/utils/canvas";
import {
  calculateDistanceBetweenTwoPoints,
  calculateLineIntersectInLinearEquation,
  isInside,
  solveLinearEquation,
} from "@/utils/geometry";
import { IEvent, Intersection } from "fabric/fabric-impl";

const topic = indexTopicMap.get(2) as Topic;

type Circle = fabric.Circle & {
  // eslint-disable-next-line
  [key: string]: any;
  topBound?: Circle;
  bottomBound?: Circle;
};

type Line = fabric.Line & {
  p1?: Circle;
  p2?: Circle;
  m?: number;
  b?: number;
};

export default defineComponent({
  setup() {
    return { topic };
  },
  mounted() {
    // Ceva Theorem
    (() => {
      const canvas = new fabric.Canvas("ceva-theorem-canvas", {
        selection: false,
        backgroundColor: "floralwhite",
      });

      const lineAD = makeLine();
      const lineBE = makeLine();
      const lineCF = makeLine();

      const aLabel = makeLabel("A");
      const bLabel = makeLabel("B");
      const cLabel = makeLabel("C");
      const dLabel = makeLabel("D");
      const eLabel = makeLabel("E");
      const fLabel = makeLabel("F");
      const pLabel = makeLabel("P");

      const dNode = makeCircle();
      const eNode = makeCircle();
      const fNode = makeCircle();

      const valueBD = makeLabel("0");
      const valueDC = makeLabel("0");
      const valueCE = makeLabel("0");
      const valueEA = makeLabel("0");
      const valueAF = makeLabel("0");
      const valueFB = makeLabel("0");
      const resultText = makeLabel("1");

      // create triangle
      const triangle = new fabric.Polygon(
        [
          { x: 150, y: 100 },
          { x: 50, y: 400 },
          { x: 350, y: 400 },
        ],
        {
          fill: "transparent",
          strokeWidth: 1.5,
          stroke: "black",
          objectCaching: false,
          transparentCorners: false,
          cornerStyle: "circle",
          cornerColor: "transparent",
          cornerSize: 20,
          hasBorders: false,
          evented: false,
        }
      );

      const vertices = [
        new fabric.Point(150, 100),
        new fabric.Point(50, 400),
        new fabric.Point(350, 400),
      ];

      // set coordinates and options for point P inside the triangle
      const pointP = makeMovablePoint(new fabric.Point(180, 310));

      pointP.set({
        originX: "center",
        originY: "center",
        radius: 3,
        padding: 20,
        fill: "red",
      });

      // function to find points D,E,F and draw lines that depends on coordinates of point P
      function movePointP() {
        // find slope and intercept of the line segments from vertices to point P
        const points = triangle.points as fabric.Point[];
        const lineAP = solveLinearEquation(points[0], new fabric.Point(pointP.left!, pointP.top!));
        const lineBP = solveLinearEquation(points[1], new fabric.Point(pointP.left!, pointP.top!));
        const lineCP = solveLinearEquation(points[2], new fabric.Point(pointP.left!, pointP.top!));

        // find slope and intercept of the line segments between the vertices
        const lineAB = solveLinearEquation(points[0], points[1]);
        const lineBC = solveLinearEquation(points[1], points[2]);
        const lineAC = solveLinearEquation(points[0], points[2]);

        // calculate intersection points D,E,F
        const pointD = calculateLineIntersectInLinearEquation(
          lineAP.m,
          lineAP.b,
          lineBC.m,
          lineBC.b
        );

        const pointE = calculateLineIntersectInLinearEquation(
          lineBP.m,
          lineBP.b,
          lineAC.m,
          lineAC.b
        );

        const pointF = calculateLineIntersectInLinearEquation(
          lineCP.m,
          lineCP.b,
          lineAB.m,
          lineAB.b
        );

        // draw lines from vertices to points D,E,F
        lineAD.set({
          x1: points[0].x,
          y1: points[0].y,
          x2: pointD.x,
          y2: pointD.y,
          stroke: "green",
        });

        lineBE.set({
          x1: points[1].x,
          y1: points[1].y,
          x2: pointE.x,
          y2: pointE.y,
          stroke: "green",
        });

        lineCF.set({
          x1: points[2].x,
          y1: points[2].y,
          x2: pointF.x,
          y2: pointF.y,
          stroke: "green",
        });

        // set coordinates for the labels
        aLabel.set({
          left: points[0].x - 5,
          top: points[0].y - 25,
          fontSize: 20,
        });

        bLabel.set({
          left: points[1].x - 10,
          top: points[1].y + 3,
          fontSize: 20,
        });

        cLabel.set({
          left: points[2].x,
          top: points[2].y,
          fontSize: 20,
        });

        dLabel.set({
          left: pointD.x - 5,
          top: pointD.y + 10,
          fontSize: 20,
          fill: "blue",
        });

        eLabel.set({
          left: pointE.x + 15,
          top: pointE.y - 15,
          fontSize: 20,
          fill: "blue",
        });

        fLabel.set({
          left: pointF.x - 20,
          top: pointF.y - 15,
          fontSize: 20,
          fill: "blue",
        });

        pLabel.set({
          left: (pointP.left as number) - 25,
          top: (pointP.top as number) - 10,
          fontSize: 20,
          fill: "red",
        });

        // set coordinates for the nodes
        dNode.set({
          left: pointD.x - 0.75 * 3,
          top: pointD.y - 0.75 * 3,
          radius: 2,
          fill: "blacks",
        });

        eNode.set({
          left: pointE.x - 0.75 * 3,
          top: pointE.y - 0.75 * 3,
          radius: 2,
          fill: "blacks",
        });

        fNode.set({
          left: pointF.x - 0.75 * 3,
          top: pointF.y - 0.75 * 3,
          radius: 2,
          fill: "blacks",
        });

        // calculate values for text
        const distBD = calculateDistanceBetweenTwoPoints(points[1], pointD) / 100;
        const distDC = calculateDistanceBetweenTwoPoints(points[2], pointD) / 100;
        const distCE = calculateDistanceBetweenTwoPoints(points[2], pointE) / 100;
        const distEA = calculateDistanceBetweenTwoPoints(points[0], pointE) / 100;
        const distAF = calculateDistanceBetweenTwoPoints(points[0], pointF) / 100;
        const distFB = calculateDistanceBetweenTwoPoints(points[1], pointF) / 100;

        // update values for text
        const valueList = [
          [distBD, distDC],
          [distCE, distEA],
          [distAF, distFB],
        ];
        const valueConst = [
          [valueBD, valueDC],
          [valueCE, valueEA],
          [valueAF, valueFB],
        ];

        valueList.forEach((value, i) => {
          const num = [
            String((Math.round(value[0] * 100) / 100).toFixed(2)),
            String((Math.round(value[1] * 100) / 100).toFixed(2)),
          ];

          valueConst[i][0].set({
            text: num[0],
            left: 330 + i * 50,
            top: 85,
            fontSize: 20,
            stroke: "black",
          });

          valueConst[i][1].set({
            text: num[1],
            left: 330 + i * 50,
            top: 110,
            fontSize: 20,
            stroke: "black",
          });
        });

        // calculate and update result
        const resultValue = Math.round(((((distBD / distDC) * distCE) / distEA) * distAF) / distFB);
        resultText.set({
          text: "=  " + String(resultValue.toFixed(2)),
          left: 306,
          top: 155,
          fontSize: 20,
          stroke: "black",
        });
      }

      const movePoint = (e: IEvent): void => {
        const p = e.target! as Circle;
        if (
          isInside(
            vertices[0].x,
            vertices[0].y,
            vertices[1].x,
            vertices[1].y,
            vertices[2].x,
            vertices[2].y,
            p.left as number,
            p.top as number
          )
        ) {
          p.set({
            left: p.left as number,
            top: p.top as number,
          });
        } else {
          return;
        }
        movePointP();
      };

      movePointP();
      canvas.on("object:moving", movePoint);

      const equationText = [
        ["BD", "DC"],
        ["CE", "EA"],
        ["AF", "FB"],
      ];
      equationText.forEach((equation, i) => {
        canvas.add(
          new fabric.Text(equation[0]).set({
            left: 335 + i * 50,
            top: 25,
            strokeWidth: 1,
            stroke: "black",
            fill: "black",
            fontSize: 20,
            evented: false,
          })
        );

        canvas.add(
          new fabric.Text(equation[1]).set({
            left: 335 + i * 50,
            top: 50,
            strokeWidth: 1,
            stroke: "black",
            fill: "black",
            fontSize: 20,
            evented: false,
          })
        );
      });

      const signCoords = [27, 87];
      signCoords.forEach((signCoord) => {
        canvas.add(
          new fabric.Text("____  ____  ____", {
            left: 330,
            top: signCoord,
            strokeWidth: 1,
            stroke: "black",
            fill: "black",
            fontSize: 20,
            evented: false,
          })
        );

        canvas.add(
          new fabric.Text("             .         .", {
            left: 306,
            top: signCoord + 5,
            strokeWidth: 1,
            stroke: "black",
            fill: "black",
            fontSize: 20,
            evented: false,
          })
        );
      });

      canvas.add(triangle);
      canvas.add(lineAD);
      canvas.add(lineBE);
      canvas.add(lineCF);

      canvas.add(aLabel);
      canvas.add(bLabel);
      canvas.add(cLabel);
      canvas.add(dLabel);
      canvas.add(eLabel);
      canvas.add(fLabel);
      canvas.add(pLabel);

      canvas.add(pointP);
      canvas.add(dNode);
      canvas.add(eNode);
      canvas.add(fNode);

      canvas.add(valueBD);
      canvas.add(valueDC);
      canvas.add(valueCE);
      canvas.add(valueEA);
      canvas.add(valueAF);
      canvas.add(valueFB);
      canvas.add(resultText);
    })();

    // Menelaus Theorems
    (() => {
      const canvas = new fabric.Canvas("menelaus-theorem-canvas", {
        selection: false,
        backgroundColor: "floralwhite",
      });

      function createCircle(x?: number, y?: number, radius?: number, fill?: string): fabric.Circle {
        return new fabric.Circle({
          originX: "center",
          originY: "center",
          left: x || 0,
          top: y || 0,
          hasControls: false,
          hasBorders: false,
          evented: false,
          radius: radius || 1,
          fill: fill || "transparent",
          padding: 3,
        });
      }

      function createPolygon(vertices?: fabric.Point[]): fabric.Polygon {
        return new fabric.Polygon(vertices || [], {
          fill: "transparent",
          stroke: "black",
          strokeWidth: 1.5,
          objectCaching: false,
          transparentCorners: false,
          cornerColor: "transparent",
          hasBorders: false,
          hasControls: false,
          hoverCursor: "normal",
          lockMovementX: true,
          lockMovementY: true,
        });
      }

      function createLabel(text: string, fontSize?: number, color?: string) {
        return new fabric.Text(text, {
          hasControls: false,
          hasBorders: false,
          evented: false,
          fontSize: fontSize || 24,
          fill: color || "black",
        });
      }

      function createLine(points?: number[], color?: string, strokeWidth?: number) {
        return new fabric.Line(points || [], {
          hasControls: false,
          hasBorders: false,
          evented: false,
          stroke: color || "black",
          strokeWidth: strokeWidth || 1,
        });
      }

      const movablePointQ = createCircle(450, 150, 3, "black").set({
        padding: 20,
        evented: true,
        lockMovementX: true,
      });
      const movablePointP = createCircle(40, 390, 3, "black").set({
        padding: 20,
        evented: true,
        lockMovementX: true,
      });

      const triangle = createPolygon();

      const dNode = createCircle(0, 0, 2.5, "red");
      const eNode = createCircle(0, 0, 2.5, "red");
      const fNode = createCircle(0, 0, 2.5, "red");

      const linePQ = createLine([], "green");
      const lineBC = createLine();

      const linePA = makeLine() as Line;
      const linePB = makeLine() as Line;
      const lineQA = makeLine() as Line;
      const lineQB = makeLine() as Line;
      const lineQG = makeLine() as Line;

      const aLabel = createLabel("A");
      const bLabel = createLabel("B");
      const cLabel = createLabel("C");
      const dLabel = createLabel("D");
      const pLabel = createLabel("P");
      const qLabel = createLabel("Q");
      const eLabel = createLabel("E");
      const fLabel = createLabel("F");

      const vertices = [
        new fabric.Point(340, 150),
        new fabric.Point(170, 350),
        new fabric.Point(440, 350),
      ];
      triangle.set({
        points: vertices,
      });
      aLabel.set({
        left: vertices[0].x - 10,
        top: vertices[0].y - 35,
      });
      bLabel.set({
        left: vertices[1].x - 20,
        top: vertices[1].y,
      });
      cLabel.set({
        left: vertices[2].x,
        top: vertices[2].y - 2,
      });
      const extendPoint = new fabric.Point(
        vertices[1].lerp(vertices[2], -0.5).x,
        vertices[1].lerp(vertices[2], -0.5).y
      );
      lineBC.set({
        x1: vertices[2].x,
        y1: vertices[2].y,
        x2: extendPoint.x,
        y2: extendPoint.y,
      });

      const pointP = new fabric.Point(movablePointP.left as number, movablePointP.top as number);
      const pointQ = new fabric.Point(movablePointQ.left as number, movablePointQ.top as number);

      let pointE = (
        fabric.Intersection.intersectLineLine(
          pointP,
          pointQ,
          vertices[0],
          vertices[1]
        ) as Intersection
      ).points![0];

      let pointF = (
        fabric.Intersection.intersectLineLine(
          pointP,
          pointQ,
          vertices[0],
          vertices[2]
        ) as Intersection
      ).points![0];

      let pointD = (
        fabric.Intersection.intersectLineLine(
          pointP,
          pointQ,
          vertices[1],
          extendPoint
        ) as Intersection
      ).points![0];

      dNode.set({
        left: pointD.x,
        top: pointD.y,
      });
      eNode.set({
        left: pointE.x,
        top: pointE.y,
      });
      fNode.set({
        left: pointF.x,
        top: pointF.y,
      });

      linePQ.set({
        x1: movablePointQ.left,
        y1: movablePointQ.top,
        x2: movablePointP.left,
        y2: movablePointP.top,
      });

      pLabel.set({
        left: pointP.x - 15,
        top: pointP.y,
        fill: "red",
      });
      qLabel.set({
        left: movablePointQ.left,
        top: movablePointQ.top,
        fill: "red",
      });

      dLabel.set({
        left: pointD.x - 20,
        top: pointD.y - 25,
        fill: "blue",
        fontSize: 20,
      });
      eLabel.set({
        left: pointE.x - 20,
        top: pointE.y - 25,
        fill: "blue",
        fontSize: 20,
      });
      fLabel.set({
        left: pointF.x - 3,
        top: pointF.y - 30,
        fill: "blue",
        fontSize: 20,
      });

      const onPointMove = (e: IEvent): void => {
        const p = e.target! as Circle;

        pointP.x = movablePointP.left as number;
        pointP.y = movablePointP.top as number;
        pointQ.x = movablePointQ.left as number;
        pointQ.y = movablePointQ.top as number;

        Object.assign(
          linePA,
          solveLinearEquation({ x: movablePointP.left!, y: movablePointP.top! }, vertices[0])
        );
        Object.assign(
          linePB,
          solveLinearEquation({ x: movablePointP.left!, y: movablePointP.top! }, vertices[1])
        );
        Object.assign(
          lineQA,
          solveLinearEquation({ x: movablePointQ.left!, y: movablePointQ.top! }, vertices[0])
        );
        Object.assign(
          lineQB,
          solveLinearEquation({ x: movablePointQ.left!, y: movablePointQ.top! }, vertices[1])
        );
        Object.assign(
          lineQG,
          solveLinearEquation({ x: movablePointQ.left!, y: movablePointQ.top! }, extendPoint)
        );

        if (pointE === pointF) {
          if ((p.left as number) === 450) {
            const y = linePA.m! * p.left! + linePA.b!;
            p.set("top", y);
          } else if ((p.left as number) === 40) {
            const y = lineQA.m! * p.left! + lineQA.b!;
            p.set("top", y);
          }
        }
        if (pointD === pointE) {
          if ((p.left as number) === 450) {
            const y = linePB.m! * p.left! + linePB.b!;
            p.set("top", y);
          } else if ((p.left as number) === 40) {
            const y = lineQB.m! * p.left! + lineQB.b!;
            p.set("top", y);
          }
        }
        if (pointD === undefined && pointE !== undefined) {
          if ((p.left as number) === 40) {
            const y = lineQG.m! * p.left! + lineQG.b!;
            p.set("top", y);
          }
        }

        // const p1 = [movablePointP.left as number, movablePointP.top as number];
        // const q1 = [movablePointQ.left as number, movablePointQ.top as number];

        // const a1 = [vertices[0].x, vertices[0].y];
        // const b1 = [vertices[1].x, vertices[1].y];
        // const c1 = [vertices[2].x, vertices[2].y];
        // const ex1 = [extendPoint.x, extendPoint.y];

        // const E1 = lineLineIntersection(p1, q1, a1, b1);
        // const F1 = lineLineIntersection(p1, q1, a1, c1);
        // const D1 = lineLineIntersection(p1, q1, ex1, c1);

        pointE = (
          fabric.Intersection.intersectLineLine(
            pointP,
            pointQ,
            vertices[0],
            vertices[1]
          ) as Intersection
        ).points![0];

        pointF = (
          fabric.Intersection.intersectLineLine(
            pointP,
            pointQ,
            vertices[0],
            vertices[2]
          ) as Intersection
        ).points![0];

        pointD = (
          fabric.Intersection.intersectLineLine(
            pointP,
            pointQ,
            vertices[1],
            extendPoint
          ) as Intersection
        ).points![0];

        if (pointD !== undefined && pointE !== undefined && pointF !== undefined) {
          linePQ.set({
            x1: movablePointQ.left,
            y1: movablePointQ.top,
            x2: movablePointP.left,
            y2: movablePointP.top,
          });

          dNode.set({
            left: pointD.x,
            top: pointD.y,
          });
          eNode.set({
            left: pointE.x,
            top: pointE.y,
          });

          fNode.set({
            left: pointF.x,
            top: pointF.y,
          });

          pLabel.set({
            left: pointP.x - 15,
            top: pointP.y,
            fill: "red",
          });
          qLabel.set({
            left: movablePointQ.left,
            top: movablePointQ.top,
            fill: "red",
          });

          dLabel.set({
            left: pointD.x - 20,
            top: pointD.y - 25,
            fill: "blue",
            fontSize: 20,
          });
          eLabel.set({
            left: pointE.x - 20,
            top: pointE.y - 25,
            fill: "blue",
            fontSize: 20,
          });
          fLabel.set({
            left: pointF.x - 3,
            top: pointF.y - 30,
            fill: "blue",
            fontSize: 20,
          });
        }
      };

      canvas.on("object:moving", onPointMove);
      canvas.add(triangle);
      canvas.add(movablePointP);
      canvas.add(movablePointQ);
      canvas.add(dNode);
      canvas.add(eNode);
      canvas.add(fNode);
      canvas.add(linePQ);
      canvas.add(lineBC);
      canvas.add(aLabel);
      canvas.add(bLabel);
      canvas.add(cLabel);
      canvas.add(dLabel);
      canvas.add(eLabel);
      canvas.add(fLabel);
      canvas.add(pLabel);
      canvas.add(qLabel);
    })();
  },
});
</script>
