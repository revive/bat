set(BAT_SRC BCAux.cxx BCDataSet.cxx BCH1D.cxx BCLog.cxx BCObservable.cxx BCPositiveDefinitePrior.cxx BCTF1LogPrior.cxx BCCauchyPrior.cxx BCEmptyModel.cxx BCH2D.cxx BCMath.cxx BCObservableSet.cxx BCPrior.cxx BCTF1Prior.cxx BCConstantPrior.cxx BCEngineMCMC.cxx BCHistogramBase.cxx BCModel.cxx BCParameter.cxx BCPriorModel.cxx BCTH1Prior.cxx BCDataPoint.cxx BCGaussianPrior.cxx BCIntegrate.cxx BCModelManager.cxx BCParameterSet.cxx BCSplitGaussianPrior.cxx BCVariable.cxx)

file(GLOB headers "${PROJECT_SOURCE_DIR}/BAT/*.h")
#list(APPEND updated_headers "")
foreach(HEADER ${headers})
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

#if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/libBAT_rdict.cxx)
#   ROOT_GENERATE_DICTIONARY(libBAT_rdict
#     ${updated_headers}
#     LINKDEF ${CMAKE_CURRENT_SOURCE_DIR}/Linkdef.h
#)
#endif()

add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/libBAT_rdict.cxx
  COMMAND ${ROOTCINT_EXECUTABLE} -cint -f ${CMAKE_CURRENT_BINARY_DIR}/libBAT_rdict.cxx -c -I${CMAKE_CURRENT_BINARY_DIR}/include ${updated_headers} ${CMAKE_CURRENT_SOURCE_DIR}/LinkDef.h
  DEPENDS ${updated_headers}
  )


#add_custom_target(
#  root_headers ALL
#  DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/libBAT_rdict.cxx
#  )

if (BAT_WITH_CUBA)
  ExternalProject_Get_Property(Cuba INSTALL_DIR)
  include_directories(${INSTALL_DIR}/include)
endif (BAT_WITH_CUBA)

add_library(BAT SHARED ${BAT_SRC} ${CMAKE_CURRENT_BINARY_DIR}/libBAT_rdict.cxx)
target_link_libraries(BAT ${ROOT_LIBRARIES})
set_target_properties(BAT PROPERTIES VERSION ${BAT_VERSION_MAJOR} SOVERSION ${BAT_VERSION})
#create_libtool_file(BAT ${CMAKE_INSTALL_PREFIX}/lib)

if (ENABLE_PARALLELIZATION)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif (ENABLE_PARALLELIZATION)

if (BAT_WITH_CUBA)
  ExternalProject_Get_Property(Cuba INSTALL_DIR)
  include_directories(${INSTALL_DIR}/include)
  target_link_libraries(BAT ${INSTALL_DIR}/lib/libcuba.a)
endif (BAT_WITH_CUBA)

install(TARGETS BAT LIBRARY DESTINATION lib )
install(FILES ${updated_headers} DESTINATION include/BAT)
