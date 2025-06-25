cmake_minimum_required(VERSION 3.16)

#
# PROJECT name and version
#

project(tegia-node VERSION 2.0.0 LANGUAGES CXX)

option(USE_SYSTEM_MYSQL "Use system libmysql" ON)

#
# SET C++20 standard
#

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# PROD
# set(TEGIA_FLAGS "-rdynamic -march=native -m64 -Og")

# DEV
set(TEGIA_FLAGS -rdynamic -march=native -m64 -Og -g -Wpedantic -Wshadow=compatible-local -Wl,--no-as-needed)

# DEBUG
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address,leak")

if (NOT MSVC)
    add_compile_options(${TEGIA_FLAGS})
endif()

# SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} ${TEGIA_FLAGS}")


#
# BUILD
#

file(GLOB_RECURSE TEGIA_SOURCES "./src/*.cpp")
add_executable(${PROJECT_NAME} ${TEGIA_SOURCES})

#
# INCLUDE
#

target_include_directories(${PROJECT_NAME} PRIVATE ./include)
# target_include_directories(${PROJECT_NAME} PRIVATE ../vendors)

# HEADER-ONLY LIBRARY
include(FetchContent)
FetchContent_Declare(
    xml2json
    GIT_REPOSITORY https://github.com/Cheedoong/xml2json.git
)
FetchContent_MakeAvailable(xml2json)

add_library(xml2json::xml2json INTERFACE IMPORTED)

target_include_directories(xml2json::xml2json INTERFACE
  "${xml2json_SOURCE_DIR}/"
)
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/xml2json/include)
file(GENERATE OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xml2json/include/xml2json.hpp
    CONTENT "#include <${xml2json_SOURCE_DIR}/include/xml2json.hpp>"
)
#file(CREATE_LINK ${xml2json_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/xml2json/ RESULT link_res COPY_ON_ERROR SYMBOLIC)
target_include_directories(xml2json::xml2json INTERFACE
    "${CMAKE_CURRENT_BINARY_DIR}"
)
message(STATUS "xml2json source dir: ${xml2json_SOURCE_DIR}")
message(STATUS "link result: ${link_res}")+
# target_include_directories(${PROJECT_NAME} PRIVATE /usr/include/libxml2)

#
# GLOBAL LIBRARY
#

find_package(PkgConfig REQUIRED)

find_package(ZLIB REQUIRED)
find_package(Threads REQUIRED)
find_package(CURL REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(LibXml2 REQUIRED)
pkg_check_modules(UUID uuid IMPORTED_TARGET)


if(DEFINED VCPKG_TARGET_TRIPLET OR NOT USE_SYSTEM_MYSQL)
    find_package(unofficial-libmysql REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE unofficial::libmysql::libmysql)
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE mysqlclient)
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads) #pthread)
target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_DL_LIBS}) #dl)
target_link_libraries(${PROJECT_NAME} PRIVATE PkgConfig::UUID) #uuid)
target_link_libraries(${PROJECT_NAME} PRIVATE ZLIB::ZLIB) #z)
target_link_libraries(${PROJECT_NAME} PRIVATE CURL::libcurl) #curl)
target_link_libraries(${PROJECT_NAME} PRIVATE OpenSSL::Crypto)
target_link_libraries(${PROJECT_NAME} PRIVATE LibXml2::LibXml2)
target_link_libraries(${PROJECT_NAME} PRIVATE xml2json::xml2json)

#
# VCPKG LIBRARY
#

find_package(fmt CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE fmt::fmt)

find_package(nlohmann_json CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE nlohmann_json::nlohmann_json)

find_package(nlohmann_json_schema_validator REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE nlohmann_json_schema_validator::validator)

find_package(cpp-jwt REQUIRED)
# add_subdirectory(../vendors/cpp-jwt)
target_link_libraries(${PROJECT_NAME} PRIVATE cpp-jwt::cpp-jwt)
