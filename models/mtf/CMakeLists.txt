set(MODELS_MTF_SRC BCMTFAnalysisFacility.cxx  BCMTFChannel.cxx  BCMTFComparisonTool.cxx  BCMTF.cxx  BCMTFProcess.cxx  BCMTFSystematic.cxx  BCMTFSystematicVariation.cxx  BCMTFTemplate.cxx)

file(GLOB headers "${CMAKE_CURRENT_SOURCE_DIR}/*.h")
#list(APPEND updated_headers "")
foreach(HEADER ${headers})
  message(${HEADER})
  get_filename_component(HEADER_BASE ${HEADER} NAME)
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}/${HEADER_BASE}
    DEPENDS ${HEADER}
    COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}
    COMMAND sed 's|include \"\\\(BC.*\\.h\\\)\"|include <${PROJECT_NAME}/\\1>| \; s|include \"\\.\\./\\.\\./\\\(${PROJECT_NAME}/BC.*\\.h\\\)\"|include <\\1>|' ${HEADER} > ${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}/${HEADER_BASE}
    )
  list(APPEND updated_headers ${CMAKE_CURRENT_BINARY_DIR}/include/${PROJECT_NAME}/${HEADER_BASE})
endforeach()
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/../../src/include)

add_library(BATmtf SHARED ${MODELS_MTF_SRC})
target_link_libraries(BATmtf ${ROOT_LIBRARIES} BAT)
set_target_properties(BATmtf PROPERTIES VERSION ${BAT_VERSION_MAJOR} SOVERSION ${BAT_VERSION})

install(TARGETS BATmtf LIBRARY DESTINATION lib)

