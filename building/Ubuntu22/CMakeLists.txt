# Copyright 2023-2024 DreamWorks Animation LLC
# SPDX-License-Identifier: Apache-2.0

# These ExternalProject targets can be used to download, build and
# install many of the Moonray dependencies.
# The targets are chained using dependencies so that they run
# serially.

cmake_minimum_required (VERSION 3.23.1)
project(openmoonray_third_party)

include(ExternalProject)

set(InstallRoot /opt/MoonRay/installs CACHE FILEPATH "Install root for dependencies")

ExternalProject_Add(TBB
  URL https://github.com/uxlfoundation/oneTBB/releases/download/v2020.3/tbb-2020.3-lin.tgz
  URL_HASH SHA256=bb8cddd0277605d3ee7f4e19b138c983f298d69fcbb585385b59ef7239d5ef83
  DOWNLOAD_DIR ${CMAKE_BINARY_DIR}/downloads
  SOURCE_DIR ${InstallRoot}/tbb-2020.3
  CONFIGURE_COMMAND ""  # Skip configure step
  BUILD_COMMAND ""      # Skip build step
  INSTALL_COMMAND ""    # Skip install step
  )
set(CHAIN TBB)

ExternalProject_Add(JsonCpp
    GIT_REPOSITORY https://github.com/open-source-parsers/jsoncpp.git
    GIT_TAG 5defb4ed1a4293b8e2bf641e16b156fb9de498cc # 1.9.5
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DBUILD_SHARED_LIBS=1
    -DPYTHON_EXECUTABLE=/usr/bin/python3
    -DJSONCPP_LIB_BUILD_SHARED:BOOL=ON
    -DJSONCPP_WITH_PKGCONFIG_SUPPORT=OFF
	)
set(CHAIN JsonCpp)

ExternalProject_Add(OpenSubdiv
    GIT_REPOSITORY https://github.com/PixarAnimationStudios/OpenSubdiv
    GIT_TAG 8ffa2b6566be10209529d7a0d1db02a0796b160c # v3_5_0
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DCMAKE_BUILD_TYPE=Release
    -DPYTHON_EXECUTABLE=/usr/bin/python3
    -DNO_PTEX=1 -DNO_OMP=1 -DNO_TBB=1 -DNO_CUDA=1 -DNO_GLFW_X11=1 -DNO_DOC=1
    -DNO_OPENCL=1 -DNO_CLEW=1 -DNO_REGRESSION=1 -DNO_EXAMPLES=1 -DNO_TUTORIALS=1 -DNO_GLTESTS=1
    -DNO_MACOS_FRAMEWORK=1 -DNO_METAL=1 -DNO_TESTS=1
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenSubdiv)

ExternalProject_Add(OpenEXR
    GIT_REPOSITORY https://github.com/AcademySoftwareFoundation/openexr
    GIT_TAG 8bc3741131db146ad08a5b83af9e6e48f0e94a03 # v2.5.7
    PATCH_COMMAND patch IlmBase/Half/CMakeLists.txt ${CMAKE_CURRENT_SOURCE_DIR}/../Imath_include_paths.patch
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DBUILD_SHARED_LIBS=OFF
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenEXR)

ExternalProject_Add(Random123
    GIT_REPOSITORY https://github.com/DEShawResearch/random123
    GIT_TAG 726a093cd9a73f3ec3c8d7a70ff10ed8efec8d13 # v1.14.0
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND make install-include prefix=${InstallRoot}
    DEPENDS ${CHAIN}
	)
set(CHAIN Random123)

ExternalProject_Add(ISPC
    URL https://github.com/ispc/ispc/releases/download/v1.20.0/ispc-v1.20.0-linux.tar.gz
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND cp bin/ispc ${InstallRoot}/bin
    DEPENDS ${CHAIN}
	)
set(CHAIN ISPC)

ExternalProject_Add(embree
    GIT_REPOSITORY https://github.com/embree/embree
    GIT_TAG 341ef8c45d1ae072ead1ab65cd76e88b03d9302c # v4.2.0
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DEMBREE_ISPC_EXECUTABLE=<INSTALL_DIR>/bin/ispc
    -DEMBREE_ISPC_SUPPORT=ON
    -DEMBREE_IGNORE_INVALID_RAYS=ON
    -DEMBREE_RAY_MASK=ON
    -DEMBREE_MAX_ISA=AVX512
    -DEMBREE_TUTORIALS=OFF
    -DBUILD_SHARED_LIBS=ON
	-DTBB_DIR=${InstallRoot}/tbb-2020.3/tbb/cmake
    DEPENDS ${CHAIN}
	)
