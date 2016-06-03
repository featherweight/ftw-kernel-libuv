#
#   Copyright (c) 2016 Jack R. Dunaway. All rights reserved.
#
#   Permission is hereby granted, free of charge, to any person obtaining a copy
#   of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom
#   the Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
#   IN THE SOFTWARE.
#

cmake_minimum_required (VERSION 2.8.7)

project (libuv C)

# Detect library version
file (READ include/uv-version.h UV_HDR_STR)
string (REGEX REPLACE ".*#define +UV_VERSION_MAJOR +([0-9]+).*" "\\1" UV_VERSION_MAJOR "${UV_HDR_STR}")
string (REGEX REPLACE ".*#define +UV_VERSION_MINOR +([0-9]+).*" "\\1" UV_VERSION_MINOR "${UV_HDR_STR}")
string (REGEX REPLACE ".*#define +UV_VERSION_PATCH +([0-9]+).*" "\\1" UV_VERSION_PATCH "${UV_HDR_STR}")

if ((UV_VERSION_MAJOR STREQUAL "") OR (UV_VERSION_MINOR STREQUAL "") OR (UV_VERSION_PATCH STREQUAL ""))
    message (FATAL_ERROR "Could not read ABI version from include/uv-version.h")
else ()
    set (UV_ABI_VERSION "${UV_VERSION_MAJOR}.${UV_VERSION_MINOR}")
    set (UV_LIB_VERSION "${UV_VERSION_MAJOR}.${UV_VERSION_MINOR}.${UV_VERSION_PATCH}")
    message (STATUS "Detected libuv ABI v${UV_ABI_VERSION}")
    message (STATUS "Detected libuv API v${UV_LIB_VERSION}")
endif ()

option (UV_STATIC_LIB "Build static library instead of shared library." ON)
option (UV_ENABLE_TESTS "Build and run tests" ON)

set (UV_SOURCE_FILES
    common.gypi
    include/uv.h
    include/tree.h
    include/uv-errno.h
    include/uv-threadpool.h
    include/uv-version.h
    src/fs-poll.c
    src/heap-inl.h
    src/inet.c
    src/queue.h
    src/threadpool.c
    src/uv-common.c
    src/uv-common.h
    src/version.c
)

set (UV_REQUIRED_LIBRARIES "")

if (WIN32)
    list (APPEND UV_SOURCE_FILES
        include/uv-win.h
        src/win/async.c
        src/win/atomicops-inl.h
        src/win/core.c
        src/win/dl.c
        src/win/error.c
        src/win/fs.c
        src/win/fs-event.c
        src/win/getaddrinfo.c
        src/win/getnameinfo.c
        src/win/handle.c
        src/win/handle-inl.h
        src/win/internal.h
        src/win/loop-watcher.c
        src/win/pipe.c
        src/win/thread.c
        src/win/poll.c
        src/win/process.c
        src/win/process-stdio.c
        src/win/req.c
        src/win/req-inl.h
        src/win/signal.c
        src/win/snprintf.c
        src/win/stream.c
        src/win/stream-inl.h
        src/win/tcp.c
        src/win/tty.c
        src/win/timer.c
        src/win/udp.c
        src/win/util.c
        src/win/winapi.c
        src/win/winapi.h
        src/win/winsock.c
        src/win/winsock.h
    )
else ()
    list (APPEND UV_SOURCE_FILES
        include/uv-unix.h
        include/uv-linux.h
        include/uv-sunos.h
        include/uv-darwin.h
        include/uv-bsd.h
        include/uv-aix.h
        src/unix/async.c
        src/unix/atomic-ops.h
        src/unix/core.c
        src/unix/dl.c
        src/unix/fs.c
        src/unix/getaddrinfo.c
        src/unix/getnameinfo.c
        src/unix/internal.h
        src/unix/loop-watcher.c
        src/unix/loop.c
        src/unix/pipe.c
        src/unix/poll.c
        src/unix/process.c
        src/unix/signal.c
        src/unix/spinlock.h
        src/unix/stream.c
        src/unix/tcp.c
        src/unix/thread.c
        src/unix/timer.c
        src/unix/tty.c
        src/unix/udp.c
    )

    find_package (Threads REQUIRED)

endif ()

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    add_definitions (-D_POSIX_C_SOURCE=200112)
    add_definitions (-D_GNU_SOURCE)
    list (APPEND UV_SOURCE_FILES
        src/unix/linux-core.c
        src/unix/linux-inotify.c
        src/unix/linux-syscalls.c
        src/unix/linux-syscalls.h
        src/unix/proctitle.c
    )
    list (APPEND UV_REQUIRED_LIBRARIES
        dl
        rt
        util
    )
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    add_definitions (-D_DARWIN_USE_64_BIT_INODE=1)
    add_definitions (-D_DARWIN_UNLIMITED_SELECT=1)
    list (APPEND UV_SOURCE_FILES
        src/unix/darwin.c
        src/unix/fsevents.c
        src/unix/darwin-proctitle.c
        src/unix/pthread-barrier.c
        src/unix/proctitle.c
    )
elseif (CMAKE_SYSTEM_NAME MATCHES "android")
    list (APPEND UV_SOURCE_FILES
        src/unix/linux-core.c
        src/unix/linux-inotify.c
        src/unix/linux-syscalls.c
        src/unix/linux-syscalls.h
        src/unix/pthread-fixes.c
        src/unix/android-ifaddrs.c
        src/unix/pthread-barrier.c
    )
    list (APPEND UV_REQUIRED_LIBRARIES
        dl
    )
elseif (CMAKE_SYSTEM_NAME MATCHES "Windows")
    add_definitions (-D_WIN32_WINNT=0x0600)
    add_definitions (-D_GNU_SOURCE)
    add_definitions (-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions (-D_CRT_NONSTDC_NO_DEPRECATE)
    list (APPEND UV_REQUIRED_LIBRARIES
        advapi32
        iphlpapi
        psapi
        shell32
        user32
        userenv
        ws2_32
    )
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD")

elseif (CMAKE_SYSTEM_NAME MATCHES "NetBSD")

elseif (CMAKE_SYSTEM_NAME MATCHES "OpenBSD")

elseif (CMAKE_SYSTEM_NAME MATCHES "Solaris|SunOS")

else ()
    message (AUTHOR_WARNING "WARNING: This platform may or may not be supported: ${CMAKE_SYSTEM_NAME}")
endif ()

if (UV_STATIC_LIB)
    add_library (${PROJECT_NAME} STATIC ${UV_SOURCE_FILES})
else ()
    add_library (${PROJECT_NAME} SHARED ${UV_SOURCE_FILES})
    add_definitions (-DUV_SHARED_LIB)
    set_target_properties (${PROJECT_NAME} PROPERTIES
        VERSION "${UV_API_VERSION}"
        SOVERSION "${UV_ABI_VERSION}")
endif ()

target_include_directories (${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)
target_link_libraries (${PROJECT_NAME} ${UV_REQUIRED_LIBRARIES})

#  Build unit tests.
if (UV_TEST)
    enable_testing ()
    set (UV_TESTS "")
endif ()
