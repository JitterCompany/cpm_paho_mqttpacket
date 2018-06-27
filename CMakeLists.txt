cmake_minimum_required(VERSION 3.5.0 FATAL_ERROR)


set(CPM_MODULE_NAME cpm_paho_mqttpacket)

#-----------------------------------------------------------------------
# Required CPM Setup - no need to modify - See: https://github.com/iauns/cpm
#-----------------------------------------------------------------------
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
    set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
    include(CPM)
else()
    set (CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
    find_package(Git)
    if(NOT GIT_FOUND)
        message(FATAL_ERROR "CPM requires Git.")
    endif()
    if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
        message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
        execute_process(
            COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
            RESULT_VARIABLE error_code
            OUTPUT_QUIET ERROR_QUIET)
        if(error_code)
            message(FATAL_ERROR "CPM failed to get the hash for HEAD")
        endif()
    endif()
    include(${CPM_DIR}/CPM.cmake)
endif()

#-----------------------------------------------------------------------
# Project sources / settings
#-----------------------------------------------------------------------

# this is required because this is an 'external' module
CPM_ForceOnlyOneModuleVersion()


set(SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/external/paho_mqtt.src")

CPM_EnsureRepoIsCurrent(
    TARGET_DIR ${SOURCE_DIR}
    GIT_REPOSITORY "https://github.com/eclipse/paho.mqtt.embedded-c.git"
    GIT_TAG "v1.1.0"
    USE_CACHING TRUE
    )

# allow including as <paho_mqtt/MQTTPacket.h>
set(INC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/paho_mqtt")
execute_process(
    COMMAND ln -s ${SOURCE_DIR}/MQTTPacket/src "${INC_DIR}"
    )  

file(GLOB SOURCES 
    "${SOURCE_DIR}/MQTTPacket/src/*.c"
    )

add_library(${CPM_LIB_TARGET_NAME} ${SOURCES})
target_link_libraries(${CPM_LIB_TARGET_NAME} ${CPM_LIBRARIES})
target_link_libraries(${CPM_LIB_TARGET_NAME} c)

CPM_InitModule(${CPM_MODULE_NAME})

