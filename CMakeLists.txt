cmake_minimum_required(VERSION 2.8)


SET(TOOLCHAIN_FILE ${PROJECT_SOURCE_DIR}/Toolchain-windows-mingw32.cmake)
SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/module)

INCLUDE(${TOOLCHAIN_FILE})
INCLUDE(ExternalProject)

ExternalProject_Add(eigen
	URL http://bitbucket.org/eigen/eigen/get/3.2.0.tar.bz2
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
	INSTALL_COMMAND cp -r ../eigen/Eigen/ ${CMAKE_MODULE_PATH}/eigen
)

ExternalProject_Add(flann 
	URL http://people.cs.ubc.ca/~mariusm/uploads/FLANN/flann-1.8.4-src.zip
	PATCH_COMMAND patch -p0 < ${PROJECT_SOURCE_DIR}/FLANN_CMakeLists.patch
	CMAKE_ARGS
		-DCMAKE_INSTALL_PREFIX=${CMAKE_MODULE_PATH}/flann
		-DCMAKE_TOOLCHAIN_FILE=${TOOLCHAIN_FILE}
		-DBUILD_C_BINDINGS=OFF
)

ExternalProject_Add(zlib 
	URL http://zlib.net/zlib-1.2.8.tar.gz
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
	INSTALL_COMMAND ""
)

ExternalProject_Add(bzip2
	URL http://www.bzip.org/1.0.6/bzip2-1.0.6.tar.gz
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
	INSTALL_COMMAND ""
)
SET( Boost_bjam_ARGS
		toolset=gcc-mingw
		target-os=windows
		threadapi=win32
		threading=multi
		variant=release
		--user-config=./user-config.jam
		--layout=tagged 
		--prefix=${CMAKE_MODULE_PATH}/boost
		-s BZIP2_SOURCE=${PROJECT_BINARY_DIR}/bzip2-prefix/src/bzip2
		-s ZLIB_SOURCE=${PROJECT_BINARY_DIR}/zlib-prefix/src/zlib
)

ExternalProject_Add(boost
	URL http://sourceforge.net/projects/boost/files/boost/1.54.0/boost_1_54_0.tar.bz2/download
	PATCH_COMMAND echo using gcc : mingw : i586-mingw32msvc-g++ : 4.2 \\073 >> user-config.jam
	BUILD_IN_SOURCE 1
	#CONFIGURE_COMMAND ""
	CONFIGURE_COMMAND ./bootstrap.sh
		--with-libraries=program_options,system,thread,mpi,filesystem,serialization,date_time,iostreams,chrono
	BUILD_COMMAND ./bjam ${Boost_bjam_ARGS}
	INSTALL_COMMAND ./bjam install ${Boost_bjam_ARGS}
)

#ADD_CUSTOM_TARGET(boost
#	COMMAND echo "boost"
#	COMMENT "Retreive and compile the dependency boost"
#	)

ADD_CUSTOM_TARGET(pcl ALL
	COMMAND echo "PCL"
	COMMENT "Compile PCL"
)

set_target_properties(eigen boost flann zlib bzip2 PROPERTIES EXCLUDE_FROM_ALL 1)

ADD_DEPENDENCIES( boost zlib bzip2 )
ADD_DEPENDENCIES( pcl boost eigen flann )

#add_subdirectory(pcl)