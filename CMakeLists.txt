#
# project: fips-efsw
#

if (NOT FIPS_IMPORT)
    cmake_minimum_required(VERSION 2.8)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    fips_setup()
    fips_project(fips-efsw)
endif()

if (FIPS_ANDROID OR FIPS_NACL OR FIPS_EMSCRIPTEN)
	message(ERROR Platform not supported)
endif() # CMAKE_TOOLCHAIN_FILE

#-------------------------------------------------------------------------------
option(EFSW_VERBOSE "Build efsw with verbose mode.")

if (EFSW_VERBOSE)
    add_definitions(-DEFSW_VERBOSE)
endif()

#-[ libs ]---------------------------------------------------------------------

fips_begin_lib(efsw)
    fips_include_directories(efsw/include efsw/src)
    fips_files(
        efsw/src/efsw/Debug.cpp
        efsw/src/efsw/DirectorySnapshot.cpp
        efsw/src/efsw/DirectorySnapshotDiff.cpp
        efsw/src/efsw/DirWatcherGeneric.cpp
        efsw/src/efsw/FileInfo.cpp
        efsw/src/efsw/FileSystem.cpp
        efsw/src/efsw/FileWatcher.cpp
        efsw/src/efsw/FileWatcherCWrapper.cpp
        efsw/src/efsw/FileWatcherGeneric.cpp
        efsw/src/efsw/FileWatcherImpl.cpp
        efsw/src/efsw/Log.cpp
        efsw/src/efsw/Mutex.cpp
        efsw/src/efsw/String.cpp
        efsw/src/efsw/System.cpp
        efsw/src/efsw/Thread.cpp
        efsw/src/efsw/Watcher.cpp
        efsw/src/efsw/WatcherGeneric.cpp
    )

    if (FIPS_WINDOWS)
        fips_files(
            efsw/src/efsw/platform/win/FileSystemImpl.cpp
            efsw/src/efsw/platform/win/MutexImpl.cpp
            efsw/src/efsw/platform/win/SystemImpl.cpp
            efsw/src/efsw/platform/win/ThreadImpl.cpp
            efsw/src/efsw/WatcherWin32.cpp
            efsw/src/efsw/FileWatcherWin32.cpp
        )
    elseif (FIPS_MACOS)
        fips_files(
            efsw/src/efsw/platform/posix/FileSystemImpl.cpp
            efsw/src/efsw/platform/posix/MutexImpl.cpp
            efsw/src/efsw/platform/posix/SystemImpl.cpp
            efsw/src/efsw/platform/posix/ThreadImpl.cpp
            efsw/src/efsw/FileWatcherFSEvents.cpp
            efsw/src/efsw/FileWatcherKqueue.cpp
            efsw/src/efsw/WatcherFSEvents.cpp
            efsw/src/efsw/WatcherKqueue.cpp
        )
        exec_program(uname ARGS -v  OUTPUT_VARIABLE OSX_VERSION)
        string(REGEX MATCH "[0-9]+" OSX_VERSION ${OSX_VERSION})
        if (NOT OSX_VERSION GREATER 9)
            add_definitions(-DEFSW_FSEVENTS_NOT_SUPPORTED)
        endif()
        fips_frameworks_osx(CoreFoundation CoreServices)
    elseif (FIPS_LINUX)
        fips_files(
            efsw/src/efsw/platform/posix/FileSystemImpl.cpp
            efsw/src/efsw/platform/posix/MutexImpl.cpp
            efsw/src/efsw/platform/posix/SystemImpl.cpp
            efsw/src/efsw/platform/posix/ThreadImpl.cpp
            efsw/src/efsw/WatcherInotify.cpp
            efsw/src/efsw/FileWatcherInotify.cpp
        )
        if (NOT EXISTS "/usr/include/sys/inotify.h" AND NOT EXISTS "/usr/local/include/sys/inotify.h")
            add_definitions(-DEFSW_INOTIFY_NOSYS)
        endif()
        fips_deps(pthread)
    elseif (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
        fips_files(
            efsw/src/efsw/platform/posix/FileSystemImpl.cpp
            efsw/src/efsw/platform/posix/MutexImpl.cpp
            efsw/src/efsw/platform/posix/SystemImpl.cpp
            efsw/src/efsw/platform/posix/ThreadImpl.cpp
            efsw/src/efsw/FileWatcherKqueue.cpp
            efsw/src/efsw/WatcherKqueue.cpp
        )
        fips_deps(pthread)
    endif()

    if (MSVC)
        add_definitions(-D_SCL_SECURE_NO_WARNINGS)
    else()
        add_definitions(-Wall -Wno-long-long)
    endif()

    if (FIPS_DEBUG)
        add_definitions(-DDEBUG)
    else()
        add_definitions(-DNDEBUG)
    endif()
fips_end_lib()

#-[ execs ]---------------------------------------------------------------------

fips_ide_group(Tests)
fips_begin_app(efsw-test cmdline)
    fips_files(efsw/src/test/efsw-test.cpp)
    fips_deps(efsw)
fips_end_app()

#-------------------------------------------------------------------------------
