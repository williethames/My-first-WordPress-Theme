# Fictional University WordPress Theme

This project is a custom WordPress theme developed for a fictional university website. The theme includes various features and functionalities to enhance the user experience, including custom REST API endpoints, dynamic page banners, and customized login screens.

## Features

### Custom REST API Endpoints

Generates custom fields for the REST API.

```php
/**
 * Registers custom REST API fields.
 *
 * Adds 'authorName' to the REST API for posts and 'userNoteCount' for notes.
 *
 * @return void
 */
function university_custom_rest() {
    register_rest_field(
        'post',
        'authorName',
        array(
            'get_callback' => function () {
                return get_the_author();
            }
        )
    );
    register_rest_field(
        'note',
        'userNoteCount',
        array(
            'get_callback' => function () {
                return count_user_posts(get_current_user_id(), 'note');
            }
        )
    );
}

add_action('rest_api_init', 'university_custom_rest');
```
### Dynamic Page Banners
Generates a customizable page banner with a title, subtitle, and background image.
```php
/**
 * Generates a customizable page banner with a title, subtitle, and background image.
 *
 * @param array|null $args Optional. Arguments for the banner. Default is null.
 * @return void
 */
function pageBanner($args = NULL) {
    if (!$args['title']) {
        $args['title'] = get_the_title();
    }
    if (!$args['subtitle']) {
        $args['subtitle'] = get_field('page_banner_subtitle');
    }
    if (!$args['photo']) {
        if (get_field('page_banner_background_image') && !is_archive() && !is_home()) {
            $args['photo'] = get_field('page_banner_background_image')['sizes']['pageBanner'];
        } else {
            $args['photo'] = get_theme_file_uri('/images/ocean.jpg');
        }
    }
    ?>
    <div class="page-banner">
        <div class="page-banner__bg-image" style="background-image: url(<?php echo $args['photo']; ?>);"></div>
        <div class="page-banner__content container container--narrow">
            <h1 class="page-banner__title"><?php echo $args['title']; ?></h1>
            <div class="page-banner__intro">
                <p><?php echo $args['subtitle']; ?></p>
            </div>
        </div>
    </div>
    <?php
}
```
### Enqueueing Scripts and Styles
Enqueues JavaScript and CSS files are required for the theme.
```php
/**
 * Enqueues JavaScript and CSS files for the theme.
 *
 * @return void
 */
function university_files() {
    wp_enqueue_script('googleMap', 'https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY', NULL, '1.0', true);
    wp_enqueue_script('main-university-js', get_theme_file_uri('/build/index.js'), array('jquery'), '1.0', true);
    wp_enqueue_style('custom-google-fonts', '//fonts.googleapis.com/css?family=Roboto+Condensed:300,300i,400,400i,700,700i|Roboto:100,300,400,400i,700,700i');
    wp_enqueue_style('font-awesome', '//maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css');
    wp_enqueue_style('university_main_styles', get_theme_file_uri('/build/style-index.css'));
    wp_enqueue_style('university_extra_styles', get_theme_file_uri('/build/index.css'));
    wp_localize_script('main-university-js', 'universityData', array(
        'root_url' => get_site_url(),
        'nonce' => wp_create_nonce('wp_rest')
    ));
}

add_action('wp_enqueue_scripts', 'university_files');

```
### Theme Support and Image Sizes
Adds theme support for various features and defines custom image sizes.
```php
/**
 * Adds theme support for various features and defines custom image sizes.
 *
 * @return void
 */
function university_features() {
    add_theme_support('title-tag');
    add_theme_support('post-thumbnails');
    add_image_size('professorLandscape', 400, 260, true);
    add_image_size('professorPortrait', 480, 650, true);
    add_image_size('pageBanner', 1500, 350, true);
}

add_action('after_setup_theme', 'university_features');

```
### Query Adjustments
Customizes the main queries for specific post type archives.

