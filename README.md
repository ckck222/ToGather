# ToGather

### **일정 조율 시스템 설계**

---

### **핵심 개요**
- **목적**: 대학생 팀플, 동아리 모임, 미팅 등에서 일정 조율과 장소 선택을 간편하게 관리.
- **주요 기능**:
  - 가능한 날짜/시간 선택 및 투표.
  - 참여자의 출발지점을 기반으로 중간지점 추천.
  - 장소(카페, 합주실, 음식점 등) 표시.
  - QR 코드, 링크, 또는 코드를 활용한 이벤트 참여.
  - TIME ZONE이 다를 시 시차를 선택하여 조회

---

### **기능 상세**

#### **1. 첫 페이지 구성**
- **영역 구분**: 
  1. **참여하기**: 링크/코드/QR로 이벤트에 참여 후 일정 투표.
  2. **이벤트 생성하기**: 날짜/시간 범위를 설정하고 투표 화면으로 연결.
  
#### **2. 일정 투표 기능**
- **날짜 및 시간 선택**:
  - 최대 14일까지 선택 가능.
  - 시간/날짜 중복 선택 허용.
- **빠른 선택 옵션 제공**:
  - "이번 주말", "평일 저녁" 등 사용자 맞춤 패턴 추가.
- **참여 상황 표시**:
  - 가장 인기 있는 옵션 강조.
  - 사용자가 선택한 옵션 시각적으로 구분.

#### **3. 장소 추천 및 지도 표시**
- **중간 지점 계산**:
  - 참여자들의 출발지점 기반으로 중간 좌표 계산.
- **장소 표시**:
  - 중간지점 근처 카페, 합주실, 음식점 등을 지도에 핀으로 표시.
  - 카카오 지도 API를 활용.

#### **4. 이벤트 참여 및 공유**
- **이벤트 생성 완료 후**:
  - "이벤트 링크 복사" 버튼 제공.
  - 카카오톡, 이메일, 메신저로 공유 가능.
- **QR 코드 제공**:
  - 링크를 QR 코드로 생성해 간편 공유.
- **코드 유효성 검증**:
  - 이벤트 코드가 유효하지 않을 경우 즉시 알림.

#### **5. 커뮤니케이션 및 마감 관리**
- **참여자 소통 기능**:
  - 간단한 메시지 게시판 추가.
  - 예: "A 합주실 18-20시 예약했습니다."
- **응답 마감 시간 설정**:
  - 이벤트 생성자가 투표 마감 시간 설정 가능.
  - 마감 후 인기 있는 옵션 자동 선택 (예외 가능).

#### **6. 반응형 UI**
- **화면 최적화**:
  - PC 우선

---

### **기술 스택 및 도구**
1. **프론트엔드**:
   - UI 라이브러리: **FullCalendar.js** (날짜/시간 선택)
   - 지도 API: **카카오 지도 API** (중간지점 추천 및 장소 표시).
2. **백엔드**:
   - **Spring**
   - 데이터베이스: **PostgreSQL**.
3. **공유 기능**:
   - **QR 코드 라이브러리**: `qrcode.js` (링크를 QR 코드로 생성).

---

### **사용자 흐름 요약**
1. **첫 페이지**:
   - "참여하기" 또는 "이벤트 생성하기" 선택.
2. **참여하기**:
   - 링크/코드/QR 입력 → 유효성 확인 → 투표 화면으로 이동.
3. **이벤트 생성하기**:
   - 날짜/시간 범위 설정 → 생성 완료 → 링크/QR 제공 → 투표 화면 이동.
4. **투표 화면**:
   - 가능한 시간 선택 및 제출.
   - 참여 상황을 히트맵으로 시각화.
5. **결과 페이지**:
   - 최다 선택 시간/장소 표시.
   - 중간지점 근처 장소 추천 및 지도 핀 표시.

---

