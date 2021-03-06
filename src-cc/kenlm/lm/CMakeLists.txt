cmake_minimum_required(VERSION 2.8.8)
#
# The KenLM cmake files make use of add_library(... OBJECTS ...)
# 
# This syntax allows grouping of source files when compiling
# (effectively creating "fake" libraries based on source subdirs).
# 
# This syntax was only added in cmake version 2.8.8
#
# see http://www.cmake.org/Wiki/CMake/Tutorials/Object_Library


# This CMake file was created by Lane Schwartz <dowobeha@gmail.com>


set(KENLM_MAX_ORDER 6)

add_definitions(-DKENLM_MAX_ORDER=${KENLM_MAX_ORDER})


# Explicitly list the source files for this subdirectory
#
# If you add any source files to this subdirectory
#    that should be included in the kenlm library,
#        (this excludes any unit test files)
#    you should add them to the following list:
set(KENLM_SOURCE 
	bhiksha.cc
	binary_format.cc
	config.cc
	lm_exception.cc
	model.cc
	quantize.cc
	read_arpa.cc
	search_hashed.cc
	search_trie.cc
	sizes.cc
	trie.cc
	trie_sort.cc
	value_build.cc
	virtual_interface.cc
	vocab.cc
)


# Group these objects together for later use. 
#
# Given add_library(foo OBJECT ${my_foo_sources}),
# refer to these objects as $<TARGET_OBJECTS:foo>
#
add_library(kenlm OBJECT ${KENLM_SOURCE})

# This directory has children that need to be processed
add_subdirectory(builder)
add_subdirectory(common)
add_subdirectory(filter)



# Explicitly list the executable files to be compiled
set(EXE_LIST
  query
  fragment
  build_binary
)

# Iterate through the executable list   
foreach(exe ${EXE_LIST})

  # Compile the executable, linking against the requisite dependent object files
  add_executable(${exe} ${exe}_main.cc $<TARGET_OBJECTS:kenlm> $<TARGET_OBJECTS:kenlm_util>)

  # Link the executable against boost
  target_link_libraries(${exe} ${Boost_LIBRARIES} pthread)

  # Group executables together
  set_target_properties(${exe} PROPERTIES FOLDER executables)

# End for loop
endforeach(exe)


# Install the executable files
install(TARGETS ${EXE_LIST} DESTINATION bin)



if(BUILD_TESTING)

    # Explicitly list the Boost test files to be compiled
    set(KENLM_BOOST_TESTS_LIST
      left_test
      model_test
      partial_test
    )

    # Iterate through the Boost tests list   
    foreach(test ${KENLM_BOOST_TESTS_LIST})

      # Compile the executable, linking against the requisite dependent object files
      add_executable(${test} ${test}.cc $<TARGET_OBJECTS:kenlm> $<TARGET_OBJECTS:kenlm_util>)

      # Require the following compile flag
      set_target_properties(${test} PROPERTIES COMPILE_FLAGS -DBOOST_TEST_DYN_LINK)
  
      # Link the executable against boost
      target_link_libraries(${test} ${Boost_LIBRARIES} pthread)

      # model_test requires an extra command line parameter
      if ("${test}" STREQUAL "model_test")
        set(test_params 
            ${CMAKE_CURRENT_SOURCE_DIR}/test.arpa 
            ${CMAKE_CURRENT_SOURCE_DIR}/test_nounk.arpa
        )
      else()
        set(test_params 
            ${CMAKE_CURRENT_SOURCE_DIR}/test.arpa
        )      
      endif()
      
      # Specify command arguments for how to run each unit test
      #
      # Assuming that foo was defined via add_executable(foo ...),
      #   the syntax $<TARGET_FILE:foo> gives the full path to the executable.
      #
      add_test(NAME ${test}_test 
               COMMAND $<TARGET_FILE:${test}> ${test_params})

      # Group unit tests together
      set_target_properties(${test} PROPERTIES FOLDER "unit_tests")
      
    # End for loop
    endforeach(test)

endif()




