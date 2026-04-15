# 24. ROS2의 패키지 파일

## 24.1 패키지 파일

패키지 파일은 설정 파일 package.xml, 빌드 설정 파일 CMakeLists.txt, 파이썬 패키지 설정 파일 setup.py, 파이썬 패키지 환경 설정 파일 setup.cfg, RQt 플러그인 설정 파일 plugin.xml, 패키지 변경 로그 파일 CHANGELOG.rst, 라이선스 파일 LICENSE, 패키지 설명 파일 README.md등으로 구성 됩니다.

## 24.2 패키지 설정 파일 (package.xml)

모든 ROS 패키지는 xml 형식의 패키지 설정 파일(package.xml)을 반드시 1개 포함합니다. 이 파일은 ROS 패키지의 필수 구성 요소로, 패키지 이름, 작성자, 라이선스, 의존성 패키지 등 패키지 정보를 기술합니다.

<?xml> xml 버전 1.0 로 문서의 문법을 정의
**<package>** 이 구문부터 맨 끝의 </package>까지가 ROS 패키지 설정 부분입니다. 세부 사항으로 format="3" 이라고 패키지 설정 파일의 버전을 기재합니다. ROS 2는 3을 사용합니다.
<name> 패키지의 이름
<version> 패키지의 버전
<description> 패키지의 간단한 설명
<maintainer> 패키지 관리자의 이름과 이메일 주소
<license> 라이선스 (Apache 2.0, BSD, MIT, Boost Software License, GPLv2, GPLv3, LGPLv2.1, LGPLv3, Proprietary)
<url> 패키지를 설명하는 웹 페이지 또는 버그 관리, 소스 코드 저장소 등의 주소
<author> 패키지 개발에 참여한 개발자의 이름과 이메일 주소
**<buildtool_depend>** 빌드 툴의 의존성
**<build_depend>** 패키지를 빌드할 때 필요한 의존 패키지 이름
<exec_depend> 패키지를 실행할 때 필요한 의존 패키지 이름
**<test_depend>** 패키지를 테스트할 때 필요한 의존 패키지 이름


## 24.3 빌드 설정 파일(CMakeList.txt)
ROS 2의 빌드 시스템인 ament에서는 C++ 프로그래밍 언어를 사용한 패키지나 RQt Plugin의 경우 CMake(Cross Platform Make)를 이용합니다. 패키지 폴더의 CMakeLists.txt라는 파일에 빌드 환경을 기술하여 사용하고 있습니다. 이 빌드 설정 파일에서는 실행 파일 생성, 의존성 패키지 우선 빌드, 링크 생성 등을 설정합니다.
다양한 운영체제, IDE 지원. 
인터페이스를 추가하거나, 의존성 패키지를 기술하게 된다. 경로 표기.
설치 옵션과 빌드 옵션. 특정 폴더에 대한 기술. 실행 파일에 대한 경로. 
빌드해야할 코드의 의존성 여부, 라이브러리화 시키는 경우 등.

## 24.4 파이썬 패키지 설정 파일(setup.py)
ROS 2 Python 패키지에서는 CMakeLists.txt 파일 대신 setup.py 파일을 사용합니다. 이 파일은 ROS 2 C++ 패키지의 CMakeLists.txt와 package.xml의 역할을 합니다. setup.py 파일은 setuptools을 사용하여 다양한 배포를 위한 설정을 하게 됩니다. 그러나 package.xml 파일은 ROS 패키지의 필수 구성 요소이므로, 비슷한 내용을 포함하더라도 패키지에 반드시 포함되어야 합니다.

name: 패키지 이름을 기입합니다.
version: 패키지 버전을 기입합니다.
packages: 의존하는 패키지를 하나씩 나열해도 되지만, find_packages()를 사용하면 자동으로 의존하는 패키지를 찾아줍니다.
data_files: 이 패키지에서 사용되는 파일들을 함께 배포하기 위해 기입합니다.
install_requires: 이 패키지를 설치할 때 함께 설치할 패키지를 기입합니다.
tests_require: 이 패키지를 테스트하는데 필요한 패키지를 기입합니다. ROS에서는 pytest를 사용합니다.
zip_safe: 설치할 때 zip 파일로 아카이브할지 여부를 설정합니다.
keywords: 이 패키지의 키워드를 기입합니다. Python Package Index (PyPI) [8] 에서 검색하여 이 패키지를 찾을 수 있도록 합니다.
author, author_email, maintainer, maintainer_email: 저작자와 관리자의 이름과 이메일을 기입합니다.
description: 패키지 설명을 기입합니다.
license: 라이선스 종류를 기입합니다.
entry_points: 플랫폼 별로 콘솔 스크립트를 설치하도록 콘솔 스크립트 이름과 호출 함수를 기입합니다.

## 24.5 파이썬 패키지 환경설정파일(setup.cfg)
순수한 ROS2 파이썬 패키지에서만 사용하는 배포를 위한 구성 파일. setup.py의 setup함수에서 설정하지 못하는 기타 옵션을 이 구성 파일에서 정의. develop과 install 옵션을 설정하여 스크립트의 저장 위치 설정. 

## 24.6 RQt 플러그인 설정 파일(plugin.xml)
RQT 플러그인으로 패키지를 작성할 때의 필수 구성 요소로 XML 태그로 각 속성을 기술하는 파일. 

## 24.7 패키지 변경로그 파일(CHANGELOG.rst)
패키지의 업데이트 내역을 기술. 간단한 문법을 사용한다. 패키지 명과 버전, 업데이트된 날짜와 코드 작업 내용, 작업자 기술한다. 
바이너리 패키지 공개 시 필수로 포함되어야 하며, 사용자들에게 변경 사항을 버전별로 공지할 수 있게 된다.

## 24.8 라이선스 파일(LICENSE)
패키지 코드에 사용된 라이선스를 기술하는 파일. 단일 라이선스의 경우 확장자 없이, 복수 라이센스의 경우 폴더 생성 후 파일명으로 구분.

## 24.9 패키지 설명 파일(README.md)
패키지의 부가 설명을 기술하는 파일로 필수 파일은 아니지만 사용자를 배려하는 문서. 
마크다운 문법을 따르며 개발 환경, 의존성 패키지, 설치 방법, 사용 방법등을 기재. 자세한 내용을 다룬 매뉴얼이 있으면 링크로 달아준다.
