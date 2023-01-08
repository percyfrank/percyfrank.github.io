---
title: "[IntelliJ IDEA] Code Convention Formatting 자동화"
excerpt: "자주 사용하는 Code Convention 자동화 설정들을 정리한다."
categories:
  - IDE
tags:
  - IDE
toc: true
toc_sticky: true
last_modified_at: 2020-09-25T20:13:00-05:00
---

## 1. Code Convention Auto-Formattiong

파일 저장시 Import 및 Line 정렬 등 Auto Formatting을 지원한다. Save Actions 플러그인을 설치하고 몇 가지 설정만 해주면 Code Convention Formatting을 자동화할 수 있다.

<br>

## 2. Save Actions

![image](https://user-images.githubusercontent.com/56240505/95673812-07bb5300-0be7-11eb-9875-4a88182ca936.png)

플러그인을 설치한 뒤, ``Settings - Other Settings - Save Actions`` 탭으로 이동하여 다음과 같이 설정해준다. batch 등의 옵션은 생략해도 무방하다.

![image](https://user-images.githubusercontent.com/56240505/95673887-77314280-0be7-11eb-9193-f8a9bede5592.png)

정렬을 원하지 않는 파일들은 ``File Path Exclusions``에 항목을 추가해준다.

<br>

## 3. IntelliJ Setting

![image](https://user-images.githubusercontent.com/56240505/95674019-73ea8680-0be8-11eb-9112-4d670a0b592b.png)

``Settings - Editor - General`` 탭의 Save Files 항목에서 ``Ensure an empty line at the end of a file on Save``를 체크해준다. 모든 파일의 마지막 줄에 Newline을 추가해준다.

![image](https://user-images.githubusercontent.com/56240505/95673933-bfe8fb80-0be7-11eb-9446-8ba0654724f2.png)

``Settings - Editor - General - Auto Import`` 탭에서, ``Add unambiguous imports on the fly`` 및 ``Optimize imports on the fly`` 항목을 체크해준다. 해당 옵션들은 Import를 자동으로 체크해 최적화해준다.

<br>

---

## References

* [How to make IntelliJ IDEA insert a new line at every end of file?](https://stackoverflow.com/questions/16761227/how-to-make-intellij-idea-insert-a-new-line-at-every-end-of-file)
* [[IntelliJ IDEA] Auto formatting](https://namocom.tistory.com/781)
