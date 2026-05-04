## bitbake
```
$ bitbake <recipe file name>                    # 레시피의 실행
$ bitbake <recipe file name> -k                 # 레시피의 실행(실패해도 종료하지 않고 끝까지 수행)
$ bitbake <recipe file name> -e                 # 사용되는 환경 변수 출력
$ bitbake <recipe file name> -c <task name>     # 레시피의 해당 테스크 실행
$ bitbake <recipe file name> -c <task name> -f  # 레시피의 해당 테스크 강제 실행
$ bitbake <recipe file name> -C <task name>     # 레시피의 해당 Task부터 "강제로" 다시 실행
$ bitbake <recipe file name> --runall=<task>    # 레시피와 연관된 모든 해당 테스크 강제 실행
$ bitbake <recipe file name> -c cleanall        # 레시피와 연관된 결과물 삭제
$ bitbake-layers show-layers                    # layer 목록 출력
$ bitbake-getvar <variable name>                # 변수 값 출력
$ bitbake-layers show-appends                   # 모든 레시피 확장 목록 조회
$ bitbake-layers flatten <output directory>     # 여러 레이어에서 사용된 메타데이터들을 단일 계층 디렉토리화 하여 보여준다.

[devshell]
$ bitbake <recipe name> -c devshell
- build에 필요한 환경 변수까지 같이 load 됨
- kernel의 경우 devshell에서 bitbake 명령어 없이, make로 제어 가능

[u-boot]
<devshell>
$ bitbake -c devshell u-boot

[kernel]
<devshell>
$ bitbake virtual/kernel -c devshell

<search kernel source path>
$ bitbake-getvar -r <kernel-recipe name> STAGING_KERNEL_DIR
$ bitbake-getvar -r virtual/kernel STAGING_KERNEL_DIR

<menuconfig>
$ bitbake -c menuconfig <kernel-recipe name>
$ bitbake -c menuconfig virtual/kernel

<configure>
$ bitbake -c kernel_configme <kernel-recipe name>
$ bitbake -c kernel_configme virtual/kernel

<savedefconfig>
$ bitbake -c savedefconfig <kernel-recipe name>
$ bitbake -c savedefconfig virtual/kernel

<diffconfig>
$ bitbake -c diffconfig virtual/kernel

<.config 무결성을 검증>
$ bitbake -c kernel_configcheck -f virtual/kernel
- 비정상적인 옵션 탐지: 존재하지 않는 커널 옵션을 설정했는지 확인.
- 의존성 누락 확인: 특정 옵션을 활성화(=y)했지만, 그 옵션이 필요로 하는 상위 옵션이 꺼져 있어 실제 결과물(.config)에서 누락된 경우.
- 중복 및 충돌 탐지: 여러 설정 파일에서 동일한 옵션을 서로 다르게 정의했을 경우.

[example]
$ bitbake core-image-minimal
$ bitbake core-image-minimal -C rootfs
$ bitbake core-image-minimal -C helloworld

$ bitbake-layers show-appends | grep "core-image-minimal"


$ bitbake-layers flatten result_recipes
```

## runqemu
```
$ runqemu <recipe file name> <option>

[example]
$ runqemu core-image-minimal nogrphic
```

## Files
```
Type    | Name          | Role
-----------------------------------------------------------------------------------
conf    | Configuration | 빌드 환경에 대한 변수 및 정책 설정 (머신 종류, 배포판 정보 등)
bb      | Recipe        | 패키지를 어떻게 빌드할지 기술 (소스 위치, 컴파일 방법 등)
bbclass | Class         | 여러 레시피에서 공통적으로 사용하는 빌드 로직을 모듈화 (재사용 가능)
```

