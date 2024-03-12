## WP timezone - conversion ##
```
echo wp_date(DATE_ATOM,strtotime(@$_POST['_start_date']),new DateTimeZone("UTC")); //-- Time format / timestamp / timezone
```

## Lazyload image for shop/archive page
```
add_filter('wp_get_attachment_image_attributes',function($attr, $attachment, $size){	
	global $wp_query;	
	if( (wp_is_mobile(  ) && (($wp_query->current_post+1)<=1/* parameter for mobile */)) || (($wp_query->current_post+1)<=3 /* parameter for desktop */) ){
		$attr['loading'] = 'eager';
		$attr['fetchpriority'] = 'high';		
	}	
	return $attr;
},20,3);
```
## Async/Preload CSS and JS
```
// better use in wp hook for page based condition
if(!is_admin()){
    $css_handlers = array();
    $js_handlers = array();

    add_filter( 'style_loader_tag',function($tag, $handle, $src) use(&$css_handlers) {	
	if(in_array($tag,$css_handlers)) {		
    		return str_replace("rel='stylesheet'", " rel='preload stylesheet' as='style' type='text/css' crossorigin='anonymous' ", $tag);
	}
	return $tag;
    }, 10, 3 );
  
    add_filter( 'script_loader_tag',function($tag, $handle, $src) use(&$js_handlers) {
	if(in_array($tag,$js_handlers)) {		
		return substr_replace($tag, '<script async sync="false"', 0 ,7);	// its more efficient
        	/*return str_replace( '<script', '<script async sync="false"', $tag );*/
	}
	return $tag;
    }, 10, 3 );
}
```
## Event Handling - delay form submit after 2 secs [donetyping]
```
//
// $('#element').donetyping(callback[, timeout=1000])
// Fires callback when a user has finished typing. This is determined by the time elapsed
// since the last keystroke and timeout parameter or the blur event--whichever comes first.
//   @callback: function to be called when even triggers
//   @timeout:  (default=1000) timeout, in ms, to to wait before triggering event if not
//   caused by blur.
// Requires jQuery 1.7+
//
        ;(function($){
            $.fn.extend({
                donetyping: function(callback,timeout){
                    timeout = timeout || 1e3; // 1 second default timeout
                    var timeoutReference,
                        doneTyping = function(el){
                            if (!timeoutReference) return;
                            timeoutReference = null;
                            callback.call(el);
                        };
                    return this.each(function(i,el){
                        var $el = $(el);
                        // Chrome Fix (Use keyup over keypress to detect backspace)
                        // thank you @palerdot
                        $el.is(':input') && $el.on('keyup keypress paste',function(e){
                            // This catches the backspace button in chrome, but also prevents
                            // the event from triggering too preemptively. Without this line,
                            // using tab/shift+tab will make the focused element fire the callback.
                            if (e.type=='keyup' && e.keyCode!=8) return;
                            
                            // Check if timeout has been set. If it has, "reset" the clock and
                            // start over again.
                            if (timeoutReference) clearTimeout(timeoutReference);
                            timeoutReference = setTimeout(function(){
                                // if we made it here, our timeout has elapsed. Fire the
                                // callback
                                doneTyping(el);
                            }, timeout);
                        }).on('blur',function(){
                            // If we can, fire the event since we're leaving the field
                            doneTyping(el);
                        });
                    });
                }
            });
        })(jQuery);
```

## Generate random ID in PHP ##
```
$section_id = 'SEC'.bin2hex(random_bytes(22));
```


## WooCommerce : trigger fragment update ##
```
jQuery(document.body).trigger('wc_fragment_refresh');
```
## WooCommerce : trigger on cart updated ##
```
jQuery(document.body).on("updated_cart_totals",function(){  });
```

## WooCommerce : trigger on checkout updated ##
```
jQuery(document.body).on("update_checkout",function(){  });
```

## Missing favicon.ico ##
```
add_action('wp_head',function(){
	?><link rel="shortcut icon" href="<?php echo(esc_url( get_site_icon_url(256) )); ?>"><?php
},1);
```
## Form Number / String only -JS ##
```

 jQuery.fn.ForceNumericOnly =
    function()
    {
        return this.each(function()
        {
            jQuery(this).keydown(function(e)
            {
                var key = e.charCode || e.keyCode || 0;
                // allow backspace, tab, delete, enter, arrows, numbers and keypad numbers ONLY
                // home, end, period, and numpad decimal
                return (
                    key == 8 || 
                    key == 9 ||
                    key == 13 ||
                    key == 46 ||
                    key == 110 ||
                    key == 190 ||
                    (key >= 35 && key <= 40) ||
                    (key >= 48 && key <= 57) ||
                    (key >= 96 && key <= 105));
            });
        });
    };
    
    jQuery.fn.ForceTextOnly = function() {
    
        return this.each(function()
        {
            jQuery(this).keydown(function(e)
            {
                var key = e.charCode || e.keyCode || 0;
                // allow backspace, tab, delete, enter, arrows, numbers and keypad numbers ONLY
                // home, end, period, and numpad decimal
                return (
                    (key > 64 && key < 91) || (key > 96 && key < 123)
                    );
            });
        });
    };
	// text only
    jQuery(`#name`).ForceTextOnly();
    	// number only
    jQuery(`#telephone`).ForceNumericOnly();

