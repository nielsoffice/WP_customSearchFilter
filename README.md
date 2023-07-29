# WP_customSearchFilter
WordPress admin all post type adding custom search filter 

```PHP

if (is_admin()){
  //this hook will create a new filter on the admin area for the specified post type
  add_action( 'restrict_manage_posts', function(){
       global $wpdb, $table_prefix;

       $post_type = (isset($_GET['post_type'])) ? $_GET['post_type'] : 'post';

       //only add filter to post type you want
       if ($post_type == 'page'){
           //query database to get a list of years for the specific post type:
           $values = array();
           $query_years = $wpdb->get_results("SELECT year(post_date) as year from ".$table_prefix."posts
                   where post_type='".$post_type."'
                   group by year(post_date)
                   order by post_date");
           foreach ($query_years as &$data){
               $values[$data->year] = $data->year;
           }

           //give a unique name in the select field
           ?><select name="admin_filter_year">
                <option value="">All years</option>

               <?php 
               $current_v = isset($_GET['admin_filter_year'])? $_GET['admin_filter_year'] : '';
               foreach ($values as $label => $value) {
                   printf(
                       '<option value="%s"%s>%s</option>',
                       $value,
                       $value == $current_v? ' selected="selected"':'',
                       $label
                   );
               }
               ?>
           </select>
           <?php
       }
   });

   //this hook will alter the main query according to the user's selection of the custom filter we created above:
   add_filter( 'parse_query', function($query){
       global $pagenow;
       $post_type = (isset($_GET['post_type'])) ? $_GET['post_type'] : 'post';

       if ($post_type == 'page' && $pagenow=='edit.php' && isset($_GET['admin_filter_year']) && !empty($_GET['admin_filter_year'])) {
           $query->query_vars['year'] = $_GET['admin_filter_year'];
       }
   });
}

```

<br /> Source: 
<br /> https://www.lab21.gr/blog/wordpress-add-custom-filters-post-list-admin-area/