## conf file
```
[layer.conf]
LAYERDIR                        : layer의 최상위 디렉토리 경로
BBPATH                          : layer.conf 파일을 포함하는 layer의 경로를 저장
BBFILES                         : recipe 및 recipe 확장 파일들의 경로 저장
BBCOLLECTIONS                   : layer.conf 파일을 포함하고 있는 layer의 이름을 갖는다.
BBFILE_PATTERRN:<layer name>    : 모든 recipe 파일들의 경로를 갖고 있으며, 현재 layer에 속한 recipe들을 찾아 분석하는데 사용된다.

[bitbake.conf]
PN                          : package name
PV                          : package version
PR                          : package revision
TMPDIR = "${TOPDIR}/tmp"    : build output
CACHE  = "${TMPDIR}/cache"  : metadata 분석 결과
STAMP  = "${TMPDIR}/stamps" : 각 task 완료 시 생성되는 파일로 재빌드 여부를 결정 할 때 사용
T      = "${TMPDIR}/temp"   : 임시 생성 파일
B      = "${TMPDIR}/${PN}"  : recipe build 과정에서 함수를 실행하는 디렉토리를 가리킴.
S      = "${TMPDIR}/${PN}"  : 소스 코드가 있는 곳.
```

## Build Sequence
```
fetch --> unpack --> patch --> configure --> compile --> install
```

## Speed UP
```
# fetch 시간 절감
# Specify own PREMIRRORS location
INHERIT += "own-mirrors"
SOURCE_MIRROR_URL = "file://${HOME}/workspace/source-mirrors"

# git clone 대신 tar로 achive된 형태로 download
# compress tarballs for mirrors
BB_GENERATE_MIRROR_TARBALLS = "1"

# tmp 디렉토리를 삭제하고 rebuild 할 경우 build 시간 단축
# make shared state cache mirror
SSTATE_MIRRORS = "file://.* file://${COREBASE}/../sstate-cache/PATH"
SSTATE_DIR = "${TOPDIR}/sstate-cache"
```

## Message
```
Log Level | Python            | Shell
-----------------------------------------------
plain     | bb.plain(message) | bbplain message
debug     | bb.debug(message) | bbdebug message
note      | bb.note(message)  | bbnote message
warn      | bb.warn(message)  | bbwarn message
error     | bb.error(message) | bberror message
fetal     | bb.fetal(message) | bbfetal message
```

## Operator
```
Category         | Operator | Name               | 작동 Operation
------------------------------------------------------------------------------------------------------------
Basic Assignment | =        | Deferred           | 변수를 사용하는 시점에 값을 해석
                 | :=       | Immediate          | 변수를 선언하는 시점에 값을 고정
Append/Prepend   | +=       | Append (Space)     | 기존 값 뒤에 공백 하나를 두고 추가
                 | .=       | Append (No Space)  | 기존 값 뒤에 공백 없이 바로 붙임
                 | =+       | Prepend (Space)    | 기존 값 앞에 공백 하나를 두고 추가
                 | =.       | Prepend (No Space) | 기존 값 앞에 공백 없이 바로 붙임
Conditional      | ?=       | Soft Assignment    | 값이 설정되지 않았을 때만 대입
                 | ??=      | Weak Assignment    | 빌드 최종 단계까지 값이 없으면 대입
Overrides        | :append  | Override Append    | Override Append 파싱 마지막 단계에서 뒤에 공백 없이 강제 결합
                 | :prepend | Override Prepend   | 파싱 마지막 단계에서 앞에 공백 없이 강제 결합
                 | :remove  | Override Remove    | 해당 값이 포함되어 있으면 모두 삭제
```

## create & add layer
```
bitbake-layers create-layer ../layers/meta-hello
.
├── conf
│   └── layer.conf
├── COPYING.MIT
├── README
└── recipes-example
    └── example
        └── example_0.1.bb
bitbake-layers add-layer ../layers/meta-hello
```

