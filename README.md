# 🧹 SQL 데이터 클리닝 프로젝트: Nashville Housing Data

## 📌 1. 프로젝트 개요
미국 내슈빌(Nashville) 지역의 부동산 데이터셋(`NashvilleHousing`)을 분석하기에 적합한 형태로 정제(Data Cleaning)하는 프로젝트입니다. 원시 데이터(Raw Data)에 존재하는 결측치, 일관되지 않은 형식, 중복 데이터 등의 문제를 SQL을 활용하여 해결하고 데이터의 신뢰성을 높였습니다.

- **사용 툴:** SQL Server (SSMS)
- **주요 테크닉:** Window Function (`ROW_NUMBER`), CTE, 문자열 파싱 (`SUBSTRING`, `PARSENAME`), 가상 테이블 수정 (`ALTER`, `UPDATE`)

---

## 🛠️ 2. 핵심 데이터 정제 과정

### ① 날짜 형식 표준화 (`Standardize Date Format`)
- **문제점:** `SaleDate` 컬럼의 형식이 분석에 불필요한 시/분/초 정보까지 포함하여 일관성이 떨어짐.
- **해결 방식:** `CONVERT(Date, SaleDate)`를 활용해 연-월-일 형식으로 통일하고, `ALTER TABLE`을 통해 `SaleDateConverted` 새 컬럼을 추가하여 정제된 데이터를 안전하게 보관함.

### ② 주소 결측치 처리 (`Populate Property Address data`)
- **문제점:** 일부 데이터에서 `PropertyAddress`(주소)가 `NULL`로 비어 있는 현상 발견.
- **해결 방식:** 동일한 `ParcelID`를 가진 다른 데이터는 주소가 동일하다는 규칙을 발견. 자기 자신을 조인(`Self JOIN`)한 후 `ISNULL()` 함수를 사용하여 결측치를 다른 행의 정상 주소 데이터로 채워 넣음.

### ③ 문자열 분리를 통한 주소 데이터 세분화 (`Breaking out Address`)
- **문제점:** 주소, 도시, 주(State) 정보가 쉼표(`,`) 하나로 묶여 있어 개별 분석(예: 도시별 통계)이 어려움.
- **해결 방식:** - `SUBSTRING`과 `CHARINDEX`를 조합하여 첫 번째 주소와 도시를 분리.
  - `OwnerAddress`는 세 덩어리 분리가 필요하여, `REPLACE`로 쉼표를 점(`.`)으로 바꾼 뒤 `PARSENAME()` 함수를 활용해 주소, 도시, 주(State)를 깔끔하게 3개의 독립된 컬럼으로 파싱함.

### ④ 불필요하게 세분화된 데이터 통합 (`Change Y and N to Yes and No`)
- **문제점:** `SoldAsVacant`(공실 여부) 컬럼에 `Yes`, `No`, `Y`, `N` 4가지 값이 혼용되어 데이터의 일관성이 깨짐.
- **해결 방식:** `CASE WHEN` 문을 사용하여 `Y`는 `Yes`로, `N`는 `No`로 일괄 업데이트하여 텍스트 데이터의 도메인을 통일함.

### ⑤ 중복 데이터 제거 (`Remove Duplicates`)
- **문제점:** 동일한 부동산 거래 건이 중복 입력되어 데이터 분석 시 왜곡을 유발할 수 있음.
- **해결 방식:** `ParcelID`, `PropertyAddress`, `SalePrice`, `SaleDate`, `LegalReference` 등 핵심 컬럼이 완전히 일치하는 행을 찾기 위해 **`ROW_NUMBER() OVER (PARTITION BY ...)`** 윈도우 함수를 적용. 이를 **`CTE(공통 테이블 식)`**로 감싸서 중복된 데이터(행 번호 > 1)를 정확하게 식별하고 필터링함.

### ⑥ 미사용 컬럼 삭제 (`Delete Unused Columns`)
- **문제점:** 텍스트 파싱 및 정제가 완료되어 더 이상 원본 컬럼(`OwnerAddress`, `PropertyAddress`, `SaleDate` 등)과 불필요한 메타데이터가 필요 없어짐.
- **해결 방식:** `ALTER TABLE DROP COLUMN`을 사용하여 저장 공간을 최적화하고 테이블 구조를 간소화함.

---

## 📈 3. 프로젝트를 통해 얻은 역량
- `Self JOIN` 및 `ISNULL`을 활용한 데이터 보간(Imputation) 능력
- 복잡한 문자열 데이터를 원하는 형태로 자유롭게 파싱하는 가공 능력
- `CTE`와 `Window Function`을 조합하여 대용량 데이터에서 중복 데이터를 효율적으로 처리하는 실무 쿼리 작성 역량
