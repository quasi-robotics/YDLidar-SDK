#cmake_policy(SET CMP0078 NEW)
#cmake_policy(SET CMP0086 OLD)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
add_definitions(-std=c++11) # Use C++11

#Include directories
INCLUDE_DIRECTORIES(
     ${CMAKE_SOURCE_DIR}
     ${CMAKE_SOURCE_DIR}/../
     ${CMAKE_CURRENT_BINARY_DIR}
     ${CMAKE_BINARY_DIR}
)
SET(MODULE_NAME ydlidar)

#################################3
add_subdirectory(examples)
############################################################################
# build ydlidar sdk python version
###############################################################################
# This is a CMake YDLIDAR SDK for Python
INCLUDE(${SWIG_USE_FILE})
INCLUDE_DIRECTORIES(${Python3_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
SET(CMAKE_SWIG_FLAGS "")
#set(CMAKE_SWIG_OUTDIR "${CMAKE_BINARY_DIR}")
SET(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR})
set_property(SOURCE ydlidar_sdk.i PROPERTY SWIG_FLAGS "-module" "ydlidar")
SET_SOURCE_FILES_PROPERTIES(ydlidar_sdk.i PROPERTIES CPLUSPLUS ON)
###############################################################################
# This is a CMake YDLIDAR SDK for Python
swig_add_library(${MODULE_NAME} LANGUAGE python SOURCES ydlidar_sdk.i)
target_link_libraries(${SWIG_MODULE_${MODULE_NAME}_REAL_NAME} Python3::Module ydlidar_sdk)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/__init__.py "__all__ = ['${MODULE_NAME}']")

add_test(NAME ydlidar_py_test COMMAND
         ${Python3_EXECUTABLE}
         "${PROJECT_SOURCE_DIR}/python/test/pytest.py")
set_property(TEST ydlidar_py_test PROPERTY ENVIRONMENT
             "PYTHONPATH=$ENV{PYTHONPATH}:${PROJECT_BINARY_DIR}/python")

# distutils was removed in Python 3.12; use sysconfig instead. Debian/Ubuntu
# provide the 'deb_system' scheme, which matches what distutils used to return.
execute_process(COMMAND "${Python3_EXECUTABLE}" -c "if True:
                  import sysconfig
                  scheme = 'deb_system' if 'deb_system' in sysconfig.get_scheme_names() else 'posix_prefix'
                  print(sysconfig.get_path('platlib', scheme, vars={'base': '', 'platbase': ''}).lstrip('/'))"
                OUTPUT_VARIABLE PYTHON_SITE
                OUTPUT_STRIP_TRAILING_WHITESPACE)
# Install the wrapper.
install(TARGETS ${SWIG_MODULE_${MODULE_NAME}_REAL_NAME} DESTINATION ${PYTHON_SITE})
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${MODULE_NAME}.py" DESTINATION ${PYTHON_SITE})