## Variable
```
Variable        | Name                  | Description
----------------------------------------------------------------------------------------------------
THISDIR         | This Directory        | 현재 레시피(.bb) 파일이 위치한 경로를 의미합니다.
FILESEXTRAPATHS | Files Extra Paths     | SRC_URI에 명시된 파일을 찾을 때 추가로 검색할 경로 리스트입니다.
WORKDIR         | Working Directory     | 레시피 빌드를 위한 최상위 작업 공간입니다. (소스, 로그, 결과물 집결지)
S               | Source Directory      | 소스 코드가 압축 해제되거나 복사되어 놓이는 실제 경로입니다.
B               | Build Directory       | 컴파일 작업(Makefile 실행 등)이 실제로 수행되는 경로입니다.
D               | Destination Directory | do_install 시 파일이 임시로 설치되는 가상 루트(Root) 경로입니다.
PN              | Package Name          | 레시피의 이름을 나타냅니다. (파일명에서 버전 제외)
```

## License
```
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://<License File>;md5=<License File의 md5 값>"

LIC_FILES_CHKSUM 를 작성하길 원치 않을경우
LICENSE 를 "CLOSED" 로 정의해야한다.
```

## SRC_URI의 각 파일에 대한 checksum 설정
```
[여러 파일을 지정하는 경우]
SRC_URI = "\
    file://test1.c;name=test1 \
    file://test2.c;name=test2 \
    "
SRC_URI[test1.md5sum] = "<md5 value>"
SRC_URI[test2.md5sum] = "<md5 value>"
```

## Predefined Variable of Directory 
```
Variable Name | Destination
------------------------------
bindir        | /usr/bin
sbindir       | /usr/sbin
libdir        | /usr/lib
libexecdir    | /usr/lib
sysconfdir    | /etc
datadir       | /usr/share
mandir        | /usr/share/man
includedir    | /usr/include
```

## systemd
```
[conf]
# systemd configuration
DISTRO_FEATURES:append = " systemd"
DISTRO_FEATURES:remove = " sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = "systemd-compat-units"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscript = "systemd-compat-units"

[recipe]
inherit systemd
SYSTEMD_SERVICE:${PN} = "<service name>.service"
SYSTEMD_AUTO_ENABLE = "enable"
...
do_install() {
    ...

    # Install the systemd service file
    install -d ${D}${systemd_system_unitdir}
    install -m 0644 <service name>.service ${D}${systemd_system_unitdir}
}
...
FILES:${PN} += "${systemd_system_unitdir}/<service name>.service"

Variable                  | Basic Path                   |Description
---------------------------------------------------------------------------------------------------------------------
${systemd_unitdir}        | /lib/systemd                 | systemd의 최상위 유닛 디렉토리
${systemd_system_unitdir} | ${systemd_unitdir}/system    | 서비스 파일(.service), 타겟(.target) 등이 위치하는 핵심 경로
${systemd_user_unitdir}   | /usr/lib/systemd/user        | 사용자 세션용 유닛 파일 경로
${nonarch_libdir}         | /lib                         | 아키텍처에 종속되지 않는 라이브러리 경로 (systemd 유닛 설치 시 기반)
${sysconfdir}             | /etc                         | 시스템 설정 파일 경로
${systemd_system_confdir} | ${sysconfdir}/systemd/system | 관리자가 수정하는 시스템 서비스 설정/오버라이드 경로
```

