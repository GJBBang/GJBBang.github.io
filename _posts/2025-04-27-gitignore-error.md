---

layout: post

title: .gitignore 적용이 왜 안되는 것인가 .. !

categories: git

tag: [error, oops, easyfix]
---



"최근 들어 아니, 이미 오래 전부터 나태해진 나 자신을 돌아보게 된 계기(?)가 생겼다..
조금은 생산성 있는 하루하루를 보내기 위해 혼자서 간단한 프로젝트를 시작했다."

**그런데 .. !**

#### 시작하자마자 위기 봉착.. 😭

싱글벙글 git hub repo를 생성하고, 프로젝트도 생성해서 올렸다.
repo를 확인 하던 중, application.properties 파일이 눈에 띄었다.

'어 ? 얘는 .gitignore에 포함되어 있을텐데 ?.?'

```properties
## .gitignore ##
...
# === 환경 변수 파일 / 민감 설정 파일 ===
*.env
.env
application-*.yml
application-*.properties
...
```

gpt : "application-*.properties 여기에 하이픈 '-' 이 있으니까 너가 작성한 application.properties는 적용이 안되는거야. 알겠니 ?"

'아하! *.properties 를 그냥 추가 해야겠다!'

*.properties를 .gitignore에 추가하고 properties 파일을 수정 테스트를 했는데,

![스크린샷 2025-04-27 오후 5.19.46](assets/images/202504/스크린샷 2025-04-27 오후 5.19.46.png)

'뭐지 ? 왜 계속 추적당하는거지 ??'

### 문제 가능성 2가지

1. **Git 캐시가 깨끗하게 비워지지 않았다**
   - 파일은 깃에서 제거되지 않고 남아있을 수도 있어.
2. **.gitignore 적용이 제대로 안 됐다**
   - .gitignore를 수정했는데 Git이 아직 새 규칙을 읽지 못했을 수도 있어.

우리의 gpt 선생님의 말씀이다.. 나의 경우엔 99,999999% 1번의 경우라고 확신하고,
git 캐시를 지우고 다시 시도했다.

```bash
## 1. git 캐시 비우기 (실제 파일 제거가 아님)
git rm -r --cached .

## 2. 다시 git에 추가
git add .

## 3. 커밋
git commit -m "Apply updated .gitignore rules"

```

### 왜 이렇게 해야 하냐?

- `.gitignore`는 "git add 하기 전에" 걸러주는 필터야.
- "이미 git add 되어 있는 파일"은 `.gitignore` 규칙이 적용되지 않아.
- `git rm --cached`는 **"추적만 제거"** 해서, .gitignore가 먹힐 기회를 다시 주는 거야.



즉, 한 번이라도 git add 처리된 파일은 .gitignore에 작성하더라도 적용되지 않는다.

![스크린샷 2025-04-27 오후 5.27.44](assets/images/202504/스크린샷 2025-04-27 오후 5.27.44.png)

실제로 git 캐시를 지우고 나서, 해당 파일은 ~~똥색~~이 되었다. (인텔리제이 Xcode-dark 테마)

git도 오랜만이고, 초기 세팅도 오랜만이라 벌써부터 고생길이 훤하다..

