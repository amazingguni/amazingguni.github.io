---
layout: post
title: 데브옵스 4장 - 디스크 문제 해결하기
excerpt: "데브옵스"
tags: [devops, 데브옵스, 디스크, disk]
comments: true
---

위키북스에서 출판한 데브옵스 4장 `왜 디스크에 쓸 수 없는가? 용량이 가득 찼거나 오류가 생긴 디스크 문제 해결하기` 를 읽고 정리한 포스트입니다.

### Overview

하드 디스크는 리눅스 서버에서 가장 많은 문제의 원인으로 지목됩니다. 

* 서버가 동작하지 않게 하는 주요 원인
* 어플리케이션의 주요 병목 중 하나
* 여유 공간이 없는 디스크와 오류가 난 파일 시스템

파일 시스템 오류 문제는 항상 사용자 혹은 프로그램이 디스크에 데이터를 쓸 수 없다 라는 증상을 보이며 시작됩니다.

### 1. 디스크가 가득 찼을 때

디스크가 가득 차면 리눅스는 아래와 같은 메시지를 보여줍니다.

```sh
$ cp /var/log/syslog syslogbackup
cp: writing `syslogbackup`: No space left on device
```

문제 해결은 아래와 같이 할 수 있을 것 입니다.

1. `df` 를 사용해 마운트된 모든 파티션의 크기, 사용 중인 공간, 사용 가능한 공간에 대한 정보를 확인합니다.
	
	```sh
	$ df -h   # -h 옵션은 읽기 편하게 Size를 변환해줍니다.
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/sda1       7.8G  7.4G   60K 100% /
	none            245M  192K  245M   1% /dev
	none            249M    0K  245M   0% /dev/shm
	none            249M   36K  245M   1% /var/run
	none            249M    0K  245M   0% /var/lock
	none            249M    0K  245M   0% /lib/init/rw
	```

	마운트된 */dev/sda1* 이 가득차서 사용 가능한 공간이 60kb만 있습니다. 전체 7.8G 중 7.4G를 사용했다면 4G정도는 남는게 정상인데, 그렇지 않은 이유는 `예약 블록`으로 5%정도의 공간이 할당되었기 때문입니다. 

	여기서 *예약블록* 이란 리눅스가 긴급 상황(그리고 단편화를 방지하기 위해)을 위해 따로 떼어 놓은 파일 시스템의 블록입니다. 
	
	*tune2fs* 도구를 통해 *예약블록* 의 상태를 확인 할 수 있습니다.
		
	```sh
	$ sudo tune2fs -l /dev/sda1 | grep -i "block count"
	Block count:          2073344
	Reserved block count: 103667
	```

	루트 사용자만이 이 공간을 쓸 수 있기 때문에 가득 찻다는 메시지가 뜬 이후에도 루트 사용자는 일부 파일을 옮기는 여유 공간을 확보할 수 있습니다. 
	
2. 가장 큰 디렉토리 추적하기
	
	`du` 명령어로 각 디렉토리가 차지하는 디스크 공간을 확인할 수 있습니다.

	```sh
	# 자주 볼 수 있기 때문에 임시 파일에 저장한다.
 	# 디렉토리 공간을 정렬하고 파일에 쓴다	
	$ cd /
	$ sudo du -ckx | sort -n > /tmp/duck-root
	```
	
	이후 tail 명령어로 가장 많이 용량을 차지하는 상위 10개의 디렉토리를 확인할 수 있습니다.
	
	```sh
	$ tail /tmp/duck-root
	67872         /lib/modules/2.6.24-19-server
	67876         /lib/modules	
	69092         /var/cache/apt
	69448         /var/cache
	76924         /usr/share
	82832         /lib
	124164        /usr
	404168        /
	404168        total
	```
	
	여기서 올라가면서 제거할 수 있는 디렉토리를 확인하는 것이 좋습니다.

	> /usr는 일반적으로 지우면 안될 것이다.

