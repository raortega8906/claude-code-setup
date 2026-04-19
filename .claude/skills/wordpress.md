# Skill: WordPress Development

> Claude carga este skill cuando trabaja en código WordPress.
> Aplica tanto a temas hijos (functions.php) como a desarrollo de plugins.

## Entorno WordPress
- WordPress: ver `wp-includes/version.php`
- PHP: 8.2+ con strict_types
- WP-CLI disponible: `wp [comando]`
- Prefijo de proyecto: `[DEFINIR_PREFIJO]_` (ej: `miapp_`)

## Reglas absolutas de WordPress

### Siempre sanitizar inputs
```php
// ✅ Siempre sanitizar antes de usar
$nombre = sanitize_text_field( $_POST['nombre'] );
$email  = sanitize_email( $_POST['email'] );
$id     = absint( $_GET['id'] );
$url    = esc_url_raw( $_POST['url'] );
$html   = wp_kses_post( $_POST['contenido'] );

// ❌ NUNCA usar datos del usuario sin sanitizar
$nombre = $_POST['nombre'];
```

### Siempre escapar outputs
```php
// ✅ Siempre escapar antes de mostrar
echo esc_html( $titulo );
echo esc_attr( $valor_atributo );
echo esc_url( $enlace );
echo wp_kses_post( $contenido_html );
?><a href="<?php echo esc_url( $url ); ?>"><?php echo esc_html( $texto ); ?></a>

// ❌ NUNCA hacer echo directo de variables
echo $titulo;
```

### Siempre usar nonces
```php
// En el formulario
wp_nonce_field( 'miapp_guardar_datos', 'miapp_nonce' );

// En el handler
if ( ! check_admin_referer( 'miapp_guardar_datos', 'miapp_nonce' ) ) {
    wp_die( 'Acción no autorizada.' );
}

// En AJAX
wp_nonce_field( 'miapp_ajax', 'nonce' );
// JS: data: { nonce: miapp_vars.nonce, ... }
// PHP: check_ajax_referer( 'miapp_ajax', 'nonce' );
```

### Siempre verificar permisos
```php
if ( ! current_user_can( 'edit_posts' ) ) {
    wp_die( 'Sin permisos.' );
}
```

## Desarrollo de tema hijo — functions.php

### Estructura recomendada
```php
<?php
declare(strict_types=1);

/**
 * Tema hijo: [Nombre del Tema]
 * Prefijo de funciones: miapp_
 */

// Evitar acceso directo
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

// Cargar estilos del tema padre
add_action( 'wp_enqueue_scripts', 'miapp_enqueue_styles' );
function miapp_enqueue_styles(): void {
    wp_enqueue_style(
        'parent-style',
        get_template_directory_uri() . '/style.css'
    );
}

// Incluir archivos de funcionalidad separados
require_once get_stylesheet_directory() . '/includes/custom-post-types.php';
require_once get_stylesheet_directory() . '/includes/shortcodes.php';
require_once get_stylesheet_directory() . '/includes/ajax-handlers.php';
```

### Convenciones de hooks
```php
// Nombrado: prefijo_descripción_acción
add_action( 'wp_head', 'miapp_add_meta_tags' );
add_filter( 'the_title', 'miapp_modify_title' );

// Prioridad solo si es necesario y comentada
add_action( 'wp_footer', 'miapp_load_scripts', 20 ); // después del tema padre (10)
```

## Desarrollo de plugins

### Estructura de plugin
```
mi-plugin/
├── mi-plugin.php              # Archivo principal (header del plugin)
├── includes/
│   ├── class-mi-plugin.php    # Clase principal (singleton)
│   ├── class-admin.php        # Funcionalidad de admin
│   ├── class-public.php       # Funcionalidad pública
│   └── class-ajax.php         # Handlers AJAX
├── admin/
│   ├── css/
│   ├── js/
│   └── views/                 # Templates del admin
├── public/
│   ├── css/
│   ├── js/
│   └── views/
├── languages/                 # i18n
└── uninstall.php              # Limpiar al desinstalar
```

