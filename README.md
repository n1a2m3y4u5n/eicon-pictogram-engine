# eicon-pictogram-engine

`.p` 소스 파일을 픽토그램 PNG와 PDF로 컴파일하는 작은 컴파일러입니다. SnapCare가 쓰는 픽토그램 합성 엔진(SIGIL)의 독립 버전입니다.

## 소개

글을 읽기 어려운 환자를 위한 의약품, 의료기기 사용 안내용 픽토그램을 만듭니다. `\twice_daily{\pill_bottle{\food_bowl}}`처럼 머리(head)와 수식어(modifier)를 조합한 매크로 한 줄이 그림 한 장이 됩니다. 정해진 그림 중 하나를 고르는 방식이 아니라 부품을 조합해 새 그림을 만들기 때문에, 사전에 부품을 추가하면 코드 변경 없이 표현이 늘어납니다. SnapCare 저장소의 `engine/`과 같은 `.p` 문법을 씁니다.

## 주요 기능

- 한 줄이 한 개의 라벨입니다. 라벨 하나가 PNG 한 장이 되고, 모든 라벨을 순서대로 모은 PDF도 만듭니다.
- 숫자를 쓰지 않습니다. 수량은 반복(`\twice_daily`, `\three_times_daily`, `\four_times_daily`)이나 `\half{...}`로 나타냅니다.
- 여러 단계는 왼쪽에서 오른쪽 배치로만 순서를 나타냅니다. 출력에 글자나 숫자가 없습니다.
- `dictionary.json`에 항목을 추가하고 512x512 투명 PNG를 `assets/head/` 또는 `assets/modifier/`에 넣으면 부품이 늘어납니다. 그리는 규격은 `assets/DESIGN_GUIDE.md`에 있습니다.

### 문법

```
\head                                  머리만
\head{\modifier}                       머리의 슬롯 안에 수식어 하나
\head{\modifier}{\modifier}            첫 번째는 안쪽, 두 번째는 머리 위
\head{\modifier}{\modifier}{\modifier} 첫 번째는 안쪽, 나머지는 위쪽 한 줄
\twice_daily{X}                        X를 2칸에 반복
\three_times_daily{X}                  3칸에 반복
\four_times_daily{X}                   4칸에 반복
\no{X}                                 전체 위에 굵은 가위표
\half{X}                               X의 왼쪽 절반만 표시
```

소스 파일 예시입니다. `%` 또는 `#`으로 시작하면 줄 끝까지 주석입니다.

```
% 항생제: 손을 씻고, 하루 두 번 음식과 함께 알약
\hand_wash \twice_daily{\pill_bottle{\food_bowl}}

% 반 알을 하루 세 번 물과 함께
\three_times_daily{\half{\pill_bottle{\water_glass}}}

% 안약 하루 두 번, 임신 중에는 사용 금지
\hand_wash \twice_daily{\dropper_bottle{\body_eye}} \no{\state_pregnant}
```

첫 번째 수식어는 머리의 슬롯 안에 `frame.modifier_scale` x 패널 크기(기본 0.67)로 그려지고, 나머지는 머리 위에 가로 한 줄로 놓입니다. 중요한 수식어를 먼저 쓰세요.

## 기술

- Python 3.10+, Pillow
- 구성: `main.py`(컴파일러), `engine.py`(파서, 합성기, 렌더러), `dictionary.json`(어휘)

## 실행 방법

```bash
pip install Pillow
python main.py examples/sample.p
```

`path/to/test1.p`를 컴파일하면 `path/to/test1/` 폴더에 라벨별 번호 PNG와 `test1.pdf`가 생깁니다.

옵션:

- `--out DIR` 출력 폴더 지정
- `--no-pdf` PDF 생략
- `--size N` 패널 크기(픽셀, 기본 512)
- `--list` 사용할 수 있는 매크로 목록을 출력하고 종료
- `--info` 매니페스트 요약을 출력하고 종료