### **개선 포인트**
- **확장성**: 장소 추천 기능에 필터(예: "조용한 카페" 또는 "넓은 공간") 추가 가능.

---

## **1. 데이터베이스 설계**
### **a. 테이블 설계**

#### **기본 테이블 구조**
1. **Event Table**: 이벤트에 대한 기본 정보를 저장.
   - `event_id`: 고유 식별자 (Primary Key).
   - `event_name`: 이벤트 이름.
   - `created_by`: 이벤트 생성자.
   - `created_at`: 생성 날짜.

2. **Participants Table**: 이벤트에 참여하는 사용자 정보를 저장.
   - `participant_id`: 고유 식별자.
   - `event_id`: `Event` 테이블과 연결.
   - `user_id`: 사용자 고유 ID.
   - `user_password`: 사용자 고유 password.

---

### **b. 데이터 샘플**
#### **Event Table**
| event_id | event_name   | created_by | created_at          |
|----------|--------------|------------|---------------------|
| 1        | Team Meeting | user123    | 2024-11-19 10:00:00 |


#### **Participants Table**
| participant_id | event_id | user_id  | user_password |
|----------------|----------|----------|----------|
| 1              | 1        | user456  |4522      |

---
아주 좋습니다! 말씀하신 사항들을 기반으로 히트맵을 개선해보겠습니다. 아래는 개선된 설계와 구현 방법입니다.

---

## **개선사항 적용**

### **개선된 요구사항**

1. **시간 순서 변경**:
   - 시간은 위에서 아래로 진행 (10:00 → 17:00).

2. **한 사용자당 최대 1번 클릭 가능**:
   - 로그인 기능 추가 전에는 임시로 JavaScript에서 한 번만 클릭 가능하도록 설정.

3. **드래그 기능 추가**:
   - 사용자가 여러 셀을 한 번에 선택할 수 있도록 구현.

---

### **구현 코드**

#### **1. HTML 및 JavaScript**

