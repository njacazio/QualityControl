# ---- Library ----

add_library(QcITS
            src/ITSRawTask.cxx
            src/TrendingTaskITSThr.cxx
            src/TrendingTaskITSFhr.cxx
            src/TrendingTaskConfigITS.cxx
            src/TH2XlineReductor.cxx
            src/ITSFhrTask.cxx
            src/ITSFeeTask.cxx
            src/ITSClusterTask.cxx
            src/ITSTrackTask.cxx
            src/ITSFhrCheck.cxx
            src/ITSClusterCheck.cxx
            src/ITSTrackCheck.cxx
            src/ITSFeeCheck.cxx
            )

target_sources(QcITS PRIVATE src/TH2XlineReductor.cxx)

target_include_directories(
  QcITS
  PUBLIC $<INSTALL_INTERFACE:include>
         $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

include_directories(${O2_ROOT}/include/GPU)

target_link_libraries(QcITS PUBLIC QualityControl O2::ITSBase O2::ITSMFTBase O2::ITSMFTReconstruction ROOT::Hist)

install(TARGETS QcITS
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

# ---- ROOT dictionary  ----

add_root_dictionary(QcITS
                    HEADERS include/ITS/ITSRawTask.h
                            include/ITS/TrendingTaskITSThr.h
                            include/ITS/TrendingTaskITSFhr.h
                            include/ITS/TH2XlineReductor.h
                            include/ITS/ITSFhrTask.h
                            include/ITS/ITSFeeTask.h
                            include/ITS/ITSClusterTask.h
                            include/ITS/ITSTrackTask.h
                            include/ITS/ITSFhrCheck.h
                            include/ITS/ITSClusterCheck.h
                            include/ITS/ITSTrackCheck.h
                            include/ITS/ITSFeeCheck.h
                    LINKDEF include/ITS/LinkDef.h
                    BASENAME QcITS)

# ---- Test(s) ----

add_executable(testQcITS test/testITS.cxx)
target_link_libraries(testQcITS PRIVATE QcITS Boost::unit_test_framework O2::ITSMFTReconstruction O2::ITSBase O2::DetectorsBase QualityControl)
add_test(NAME testQcITS COMMAND testQcITS)
set_tests_properties(testQcITS PROPERTIES TIMEOUT 60)
install(
  TARGETS testQcITS
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# ---- Executables ----

set(EXE_SRCS
    src/runITS.cxx)

set(EXE_NAMES
    o2-qc-run-its)

# These were the original names before the convention changed. We will get rid
# of them but for the time being we want to create symlinks to avoid confusion.
set(EXE_OLD_NAMES
    qcRunITS)

list(LENGTH EXE_SRCS count)
math(EXPR count "${count}-1")
foreach(i RANGE ${count})
  list(GET EXE_SRCS ${i} src)
  list(GET EXE_NAMES ${i} name)
  list(GET EXE_OLD_NAMES ${i} oldname)
  add_executable(${name} ${src})
  target_link_libraries(${name} PRIVATE QualityControl CURL::libcurl O2::ITSQCDataReaderWorkflow O2::DetectorsBase ROOT::Tree)
  install_symlink(${name} ${CMAKE_INSTALL_FULL_BINDIR}/${oldname})
endforeach()

install(
  TARGETS ${EXE_NAMES}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# ---- Extra scripts ----

install(FILES
        its.json
        itsQCTrendingThr.json
        itsQCTrendingFhr.json
        itsFhr.json
        itsFee.json
        itsCluster.json
        DESTINATION etc)

get_property(dirs
             DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
             PROPERTY INCLUDE_DIRECTORIES)
foreach(dir ${dirs})
  message(STATUS "dir='${dir}'")
endforeach()
