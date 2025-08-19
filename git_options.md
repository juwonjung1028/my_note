# GitHub SSH 원격 연결 방법(Window 기준)  

## 1. SSH 키 만들고 GitHub에 등록하기  
1.1 기존 SSH 키가 있는지 확인  
 > ls -al ~/.ssh  
- id_ed25519 또는 id_rsa 파일쌍이 이미 있다면 건너뛸 수 있다.  

1.2 새 SSH 키 생성  
- 가능하면 Ed25519 방식 사용  
 > ssh-keygen -t ed25519 -C "깃허브로그인이메일"  
- 파일 저장 위치는 기본값 엔터  
- 패스프레이즈는 빈칸도 가능하지만 보안을 위해 입력 권장  
- 구형 환경에서 Ed25519가 안 되면 다음을 사용  
 > ssh-keygen -t rsa -b 4096 -C "깃허브로그인이메일"  

1.3 ssh-agent에 키 등록  
 > Windows PowerShell 에서  
 Get-Service ssh-agent | Set-Service -StartupType Automatic  
 Start-Service ssh-agent  
 ssh-add ~/.ssh/id_ed25519  


 **혹은 Git Bash에서**  
 > eval "$(ssh-agent -s)"
 ssh-add ~/.ssh/id_ed25519  


1.4 공개키를 GitHub에 등록  
- 공개키 내용을 복사  
 > cat ~/.ssh/id_ed25519.pub  
- GitHub 웹사이트 로그인  
- 우측 상단 프로필 사진 클릭  
- Settings  
- 좌측 메뉴에서 SSH and GPG keys  
- New SSH key  
- Title은 구분용으로 아무거나, Key에는 복사한 내용 전체 붙여넣기  
- Add SSH key  

1.5 연결 테스트  
 > ssh -T git@github.com  
- 처음이면 연결 신뢰 여부 물음에 yes 입력  
- 성공 메시지가 나오면 준비 완료  

## 2. GitHub에 저장소 만들기  
- GitHub 로그인 후 우측 상단 New repository  
- Repository name 입력  
- Public 또는 Private 선택  
- 아래 Initialize 섹션은 체크하지 않는 것을 권장  
- Readme나 License 등을 미리 만들면 첫 푸시에 충돌이 날 수 있다  
- Create repository 클릭  
- 생성 직후 표시되는 SSH 주소를 복사  

## 3. 내 컴퓨터의 특정 폴더를 Git 저장소로 만들기  
 > 아래에서 자신의 실제 경로로 바꾸어 입력  

3.1 폴더로 이동  
 > cd "C:\Users\사용자이름\Documents\MyProject"  

3.2 Git 초기화 및 기본 브랜치 설정  
 > git init  
 git branch -M main  

3.3 사용자 정보 설정 (한번만 하면 됨)  
 > git config --global user.name "홍길동"  
 git config --global user.email "깃허브로그인이메일"  

3.4 (선택 사항)불필요한 파일 제외 설정  
 > echo .DS_Store>>.gitignore  
 echo Thumbs.db>>.gitignore  
 echo .env>>.gitignore  
 echo node_modules/>>.gitignore  
 echo venv/>>.gitignore  
 echo .venv/>>.gitignore  

3.5 현재 폴더의 모든 파일을 스테이징  
 > git add .  

3.6 첫 커밋  
 > git commit -m "Initial commit"  

3.7 원격 저장소 연결  
 > git remote add origin git@github.com:계정명/MyProject.git  
 git remote -v  
- 복사해둔 SSH URL을 사용  

3.8 GitHub로 업로드 푸시  
 > git push -u origin main  

## 4. 이후 업데이트 방법  
- 폴더에서 파일을 수정하거나 추가한 뒤  
 > git add .  
 git commit -m "변경 내용 짧게 메모"  
 git push  


## 5. 자주 만나는 오류와 해결  
- Permission denied publickey  
→ 공개키를 GitHub에 올렸는지 확인  
→ ssh-agent에 키가 등록되어 있는지 확인  
 > ssh-add -L  
→ 다른 키 이름을 썼다면 ~/.ssh/config에 IdentityFile 경로를 맞춘다  
→ 상세 원인 보기  
  > ssh -vT git@github.com  

- Repository not found  
→ git remote -v 로 주소 확인. 계정명과 저장소명이 정확한지 확인  
→ 권한이 없는 개인이나 조직 저장소가 아닌지 확인  

**- Updates were rejected because the remote contains work**  
→ 저장소를 만들 때 README 등을 체크하여 원격에 먼저 커밋이 생긴 경우  
→ 해결  
 > git pull --rebase origin main  
 git push  
 
 또는 로컬이 비어 있고 원격만 유지하고 싶으면  

 > git fetch origin  
 git reset --hard origin/main  

**→ 그래도 충돌날 때 해결**  
 - 로컬 파일을 살리고 싶다면  
   > git checkout --theirs README.md  
 - 원격의 README를 살리고 싶다면  
   > git checkout --ours README.md  

 - 그다음 절차 수행  
 - 충돌 해결 마크 제거 완료로 표시  
   > git add README.md  
   git rebase --continue  
 - 푸시  
   > git push -u origin main  
   **→ 푸시 안될 때(강제 푸시)**  
     > git rebase --abort  
     git push --force-with-lease origin main  

- 파일이 100MB를 넘음  
→ GitHub 단일 파일 제한. Git LFS 사용  
→ Git LFS 설치 후  
 > git lfs install  
 git lfs track "*.psd"  
 git add .gitattributes  
 git add 경로(예: C:/XX.csv)  
 git commit -m "Track large files with LFS"  
 git push  

- Windows 경로 길이 제한  
→ 관리자 권한 PowerShell  
 > git config --system core.longpaths true  

- 라인 엔딩 경고가 거슬릴 때  
→ Windows에서 통일하려면  
 > git config --global core.autocrlf true  

- 줄바꿈 경고 없애기(Windows 권장)  
 > git config --global core.autocrlf true  