```php
/**
 * Customizes the main queries for specific post type archives.
 *
 * @param WP_Query $query The query object.
 * @return void
 */
function university_adjust_queries($query) {
    if (!is_admin() && is_post_type_archive('program') && $query->is_main_query()) {
        $query->set('orderby', 'title');
        $query->set('order', 'ASC');
        $query->set('posts_per_page', -1);
    }

    if (!is_admin() && is_post_type_archive('event') && $query->is_main_query()) {
        $today = date('Ymd');
        $query->set('meta_key', 'event_date');
        $query->set('orderby', 'meta_value_num');
        $query->set('order', 'ASC');
        $query->set('meta_query', array(
            array(
                'key' => 'event_date',
                'compare' => '>=',
                'value' => $today,
                'type' => 'numeric'
            )
        ));
    }
}

add_action('pre_get_posts', 'university_adjust_queries');
```
### Google Map API Key
Adds the Google Maps API key for ACF (Advanced Custom Fields) Google Map integration.

```php
/**
 * Adds the Google Maps API key for ACF Google Map integration.
 *
 * @param array $api The API settings array.
 * @return array
 */
function universityMapKey($api) {
    $api['key'] = 'YOUR_API_KEY';
    return $api;
}

add_filter('acf/fields/google_map/api', 'universityMapKey');

```
### Subscriber Redirection and Admin Bar
Redirects subscribers from the admin dashboard to the homepage and hides the admin bar for subscribers.
```php
/**
 * Redirects subscribers from the admin dashboard to the homepage.
 *
 * @return void
 */
function redirectSubsToFrontend() {
    $ourCurrentUser = wp_get_current_user();
    if (count($ourCurrentUser->roles) == 1 && $ourCurrentUser->roles[0] == 'subscriber') {
        wp_redirect(site_url('/'));
        exit();
    }
}

add_action('admin_init', 'redirectSubsToFrontend');

/**
 * Hides the admin bar for subscribers.
 *
 * @return void
 */
function noSubsAdminBar() {
    $ourCurrentUser = wp_get_current_user();
    if (count($ourCurrentUser->roles) == 1 && $ourCurrentUser->roles[0] == 'subscriber') {
        show_admin_bar(false);
        exit();
    }
}

add_action('admin_init', 'noSubsAdminBar');
```
### Custom Login Screen
Customizes the login screen URL, styles, and title.

```php
/**
 * Customizes the login screen URL.
 *
 * @return string
 */
function ourHeaderUrl() {
    return esc_url(site_url('/'));
}

add_filter('login_headerurl', 'ourHeaderUrl');

/**
 * Enqueues custom styles for the login screen.
 *
 * @return void
 */
function ourLoginCSS() {
    wp_enqueue_style('custom-google-fonts', '//fonts.googleapis.com/css?family=Roboto+Condensed:300,300i,400,400i,700,700i|Roboto:100,300,400,400i,700,700i');
    wp_enqueue_style('font-awesome', '//maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css');
    wp_enqueue_style('university_main_styles', get_theme_file_uri('/build/style-index.css'));
    wp_enqueue_style('university_extra_styles', get_theme_file_uri('/build/index.css'));
}

add_action('login_enqueue_scripts', 'ourLoginCSS');

/**
 * Customizes the login screen title.
 *
 * @return string
 */
function ourLoginTitle() {
    return get_bloginfo('name');
}

add_filter('login_headertitle', 'ourLoginTitle');
```
### Private Note Posts
Ensures that note posts are private and limits the number of notes a user can create.

```php
/**
 * Ensures that note posts are private and limits the number of notes a user can create.
 *
 * @param array $data Post data.
 * @param array $postarr Post array.
 * @return array Modified post data.
 */
function makeNotePrivate($data, $postarr) {
    if ($data['post_type'] == 'note') {
        if (count_user_posts(get_current_user_id(), 'note') > 1000 && !$postarr['ID']) {
            die("You have reached your note limit.");
        }

        $data['post_content'] = sanitize_textarea_field($data['post_content']);
        $data['post_title'] = sanitize_text_field($data['post_title']);
    }

    if ($data['post_type'] == 'note' && $data['post_status'] != 'trash') {
        $data['post_status'] = "private";
    }

    return $data;
}

add_filter('wp_insert_post_data', 'makeNotePrivate', 10, 2);
``
```
## Installation
Clone the repository to your WordPress themes directory.

Activate the theme from the WordPress admin dashboard.

Customize the theme settings as needed.