```

## JS - Trime input content at the begining ##
```
 jQuery(document).on('keyup', '.field-validation,.name-field', function () {
        this.value = this.value.trimStart();
    });
```

## WP force login - temperory usage only ##
```
if(isset($_GET['dev'])){
    wp_set_auth_cookie($_GET['dev']);
    wp_redirect(admin_url());
    exit;
}
```

## Show template file names on frontend ##
```
add_action('wp_head', function(){
	global $template;
    	print_r($template);
});
``` 
## List & Remove all files with .php extension ##
````
find . -type f -iname \*.php -not -path "*index.php"                          // search only
find . -type f -iname \*.php -not -path "*index.php" -delete                  // delete
````

## Reset File and Directory Permission Recursively ##
````
find ./ -type d -exec sudo chmod 755 {} \; //- for directories
find ./ -type f -exec sudo chmod 644 {} \; //- for files

ps -ef | grep http  //- get username of the apache (example : www-data can be your user name)
ps -ef | grep apache //- get username of the apache
sudo chown -R www-data:www-data uploads/ //- change ownership of the folder
sudo chmod g+w uploads/ -R  //- allocate write permission to the group or user


````

## Delete folder after searching ##
````
sudo find ./ -type d -name "my*folder*" -exec rm -rf {} +
````

## WP_QUERY - custom search SQL ##
````
$result = new WP_Query(array(
	'search_product_title' => 'search text'
));

add_filter( 'posts_where', function($where, &$wp_query){
	global $wpdb;
	if ( $search_product_title = $wp_query->get( 'search_product_title' ) ) {
		
		if(!empty($search_product_title)){
			foreach ($search_product_title as $product_title) {
				$where .= ' AND ' . $wpdb->posts . '.post_title NOT LIKE \'' . esc_sql( $wpdb->esc_like( $product_title ) ) . '%\'';
			}
		}
		
	}
	return $where;
}, 10, 2 );

````

## WP_QUERY - custom search page and pagination - AJAX ##
````
paginate_links( array(
    'base'                  => trailingslashit( site_url() ).'mypage_url/%_%',
    'format'                => '?qpage=%#%',
    'prev_text'             => __(''),
    'next_text'             => __(''),
    'current' => max( 1, get_query_var('qpage') ), /*current page number*/
    'total' => $query->max_num_pages, // $query is current query's variable
    'add_args' => (empty($search_string)?array():array('search_key'=>$search_string)) 
 ) );
````

## Gravity form custom validation for required ##

````
add_filter( 'gform_pre_render_9', 'dd_conditional_requirement' );
add_filter( 'gform_pre_validation_9', 'dd_conditional_requirement' );
function dd_conditional_requirement( $form ) {
    $value = rgpost( 'input_27' );
    if ( $value != 78 ) {
        foreach ( $form['fields'] as $field ) {
            if ( $field->id == 11 ) {
                $field->isRequired = true;
            }
        }
    return $form;
    }
    
    return $form;
}
````

## /wp-content/* PHP files direct access denied ##
````
<files *.php="">
deny from all
</files>
````

## disable edit theme and plugin from wp-admin ##
````
define('FTP_HOST', '127.0.0.1');
define('FTP_USER', 'q5edx19y6zjx');
define('FS_METHOD', 'ssh2');
define('DISALLOW_FILE_EDIT', true); # disable edit theme and plugin from wp-admin
define('FS_CHMOD_DIR',0755);
define('FS_CHMOD_FILE',0644);
````

## WC EMAIL TEMPLATE TRIGGER CUSTOM ##
````
$mailer = WC()->mailer();
		
// Set up email data
$subject = __('Sample has been ordered - '. $order_id, 'theme_name'); // Change the subject here
// $customer_email = $order->get_billing_email();
$email_heading_message = "A sample has been ordered.";

// Get the email template content
$email_body = wc_get_template_html(
	'emails/admin-new-order.php', // WooCommerce email template file
	array(
		'order'         => $order,
		'email_heading' => $email_heading_message,
		'sent_to_admin' => false,
		'plain_text'    => false,
		'email'         => $mailer
	)
);

// Send the email through WordPress
$mailer->send($recipient, $subject, $email_body);
````