```html
<div id="heatmap"></div>
<script src="https://cdn.jsdelivr.net/npm/apexcharts"></script>

<script>
    // 시간 범위 및 날짜 초기화
    const selectedDates = ["11/22", "11/23", "11/25"];
    const timeRange = ["10:00", "10:30", "11:00", "11:30", "12:00", "12:30", "13:00", "13:30", "14:00", "14:30", "15:00", "15:30", "16:00", "16:30", "17:00"].reverse(); // 시간 뒤집기

    // 사용자당 선택 횟수 제한
    const userSelections = new Set();

    // 히트맵 데이터 초기화
    const heatmapData = timeRange.map(time => ({
        name: time,
        data: selectedDates.map(date => ({ x: date, y: 0 }))
    }));

    // ApexCharts 설정
    const options = {
        chart: {
            type: "heatmap",
            height: 450,
            events: {
                // 셀 클릭 이벤트
                click: function(event, chartContext, config) {
                    const { seriesIndex, dataPointIndex } = config;

                    if (seriesIndex !== undefined && dataPointIndex !== undefined) {
                        const time = heatmapData[seriesIndex].name;
                        const date = heatmapData[seriesIndex].data[dataPointIndex].x;

                        // 이미 선택된 셀인지 확인
                        const cellId = `${date}-${time}`;
                        if (userSelections.has(cellId)) {
                            alert("이 시간과 날짜는 이미 선택했습니다!");
                            return;
                        }

                        // 선택 횟수 증가 및 사용자 선택 추가
                        heatmapData[seriesIndex].data[dataPointIndex].y += 1;
                        userSelections.add(cellId);

                        // 히트맵 업데이트
                        chart.updateSeries(heatmapData);
                        console.log(`선택된 시간: ${time}, 날짜: ${date}`);
                    }
                }
            }
        },
        plotOptions: {
            heatmap: {
                shadeIntensity: 0.5,
                colorScale: {
                    ranges: [
                        { from: 0, to: 1, name: "Low", color: "#D6EAF8" },
                        { from: 2, to: 3, name: "Medium", color: "#5DADE2" },
                        { from: 4, to: 5, name: "High", color: "#154360" }
                    ]
                }
            }
        },
        dataLabels: {
            enabled: true,
            style: {
                colors: ["#fff"]
            }
        },
        series: heatmapData,
        xaxis: {
            type: "category",
            title: {
                text: "날짜"
            }
        },
        yaxis: {
            title: {
                text: "시간"
            }
        },
        title: {
            text: "시간-날짜 히트맵",
            align: "center"
        }
    };

    // 히트맵 렌더링
    const chart = new ApexCharts(document.querySelector("#heatmap"), options);
    chart.render();

    // 드래그 선택 기능 구현
    let isDragging = false;
    let startCell = null;

    document.querySelector("#heatmap").addEventListener("mousedown", (event) => {
        isDragging = true;
        startCell = getCellUnderCursor(event); // 드래그 시작 셀 저장
    });

    document.querySelector("#heatmap").addEventListener("mousemove", (event) => {
        if (!isDragging) return;
        const currentCell = getCellUnderCursor(event); // 드래그 중인 현재 셀
        if (startCell && currentCell && startCell !== currentCell) {
            const [date, time] = currentCell.split("-");
            // 히트맵 데이터 업데이트
            updateCell(date, time);
        }
    });

    document.addEventListener("mouseup", () => {
        isDragging = false;
        startCell = null;
    });

    function getCellUnderCursor(event) {
        const target = event.target.closest(".apexcharts-heatmap-rect");
        return target ? target.getAttribute("data-cell-id") : null;
    }

    function updateCell(date, time) {
        const cellId = `${date}-${time}`;
        if (!userSelections.has(cellId)) {
            // 셀 업데이트 및 선택 추가
            const seriesIndex = timeRange.indexOf(time);
            const dataPointIndex = selectedDates.indexOf(date);
            heatmapData[seriesIndex].data[dataPointIndex].y += 1;
            userSelections.add(cellId);
            chart.updateSeries(heatmapData);
        }
    }
</script>
```

---

### **개선된 기능**

1. **시간 순서 변경**:
   - `timeRange.reverse()`를 사용해 시간 순서를 위에서 아래로 변경.

2. **사용자당 최대 1번 클릭**:
   - `Set` 객체를 사용해 사용자가 이미 선택한 날짜-시간 셀을 추적.
   - 동일한 셀을 클릭하면 알림(`alert`)을 띄워 중복 선택 방지.

3. **드래그 기능**:
   - 마우스 드래그를 통해 여러 셀을 한 번에 선택 가능.
   - `mousedown`, `mousemove`, `mouseup` 이벤트로 구현.

---

### **사용 예시**

1. **히트맵 초기 상태**:
   - 모든 셀이 초기값 `0`으로 표시.

2. **셀 클릭**:
   - 특정 셀(날짜-시간 조합)을 클릭하면 해당 셀의 값이 `1` 증가.
   - 동일 셀 재선택 불가.

3. **드래그 선택**:
   - 마우스를 드래그하여 여러 셀을 한 번에 선택.
   - 드래그한 모든 셀의 값이 증가.

---

### **결과 화면**

- **X축**: 날짜(`11/22`, `11/23`, `11/25`).
- **Y축**: 시간(`10:00`, `10:30`, ..., `17:00`).
- **셀 색상**: 선택 횟수에 따라 점점 짙어짐.

---

### **추가 구현 가능 사항**

1. **서버 저장**:
   - 사용자가 선택한 데이터를 AJAX로 서버에 저장.

2. **실시간 업데이트**:
   - 다른 사용자의 선택을 실시간으로 반영.

3. **선택 제거 기능**:
   - 드래그한 셀을 다시 클릭하면 선택 해제.

추가적으로 궁금한 점이나 필요한 사항이 있으면 말씀해주세요! 😊

