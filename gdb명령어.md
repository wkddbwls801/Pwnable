* 시작과 종료   
-시작 : gdb[프로그램명][core 파일명][PID]   
-끝 : q or ctrl+d   

* 소스보기(list)   
-list : main함수 기점으로 소스 출력   
-list 10 : 10행을 기준으로 출력   
-list func : func함수의 소스를 출력(c++의 클래스 멤버의 경우 클래스 이름도 입력해야 함)   
-list file.c : file의 func 함수 부분을 출력   
-file.c:10 : file의 10행을 기준으로 출력   

* 브레이크 포인트(break or b)   
-break func : func 함수의 시작부분에 브레이크 포인트 설정   
-break 10 : 10행에 브레이크 포인트 설정   
-break file.c:func : file.c 파일의 func함수에 브레이크 포인트 설정   
-brek file.c:10 : file.c파일의 10행에 브레이크 포인트 설정   
-break +2 : 현재 행에서 2개 행 이후 브레이크 포인트 설정   
-break -2 : 현재 행에서 2개 행 이전 지점에 브레이크 포인트 설정   
-break * [주소] : 특정 주소에 브레이크 포인트 설정   
-break 10 if var==0 : 10행에 브레이크 포인트를 설정하는데 var값이 0일 때 작동   
-tb : break와 같으나 1회용 브레이크 포인트임. 문법은 b와 같음   
-info break : 현재 브레이크 포인트 보기   
-cl : 브레이크 포인트 지우기(옵션은 b와 유사함)   
-d : 모든 브레이크 포인트 지우기   

* 진행 명령어   
-run(r) : 프로그램 수행(ndk 환경에서는 지원되지 않음)   
-kill(k) : 프로그램 수행 종료   
-step(s) : 현재 행 수행 후 정지, 함수 호출 시 함수 안으로 들어감   
 -s 5 : step 다섯번 수행과 동일   