## build history
```
[conf]
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"
BUILDHISTORY_COMMIT_AUTHOR = "JunKi Hong <dylanhong920509@gmail.com>"
BUILDHISTORY_DIR = "${TOPDIR}/buildhistory"
BUILDHISTORY_IMAGE_FILES = "/etc/passwd /etc/group"

Variable Name               | Default Value & Example        | Description
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
BUILDHISTORY_DIR            | ${TOPDIR}/buildhistory         | 빌드 기록 데이터가 저장되는 최상위 디렉토리 경로입니다. 이 디렉토리는 보통 Git 저장소로 초기화되어 관리됩니다.
BUILDHISTORY_COMMIT         | 0 (False)                      | 빌드가 끝날 때마다 변경 사항을 자동으로 Git 커밋할지 여부를 결정합니다. 1로 설정하면 매 빌드 완료 후 자동으로 커밋이 생성됩니다.
BUILDHISTORY_COMMIT_AUTHOR  | buildhistory <buildhistory@oe> | 자동 커밋 시 사용될 작성자(Author) 정보입니다. "이름 <이메일>" 형식으로 지정하여 누가(또는 어떤 시스템이) 기록했는지 식별합니다.
BUILDHISTORY_IMAGE_FILES    | /etc/passwd /etc/group         | 이미지 빌드 기록 시 추가로 캡처할 타겟 파일 리스트입니다. 기본적으로 설치된 패키지 목록 외에 특정 설정 파일의 변경점을 추적하고 싶을 때 공백으로 구분하여 추가합니다.

## buildhistory-diff
Changes to images/qemuarm64/glibc/core-image-minimal (files-in-image.txt):
  /lib/libudev.so.1.7.3 was added
  /lib/libudev.so.1 was added
  /lib/libusb-1.0.so.0.3.0 was added
  /lib/libusb-1.0.so.0 was added
  /usr/bin/lsusb.usbutils was added
  /usr/bin/lsusb was added
  /usr/bin/usb-devices was added
  /usr/bin/usbhid-dump was added
  /usr/lib/opkg/alternatives/lsusb was added
Changes to images/qemuarm64/glibc/core-image-minimal (installed-package-names.txt):
  libusb-1.0-0 was added
  usbutils was added
  libudev1 was added
```
## rm_work
```
<appends/nano_6.0.bbappend>:
inherit rm_work

<conf/layer.conf>:
...
BBFILES += "\
    ...
    ${LAYERDIR}/appends/*.bbappend \
    ${LAYERDIR}/appends/*/*.bbappend \
    "
...

[before]
# du -sh build/tmp/work/cortexa57-poky-linux/nano
3.3M    build/tmp/work/cortexa57-poky-linux/nano

[after]
# du -sh build/tmp/work/cortexa57-poky-linux/nano
380K    build/tmp/work/cortexa57-poky-linux/nano
```
## Find Source PATH
```
# bitbake-getvar -r nano S
WARNING: You are using a local hash equivalence server but have configured an sstate mirror. This will likely mean no sstate will match from the mirror. You may wish to disable the hash equivalence use (BB_HASHSERVE), or use a hash equivalence server alongside the sstate mirror.
#
# $S [2 operations]
#   set /home/dylan7h/workspace/yocto-practice/poky/meta/conf/bitbake.conf:405
#     "${WORKDIR}/${BP}"
#   set /home/dylan7h/workspace/yocto-practice/poky/meta/conf/documentation.conf:369
#     [doc] "The location in the Build Directory where unpacked package source code resides."
# pre-expansion value:
#   "${WORKDIR}/${BP}"
S="/home/dylan7h/workspace/yocto-practice/build/tmp/work/cortexa57-poky-linux/nano/6.0-r0/nano-6.0"
```


## externalsrc
```
※ Note:
- do_fetch, do_unpack, do_patch 테스크가 생략됨.

[appends/nano_6.0.bbappend]
inherit externalsrc
EXTERNALSRC = "${COREBASE}/../externalsrc/nano"

[conf]
# External source configuration
INHERIT += "externalsrc"
EXTERNALSRC_pn-nano = "${COREBASE}/../externalsrc/nano"
EXTERNALSRC_BUILD_pn-nano = "${COREBASE}/../externalsrc/nano"
```