3. 로그 파일을 줄여본다.

	로그파일로 인해 여유 공간이 빠르게 사라지는 경우가 있습니다.  
	*/var/log* 에서 *sudo ls -lS* 로 크기별로 로그의 크기를 확인해보고 아래 명령어로 내용을 지울 수 있습니다.

	```sh
	$ sudo sh -c "> /var/log/messages"
	```
	
	압축되지 않은 로그 때문에 자주 이런 문제가 발생한다면 */etc/logrotate.conf* 와 */etc/logrotate.d/* 안의 로그 순환 설정을 수정해서 자동으로 압축할 수 있습니다.

	> 설정법에 대한 자세한 설명은 [이런 저런 이야기 :: logrotate 사용법(로그 세대관리)](http://culturescrap.tistory.com/entry/logrotate-%EC%82%AC%EC%9A%A9%EB%B2%95%EB%A1%9C%EA%B7%B8-%EC%84%B8%EB%8C%80%EA%B4%80%EB%A6%AC) 를 참조바랍니다.

4. 대용량의 *.swp*

	큰 로그를 열면서 생성되는 *.swp* 파일로 인해 디스크가 가득 찰 경우도 있다.


### 2. inode가 고갈된 경우

*df* 를 실행했을 때는 여유 공간이 있지만 공간이 없다는 에러메시지가 나오는 경우입니다.

이런 경우 `inode`가 고갈됬을 수 있습니다. 일반적인 일은 아니지만 `inode`가 고갈되면 새로운 파일이 생성되지 않습니다.

> inode는 파일에 대한 정보가 저장되어 있는 데이터 구조다. 각각의 파일은 유일한 inode를 얻는다. 

*df -i* 명령어로 inode 사용량에 대한 정보를 확인할 수 있습니다.

```sh
$ df -i
Filesystem       Inodes  IUsed    IFree IUse% Mounted on
/dev/sda1        520192  17539   502653    4% /
```

IUse%가 100%라면(그럴일은 거의 없지만) 파일들을 옮기거나 압축을 통해 파일 갯수를 줄여서 문제를 해결할 수 있습니다.

### 3. 손상된 파일 시스템 고치기

리눅스는 파일 시스템 복구를 위해 부팅 시점에 파일 시스템 체크 명령(`fsck`)를 실행합니다. 이로 문제는 해결하지 못할 경우 복구 shell로 빠지게 됩니다.

1. 수동으로 `fsck`를 실행하기 전에는 반드시 파일 시스템을 먼저 내려야 합니다.
	
	fsck는 잠재적으로 파일 시스템을 더 손상시킬 수 있기 때문

	```sh
	$ mount
	$ unmount <devicename> # 반복..
	```

2. `fsck` 실행
	
	파일 시스템 */dev/sda5* 에서 발생한 오류를 보고 이를 고치려면 아래와 같이 입력합니다.

	```sh
	$ fsck -y -C /dev/sda5
	```

	*-y 옵션* 은 자동으로 yes라고 답하기 위해, *-C 옵션* 은 진행 표시줄을 얻기 위해 사용합니다.

### 4. 소프트웨어 RAID 고치기
	
리눅스 소프트웨어 RAID를 사용하는 시스템을 운영하고 있다면 알아 두는 것이 좋습니다.

> mdadm은 소프트웨어 RAID 장치들을 관리하기 위한 리눅스 유틸리티이며 자세한 내용은 [mdadm 명령을 사용하여 RAID 기반, 멀티패스 스토리지를 설정하는 방법](http://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-ig-s390-multi-ko-4/s1-s390info-raid.html) 에 잘 설명되어 있습니다.

1. RAID가 고장 났을 때 감지하는 방법

	일반적으로는 문제가 발생할 경우 루트 사용자에게 email이 발송되도록 mdadm 설정이 되어 있어야 합니다.

	> [How To Configure mdadm RAID e-mail Notifications via ISP SMTP Using msmtp](http://ubuntuforums.org/showthread.php?t=1185134) 를 참조

	그렇지 않은 경우 `/proc/mdstat` 파일로 문제를 확인할 수 있습니다.

	```sh
	$ cat /proc/mdstat
	Personalities : [linear] [multipath] [raid0] [raid1] [raid6]
	[raid5] [raid4] [raid10]
	md0 : active raid5 sdb1[0] sdd1[3](F) sdc1[1]
	16771584 blocks level 5, 64k chunk, algorithm 2 [3/2] [UU_]
	unused devices: <none>
	```
	
	여기서 `sdd1`이 고장 났기 때문에 `(F)`로 표시되어 있습니다.

2. */dev/md0* 에서 디스크를 제거
	
	*/dev/md0* 는 /etc/mdadm.conf에서 정의한 ARRAY 값입니다(RAID 장치를 의미).

	```sh
	# 삭제를 위해 --remove 옵션 사용
	# 결함이 있다고 설정하기 위해 --fail 옵션 사용
	$ sudo mdadm /dev/md0 --fail /dev/sdd1 --remove /dev/sdd1
	```
	
	삭제 이후 `/proc/mdstat`을 확인하면 드라이브가 빠진것을 볼 수 잇습니다.

	```sh
	$ cat /proc/mdstat
	Personalities : [linear] [multipath] [raid0] [raid1] [raid6]
	[raid5] [raid4] [raid10]
	md0 : active raid5 sdb1[0] sdc1[1]
	16771584 blocks level 5, 64k chunk, algorithm 2 [3/2] [UU_]
	unused devices: <none>
	```

3. 시스템 전원을 내리고 하드디스크 교체(핫스왑을 지원한다면 곧바로 교체)

	파티션이 준비되면 RAID 배열에 `--add` 명령으로 새 파티션을 추가합니다.

	> 디스크를 교체할 때는 반드시 RAID 배열 안에(ARRAY) 나머지 파티션과 동일하거나 더 큰 크기로 새 파티션을 만들어줍니다.

	```sh
	$ sudo mdadm /dev/md0 --add /dev/sdd1
	```
	
	명령어 이후 `mdadm`이 데이터를 다시 동기화하는 과정을 시작하며 `/proc/mdstat`을 통해 진행 상태를 모니터할 수 있다.

	```sh
	# 5초마다 명령을 실행하기 위해 watch 명령어 사용.
	$ watch -n 5 "cat /proc/mdstat"
	Personalities : [linear] [multipath] [raid0] [raid1] [raid6]
	[raid5] [raid4] [raid10]
	md0 : active raid5 sdd1[3] sdb1[0] sdc1[1]
	16771584 blocks level 5, 64k chunk, algorithm 2 [3/2] [UU_]
	[>..................] recovery = 2.0% (170112/8385792)
	finish=1.6min speed=85056K/sec
	unused devices: <none>
	```