set(CHAIN embree)

ExternalProject_Add(OpenColorIO
    GIT_REPOSITORY https://github.com/AcademySoftwareFoundation/OpenColorIO
    GIT_TAG 056b7b0cb0d087961e9dba75104820e44faf52a1 # v2.0.2
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DOCIO_BUILD_APPS=OFF
    -DOCIO_BUILD_TESTS=OFF
    -DOCIO_BUILD_GPU_TESTS=OFF
    -DOCIO_USE_SSE=OFF
    -DOCIO_BUILD_PYTHON=OFF
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DBUILD_SHARED_LIBS=ON
    -DOCIO_BUILD_STATIC=OFF
    -DCMAKE_CXX_STANDARD=17
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenColorIO)

ExternalProject_Add(OpenImageIO
    GIT_REPOSITORY https://github.com/OpenImageIO/oiio
    GIT_TAG 331a323468928c8017ad048b26d47c4e57a724a7 # 2.3.20.0
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DOpenEXR_ROOT=<INSTALL_DIR>
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DUSE_QT=0
    -DUSE_PYTHON=1
	-DJPEG_INCLUDE_DIR=/usr/include/x86_64-linux-gnu
	-DPYTHON_VERSION=3.10
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenImageIO)

ExternalProject_Add(OpenImageDenoise
    URL https://github.com/OpenImageDenoise/oidn/releases/download/v2.0.1/oidn-2.0.1.src.tar.gz
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DISPC_EXECUTABLE=<INSTALL_DIR>/bin/ispc
	-DTBB_DIR=${InstallRoot}/tbb-2020.3/tbb/cmake
    -DOIDN_APPS=OFF
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenImageDenoise)

ExternalProject_Add(OpenVDB
    URL https://github.com/AcademySoftwareFoundation/openvdb/archive/refs/tags/v9.1.0.tar.gz
    INSTALL_DIR ${InstallRoot}
    CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
	-DOPENVDB_CORE_STATIC=OFF
	-DTBB_ROOT=${InstallRoot}/tbb-2020.3/tbb
	-DTBB_LIBRARYDIR=${InstallRoot}/tbb-2020.3/tbb/lib/intel64/gcc4.8
	-DTBB_INCLUDEDIR=${InstallRoot}/tbb-2020.3/tbb/include
    DEPENDS ${CHAIN}
	)
set(CHAIN OpenVDB)

if(NOT NO_USD)
    ExternalProject_Add(USD
        GIT_REPOSITORY https://github.com/PixarAnimationStudios/USD
        GIT_TAG 0c7b9a95f155c221ff7df9270a39a52e3b23af8b # v22.11
        INSTALL_DIR ${InstallRoot}
        CMAKE_ARGS
		-Wno-dev
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_PREFIX_PATH=<INSTALL_DIR>
        -DPXR_ENABLE_PYTHON_SUPPORT=ON
        -DPXR_USE_PYTHON_3=ON
        -DPYTHON_INCLUDE_DIRS=/usr/include
        -DTBB_USE_DEBUG_BUILD=OFF
        -DTBB_DIR=${InstallRoot}/tbb-2020.3/tbb/cmake
		-DTBB_LIBRARY=${InstallRoot}/tbb-2020.3/tbb/lib/intel64/gcc4.8
		-DTBB_INCLUDE_DIRS=${InstallRoot}/tbb-2020.3/tbb/include
        -DPXR_BUILD_TESTS=OFF
        -DPXR_BUILD_EXAMPLES=OFF
        -DPXR_BUILD_TUTORIALS=OFF
        -DPXR_BUILD_USD_TOOLS=OFF
        -DPXR_ENABLE_PTEX_SUPPORT=OFF
        -DPXR_ENABLE_OPENVDB_SUPPORT=ON
        -DPXR_BUILD_USDVIEW=OFF
        -DTBB_SUPPRESS_DEPRECATED_MESSAGES=1
        DEPENDS ${CHAIN}
		)
    set(CHAIN USD)
endif()

