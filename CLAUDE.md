# Travel Itinerary Project

## Project Structure

```
여행일정/
├── CLAUDE.md
├── package.json          # md-to-pdf, pdf-to-img
├── pnpm-lock.yaml
├── .gitignore
└── schedules/
    └── {trip-name}/
        ├── {trip-name}.md   # 마크다운 원본
        ├── {trip-name}.pdf  # PDF 변환본
        └── images/          # 페이지별 PNG 이미지
```

## Skills

### 여행 일정 마크다운 작성
- 기존 일정(schedules/ 하위)의 포맷과 톤을 따른다
- 구성: 여행 개요 테이블 → 상세 일정(Day별) → 숙박 요약 → 이동 경로 → 비용 정리 → 체크리스트
- 이모지 사용 OK (여행 일정 특성상)
- 교통비, 입장료 등 구체적 금액 포함
- 맛집/숙소 추천은 볼드 처리

### MD → PDF 변환
- **JS 파일 만들지 말고 npm 패키지 CLI를 직접 사용한다**
- 명령어:
  ```bash
  npx md-to-pdf --pdf-options '{"format":"A4","margin":{"top":"20mm","bottom":"20mm","left":"15mm","right":"15mm"},"printBackground":true}' schedules/{trip-name}/{file}.md
  ```

### PDF → 이미지 변환
- node one-liner로 실행 (별도 스크립트 파일 불필요):
  ```bash
  node -e "
  import('pdf-to-img').then(async({pdf})=>{
    const fs=require('fs'), path=require('path');
    const f=process.argv[1];
    const d=path.join(path.dirname(f),'images');
    fs.mkdirSync(d,{recursive:true});
    let i=1;
    for await(const p of await pdf(f,{scale:2})){
      fs.writeFileSync(path.join(d,'page_'+String(i++).padStart(2,'0')+'.png'),p);
      console.log('page '+(i-1));
    }
  })" -- schedules/{trip-name}/{file}.pdf
  ```

### 새 여행 일정 추가 시 워크플로우
1. `schedules/{trip-name}/` 폴더 생성
2. 마크다운 작성 (`{trip-name}.md`)
3. `npx md-to-pdf` 로 PDF 변환
4. `node -e "import('pdf-to-img')..."` 로 이미지 변환
5. 이미지 확인

### 폴더/파일 네이밍 규칙
- 여행 폴더: 한글 이름 사용 (예: `미서부여행`, `오사카나라교토`)
- 이미지 폴더: 반드시 `images/` (pages 아님)
- 이미지 파일: `page_01.png`, `page_02.png` ... (2자리 zero-pad)
- PDF/MD 파일: 폴더명과 동일하게 + `_{연도}` suffix (예: `_2026`)

## Preferences

- 한국어로 대화한다
- 불필요한 중간 스크립트 파일 만들지 않기 — npm 패키지 CLI 직접 활용
- 음성 입력 사용하는 유저 — 음성 인식 오류 감안하여 맥락으로 의도 파악할 것
- 새 여행 일정 요청 시 바로 CLAUDE.md의 워크플로우대로 실행할 것 (폴더 생성 → MD 작성 → PDF → images)
