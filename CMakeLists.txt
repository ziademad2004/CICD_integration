cmake_minimum_required(VERSION 3.10)

project(ProfessionalTicTacToe VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Enable testing
enable_testing()

# Try to find Qt6 first, fall back to Qt5 if not found
find_package(Qt6 COMPONENTS Core Gui Widgets Test QUIET)
if (NOT Qt6_FOUND)
    find_package(Qt5 COMPONENTS Core Gui Widgets Test REQUIRED)
    message(STATUS "Using Qt5")
else()
    message(STATUS "Using Qt6")
endif()

set(SOURCES
    src/main.cpp
    src/mainwindow.cpp
    src/gamelogic.cpp
    src/gamehistory.cpp
    src/authentication.cpp
    src/user.cpp
    src/database.cpp
    src/aiopponent.cpp
)

set(HEADERS
    include/mainwindow.h
    include/gamelogic.h
    include/gamehistory.h
    include/authentication.h
    include/user.h
    include/database.h
    include/aiopponent.h
)

set(RESOURCES
    resources/resources.qrc
)

# Create a library for testing (excluding main.cpp)
set(LIB_SOURCES
    src/mainwindow.cpp
    src/gamelogic.cpp
    src/gamehistory.cpp
    src/authentication.cpp
    src/user.cpp
    src/database.cpp
    src/aiopponent.cpp
)

add_library(TicTacToeLib STATIC ${LIB_SOURCES} ${HEADERS})
target_include_directories(TicTacToeLib PUBLIC include)

# Link Qt libraries to the library
if (Qt6_FOUND)
    target_link_libraries(TicTacToeLib PUBLIC
        Qt6::Core
        Qt6::Gui
        Qt6::Widgets
    )
else()
    target_link_libraries(TicTacToeLib PUBLIC
        Qt5::Core
        Qt5::Gui
        Qt5::Widgets
    )
endif()

# Main executable
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${RESOURCES}
    .github/workflows/ci.yml)
target_include_directories(${PROJECT_NAME} PRIVATE include)

# Link against Qt5 or Qt6 depending on what was found
if (Qt6_FOUND)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt6::Core
        Qt6::Gui
        Qt6::Widgets
    )
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt5::Core
        Qt5::Gui
        Qt5::Widgets
    )
endif()

if(WIN32)
    set_target_properties(${PROJECT_NAME} PROPERTIES WIN32_EXECUTABLE TRUE)
endif()

# Function to create test executables
function(create_test test_name test_source)
    add_executable(${test_name} ${test_source})
    target_include_directories(${test_name} PRIVATE include)
    target_link_libraries(${test_name} TicTacToeLib)

    # Ensure the test file is processed by MOC
    set_target_properties(${test_name} PROPERTIES AUTOMOC TRUE)

    if (Qt6_FOUND)
        target_link_libraries(${test_name} Qt6::Test Qt6::Core)
    else()
        target_link_libraries(${test_name} Qt5::Test Qt5::Core)
    endif()

    # Set output directory to build root
    set_target_properties(${test_name} PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
        RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}
        RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}
    )

    # Register with CTest
    add_test(NAME ${test_name} COMMAND ${test_name})
    set_tests_properties(${test_name} PROPERTIES WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
endfunction()

# Create test executables
create_test(test_gamelogic tests/test_gamelogic.cpp)
create_test(test_authentication tests/test_authentication.cpp)
create_test(test_user tests/test_user.cpp)
create_test(test_aiopponent tests/test_aiopponent.cpp)
create_test(test_database tests/test_database.cpp)

# Define install directories if not already defined
include(GNUInstallDirs)

install(TARGETS ${PROJECT_NAME}
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

message(STATUS "=== Configuration Summary ===")
message(STATUS "Project: ${PROJECT_NAME} v${PROJECT_VERSION}")
message(STATUS "Build Directory: ${CMAKE_BINARY_DIR}")
message(STATUS "Test executables will be in: ${CMAKE_BINARY_DIR}")
message(STATUS "=============================")

# Add version information
set(PROJECT_VERSION_MAJOR 1)
set(PROJECT_VERSION_MINOR 0)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

# CPack configuration for releases
set(CPACK_PACKAGE_NAME "ProfessionalTicTacToe")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Advanced Tic Tac Toe Game")
set(CPACK_PACKAGE_VENDOR "embedded team")
set(CPACK_PACKAGE_CONTACT "ziad2004624@gmail.com")

if(WIN32)
    set(CPACK_GENERATOR "NSIS")
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop")
else()
    set(CPACK_GENERATOR "DEB")
endif()

include(CPack)
