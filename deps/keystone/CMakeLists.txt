
if(KEYSTONE_LOCAL_DIR)
	message(STATUS "Keystone: using local Keystone directory.")

	ExternalProject_Add(keystone-project
		DOWNLOAD_COMMAND ""
		SOURCE_DIR "${KEYSTONE_LOCAL_DIR}"
		CMAKE_ARGS
			# This does not work on MSVC, but may be useful on Linux.
			-DCMAKE_BUILD_TYPE=Release
			# Force python3 version set in this cmake file.
			# Python is used to generate LLVMBuild.cmake file. Python2 generates this file
			# with UNIX paths, which do not work. Python3 generates proper Windows paths.
			-DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
			-DKEYSTONE_BUILD_STATIC_RUNTIME=${RETDEC_MSVC_STATIC_RUNTIME}
			# Force the use of the same compiler as used to build the top-level
			# project. Otherwise, the external project may pick up a different
			# compiler, which may result in link errors.
			"${CMAKE_C_COMPILER_OPTION}"
			"${CMAKE_CXX_COMPILER_OPTION}"
		# Disable the update step.
		UPDATE_COMMAND ""
		# Disable the install step.
		INSTALL_COMMAND ""
	)
	force_configure_step(keystone-project)
else()
	message(STATUS "Keystone: using remote Keystone revision.")

	ExternalProject_Add(keystone-project
		URL ${KEYSTONE_URL}
		URL_HASH SHA256=${KEYSTONE_ARCHIVE_SHA256}
		DOWNLOAD_NAME keystone.zip
		CMAKE_ARGS
			# This does not work on MSVC, but may be useful on Linux.
			-DCMAKE_BUILD_TYPE=Release
			# Force python3 version set in this cmake file.
			# Python is used to generate LLVMBuild.cmake file. Python2 generates this file
			# with UNIX paths, which do not work. Python3 generates proper Windows paths.
			-DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
			-DKEYSTONE_BUILD_STATIC_RUNTIME=${RETDEC_MSVC_STATIC_RUNTIME}
			# Force the use of the same compiler as used to build the top-level
			# project. Otherwise, the external project may pick up a different
			# compiler, which may result in link errors.
			"${CMAKE_C_COMPILER_OPTION}"
			"${CMAKE_CXX_COMPILER_OPTION}"
		# Disable the update step.
		UPDATE_COMMAND ""
		# Disable the install step.
		INSTALL_COMMAND ""
		LOG_DOWNLOAD ON
		LOG_CONFIGURE ON
		LOG_BUILD ON
	)
endif()

check_if_variable_changed(KEYSTONE_LOCAL_DIR CHANGED)
if(CHANGED)
	ExternalProject_Get_Property(keystone-project binary_dir)
	message(STATUS "Keystone: path to Keystone directory changed -> cleaning CMake files in ${binary_dir}.")
	clean_cmake_files(${binary_dir})
endif()

ExternalProject_Get_Property(keystone-project source_dir)
ExternalProject_Get_Property(keystone-project binary_dir)

# Create target.
add_library(keystone INTERFACE)
add_library(retdec::deps::keystone ALIAS keystone)
add_dependencies(keystone keystone-project)

target_include_directories(keystone
	SYSTEM INTERFACE
		$<BUILD_INTERFACE:${source_dir}/include>
)

target_link_libraries(keystone INTERFACE
	debug
		${binary_dir}/llvm/lib/${DEBUG_DIR}${CMAKE_STATIC_LIBRARY_PREFIX}keystone${CMAKE_STATIC_LIBRARY_SUFFIX}
	optimized
		${binary_dir}/llvm/lib/${RELEASE_DIR}${CMAKE_STATIC_LIBRARY_PREFIX}keystone${CMAKE_STATIC_LIBRARY_SUFFIX}
)