## 빌드 의존성(DEPENDS) vs 실행 시간 의존성(RDEPENDS)
```
Category           | Build-time Dependency (DEPENDS)                                   | Runtime Dependency (RDEPENDS)
----------------------------------------------------------------------------------------------------------------------------------------------------------------
Concept Definition | Requirements for compiling and linking the source code.           | Requirements for running the program on the target board.
Target Object      | Recipe name or PROVIDES value.                                    | Package name (individual unit within an image).
Key Variable       | DEPENDS = "ncurses"                                               | RDEPENDS:${PN} = "bash"
Trigger Stage      | do_configure, do_compile                                          | do_rootfs (image creation) or during package installation.
Key Deliverables   | Header files (.h), Static/Shared libraries (.a, .so).             | Shell scripts, runtime libraries, config files, fonts, etc.
Mechanism          | Artifacts are installed into the Build Sysroot.                   | Packages are forcibly included in the Root File System.
Bitbake Process    | Ensures do_populate_sysroot of the dependency is completed first. |     Adds the package to the package manager's (RPM, IPK) install list.
Auto-Detection     | Manual. Must be explicitly defined by the developer.              | Partial. Automatically tracked via .so dynamic linking (shlibs).

[Supplemental Terms for Clarity]
Build Sysroot:
    A staging area on the host machine where headers and libraries are collected for the compiler.

Rootfs (Root File System):
    The actual file system that is flashed onto the target hardware.

${PN}:
    A Bitbake variable representing the Package Name. In RDEPENDS, it specifies which output package requires the dependency (e.g., the main package, not the -dev or -dbg versions).

shlibs:
    A Yocto mechanism that scans compiled binaries for library dependencies and automatically adds them to RDEPENDS.

Category | Variable           | Description
---------------------------------------------------------------------------------------------
Path     | STAGING_DIR_TARGET | Sysroot for the target architecture (libraries/headers).
Path     | STAGING_DIR_NATIVE | Sysroot for the build host (native tools/utilities).
Runtime  | RDEPENDS           | Hard requirement; must be installed for the package to work.
Runtime  | RRECOMMENDS        | Soft requirement; installed by default but can be omitted.
Runtime  | RPROVIDES          | Provides a virtual package name or alias.
Runtime  | RCONFLICTS         | List of packages that cannot be installed simultaneously.
Runtime  | RREPLACES          | List of packages to be replaced by this package.

STAGING_DIR_TARGET은 컴파일러가 타겟용 코드를 만들 때 곁눈질하는 곳입니다.

RDEPENDS는 타겟 보드에서 프로그램이 실행될 때 "내 친구도 같이 데려와!"라고 외치는 것과 같습니다.

RREPLACES와 RCONFLICTS는 주로 시스템 업데이트나 패키지 교체 시나리오에서 패키지 매니저(opkg, rpm)가 올바른 결정을 내리도록 돕는 이정표 역할을 합니다.
```

## IMAGE_INSTALL, IMAGE_FEATURES, EXTRA_IMAGE_FEATURES
```
Variable             | Description                                     | Key Characteristics
IMAGE_INSTALL        | 이미지에 설치할 개별 패키지를 명시적으로 지정합니다.       | 레시피에서 생성된 특정 패키지(예: bash, vim)를 직접 추가할 때 사용합니다. 주로 core-image.bb 같은 이미지 레시피 내부에서 설정합니다.

IMAGE_FEATURES       |   이미지의 **기능 단위(Feature sets)**를 활성화합니다. | 패키지 하나하나가 아닌, 특정 목적을 위한 패키지 그룹(예: ssh-server-openssh, x11)을 통째로 추가할 때 사용합니다. 이미지 레시피 내에 정의됩니다.

EXTRA_IMAGE_FEATURES |   IMAGE_FEATURES에 추가적인 기능을 더할 때 사용합니다.  | 주로 이미지 레시피를 수정하지 않고 local.conf에서 빌드 시점에 유연하게 기능을 추가(예: debug-tweaks)할 때 사용합니다.
```

