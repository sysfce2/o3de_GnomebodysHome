
# Currently we are in the GameJam2021/Code folder: ${CMAKE_CURRENT_LIST_DIR}
# Get the platform specific folder ${pal_dir} for the current folder: ${CMAKE_CURRENT_LIST_DIR}/Platform/${PAL_PLATFORM_NAME}
# Note: ly_get_list_relative_pal_filename will take care of the details for us, as this may be a restricted platform
#       in which case it will see if that platform is present here or in the restricted folder.
#       i.e. It could here : GameJam2021/Code/Platform/<platorm_name>  or
#            <restricted_folder>/<platform_name>/GameJam2021/Code
ly_get_list_relative_pal_filename(pal_dir ${CMAKE_CURRENT_LIST_DIR}/Platform/${PAL_PLATFORM_NAME} ${o3de_project_restricted_path} ${o3de_project_path} ${o3de_project_name})

# Now that we have the platform abstraction layer (PAL) folder for this folder, thats where we will find the
# traits for this platform. Traits for a platform are defines for things like whether or not something in this project
# is supported by this platform.
include(${pal_dir}/PAL_${PAL_PLATFORM_NAME_LOWERCASE}.cmake)

# Now that we have loaded our project traits for this platform, see if this project is even supported on this platform.
# If its not supported we just return after including the unsupported.
if(NOT PAL_TRAIT_GAMEJAM2021_SUPPORTED)
    return()
endif()

# We are on a supported platform, so add the GameJam2021 target
# Note: We include the common files and the platform specific files which are set in gamejam2021_files.cmake and
# in ${pal_dir}/gamejam2021_${PAL_PLATFORM_NAME_LOWERCASE}_files.cmake
ly_add_target(
    NAME GameJam2021.Static STATIC
    NAMESPACE Gem
    FILES_CMAKE
        gamejam2021_files.cmake
        ${pal_dir}/gamejam2021_${PAL_PLATFORM_NAME_LOWERCASE}_files.cmake
    INCLUDE_DIRECTORIES
        PUBLIC
            Include
    BUILD_DEPENDENCIES
        PRIVATE
            AZ::AzGameFramework
            Gem::Atom_AtomBridge.Static
)

ly_add_target(
    NAME GameJam2021 ${PAL_TRAIT_MONOLITHIC_DRIVEN_MODULE_TYPE}
    NAMESPACE Gem
    FILES_CMAKE
        gamejam2021_shared_files.cmake
        ${pal_dir}/gamejam2021_shared_${PAL_PLATFORM_NAME_LOWERCASE}_files.cmake
    INCLUDE_DIRECTORIES
        PUBLIC
            Include
    BUILD_DEPENDENCIES
        PRIVATE
            Gem::GameJam2021.Static
            AZ::AzCore
)

# if enabled, GameJam2021 is used by all kinds of applications
ly_create_alias(NAME GameJam2021.Builders NAMESPACE Gem TARGETS Gem::GameJam2021)
ly_create_alias(NAME GameJam2021.Tools    NAMESPACE Gem TARGETS Gem::GameJam2021)
ly_create_alias(NAME GameJam2021.Clients  NAMESPACE Gem TARGETS Gem::GameJam2021)
ly_create_alias(NAME GameJam2021.Servers  NAMESPACE Gem TARGETS Gem::GameJam2021)

################################################################################
# Gem dependencies
################################################################################

# Enable the specified list of gems from GEM_FILE or GEMS list for this specific project:
ly_enable_gems(PROJECT_NAME GameJam2021 GEM_FILE enabled_gems.cmake)

if(PAL_TRAIT_BUILD_SERVER_SUPPORTED)
    # this property causes it to actually make a ServerLauncher.
    # if you don't want a Server application, you can remove this and the
    # following ly_enable_gems lines.
    set_property(GLOBAL APPEND PROPERTY LY_LAUNCHER_SERVER_PROJECTS GameJam2021)
endif()
