#A CMakeLists.txt that automatically sets up all dependencies for this package.

cmake_minimum_required(VERSION 2.8.12)

include(ExternalProject)

set(SOURCE_CODE_DIR ${CMAKE_SOURCE_DIR}/../..)
set(CVS_DOWNLOAD_COMMAND)

if(EXISTS ${SOURCE_CODE_DIR}/Ana AND NOT ${CMAKE_NOT_FIRST_RUN})
  message("Ana already exists.  I'm not going to let CVS download anything so it doesn't overwrite your hard work.")
  set(CVS_DOWNLOAD_COMMAND "DOWNLOAD_COMMAND \"\"")
endif()

#Automatically pull the latest changes to all packages and update them
#if(NOT DISABLE_AUTO_UPDATE)
#  set(DISABLE_AUTO_UPDATE false)
#endif()
#
##Hack to turn off UPDATE_DISCONNECTED for CMake < 3.2.0 when it didn't exist.
##This affects the default cmake version on the SL7 MINERvA GPVMs.
#if(${CMAKE_VERSION} VERSION_GREATER "3.2.0")
#  set(UPDATE_OPTION "UPDATE_DISCONNECTED ${DISABLE_AUTO_UPDATE}")
#endif()

ExternalProject_Add(PlotUtils
                    CVS_REPOSITORY minervacvs@cdcvs.fnal.gov:/cvs/mnvsoft
                    CVS_MODULE Ana/PlotUtils
                    ${CVS_DOWNLOAD_COMMAND}
                    SOURCE_DIR "${SOURCE_CODE_DIR}/Ana/PlotUtils"
                    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE})

ExternalProject_Add(UnfoldUtils
                    CVS_REPOSITORY minervacvs@cdcvs.fnal.gov:/cvs/mnvsoft
                    CVS_MODULE Ana/UnfoldUtils
                    ${CVS_DOWNLOAD_COMMAND}
                    SOURCE_DIR "${SOURCE_CODE_DIR}/Ana/UnfoldUtils"
                    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
                    DEPENDS PlotUtils)

ExternalProject_Add(GENIEXSecExtract
                    CVS_REPOSITORY minervacvs@cdcvs.fnal.gov:/cvs/mnvsoft
                    CVS_MODULE GENIEXSecExtract
                    ${CVS_DOWNLOAD_COMMAND}
                    SOURCE_DIR "${SOURCE_CODE_DIR}/GENIEXSecExtract"
                    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
                    DEPENDS PlotUtils)


#MParamFiles doesn't need to be built or installed.  It's a bunch of text and .root files that programs need to be able to find.
#Nothing depends on MParamFiles to build, but it does need to be installed before much of libPlotUtils can be used.
ExternalProject_Add(MParamFiles
                    CVS_REPOSITORY minervacvs@cdcvs.fnal.gov:/cvs/mnvsoft
                    CVS_MODULE MParamFiles
                    ${CVS_DOWNLOAD_COMMAND}
                    SOURCE_DIR "${SOURCE_CODE_DIR}/MParamFiles"
                    CONFIGURE_COMMAND ""
                    BUILD_COMMAND ""
                    INSTALL_COMMAND ""
                    )

#Configure the tutorial itself from the source code in the parent directory.
#No need to download!  This is necessary because ExternalProject_Add()
#happens at build time.  FetchContent() fixes this in CMake 3.11, but
#that's newer than the CMake version on the GPVMs or my laptop.
ExternalProject_Add(MINERvA-101-Cross-Section
                    SOURCE_DIR "${SOURCE_CODE_DIR}/MINERvA-101-Cross-Section"
                    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX} -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
                    UPDATE_COMMAND ""
                    PATCH_COMMAND ""
                    DEPENDS PlotUtils UnfoldUtils GENIEXSecExtract)

#Put MParamFiles into the directory structure
install(DIRECTORY ${SOURCE_CODE_DIR}/MParamFiles DESTINATION etc)

#Tell setup.sh about the installation directory of this package
configure_file(setup.sh.in setup.sh @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/setup.sh DESTINATION bin)
install(FILES setupROOT6OnGPVMs.sh DESTINATION bin)
