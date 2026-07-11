# Apply the small MoErgo ZMK integration patch before ZMK sources are compiled.
# The repository pins the exact upstream revision in config/west.yml, so a
# failed context check is treated as a hard error instead of silently building
# firmware without Magic-key status support.
set(magic_status_patch "${CMAKE_CURRENT_LIST_DIR}/patches/moergo-magic-status.patch")
set(zmk_source_root "${APPLICATION_SOURCE_DIR}/..")

execute_process(
  COMMAND git -C "${zmk_source_root}" apply --check "${magic_status_patch}"
  RESULT_VARIABLE patch_check_result
  OUTPUT_QUIET
  ERROR_QUIET
)

if(patch_check_result EQUAL 0)
  execute_process(
    COMMAND git -C "${zmk_source_root}" apply "${magic_status_patch}"
    RESULT_VARIABLE patch_apply_result
  )
  if(NOT patch_apply_result EQUAL 0)
    message(FATAL_ERROR "Failed to apply the Magic status patch")
  endif()
else()
  execute_process(
    COMMAND git -C "${zmk_source_root}" apply --reverse --check "${magic_status_patch}"
    RESULT_VARIABLE patch_already_applied_result
    OUTPUT_QUIET
    ERROR_QUIET
  )
  if(NOT patch_already_applied_result EQUAL 0)
    message(FATAL_ERROR "Magic status patch does not match the pinned MoErgo ZMK revision")
  endif()
endif()
