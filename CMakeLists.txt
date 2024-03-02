cmake_minimum_required(VERSION 3.13)

# if PICO_SDK_PATH is explicitly set then do a Pico BUILD
if (DEFINED PICO_SDK_PATH AND NOT DEFINED PICO_BUILD)
    set(PICO_BUILD 1)
endif()

if (DEFINED ENV{PICO_SDK_PATH} AND NOT DEFINED PICO_SDK_PATH)
    set(PICO_SDK_PATH $ENV{PICO_SDK_PATH})
endif()

if (PICO_BUILD AND NOT PICO_SDK_PATH)
    message(FATAL_ERROR "PICO_SDK_PATH is not set")
endif()

if (PICO_BUILD)
    if (PICO_ON_DEVICE)
        message("Building b-em for RP2040 using the Pico SDK")
    elseif(PI_BUILD)
        message("Building b-em for Pi using the Pico SDK")
    else()
        message("Building b-em for generic host mode using the Pico SDK")
    endif()
    list(APPEND PICO_BOARD_HEADER_DIRS ${CMAKE_CURRENT_LIST_DIR}/src/pico/boards)

    include(pico_sdk_import.cmake)
    # We also need PICO EXTRAS
    include(pico_extras_import.cmake)
else()
    message("Building b-em via regular build")
endif()

project(b_em C CXX)

if (PICO_BUILD)
    enable_language(ASM)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)

set(CMAKE_CXX_STANDARD 17)

# PI_BUILD (named because we are using the PICO SDK still) basically
# focuses on using ARM assembly support
option(PI_BUILD "Build for Raspberry Pi" OFF)

if (NOT PICO_BUILD)
    find_package(Allegro REQUIRED)
    if (NOT Allegro_FOUND)
        message(WARNING "Allegro is not found, so targets that require it will not be built; to build with Pico SDK pass PICO_BUILD=1 or pass PICO_SDK_PATH")
    endif()
endif()

if (PI_BUILD)
    if ("aarch64" STREQUAL ${CMAKE_SYSTEM_PROCESSOR})
        message("Disabling PI_ASM32 on 64 bit arm")
        message("WARNING: Pi 64-bit build is not yet worthwhile (except on Pi 5)")
        set(PI_ASM32 "0" CACHE INTERNAL "")
    else()
        option(PI_ASM32 "Use ASM when PI_BUILD is true" ON)
    endif()
    if (PI_ASM32)
        message("Will use ASM code compiled as ARM32")
        enable_language(ASM)
    endif()
endif()

# --------------------------------------------------------------
# b_em_core is the guts of b-em
add_library(b-em_core INTERFACE)

target_compile_definitions(b-em_core INTERFACE
        BEM
        USE_MEMORY_POINTER
        HAVE_STPCPY
        #_DEBUG
)

target_include_directories(b-em_core INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src)

target_sources(b-em_core INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/acia.c
        ${CMAKE_CURRENT_LIST_DIR}/src/adc.c
        ${CMAKE_CURRENT_LIST_DIR}/src/cmos.c
        ${CMAKE_CURRENT_LIST_DIR}/src/compact_joystick.c
        ${CMAKE_CURRENT_LIST_DIR}/src/compactcmos.c
        ${CMAKE_CURRENT_LIST_DIR}/src/compat_wrappers.c
        ${CMAKE_CURRENT_LIST_DIR}/src/config.c
        ${CMAKE_CURRENT_LIST_DIR}/src/csw.c
        ${CMAKE_CURRENT_LIST_DIR}/src/ddnoise.c
        ${CMAKE_CURRENT_LIST_DIR}/src/disc.c
        ${CMAKE_CURRENT_LIST_DIR}/src/fdi.c
        ${CMAKE_CURRENT_LIST_DIR}/src/fdi2raw.c
        ${CMAKE_CURRENT_LIST_DIR}/src/i8271.c
        ${CMAKE_CURRENT_LIST_DIR}/src/ide.c
        ${CMAKE_CURRENT_LIST_DIR}/src/joystick.c
        ${CMAKE_CURRENT_LIST_DIR}/src/keyboard.c
        ${CMAKE_CURRENT_LIST_DIR}/src/logging.c
        ${CMAKE_CURRENT_LIST_DIR}/src/main.c
        ${CMAKE_CURRENT_LIST_DIR}/src/mem.c
        ${CMAKE_CURRENT_LIST_DIR}/src/model.c
        ${CMAKE_CURRENT_LIST_DIR}/src/mouse.c
        ${CMAKE_CURRENT_LIST_DIR}/src/midi-linux.c
        ${CMAKE_CURRENT_LIST_DIR}/src/music2000.c
        ${CMAKE_CURRENT_LIST_DIR}/src/music4000.c
        ${CMAKE_CURRENT_LIST_DIR}/src/music5000.c
        ${CMAKE_CURRENT_LIST_DIR}/src/pal.c
        ${CMAKE_CURRENT_LIST_DIR}/src/scsi.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sdf-acc.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sdf-geo.c
        ${CMAKE_CURRENT_LIST_DIR}/src/serial.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sn76489.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sysacia.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sysvia.c
        ${CMAKE_CURRENT_LIST_DIR}/src/tape.c
        ${CMAKE_CURRENT_LIST_DIR}/src/tapenoise.c
        ${CMAKE_CURRENT_LIST_DIR}/src/uef.c
        ${CMAKE_CURRENT_LIST_DIR}/src/uservia.c
        ${CMAKE_CURRENT_LIST_DIR}/src/via.c
        ${CMAKE_CURRENT_LIST_DIR}/src/wd1770.c
        ${CMAKE_CURRENT_LIST_DIR}/src/win.c
        ${CMAKE_CURRENT_LIST_DIR}/src/linux.c
        )

