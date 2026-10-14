# SOE Fiscal Risk Dashboard — GitHub/Streamlit 업데이트 안내

## 기존 데이터와 화면 보존

- 프로젝트 시작 시 기존 8개 SOE의 **데모 포트폴리오**를 그대로 표시합니다.
- 세네갈 Senelec의 실측 데이터는 프로젝트에 포함하지 않았습니다.
- 상단 **Upload data**에서 `tester_Senegal_Senelec_2021-2024.xlsx`를 직접 선택하면 해당 데이터로 전환됩니다.
- 앱을 새로고침하거나 데모 데이터를 다시 선택하면 원래 데이터로 돌아갑니다.
- 9개 메뉴와 기존 계산 방식, 수익성 드롭다운의 기존 항목을 유지합니다.

## 기본 화면부터 제공되는 추가 분석 (업로드 시 데이터 갱신)

1. **KPI Dashboard**: 기존 네 개의 KPI 차트 아래에 매출에 포함된 요금보전액을 제외한 EBIT 마진의 민감도 분석이 나타납니다. 회계적으로 재작성한 실적이 아니라 단순 민감도 계산입니다.
2. **Government Support**: 기존 정부지원/등급 조정 분석 아래에 연도별 요금보전액, 운영보조금, 투자보조금, 자본투입을 구분한 그래프와 표가 나타납니다. 항목을 자동 합산하지 않으므로 중복집계되지 않습니다.
3. **Assumptions**: 업로드된 엑셀의 `Grants_detail`, `Source_lines`, `Validation`, `Checks`, `Definitions`에서 추가 근거와 검사 결과를 표시합니다.
4. **Shock Scenarios**: 관측된 충격 노출 비율이 없을 때 기본 가정값이 사용된다는 안내가 나타납니다.
5. 선그래프는 기존 그래프를 포함해 **모두 부드러운 곡선**으로 표시됩니다. `ADDED` 표시를 하지 않습니다.

## GitHub에 전체 프로젝트 올리기

1. 제공된 ZIP을 **압축 해제**합니다. ZIP 내부 `SOE-main` 폴더의 내용을 확인합니다.
2. 본인의 GitHub Repository에서 **Add file → Upload files**를 선택하고 해당 폴더 안의 파일 및 하위 폴더를 업로드합니다. `Home.py`, `report.py`, `schema.py`, `templates/` 등 구조를 유지하세요. 기존 저장소에 덮어쓸 경우 먼저 원본을 백업하거나 커밋을 남기세요.
3. 변경 사항을 Commit합니다. Streamlit Community Cloud에서는 기존 앱이 `Home.py`를 실행하도록 유지하면 배포가 갱신됩니다.
4. 앱이 열리면 첫 화면의 기존 데모 데이터를 확인한 다음, **Upload data**에서 Senelec 파일을 선택해 추가 분석을 확인합니다.

로컬 실행:

```bash
pip install -r requirements.txt
streamlit run Home.py
```

## 참고

- `.xlsx` 업로드는 기존 앱과 마찬가지로 브라우저에서 외부 SheetJS 라이브러리를 한 번 불러옵니다. 외부 CDN이 차단된 환경이라면 해당 라이브러리의 접근 허용이 필요합니다.
- 파일에 `Data`라는 시트가 있으면 이를 기본 시트로 사용하며, 없으면 첫 번째 시트를 사용합니다.
- Senelec의 `Utility` 업종명은 기존 모델의 `Power / Utilities`로 자동 정규화됩니다.
- 실측 데이터가 부족한 충격 관련 지표는 기존 모델의 업종별 가정이 사용될 수 있으므로 해석에 유의하세요.

## Cost Recovery (KPI Dashboard → Profitability → Ratio)

- **Cost recovery (x)**를 선택하면 기존 Profitability 카드 안에서 같은 연도별 곡선 그래프를 조회할 수 있습니다. 별도 화면이나 그래프는 생성하지 않습니다. Compare across SOEs와 Full scorecard에도 동일 KPI가 적용됩니다.
- 공식: **(Revenues − Government Grants) / (Cost of Goods Sold + Other Operating Expenses)**.
- 엑셀 열 이름 `Cost of Goods Sold`, `Other Operating Expenses` 또는 표준 키 `cost_of_goods_sold`, `other_operating_expenses`를 인식합니다.
- 값이 없는 경우 `Operating Expense`로 대신 추정하지 않습니다. **Cost recovery (x) 선택지는 항상 표시**되며, 필요한 원시 입력값이 없는 경우 그래프 대신 `Data unavailable` 메시지를 표시합니다. 분모가 0 이하이거나 일부 연도만 데이터가 누락된 경우에도 해당 값은 `n/a`입니다.
- `Government Grants`는 **Revenues에 포함된 정부지원금**이어야 합니다. 이미 정부지원금을 제외한 매출액을 넣었다면 이 공식을 그대로 적용하면 이중차감됩니다. PSO/RMA가 `Government Grants`에 이미 포함되었다면 중복으로 추가 차감하지 않습니다. 투자보조금이나 자본투입은 비용회수율의 이 항목에 넣지 않습니다.
- 기준선은 임시 예시로 **alert < 1.00×, good ≥ 1.10×**입니다. 실제 사용에서는 정책 목적과 업종에 따라 보정하세요.
- 기본 데모 원본은 변경하지 않습니다. COGS/기타 영업비용이 없는 기본 데모에서도 **Profitability → Ratio → Cost recovery (x) 선택지는 항상 보입니다**. 이 경우에는 `Data unavailable` 안내가 나타나며 Senelec 엑셀을 업로드하면 자동으로 연도별 그래프를 표시합니다.
- Senelec `tester_Senegal_Senelec_2021-2024.xlsx`에서의 검산: 2021=0.9577×, 2022=0.7627×, 2023=0.9023×, 2024=0.9595×.

## 모든 메뉴와 KPI 항상 표시 (업로드 불필요)

- 상단 9개 메뉴는 기본 내장 데모 데이터로 시작할 때부터 항상 조회할 수 있습니다. 현재 선택한 탭 외에 숨겨진 HTML 섹션은 통상적인 탭 전환 동작이며 메뉴를 삭제하는 것이 아닙니다.
- KPI Dashboard의 네 카테고리(Profitability, Liquidity, Solvency, Fiscal dependency) 드롭다운은 **등록된 모든 지표를 항상 제공**합니다. 기본 데이터에 원시 입력값이 없어도 Cost recovery 등 조회할 수 있습니다. EBITDA margin은 이전 요청에 따라 등록 목록에서 제외한 상태를 유지합니다.
- 필요한 원시 데이터가 없으면 해당 차트 영역에 **Data unavailable**을 표시하고 누락 필드를 안내합니다. 결측을 0으로 가정하지 않으며, 실제 데이터가 부족한 지표는 포트폴리오 위험 점수 계산에서 제외합니다.
- KPI 전체 Scorecard의 분류 메뉴와 표 열도 상시 조회됩니다. 값이 없는 칸은 n/a로 유지하며 알림 건수에 임의 포함하지 않습니다.
- 추가 정부지원 그래프와 근거 패널의 자리도 항상 표시합니다. 신규 데이터 업로드 시 같은 화면에서 값만 갱신합니다.
