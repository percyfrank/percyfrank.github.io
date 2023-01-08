---
title: "[우아한테크코스 3기] 웹 백엔드 최종 오프라인 코딩 테스트 후기"
excerpt: "최선을 다했지만 아쉬움이 남는 시험이었다."
categories:
  - Woowacourse
tags:
  - 우아한테크코스
toc: true
toc_sticky: true
last_modified_at: 2020-12-20T08:30:00-05:00
---

## 1. 오프라인이라 쓰고, 온라인이라 읽는다

![image](../../assets/images/woowacourse.png)

우아한테크코스 3기 웹 백엔드 최종 코딩 테스트를 응시했습니다. 1기 및 2기의 최종 코딩 테스트는 오프라인으로 진행되었는데, 올해는 코로나 시국으로 인해 부득이하게 온라인으로 진행되었습니다. 오프라인 테스트였다면 잠실 교육장 건물도 구경하고 우테코 관계자님들과 Q&A 시간을 가질 수 있었을텐데 아쉽네요. 😅 부정행위 방지를 위해 ZOOM을 접속한 상태에서 시험을 보았는데, 신선한 경험이었습니다.

최종 코딩 테스트의 주제는 최단 거리 및 최소 시간 기준으로 지하철 노선의 경로를 조회하는 기능의 구현이었습니다. 경로 조회를 위해 힌트로 제공된 jgrapht 라이브러리 및 Dijkstra 알고리즘을 사용해야 했는데, 사용 방법에 대한 테스트가 주어졌음에도 어떻게 활용해야 할지 막막해서 여러모로 멘붕이었습니다.

겨우 기능을 구현해서 제출했는데, 정신이 없다보니 예외 처리 누락 및 불분명한 네이밍 등 많은 실수를 하고 말았네요. 최선을 다했지만 너무 아쉽습니다. 😢

