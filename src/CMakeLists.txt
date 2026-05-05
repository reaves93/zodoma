cmake_minimum_required(VERSION 3.10)
project(tfs)

# Configuraciones básicas de seguridad y rutas
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

# Forzar C++17 para evitar errores de sintaxis modernos
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Tipo de construcción: Release para máximo rendimiento
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif()

# Buscar paquetes instalados por vcpkg
find_package(cryptopp REQUIRED)
find_package(MySQL REQUIRED)
find_package(Threads REQUIRED)
find_package(Boost REQUIRED COMPONENTS system filesystem iostreams asio)
find_package(Lua REQUIRED)

# Intentar buscar PugiXML, pero no morir si falla (la buscará en tu carpeta src)
find_package(PugiXML)
if(PugiXML_FOUND)
    message("PugiXML encontrada en el sistema")
    set(PUGIXML_LIB pugixml)
else()
    message("PugiXML no encontrada, se usará la versión interna de src")
    set(PUGIXML_LIB "")
endif()

# Buscar automáticamente todos los archivos en src (tu nueva estructura)
file(GLOB_RECURSE tfs_SRC 
    "src/*.cpp" 
    "src/*.hpp" 
    "src/*.h"
)

# Crear el ejecutable
add_executable(tfs ${tfs_SRC})

# Directorios de inclusión (Dónde están los .h)
target_include_directories(tfs PRIVATE src)
target_include_directories(tfs PRIVATE 
    ${MYSQL_INCLUDE_DIR} 
    ${LUA_INCLUDE_DIR} 
    ${Boost_INCLUDE_DIRS} 
    ${PUGIXML_INCLUDE_DIR}
)

# Conectar las librerías al ejecutable final
target_link_libraries(tfs PRIVATE 
    ${MYSQL_LIBRARIES} 
    ${LUA_LIBRARIES} 
    cryptopp-static
    Boost::system
    Boost::filesystem
    Boost::iostreams
    ${CMAKE_THREAD_LIBS_INIT}
)

# Configuraciones especiales para Windows (MSVC)
if(MSVC)
    target_compile_definitions(tfs PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_compile_definitions(tfs PRIVATE _WINSOCK_DEPRECATED_NO_WARNINGS)
    target_compile_options(tfs PRIVATE /W3 /utf-8)
endif()
