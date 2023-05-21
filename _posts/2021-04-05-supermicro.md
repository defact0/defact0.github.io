---
title: "슈퍼마이크로 서버"
date: 2021-04-05
categories:
  - Server
tags:
  - RAID
---

> 어느 날 회사에 `2u서버`가 도착했다.  
> 서버실 랙에 `2u서버`를 밀어 넣고 OS를 설치를 시도했다…

![](/images/2021-04-05-supermicro/oh.jpg)

그런데 OS에는 파티션 설정을 해야 할 하드디스크가 보이지 않았다.  
제조사에서 부품만 장착하고 회사로 배송한 거 같아 보였다.

## **BIOS**

`supermicro` 에서 만든 서버이고, 아무런 설정도 하지 않은 상태이다.  
Server Booting 시, `[DEL]` 키를 눌러 아래 두 가지 설정을 먼저 해야 한다

- SSD => RAID 1 설정 필요
- Network Lan Card 설정 필요

### **BIOS > RAID 1 준비**

- 부팅 > `[DEL]` 키 입력
- PCH SATA Configuration > Configure SATA as > `RAID` 변경

![](/images/2021-04-05-supermicro/raid1_01.png)

### **BIOS > Network Lan Card 설정**

- PCIe/PCI/PnP Configuration > Onboard LAN 1 OPROM > `EFI`
- PCIe/PCI/PnP Configuration > Network Stack Configuration > Network Stack > `Disabled`
- `[F4]` 키 입력 후 `Yes` 선택하면 설정된 내용을 적용하고 `재부팅`을 하게 된다.

![](/images/2021-04-05-supermicro/raid1_02.png)

### **RAID 유틸리티 설정**

재부팅 중 `Ctrl + R` 키를 누르면 유틸리티에 들어간다.

- Virtual Drive Management 화면에서 `[ No Configuration Present ! ]` 항목에 엔터키 입력

![](/images/2021-04-05-supermicro/raid1_03.png)

- RAID 항목에서 엔터키를 입력하면 RAID 버전을 선택 할 수 있다.
- `RAID-1` 항목을 선택한다.

![](/images/2021-04-05-supermicro/raid1_04.png)

- Drives 항목에 SSD가 2개 있는데 엔터키를 누르면 [X] 표시가 생긴다.

![](/images/2021-04-05-supermicro/raid1_05.png)

- OK 버튼을 누르면 `Virtual Drive Management` 화면으로 이동하는데,`[F2]` 키를 입력 하면 서브메뉴가 활성화 된다.
- Initialization > `Start FGI` 선택

![](/images/2021-04-05-supermicro/raid1_06.png)

`[ESC]` 키 누르고 `YES` 선택하면 `ctrl + alt + del` 키를 누르면 된다.

### **이후..**

드디어 하드디스크가 활성화되어 OS 설치를 진행 할 수 있게 되었다.

---

## **LAN 연결 이슈**

인터넷 연결이 안되어 서버실 관리자 분께 물어보니,일부 모델은 gigabit ethernet 스위치에서 연결된 Lan Cable로 연결해야 한다고 한다.

일반 ethernet 스위치로 연결시 diconnectd 상태로 있었다.

![](/images/2021-04-05-supermicro/raid1_07.png)