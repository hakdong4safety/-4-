# 현장 안전점검 사진대지 시스템

현장에서 사진을 찍어 바로 등록하면, 사무실에서 날짜별로 확인 후 엑셀에 붙여넣기만 하면 되는 웹 앱입니다.
로그인 없이 링크 하나로 누구나 등록/열람할 수 있습니다.

---

## 폴더 안 파일

| 파일 | 용도 |
|---|---|
| `index.html` (= `현장_사진대지_시스템_firebase.html`을 이름 바꾼 것) | 실제 배포용 앱. **이 파일 하나만 있으면 됩니다.** |
| `README.md` | 이 설명 파일 |

> `_firebase.html`이 붙지 않은 버전은 클로드(Claude) 채팅 안에서만 동작하는 테스트용이라, 깃허브에는 **`_firebase.html` 버전을 `index.html`로 이름을 바꿔서** 올려야 합니다.

---

## 1. 처음 배포하기

### 1) Firebase 설정값 넣기
1. [Firebase 콘솔](https://console.firebase.google.com)에서 프로젝트 생성
2. **Build → Firestore Database → Create database** (테스트 모드로 시작)
3. 프로젝트 개요에서 **`</>` 웹 아이콘 → 앱 등록** → 나오는 `firebaseConfig` 값을 복사
4. `index.html` 파일을 열어서 상단 `<script>` 안의 아래 부분을 방금 복사한 값으로 교체

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

5. Firestore **Rules** 탭에서 아래 규칙을 붙여넣고 Publish (내부용 도구이므로 로그인 없이 읽고 쓸 수 있게 설정)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /appdata/{doc} {
      allow read, write: if true;
    }
  }
}
```

### 2) 깃허브에 올리기
1. [github.com](https://github.com)에서 새 저장소(Repository) 생성 (Public)
2. **Add file → Upload files** 로 `index.html` 업로드 → Commit
3. 저장소 **Settings → Pages** → Source: `Deploy from a branch`, Branch: `main` / `(root)` → Save
4. 1~2분 후 뜨는 `https://사용자명.github.io/저장소이름/` 링크가 실제 배포 주소

이 링크를 현장 작업자와 사무실 직원 모두에게 공유하면 됩니다.

---

## 2. 사용 방법

### 📋 현장등록 탭
- 상단에서 날짜 선택 (기본값: 오늘)
- **일일 작업계획서 점검 결과 보고**: 사진 카드 4칸 중 빈 칸을 눌러 사진 촬영/업로드 + 내용 문구 선택
- **사진대지 등록 현황**: 7개 항목(고위험작업 회의, 5분 특별안전교육 등) 카드를 눌러 사진 등록. 등록되면 "등록완료" 배지로 바뀝니다.

### 🖨 인쇄 탭
- 날짜 선택 후 **전체 내보내기**를 누르면 그날 등록된 모든 항목이 자동 선택되어 미리보기가 만들어집니다.
- 항목별로 체크해서 일부만 내보낼 수도 있습니다 (사진 없는 항목은 자동 비활성화).
- 미리보기가 뜨면 **복사하기** 버튼 클릭 → 엑셀 시트에서 `Ctrl+V`로 붙여넣기.
  - 이미지가 붙여넣기 안 되면 사진을 개별로 우클릭 → 복사 → 엑셀에 붙여넣기.

### ⚙ 설정 탭
- 현장명, 회사명, 로고 이미지 변경
- 사진 내용 문구 목록 추가/수정/삭제 (문구에 `{날짜}`를 넣으면 등록일로 자동 치환)

---

## 3. 데이터 저장 구조 (참고용)

Firestore의 `appdata` 컬렉션에 문서 단위로 저장됩니다. 사진이 커서 하루치를 통째로 저장하면 Firestore 문서 용량 제한(1MB)에 걸릴 수 있어, 슬롯/항목 단위로 나눠 저장합니다.

- `settings` : 현장명, 회사명, 로고, 담당자 이름
- `phrases` : A양식 사진 내용 문구 목록
- `day:{날짜}:A0` ~ `A3` : 일일 작업계획서 점검 결과 보고, 사진 슬롯 1~4번
- `day:{날짜}:B0` ~ `B6` : 사진대지 7종류 (일일 고위험작업 회의 / 5분 특별안전교육 / 근로자 청취 / O.P.C / 고위험 특별 안전교육 / 피드백 회의 / 노사협의체)

사진은 촬영 즉시 브라우저에서 압축(가로 1000px, JPEG)한 뒤 base64로 저장됩니다.

---

## 4. 수정할 때

파일(`index.html`)을 로컬에서 수정한 뒤, 깃허브 저장소에서 **Add file → Upload files** 로 같은 파일명으로 다시 올리고 Commit하면 몇 초 뒤 같은 배포 링크에 자동 반영됩니다.

---

## 5. 보안 관련 참고

이 앱은 로그인이 없는 내부용 도구입니다. Firestore 규칙을 `allow read, write: if true`로 열어두었기 때문에, **배포 링크와 Firebase 프로젝트 정보를 외부에 공개하지 않도록 주의**해 주세요. 더 강한 보안이 필요하면 Firebase Authentication을 추가하는 방법도 있습니다 (필요하시면 요청해 주세요).
