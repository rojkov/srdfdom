cmake_minimum_required(VERSION 2.8.3)
project(srdfdom)

find_package(Boost REQUIRED)
include_directories(SYSTEM ${Boost_INCLUDE_DIR})

find_package(console_bridge REQUIRED)
find_package(urdfdom_headers REQUIRED)

find_package(catkin REQUIRED COMPONENTS cmake_modules urdfdom_py)

find_package(TinyXML REQUIRED)

include_directories(include ${TinyXML_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS} ${console_bridge_INCLUDE_DIRS} ${urdfdom_headers_INCLUDE_DIRS})
link_directories(${catkin_LIBRARY_DIRS})


### Maintain compatibility to old urdfdom_headers (version < 4.0)
if( "0.4.0" VERSION_GREATER "${urdfdom_headers_VERSION}")
  set(HAVE_URDFDOM_4 0)
else()
  set(HAVE_URDFDOM_4 1)
endif()
set(generated_compat_header "${CATKIN_DEVEL_PREFIX}/include/${PROJECT_NAME}/urdfdom_compatibility.h")
include_directories("${CATKIN_DEVEL_PREFIX}/include")
configure_file(urdfdom_compatibility.h.in "${generated_compat_header}" @ONLY)


catkin_python_setup()

catkin_package(
  LIBRARIES ${PROJECT_NAME}
  INCLUDE_DIRS include ${TinyXML_INCLUDE_DIRS} ${CATKIN_DEVEL_PREFIX}/include
  DEPENDS console_bridge urdfdom_headers urdfdom_py
)

add_library(${PROJECT_NAME} 
  src/model.cpp
  src/srdf_writer.cpp
)
target_link_libraries(${PROJECT_NAME} ${TinyXML_LIBRARIES} ${catkin_LIBRARIES} ${console_bridge_LIBRARIES} ${urdfdom_headers_LIBRARIES})


install(TARGETS ${PROJECT_NAME}
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)
install(FILES ${generated_compat_header} DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

install(PROGRAMS 
  scripts/display_srdf
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

if(CATKIN_ENABLE_TESTING)
  find_package(rostest REQUIRED)
  add_rostest(test/srdf_parser.test)
endif()
