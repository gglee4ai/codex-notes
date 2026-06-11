# R/Rmd Coding Policy

이 문서는 Codex가 R code, R Markdown, R Notebook을 수정하거나 새로 작성할 때 따를 작업 기준이다. 목적은 코드가 과하게 추상화되지 않고, 노트북 흐름이 읽히며, 나중에 사용자가 직접 이어서 고치기 쉬운 상태를 유지하는 것이다.

## 기본 원칙

- R 분석 코드는 `tidyverse`를 기본 도구로 사용할 수 있다.
- 다만 `tidyverse` 안에서도 experimental 기능, deprecated 기능, lifecycle이 불안정한 문법은 가급적 피한다.
- 이미 repo 안에서 안정적으로 쓰이는 패턴이 있으면 새 스타일을 만들지 말고 기존 패턴을 따른다.
- 복잡한 함수화보다 노트북의 분석 흐름이 보이는 청크 단위 구성을 우선한다.
- 함수는 같은 작업이 여러 번 반복되거나, 계산 의미를 이름으로 분명히 드러내는 경우에만 만든다.
- 새 패키지 의존성은 명확한 이득이 있을 때만 추가하고, 로컬 설치가 필요하면 먼저 사용자에게 설명하고 확인한다.

## R Markdown / Notebook 구성

- Rmd는 날짜별 작업 기록과 분석 흐름이 보이는 노트북으로 작성한다.
- 큰 작업은 setup, 데이터 로드, 전처리, 확인, 모델링, 그림 저장, 요약처럼 의미 있는 청크로 나눈다.
- `html_notebook`에서 불필요한 전역 `knitr::opts_chunk$set()` 설정은 넣지 않는다.
- `html_notebook`에서 표 출력을 위해 `knitr::kable()`을 붙이지 않는다. tibble/data.frame 자체를 출력한다.
- Rmd에서 필요 없는 코드펜스, 예를 들어 단순 설명용 triple-backtick `text` 블록은 쓰지 않는다.
- 사람이 읽는 설명, 해석, 메모는 가능하면 한글로 쓴다.
- 함수명, 객체명, 파일명, 모델명, 패키지명처럼 영어가 자연스러운 부분은 영어를 유지한다.

## 코드 스타일

- pipe는 base pipe `|>`를 우선 사용한다.
- 긴 pipe chain이나 여러 줄 할당문은 왼쪽 변수명과 `<-`만 먼저 쓰고 줄을 바꾼 뒤 오른쪽 표현식을 시작한다.

```r
curve_property_check <-
  curves_joined |>
  arrange(specimen_number, strain_in_in) |>
  summarize(...)
```

- 한 줄에 너무 많은 작업을 넣지 말고, 중간 객체를 적절히 만들어 검토 가능하게 둔다.
- 중간 객체 이름은 `*_raw`, `*_joined`, `*_plot`, `*_summary`, `*_check`처럼 역할이 드러나게 붙인다.
- 단위 변환 함수처럼 의미가 분명하고 자주 쓰는 함수는 짧고 읽기 쉬운 이름을 쓴다. 예: `ksi_to_MPa()`, `F_to_C()`.
- 문자열 파싱, factor level 조정, join key 정리는 별도 청크나 별도 mutate 블록으로 분리해 실수를 확인하기 쉽게 한다.
- `rowwise()`는 필요한 경우에만 제한적으로 사용한다. 벡터화, `pmap()`, list-column이 더 명확하면 그쪽을 우선 검토한다.

## tidyverse 사용 기준

- `dplyr`, `ggplot2`, `readr`, `stringr`, `forcats`, `tidyr`, `purrr` 등 널리 쓰이고 안정적인 tidyverse 기능을 우선한다.
- deprecated 함수는 새 코드에 쓰지 않는다. 예: `mutate_at()`, `summarise_at()`, `gather()`, `spread()`, `aes_string()`.
- lifecycle이 experimental로 표시된 기능은 꼭 필요한 경우가 아니면 피한다.
- `across()`는 안정적인 column-wise 작업에 사용한다.
- `.by`는 간단한 그룹 요약에서 사용할 수 있지만, 긴 분석 흐름이나 여러 단계 그룹 조작에서는 `group_by()` / `ungroup()`이 더 읽기 쉬운지 검토한다.
- `reframe()`, `pick()`처럼 비교적 새 문법은 기존 코드와 맞고 의도가 명확할 때만 사용한다.
- factor 순서가 plot facet이나 legend와 연결되는 경우 `fct_relevel()`, `fct_recode()` 등을 명시적으로 사용한다.

## Plotting Policy

- 최종 결과 그림과 중간 확인 그림을 구분한다.
- 중간 확인용 그림은 과도한 색상, theme, annotation을 넣지 않고 확인 목적에 집중한다.
- 색상은 그룹 구분이 실제 해석에 필요할 때만 사용한다.
- 여러 시편이나 조건을 비교할 때 facet 구조를 먼저 검토한다.
- `facet_wrap()`은 순서가 중요한 작은 multiples에 사용하고, 행/열 의미가 분명한 경우에만 `facet_grid()`를 쓴다.
- 같은 시편의 저온/고온 조건처럼 행/열 의미가 중요한 경우 factor level과 facet 배치가 데이터 의미와 맞는지 확인한다.
- point, line, facet에 사용하는 데이터가 다르면 두 데이터 모두 같은 facet variable과 factor level을 갖도록 맞춘다.
- legend는 꼭 필요한 mapping만 남긴다. 점 모양, 선 종류, 색의 의미가 겹치면 하나를 줄이거나 설명을 caption/note로 옮긴다.
- caption은 그림이 실제로 보여주는 내용과 맞아야 한다. 이전 그림에서 복사한 caption을 그대로 두지 않는다.

## 데이터 처리와 재현성

- 입력 파일 경로, 결과 폴더, 저장 파일 이름은 노트북 상단에서 확인 가능하게 둔다.
- repo 기준 경로가 필요하면 단순하게 `repo_root <- ".."`처럼 현재 노트북 위치에 맞춘다.
- 임시 다운로드 경로나 개인 환경 경로는 객체로 오래 남기지 않는다. 꼭 필요할 때만 가까운 위치에서 직접 쓴다.
- 원자료에서 가져온 값과 digitized/계산값이 섞이는 경우 이름에 `table`, `digitized`, `curve`, `model` 등을 넣어 출처를 구분한다.
- 단위는 컬럼명에 명시한다. 예: `stress_MPa`, `stress_ksi`, `test_temp_C`, `test_temp_F`.
- 저장된 결과물은 노트북 목적과 폴더 이름이 맞도록 유지한다.

## 수정 범위

- 사용자가 특정 노트북이나 청크를 지정하면 그 범위 안에서 우선 해결한다.
- 전체 코드 리팩토링, 폴더 재구성, 공통 utility 정리는 명시적으로 요청받았을 때만 한다.
- 관련 없는 파일의 포맷팅, 이름 변경, helper 이동은 하지 않는다.
- 이미 사용자가 수정한 파일이나 dirty worktree는 되돌리지 않는다.

## 검토 기준

- 코드 변경 후에는 최소한 문법적으로 눈에 띄는 오류, 잘못된 객체명, 잘못된 caption, facet/point 데이터 불일치를 확인한다.
- 실행 검증이 필요한 경우 가능한 가장 작은 청크나 관련 노트북 단위로 확인한다.
- 검증하지 못한 부분은 최종 보고에 명확히 남긴다.
