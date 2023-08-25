cmake_minimum_required(VERSION 3.12)

project(Chat)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Включаем генерацию промежуточных файлов в директорию сборки
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Добавляем флаги компиляции для GCC и MSVC
if (CMAKE_COMPILER_IS_GNUCC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
endif()
if (MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
endif()

# Создаем список библиотек, включая модули Qt
set(CHAT_LIBRARIES)

# Список подключаемых модулей Qt
set(Qt5Modules Core Widgets)

# Добавляем библиотеки Qt к CHAT_LIBRARIES
foreach(QtModule IN LISTS Qt5Modules)
    list(APPEND CHAT_LIBRARIES Qt5::${QtModule})
endforeach()

# Проверяем наличие необходимых модулей и компонентов Qt
find_package(Qt5 COMPONENTS ${Qt5Modules} REQUIRED LinguistTools)






# Формируем список всех исходников проекта
file(GLOB_RECURSE SRC_FILES sources/*.cpp)
file(GLOB_RECURSE HEADERS_FILES sources/*.h)
file(GLOB_RECURSE RC_FILES sources/*.qrc)
file(GLOB_RECURSE UI_FILES sources/*.ui)

# Включаем автоматическую обработку ui и qrc файлов
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# Генерируем исходники из ui и qrc файлов
qt5_wrap_ui(QUI ${UI_FILES})
qt5_add_resources(QRC ${RC_FILES})

# Создаем файлы переводов и зависимости для их генерации
set(TS_FILES ${CMAKE_CURRENT_SOURCE_DIR}/translations/my_ru.ts)
file(GLOB_RECURSE TS_SOURCES "*.cpp" "*.h" ".ui")
add_custom_target(translations)
foreach(_ts_file ${TS_FILES})
    get_filename_component(_ts_name ${_ts_file} NAME_WE)
    add_custom_command(
        OUTPUT "${_ts_file}"
        DEPENDS ${TS_SOURCES}
        COMMAND ${Qt5_LUPDATE_EXECUTABLE} -recursive
        ${CMAKE_CURRENT_SOURCE_DIR} -ts ${_ts_file}
    )
    add_custom_target(translate_${_ts_name} DEPENDS ${_ts_file})
    add_dependencies(translations translate_${_ts_name})
endforeach()

# Добавляем файлы переводов к проекту
qt5_add_translation(QM_FILES ${TS_FILES})

# Создаем исполняемый файл проекта
add_executable(Chat ${QM_FILES} ${SRC_FILES} ${HEADERS_FILES}
    ${RC_FILES} ${UI_FILES}

)

# Подключаем библиотеки к проекту
target_link_libraries(Chat PUBLIC ${CHAT_LIBRARIES})

# Добавляем поддержку библиотеки MySQL к проекту
#target_link_libraries(Chat PRIVATE Qt5::Sql)
