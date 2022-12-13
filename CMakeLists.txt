cmake_minimum_required(VERSION 2.8.3)
project(yaml_cpp_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

# Release versions currently don't use C++11 and create more compiler warnings.
set(YAML_CPP_TAG 11607eb5bf1258641d80f7051e7cf09e317b4746)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

if(ANDROID_NREAL)
    string(REPLACE "\"" "" TMP_BUILD_CONFIG ${ANDROID_NREAL_BUILD_CONFIG})
    string(REPLACE " " ";" BUILD_CONFIG ${TMP_BUILD_CONFIG})
endif()

ExternalProject_Add(yaml_cpp_src
  #GIT_REPOSITORY  https://github.com/jbeder/yaml-cpp
  #GIT_TAG ${YAML_CPP_TAG}
  URL file://${CMAKE_SOURCE_DIR}/11607eb5.zip
  PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/extra_version.patch &&
                patch -p0 < ${CMAKE_SOURCE_DIR}/extra_version_again.patch
  CONFIGURE_COMMAND cd ../yaml_cpp_src/ && cmake .
    -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_DEVEL_PREFIX}
    -DBUILD_SHARED_LIBS=OFF
    ${BUILD_CONFIG}
  BUILD_COMMAND cd ../yaml_cpp_src/ && make -j 8
  INSTALL_COMMAND cd ../yaml_cpp_src/ && make install -j 8
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} yaml_cpp_src)
target_link_libraries(${PROJECT_NAME} ${CATKIN_DEVEL_PREFIX}/lib/libyaml-cpp0.5${CMAKE_STATIC_LIBRARY_SUFFIX})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS yaml-cpp-extras.cmake)
