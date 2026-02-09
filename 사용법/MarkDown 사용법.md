# 샵 1개 Heading 
## 샵2개 Heading
### 샵 3개 Heading
### Heading 을 # 을 쓰고 띄어쓰기 사용 가능
### 주의사항 : # 을 쓰고 안 띄어쓰면 #해시태그 이렇게 됨.

- 블랙포인트는 -를입력하고 띄어쓰기로 사용
- Enter로 계속 유지
**글씨 굵게 : ** 


# 추가 Markdown 사용법

## 텍스트 서식

-   **기울임꼴 (Italic)**:
    -   `*기울임꼴*` 또는 `_기울임꼴_`
    -   예시: *이것은 기울임꼴입니다.*

-   **취소선 (Strikethrough)**:
    -   `~~취소선~~`
    -   예시: ~~이것은 취소선입니다.~~

-   **코드 (Code)**:
    -   **인라인 코드**: `백틱(``)으로 감싸기`
        -   예시: `console.log("Hello, Markdown!");`
    -   **코드 블록**: 백틱 3개(```)로 시작하고 끝내기. 첫 백틱 뒤에 언어 이름을 추가하면 문법 강조(Syntax Highlighting)가 됩니다.
        ```python
        def hello_world():
            print("Hello, Markdown!")
        ```

## 링크 및 이미지

-   **링크 (Link)**:
    -   `[링크 텍스트](URL)`
    -   예시: [Obsidian 공식 웹사이트](https://obsidian.md/)
    -   **Obsidian 내부 노트 링크**: `[[노트 제목]]`
        -   예시: <a href="obsidian://open?file=%EC%82%AC%EC%9A%A9%EB%B2%95%2FMarkDown%20%EC%82%AC%EC%9A%A9%EB%B2%95.md">MarkDown 사용법</a>

-   **이미지 (Image)**:
    -   **웹 이미지**: `![대체 텍스트](이미지 URL)`
        -   예시: ![Obsidian Logo](https://obsidian.md/images/obsidian-logo.png)
    -   **Obsidian 내부 이미지**: `![[이미지 파일명.png]]`
        -   예시: ![[내부 이미지.png]]

## 인용 및 목록

-   **인용 (Blockquote)**:
    -   `> 인용할 내용`
    -   예시:
        > 이것은 인용문입니다.
        > 여러 줄로 이어질 수 있습니다.

-   **순서 있는 목록 (Ordered List)**:
    -   `1. 첫 번째 항목`
    -   `2. 두 번째 항목`
    -   예시:
        1.  사과
        2.  바나나
        3.  오렌지

## 구분선

-   **수평선 (Horizontal Rule)**:
    -   `---` 또는 `***` (세 개 이상의 하이픈 또는 별표)
    -   예시:
        ---
        위와 아래를 구분하는 선입니다.

## 표 (Table)

-   `| 헤더1 | 헤더2 |`
-   `|---|---|` (헤더와 내용을 구분하는 선)
-   `| 내용1 | 내용2 |`
-   예시:

| 항목       | 설명            |
| -------- | ------------- |
| Markdown | 텍스트 기반 마크업 언어 |
| Obsidian | 지식 관리 도구      |
| Copilot  | AI 비서         |

---