# ChatGPT — Plugin List Posts

Here's a high-level overview of how you can create a Wordpress plugin called "list posts" that lists the categories and tags of the last 10 posts and exports them to a CSV file:

Start by creating a new plugin folder and file in the wp-content/plugins directory of your Wordpress installation. For example, you could name the folder list-posts and the main file list-posts.php.

In the main plugin file, define the plugin's basic information such as its name, version, and description using Wordpress' plugin header comment block. You can also use this block to define any dependencies that your plugin may have.
```
<?php
/*
Plugin Name: List Posts
Plugin URI: https://localhost/plugins/list-posts
Description: A plugin that lists the categories and tags of the last 10 posts and exports them to a CSV file.
Version: 1.0.0
Author: Italo Mariano GPT
Author URI: https://www.linkedin.com/in/italomsr/
License: GPLv2 or later
Text Domain: list-posts
*/
```

Next, create a new class for your plugin and define its basic structure. In this class, you can add methods to handle various tasks such as registering settings, creating the custom admin screen, and exporting the data to a CSV file.

```
class List_Posts {

    public function __construct() {
        // Register settings
    }

    public function create_admin_screen() {
        // Create the custom admin screen
    }

    public function export_to_csv() {
        // Export data to a CSV file
    }
}

```
In the __construct() method, register any settings that your plugin uses. For example, you can register a setting to store the number of posts to list.

```
public function __construct() {
    register_setting( 'list-posts-settings', 'list_posts_num_posts', array(
        'type' => 'integer',
        'default' => 10,
        'sanitize_callback' => 'absint',
    ) );
}

```
In the create_admin_screen() method, create the custom admin screen for your plugin. This screen should display the categories and tags of the last 10 posts, as well as an export to CSV button.


```
public function create_admin_screen() {
    // Check if the user has the required permissions
    if ( ! current_user_can( 'manage_options' ) ) {
        return;
    }

    ?>
    <div class="wrap">
        <h1><?php esc_html_e( 'List Posts', 'list-posts' ); ?></h1>

        <table class="widefat striped">
            <thead>
                <tr>
                    <th><?php esc_html_e( 'Post Title', 'list-posts' ); ?></th>
                    <th><?php esc_html_e( 'Categories', 'list-posts' ); ?></th>
                    <th><?php esc_html_e( 'Tags', 'list-posts' ); ?></th>
                </tr>
            </thead>
            <tbody>
                <?php
                $num_posts = get_option( 'list_posts_num_posts', 10 );

               
 ```
To populate the table with the last 10 posts' categories and tags, you can use the get_posts() function to retrieve the posts and then use the get_the_category() and get_the_tags() functions to get the categories and tags for each post.

```
public function create_admin_screen() {
  // Check if the user has the required permissions
  if ( ! current_user_can( 'manage_options' ) ) {
      return;
  }

  ?>
  <div class="wrap">
      <h1><?php esc_html_e( 'List Posts', 'list-posts' ); ?></h1>

      <table class="widefat striped">
          <thead>
              <tr>
                  <th><?php esc_html_e( 'Post Title', 'list-posts' ); ?></th>
                  <th><?php esc_html_e( 'Categories', 'list-posts' ); ?></th>
                  <th><?php esc_html_e( 'Tags', 'list-posts' ); ?></th>
              </tr>
          </thead>
          <tbody>
              <?php
              $num_posts = get_option( 'list_posts_num_posts', 10 );
              $posts = get_posts( array(
                  'numberposts' => $num_posts,
              ) );

              foreach ( $posts as $post ) {
                  $categories = get_the_category( $post->ID );
                  $tags = get_the_tags( $post->ID );

                  ?>
                  <tr>
                      <td><?php echo esc_html( $post->post_title ); ?></td>
                      <td>
                          <?php
                          if ( ! empty( $categories ) ) {
                              $category_names = wp_list_pluck( $categories, 'name' );
                              echo esc_html( implode( ', ', $category_names ) );
                          } else {
                              esc_html_e( 'None', 'list-posts' );
                          }
                          ?>
                      </td>
                      <td>
                          <?php
                          if ( ! empty( $tags ) ) {
                              $tag_names = wp_list_pluck( $tags, 'name' );
                              echo esc_html( implode( ', ', $tag_names ) );
                          } else {
                              esc_html_e( 'None', 'list-posts' );
                          }
                          ?>
                      </td>
                  </tr>
                  <?php
              }
              ?>
          </tbody>
      </table>

      <p>
          <a href="<?php echo esc_url( admin_url( 'admin-post.php?action=list_posts_export' ) ); ?>" class="button button-primary"><?php esc_html_e( 'Export to CSV', 'list-posts' ); ?></a>
      </p>
  </div>
  <?php
}
```


Finally, in the export_to_csv() method, you can retrieve the same data as in the `create_admin_screen
```
public function export_to_csv() {
  // Check if the user has the required permissions
  if ( ! current_user_can( 'manage_options' ) ) {
      wp_die( esc_html__( 'You do not have sufficient permissions to access this page.', 'list-posts' ) );
  }

  $num_posts = get_option( 'list_posts_num_posts', 10 );
  $posts = get_posts( array(
      'numberposts' => $num_posts,
  ) );

  // Set the CSV header
  header( 'Content-Type: text/csv' );
  header( 'Content-Disposition: attachment; filename="list-posts.csv"' );
  header( 'Pragma: no-cache' );
  header( 'Expires: 0' );

  // Open the output stream
  $output = fopen( 'php://output', 'w' );

  // Write the column headers
  fputcsv( $output, array(
      'Post Title',
      'Categories',
      'Tags',
  ) );

  foreach ( $posts as $post ) {
      $categories = get_the_category( $post->ID );
      $tags = get_the_tags( $post->ID );

      $category_names = array();
      if ( ! empty( $categories ) ) {
          $category_names = wp_list_pluck( $categories, 'name' );
      }

      $tag_names = array();
      if ( ! empty( $tags ) ) {
          $tag_names = wp_list_pluck( $tags, 'name' );
      }

      // Write the post data to the CSV file
      fputcsv( $output, array(
          $post->post_title,
          implode( ', ', $category_names ),
          implode( ', ', $tag_names ),
      ) );
  }

  // Close the output stream
  fclose( $output );

  // Stop WordPress from executing further
  exit;
}
```

To actually display the custom admin screen and handle the export to CSV action, you can use Wordpress' admin_menu and admin_post actions.


```
function list_posts_admin_menu() {
  add_menu_page(
      __( 'List Posts', 'list-posts' ),
      __( 'List Posts', 'list-posts' ),
      'manage_options',
      'list-posts',
      array( 'List_Posts', 'create_admin_screen' ),
      'dashicons-admin-generic',
      25
  );
}
add_action( 'admin_menu', 'list_posts_admin_menu' );

function list_posts_export() {
  $list_posts = new List_Posts();
  $list_posts->export_to_csv();
}
add_action( 'admin_post_list_posts_export', 'list_posts_export' );
```
And that's it! This should be a good starting point for creating a Wordpress plugin that lists the categories and tags of the last 10 posts and tags.






