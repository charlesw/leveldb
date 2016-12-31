cmake_minimum_required (VERSION 3.7.1 FATAL_ERROR)

# Only interpret if() arguments as variables or keywords when unquoted.
cmake_policy(SET CMP0054 NEW) 

# Hunter Package Manager init
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.17.10.tar.gz"
    SHA1 "9dd07d0cae492bbbdf5d3049721b9526f4581b67"
)

# Project name and version
project(leveldb VERSION 0.1 LANGUAGES CXX)

# Platform detection
set(PROJECT_ADDITIONAL_DEFINES)
if(UNIX) 
    set(LEVELDB_PLATFORM LEVELDB_PLATFORM_POSIX)   
    if(CYGWIN)        
        set(LEVELDB_OS OS_LINUX)
        list(APPEND PROJECT_ADDITIONAL_DEFINES CYGWIN)    
    elseif(MSYS)        
        set(LEVELDB_OS OS_LINUX)
        list(APPEND PROJECT_ADDITIONAL_DEFINES MSYS) 
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")                    
        set(LEVELDB_OS OS_LINUX)        
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")                    
        set(LEVELDB_OS OS_MACOSX)          
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Android")                    
        set(LEVELDB_OS OS_ANDROID)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "IOS")                     
        set(LEVELDB_OS OS_MACOSX)
    else()
        message(FATAL_ERROR "The unix system ${CMAKE_SYSTEM_NAME} is not supported." )
    endif() 
elseif(WIN32)
    set(LEVELDB_PLATFORM LEVELDB_PLATFORM_WINDOWS)
    set(LEVELDB_OS OS_WINDOWS)
    # Force unicode character set on windows
    list(APPEND PROJECT_ADDITIONAL_DEFINES UNICODE _UNICODE) 
else()
    message(FATAL_ERROR "The system ${CMAKE_SYSTEM_NAME} is not supported." )
endif()

# Public headers
set(PROJECT_PUBLIC_HEADERS
    include/leveldb/c.h
    include/leveldb/cache.h
    include/leveldb/comparator.h
    include/leveldb/db.h
    include/leveldb/dumpfile.h
    include/leveldb/env.h
    include/leveldb/filter_policy.h
    include/leveldb/iterator.h
    include/leveldb/options.h
    include/leveldb/slice.h
    include/leveldb/status.h
    include/leveldb/table.h
    include/leveldb/table_builder.h
    include/leveldb/write_batch.h
)

# DB sources
set(PROJECT_DB_SOURCES
    db/builder.cc
    db/builder.h
    # db/c.cc - include for CAPI
    db/db_impl.cc
    db/db_impl.h
    db/db_iter.cc
    db/db_iter.h
    db/dbformat.cc
    db/dbformat.h
    db/dumpfile.cc
    db/filename.cc
    db/filename.h
    db/leveldbutil.cc
    db/log_format.h
    db/log_reader.cc
    db/log_reader.h
    db/log_writer.cc
    db/log_writer.h
    db/memtable.cc
    db/memtable.h
    db/repair.cc
    db/skiplist.h
    db/snapshot.h
    db/table_cache.cc
    db/table_cache.h
    db/version_edit.cc
    db/version_edit.h
    db/version_set.cc
    db/version_set.h
    db/write_batch.cc
    db/write_batch_internal.h
)

# helpers/memenv sources
set(PROJECT_HELPERS_SOURCES
    helpers/memenv/memenv.cc
    helpers/memenv/memenv.h
)

# port sources
set(PROJECT_PORT_SOURCES
    port/port.h
    port/thread_annotations.h
)

if(LEVELDB_PLATFORM STREQUAL "LEVELDB_PLATFORM_POSIX")    
    # Port - posix
    list(APPEND PROJECT_PORT_SOURCES
        port/port_posix.cc
        port/port_posix.h
        port/atomic_pointer.h
    )
elseif(LEVELDB_PLATFORM STREQUAL "LEVELDB_PLATFORM_WINDOWS")
    # Port - Windows
    list(APPEND PROJECT_PORT_SOURCES
        port/port_windows.cc
        port/port_windows.h
    )
endif()

## Port windows

# table sources
set(PROJECT_TABLE_SOURCES
    table/block.cc
    table/block.h
    table/block_builder.cc
    table/block_builder.h
    table/filter_block.cc
    table/filter_block.h
    table/format.cc
    table/format.h
    table/iterator.cc
    table/iterator_wrapper.h
    table/merger.cc
    table/merger.h
    table/table.cc
    table/table_builder.cc
    table/two_level_iterator.cc
    table/two_level_iterator.h
)

# util sources
set(PROJECT_UTIL_SOURCES
    util/arena.cc
    util/arena.h
    util/bloom.cc
    util/cache.cc
    util/coding.cc
    util/coding.h
    util/comparator.cc
    util/crc32c.cc
    util/crc32c.h
    util/env.cc
    util/filter_policy.cc
    util/hash.cc
    util/hash.h
    util/histogram.cc
    util/histogram.h
    util/logging.cc
    util/logging.h
    util/mutexlock.h
    util/options.cc
    util/random.h
    util/status.cc
)

if(LEVELDB_PLATFORM STREQUAL "LEVELDB_PLATFORM_POSIX")    
    # Port - posix
    list(APPEND PROJECT_UTIL_SOURCES
        util/env_posix.cc
        util/posix_logger.h
    )
elseif(LEVELDB_PLATFORM STREQUAL "LEVELDB_PLATFORM_WINDOWS")
    # Port - Windows
    list(APPEND PROJECT_UTIL_SOURCES
        util/env_windows.cc
    )
endif()

# Setup source groups for Visual Studio
SOURCE_GROUP(include\\leveldb FILES ${PROJECT_PUBLIC_HEADERS})
SOURCE_GROUP(db FILES ${PROJECT_DB_SOURCES})
SOURCE_GROUP(helpers FILES ${PROJECT_HELPERS_SOURCES})
SOURCE_GROUP(port FILES ${PROJECT_PORT_SOURCES})
SOURCE_GROUP(table FILES ${PROJECT_TABLE_SOURCES})
SOURCE_GROUP(util FILES ${PROJECT_UTIL_SOURCES})

add_library(${PROJECT_NAME} 
    ${PROJECT_PUBLIC_HEADERS}
    ${PROJECT_DB_SOURCES}
    ${PROJECT_HELPERS_SOURCES}
    ${PROJECT_PORT_SOURCES}
    ${PROJECT_TABLE_SOURCES}
    ${PROJECT_UTIL_SOURCES})
    
# Target C++14
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)

# Set target compiler definitions
target_compile_definitions(${PROJECT_NAME} PRIVATE
    ${LEVELDB_PLATFORM}
    ${LEVELDB_OS}
    ${PROJECT_ADDITIONAL_DEFINES}
)

# Define headers for this library. PUBLIC headers are contained in 
# ``/include/${LIBRARY_NAME}`` are used for compiling the library, 
# and will be added to consumers' include paths. While PRIVATE 
# headers are only used to be 
target_include_directories(${PROJECT_NAME} PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Add the hunter libraries that the project requires
#hunter_add_package(Boost COMPONENTS regex system filesystem)
#find_package(Boost CONFIG REQUIRED regex system filesystem)

# Threads 
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(${PROJECT_NAME} 
    Threads::Threads)

#target_link_libraries(${PROJECT_NAME}
#    PUBLIC Boost::regex Boost::system Boost::filesystem)

# TODO: Add unit\integration tests

# TODO: What about install target (do we need this???) 
