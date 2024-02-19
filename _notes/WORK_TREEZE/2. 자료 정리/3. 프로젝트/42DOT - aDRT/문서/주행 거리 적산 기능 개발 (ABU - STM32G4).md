## 주행 거리 적산 기능 (ODO) 개발 리포트
### 개발 내용
기존 ILC - STM32F7 에서 개발된 기능 과 동일한 기능으로 개발 되었다.
**모터 RPM 을 통해 연산된 차속으로 주행 거리를 적산한다.**
**20ms 단위로 연산되며, EPB 체결시마다 연산된 주행거리를 적산한다.**

적산 주행거리는, CAN (0x440) 메세지를 통해 임의로 설정 할 수 있으며,
Key on 이후 현재 주행거리와, 적산 된 총 주행거리를 100ms Delay Time 으로 (CAN 0x612)
메세지를 통해 상위제어기로 전송한다.

**주행 거리 적산에 사용되는 Flash Memory 운용은 아래와 같다.**

- Index Page, Data Page 로 구별하여 데이터를 운용한다.
- Index Page 는 단일 Page (Page 100) 로써 사용되며, Data Page 에 Write 된 변수들의 ID, Addr, CRC, Data 를 Read/Wirte 하는 목적으로 사용된다.
- Data Page 는 다수의 Page 로 사용되며 (Page 0 ~ 99), 주행거리를 적산 하는 목적으로 사용된다.
- Background Erase 기능을 구현하여, System Boot 시에만 메모리를 Erase 하여 주행 시스템에 영향이 없게 하였다.
- Background Erase 의 조건은 현재 작성하고 있는 Page의 다음 Page 가 비어있지 않을 때 Erase 하게 된다. 
- Read 된 Data 는 CRC 를 통해 유효성 검증을 한다. 이는 Diag를 통해 상위에게 전달한다.
- Flash Memory 의 크기는 기존 STM32F7 과는 다르게 BANK2 (Page0 ~ Page100) 을 운용하며 각 Page 는 각각 2KB 의 크기를 가진다. 따라서 Data Page (200KB), Index Page (2KB) 의 크기를 갖는다.
- Data Page (200KB) 는 64Bit Write 기준으로 약 2,5000 * 10000 의 Write 수명을 갖는다.
- Index Page (2KB) 는 64Bit Write 기준으로 약 250 * 10000 의 Write 수명을 갖는다.


### 시험 결과
==**Index Page**==
- Index 작성 중인 모습

	![[Pasted image 20240206103746.png]]
	![[Pasted image 20240206103732.png]]
 

- Index Page 모두 작성 뒤, Background Erase 이후 작성 중인 모습

	- 모두 작성 된 모습
	![[Pasted image 20240206104511.png]]
	![[Pasted image 20240206104554.png]]
	
	- Background Erase 이후
	![[Pasted image 20240206104751.png]]
	![[Pasted image 20240206104811.png]]

==Data Page==
- Data Page 를 사용중인 모습 (0Page 252번)
	![[Pasted image 20240206105005.png]]
	![[Pasted image 20240206105116.png]]

- Data Page 를 모두 작성 뒤, 다음 Page 로 넘어가 작성 중인 모습 (1Page 3번)
	 ![[Pasted image 20240206105257.png]]
	 ![[Pasted image 20240206105311.png]]

 