# Large swathes of code have been separated into their own libraries that are opt in/out per exe target

# --------------------------------------------------------------
# tube is tube support
add_library(tube INTERFACE)
target_sources(tube INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/32016.c
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/Decode.c
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/NSDis.c
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/Profile.c
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/Trap.c
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/mem32016.c

        ${CMAKE_CURRENT_LIST_DIR}/src/6502tube.c
        ${CMAKE_CURRENT_LIST_DIR}/src/6809tube.c

        ${CMAKE_CURRENT_LIST_DIR}/src/arm.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/darm.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/darm-tbl.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/armv7.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/armv7-tbl.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/thumb.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/thumb-tbl.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/thumb2.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/thumb2-decoder.c
        ${CMAKE_CURRENT_LIST_DIR}/src/darm/thumb2-tbl.c
        ${CMAKE_CURRENT_LIST_DIR}/src/tube.c
        ${CMAKE_CURRENT_LIST_DIR}/src/x86.c
        ${CMAKE_CURRENT_LIST_DIR}/src/x86dasm.c

        ${CMAKE_CURRENT_LIST_DIR}/src/z80.c
        ${CMAKE_CURRENT_LIST_DIR}/src/z80dis.c

        ${CMAKE_CURRENT_LIST_DIR}/src/mc6809nc/mc6809nc.c
        ${CMAKE_CURRENT_LIST_DIR}/src/mc6809nc/mc6809_dis.c

        ${CMAKE_CURRENT_LIST_DIR}/src/65816.c
        )

target_include_directories(tube INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/darm
        ${CMAKE_CURRENT_LIST_DIR}/src/mc6809nc
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/pandora
)

# --------------------------------------------------------------
add_library(tube_debugger INTERFACE)
target_sources(tube_debugger INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/NS32016/32016_debug.c
        ${CMAKE_CURRENT_LIST_DIR}/src/mc6809nc/mc6809_debug.c
        )

# --------------------------------------------------------------
add_library(debugger INTERFACE)
target_compile_definitions(debugger INTERFACE
        INCLUDE_DEBUGGER
)

target_sources(debugger INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/6502debug.c
        ${CMAKE_CURRENT_LIST_DIR}/src/debugger.c
        )

# --------------------------------------------------------------
add_library(vdfs INTERFACE)
target_sources(vdfs INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/vdfs.c)

# --------------------------------------------------------------
add_library(save_state INTERFACE)
target_sources(save_state INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/savestate.c)

# --------------------------------------------------------------
add_library(resid INTERFACE)
target_sources(resid INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/resid.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/convolve-sse.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/convolve.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/envelope.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/extfilt.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/filter.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/pot.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/sid.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/voice.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave6581_PST.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave6581_PS_.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave6581_P_T.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave6581__ST.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave8580_PST.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave8580_PS_.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave8580_P_T.cc
        ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp/wave8580__ST.cc
        )
