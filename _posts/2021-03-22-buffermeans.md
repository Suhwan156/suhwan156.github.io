---
layout: single
title:  "버퍼오버플로우 공격이란?"
---
# 보안기사를 공부하며 정리했던 내용입니다.

	1) 버퍼오버플로우 공격(Buffer overflow Attack)
		○ 개요
			i. 버퍼 오버플로우
				1) 연속된 메모리 공간을 사용한느 프로그램에서 할당된 메모리의 범위를 넘어선 위치에 자료를 읽거나 쓰려고 할때 사용
				2) overflow 발생 시 프로그램의 오작동을 유발시키거나, 악의적인 코드를 실행 -> 공격자 통제권한 획득
			ii. Stack / Heap
				1) Stack buffer overflow : 스택은 함수 처리를 위해 지역변수 및 매개변수 위치하는 메모리 영역, 스택에 할당된 버퍼들이 문자열 계산 등에 의해 정의된 버퍼의 한계치를 넘는 경우 버퍼 오버플로우 발생하여 Return Address 변경하고 공격자가 원하는 임의코드 실행
				2) Heap buffer overflow : 힙은 사용자가 동적으로 할당하는 메모리영역(malloc이용), 힙에 할당된 버퍼들에 문자열 등이 저장될 때, 최초 정의된 힙의 메모리 사이즈를 초과하여 문자열 등이 저장되는 경우 버퍼 오버플로우가 발생하여 데이터와 함수 주소등을 변경하여 공격자가 원하는 임의 코드를 실행
				
		○ C언어 함수
			i. 주요 함수
				1) strcpy(char *dst, const char *src) : src 문자열을 dst 버퍼에 저장, src 문자열의 길이를 체크하지 않으므로 dst 버퍼를 초과하는 결과발생
				2) strncpy(char *dst, const char *src, size_t len) : src 문자열을 len 만큼 dst버퍼에 저장, src 문자열의 길이를 제한하기 때문에 overflow 방지
				3) size_t strlen(const char *str) : 문자열의 null 문자를 제외한 바이트수를 반환
				4) sizeof(피연산자) : 피연산자의 크기를 반환
			ii. C언어에서 문자열 처리방식
				1) C언어에서 문자열은 null 문자(0x00)로 문자열 끝 반환 ex) ABCD 문자는 실제로 “ABCD\0” 으로 저장
				
		○ Stack overflow 발생 시 스택 구조

			i. Stack frame : 모든 함수는 호출이 되면 자신만의 스택공간이 할당, frame 공간 내에서 SFP를 기준점으로 하여 stack pointer에 상대주소(offset)을 저장하여 메모리 접근을 하게 됨
			ii. 현재 실행중인 함수의 SFP는 EBP 레지스터에 저장되고, 스택포인터는 ESP 레지스터에 저장, 다음 실행 명령어 주소는 EIP 레지스터에 저장
			iii. 함수 호출되면 이전 함수의 다음 실행할 명령어의 주소정보와 SFP를 먼저 스택에 저장
			iv. RET영역(Return Address, 이전 함수의 다음 실행 명령어의 주소를 저장하는 영역), SFP영역(이전함수의 SFP 저장하는 영역) / 만약 RET 영역이 악성코드가 위치한 주소로 변조 -> 함수 종료 후 악성코드 실행 가능
			v. 공격자는 RET영역 변조를 위해 버퍼오버플로우 이용하여 SFP영역을 덮어쓴 후 악성코드가 위치한 주소값으로 RET 영역을 덮어 씀 => main 함수 종료 시 RET 주소값을 참조하여 악성코드실행
		
		○ Buffer overflow 대응방안
			i. 안전한 함수 사용
				□  strcpy -> strncpy 교체 사용, strncpy(char *dst, const char *src, size_t len)은 len만큼 dst버퍼에 저장하기 때문에 오버플로우 발생하지 않음
				□ sizebuffer-1로 값 설정, c언어에서 문자열은 null로 끝나도록 정의하기 때문에 입력값이 아무리 크다해도 null 포함될수 있도록 하기위해 1byte 공간을 남기고 복사하기 때문임
			ii. 입력값 사전 검증
				1) 입력값이 버퍼와 크거나 같으면 overflow 발생하므로 값 검증하여 종료시킴(null제외로 동일값도 종료)
		○ stack buffer overflow 대응기술
			i. stack guard : 메모리상에서 프로그램의 return address와 변수 사이에 특정 값을 저장해 두었다가 그 값이 변경되었을 경우 overflow로 가정하여 프로그램 중단시킴
			ii. stack shield : 함수 시작 시 return address를 GLOBAL RET에 저장해두었다가 함수 종료시 저장했던 값과 비교하여 overflow 판단
			iii. ASLR(address space layout randomization) : 메모리 공격을 방어하기 위해 주소 공간 배치를 난수화 -> 실행시마다 메모리주소 변경시켜 악성코드 의한 특정주소 호출 방지
