cmake_minimum_required(VERSION 2.8)
project(DLoopDetector)

# Peter for windows
# boost
if (MSVC)
set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_STATIC_RUNTIME OFF) # new idea for prebuilt version from boost.org?
#find_package (Boost COMPONENTS thread system filesystem program_options serialization REQUIRED)
find_package (Boost REQUIRED)
IF (Boost_FOUND)
	include_directories(${Boost_INCLUDE_DIR})
	link_directories(${Boost_LIBRARY_DIRS})
	add_definitions( "-DHAS_BOOST" ) # sure...
ENDIF()
endif()

option(BUILD_DemoBRIEF  "Build demo application with BRIEF features" OFF)
option(BUILD_DemoSURF   "Build demo application with SURF features"  OFF)

set(HDRS
  include/DLoopDetector/DLoopDetector.h         include/DLoopDetector/TemplatedLoopDetector.h)

find_package(OpenCV REQUIRED)
find_package(DLib REQUIRED)
find_package(DBoW2 REQUIRED)

include_directories(include/DLoopDetector/ ${OpenCV_INCLUDE_DIRS} ${DLib_INCLUDE_DIRS} ${DBoW2_INCLUDE_DIRS})

if(BUILD_DemoBRIEF)
  add_executable(demo_brief demo/demo_brief.cpp)
  target_link_libraries(demo_brief ${OpenCV_LIBS} ${DLib_LIBS} ${DBoW2_LIBS})
endif(BUILD_DemoBRIEF)

if(BUILD_DemoSURF)
  add_executable(demo_surf demo/demo_surf.cpp)
  target_link_libraries(demo_surf ${OpenCV_LIBS} ${DLib_LIBS} ${DBoW2_LIBS})
endif(BUILD_DemoSURF)

if(BUILD_DemoBRIEF OR BUILD_DemoSURF)
  set(RESOURCE_FILE ${CMAKE_BINARY_DIR}/resources.tar.gz)
  if(NOT EXISTS ${CMAKE_BINARY_DIR}/resources/)
    if(NOT EXISTS ${RESOURCE_FILE})
      file(DOWNLOAD http://doriangalvez.com/resources/DLoopDetector/resources.tar.gz 
        ${RESOURCE_FILE} STATUS status SHOW_PROGRESS EXPECTED_MD5 c001da68ddaceed1de9c16aaac22ed80)
      if(${status})
        message(FATAL_ERROR "Error downloading resources for demo applications")
      endif(${status})
    endif(NOT EXISTS ${RESOURCE_FILE})
	# Peter
	if (MSVC)
	# somehow unztar from command line
#    execute_process(COMMAND ${CMAKE_COMMAND} -E tar xzf ${RESOURCE_FILE})	
	else()
		execute_process(COMMAND ${CMAKE_COMMAND} -E tar xzf ${RESOURCE_FILE})
		file(REMOVE ${RESOURCE_FILE})
	endif()
  endif(NOT EXISTS ${CMAKE_BINARY_DIR}/resources/)
endif(BUILD_DemoBRIEF OR BUILD_DemoSURF)

configure_file(src/DLoopDetector.cmake.in
  "${PROJECT_BINARY_DIR}/DLoopDetectorConfig.cmake" @ONLY)

install(DIRECTORY include/DLoopDetector DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/DLoopDetectorConfig.cmake"
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME})
install(FILES "${PROJECT_BINARY_DIR}/DLoopDetectorConfig.cmake" 
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/DLoopDetector/)
