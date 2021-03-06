cmake_minimum_required(VERSION 2.6)
project(vcflib)

include(ExternalProject)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -D_FILE_OFFSET_BITS=64")

include_directories(include)
include_directories(tabixpp)
include_directories(fastahack)
include_directories(intervaltree)
include_directories(smithwaterman)
include_directories(multichoose)
include_directories(filevercmp)

add_library(vcflib STATIC 
    src/Variant.h 
    src/split.h 
    src/pdflib.hpp 
    src/var.hpp 
    src/cdflib.hpp 
    src/rnglib.hpp 
    src/join.h
    src/Variant.cpp 
    src/rnglib.cpp 
    src/var.cpp 
    src/pdflib.cpp 
    src/cdflib.cpp 
    src/split.cpp
    src/ssw.h
    src/ssw_cpp.h
    tabixpp/tabix.cpp
    fastahack/Fasta.cpp
    smithwaterman/SmithWatermanGotoh.cpp
    smithwaterman/Repeats.cpp
    smithwaterman/IndelAllele.cpp
    smithwaterman/disorder.cpp
    smithwaterman/LeftAlign.cpp
    fsom/fsom.c
    filevercmp/filevercmp.c
    )

set(BINS 
    vcfecho
    dumpContigsFromHeader
    bFst
    pVst
    hapLrt
    popStats
    wcFst
    iHS
    segmentFst
    segmentIhs
    genotypeSummary
    sequenceDiversity
    pFst
    smoother
    LD
    plotHaps
    abba-baba
    permuteGPAT++
    permuteSmooth
    normalize-iHS
    meltEHH
    vcfaltcount
    vcfhetcount
    vcfhethomratio
    vcffilter
    vcf2tsv
    vcfgenotypes
    vcfannotategenotypes
    vcfcommonsamples
    vcfremovesamples
    vcfkeepsamples
    vcfsamplenames
    vcfgenotypecompare
    vcffixup
    vcfclassify
    vcfsamplediff
    vcfremoveaberrantgenotypes
    vcfrandom
    vcfparsealts
    vcfstats
    vcfflatten
    vcfprimers
    vcfnumalt
    vcfcleancomplex
    vcfintersect
    vcfannotate
    vcfallelicprimitives
    vcfoverlay
    vcfaddinfo
    vcfkeepinfo
    vcfkeepgeno
    vcfafpath
    vcfcountalleles
    vcflength
    vcfdistance
    vcfrandomsample
    vcfentropy
    vcfglxgt
    vcfroc
    vcfcheck
    vcfstreamsort
    vcfuniq
    vcfuniqalleles
    vcfremap
    vcf2fasta
    vcfsitesummarize
    vcfbreakmulti
    vcfcreatemulti
    vcfevenregions
    vcfcat
    vcfgenosummarize
    vcfgenosamplenames
    vcfgeno2haplo
    vcfleftalign
    vcfcombine
    vcfgeno2alleles
    vcfindex
    vcf2dag
    vcfsample2info
    vcfqual2info
    vcfinfo2qual
    vcfglbound
    vcfinfosummarize)

execute_process(COMMAND git describe --abbrev=4 --dirty --always 
  OUTPUT_VARIABLE GIT_VERSION 
  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
  OUTPUT_STRIP_TRAILING_WHITESPACE)

ExternalProject_Add(htslib-build
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/tabixpp/htslib
  BINARY_DIR ${CMAKE_SOURCE_DIR}/tabixpp/htslib
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}/htslib
  CONFIGURE_COMMAND ""
  BUILD_COMMAND make lib-static
  INSTALL_COMMAND ""
  )

add_library(htslib STATIC IMPORTED)
set_property(TARGET htslib PROPERTY 
  IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/tabixpp/htslib/libhts.a)
add_dependencies(htslib htslib-build)

find_package(ZLIB REQUIRED)
find_package(Threads REQUIRED)

foreach(BIN ${BINS})
  add_executable(${BIN} src/${BIN}.cpp)
  target_link_libraries(${BIN} vcflib htslib ${CMAKE_THREAD_LIBS_INIT} ${ZLIB_LIBRARIES})
  target_compile_definitions(${BIN} PRIVATE -DVERSION="${GIT_VERSION}")
endforeach(BIN ${BINS})

install(TARGETS ${BINS} RUNTIME DESTINATION bin)
install(TARGETS vcflib ARCHIVE DESTINATION lib)

file(GLOB INCLUDES
  src/*.h*
  multichoose/*.h*
  intervaltree/*.h*
  tabixpp/*.h*
  smithwaterman/*.h*
  fastahack/*.h*
  filevercmp/*.h*)

install(FILES ${INCLUDES} DESTINATION include)
