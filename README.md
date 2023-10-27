# 1. 프로젝트 소개
### 1-1)프로젝트 명
#### ZNS를 이용한 키-밸류 스토어 성능 개선 연구

### 1-2)목적
#### 대표적인 키-밸류 스토어인 RocksDB를 차세대 저장 장치인 ZNS SSD의 특성에 맞게 사용하여 I/O 성능을 개선한다.

### 1-3)개요
#### RocksDB는 내장 파일시스템인 ZenFS를 사용하여 ZNS SSD를 지원한다. out-place-update의 특성을 가진 ZNS SSD를 위해 ZenFS는 여유 공간을 확보하기 위한 Zone Erase 정책과 Garbage Collection을 피하기 위한 Zone 할당 정책을 가지고 있다. 하지만 현 ZenFS의 Zone 할당 정책은 Zone Erase 정책을 잘 활용하지 못하고 있다. 따라서 본 프로젝트를 통해 ZenFS의 Zone Erase 정책을 잘 활용하도록 Zone 할당 정책을 개선하고 그로 인해 발생한 추가적 문제를 해결한다.

# 2. 팀 소개 System_A

* 배재홍 bjh0820@naver.com , RocksDB 코드 분석 및 동작분석, Zone Erase정책 고안 및 구현

* 이재석 jaeseok91@naver.com , RocksDB 코드 분석 및 동작분석

* 조준호 adagazua@pusan.ac.kr, ZenFS 코드 분석 및 동작분석

# 3. 구성도
### [시스템 구성도]
![ZenFS](https://github.com/pnucse-capstone/capstone-2023-1-46/assets/81742357/00367071-c2aa-4a59-a629-3f562c4b4cbf)

* RocksDB는 내장 파일시스템인 ZenFS를 사용하여 ZNS SSD 지원
* ZenFS에는 Zone을 관리하기 위한 정책 존재
<br/>

### [Zone 할당 정책]
![Zone 할당 정책](https://github.com/pnucse-capstone/capstone-2023-1-46/assets/81742357/a86d2982-62e6-487d-89e5-4e50969b6f7c)
* 좌: 현 ZenFS의 Zone 할당 정책 - Zone에 처음 write된 파일의 Lifetime보다 작은 Lifetime을 가진 파일들이 write
* 우: Strong Lifetime based Allocation(SLA) - 같은 Zone에 같은 Lifetime인 파일만 write
* ZenFS의 Zone Erase 정책 - Zone의 모든 데이터가 Invalid되면 Erase
* 현 ZenFS는 높은 Lifetime의 파일에 의해 Zone Erase 정책을 잘 활용하지 못하고 Invalid 데이터가 많이 남음
* SLA는 한 Zone에 파일들의 Lifetime이 같으므로 Zone Erase 정책을 잘 활용하고 여유 공간 확보
<br/>

### [Zone Erase 정책]
![Zone Erase 정책](https://github.com/pnucse-capstone/capstone-2023-1-46/assets/81742357/edd81566-970d-4d73-bcbf-aba6b5891c9c)
* 좌: 현 ZenFS의 Zone Erase 정책 - Zone의 모든 데이터가 Invalid이면 Zone이 가득 차있지 않더라도 Erase
* 우: Lazy Zone Reset(LZR) - Lifetime이 2인 데이터만 존재하는 Zone에 대해서 모든 데이터가 Invalid이고 Zone이 가득 차면 Erase
* SLA를 사용하면 Lifetime이 2인 데이터들이 자주 지워져 Zone Erase 수가 증가해 ZNS SSD의 수명에 악영향
* Lifetime이 2인 파일이 저장된 Zone의 Erase를 늦춰 해결

# 4. 소개 및 시연 영상
### [소개 영상]
[![System_A 소개영상](http://img.youtube.com/vi/WlXwOGqMEa8/0.jpg)](https://www.youtube.com/watch?v=WlXwOGqMEa8) 
<br/>

### [시연 영상]

* used_zone: 사용 중인 Zone의 수
* #200~255 zone: 일부 Zone의 데이터 현황
  * -: Invalid Data
  * #: Valid Data
* total migration: GC로 인해 발생한 migration 양 byte (MB)
* lifetime n erase count: lifetime이 n인 Zone의 erase 수
* total erase count: 총 erase 된 Zone의 수
<br/>
[![Default](http://img.youtube.com/vi/mRMwI-QpY38/0.jpg)](https://www.youtube.com/watch?v=mRMwI-QpY38) 
#### [Default]

[![Default](http://img.youtube.com/vi/ownhIZsZFDA/0.jpg)](https://www.youtube.com/watch?v=ownhIZsZFDA) 
#### [SLA]

[![SLA-LZR](http://img.youtube.com/vi/G5OP8r8xZjY/0.jpg)](https://www.youtube.com/watch?v=G5OP8r8xZjY) 
#### [SLA-LZR]


# 5. 사용법
### [Linux]
#### Dependencies
* zlib - a library for data compression.
* bzip2 - a library for data compression.
* lz4 - a library for extremely fast data compression.
* snappy - a library for fast data compression.
* zstandard - Fast real-time compression algorithm.
* gflags - a library that handles command line flags processing. You can compile rocksdb library even if you don't have gflags installed.
  
##### Install gflags:
```
$ sudo apt install libgflags-dev -y
```
##### Install snappy
```
$ sudo apt install libsnappy-dev -y
```

##### Install zlib
```
$ sudo apt install zlib1g-dev -y
```

##### Install bzip2
```
$ sudo apt install libbz2-dev -y
```

##### Install lz4
```
$ sudo apt install liblz4-dev -y
```

##### Install zstandard
```
$ sudo apt install libzstd-dev -y
```

##### Install libzbd
```
$ sudo git clone https://github.com/westerndigitalcorporation/libzbd.git
$ cd libzbd
$ sudo sh ./autogen.sh
$ sudo ./configure
$ sudo make
$ sudo make install
```

#### RocksDB & ZenFS
##### CPU core 수 확인
```
$ grep -c processor /proc/cpuinfo 
```
#### RocksDB & ZenFS 설치
```
$ sudo git clone https://github.com/pnucse-capstone/capstone-2023-1-46.git
$ cd capstone-2023-1-46/rocksdb
$ sudo DEBUG_LEVEL=0 ROCKSDB_PLUGINS=zenfs make -j(코어 수) db_bench install
$ pushd
$ cd plugin/zenfs/util
$ make
$ popd
```
<br/>

### [Windows]
* For building with MS Visual Studio 13 you will need Update 4 installed.
* Read and follow the instructions at CMakeLists.txt
* Or install via [vcpkg](https://github.com/microsoft/vcpkg).
    * run vcpkg install rocksdb:x64-windows

### [참고 자료]
* [RocksDB](https://github.com/facebook/rocksdb/blob/master/INSTALL.md)
* [ZenFS](https://github.com/westerndigitalcorporation/zenfs)
* [RocksDB API](https://rocksdb.org/docs/getting-started.html)
* [RocksDB C++ API](https://github.com/facebook/rocksdb/tree/main/include/rocksdb)
* [RocksDB JAVA API](https://github.com/facebook/rocksdb/tree/main/java/src/main/java/org/rocksdb)
