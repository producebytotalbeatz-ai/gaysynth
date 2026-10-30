cmake_minimum_required(VERSION 3.22)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
project(Necrosynth VERSION 1.0.0)

include(FetchContent)
FetchContent_Declare(JUCE
    GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
    GIT_TAG 8.0.6
    GIT_SHALLOW ON)
FetchContent_MakeAvailable(JUCE)

# Builds the same plugin twice:
#   Necrosynth      = full custom interface
#   NecrosynthSafe  = same sound engine, but JUCE's plain generic editor (for crash diagnosis)
function(necro_plugin target name code safe)
    juce_add_plugin(${target}
        COMPANY_NAME "Necro Audio"
        PLUGIN_MANUFACTURER_CODE Ncra
        PLUGIN_CODE ${code}
        IS_SYNTH TRUE
        NEEDS_MIDI_INPUT TRUE
        EDITOR_WANTS_KEYBOARD_FOCUS FALSE
        COPY_PLUGIN_AFTER_BUILD FALSE
        FORMATS VST3
        PRODUCT_NAME "${name}")

    juce_generate_juce_header(${target})

    target_sources(${target} PRIVATE
        Source/PluginProcessor.cpp
        Source/PluginEditor.cpp)

    target_compile_definitions(${target} PUBLIC
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0
        JUCE_VST3_EMULATE_MIDI_CC_WITH_PARAMETERS=0
        JUCE_DISPLAY_SPLASH_SCREEN=0)

    if(safe)
        target_compile_definitions(${target} PUBLIC NECRO_SAFE_EDITOR=1)
    endif()

    target_link_libraries(${target}
        PRIVATE
            juce::juce_audio_utils
            juce::juce_dsp
        PUBLIC
            juce::juce_recommended_config_flags
            juce::juce_recommended_lto_flags
            juce::juce_recommended_warning_flags)
endfunction()

necro_plugin(Necrosynth     "Necrosynth"     Ncsy FALSE)
necro_plugin(NecrosynthSafe "NecrosynthSafe" Ncsf TRUE)
