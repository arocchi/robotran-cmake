project(robotran-cmake)
cmake_minimum_required(VERSION 2.2)

if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

# we need Matlab to compile the s-function
find_package(Matlab REQUIRED)

# user directories
set(MATLAB_PATH "/usr/local/MATLAB/R2013a" CACHE PATH "Matlab installation path")
set(COMAN_PRJ_NAME "CoMan_LegsCad_23Dof_V4_FB" CACHE PATH "Robotran Project Name")
set(MBS_DIR "/home/arocchi/Documents/MBProjects" CACHE PATH "Path where MBS projects reside")
set(MSYSPAD_DIR "/home/arocchi/.robotran/MBsysLab" CACHE PATH "MBSYSPAD installation dir")
set(COMPILE_DIRDYNARED ON CACHE BOOL "Compile DIRDYNARED or INVDYNARED modules? (can be just one of the two)")

set(MBS_SYMBOLICR "${MBS_DIR}/${COMAN_PRJ_NAME}/symbolicR")
set(MSYSPAD_SOURCE_C "${MSYSPAD_DIR}/mbs_simulink/mbs_sourceC")
set(MSYSPAD_SRC_USER "${MBS_DIR}/${COMAN_PRJ_NAME}/SfunctionsR/src_user")
set(COMAN_PRJ_DIR "${MBS_DIR}/${COMAN_PRJ_NAME}/")

set(SRCS_COMAN_PROJECT  "${MSYSPAD_SOURCE_C}/LocalDataStruct.c"
                        "${MSYSPAD_SOURCE_C}/mbs_close_loops.c"
                        "${MSYSPAD_SOURCE_C}/mbs_compute_model.c"
                        "${MSYSPAD_SOURCE_C}/mbs_dirdynared.c"
                        "${MSYSPAD_SOURCE_C}/mbs_sf_main.c"
                        "${MSYSPAD_SOURCE_C}/MBSdataStruct.c"
                        "${MSYSPAD_SOURCE_C}/MBSsensorStruct.c"
                        "${MSYSPAD_SOURCE_C}/sf_InitCond.c"
                        "${MSYSPAD_SOURCE_C}/sf_IOPort.c")

set(SRCS_COMAN_SYMBOLIC "${MBS_SYMBOLICR}/mbs_cons_hJ_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_cons_jdqd_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_dirdyna_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_extforces_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_gensensor_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_link_${COMAN_PRJ_NAME}.c"
                        "${MBS_SYMBOLICR}/mbs_sensor_${COMAN_PRJ_NAME}.c")

# TODO: find and compile all files in src_extra

set(SRCS_COMAN_USER_rel "user_compute_output.c"
                        "user_cons_hJ.c"
                        "user_cons_jdqd.c"
                        "user_Derivative.c"
                        "user_DrivenJoints.c"
                        "user_ExtForces.c"
                        "user_GroundLevel.c"
                        "user_initialization.c"
                        "user_JointForces.c"
                        "user_Link3Dforces.c"
                        "user_Linkforces.c"
                        "user_sf_IO.c"
                        "user_WheelForces.c"
                        "UserModelStruct.c")

foreach(SRCS_COMAN_USER_file ${SRCS_COMAN_USER_rel})
  list(APPEND SRCS_COMAN_USER "${MSYSPAD_SRC_USER}/${SRCS_COMAN_USER_file}")
endforeach()

# compile as cpp files
#SET_SOURCE_FILES_PROPERTIES( ${SRCS_COMAN_SENSOR} PROPERTIES LANGUAGE CXX )


set(SRCS_COMAN_TOOL_rel choldc.c
                        cholsl.c
                        lubksb.c
                        ludcmp.c
                        lut.c
                        mbs_bakker.c
                        mbs_calspan.c
                        mbs_kine_wheel.c
                        mbs_matrix.c
                        mbs_tool.c
                        norm.c
                        nrutil.c
                        svbksb.c
                        svdcmp.c)

foreach(SRCS_COMAN_TOOL_file ${SRCS_COMAN_TOOL_rel})
  list(APPEND SRCS_COMAN_TOOL "${MSYSPAD_SOURCE_C}/${SRCS_COMAN_TOOL_file}")
endforeach()

include_directories("${MATLAB_PATH}/simulink/include")
include_directories("${MATLAB_PATH}/extern/include")
include_directories("${MSYSPAD_SOURCE_C}")
include_directories("${MBS_SYMBOLICR}")
include_directories("${MSYSPAD_SRC_USER}")

add_definitions( -DMATLAB_MEX_FILE )
if(COMPILE_DIRDYNARED)
    add_definitions( -DDIRDYNARED )
    set(SFUNCTION_OUTPUT_NAME "mbs_sf_dirdynared_${COMAN_PRJ_NAME}")
elseif(COMPILE_INVDYNARED)
    add_definitions( -DINVDYNARED )
    set(SFUNCTION_OUTPUT_NAME "mbs_sf_invdynared_${COMAN_PRJ_NAME}")
endif()

if(APPLE)
    set(SFUNCTION_SUFFIX ".mexmaci64")
elseif(UNIX)
    set(SFUNCTION_SUFFIX ".mexa64")
elseif(WIN32)
    set(SFUNCTION_SUFFIX ".mexw64")
endif()

link_directories(${CMAKE_BINARY_DIR})
link_directories("${MATLAB_PATH}/bin/glnxa64")

# Compile CoMan s-function
add_library(robotran_S_function SHARED
                        ${SRCS_COMAN_TOOL}
                        ${SRCS_COMAN_SYMBOLIC}
                        ${SRCS_COMAN_PROJECT}
                        ${SRCS_COMAN_USER})

set(CMAKE_CXX_FLAGS "-fpermissive")
target_link_libraries(robotran_S_function mx mex mat m)

set_target_properties(  robotran_S_function
                        PROPERTIES
                        OUTPUT_NAME "${SFUNCTION_OUTPUT_NAME}"
                        SUFFIX  "${SFUNCTION_SUFFIX}"
                        PREFIX  "")