구현 내용은 [최종 코딩 테스트 저장소](https://github.com/xlffm3/java-subway-path-precourse/tree/xlffm3)에 업로드했습니다.

<br>

## 2. 무엇이 부족했는가

### 2.1. 구현 기능 목록

구현 기능 목록을 조금 더 구체적으로 작성하지 못해서 아쉽습니다. 테스트는 프리코스 3주차 미션처럼 지하철 역과 노선 및 구간 등을 조회 및 삭제하는 기능을 요구하지 않습니다. 그러나 초기 데이터 설정을 위해 일부 기능들을 구현해야 했습니다.

StationService 및 LineService 등 지하철 역과 노선을 등록하는 기능들은 사용자가 UI에서 사용할 수 없더라도, 구현 기능 목록에 정확하게 명시하는게 더 바람직하지 않나 생각해봅니다.

그리고 지금 보니 오타가 장난아니게 많네요. 😫

### 2.2. Commit 단위

최대한 구현 기능 목록에 정리한 기능의 단위대로 커밋하려고 했지만, 일부 커밋들은 지나고 보니 단위가 너무 크다는 것을 느꼈습니다. 또한 커밋 메시지가 다소 난잡하고 오타가 범람하네요.

마감 시간은 다가오는데 코딩이 생각하는 대로 잘 풀리지 않아서, 깊게 고민하지 못하고 커밋을 날렸던 것 같습니다. 하나의 유의미한 변화인지 곰곰히 생각해보고 커밋하는 습관을 가져야겠습니다.

### 2.3. 예외 처리 누락

> Stations.java

```java
public void addStation(Station station, Section section) {
    if (stations.contains(station)) { //누락된 예외 처리
        throw new IllegalArgumentException();
    }
    int stationCounts = stations.size();
    Station lastStation = stations.get(stationCounts - INDEX_ADJUSTMENT);
    SUBWAY_MAP_GRAPH.addStationToGraph(lastStation, station, section);
    stations.add(station);
}
```

프로그램의 UI를 통해 사용자가 선택할 수 있는 경로 탐색 로직에서 사용하는 부분은 아닙니다. 다만, 초기 데이터 셋업을 위해 구현해놓은 기능인데 구현 기능 목록에 작성해놓고 누락했네요.

> LineService.java

```java
public static void addLine(String lineName, Stations stations) {
    LineRepository.findByName(lineName)
            .ifPresent(line -> {
                throw new IllegalArgumentException();
            });
    Line line = new Line(lineName, stations);
    LineRepository.addLine(line);
}
```

이 또한 초기 데이터 셋업을 위해 노선을 생성하는 로직입니다. 예외 처리는 되어 있으나, 예외 메시지를 리턴하는 것을 까먹었습니다. UI에서 노선을 추가할 일이 없기 때문에, 해당 부분에서의 에러 메시지를 볼 일은 없겠지만 조금 더 명시했으면 어땠을까 하는 아쉬움이 남네요.

### 2.4. 변수 및 메서드 네이밍 통일

처음 코드에서 경로라는 단어를 Route라고 사용하다가, 추후 Path로 변경하게 되었습니다. 제출하고 보니, 일부 메서드는 여전히 Route라는 네이밍을 사용하고 있었습니다. 또한 클래스 및 변수 명에서 관습적으로 사용하던 map이라는 단어를 전부 삭제했는데, 일부 클래스에서는 여전히 map이라는 단어가 포함된 변수명을 사용하고 있었습니다.

너무 조급했던 나머지 기본적인 부분을 체크하지 못했네요. 😫😫

### 2.5. 불분명한 네이밍

> SubwayGraph.java

```java
public void addStationToGraph(Station firstStation, Station lastStation, Section section) {
   int distance = section.getDistance();
   int time = section.getTime();
   addShortestDistanceGraph(firstStation, lastStation, distance);
   addMinimumDistanceGraph(firstStation, lastStation, time);
}
```

두 지하철 역과 그 사이의 거리 및 시간 정보들을 그래프에 저장하는데요. ``addStationToGraph()`` 라는 네이밍은 그래프에 역을 추가한다는 의미이기 때문에 다소 불분명해보입니다.

그리고 MinimumDistanceGraph가 아니라 MinimumTimeGraph여야 하는데, 이 부분에도 오타가 존재하네요. 왜 이렇게 오타를 많이 냈을까... 😑

> SubwayGraph.java

```java
private PathResponseDto calculateShortestPath(List<Station> stations) {
    int distanceTotal = getEdgeWeightTotal(stations, shortestDistanceGraph);
    int timeTotal = getEdgeWeightTotal(stations, minimumTimeGraph);
    List<String> stationNames = stations.stream()
            .map(Station::getName)
            .collect(Collectors.toList());
    return new PathResponseDto(distanceTotal, timeTotal, stationNames);
}

private int getEdgeWeightTotal(List<Station> stations, WeightedMultigraph<Station, DefaultWeightedEdge> weightedMultigraph) {
    int stationCounts = stations.size();
    int edgeWeightTotal = ZERO;
    for (int i = ZERO; i < stationCounts - ONE; i++) {
        Station firstStation = stations.get(i);
        Station nextStation = stations.get(i + ONE);
        DefaultWeightedEdge defaultWeightedEdge = weightedMultigraph.getEdge(firstStation, nextStation);
        edgeWeightTotal += (int) weightedMultigraph.getEdgeWeight(defaultWeightedEdge);
    }
    return edgeWeightTotal;
}
```

특정 기준으로 경로를 조회할 때, 소요되는 시간과 거리 및 역 명단을 반환합니다. ``calculateShortestPath()`` 라는 메서드 이름이 그 의도를 잘 드러내지 못하는 것 같습니다.

또한 소요 시간 및 거리 등을 계산할 때, 주어진 라이브러리의 간선 가중치를 사용합니다. 다른 사람들이 볼 때 ``getEdgeWeightTotal()`` 메서드 네이밍의 의미를 잘 이해하지 못할 것 같습니다. 조금 더 직관적인 네이밍 연습이 필요할 듯 합니다.

전반적으로 Station 객체 변수명을 firstStation 및 nextStation(혹은 lastStation)이라고 명명했는데, beforeStation과 nextStation 등으로 통일하는 것이 더 명확하다는 생각이 드네요.

### 2.6. 코드 중복

> SubwayGraph.java

```java

public void addStationToGraph(Station firstStation, Station lastStation, Section section) {
    int distance = section.getDistance();
    int time = section.getTime();
    addShortestDistanceGraph(firstStation, lastStation, distance);
    addMinimumDistanceGraph(firstStation, lastStation, time);
}

private void addShortestDistanceGraph(Station firstStation, Station lastStation, int distance) {
    shortestDistanceGraph.addVertex(firstStation);
    shortestDistanceGraph.addVertex(lastStation);
    shortestDistanceGraph.setEdgeWeight(shortestDistanceGraph.addEdge(firstStation, lastStation), distance);
}

private void addMinimumDistanceGraph(Station firstStation, Station lastStation, int time) {
    minimumTimeGraph.addVertex(firstStation);
    minimumTimeGraph.addVertex(lastStation);
    minimumTimeGraph.setEdgeWeight(minimumTimeGraph.addEdge(firstStation, lastStation), time);
}
```

똑같은 로직의 코드가 중복되고 있습니다. 해당 부분의 중복을 효과적으로 제거하지 못해서 아쉽네요. 코드를 구현할 당시, 간단하게 중복을 제거하는 방법을 인지하긴 했으나 메서드 파라미터가 4개로 늘어나기 때문에 다른 방법을 강구했었는데... 결국 시간 내에 해결 방안을 찾지 못하고 중복된 코드를 제출했습니다. 😓

SubwayGraph 클래스에 너무 많은 책임이 할당된 것 같습니다. 만약 최단 거리 및 최소 시간 말고도 다른 경로 조회  기준이 생긴다면, 그 만큼 SubwayGraph 클래스에 중복되는 로직의 코드가 반복될 것입니다. PathGraph라는 인터페이스를 추상화하고 DistancePathGraph 및 TimePathGraph 등의 하위 클래스의 구현을 통해 중복을 제거할 수 있을 것 같네요.

### 2.7. 컨벤션 준수

일반적으로 클래스에 선언된 상수와 인스턴스 변수 사이에 띄어쓰기를 합니다. 또한 public 메서드에서 사용되는 private 메서드를, 해당 public 메서드 밑에 순서대로 배치해야 합니다.

SubwayGraph 클래스에서 이런 부분을 잘 준수하지 못했습니다.

<br>

## 3. 마치며

프리코스와 달리 한정된 시간 속에서 기능 요구 사항들을 구현하다 보니, 프리코스에서 배웠던 내용들을 완벽하게 보여주지 못해 아쉽습니다. 정말 아슬아슬하게 기능을 구현했거든요. 😅 제 코드를 보니 자괴감이 많이 드네요.

이번 테스트를 통해 제가 어느 부분이 부족한지 많이 느꼈습니다. 이번 테스트도 indent(depth) 1 및 메서드 라인 10줄을 준수하면서 구현했는데, 그보다는 네이밍에 대해 더 고민하는 습관을 가져야겠습니다.

시험 보셨던 분들 모두 고생 많으셨습니다. 모두 좋은 결과 있으시길!

**(2020.12.30 추가)**

![image](https://user-images.githubusercontent.com/56240505/103333706-1f9dc700-4ab2-11eb-8f51-fa15e2d84777.png)

합격했습니다. 2020년은 개발 공부하면서 자존감도 많이 떨어지고 코로나 시국이 겹치는 등 여러모로 힘든 한 해였습니다. 아직도 합격이 믿기지가 않네요. 누구보다 간절했던 만큼, 2021년에도 치열하게 노력하겠습니다. 🎈🎈

<br>

---
