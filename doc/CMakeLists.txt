find_program(SPHINX_EXECUTABLE NAMES sphinx-build
  HINTS $ENV{SPHINX_DIR}
  PATH_SUFFIXES bin
  )

if(NOT SPHINX_EXECUTABLE)
  message(FATAL_ERROR "SPHINX_EXECUTABLE (sphinx-build) is not found!")
endif()

find_package(LATEX)

add_subdirectory(en)
add_subdirectory(jp)

add_custom_target(doc DEPENDS doc-jp doc-en)
