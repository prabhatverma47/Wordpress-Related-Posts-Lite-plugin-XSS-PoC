# Related Posts Lite XSS PoC

Proof of Concept (PoC) for a **Stored Cross-Site Scripting (XSS)** vulnerability in the WordPress plugin **Related Posts Lite** (slug: `related-posts-lite`). 
Tested version : v1.12

The issue exists because the plugin saves unsanitized option values from its settings page and later renders them without proper escaping in the Related Posts output. This allows attackers to inject arbitrary JavaScript that executes in the context of WordPress, leading to account takeover or other malicious actions. Additionally, the settings form lacks nonce and capability checks, making exploitation possible via CSRF.

---

## Vulnerability Information

The issue is caused by the plugin saving settings in `backend/settings.php` using:

update_option('rpl_options', $_POST);

This stores all POST data directly without sanitization, escaping, capability checks, or nonce validation. These values, such as the **Plugin title**, are later retrieved and echoed on the frontend by the `[wpdreams_rpl]` shortcode without functions like `esc_html()`.

As a result, malicious input (e.g. `<svg onload=alert(1)>`) is stored in the database and executed when related posts are displayed, making this a **stored XSS** that can also be exploited via CSRF.

---

### Exploitation Flow

[Attacker/CSRF]  
    ↓  (malicious POST to wp-admin/admin.php?page=related-posts-lite)  
backend/settings.php  
    ↓  (unsanitized update_option)  
Database (wp_options → rpl_options)  
    ↓  (retrieved when rendering shortcode)  
Frontend Output (echo)  
    ↓  
JavaScript Execution (Stored XSS)  

---

## PoC Steps

1. Install WordPress and download the official plugin:  
   https://wordpress.org/plugins/related-posts-lite/  
   (this will add the "Related Posts Lite" menu item in the wp-admin panel).





2. In **wp-admin**, go to **Related Posts Lite → Layout Options**, under **Plugin title** insert the payload:  

   `<svg onload=alert(1)>`  





   Save the settings (the plugin stores this value directly into the database without sanitization or escaping).

3. Create a new post via **Posts → Add New**, then publish it (the related posts section is automatically appended or can be displayed using the `[wpdreams_rpl]` shortcode).  



4. Visit the published post in the frontend (example: `http://127.0.0.1:8080/?p=13`) and the malicious payload is retrieved from the database and echoed unescaped, causing the JavaScript to execute and proving the stored XSS vulnerability.  



---

## Prevention

To mitigate this issue the plugin should sanitize all incoming POST values before saving with functions such as `sanitize_text_field()` or `esc_html()`, and escape all output using WordPress helpers like `esc_html()`, `esc_attr()`, or `wp_kses_post()` depending on context.

---

## Disclaimer

This PoC is for educational and research purposes only. Do not use it on systems you do not own or have permission to test.
