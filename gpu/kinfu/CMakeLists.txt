set(SUBSYS_NAME gpu_kinfu)
set(SUBSYS_PATH gpu/kinfu)
set(SUBSYS_DESC "Kinect Fusion implementation")
set(SUBSYS_DEPS common io gpu_utils gpu_containers geometry search)
if(${CUDA_VERSION_STRING} VERSION_GREATER_EQUAL "12.0")
  set(DEFAULT TRUE)
else()
  set(DEFAULT FALSE)
  set(REASON "Updating Kinfu to CUDA 12 no backwards compat is warrantied")
endif()

PCL_SUBSYS_OPTION(build "${SUBSYS_NAME}" "${SUBSYS_DESC}" ${DEFAULT} "${REASON}")
PCL_SUBSYS_DEPEND(build NAME ${SUBSYS_NAME} DEPS ${SUBSYS_DEPS})
PCL_SET_SUBSYS_INCLUDE_DIR("${SUBSYS_NAME}" "${SUBSYS_PATH}")
mark_as_advanced("BUILD_${SUBSYS_NAME}")

if(NOT build)
  return()
endif()

file(GLOB incs include/pcl/gpu/kinfu/*.h*)
file(GLOB srcs src/*.cpp src/*.h*)
file(GLOB cuda src/cuda/*.cu src/cuda/*.h*)

source_group("Source Files\\cuda" FILES ${cuda})
source_group("Source Files" FILES ${srcs})

set(LIB_NAME "pcl_${SUBSYS_NAME}")

PCL_CUDA_ADD_LIBRARY(${LIB_NAME} COMPONENT ${SUBSYS_NAME} SOURCES ${srcs} ${incs} ${cuda})
target_link_libraries(${LIB_NAME} pcl_gpu_utils pcl_gpu_containers)
target_include_directories(${LIB_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

target_compile_options(${LIB_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:--ftz=true;--prec-div=false;--prec-sqrt=false>)

PCL_MAKE_PKGCONFIG(${LIB_NAME} COMPONENT ${SUBSYS_NAME} DESC ${SUBSYS_DESC} PCL_DEPS ${SUBSYS_DEPS} EXT_DEPS ${EXT_DEPS})

# Install include files
PCL_ADD_INCLUDES("${SUBSYS_NAME}" "${SUBSYS_PATH}" ${incs})

if(BUILD_tools)
  add_subdirectory(tools)
endif()