## Package Group
```
[Package Group 상세 요약]
구분                 | 주요 내용 (Details)
----------------------------------------------------------------------------------------------------------------------------
정의 (Definition)    | 관련 있는 여러 패키지를 하나의 논리적인 집합으로 묶어주는 가상 레시피입니다. inherit packagegroup을 통해 선언됩니다.
작동 원리 (Workflow)  | 1. 의존성 전파: IMAGE_INSTALL에 추가된 그룹명을 인식합니다.
                    | 2. RDEPENDS 분석: 레시피 내에 정의된 패키지 목록을 재귀적으로 추적합니다.
                    | 3. 패키징 단계: 빌드 시스템은 실제 바이너리를 만들지 않고, 런타임 의존성 정보만 담긴 빈 패키지를 생성하여 설치를 유도합니다.
장점 (Benefits)      | 추상화: 이미지 레시피를 수정하지 않고 패키지 구성을 변경 가능합니다.
                    | 재사용: 동일한 패키지 세트를 여러 이미지(Minimal, Full 등)에 쉽게 적용합니다.
주의 사항 (Cautions)  | 네이밍 규칙: 관례적으로 packagegroup- 접두사를 사용해야 합니다.
                    | 컴파일 불가: 소스 코드를 직접 컴파일하거나 do_install 태스크를 수행하지 않습니다.
                    | 아키텍처 설정: 하드웨어 의존성이 있는 경우 PACKAGE_ARCH = "${MACHINE_ARCH}" 설정이 필요합니다.

[Package Group의 구조적 흐름]
구성 요소              | 역할 (Role)                | 비고
----------------------------------------------------------------------------------------------------------------------------
MAGE_INSTALL         |  최종 이미지에 그룹 이름만 추가 | IMAGE_INSTALL += "packagegroup-practice"
inherit packagegroup |    패키지 그룹 전용 클래스 상속 | 기본 빌드 태스크(compile, configure)를 비활성화함
RDEPENDS:${PN}       | 그룹에 포함될 패키지 리스트 정의 | ${PN}은 레시피 이름(패키지 이름)을 의미

[주의 사항 상세]
순환 의존성 금지:
    Package Group A가 B를 포함하고, B가 다시 A를 포함하는 구조가 되지 않도록 주의해야 합니다.

노드 패키지화:
    Package Group 자체가 결과물로 생성되지만, 이는 실제 파일이 없는 Meta-package 형태입니다. 따라서 FILES:${PN} 등을 통해 파일을 수동으로 넣으려 하면 오류가 발생할 수 있습니다.

레이어 의존성:
    Package Group 내에 다른 레이어의 패키지가 포함되어 있다면, layer.conf의 LAYERDEPENDS가 정확히 설정되어 있어야 합니다.

Add>
[meta-practice/recipes-core/packagegroups/packagegroup-practice.bb]
# We have a conf and classes directory, add to BBPATH
BBPATH .= ":${LAYERDIR}"

# We have recipes-* directories, add to BBFILES
BBFILES += "\
    ${LAYERDIR}/recipes-*/*.bb \
    ${LAYERDIR}/recipes-*/*/*.bb \
    ${LAYERDIR}/recipes-*/*.bbappend \
    ${LAYERDIR}/recipes-*/*/*.bbappend \
    "

BBFILE_COLLECTIONS += "practice"
BBFILE_PATTERN_practice = "^${LAYERDIR}/"
BBFILE_PRIORITY_practice = "6"

LAYERDEPENDS_practice = "core"
# LAYERSERIES_COMPAT_practice = "kirkstone"
LAYERSERIES_COMPAT_practice = "${LAYERSERIES_COMPAT_core}"

[meta-practice/recipes-core/image/core-image-minimal.bbappend]
IMAGE_INSTALL += "packagegroup-practice"

[meta-practice/recipes-core/packagegroups/packagegroup-practice.bb]
DESCRIPTION = "this package practice is great's packages"
inherit packagegroup

PACKAGE_ARCH = "${MACHINE_ARCH}"
RDEPENDS:${PN} = "\
    helloworld \
    nano \
    "

remove>
[meta-helloworld/recipes-core/images/core-image-minimal.bbappend]
# IMAGE_INSTALL:append = " helloworld"

[meta-nano-editor/recipes-core/images/core-image-minimal.bbappend]
# IMAGE_INSTALL += "nano"
```

