cmake_minimum_required(VERSION 3.1)
project(libjpeg)

set(JPEG_LIB_VERSION 90)
set(JPEG_LIB_VERSION_MAJOR 9)
set(JPEG_LIB_VERSION_MINOR 1)

 # for now nothing to configure, it just copies
 configure_file(jconfig.h.in include/jconfig.h)

set(MEMORYMGR jmemnobs)

set(LIBSOURCES jaricom.c jcapimin.c jcapistd.c jcarith.c jccoefct.c jccolor.c
        jcdctmgr.c jchuff.c jcinit.c jcmainct.c jcmarker.c jcmaster.c
        jcomapi.c jcparam.c jcprepct.c jcsample.c jctrans.c jdapimin.c
        jdapistd.c jdarith.c jdatadst.c jdatasrc.c jdcoefct.c jdcolor.c
        jddctmgr.c jdhuff.c jdinput.c jdmainct.c jdmarker.c jdmaster.c
        jdmerge.c jdpostct.c jdsample.c jdtrans.c jerror.c jfdctflt.c
        jfdctfst.c jfdctint.c jidctflt.c jidctfst.c jidctint.c jquant1.c
        jquant2.c jutils.c jmemmgr.c ${MEMORYMGR}.c)

# Headers which are installed to support the library
set(INSTINCLUDES jerror.h jmorecfg.h jpeglib.h)

add_library(libjpeg ${LIBSOURCES})
target_include_directories(libjpeg
	PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
		$<INSTALL_INTERFACE:include>
		)
set_target_properties(libjpeg PROPERTIES
	VERSION ${JPEG_LIB_VERSION_MAJOR}.${JPEG_LIB_VERSION_MINOR}
	SOVERSION ${JPEG_LIB_VERSION}
	DEBUG_POSTFIX d)
if(UNIX)
	set_target_properties(libjpeg PROPERTIES
		OUTPUT_NAME jpeg)
endif()

# Executables to build
set(bin_PROGRAMS cjpeg djpeg jpegtran rdjpgcom wrjpgcom)

# Executable sources & libs
set(cjpeg_SOURCES cjpeg.c rdppm.c rdgif.c rdtarga.c rdrle.c rdbmp.c
        rdswitch.c cdjpeg.c)
set(djpeg_SOURCES djpeg.c wrppm.c wrgif.c wrtarga.c wrrle.c wrbmp.c
        rdcolmap.c cdjpeg.c)
set(jpegtran_SOURCES jpegtran.c rdswitch.c cdjpeg.c transupp.c)
set(rdjpgcom_SOURCES rdjpgcom.c)
set(wrjpgcom_SOURCES wrjpgcom.c)

foreach(i ${bin_PROGRAMS})
	add_executable(${i} ${${i}_SOURCES})
endforeach()

target_link_libraries(cjpeg libjpeg)
target_link_libraries(djpeg libjpeg)
target_link_libraries(jpegtran libjpeg)
target_include_directories(rdjpgcom PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/include)
target_include_directories(wrjpgcom PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/include)

install(FILES ${INSTINCLUDES} ${CMAKE_CURRENT_BINARY_DIR}/include/jconfig.h
	DESTINATION include)

install(TARGETS libjpeg EXPORT jpeg-targets
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin
	)
install(EXPORT jpeg-targets DESTINATION lib/cmake/jpeg
	FILE jpeg-config.cmake)

enable_testing()

macro(mytest name target args input output)
	add_test(
		NAME ${name}
		COMMAND ${CMAKE_COMMAND}
			"-DCOMMAND=$<TARGET_FILE:${target}> ${args}"
			-DINPUT=${input}
  			-DOUTPUT=${output}
  			-P ${CMAKE_CURRENT_SOURCE_DIR}/jpeg_test.cmake
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/test)
endmacro()

file(COPY testorig.jpg testimg.ppm testprog.jpg testimg.bmp testimg.jpg testimgp.jpg
	DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/test)

mytest(t1 djpeg    "-dct int -ppm -outfile testout.ppm testorig.jpg"
	testimg.ppm testout.ppm)

mytest(t2 djpeg    "-dct int -bmp -colors 256 -outfile testout.bmp testorig.jpg"
	testimg.bmp testout.bmp)

mytest(t3 cjpeg    "-dct int -outfile testout.jpg testimg.ppm"
	testimg.jpg testout.jpg)

mytest(t4 djpeg    "-dct int -ppm -outfile testoutp.ppm testprog.jpg"
	testimg.ppm testoutp.ppm)

mytest(t5 cjpeg    "-dct int -progressive -opt -outfile testoutp.jpg testimg.ppm"
	testimgp.jpg testoutp.jpg)

mytest(t6 jpegtran "-outfile testoutt.jpg testprog.jpg"
	testorig.jpg testoutt.jpg)