target_include_directories(resid INTERFACE ${CMAKE_CURRENT_LIST_DIR}/src/resid-fp)

# --------------------------------------------------------------
add_library(allegro_gui INTERFACE)
target_sources(allegro_gui INTERFACE
        ${CMAKE_CURRENT_LIST_DIR}/src/gui-allegro.c
        ${CMAKE_CURRENT_LIST_DIR}/src/keydef-allegro.c
        ${CMAKE_CURRENT_LIST_DIR}/src/tapecat-allegro.c
        ${CMAKE_CURRENT_LIST_DIR}/src/vidalleg.c
        ${CMAKE_CURRENT_LIST_DIR}/src/video.c
        ${CMAKE_CURRENT_LIST_DIR}/src/sound.c
        )

function(configure_b_em_exe TARGET)
    cmake_parse_arguments(CONFIG "" "VERSION;TUBE;DEBUGGER;SID;PICO_CPU;PICO_CPU_NO_ASM;ALLEGRO_GUI;SAVE_STATE;VDFS;UEF;CSW;FDI;MMB;IDE;ADC;MOUSE;MUSIC5000;SCSI;I8271" "" ${ARGN} )
    if (CONFIG_ALLEGRO_GUI AND NOT Allegro_FOUND)
        if (NOT PICO_BUILD)
            message("Skipping ${TARGET} because Allegro is not available")
        else()
            message("Skipping ${TARGET} for Pico SDK build")
        endif()
    else()
        add_executable(${TARGET})
        if (NOT DEFINED CONFIG_VDFS)
            set(CONFIG_VDFS 1)
        endif()
        if (NOT CONFIG_VERSION)
            set(CONFIG_VERSION "unknown")
        endif()
        target_compile_definitions(${TARGET} PRIVATE VERSION="${CONFIG_VERSION}")
        if (CONFIG_TUBE)
            target_link_libraries(${TARGET} PRIVATE tube)
            if (CONFIG_DEBUGGER)
                target_link_libraries(${TARGET} PRIVATE tube_debugger)
            endif()
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_TUBE)
        endif()
        if (CONFIG_DEBUGGER)
            target_link_libraries(${TARGET} PRIVATE debugger)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_DEBUGGER)
        endif()
        if (CONFIG_SID)
            target_link_libraries(${TARGET} PRIVATE resid)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_SID)
        endif()
        if (CONFIG_SAVE_STATE)
            target_link_libraries(${TARGET} PRIVATE save_state)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_SAVE_STATE)
        endif()
        if (CONFIG_VDFS)
            target_link_libraries(${TARGET} PRIVATE vdfs)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_VDFS)
        endif()
        if (CONFIG_UEF)
            target_link_libraries(${TARGET} PRIVATE z)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_UEF)
        endif()
        if (CONFIG_CSW)
            target_link_libraries(${TARGET} PRIVATE z)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_CSW)
        endif()
        if (NOT CONFIG_MMB)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_MMB)
        endif()
        if (NOT CONFIG_IDE)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_IDE)
        endif()
        if (NOT CONFIG_ADC)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_ADC)
        endif()
        if (NOT CONFIG_MOUSE)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_MOUSE)
        endif()
        if (NOT CONFIG_MUSIC5000)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_MUSIC5000)
        endif()
        if (NOT CONFIG_SCSI)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_SCSI)
        endif()
        if (NOT CONFIG_FDI)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_FDI)
        endif()
        if (NOT CONFIG_I8271)
            target_compile_definitions(${TARGET} PRIVATE NO_USE_I8271)
        endif()
        if (CONFIG_PICO_CPU_NO_ASM)
            target_compile_definitions(${TARGET} PRIVATE USE_PICO_CPU)
            target_link_libraries(${TARGET} PRIVATE pico_cpu_no_asm)
        elseif (CONFIG_PICO_CPU)
            target_compile_definitions(${TARGET} PRIVATE USE_PICO_CPU)
            target_link_libraries(${TARGET} PRIVATE pico_cpu)
        else()
            target_sources(${TARGET} PRIVATE src/6502.c)
        endif()
        if (CONFIG_ALLEGRO_GUI)
            target_link_libraries(${TARGET} PRIVATE allegro_gui)
        else()
            target_compile_definitions(${TARGET} PRIVATE NO_USE_ALLEGRO_GUI)
        endif()

        message("Configured ${TARGET} TUBE=${CONFIG_TUBE} DEBUGGER=${CONFIG_DEBUGGER} SID=${CONFIGURE_SID} PICO_CPU=${CONFIG_PICO_CPU} GUI=${CONFIGURE_ALLEGRO_GUI} SAVE=${CONFIG_SAVE_STATE} VDFS=${CONFIG_VDFS} FDI=${CONFIG_FDI} UEF=${CONFIG_UEF} CSW=${CONFIG_CSW} IDE=${CONFIG_IDE} SCSI=${CONFIG_SCSI} ADC=${CONFIG_ADC} MOUSE=${CONFIG_MOUSE} MUSIC5000=${CONFIG_MUSIC5000} I8271=${CONFIG_I8271}")
        target_link_libraries(${TARGET} PRIVATE b-em_core)
    endif()
