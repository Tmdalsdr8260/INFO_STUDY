# 파일 포맷

## 정의
- ( = 파일 형식 )
- 저장을 목적으로 정보를 컴퓨터 파일로 인코딩하는 특별한 방식

# PE 파일 포맷

## 정의
- Windows 운영체제에서 사용되는 파일 형식 중 하나
- PE 포맷을 사용하는 파일 확장자 : 
    1) 실행 파일 계열       Ex) EXE, SCR
    2) 라이브러리 계열      Ex) DLL, OCX
    3) 드라이버 계열        Ex) SYS
    4) 오브젝트 파일 계열   Ex) OBJ

## PE 파일 구조
| First Header  |
| :------------ |
| First row     | Data          | Very long data entry |
| Second row    | **Cell**      | *Cell*               |
| Third row     | Cell that spans across two columns  ||