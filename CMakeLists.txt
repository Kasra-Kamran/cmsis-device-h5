# To successfully include this file in your cmake structure, consider the following:
# Optional Variables:
#   STM32H5_CMSIS_STARTUP_FILE Specifies the startup file of STM32H5XX MCU, otherwise start-up file will be automatically chosen based on part number
#   STM32H5_CMSIS_DOXYGEN_OUTPUT_DIRECTORY Could be used for overriding the default doxygen output directory of STM32H5
# Required Variables:
#   STM32H5_CMSIS_DEVICE MCU full part number
 
set(STM32H5_CMSIS_BASE ${CMAKE_SOURCE_DIR}/external/CMSIS_5/CMSIS)
set(STM32H5_CMSIS_DEVICE STM32H573xx)
 
get_filename_component(STM32H5_CMSIS_BASE "${CMAKE_CURRENT_LIST_DIR}" REALPATH)
 
set(STM32H5_CMSIS_STARTUP_BASE ${STM32H5_CMSIS_BASE}/Source/Templates/gcc)
set(STM32H5_CMSIS_CURRENT_LEN 0)
 
# Mcu full part number in lower case
string(TOLOWER ${STM32H5_CMSIS_DEVICE} STM32H5_CMSIS_DEVICE_LOWER)
# Determine MCU main part number in upper and lower case
string(REGEX MATCH "^(STM32H5[0-9][0-9])" STM32H5_CMSIS_MCU_TYPE_UPPER "${STM32H5_CMSIS_DEVICE}")
string(TOLOWER ${STM32H5_CMSIS_MCU_TYPE_UPPER} STM32H5_CMSIS_MCU_TYPE_LOWER)
 
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm")
  # Find startup file
  if(EXISTS "${STM32H5_CMSIS_STARTUP_FILE}")
  elseif(EXISTS ${STM32H5_CMSIS_STARTUP_BASE}/startup_${STM32H5_CMSIS_DEVICE_LOWER}.s)
    # Simple solution (direct match)
    set(STM32H5_CMSIS_STARTUP_FILE ${STM32H5_CMSIS_STARTUP_BASE}/startup_${STM32H5_CMSIS_DEVICE_LOWER}.s)
  else()
    # Complicated solution, match family and chip revision
    file(GLOB STM32H5_CMSIS_STARTUP_FILES RELATIVE ${STM32H5_CMSIS_STARTUP_BASE} ${STM32H5_CMSIS_STARTUP_BASE}/startup_${STM32H5_CMSIS_MCU_TYPE_LOWER}*.s)
    foreach(FILE ${STM32H5_CMSIS_STARTUP_FILES})
      string(REGEX REPLACE "\\.[^.]*$" "" STM32H5_CMSIS_TEST_FILE ${FILE})
      string(REGEX REPLACE "x" "[a-zA-Z]" STM32H5_CMSIS_TEST_FILE ${STM32H5_CMSIS_TEST_FILE})
      set(STM32H5_CMSIS_TEST_MATCH false)
      string(REGEX MATCH "^(${STM32H5_CMSIS_TEST_FILE})" STM32H5_CMSIS_TEST_MATCH "startup_${STM32H5_CMSIS_DEVICE_LOWER}.s")
      if(STM32H5_CMSIS_TEST_MATCH)
        string(LENGTH ${FILE} STM32H5_CMSIS_FILE_LEN)
        if(${STM32H5_CMSIS_FILE_LEN} GREATER ${STM32H5_CMSIS_CURRENT_LEN}})
          set(STM32H5_CMSIS_STARTUP_FILE ${STM32H5_CMSIS_STARTUP_BASE}/${FILE})
          string(LENGTH ${FILE} STM32H5_CMSIS_CURRENT_LEN)
        endif()
      endif()
    endforeach(FILE)
  endif()
 
  if(EXISTS "${STM32H5_CMSIS_STARTUP_FILE}")
    string(REPLACE ${PROJECT_SOURCE_DIR} "" STM32H5_CMSIS_STARTUP_FILE_NO_PROJECT_DIR ${STM32H5_CMSIS_STARTUP_FILE})
    message("    Startup file: ${STM32H5_CMSIS_STARTUP_FILE_NO_PROJECT_DIR}")
  else()
    message(FATAL_ERROR "Startup file ${STM32H5_CMSIS_STARTUP_FILE} not found")
  endif()
 
  # Set system file name
  set(STM32H5_CMSIS_SYSTEM_FILE ${STM32H5_CMSIS_BASE}/Source/Templates/system_stm32h5xx.c)
 
  add_library(stm32h5-cmsis
    ${STM32H5_CMSIS_STARTUP_FILE}
    ${STM32H5_CMSIS_SYSTEM_FILE}
  )
 
  target_include_directories(stm32h5-cmsis SYSTEM
    PUBLIC
    ${STM32H5_CMSIS_BASE}/Include
  )
 
  target_link_libraries(stm32h5-cmsis
    PUBLIC
    cmsis
    stm32h5xx-hal
  )
 
  target_compile_options(stm32h5-cmsis
    PUBLIC
    -DSTM32H5 -D${STM32H5_CMSIS_MCU_TYPE_UPPER}xx
  )
 
  target_compile_definitions(stm32h5-cmsis INTERFACE
    STM32H573xx
  )
 
  target_compile_definitions(stm32h5-cmsis INTERFACE
    USE_HAL_DRIVER
  )
 
else()
 
  add_library(stm32h5-cmsis INTERFACE
  )
 
  target_include_directories(stm32h5-cmsis SYSTEM
    INTERFACE
    ${STM32H5_CMSIS_BASE}/Include
  )
 
  target_link_libraries(stm32h5-cmsis
    INTERFACE
    cmsis
    stm32h5xx-hal
  )
 
  target_compile_options(stm32h5-cmsis
    INTERFACE
    -DSTM32H5 -D${STM32H5_CMSIS_MCU_TYPE_UPPER}xx
  )
 
  target_compile_definitions(stm32h5-cmsis INTERFACE
    STM32H573xx
  )
 
  target_compile_definitions(stm32h5-cmsis INTERFACE
    USE_HAL_DRIVER
  )
 
endif()