set(LIBRARY CAFAnaFitExt)

file(GLOB HEADER_FILES *.h *.txx)
file(GLOB SOURCES *.cxx)

add_library(${LIBRARY} SHARED
        ${HEADER_FILES}
        ${SOURCES}
)

link_root(${LIBRARY})
link_ifdhc(${LIBRARY})

install(TARGETS ${LIBRARY} LIBRARY DESTINATION ${TARGET_LIBDIR})
install_headers("${HEADER_FILES}")
