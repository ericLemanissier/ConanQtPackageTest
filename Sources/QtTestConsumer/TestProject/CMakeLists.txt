cmake_minimum_required(VERSION 3.16.0)
project(ConanQtPackageTest)

#SET(CMAKE_CXX_STANDARD 14)
#SET(CMAKE_CXX_STANDARD_REQUIRED ON)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)

function( get_dll_location dllPathOut target)
    string(TOUPPER "${COMPILER_CONFIG}" upperConfig)
    get_property(absDllPath TARGET ${target} PROPERTY IMPORTED_LOCATION_${upperConfig})
    if(NOT absDllPath)
        get_property(absDllPath TARGET ${target} PROPERTY LOCATION_${upperConfig})
    endif()
    set(${dllPathOut} ${absDllPath} PARENT_SCOPE)
endfunction()

conan_basic_setup(TARGETS)
#conan_basic_setup()
message(STATUS "Looking for Qt cmake files at ${CONAN_LIB_DIRS_QT}/cmake/Qt5")
find_package(Qt5 CONFIG REQUIRED COMPONENTS Core Gui Widgets PATHS ${CONAN_LIB_DIRS_QT}/cmake/Qt5 )
message(STATUS "Using qt at ${Qt5_DIR}")


foreach(qtTarget Qt5::Core Qt5::Gui Qt5::Widgets)

    get_dll_location(absDllPath ${qtTarget})
    file(COPY ${absDllPath} DESTINATION ${CMAKE_BINARY_DIR}/bin)

endforeach()

if(${CMAKE_SYSTEM_NAME} STREQUAL Windows)

    get_dll_location(absDllPath Qt5::QWindowsIntegrationPlugin)
    file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/bin/platforms)
    file(COPY ${absDllPath} DESTINATION ${CMAKE_BINARY_DIR}/bin/platforms)

elseif(${CMAKE_SYSTEM_NAME} STREQUAL Linux)

    get_dll_location(absDllPath Qt5::QXcbIntegrationPlugin)
    file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/bin/platforms)
    file(COPY ${absDllPath} DESTINATION ${CMAKE_BINARY_DIR}/bin/platforms)

endif()

# Add the test executable
add_executable(test test.cpp "${CMAKE_BINARY_DIR}/conanbuildinfo.cmake" "${CMAKE_BINARY_DIR}/conanbuildinfo.txt" "${CMAKE_BINARY_DIR}/conaninfo.txt" )
#target_link_libraries(test CONAN_PKG::qt)
target_link_libraries(test Qt5::Widgets)
