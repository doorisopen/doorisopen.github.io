---
layout: post
title: "우아한테크코스 3기 지원후기(2차 코딩테스트)"
date: 2020-12-21 00:50:59
author: me
categories: Woowacourse
tags: woowacourse
cover: "/assets/images/2020/woowacourse/woowacourse-cover.png"
---


해당 내용은 2020년도 10월 우아한테크코스(이하, 우테코)를 어떤 생각을 하며 지원했는지에 대한 이야기를 다룹니다.

12월 19일 **2차 코딩테스트**를 치뤘습니다. 사실 2차 코딩테스트는 오프라인으로 진행되어야 했지만 코로나로 인해 결국 온라인으로 진행하게 되었습니다.

2차 코딩테스트는 3주차와 비슷한 유형이지만 **경로를 탐색**하는 기능을 구현하는 것이 핵심(?)이라고 할 수 있는 미션이었습니다.

개인적으로 이번 2차 코딩테스트는 난이도가 있다고 느껴진 부분이 __라이브러리를 활용하는 부분__ 에서 미숙함과 제한된 시간의 압박감 느꼈습니다.

## 어려웠던 점 회고
**첫 번째**는 객체지향스럽게 코드를 짜려다 보니까 어려움을 느꼈습니다. 처음 미션 요구사항을 설계할때 MVC 패턴의 형태로 구현하려고 하였습니다.

다시 말해 String 을 그대로 사용하지 않고 객체로 한번 포장해서 사용하려다 보니까 구현 난이도가 확 오르게 되었고 이러한 점 때문에 어려움이 많았습니다.

**두 번째**는 라이브러리 활용의 미숙함 입니다. 첫 번째 이유와 라이브러리 활용의 미숙함이 더해지면서 가장 중요한 기능을 구현하지 못하는 일이 발생했습니다.

### 라이브러리 활용의 미숙함..
`jgrapht` 라이브러리는 최단거리를 조회할 수 있는 라이브러리 입니다. 이를 활용해서 최단 거리(+시간) or 최단 시간(+거리)를 조회하는 기능을 구현해야 했습니다.

```java
WeightedMultigraph<String, DefaultWeightedEdge> graph = new WeightedMultigraph(DefaultWeightedEdge.class);
graph.addVertex("v1");
graph.addVertex("v2");
graph.addVertex("v3");
graph.setEdgeWeight(graph.addEdge("v1", "v2"), 2);
graph.setEdgeWeight(graph.addEdge("v2", "v3"), 2);
graph.setEdgeWeight(graph.addEdge("v1", "v3"), 100);

DijkstraShortestPath dijkstraShortestPath = new DijkstraShortestPath(graph);
List<String> shortestPath = dijkstraShortestPath.getPath("v3", "v1").getVertexList();
```

그러나 위 코드와 같이 `graph.setEdgeWeight(graph.addEdge("v1", "v2"), 2)` 와 같이 weight 값(2)을 하나만 등록하는 방식이었고 **최단 거리를 조회할때 걸린 시간을 구해야했기 때문에** 저는 `distance` 거리 값과 `time` 시간 값 두개의 정보를 아래 코드와 같이 Section 객체로 포장해서 구현하려고 시도했습니다. 그러나 결국 실패하였고 결국 String 그대로 조회하는 방식을 선택했습니다.

```java
public class Section {
    private Station startStation;
    private Station endStation;
    private int time;
    private int distance;

    public Section(Station startStation, Station endStation, int time, int distance) {
        this.startStation = startStation;
        this.endStation = endStation;
        this.time = time;
        this.distance = distance;
    }
    ...(중략)
    public static Section of(Station start, Station end, int time, int distance) {
        return new Section(start, end, time, distance);
    }
}
```

그리고 경로 기준을 선택할때 **최적의 거리의 경우 최적의 거리에 해당하는 시간을 계산이 필요** 했습니다. 이 또한 라이브러리 활용의 미숙함으로 시험 시간 안에 구현하지 못했고

시험 끝나고 복습할겸 아래와 같이 문자열을 파싱해서 구현했습니다.

```java
public static int getShortestPathTime(GraphPath graphPath) {
    int time = 0;
    SubwayPathFactory.init();
    SubwayPathFactory.creatTimeFirstGraph();
    for (Object s : graphPath.getEdgeList()) {
        String[] temp = s.toString().split(" : ");
        String first = temp[0].substring(1, temp[0].length());
        String second = temp[1].substring(0, temp[1].length()-1);
        time += (int) lineStation.getEdgeWeight(lineStation.getEdge(first, second));
    }
    return time;
}

public static int getShortestTimePath(GraphPath graphPath) {
    int path = 0;
    SubwayPathFactory.init();
    SubwayPathFactory.creatPathFirstGraph();
    for (Object s : graphPath.getEdgeList()) {
        String[] temp = s.toString().split(" : ");
        String first = temp[0].substring(1, temp[0].length());
        String second = temp[1].substring(0, temp[1].length()-1);
        path += (int) lineStation.getEdgeWeight(lineStation.getEdge(first, second));
    }
    return path;
}
```

## 느낀점...
이번 시험은 가장 중요한 기능을 구현하지 못했고 프리코스 때보다 더 나은 코드를 작성하지 못했습니다.

스스로에게 많은 실망과 아쉬움이 느껴졌네요..ㅎ

그래도 이번 시험 결과물에 실망한 만큼 더 큰 깨닳음을 얻은것 같아요!

그건 부족하기 때문에 더 성장할 수 있다는거죠!! 더 노력하겠습니다!!!