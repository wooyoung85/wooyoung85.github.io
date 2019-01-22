> Mac을 사용한지 6개월 정도 된 초보 유저입니다.  
`Marp` 라는 어플리케이션을 사용하다 창이 화면 밖으로 사라져서 고생했었던 경험을 정리해 보겠습니다.

## Marp 간단한 소개
#### Markdown 문서 작성 Tool
#### 주로 사용하는 기능은?
- Slide Show 형태로 Presentation
- PDF 문서로 변환

### 암튼 이 어플리케이션을 잘 쓰다가 어느날 갑자기 화면 밖으로 창이 사라져 버렸다 ㅠ.ㅠ  
~~DisplayLink 사용해 보겠다고 드라이버 설치하고 난리치다가 이런 현상이 생긴 것 같다;;~~
### 아무리 만지작거려도 벗어난 창은 돌아올 생각이 없고...나는 아직도 컴맹을 벗어나지 못했구나 자책하면서 이리저리 찾아보다가 AppleScript로 창 크기와 위치를 조절할 수 있다는 글을 보았다. 👏

## AppleScript 실행
```applescript
tell application "System Events" to tell window 1 of process "Marp"
	set position to {100, 100}
	set size to {800, 800}
end tell
```