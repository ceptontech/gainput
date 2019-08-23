project(gainput)
message(STATUS "GAINPUT version ${GAINPUT_VERSION}")

set(CMAKE_MACOSX_RPATH 1)

if(CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++98 -Wall -Wextra -pedantic -Wshadow -Wno-variadic-macros")
endif()

include_directories (include/)

file(GLOB_RECURSE sources source/*.cpp source/*.h include/*.h)

if(APPLE)
	file(GLOB_RECURSE mmsources source/*.mm)
endif()

## build STATIC *or* SHARED
if (GAINPUT_BUILD_SHARED)
  message(STATUS "..Building shared libraries (-DGAINPUT_BUILD_SHARED=OFF to disable)")
  add_library(gainput SHARED ${sources} ${mmsources})
  set_target_properties(gainput PROPERTIES
    OUTPUT_NAME gainput
    VERSION ${GAINPUT_VERSION}
    SOVERSION ${GAINPUT_MAJOR_VERSION}
    FOLDER gainput
  )
  set(install_libs ${install_libs} gainput)
endif (GAINPUT_BUILD_SHARED)

if (GAINPUT_BUILD_STATIC)
  message(STATUS "..Building static libraries (-DGAINPUT_BUILD_STATIC=OFF to disable)")
  add_library(gainputstatic STATIC ${sources} ${mmsources})
  set_target_properties(gainputstatic PROPERTIES FOLDER gainput)
  set(install_libs ${install_libs} gainputstatic)
endif (GAINPUT_BUILD_STATIC)

if(WIN32)
  add_definitions(-DGAINPUT_LIB_DYNAMIC=1)
elseif(APPLE)
  find_library(FOUNDATION Foundation)
  find_library(IOKIT IOKit)
  find_library(GAME_CONTROLLER GameController)
  if(IOS)
      find_library(UIKIT UIKit)
      find_library(COREMOTION CoreMotion)
      find_library(QUARTZCORE QuartzCore)
  else()
      find_library(APPKIT AppKit)
  endif()
endif()
foreach(name IN LISTS install_libs)
  if(WIN32)
  	target_link_libraries(${name} ${XINPUT} ws2_32)
  elseif(ANDROID)
  	target_link_libraries(${name} native_app_glue log android)
  elseif(APPLE)
  	target_link_libraries(${name} ${FOUNDATION} ${IOKIT} ${GAME_CONTROLLER})
    if(IOS)
        target_link_libraries(${name} ${UIKIT} ${COREMOTION})
    else()
        target_link_libraries(${name} ${APPKIT})
    endif()
  endif()
endforeach()

# Library installation directory
if(NOT DEFINED CMAKE_INSTALL_LIBDIR)
    set(CMAKE_INSTALL_LIBDIR lib)
endif(NOT DEFINED CMAKE_INSTALL_LIBDIR)
set(libdir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})

install(
    DIRECTORY "include/gainput"
    DESTINATION "include"
    FILES_MATCHING PATTERN "*.h"
)

install(
    TARGETS ${install_libs}
    LIBRARY DESTINATION "${libdir}"
    ARCHIVE DESTINATION "${libdir}"
    RUNTIME DESTINATION "bin"
)
