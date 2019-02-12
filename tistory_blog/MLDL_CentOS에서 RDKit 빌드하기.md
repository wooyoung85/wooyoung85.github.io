> CentOS에서 RDKit을 사용하기 위해 RDKit 소스코드를 빌드하는 과정을 정리해 보았습니다.  

# RDKit이란?
**RDKit is a <span style="color:red">collection of cheminformatics</span><sup>(*)</sup> and <span style="color:red">machine-learning software</span> written in C++ and Python.**

- Core data structures 과 algorithms은 C++로 만들어져 있음
- Python 3.x wrapper는 Boost.Python를 사용하여 만듬 
- Java나 C# wrapper는 SWIG을 활용하여 만듬

><sup>(*)</sup> 화학정보학 ( chemoinformatics, chemioinformatics, chemical informatics)은 컴퓨터 및 정보 기술를 적용하여 화학분야의 다양한 문제들에 대하여 적용하는 학문  
[출처 : 위키백과](https://ko.wikipedia.org/wiki/%ED%99%94%ED%95%99%EC%A0%95%EB%B3%B4%ED%95%99)

# RDKit Install
[RDKit Document](https://www.rdkit.org/docs/Index.html)에서 RDKit 설치 방법에 대해 자세히 설명하고 있음
> 가장 편하고 빠르게 RDKit을 사용하려면 **Anaconda 환경에서 RDKit을 설치 후 사용**하면 되지만 Anaconda를 사용하지 않는다면 운영체제 별로 알아서 잘 설치해야 함 ㅠ.ㅠ

## CentOS RDKit Install
-  Fedora, CentOS, and RHEL OS는 공식 Fedora Repositories에 binary 파일이 있다고 나와 있음 ([RDKit Document](https://www.rdkit.org/docs/Install.html#fedora-centos-and-rhel) 참고)
- Fedora Packages 사이트에서 [rdkit 패키지](https://apps.fedoraproject.org/packages/s/rdkit)를 찾긴 했지만 ~~설치는 실패했다 ㅠ.ㅠ~~

#### 그래서 RDKit을 직접 빌드해 보기로 했다 :)

# CentOS에서 RDKit 빌드하기
- Virtual Box에 CentOS를 설치해서 빌드 작업 진행  
([Virtual Box 설정 참고](https://wooyoung85.tistory.com/35))
- 아래 작업은 root 계정을 사용하여 진행함
- 빠른 빌드를 위해 가상머신의 cpu는 4개로 설정 

## TL;DR
```shell
# 기본 Package 설치
$> yum groupinstall -y "Development Tools"
$> yum installm -y wget vim eigen3-devel

# Software Collections 설치 및 활성화
$> yum --enablerepo=extras install centos-release-scl
$> yum update
$> yum install -y devtoolset-7
$> yum install -y rh-python36 rh-python36-python-devel
$> scl enable rh-python36 devtoolset-7 bash

# CMake 설치
$> curl -L -O https://cmake.org/files/v3.10/cmake-3.10.2.tar.gz
$> tar -xvzf cmake-3.10.2.tar.gz
$> cd cmake-3.10.2
$> ./bootstrap
$> make
$> make install

# Boost 설치
$> curl -L -O https://dl.bintray.com/boostorg/release/1.65.1/source/boost_1_65_1.tar.gz
$> tar -xzvf boost_1_65_1.tar.gz
$> cd boost_1_65_1
$> ./bootstrap.sh --with-python=python3.6 --with-libraries=python,serialization 
$> vim project-config.jam
   ### Python 관련 설정 수정 ###
   ...
   import python ;
   if ! [ python.configured ]
   {
       #using python : 3.6 : /usr/bin/python3.6 ;
       using python : 3.6 : /opt/rh/rh-python36/root/usr/bin/python3.6 : /opt/rh/rh-python36/root/usr/include/python3.6m : /opt/rh/rh-python36/root/usr/lib64 ;
   }
   ...
$> ./b2 install

# 환경 변수 설정
$> vim ~/.bash_profile
   ### Boost 관련 환경 변수 추가 ###
   export BOOST_INCLUDEDIR=/root/boost_1_65_1
   export BOOST_ROOT=/root/boost_1_65_1
   export BOOST_LIBRARYDIR=/root/boost_1_65_1/stage/lib
$> source ~/.bash_profile

# RDKit 빌드하기
$> curl -L -O https://github.com/rdkit/rdkit/archive/Release_2018_09_1.tar.gz
$> tar -xvzf Release_2018_09_1.tar.gz
$> mv rdkit-Release_2018_09_1/ RDKit
$> vim ~/.bash_profile
   ### RDKit 빌드 관련 환경 변수 추가
   RDBASE=/root/RDKit
   ...
   export RDBASE
   export LD_LIBRARY_PATH=$RDBASE/lib:$LD_LIBRARY_PATH
   export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
   export PYTHONPATH=$RDBASE:$PYTHONPATH

$> cd RDKit
$> mkdir build
$> cd build
$> cmake ..
$> make
$> make install
```

## Software Collections 설치 및 활성화
- Developer Toolset 7<sup>(**)</sup> 설치  
- RDKit 소스코드가 g++ v4.8에서는 빌드되지 않기 때문에 7.X대 버전의 GCC를 사용하기 위해 Developer Toolset을 설치하였음 ([RDKit Document](https://www.rdkit.org/docs/Install.html#building-from-source) 참고)
    ```shell
    $> yum --enablerepo=extras install centos-release-scl
    $> yum update
    $> yum install -y devtoolset-7
    ```

- Python3.6 설치  
(Python 3.X 버전에서 RDKit 을 사용할 예정임)
    ```shell
    $> yum install -y rh-python36 rh-python36-python-devel
    ```
- Software Collections 활성화
    ```shell
    $> scl enable rh-python36 devtoolset-7 bash

    $> gcc --version
    gcc (GCC) 7.3.1 20180303 (Red Hat 7.3.1-5)
    Copyright (C) 2017 Free Software Foundation, Inc.
    This is free software; see the source for copying conditions.  There is NO
    warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

    $> python -V
    Python 3.6.3
    ```
- Software Collections 비활성화
    ```shell
    $> exit
    ```

>(**) [Developer Toolset 7](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-7/)  
    CentOS or Red Hat Enterprise Linux platform에서 작동하는 **개발자를 위한 도구 모음**.  
    현재 버전의 GCC(GNU Compiler Collection),  GNU Debugger와 개발, 디버깅, 성능 모니터링 Tool들을 제공함.

## CMake 설치
- CMake는 멀티 플랫폼을 위한 빌드 지원 시스템  
- CMake는 3.1 이상 버전을 사용해야 함 ([RDKit Document](https://www.rdkit.org/docs/Install.html#installing-prerequisites-from-source) 참고)
    ```shell
    # 3.10 버전 다운로드
    $> curl -L -O https://cmake.org/files/v3.10/cmake-3.10.2.tar.gz
    $> tar -xvzf cmake-3.10.2.tar.gz
    $> cd cmake-3.10.2
    $> ./bootstrap
    $> make -j 4
    $> make install
    ```

## Boost 설치
- C++ 프로그래밍 언어를 위한 라이브러리들의 집합
- Boost 1.58 이상 버전을 사용해야 하고, python과 serialization 라이브러리를 포함하는 boost-devel 패키지를 가져야 한다고 나와있음 ([RDKit Document](https://www.rdkit.org/docs/Install.html#installing-boost) 참고)
- 이 Post에서는 Boost 패키지를 다운받는 것이 아니라 소스코드를 직접 빌드함

- Boost 1.65 버전 다운로드

    ```shell
    $> curl -L -O https://dl.bintray.com/boostorg/release/1.65.1/source/boost_1_65_1.tar.gz
    $> tar -xzvf boost_1_65_1.tar.gz
    ```
- `bootstrap.sh` 실행  
(RDKit은 Boost.Python을 활용하여 Python 3.x wrapper 을 만들었기 때문에 `bootstrap.sh` 실행 시 Python 버전과 포함되어야 하는 라이브러리를 아래와 같이 꼭 지정해야 함)
    ```shell
    $> cd boost_1_65_1
    $> ./bootstrap.sh --with-python=python3.6 --with-libraries=python,serialization 
    ```
- `project-config.jam` 파일 수정  
(Install 시 사용할 Python에 대해 설정해 줌)
    ```shell
    $> vim project-config.jam
    ```

    ```shell
    ...
    import python ;
    if ! [ python.configured ]
    {
        #using python : 3.6 : /usr/bin/python3.6 ;
        using python : 3.6 : /opt/rh/rh-python36/root/usr/bin/python3.6 : /opt/rh/rh-python36/root/usr/include/python3.6m : /opt/rh/rh-python36/root/usr/lib64 ;
    }
    ...
    ```
- Install
    ```shell
    $> ./b2 install
    ```
- 성공적으로 Install 작업이 완료되면 바이너리 파일이 `/usr/local/lib` 밑으로 떨어짐

## 환경 변수 설정
- `.bash_profile` 수정
    ```shell
    $> vim ~/.bash_profile
    ```
    ```shell
    export BOOST_INCLUDEDIR=/root/boost_1_65_1
    export BOOST_ROOT=/root/boost_1_65_1
    export BOOST_LIBRARYDIR=/root/boost_1_65_1/stage/lib
    ```
- `.bash_profile` 적용
    ```shell
    $> source ~/.bash_profile
    ```

## RDKit 빌드하기
- RDKit 다운로드
    ```shell
    $> curl -L -O https://github.com/rdkit/rdkit/archive/Release_2018_09_1.tar.gz
    $> tar -xvzf Release_2018_09_1.tar.gz

    # 폴더명 변경
    $> mv rdkit-Release_2018_09_1/ RDKit
    ```
    
- RDKit 빌드 관련 환경 변수 설정
    ```shell
    $> vim ~/.bash_profile
    ```
    ```shell
    RDBASE=/root/RDKit
    ...
    export RDBASE
    export LD_LIBRARY_PATH=$RDBASE/lib:$LD_LIBRARY_PATH
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
    export PYTHONPATH=$RDBASE:$PYTHONPATH
    ```
- RDKit 빌드하기
    ```shell
    $> cd RDKit
    $> mkdir build
    $> cd build
    $> cmake ..
    $> make -j 4
    $> make install
    ```

## Python 패키지에 RDKit 추가하기
- Python 패키지 경로 확인
    ```shell
    $> python -m site
    sys.path = [
        '/usr/local/lib',
        '/opt/rh/devtoolset-7/root/usr/lib64/python2.7/site-packages',
        '/opt/rh/devtoolset-7/root/usr/lib/python2.7/site-packages',
        '/root/RDKit',
        '/opt/rh/rh-python36/root/usr/lib64/python36.zip',
        '/opt/rh/rh-python36/root/usr/lib64/python3.6',
        '/opt/rh/rh-python36/root/usr/lib64/python3.6/lib-dynload',
        '/opt/rh/rh-python36/root/usr/lib64/python3.6/site-packages',
        '/opt/rh/rh-python36/root/usr/lib/python3.6/site-packages',
    ]
    USER_BASE: '/root/.local' (doesn't exist)
    USER_SITE: '/root/.local/lib/python3.6/site-packages' (doesn't exist)
    ENABLE_USER_SITE: True
    ```
- rdkit 폴더(빌드 결과물) 복사
    ```shell
    $> cp -rf ~/RDKit/rdkit /opt/rh/rh-python36/root/usr/lib64/python3.6/site-packages
    ```

# Getting Started with the RDKit in Python
- rdkit import 하기  
(import만 잘 되면 성공~!!)
    ```python
    >>> from rdkit import Chem
    ```
- Smiles 코드 다른 형태로 변환하기
    ```shell
    >>> m = Chem.MolFromSmiles('C1CCC1')
    >>> print(Chem.MolToMolBlock(m))    

        RDKit          2D

    4  4  0  0  0  0  0  0  0  0999 V2000
        1.0607    0.0000    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    -0.0000   -1.0607    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    -1.0607    0.0000    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
        0.0000    1.0607    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    1  2  1  0
    2  3  1  0
    3  4  1  0
    4  1  1  0
    M  END
    
    >>> from rdkit.Chem import Draw
    >>> Draw.MolToFile(m,'test1.png')
    ```

## 참고자료
[Installation - RDKit Document](https://www.rdkit.org/docs/Install.html)  
[Getting Started with the RDKit in Python - RDKit Document](http://www.rdkit.org/docs/GettingStartedInPython.html)  
[Developer Toolset 7 - Software Collections](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-7/)  
[How to install Python 3 on Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/13/install-python3-rhel/)  
[Configuring Boost.Build](https://www.boost.org/doc/libs/1_65_1/libs/python/doc/html/building/configuring_boost_build.html)