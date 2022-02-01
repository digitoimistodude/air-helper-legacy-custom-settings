<?php
/**
 * @Author:		Elias Kautto
 * @Date:   		2022-02-01 10:52:26
 * @Last Modified by:   Elias Kautto
 * @Last Modified time: 2022-02-01 11:03:00
 * Plugin name: Air helper legacy custom settings support
 */

if ( ! function_exists( 'get_custom_setting' ) ) {
  /**
   *  Get singular setting field from defined setting group.
   *  Setting groups are posts in settings CPT and post for
   *  each group is assigned via filter.
   *
   *  @since  2.9.0
   *  @param  string $key   setting to get.
   *  @param  string $group in which group the setting is.
   *  @return mixed         boolean false if setting group or setting is not found, otherwise it's value.
   */
  function get_custom_setting( $key, $group ) {
    $post_id = get_custom_settings_post_id( $group );
    if ( empty( $post_id ) ) {
      return false;
    }

    $value = get_field( $key, $post_id );
    if ( empty( $value ) ) {
      $value = get_field( "{$group}_{$key}", $post_id );
    }

    return $value;
  } // end get_custom_setting
} // end if

if ( ! function_exists( 'get_custom_settings_post_id' ) ) {
  /**
   * Get the custom settings group post id.
   *
   * Post id's are usually defined in the theme and air-light handles this automatically.
   *
   * @since  2.9.0
   * @param  string $group group key.
   * @return mixed         boolean false if settings post do not exist, otherwise integer post id.
   */
  function get_custom_settings_post_id( $group ) {
    $group_post_ids = apply_filters( 'air_helper_custom_settings_post_ids', [] );

    if ( ! isset( $group_post_ids[ $group ] ) ) {
      return false;
    }

    $post_id = pll_get_post( $group_post_ids[ $group ] ); // plugin backs us up if Polylang is not installed, no function check needed

    if ( empty( $post_id ) ) {
      // Maybe fallback to settings on main language if post is not translated
      $polylang_fallback_to_main = apply_filters( 'air_helper_custom_settings_polylang_fallback_main', false, $group, $group_post_ids );
      if ( ! $polylang_fallback_to_main ) {
        return false;
      }

      $post_id = $group_post_ids[ $group ];
    }

    return $post_id;
  } // end get_custom_settings_post_id
} // end if

if ( ! function_exists( 'use_block_editor_in_custom_setting_group' ) ) {
  /**
   * Check if to use block editor in setting group post.
   *
   * @since  2.9.0
   * @param  integer $post_id the post to check.
   * @return boolean          true if post should use block editor, otherwise false.
   */
  function use_block_editor_in_custom_setting_group( $post_id ) {
    $setting_group_post_ids = apply_filters( 'air_helper_custom_settings_post_ids', [] );

    $setting_group_post_ids = array_map( function( $post_id ) {
      return pll_get_post( $post_id );
    }, $setting_group_post_ids );

    if ( ! in_array( $post_id, $setting_group_post_ids, true ) ) {
      return false;
    }

    $block_editor_prefix = apply_filters( 'air_helper_custom_settings_block_editor_prefix', 'block-editor/' );

    $setting_group_post_ids_with_block_editor = array_filter(
      $setting_group_post_ids,
      function( $key ) use ( $block_editor_prefix ) {
        return false !== strpos( $key, $block_editor_prefix );
      },
      ARRAY_FILTER_USE_KEY
    );

    if ( in_array( $post_id, $setting_group_post_ids_with_block_editor, true ) ) {
      return true;
    }

    return false;
  } // end use_block_editor_in_custom_setting_group
} // end if

/**
 * Add post type support for editor depending on whether
 * to use block editor in current setting group
 *
 * @since  2.9.0
 */
add_action( 'admin_init', 'air_helper_editor_support_for_setting_group_post', 99, 1 );
function air_helper_editor_support_for_setting_group_post() {

  // Try find out which post id we are loading in admin
  $post_id = false;
  if ( isset( $_GET['post'] ) ) { // phpcs:ignore
    $post_id = intval( $_GET['post'] ); // phpcs:ignore
  } elseif ( isset( $_POST['post_ID'] ) ) { // phpcs:ignore
    $post_id = intval( $_POST['post_ID'] ); // phpcs:ignore
  }

  if ( ! $post_id ) {
    return;
  }

  if ( use_block_editor_in_custom_setting_group( $post_id ) ) {
    // Post type support 'editor' is needed for block editor
    add_post_type_support( 'settings', 'editor' );
  } else {
    remove_post_type_support( 'settings', 'editor' );
  }
} // end air_helper_set_editor_type_for_setting_group_post

/**
 * Check whether to use classic or block editor
 * for a certain post type as defined in the settings
 *
 * @since 2.9.0
 */
add_filter( 'use_block_editor_for_post', __NAMESPACE__ . '\air_helper_use_block_editor_in_custom_setting_group', 10, 2 );
function air_helper_use_block_editor_in_custom_setting_group( $use_block_editor, $post ) {
  $settings_post_types = apply_filters( 'air_helper_custom_settings_post_types', [ 'settings' ] );

  // Use block editor if settings page is a block editor type
  if ( in_array( $post->post_type, $settings_post_types, true ) ) {
    return use_block_editor_in_custom_setting_group( $post->ID );
  }

  return $use_block_editor;
} // end air_helper_use_block_editor_in_custom_setting_group
