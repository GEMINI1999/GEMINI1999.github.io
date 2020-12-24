---
layout: article
title: "博客公式/流程图/图标测试"
key: 9
tags: 
        - test
toc: true
mathjax: true #公式
mathjax_autoNumber: true #公式自动编号
mermaid: true #流程图
chart: true #可交互式图标
---
## 公式
$$a \ne 0$$  
$$ax^2 + bx + c = 0$$  
$$x_1 = {-b + \sqrt{b^2-4ac} \over 2a}$$  
$$x_2 = {-b - \sqrt{b^2-4ac} \over 2a} \notag$$  

## 流程图
```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D
```

```mermaid
sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
```
## 图表
```chart
{
  "type": "line",
  "data": {
    "labels": [
      "January",
      "February",
      "March",
      "April",
      "May",
      "June",
      "July"
    ],
    "datasets": [
      {
        "label": "# of bugs",
        "fill": false,
        "lineTension": 0.1,
        "backgroundColor": "rgba(75,192,192,0.4)",
        "borderColor": "rgba(75,192,192,1)",
        "borderCapStyle": "butt",
        "borderDash": [],
        "borderDashOffset": 0,
        "borderJoinStyle": "miter",
        "pointBorderColor": "rgba(75,192,192,1)",
        "pointBackgroundColor": "#fff",
        "pointBorderWidth": 1,
        "pointHoverRadius": 5,
        "pointHoverBackgroundColor": "rgba(75,192,192,1)",
        "pointHoverBorderColor": "rgba(220,220,220,1)",
        "pointHoverBorderWidth": 2,
        "pointRadius": 1,
        "pointHitRadius": 10,
        "data": [
          65,
          59,
          80,
          81,
          56,
          55,
          40
        ],
        "spanGaps": false
      }
    ]
  },
  "options": {}
}
```

