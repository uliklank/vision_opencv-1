# add library
add_library(${PROJECT_NAME} cv_bridge.cpp)
add_dependencies(${PROJECT_NAME} sensor_msgs_gencpp)
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES} ${catkin_LIBRARIES})


install(TARGETS ${PROJECT_NAME} DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

# add a Boost Python library
find_package(PythonInterp REQUIRED)
find_package(PythonLibs "${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR}")

#Get the numpy include directory from its python module
if(NOT PYTHON_NUMPY_INCLUDE_DIR)
    execute_process(COMMAND ${PYTHON_EXECUTABLE} -c "import numpy; print(numpy.get_include())"
                    RESULT_VARIABLE PYTHON_NUMPY_PROCESS
                    OUTPUT_VARIABLE PYTHON_NUMPY_INCLUDE_DIR
                    OUTPUT_STRIP_TRAILING_WHITESPACE)

    if(PYTHON_NUMPY_PROCESS EQUAL 0)
       file(TO_CMAKE_PATH "${PYTHON_NUMPY_INCLUDE_DIR}" PYTHON_NUMPY_INCLUDE_CMAKE_PATH)
       set(PYTHON_NUMPY_INCLUDE_DIR ${PYTHON_NUMPY_INCLUDE_CMAKE_PATH} CACHE PATH "Numpy include directory")
    else(PYTHON_NUMPY_PROCESS EQUAL 0)
        message(SEND_ERROR "Could not determine the NumPy include directory, verify that NumPy was installed correctly.")
    endif(PYTHON_NUMPY_PROCESS EQUAL 0)
 endif(NOT PYTHON_NUMPY_INCLUDE_DIR)

include_directories(SYSTEM ${PYTHON_INCLUDE_PATH}
                           ${Boost_INCLUDE_DIRS}
                           ${PYTHON_NUMPY_INCLUDE_DIR} # cv_bridge module uses NumPy functions
)

if (PYTHON_VERSION_MAJOR VERSION_EQUAL 3)
  list( APPEND CV_BRIDGE_COMPILE_FLAGS "PYTHON3" )
  list( APPEND CV_BRIDGE_BOOST_COMPILE_FLAGS "PYTHON3" )
endif()

list( APPEND CV_BRIDGE_COMPILE_FLAGS "CV_BRIDGE_INTERNAL" )

set_target_properties( ${PROJECT_NAME}
    PROPERTIES
        COMPILE_DEFINITIONS "${CV_BRIDGE_COMPILE_FLAGS}" )

link_directories( "C:/Program Files/boost/boost_1_47/lib" )

add_library(${PROJECT_NAME}_boost module.cpp)

target_link_libraries(${PROJECT_NAME}_boost ${Boost_LIBRARIES}
                                            ${catkin_LIBRARIES}
                                            ${PYTHON_LIBRARIES}
                                            ${PROJECT_NAME}
)

set_target_properties( ${PROJECT_NAME}_boost
    PROPERTIES
        COMPILE_DEFINITIONS "${CV_BRIDGE_BOOST_COMPILE_FLAGS}" )

set_target_properties(${PROJECT_NAME}_boost PROPERTIES
                      LIBRARY_OUTPUT_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}/${PROJECT_NAME}/boost/
                      PREFIX ""
)
if(APPLE)
  set_target_properties(${PROJECT_NAME}_boost PROPERTIES
                        SUFFIX ".so")
endif()

install(TARGETS ${PROJECT_NAME}_boost DESTINATION ${CATKIN_PACKAGE_PYTHON_DESTINATION}/boost)