## 대체 패키지 생성에 따른 의존성 처리
```
copy layers/meta-helloworld/recipes-helloworld
--> layers/meta-helloworld/recipes-helloworld-ng

rename layers/meta-helloworld/recipes-helloworld-ng/helloworld.bb
--> layers/meta-helloworld/recipes-helloworld-ng/helloworld-ng.bb

...
modify
SRC_URI "\
    file://helloworld.c
    ...
    "
-->
SRC_URI "\
    file://helloworld-ng.c
    ...
    "

Add
...
RREPLACES:${PN} = "helloworld"
RPROVIDES:${PN} = "helloworld"
RCONFLICTS:${PN} = "helloworld"
...

Add BBMASK
[build/conf/local.conf]
...
# BBMASK is used to exclude certain layers or recipes from the build.
# This can be useful if you have multiple layers that contain recipes with the same name
# and you want to ensure that only one of them is used in the build.
# By adding a layer or recipe to BBMASK, you are telling BitBake to ignore it during the build process.
BBMASK = "\
    ../layers/meta-helloworld/recipes-helloworld/ \
    "

[command]
bitbake helloworld-ng
bitbake -C rootfs
```

## Custom BSP Layer
```
REF>
[poky/meta/conf/machine]
- qemuarm64.conf

[layers/meta-practice-bsp]
├── conf
│   ├── layer.conf
│   └── machine
│       └── practice.conf
├── LICENSE
├── README.md
└── recipes-kernel
    └── linux
        ├── file
        └── linux-yocto_5.15.bbappend
```

## Add Fragment Config & Patches
```

[layers/meta-practice-bsp/recipes-kernel/linux/linux-yocto_5.15.bbappend]
...
# fragment configs
SRC_URI += "\
    file://enable_ext3_fs.cfg \
    file://enable_practice.cfg \
    "

# patches
SRC_URI += "\
    file://0001-Add-practice-driver-for-build-system-testing.patch \
    "

# add the path to the kernel config fragments and patches
FILESEXTRAPATHS:prepend := "${THISDIR}/file:"
```

## internal console
```
# allows the interactive session to run within your existing terminal.
OE_TERMINAL = "tmux"
```

## kernel metadata
```
[linux-yocto.inc]
- inherit kernel.bbclass, kernel-yocto.bbclass

<required>
KMACHINE:
- About BSP Layer

KBRANCH:
- git branch

<optional>
KERNEL_FEATURES:
- TODO

LINUX_KERNEL_TYPE(=> KTYPE):
- standard
- tiny
- preempt-rt

<metadata>
- scc(Serial Configuration Control) description(.scc)
  > define: variable define
  > kconf: fragment config file
  > patch: patch file
  > include: include file
- environment setting fragment(.cfg)
- patch(.patch)
```

## non linux-yocto
```
[layers/meta-practice-bsp/conf/machine/practice.conf]
...
PREFERRED_PROVIDER_virtual/kernel = "linux-practice"
...

[layers/meta-practice-bsp/recipes-kernel/linux/file/defconfig]
- from .config

[layers/meta-practice-bsp/recipes-kernel/linux/linux-practice.bb]
DESCRIPTION = "Linux kernel from kernel.org git repository"
SECTION = "kernel"
LICENSE = "GPLv2"

inherit kernel
inherit kernel-yocto

# branch, name, tag and nocheckout are passed to git fetcher
SRC_URI = "git://github.com/practice-yocto/linux.git;protocol=https;branch=v5.15/dev"
LIC_FILES_CHKSUM = "file://COPYING;md5=6bc538ed5bd9a7fc9398086aedcd7e46"

# SRCREV is the commit hash to fetch. It can be overridden by the user when building the image, e.g.:
# SRCREV = "8bb7eca972ad531c9b149c0a51ab43a417385813"
SRCREV = "${AUTOREV}"

# defconfig file to be used for the kernel build
SRC_URI += "file://defconfig "

# SCC
SRC_URI += "\
    file://practice-kernel.scc \
    "

LINUX_VERSION ?= "5.15"
LINUX_VERSION_EXTENSION ?= "-practice"

# SRCPV: ${GIT_COMMIT} truncated to 7 characters
PROVIDES += "virtual/kernel"
PV = "${LINUX_VERSION}+git${SRCPV}"
COMPATIBLE_MACHINE = "practice"

FILESEXTRAPATHS:prepend := "${THISDIR}/file:"

$ bitbake linux-practice -C fetch
$ bitbake practice-image -C rootfs
$ runqemu practice-image nographic
```