### Archivo principal del plugin
```php
<?php
/**
 * Plugin Name:       Mi Plugin
 * Plugin URI:        https://ejemplo.com
 * Description:       Descripción breve.
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      8.2
 * Author:            Tu Nombre
 * License:           GPL v2 or later
 */

declare(strict_types=1);

if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

define( 'MIPLUGIN_VERSION', '1.0.0' );
define( 'MIPLUGIN_PATH', plugin_dir_path( __FILE__ ) );
define( 'MIPLUGIN_URL', plugin_dir_url( __FILE__ ) );

require_once MIPLUGIN_PATH . 'includes/class-mi-plugin.php';

function miplugin(): MiPlugin {
    return MiPlugin::get_instance();
}
add_action( 'plugins_loaded', 'miplugin' );
```

### Clase principal (singleton)
```php
<?php
class MiPlugin {
    private static ?self $instance = null;

    public static function get_instance(): self {
        if ( null === self::$instance ) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    private function __construct() {
        $this->define_hooks();
    }

    private function define_hooks(): void {
        add_action( 'init', [ $this, 'init' ] );
        add_action( 'admin_menu', [ $this, 'add_admin_menu' ] );
    }
}
```

### AJAX en WordPress
```php
// Registrar handlers (hooks con y sin login)
add_action( 'wp_ajax_miapp_accion', 'miapp_handle_ajax' );
add_action( 'wp_ajax_nopriv_miapp_accion', 'miapp_handle_ajax' ); // si es público

function miapp_handle_ajax(): void {
    check_ajax_referer( 'miapp_nonce', 'nonce' );
    
    if ( ! current_user_can( 'read' ) ) {
        wp_send_json_error( [ 'message' => 'Sin permisos' ], 403 );
    }
    
    $dato = sanitize_text_field( $_POST['dato'] ?? '' );
    
    // lógica...
    
    wp_send_json_success( [ 'resultado' => $resultado ] );
}

// Pasar datos a JS
add_action( 'wp_enqueue_scripts', 'miapp_enqueue_scripts' );
function miapp_enqueue_scripts(): void {
    wp_enqueue_script( 'miapp-main', MIPLUGIN_URL . 'public/js/main.js', ['jquery'], MIPLUGIN_VERSION, true );
    wp_localize_script( 'miapp-main', 'miapp_vars', [
        'ajax_url' => admin_url( 'admin-ajax.php' ),
        'nonce'    => wp_create_nonce( 'miapp_nonce' ),
    ]);
}
```

## Custom Post Types y Taxonomías
```php
function miapp_register_post_types(): void {
    register_post_type( 'miapp_producto', [
        'labels'      => [
            'name'          => __( 'Productos', 'mi-plugin' ),
            'singular_name' => __( 'Producto', 'mi-plugin' ),
        ],
        'public'      => true,
        'has_archive' => true,
        'supports'    => [ 'title', 'editor', 'thumbnail', 'excerpt' ],
        'rewrite'     => [ 'slug' => 'productos' ],
        'show_in_rest' => true, // habilitar Gutenberg y REST API
    ]);
}
add_action( 'init', 'miapp_register_post_types' );
```

## Base de datos personalizada
```php
// Usar $wpdb con placeholders — NUNCA interpolación directa
global $wpdb;
$tabla = $wpdb->prefix . 'miapp_datos';

// ✅ Correcto
$resultado = $wpdb->get_results(
    $wpdb->prepare( "SELECT * FROM {$tabla} WHERE user_id = %d AND status = %s", $user_id, 'activo' )
);

// ❌ NUNCA
$resultado = $wpdb->get_results( "SELECT * FROM {$tabla} WHERE user_id = $user_id" );
```

## WP-CLI — Comandos útiles
```bash
wp plugin list
wp plugin activate mi-plugin
wp cache flush
wp rewrite flush
wp option get siteurl
wp user list
wp db export backup.sql
wp search-replace 'http://dev.local' 'https://produccion.com' --dry-run
```