endfunction()

if (PICO_NO_HARDWARE)
    target_compile_definitions(b-em_core INTERFACE PICO_NO_HARDWARE=1 PICO_ON_DEVICE=0)
endif()

# --------------------------------------------------------------
# allegro_headers: either the real ones or our fake ones
#                  from src/pico/stub_allegro
add_library(allegro_headers INTERFACE)
if (Allegro_FOUND AND NOT PICO_ON_DEVICE)
    add_library(allegro_base INTERFACE)
    target_compile_definitions(allegro_base INTERFACE HAVE_STPCPY)
    target_link_libraries(allegro_base INTERFACE
            ${ALLEGRO_AUDIO_LIBRARY}
            ${ALLEGRO_ACODEC_LIBRARY}
            ${ALLEGRO_LIBRARY}
            ${ALLEGRO_PRIMITIVES_LIBRARY}
            ${ALLEGRO_DIALOG_LIBRARY}
            ${ALLEGRO_IMAGE_LIBRARY}
            ${ALLEGRO_FONT_LIBRARY}
            m
            )

    if (NOT WIN32)
        target_link_libraries(allegro_base INTERFACE
                ${ALLEGRO_MAIN_LIBRARY}
                pthread)
    endif()
    target_include_directories(allegro_headers INTERFACE ${ALLEGRO_INCLUDE_DIRS})
    target_link_libraries(allegro_base INTERFACE allegro_headers)
    target_link_libraries(allegro_gui INTERFACE allegro_base)
else()
    target_include_directories(allegro_headers INTERFACE
            ${CMAKE_CURRENT_LIST_DIR}/src/pico/stub_allegro5/include)
    target_compile_definitions(allegro_headers INTERFACE NEVER_USE_ALLEGRO=1)
endif()
target_link_libraries(b-em_core INTERFACE allegro_headers)

if (Allegro_FOUND AND NOT PICO_ON_DEVICE)
    add_executable(hdmft
        src/hdfmt.c)

    add_executable(jstest
        src/jstest.c)

    target_link_libraries(jstest allegro_base)

    add_executable(gtest
            src/sdf-gtest.c
            src/sdf-geo.c
    )

    target_link_libraries(gtest allegro_base)
endif()


# This is the regular b-em (we hope)
configure_b_em_exe(b-em
        VERSION 2.2?-full
        TUBE 1
        DEBUGGER 1
        SID 1
        ALLEGRO_GUI 1
        VDFS 1
        UEF 1
        CSW 1
        FDI 1
        MMB 1

        IDE 1
        ADC 1
        MOUSE 1
        MUSIC5000 1
        SCSI 1
        I8271 1
        SAVE_STATE 1)

# This is b-em with a bunch of stuff turned off
configure_b_em_exe(b-em-reduced
        VERSION 2.2?-reduced
        TUBE 0
        DEBUGGER 0
        SID 0
        ALLEGRO_GUI 1
        VDFS 0
        FDI 0
        SAVE_STATE 0)

# This is the same as b-em-reduced but using the C version of src/thumb_cpu
configure_b_em_exe(b-em-reduced-thumb-cpu
        VERSION 2.2?-reduced-thumb-cpu
        TUBE 0
        DEBUGGER 0
        SID 0
        PICO_CPU_NO_ASM 1
        ALLEGRO_GUI 1
        VDFS 0
        FDI 0
        SAVE_STATE 0)

add_subdirectory(src/thumb_cpu)

if (PICO_BUILD)
    add_subdirectory(src/pico)
endif()