## external kernel source
```
[layers/meta-practice-bsp/append/linux-practice.bbappend]
# This bbappend file is used to modify the linux-practice recipe to use the kernel source from the externalsrc layer.
inherit externalsrc

# We want to use the kernel source from the externalsrc layer, so we set EXTERNALSRC to point to it.
EXTERNALSRC = "${COREBASE}/../externalsrc/kernel-source"

[layers/meta-practice-bsp/conf/layer.conf]
...
BBFILES += "\
    ...
    ${LAYERDIR}/append/*.bbappend \
    ${LAYERDIR}/append/*/*.bbappend \
    "
...
```

## using defconfig in kernel source
```
[layers/meta-practice-bsp/recipes-kernel/linux/linux-practice.bb]
...
# It is copied to the kernel source directory and used as the default configuration for the kernel build.
KBUILD_DEFCONFIG = "practice_defconfig"
...
```

## external kernel module
```
[layers/meta-practice-bsp/recipes-practice-kernel-module]
├── file
│   ├── COPYING
│   ├── Makefile
│   └── practice-kernel-module.c
└── practice-kernel-module.bb

[layers/meta-practice-bsp/recipes-practice-kernel-module/practice-kernel-module.bb]
SUMMARY = "Example of how to build an external linux kernel module"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://COPYING;md5=c7e03b2484dde1e8b91e76aa5e668f8f"

inherit module

SRC_URI = "\
    file://practice-kernel-module.c \
    file://Makefile \
    file://COPYING \
    "

KERNEL_MODULE_AUTOLOAD += "practice-kernel-module"
S = "${WORKDIR}"
ALLOW_EMPTY_${PN} = "1"
FILESEXTRAPATHS:prepend := "${THISDIR}/file:"

[layers/meta-practice/recipes-core/image/practice-image.bb]
...
IMAGE_INSTALL += "packagegroup-practice"
...

$ bitbake practice-kernel-module
$ bitbake practice-image -C rootfs

MACHINE_EXTRA_RDEPENDS
- 부팅 시 필수적으로 사용되는 패키지가 아닐때.
MACHINE_ESSENTIAL_EXTRA_RDEPENDS
- 부팅 시 필수적으로 사용되는 패키지 일때.

[layers/meta-practice/recipes-core/image/practice-image.bb]
...
# IMAGE_INSTALL += "practice-kernel-module"
...

[layers/meta-practice-bsp/conf/machine/practice.conf]
...
# MACHINE_ESSENTIAL_EXTRA_RDEPENDS += "kernel-module-<module name>"
MACHINE_ESSENTIAL_EXTRA_RDEPENDS += "kernel-module-practice-kernel-module"
...

[layers/meta-practice-bsp/recipes-practice-kernel-module/practice-kernel-module.bb]
...
RPROVIDES:${PN} += "kernel-module-practice-kernel-module"
...

<module>
root@practice:~# cat /etc/modules-load.d/practice-kernel-module.conf
practice-kernel-module

root@practice:~# systemctl status systemd-modules-load
* systemd-modules-load.service - Load Kernel Modules
     Loaded: loaded (/lib/systemd/system/systemd-modules-load.service; static)
     Active: active (exited) since Mon 2026-05-04 12:01:12 UTC; 15s ago
       Docs: man:systemd-modules-load.service(8)
             man:modules-load.d(5)
    Process: 133 ExecStart=/lib/systemd/systemd-modules-load (code=exited, status=0/SUCCESS)
   Main PID: 133 (code=exited, status=0/SUCCESS)

Notice: journal has been rotated since unit was started, output may be incomplete.

- /etc/modules-load.d/*.conf
- /run/modules-load.d/*.conf
- /usr/lib/modules-load.d/*.conf
```
