cmake_minimum_required(VERSION 3.1)
project(boostbuilder)

find_package(Git REQUIRED)

set(URL "https://github.com/boostorg/boost.git")
if(NOT BRANCH)
    set(BRANCH master)
endif()

set(CLONE_DIR "${CMAKE_CURRENT_BINARY_DIR}/clone_dir")

if(BOOSTBUILD_ALREADY_CLONED)
    if(BRANCH AND NOT BOOSTBUILD_CURRENT_URL STREQUAL URL)
        message(STATUS "URL changed '${BOOSTBUILD_CURRENT_URL}' -> '${URL}'")
        unset(BOOSTBUILD_ALREADY_CLONED CACHE)
    endif()
    if(BRANCH AND NOT BOOSTBUILD_CURRENT_BRANCH STREQUAL BRANCH)
        message(STATUS "BRANCH changed '${BOOSTBUILD_CURRENT_BRANCH}' -> '${BRANCH}'")
        unset(BOOSTBUILD_ALREADY_CLONED CACHE)
    endif()
endif()

if(NOT BOOSTBUILD_ALREADY_CLONED)
    file(REMOVE_RECURSE "${CLONE_DIR}")
    message(STATUS "git clone --branch ${BRANCH} --depth 1 ${URL}")
    execute_process(
        COMMAND ${GIT_EXECUTABLE}
            "clone"
            "--branch" "${BRANCH}"
            "--depth" "1"
            "${URL}"
            "${CLONE_DIR}"
        RESULT_VARIABLE r)
    if(r)
        message(FATAL_ERROR "'git clone' failed.")
    endif()
    set(BOOSTBUILD_ALREADY_CLONED 1 CACHE INTERNAL "")
    set(BOOSTBUILD_CURRENT_URL "${URL}" CACHE INTERNAL "")
    set(BOOSTBUILD_CURRENT_BRANCH "${BRANCH}" CACHE INTERNAL "")
    unset(BOOSTBUILD_CURRENT_LIBS CACHE)
else()
    message(STATUS "Using existing checkout.")
    message(STATUS "URL: ${BOOSTBUILD_CURRENT_URL}")
    message(STATUS "branch: ${BOOSTBUILD_CURRENT_BRANCH}")
endif()

list(APPEND LIBS core config assert static_assert predef detail)
list(SORT LIBS)
list(REMOVE_DUPLICATES LIBS)
unset(PREFIXED_LIBS)
foreach(l IN LISTS LIBS)
    list(APPEND PREFIXED_LIBS "libs/${l}")
endforeach()
string(REPLACE ";" " " LIBS_SPACE "${LIBS}")
string(REPLACE ";" " " PREFIXED_LIBS_SPACE "${PREFIXED_LIBS}")
if(NOT BOOSTBUILD_CURRENT_LIBS STREQUAL LIBS)
    message(STATUS "cd ${CLONE_DIR} \\")
    message(STATUS "&& git submodule update --init --depth 1 ${PREFIXED_LIBS_SPACE}")
    execute_process(
        COMMAND ${GIT_EXECUTABLE}
            "submodule"
            "update"
            "--init"
            "--depth" "1"
            ${PREFIXED_LIBS}
        WORKING_DIRECTORY "${CLONE_DIR}"
        RESULT_VARIABLE r)
    if(r)
        message(FATAL_ERROR "'git submodule update' failed.")
    endif()
    set(BOOSTBUILD_CURRENT_LIBS "${LIBS}" CACHE INTERNAL "")
else()
    message(STATUS "libs: ${LIBS_SPACE}")
endif()

# For now supporting only header-only libs
foreach(l IN LISTS LIBS)
    install(
        DIRECTORY ${CLONE_DIR}/libs/${l}/include/boost
        DESTINATION include
    )
endforeach()
