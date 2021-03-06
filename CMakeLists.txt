cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(test_travis)
message(STATUS "CMake version: " ${CMAKE_VERSION})

if(${CMAKE_VERSION} VERSION_LESS 3.12)
  find_package(PythonInterp 3.4)
  if(NOT PYTHONINTERP_FOUND)
    unset(PYTHON_EXECUTABLE CACHE)
    find_package(PythonInterp 2.7 REQUIRED)
  endif(NOT PYTHONINTERP_FOUND)
  set(python_version_mm "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")
else(${CMAKE_VERSION} VERSION_LESS 3.12)
  find_package(Python3 3.4 COMPONENTS Interpreter)
  if(Python3_FOUND)
    set(PYTHON_EXECUTABLE ${Python3_EXECUTABLE})
    set(python_version_mm "${Python3_VERSION_MAJOR}.${Python3_VERSION_MINOR}")
  else(Python3_FOUND)
    find_package(Python2 2.7 COMPONENTS Interpreter REQUIRED)
    set(PYTHON_EXECUTABLE ${Python2_EXECUTABLE})
    set(python_version_mm "${Python2_VERSION_MAJOR}.${Python2_VERSION_MINOR}")
  endif(Python3_FOUND)
endif(${CMAKE_VERSION} VERSION_LESS 3.12)

include(${PROJECT_SOURCE_DIR}/config/FindPythonModule.cmake)
set(pythonpath_prefix "${CMAKE_CURRENT_BINARY_DIR}/pythonmodule")
set(pythonpath_build "${pythonpath_prefix}/lib/python${python_version_mm}/site-packages")
set(ENV{PYTHONPATH} "${pythonpath_build}:$ENV{PYTHONPATH}")
set(ENV{PATH} "${pythonpath_prefix}/bin:$ENV{PATH}")

set(module_names sphinx)
set(sphinxcontribs spelling)

foreach(module_name ${module_names})
  find_python_module(${module_name})
  string(TOUPPER ${module_name} uppername)
  if(NOT ${uppername}_FOUND)
    set(missing_modules ${missing_modules} ${module_name})
  endif(NOT ${uppername}_FOUND)
endforeach()
foreach(module_name ${sphinxcontribs})
  find_python_module("sphinxcontrib.${module_name}")
  string(TOUPPER ${module_name} uppername)
  if(NOT SPHINXCONTRIB.${uppername}_FOUND)
    set(missing_modules ${missing_modules} "sphinxcontrib-${module_name}")
  endif()
endforeach()
list(LENGTH missing_modules num_missing_modules)
if(num_missing_modules)
  find_python_module(pip)
  if(NOT PIP_FOUND)
    file(DOWNLOAD https://bootstrap.pypa.io/get-pip.py "${CMAKE_CURRENT_BINARY_DIR}/get-pip.py")
    execute_process(COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/get-pip.py --prefix=${pythonpath_prefix})
    execute_process(COMMAND ${PYTHON_EXECUTABLE} -m pip install -U pip)
  endif(NOT PIP_FOUND)
  foreach(module_name ${missing_modules})
    message(STATUS "${PYTHON_EXECUTABLE} -m pip install --prefix=${pythonpath_prefix} ${module_name}")
    execute_process(
      COMMAND ${PYTHON_EXECUTABLE} -m pip install --prefix=${pythonpath_prefix} ${module_name}
      )
  endforeach(module_name ${missing_modules})
endif(num_missing_modules)

add_subdirectory(src)

enable_testing()
add_subdirectory(test)

add_subdirectory(doc)
