---
title: "[우아한테크코스 3기] 프리코스 3주차 미션 학습 내용 및 회고"
excerpt: "지하철 노선도, 어떻게 중복을 줄일 수 있을까?"
categories:
  - Woowacourse
tags:
  - 우아한테크코스
toc: true
toc_sticky: true
last_modified_at: 2020-12-15T08:30:00-05:00
---

## 1. 지하철 노선도

우아한테크코스 3기의 프리코스 3주차 미션은 지하철 노선도 기능 구현이다. 구현 내용은 [프리코스 3주차 미션 저장소](https://github.com/xlffm3/java-subway-map-precourse/tree/xlffm3)에 업로드했다.

해당 미션은 indent(depth)를 2, 메서드 라인을 15줄까지 허용한다. 조금 더 극단적인 차원의 연습을 위해, indent 1 및 메서드 라인 10줄 컨벤션을 지켜가며 코드를 짰다. 또한 가급적 static 클래스의 사용을 자제했다. 아름다운 설계가 어려워서 코드를 셀 수 없이 갈아엎었는데, 그만큼 많은 공부를 할 수 있었다.

3주차 미션을 진행하면서 고민한 내용들에 대해 개인적으로 공부하고 정리해보았다.

<br>

## 2. Commit의 단위

TDD 사이클을 적용하며 1주차 및 2주차 과제를 진행했는데, 커밋이 대략 100개 정도 되었다.

* 테스트 추가 -> test 커밋
* 기능 구현 -> feat 커밋
* 리팩토링 -> refactor 커밋

구현하고자 하는 기능의 테스트 추가(test), 기능 구현(feat), 리팩토링(refactor)을 모두 커밋에 일일이 반영하다 보니, 과제의 사이즈에 비해 커밋 이력이 너무 많아졌다.

작게 커밋하는 습관이 나쁜 것은 아니지만, 원칙적으로 하나의 커밋은 의미있는 하나의 변경 사항을 의미한다. 나는 너무 잘게 커밋을 나누다 보니, 별 의미없는 자잘한 커밋 때문에 히스토리가 지저분해지고 가독성이 떨어졌다.

따라서 이번 과제를 진행할 때는,

* 테스트실패 -> 기능구현 -> 테스트성공 -> 커밋
* 리팩토링 -> 테스트성공 -> 커밋

사이클을 따라 커밋을 남기려고 노력했다. 그 결과, 커밋 히스토리가 (만족스러운 수준은 아니지만) 1주차 및 2주차에 비해 가독성이 개선된 것 같다.

개인적으로 혼란스러웠던 내용은, README에 정리한 기능 **단위**로 커밋하라는 요구사항이었다. 단위란 무엇인가? A라는 기능 단위를 구현한다고 하면, A 기능에 필요한 B, C 등의 하위 기능을 전부 구현해야 1개의 단위인가? 아니면 A, B, C는 모두 각각의 1개 단위인가?

사람들마다 **단위**라는 개념의 인식이 다른 것 같다. 그러다 보니 같은 기능 단위의 커밋 요구 사항에, 누구는 큼직큼직하게 누구는 세세하게 커밋을 나누는 것 같다. 커밋을 잘게 나누는 것이 나쁠 것은 없지만, 과유불급인 것 같다. 항상 커밋을 날리기 전에, 의미있는 하나의 변경 사항인지 생각해보는 습관을 가져야겠다.

<br>

## 3. MVC 패턴 + 레이어드 아키텍쳐

MVC 패턴을 적용해 View와 Domain이 서로 의존성을 가지지 않고, 독립적인 개발이 가능하도록 했다. UI를 출력하는 로직은 철저하게 View의 책임으로 두었다. ``toString()`` 메서드는 View에서 사용할 메시지를 Domain이 편리하게 제공할 수 있지만, View의 요구사항이 변할 때 마다 Domain이 변하는 의존성을 가지기 때문에 지양했다.

MVC 패턴으로 열심히 개발하던 중, 프로그래밍 요구사항으로 주어진 XXXRepository가 마음에 걸렸다.

> StationRepository.java

```java
public class StationRepository {
    private static final List<Station> stations = new ArrayList<>();

    public static List<Station> stations() {
        return Collections.unmodifiableList(stations);
    }

    public static void addStation(Station station) {
        stations.add(station);
    }

    public static boolean deleteStation(String name) {
        return stations.removeIf(station -> Objects.equals(station.getName(), name));
    }
}
```

처음에는 단순히 Station 및 Line 리스트의 일급 컬렉션 객체라고 생각하고 개발을 진행했다.

> StationRepository.java

```java
public void addStation(Station station) {
    String name = station.getName();
    if (this.containsStationName(name)) {
        throw new StationDuplicationException();
    }
    stations.add(station);
}

public boolean deleteStationByName(String name) {
    if (!this.containsStationName(name)) {
        throw new CannotFindStationException();
    }
    return stations.removeIf(station -> station.matchesName(name));
}
```

> LineRepository.java

```java
public void addLine(Line line) {
    if (this.contains(line)) {
        throw new LineDuplicationException();
    }
    lines.add(line);
}
```

StationRepository 및 LineRepository가 특정 작업을 수행할 때 내부 컬렉션을 통해 유효성을 체크한다.

> SubwayMapController.java

```java

public class SubwayMapController {

    private final StationRepository stationRepository;
    private final LineRepository lineRepository;

    public SubwayMapController(StationRepository stationRepository, LineRepository lineRepository) {
        this.stationRepository = stationRepository;
        this.lineRepository = lineRepository;
    }

    public void deleteStation(String stationName) {
        Station station = new Station(stationName);
        if (lineRepository.contains(station)) { //No!
            throw new CannotDeleteStationException();
        }
        stationRepository.deleteStation(station);
    }
}
```

문제는 Station 객체를 삭제하는 등 일부 기능의 경우, 유효성 검증 역할을 Repository 객체들이 전담하지 못한다. SubwayMapController가 외부에서 유효성 체크를 진행하는 구조가 반복되었다. 컨트롤러는 컨트롤러답게 Model과 View 사이에서 데이터를 전달해주는 역할만을 전담하게 만들고 싶었다. Domain 관련 내부 로직들이 Controller에게 노출되지 않는 방법이 있을까?

Controller와 Repository 사이에 Layer가 있으면 될 것 같다는 생각이 들었고, 가장 먼저 떠오른 개념은 레이어드 아키텍쳐였다.

> StationRepository.java

```java
public void save(Station station) {
    stations.add(station);
}

public Optional<Station> findByName(String name) {
    return stations.stream()
            .filter(station -> station.matchesName(name))
            .findFirst();
}

public List<Station> findAll() {
    return Collections.unmodifiableList(stations);
}

public boolean delete(Station targetStation) {
    return stations.removeIf(station -> station.equals(targetStation));
}
```

먼저 Repository 관련 클래스들에게서 유효성 검증 등 비즈니스 로직을 전부 삭제했다. 이를 통해, Repository 클래스들은 순수하게 CRUD만을 담당하는 DAO가 되었다.

> StationService.java

```java
public class StationService {

    private final StationRepository stationRepository;

    public StationService(StationRepository stationRepository) {
        this.stationRepository = stationRepository;
    }

    public void addStationByName(String name) {
        stationRepository.findByName(name)
                .ifPresent(station -> {
                    throw new StationDuplicationException();
                });
        Station station = new Station(name);
        stationRepository.save(station);
    }

    public boolean deleteStationByName(String name) {
        Station station = findStationByName(name);
        if (station.isRegisteredAsLineSection()) {
            throw new CannotDeleteStationException();
        }
        return stationRepository.delete(station);
    }

    public Station findStationByName(String name) {
        return stationRepository.findByName(name)
                .orElseThrow(CannotFindStationException::new);
    }
}
```

Repository를 가지는 Service 레이어를 생성한다. 기존의 Repository에서 담당하던 유효성 검증 및 예외 발생 등의 로직을 Service에서 처리한다. 최대한 Service는 도메인 간 순서 보장의 역할만 하도록 했다.

> SubwayMapController.java

```java
public class SubwayMapController {

    private final StationService stationService;
    private final LineService lineService;

    public SubwayMapController(StationService stationService, LineService lineService) {
        this.stationService = stationService;
        this.lineService = lineService;
    }

    public void addStation() {
        String stationName = inputView.inputName();
        stationService.addStationByName(stationName);
    }

    public void addLine() {
        LineDto lineDto = inputView.inputLineRequest();
        Sections sections = stationService.createSections(lineDto);
        lineService.addLine(lineDto);
    }

    public void addSection() {
      //중략
    }

    //...
    //중략 (역 삭제, 노선 삭제, 구간 삭제, 조회 등 메서드)
}
```

유효성 검증 등 내부 도메인 비즈니스 로직이 더이상 SubwayMapController에게 노출되지 않는다. 이제 해당 컨트롤러는 View와 Model 사이의 데이터 연결만을 담당하게 될 뿐이다.

<br>

## 4. Controller의 단일 책임 원칙

[3절](https://xlffm3.github.io/woowacourse/Woowacourse_precourse_subway/#3-mvc-%ED%8C%A8%ED%84%B4--%EB%A0%88%EC%9D%B4%EC%96%B4%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90)의 Service 레이어 추가를 통해, SubwayMapController가 많이 가벼워졌지만 약간의 문제가 있었다.

과연 하나의 컨트롤러가 역 관리, 노선 관리, 구간 관리를 모두 전담하는게 맞을까? 클래스는 단 한개의 책임을 가져야한다. 이 말은 곧 클래스를 변경하는 이유는 단 한 개여야 한다는 말과 일맥상통한다.

SubwayMapController는 3가지 책임을 가지고 있어서 단일 책임 원칙을 위배한다는 생각이 들었다. 먼저 CRUD 컨트롤러를 추상화했다.

> SubwayMapController.java

```java
public interface SubwayMapController {

    public void register();
    public void delete();
    public void readNames();
    public void readEntireSubwayMap();
}
```

역을 관리할 컨트롤러, 노선을 관리할 컨트롤러, 구간을 관리할 컨트롤러들이 공통적으로 가지는 기능들을 먼저 선언한다.

> StationController.java

```java
public class StationController implements SubwayMapController {

    private final StationService stationService;

    public StationController(StationService stationService) {
        this.stationService = stationService;
    }

    @Override
    public void register() {
        String stationName = inputView.inputName(ManagementType.STATION, FunctionType.REGISTER);
        stationService.addStationByName(stationName);
    }

    @Override
    public void delete() {
      //역 삭제 로직
    }

    @Override
    public void readNames() {
      //역 조회 로직
    }

    @Override
    public void readEntireSubwayMap() {
      //미구현
    }
}
```

> LineController.java

```java
public class LineController implements SubwayMapController {

    private final StationService stationService;
    private final LineService lineService;

    public LineController(StationService stationService, LineService lineService) {
        this.stationService = stationService;
        this.lineService = lineService;
    }

    @Override
    public void register() {
        LineDto lineDto = inputView.inputLineRequest(ManagementType.LINE, FunctionType.REGISTER);
        String lineName = lineDto.getLineName();
        Sections sections = stationService.createSections(lineDto);
        lineService.addLine(lineName, sections);
    }

    @Override
    public void delete() {
      //노선 삭제 로직
    }

    @Override
    public void readNames() {
      //노선 조회 로직
    }

    @Override
    public void readEntireSubwayMap() {
      //지하철 노선도 조회 로직
    }
}
```

StationController, LineController, SectionController를 생성한 다음, 각자의 책임에 맞춰 SubwayMapController의 CRUD 메서드를 구현한다. 이로써 컨트롤러 클래스들은 단 하나의 책임만을 가지게 되었다.

<br>

## 5. FrontController 패턴 및 클래스 분리를 통한 리팩토링

컨트롤러가 1개에서 3개로 변함에 따라, 다음과 같은 문제가 생겼다.

> SubwayMapApplication.java

```java
public class SubwayMapApplication {

    private final InputView inputView;
    private final SubwayMapController subwayMapController;

    //중략

    public void execute(ManagementType managementType, FunctionType functionType) {
        if (managementType == ManagementType.STATION) {
            executeStationManagement(functionType);
        }
        if (managementType == ManagementType.LINE) {
            executeLineManagement(functionType);
        }
        //중략
    }

    public void executeStationManagement(FunctionType functionType) {
        if (functionType == FunctionType.REIGSTER) {
            subwayMapController.addStation();
        }
        if (functionType == FunctionType.DELETE) {
            subwayMapController.deleteStation();
        }
        //중략
    }

    public void executeLineManagement(FunctionType functionType) {
        if (functionType == FunctionType.REIGSTER) {
            subwayMapController.addLine();
        }
        if (functionType == FunctionType.DELETE) {
            subwayMapController.deleteLine();
        }
        //중략
    }
    //... 중략 (executeSectionManagement 등)
}
```

Application 클래스는 유저가 입력한 원하는 관리 및 기능에 따라 조건적으로 인스턴스 변수 SubwayMapController에게 작업을 요청한다. 그러나 Controller를 분리함에 따라, Application 클래스는 이제 4개의 인스턴스 변수를 가져야만 한다.

인스턴스 변수가 너무 많으면 객체가 복잡해진다. 또한 현재 컨트롤러에게 작업을 요청하는 코드는 비슷한 구조로 반복되고 있다. 이를 별도의 클래스를 통해 리팩토링하기로 했다.

가장 먼저 떠오른 개념은 DispatcherServlet으로 대표되는 FrontController였다. 모든 작업 요청을 한 곳으로 받아 집중시킨 뒤, 그 곳에서 요청을 처리할 수 있는 적합한 세부 컨트롤러를 찾고 작업을 위임하면 되겠다는 생각이 들었다.

> ControllerMapper.java

```java
public class ControllerMapper {

    private final List<SubwayMapController> subwayMapControllers;

    public ControllerMapper(List<SubwayMapController> subwayMapControllers) {
        this.subwayMapControllers = subwayMapControllers;
    }

    public void delegateRequestToController(ManagementType managementType, FunctionType functionType) {
        SubwayMapController subwayMapController = findSubwayMapController(managementType);
        executeRequestFunction(subwayMapController, functionType);
    }

    private SubwayMapController findSubwayMapController(ManagementType managementType) {
        if (managementType == ManagementType.STATION) {
            return getSubwayMapController(StationController.class);
        }
        if (managementType == ManagementType.SECTION) {
            return getSubwayMapController(SectionController.class);
        }
        return getSubwayMapController(LineController.class);
    }

    private SubwayMapController getSubwayMapController(Class<? extends SubwayMapController> targetClass) {
        return subwayMapControllers.stream()
                .filter(subwayMapController -> subwayMapController.getClass() == targetClass)
                .findFirst()
                .orElseThrow(CannotFindControllerException::new);
    }

    private void executeRequestFunction(SubwayMapController subwayMapController, FunctionType functionType) {
        if (functionType == FunctionType.REGISTER) {
            subwayMapController.register();
        }
        if (functionType == FunctionType.DELETE) {
            subwayMapController.delete();
        }
        if (functionType == FunctionType.READ) {
            subwayMapController.readNames();
        }
    }
}
```

SubwayMapController 인터페이스 참조 변수의 리스트에 미리 3개의 컨트롤러를 등록해둔다. 이후 요청이 들어오면, 스트림을 순회하며 관리 유형(ManagementType)을 처리할 수 있는 적합한 컨트롤러 객체를 찾는다.

컨트롤러를 찾았으면 기능 유형(FunctionType)에 맞게 ``register()``, ``delete()`` 등의 작업을 수행해주면 된다.

이렇게 ControllerMapper 클래스를 분리함으로써,

> SubwayMapApplication.java

```java
public class SubwayMapApplication {

    private final InputView inputView;
    private final ControllerMapper controllerMapper;

    //중략

    public void execute() {
        ManagementType managementType = inputView.inputManagementType();
        FunctionType functionType = inputView.inputFunctionType();
        controllerMapper.delegateRequestToController(managementType functionType);
    }
}
```

SubwayMapApplication 클래스는 반복되는 코드 및 많은 인스턴스 변수들을 모두 제거할 수 있었다. 입력받은 관리 유형 및 기능 유형을 ControllerMapper에게 건내주고, 작업을 위임하면 모든 요청을 처리할 수 있게 된다.

요청을 수행할 수 있는 컨트롤러를 찾는다는 점에서 Spring의 HandlerMapping과 유사한데, 비슷한 것을 개발하다 보니 프레임워크가 주는 장점을 다시금 느낄 수 있었다.

<br>

## 6. Exception의 패키지 위치

사용자 정의 예외는 어떻게 관리해야 할까? Station 관련 도메인 클래스들은 subway.domain.station에 위치하는데, 해당 클래스와 관련된 커스텀 예외의 관리 방법은 다음 세 가지로 나눌 수 있다.

* subway.domain.station에 도메인 클래스들과 함께 위치시키기.
* subway.domain.station.exception처럼 하위 패키지에 위치시키기.
* subway.exceptions처럼 다른 도메인 클래스 관련 커스텀 예외들과 함께 한 곳에 몰아넣기.

[Should Exceptions be placed in a separate package?](https://stackoverflow.com/questions/825281/should-exceptions-be-placed-in-a-separate-package) 라는 글에서, 커스텀 예외는 1번 처럼 도메인 클래스들과 함께 위치시키기를 권고하고 있다.

> Package should be able to present a single unit of functionality. Refer this for an example. A Custom Exception, which is gonna thrown out of it, is a part of that unit of functionality, and should be in the same package.

패키지는 한 개의 기능 단위를 의미하고, 커스텀 예외 또한 한 개의 기능 단위의 일부분이다. 따라서 같은 패키지에 속해 있어야 한다.

Enum, Interface, Custom Exception을 별도의 패키지로 그룹할 필요가 없다. 오히려 불필요한 내부 패키지 의존성을 유발한다고 한다.

<br>

## 7. Bad Smell : 중복된 코드

한 곳 이상에서 중복된 코드 구조가 나타난다면, 그것을 합쳐서 프로그램을 개선할 수 있다.

이번 과제는 대략 4번 정도 갈아아엎었다. 나름 설계를 하고 들어갔지만, 도메인 지식이 부족하다 보니 추상화가 잘 이루어지지 않아서 비효율적으로 반복되는 코드들이 많았다.

> MainManagementState.java

```java
public enum MainManagementState {
    STATION_MANAGEMENT("1"),
    LINE_MANAGEMENT("2"),
    INTERVAL_MANAGEMENT("3"),
    MAP_PRINT_MANAGEMENT("4"),
    EXIT("Q");
}
```

> StationManagementState.java

```java
public enum StationManagementState {
    REGISTER("1"),
    DELETE("2"),
    READ("3"),
    BACK("B");
}
```

> LineManagementState.java

```java
public enum LineManagementState {
    REGISTER("1"),
    DELETE("2"),
    READ("3"),
    BACK("B");
}
```
> SectionManagementState.java

```java
public enum SectionManagementState {
    REGISTER("1"),
    DELETE("2"),
    BACK("B");
}
```

프로그램은 메인 화면에서 역 관리, 노선 관리, 구간 관리 등을 선택한 다음 세부 기능(등록, 삭제, 조회 등)을 수행하는 로직을 가지고 있다.

Enum을 통해 상태값을 관리하면 되겠다는 생각이 들었는데, 문제는 메인 화면에 기능이 추가될 수록 XXXManagementState라는 Enum이 추가되어야 했다.

그러다 보니 값을 입력하는 InputView와, View에서 받은 입력값을 Model에 전달해주는 Controller의 코드는 중복되는 구조들이 반복되었다.

> InputView.java

```java
public MainManagementState inputMainManagementFunctionNumber() {
    System.out.println(INPUT_FUNCTION_NUMBER_NOTICE);
    String functionNumber = scanner.nextLine();
    try {
        return MainManagementState.findState(functionNumber);
    } catch (RuntimeException runtimeException) {
        OutputView.printExceptionMessage(runtimeException);
        return inputMainManagementFunctionNumber();
    }
}

public StationManagementState inputStationManagementFunctionNumber() {
    //동일 로직
}

public LineManagementState inputLineManagementFunctionNumber() {
    //동일 로직
}

public SectionManagementState inputSectionManagementFunctionNumber() {
    //동일 로직
}
```

> Controller.java

```java
private static void selectManagementFunction(MainManagementState mainManagementState) {
   if (mainManagementState == MainManagementState.STATION_MANAGEMENT) {
       activateStationManagement();
       return;
   }
   if (mainManagementState == MainManagementState.LINE_MANAGEMENT) {
       activateLineManagement();
       return;
   }
   //...이하 반복
}

private static void activateStationManagement() {
    StationManagementState stationManagementState = inputView.inputStationManagementFunctionNumber();
    if (stationManagementState == StationManagementState.REGISTER) {
       //로직
    }
    if (stationManagementState == StationManagementState.DELETE) {
      //로직
    }
    if (stationManagementState == StationManagementState.READ) {
      //로직
    }
}
```

어떤 관리 항목은 등록, 조회, 삭제가 모두 가능하지만 어떤 관리 항목은 삭제가 불가능하다. 어떻게 하면 중복을 없앨 수 있을까? 의외로 Enum의 조합을 통해 쉽게 해결할 수 있었다.

> ManagementType.java

```java
public enum ManagementType {
    STATION("1",
            Arrays.asList(FunctionType.REGISTER, FunctionType.DELETE, FunctionType.READ, FunctionType.BACK)),
    LINE("2",
            Arrays.asList(FunctionType.REGISTER, FunctionType.DELETE, FunctionType.READ, FunctionType.BACK)),
    SECTION("3",
            Arrays.asList(FunctionType.REGISTER, FunctionType.DELETE, FunctionType.BACK)),
    PRINT_SUBWAY_MAP("4", Collections.emptyList()),
    EXIT("Q", Collections.emptyList());
}
```

각각의 관리(ManagementType)들이 기능(FunctionType)을 가지도록 코드를 개선했다. XXXManagementState가 FunctionType 하나로 통합됨에 따라, InputView 및 Controller에서 반복되는 구조를 효과적으로 제거할 수 있다.

> InputView.java

```java
public String inputName(ManagementType managementType, FunctionType functionType) {
    if (managementType == ManagementType.STATION) {
        return scanInputWithNoticeMessage(STATION_MESSAGE), functionType);
    }
    if (managementType == ManagementType.LINE) {
        return scanInputWithNoticeMessage(LINE_MESSAGE,, functionType);
    }
    // 비슷한 로직 반복
  }
```

기존에는 View 또한 ManagementType 및 FunctionType에 따라 if 분기를 통해 메시지를 달리 출력했다. Type Enum에 ``toString()``을 구현해서 중복되는 부분을 제거했다.

> InputView.java

```java
public String inputName(ManagementType managementType, FunctionType functionType) {
    String message = String.format(INPUT_NAME_MESSAGE_FORMAT, functionType.toString(), managementType.toString());
    return scanInputWithNoticeMessage(message);
}
```

<br>

---

## References

* [commit하는 단위는 어떻게 하는 것이 좋을까요?](https://github.com/javajigi/minesweeper-ruby/issues/5)
* [Should Exceptions be placed in a separate package?](https://stackoverflow.com/questions/825281/should-exceptions-be-placed-in-a-separate-package)
