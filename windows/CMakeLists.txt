configure_file(cantata.nsi.cmake ${CMAKE_CURRENT_BINARY_DIR}/cantata.nsi)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/cantata.nsi cantata.ico LICENSE.txt "Cantata README.txt" DESTINATION ${CMAKE_INSTALL_PREFIX})

# given a library (presumably the .lib file), try
# to find the dll and install it
function(install_lib lib_path)
  
  # tag.lib, tag.dll.a
  get_filename_component(lib_file ${lib_path}  NAME)
  
  # tag, tag.dll
  get_filename_component(lib_name ${lib_file} NAME_WE)
  
  # /path/lib
  get_filename_component(lib_folder ${lib_path} DIRECTORY)
  
  # /path
  get_filename_component(top_folder ${lib_folder} DIRECTORY)
  
  message(${lib_folder}/${CMAKE_SHARED_LIBRARY_PREFIX}${lib_name})
  
  file(GLOB lib_dlls ${lib_folder}/${CMAKE_SHARED_LIBRARY_PREFIX}${lib_name})
  foreach(dll ${lib_dlls})
    install(FILES ${dll}
	  DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
  
  file(GLOB lib_dlls ${top_folder}/bin/${CMAKE_SHARED_LIBRARY_PREFIX}${lib_name})
  foreach(dll ${lib_dlls})
    install(FILES ${dll}
	  DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
  
  file(GLOB lib_dlls ${lib_folder}/${CMAKE_SHARED_LIBRARY_PREFIX}${lib_name}${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${lib_dlls})
    install(FILES ${dll}
	  DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()

  file(GLOB lib_dlls ${top_folder}/bin/${CMAKE_SHARED_LIBRARY_PREFIX}${lib_name}${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${lib_dlls})
    install(FILES ${dll}
	  DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
endfunction()

function(install_qt_plugins)
  foreach(name ${ARGV})
      # get the full path like /c/qt/plugins/imageformats/qjpegplugin.dll
      get_target_property(plugin_path ${name} LOCATION)
	  
	  # /c/qt/plugins/imageformats/
	  get_filename_component(plugin_dir ${plugin_path} DIRECTORY)
	  
	  # imageformats/
	  get_filename_component(plugin_dirname ${plugin_dir} NAME)
	  
	  install(
	    FILES "${plugin_path}"
		DESTINATION "${CMAKE_INSTALL_PREFIX}/${plugin_dirname}"
	  )
  endforeach()
endfunction()

function(install_icu_libs)
  get_target_property(core_path Qt5::Core LOCATION)
  get_filename_component(lib_dir ${core_path} DIRECTORY)
  file(GLOB icu_libs ${lib_dir}/${CMAKE_SHARED_LIBRARY_PREFIX}icu*)
  install(FILES ${icu_libs}
    DESTINATION ${CMAKE_INSTALL_PREFIX})
endfunction()

function(install_translations)
  get_target_property(core_path Qt5::Core LOCATION)
  get_filename_component(lib_dir ${core_path} DIRECTORY)
  get_filename_component(top_dir ${lib_dir} DIRECTORY)
  file(GLOB translations ${top_dir}/translations/qt_*.qm)
  foreach(qm ${translations})
    if (NOT ${qm} MATCHES "(${top_dir}/translations/qt_help*)")
      install(FILES ${qm}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/translations/)
    endif ()
  endforeach()
endfunction()

function(install_zlib)
  get_target_property(zlib_path ZLIB::ZLIB LOCATION)
  get_filename_component(lib_dir ${zlib_path} DIRECTORY)
  get_filename_component(root_dir ${lib_dir} DIRECTORY)

  
  file(GLOB dlls ${lib_dir}/${CMAKE_SHARED_LIBRARY_PREFIX}zlib1${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${dlls})
    install(FILES ${dll}
	DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()

  
  file(GLOB dlls ${root_dir}/bin/${CMAKE_SHARED_LIBRARY_PREFIX}zlib1${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${dlls})
    install(FILES ${dll}
	DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
  
  file(GLOB dlls ${lib_dir}/${CMAKE_SHARED_LIBRARY_PREFIX}z${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${dlls})
    install(FILES ${dll}
	DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
  
  file(GLOB dlls ${root_dir}/bin/${CMAKE_SHARED_LIBRARY_PREFIX}z${CMAKE_SHARED_LIBRARY_SUFFIX})
  foreach(dll ${dlls})
    install(FILES ${dll}
	DESTINATION ${CMAKE_INSTALL_PREFIX})
	return()
  endforeach()
endfunction()

install_qt_plugins(${Qt5Gui_PLUGINS})
install_qt_plugins(${Qt5Network_PLUGINS})
install_qt_plugins(${Qt5Sql_PLUGINS})
install_qt_plugins(${Qt5Widgets_PLUGINS})
if (ENABLE_HTTP_STREAM_PLAYBACK)
    install_qt_plugins(${Qt5Multimedia_PLUGINS})
endif()

foreach(component
  Qt5::Core
  Qt5::Concurrent
  Qt5::Gui
  Qt5::Network
  Qt5::Sql
  Qt5::Svg
  Qt5::Xml
  Qt5::Widgets
  Qt5::WinExtras)
  install(FILES "$<TARGET_FILE:${component}>" DESTINATION "${CMAKE_INSTALL_PREFIX}")
endforeach()

if (ENABLE_HTTP_STREAM_PLAYBACK)
    install(FILES "$<TARGET_FILE:Qt5::Multimedia>" DESTINATION "${CMAKE_INSTALL_PREFIX}")
endif()

install_icu_libs()
install_translations()
install_zlib()

if (TAGLIB_FOUND)
    install_lib(${TAGLIB_LIBRARIES})
endif()


install(FILES LICENSE.txt DESTINATION ${CMAKE_INSTALL_PREFIX} RENAME "Cantata License (GPL V3).txt")
install(FILES ../icons/cantata32.png DESTINATION ${CMAKE_INSTALL_PREFIX}/icons/ RENAME cantata.png)
